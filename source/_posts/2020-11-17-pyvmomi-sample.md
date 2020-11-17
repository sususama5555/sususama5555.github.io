---
title: Python之使用pyvmomi管理VMware
date: 2020-11-17 23:40:48
tags:
- Python
- VMware vSphere
- vCenter
- pyvmomi
- 虚拟化
categories:
- Python
- 虚拟化
---

> ### 摘要

官方定义：

pyVmomi is the Python SDK for the VMware vSphere API that allows you to manage ESX, ESXi, and vCenter.

[pyVmomi](https://link.jianshu.com/?t=https://github.com/vmware/pyvmomi/) 是 VMware vSphere API 的一个 Python sdk，我们可以利用它来管理与交互vCenter、ESX、ESXi，获取我们需要的信息。

由于工作中需要对接vCenter，实现虚拟化平台、数据中心、物理机、物理机和存储的指标采集及监控，也需要通过启停虚拟机网卡来实现灾备切换，所以本文结合了笔者的经验和 pyVmomi 官方案例。

<!-- more -->

VMware vSphere 架构图：

{% asset_img vsphere.png %}

> ### 正文

### 环境

笔者基于 Python3.6.7 与 pyVmomi6.5.0

```shell
pip install pyVmomi==6.5.0
```

VMware vSphere 版本：6.5

### 连接 vSphere

我这里定义了一个基础的类，使用vCenter的地址、账号（默认管理员为administrator@vsphere.local）及密码实例化后，可以使用连接、断开连接、根据vSphere中唯一标识获取对应实例，及监控任务结果的方法。

```python
# -*- coding: UTF-8 -*-
from pyVim.connect import SmartConnect
import ssl
from pyVim import connect
from ssl import SSLEOFError
import sys

class vSphereBase(object):
    """
    与vCenter交互的基类
    """
    def __init__(self, user, pwd, host):
        self.host = host
        self.user = user
        self.pwd = pwd
        
	def _connect_vc_exception(self):
        """ 支持SSL连接和非SSL连接 """
        try:
            context = ssl.SSLContext(ssl.PROTOCOL_TLSv1)
            context.verify_mode = ssl.CERT_NONE
            si = SmartConnect(host=self.host, user=self.user, pwd=self.pwd, port=443, sslContext=context)
            return si, "connect VC success with SSL"
        except SSLEOFError:
            context = ssl._create_unverified_context()
            context.verify_mode = ssl.CERT_NONE
            si = SmartConnect(host=self.host, user=self.user, pwd=self.pwd, port=443, sslContext=context)
            return si, "connect VC success without SSL"
        except Exception as e:
            return None, e
        
    def _deconnect_vc(self, si):
        connect.Disconnect(si)

    def _get_content(self):
        si = self._connect_vc()
        content = si.RetrieveContent()
        return content

    def _get_obj_bymoId(self, content, vimtype, moId):
        obj = None
        container = content.viewManager.CreateContainerView(content.rootFolder, vimtype, True)
        for c in container.view:
            if moId:
                if c._moId == moId:
                    obj = c
                    break
            else:
                obj = None
                break
        return obj

    def get_obj(self, content, vimtype, name):
        """
        Return an object by name, if name is None the
        first found object is returned
        """
        obj = None
        container = content.viewManager.CreateContainerView(
            content.rootFolder, vimtype, True)
        for c in container.view:
            if name:
                if c.name == name:
                    obj = c
                    break
            else:
                obj = c
                break

        return obj

    def _wait_for_task(self, task):
        """ wait for a vCenter task to finish """
        task_done = False
        while not task_done:
            if task.info.state == 'success':
                return {"result": True, "data": task.info.result}
            if task.info.state == 'error':
                return {"result": False, "data": task.info.error.msg}
```



### Example：虚拟机网卡启停

以下定义了一个虚拟机的类，继承 vSphere 的基类，在切换虚拟机网络适配器状态的方法中，传入虚拟机的名称、网卡的编号，以及该网卡需要做connect还是disconnect变更，就可以实现该需求。

```python
# -*- coding: UTF-8 -*-
import atexit
from pyVmomi import vim
import logging
from pyVim.task import WaitForTask

logger = logging.getLogger('app')

class VirtualMachine(vSphereBase):
        def change_nic_state(self, vmname, unitnumber, state):
        # 更改虚拟机网络适配器状态
        si, message = self._connect_vc_exception_message()
        if si is None:
            logger.error(str(message))
            return {"result": False, "message": str(message)}
        logger.info(str(message))
        # disconnect vc
        atexit.register(Disconnect, si)

        content = si.RetrieveContent()
        logger.info('Searching for VM {}'.format(vmname))
        vm_obj = self.get_obj(content, [vim.VirtualMachine], vmname)

        if vm_obj:
            result = self.update_virtual_nic_state(vm_obj, unitnumber, state)
            logger.info('VM NIC {} successfully' \
                        ' state changed to {}'.format(unitnumber, state))
            return {"result": result}
        else:
            logger.error('VM not found')
            return {"result": False, "message": "VM not found"}

    def update_virtual_nic_state(self, vm_obj, nic_number, new_nic_state):
        """
        :param vm_obj: Virtual Machine Object
        :param nic_number: Network Interface Controller Number
        :param new_nic_state: Either Connect, Disconnect or Delete
        :return: True if success
        """
        nic_prefix_label = 'Network adapter '
        nic_label = nic_prefix_label + str(nic_number)
        virtual_nic_device = None
        for dev in vm_obj.config.hardware.device:
            if isinstance(dev, vim.vm.device.VirtualEthernetCard) \
                    and dev.deviceInfo.label == nic_label:
                virtual_nic_device = dev
        if not virtual_nic_device:
            raise RuntimeError('Virtual {} could not be found.'.format(nic_label))

        virtual_nic_spec = vim.vm.device.VirtualDeviceSpec()
        virtual_nic_spec.operation = \
            vim.vm.device.VirtualDeviceSpec.Operation.remove \
                if new_nic_state == 'delete' \
                else vim.vm.device.VirtualDeviceSpec.Operation.edit
        virtual_nic_spec.device = virtual_nic_device
        virtual_nic_spec.device.key = virtual_nic_device.key
        virtual_nic_spec.device.macAddress = virtual_nic_device.macAddress
        virtual_nic_spec.device.backing = virtual_nic_device.backing
        virtual_nic_spec.device.wakeOnLanEnabled = \
            virtual_nic_device.wakeOnLanEnabled
        connectable = vim.vm.device.VirtualDevice.ConnectInfo()
        if new_nic_state == 'connect':
            connectable.connected = True
            connectable.startConnected = True
        elif new_nic_state == 'disconnect':
            connectable.connected = False
            connectable.startConnected = False
        else:
            connectable = virtual_nic_device.connectable
        virtual_nic_spec.device.connectable = connectable
        dev_changes = []
        dev_changes.append(virtual_nic_spec)
        spec = vim.vm.ConfigSpec()
        spec.deviceChange = dev_changes
        task = vm_obj.ReconfigVM_Task(spec=spec)
        WaitForTask(task)
        return True
```



### 执行

在上文定义了实例化 vSphere 和切换网卡的方法后，我们使用以下代码进行调用，由于对比简单，这里不进行赘述。

```python
if __name__ == "__main__":
    account = "1.1.1.1"
    password = "administrator@vsphere.local"
    vc_host = "xxxxxx"
    # 连接vSphere，生成vSphere的实例
    vm = vSphereBase(account, password, vc_host)
    # 对该vSphere中的虚拟机进行网卡变更，返回结果
    vm_name = "device-192.168.1.1"
    unit_number = 1
    vm_state = "disconnect"
    result = vm.change_nic_state(vm_name, unit_number, vm_state)

```



### 总结

由于VMware vSphere中设备众多，且有层级嵌套的关联关系，官方只提供了所有的接口文档和少量案例，更多需求额实现需要自己去开发，就如上文中的描述，首先连接vSphere（vCenter），然后针对需要的概念模型进行数据采集或者操作变更。



******
***官方文档：***
[Github pyvmomi 官方地址](https://github.com/vmware/pyvmomi/)
[Github pyvmomi samples 官方实例](https://github.com/vmware/pyvmomi-community-samples/)