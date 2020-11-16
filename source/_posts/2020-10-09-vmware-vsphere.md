---
title: VMware vSphere 之 vCenter 指标采集
date: 2020-10-09 18:06:10
tags: 
- VMware vSphere
- vCenter
- 虚拟化
- 技术文章
categories: 
- Python
- VMware
---

> ### 摘要

VMware vSphere 是 VMware 的虚拟化平台，可将数据中心转换为包括 CPU、存储和网络资源的聚合计算基础架构。vSphere 将这些基础架构作为一个统一的运行环境进行管理，并提供工具来管理加入该环境的数据中心。

在对接或采集VMware vSphere虚拟化平台的场景中，比如配置八爪鱼，需要对其中的虚拟化数据中心、虚拟化集群、物理机、虚拟机、存储等指标进行多方位的采集，我们根据VMware提供的相应sdk接口，对接到蓝鲸的SaaS中。
<!--more-->
> ### 正文

### 一   vSphere的模型结构

#### 1.1    模型及关联关系

##### 1.1.1   名词解释

VCenter: 虚拟机管理中心，

DCenter: 虚拟化数据中心，

Cluster: 虚拟化集群

Server: 物理机

Virtual: 虚拟机

Storage: 存储

##### 1.1.2   从属关系

从属关系: VCenter => DCenter => Cluster => Server => Virtual & Storage

##### 1.1.3   指标采集

以上是最常见、最标准的多级结构，管理平台到数据中心，到集群，再到物理机，物理机包含虚拟机和存储单元。但是这并不是固定的，某些情况，我们也可以直接把物理机挂载在数据中心DCenter中，而存储也可以直接关联到集群，这就导致了我们采集各项指标的时候，事先并不知道客户环境是否按照标准化的结构来构建VCenter，关联关系也各不相同，导致采集到的数据与CMDB模型和关联关系不适配。这些问题需要调研的时候确定环境的VCenter结构，并且CMDB建模时与之适配，这才能使得数据采集上报工作正常开展。

### 二   vSphere采集

#### 2.1    对接vSphere

##### 2.1.1   使用VMware官方sdk

VMware官方提供了多种多种sdk，由于我们使用Django开发SaaS，直接使用pip安装依赖即可，版本号由vSphere版本确定，现最新版为7.0，6.x是最常见的版本。

```shell
pip install pyvmomi==6.5
```



##### 2.1.2   连接vSphere

采集之前，我们需要提供vSphere的地址，ip或者有DNS的域名都可以，以及需要vSphere的管理员账号和密码，账号通常为administrator@vsphere.local。有了以上信息后，我们通过ssl模块连接到vSphere，具体实现如下：

```python
def connect_vc(vcserver):
    try: 
        context = ssl.SSLContext(ssl.PROTOCOL_TLSv1) 
        context.verify_mode = ssl.CERT_NONE  
        si = SmartConnect(host=vcserver["ip"], user=vcserver["user"], pwd=vcserver["password"], port=443, sslContext=context)  
    except:
        context = ssl._create_unverified_context()  
        context.verify_mode = ssl.CERT_NONE  
        si = SmartConnect(host=vcserver["ip"], user=vcserver["user"], pwd=vcserver["password"], port=443, sslContext=context)  
    return si.RetrieveContent()
```



#### 2.2    采集模型指标

由于vSphere是层级结构，我们想要采集某一层的指标，就需要从上往下逐层采集，开发时，我们需要引入vSphere的sdk。

```python
from pyVim.connect import SmartConnect 
from pyVmomi import vim, vmodl 
```



##### 2.2.1   VCenter

VCenter是关联关系最上的一层结构，获取VCenter的信息：

```python
def get_vc_info(content): 
    """获取VC服务器配置信""" 
    hostname = content.setting.QueryOptions("VirtualCenter.FQDN")[0].value 
    ver = content.about.version 
    licensesinfo = [{"name": l.name, "license": l.licenseKey, "costUnit": l.costUnit,  "total": l.total if l.total != 0 else "Unlimited", "used": l.used} for l in content.licenseManager.licenses] 
    return {"hostname": hostname, "version": ver, "licensesinfo": licensesinfo} 
```

如上，使用connect_vc连接vSphere返回的content作为参数，实例代码中展示了采集版本和licenses的信息，更多的参数如日志的等级、日志文件大小及数据库信息等，可以自行添加，不做展示。而想要采集该VCenter下有多少DCenter，则需要往下继续采集统计。

