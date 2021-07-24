---
title: Zabbix安装与部署
date: 2020-12-01 00:05:19
tags: 
- Zabbix
- Linux
- 监控
- 告警
- 开源
- 阿里云
- 安装部署
categories: 
- 监控
- Zabbix
---

> # 前言

最近工作中和`Zabbix`打了不少交道，大概分为两个用途。

一种是对接它的API接口，例如创建账号、媒介、触发器、动作等，这种比较简单，照着API文档来就行了，这里不多赘述。

另一种就是，使用Python脚本创建一套Zabbix告警推送的流程，将超过阈值的Zabbix告警按照指定的流程和动作推送到告警中心，而告警中心使用Zabbix作为其中一个告警源，不断拉取并且分发告警。这应该属于监控、告警等最常见的自动化运维的场景了，我对此也比较感兴趣，所有想从Zabbix较为基础的单机版安装及部署学习，也就有了这篇文章。

事先已在本地开发环境的虚拟机中安装与部署成功，现在期望将Zabbix部署到阿里云的机器上。

***注意：安装部署时，由于环境和版本的问题也踩了不少坑，希望能给读者一些启示。***

***作者Zabbix网址***：https://zabbix.notspr.com/

<!-- more -->

> # 安装

这里我们按照官方文档的指引进行安装

官方地址：https://www.zabbix.com/download

由于本人的阿里云服务器的预先环境为`CentOS 7` + `MySQL`+ `Nginx`，所以这里不再折腾，直接使用这一套最常用的配置。

由于Zabbix 5.2是最新版本，安装指引和资料也比较少，处于稳定考虑，最终选择`Zaabix 5.0 LTS`版本。

PS:工作接触的5.0和5.2都有。

## 安装Zabbix yum 源