##### 2.2.2   DCenter

同上，由于一个vSphere只有一个VCenter，所以我们可以直接采集到所有DCenter，不需要额外的参数。

```python
def get_dcenter(self):
    container = self.connect.viewManager.CreateContainerView(self.connect.rootFolder, [vim.Datacenter], True)
    for dcenter in container.view:
        dcenter_name = dcenter.name
        dcenter_moid = dcenter._moId
    return dcenter_moid
```

self.connect为连接成功vSphere后返回的content，这样我们就采集到了DCenter的名称及moid（唯一标识），同理，需要额外参数可自行添加，官方文档有详细指标说明。

##### 2.2.3   Cluster

Cluster集群是属于DCenter数据中心下的结构，所以我们需要通过数据中心的moid来获取对应集群信息。

```python
def get_cluster(self, dcenter_list):
    for dcenter_data in dcenter_list:
        moId = dcenter_data["detail"]["moId"]
        com_info = vm_helper.get_obj_bymoId(self.connect, [vim.Datacenter], moId)
        container = self.connect.viewManager.CreateContainerView(com_info.hostFolder, [vim.ComputeResource], True)
        for cluster in container.view:
            cluster_moid = cluster._moId
            get_cluster_resource(content, cluster_moid)
```

以上代码通过DCenter的moid获取了属于它的全部Cluster信息，我们使用cluster._moId，便可采集到集群的多项指标，也是最常见的需求。

```python
def get_cluster_resource(self, content, cluster_moId):
    """获取群集资源概况"""
    cluster = get_obj_bymoId(content, [vim.ClusterComputeResource], cluster_moId)
    summary = cluster.summary
    totalCpuMhz = summary.totalCpu
    totalMemMB = summary.totalMemory
    capacity_list = []
    freeSpace_list = []
    for datastore in cluster.datastore:
        capacity_list.append(datastore.summary.capacity)
        freeSpace_list.append(datastore.summary.freeSpace)
    totalDiskTB = "%.3f" % (float(sum(capacity_list)) / 1024 / 1024 / 1024 / 1024)
    DiskUsedTB = "%.3f" % (float(sum(capacity_list) - sum(freeSpace_list)) / 1024 / 1024 / 1024 / 1024)
    resource_quickstats = cluster.resourcePool.summary.quickStats
    cpuUsedMhz = resource_quickstats.overallCpuUsage
    memUsedMB = resource_quickstats.hostMemoryUsage
    return cluster
```

以上代码中，我演示了如何获取集群CPU、内存、磁盘的一些指标，包括总容量和已使用量等，同理，如需更多指标，参照官方文档的字段说明。

##### 2.2.4   Server

物理机是隶属于集群下的，只需提供集群的moid，便可获取物理机列表，物理机各项指标通过对应物理机的moid获取，参照官方文档字段说明，不多赘述。

```python
def get_server(self, cluster_list):
    for cluster_data in cluster_list:
        moId = cluster_data["detail"]["moId"]
        com_info = vm_helper.get_obj_bymoId(self.connect, [vim.ComputeResource], moId)
        for server in com_info.host:
            server_name = server.name
            server_moid = server._moId
```



##### 2.2.5   Virtual & Storage

虚拟机和存储是挂载在物理机下的，采集方法相同，以下只列出虚拟机的采集方法。

```python
def get_vm(self, server):
    for vm in server.vm:
        if vm.summary.guest.ipAddress:
            vm_ip = vm.summary.guest.ipAddress
            vm_name = vm.name
            vm_moid = vm.moid
```

这样就可以采集到虚拟机的ip和名称，适用moid可以同集群一般，采集更多指标。

 

### 三   个人总结

#### 3.1    适用场景分析

在大多情况下，以上vSphere采集是获取vSphere资产全貌的手段，不推荐作为监控的手段，尤其是在庞大体量的VCenter中，循环且频繁地调用sdk不是长久的方法，或许会对本身产生未知影响。vSphere采集适用的场景为：适用配置八爪鱼获取VCenter下所有层级的实例数据和关联关系，然后录入CMDB，主要为虚拟机的IP，通过IP安装可监控指标的Agent，实现对物理机、虚拟机和存储的指标和健康度。

******

更多指标采集参考官方文档：

https://vdc-download.vmware.com/vmwb-repository/dcr-public/6b586ed2-655c-49d9-9029-bc416323cb22/fa0b429a-a695-4c11-b7d2-2cbc284049dc/doc/index-methods.html