官方文档：[Documentation](https://www.zabbix.com/documentation/5.0/manual/installation/install_from_packages)

### 官方yum源

以下是官方推荐的Zabbix yum源，不过在国内使用比较麻烦，下载速度慢且经常连接超时。所以推荐使用阿里云的Zabbix yum 源

```shell
rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
yum clean all
```

### 阿里云yum源

下载地址：https://mirrors.aliyun.com/zabbix/

```shell
rpm -Uvh https://mirrors.aliyun.com/zabbix/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
yum clean all
```

打开/etc/yum.repos.d/zabbix.repo，把所有的“https://repo.zabbix.com/zabbix/” 改成 “https://mirrors.aliyun.com/zabbix/” （除了zabbix-non-supported，其他的分支的URL都有两个zabbix）

## 安装Zabbix server 和agent

换了yum源应该安装很快，使用 -y 跳过选择

```shell
yum install -y zabbix-server-mysql zabbix-agent
```

## 安装Zabbix 前端

[Documentation](https://www.zabbix.com/documentation/5.0/manual/installation/frontend/frontend_on_rhel7)

Zabbix 使用`php`写的，所以有单独的安装过程：

```shell
yum install centos-release-scl
```

编辑该配置文件 `/etc/yum.repos.d/zabbix.repo`，使得enabled=1。

```ini
[zabbix-frontend]...enabled=1...
```

下载安装Zabbix 前端依赖。

```bash
yum install zabbix-web-mysql-scl zabbix-nginx-conf-scl
```

## 创建初始数据库

[Documentation](https://www.zabbix.com/documentation/5.0/manual/appendix/install/db_scripts)

在数据库主机上运行以下代码，目的是在本地MySQL数据库中创建zabbix账号和同名数据库，并且赋予相应权限。

```sql
# mysql -uroot -p
password
mysql> create database zabbix character set utf8 collate utf8_bin;
mysql> create user zabbix@localhost identified by 'password';
mysql> grant all privileges on zabbix.* to zabbix@localhost;
mysql> quit;
```

导入初始架构和数据，系统将提示您输入新创建的密码。

```shell
zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix
```

### &#x1F449; ***ATTENTION ONE***

以下就是踩坑了：

执行上一条命令时，MySQL导入数据与结构不成功，发生以下报错：

```shell
ERROR 1071 (42000) at line 90: Specified key was too long; max key length is 767 bytes
```

{% asset_img character-error.png %}

#### 错误做法

初步判断是MySQL字符集的问题，所以去查了 ***<u>Stack Overflow</u>*** 后，在MySQL命令行中，使用`show variables like 'character%'`查询默认字符集，结果发现许多编码为utf-8。

```sql
# mysql -uroot -p
password
mysql> show variables like 'character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)
```

***<u>原因分析：</u>***

MySQL的varchar主键只支持不超过767个字节，需要将MySQL的字符编码设置为`utf8mb4`

使用`vim /etc/my.cnf `编辑MySQL配置文件（不同系统、安装方式和MySQL版本会造成差异）

在[mysqld]下加入以下配置：

```ini
innodb_file_format=barracuda  
innodb_file_per_table=true  
innodb_large_prefix=true  
character-set-server=utf8mb4  
collation-server=utf8mb4_unicode_ci  
max_allowed_packet=500M  
```

如图是我的配置：

{% asset_img mysql-cfg.png %}

现在重新执行Zabbix创建初始数据库这一步，但是又有新的问题，还是字符集的报错：

{% asset_img utf8mb4.png %}

现在出现索引列大小超出的问题，这就与之前的报错相反，这就很恶心了。

#### 正确做法

到这里才发现不对劲，有点南辕北辙的感觉，为什么我在个人Linux虚拟机上就能数据迁移成功？

这肯定是MySQL的问题，所以对比检查云服务器和虚拟机的版本、配置等信息。

果不其然，正常的虚拟机是MySQL5.7版本的，而有问题的云服务器是MySQL5.6版本的，除此之外，默认编码等配置都一模一样。所以定位到了MySQL5.7和MySQL5.6的差异：

5.7除了比5.6性能更强、功能更丰富，其中5.7的索引长度也增加了，这就是报错的问题所在，所以折腾来折腾去，升级MySQL版本完美解决。

***<u>升级方案：</u>***

##### 首先全库备份mysql 5.6

```shell
mysqldump -uroot -p123456 --all-databases > /root/mysql_all.sql
```

##### yum配置mysql 5.7

可以直接使用`yum`安装，或者源码安装，但是`yum`安装是非常平滑的升级，数据库和配置文件都不用改，强烈推荐。

但是首先需要修改MySQL相关的yum源文件，因为5.6版本会默认屏蔽5.7的源，在以下两个文件中，将5.7设置为`enable=1`，5.6修改为`enable=0`。

```bash
vim /etc/yum.repos.d/mysql-community.repo
vim /etc/yum.repos.d/mysql-community-source.repo
```

```ini
# Note: MySQL 5.7 is currently in development. For use at your own risk.
# Please read with sub pages: https://dev.mysql.com/doc/relnotes/mysql/5.7/en/
[mysql57-community-dmr]
name=MySQL 5.7 Community Server Development Milestone Release
baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/7/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:/etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```

然后`yum clean all`清除`yum`缓存

##### yum安装mysql 5.7

`yum install mysql` 或者 `yum upgrade mysql-server` 应该都可以升级成功&#x1F308;

升级完就可以直接使用啦，重复**<u>*创建初始数据库*</u>** 完成这一步。

## 为Zabbix server配置数据库

编辑配置文件 `/etc/zabbix/zabbix_server.conf`，将刚才设置的密码填入该配置文件。

```ini
DBPassword=password
```

## 为Zabbix前端配置PHP

### 配置Nginx

编辑配置文件 `/etc/opt/rh/rh-nginx116/nginx/conf.d/zabbix.conf`，取消其中`listen`和`server_name`的注释。此处就是我们非常熟悉的`Nginx`，配置反向代理监听的端口和域名。

```ini
listen 80;
server_name example.com;
```

#### &#x1F449; ***ATTENTION TWO*** 

这里配置`rh-nginx116`是给之前没有安装Nginx的机器使用的，由于一台机器只能启动一个Nginx服务（Docker除外）,所以如果下一步执行`systemctl restart rh-nginx116-nginx`报错，那就是两个Nginx冲突了，Nginx服务都监听了80端口，绑定端口失败，具体错误如下：

{% asset_img nginx-error.png %}

所以我们只需要在原有的Nginx上，配置上本次Zabbix的配置就可以了。

#### ***<u>具体方案</u>***：

在`/etc/nginx/conf.d`中创建`zabbix.conf`，然后使得`/etc/nginx/nginx.conf`中包含所有`conf.d`，也就是`include /etc/nginx/conf.d/*.conf`。

将Zabbix安装的 `rh-nginx116`的配置文件`/etc/opt/rh/rh-nginx116/nginx/conf.d/zabbix.conf`的内容拷贝到`zabbix.conf`，就可以对接到已有的Nginx。

### 配置php

编辑配置文件 `/etc/opt/rh/rh-php72/php-fpm.d/zabbix.conf`，将nginx添加到listen.acl_users配置项中。

```ini
listen.acl_users = apache,nginx
```

#### &#x1F449; ***ATTENTION THREE***

如果下一步`rh-php72-php-fpm`服务启动不了，原因是 `/etc/opt/rh/rh-php72/php-fpm.d/zabbix.conf`这个配置文件`listen.acl_users = apache,nginx`。

**<u>*apache,nginx中间不能有逗号*</u>**

### 配置时区

最后取消最后一行的注释（注意注释为分号），我们也可以将市区设置为Asia/Shanghai

```ini
php_value[date.timezone] = Asia/Shanghai
```

## 启动Zabbix server和agent进程

启动Zabbix server和agent进程，并为它们设置开机自启：

```shell
systemctl restart zabbix-server zabbix-agent rh-nginx116-nginx rh-php72-php-fpm
systemctl enable zabbix-server zabbix-agent rh-nginx116-nginx rh-php72-php-fpm
```

如果已经对接到了原有的Nginx，需要忽略`rh-nginx116-nginx`这个服务。

## 配置Zabbix前端

连接到新安装的Zabbix前端： http://server_ip_or_name
根据Zabbix文件里步骤操作： [Installing frontend](https://www.zabbix.com/documentation/5.0/manual/installation/install#installing_frontend)

# 开始使用Zabbix

在浏览器中打开Zabbix前端url，如果是从软件包安装的Zabbix，url如下：

- for Apache: *http://server_ip_or_name/zabbix*
- for Nginx: *http://server_ip_or_name*

## Zabbix欢迎界面&#x1F308;

{% asset_img install_1.png %}

## 检查先决条件&#x1F308;

{% asset_img install_2.png %}

## 配置连接数据库&#x1F308;

{% asset_img install_3.png %}

## 一直点击`NEXT STEP`，完成就会跳转到登陆界面啦&#x1F308;

{% asset_img login.png %}

## 输入默认账号密码`Admin` & `zabbix`就可以登陆啦&#x1F308;

{% asset_img zabbix-home.png %}

> # 总结

Zabbix是一个基于WEB界面的提供分布式系统监视以及网络监视功能的企业级的解决方案，同时也是优秀的开源项目，在监控这个领域也是老大哥了。

绝大多数运维开发都会接触Zabbix，使用它来组成硬件监控、告警推送等解决方案。

以上就是Zabbix安装与配置的内容，也是最基础的单机版方案，虽然官方文档比较全，但是由于环境的版本的问题也踩了不少坑，希望能给读者一些启示。

**<u>TODO LIST</u>**

- Zabbix主机监控
- Zabbix告警推送（SMTP邮件、自定义脚本）

最后附上本人阿里云上的Zabbix网址：

https://zabbix.notspr.com/



******

官方文档：
[官方 - 下载安装Zabbix](https://www.zabbix.com/cn/download?zabbix=5.0&os_distribution=centos&os_version=7&db=mysql&ws=nginx)

[Zabbix Documentation 5.0](https://www.zabbix.com/documentation/5.0/manual/installation/install#installing_frontend)