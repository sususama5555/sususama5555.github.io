---
title: Zabbix安装与部署
date: 2020-12-01 00:05:19
tags: 
- Zabbix
- 监控
- 告警
- 阿里云
- 安装部署
categories: 
- 监控
- Zabbix
---

> # 前言

最近工作中和`Zabbix`打了不少交道，大概分为两个用途。

一种是对接它的API接口，例如创建账号、媒介、触发器、动作等，这种比较简单，照着API文档来就行了，这里不多赘述。

另一种就是，使用python脚本创建一套Zabbix告警推送的流程，将超过阈值的Zabbix告警按照指定的流程和动作推送到告警中心，而告警中心使用Zabbix作为其中一个告警源，不断拉取并且分发告警。这应该属于监控、告警等最常见的自动化运维的场景了，我对此也比较感兴趣，所有想从Zabbix较为基础的单机版安装及部署学习，也就有了这篇文章。

事先已在本地开发环境的虚拟机中安装与部署成功，现在期望将Zabbix部署到阿里云的机器上。

<!-- more -->

> # 安装

这里我们按照官方文档的指引进行安装

官方地址：https://www.zabbix.com/download

由于本人的阿里云服务器的预先环境为CentOS 7 + Mysql + Nginx，所以这里不再折腾，直接使用这一套最常用的配置。

由于Zabbix 5.2是最新版本，安装指引和资料也比较少，处于稳定考虑，最终选择Zaabix 5.0 LTS版本。

PS:工作接触的5.0和5.2都有。

## Install and configure Zabbix server for your platform

### a. Install Zabbix repository（安装Zabbix yum 源）

官方文档：[Documentation](https://www.zabbix.com/documentation/5.0/manual/installation/install_from_packages)

#### 官方yum源

以下是官方推荐的zabbix yum源，不过在国内使用比较麻烦，下载速度慢且经常连接超时。所以推荐使用阿里云的zabbix yum 源

```
# rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
# yum clean all
```

#### 阿里云yum源

下载地址：https://mirrors.aliyun.com/zabbix/

```
# rpm -Uvh https://mirrors.aliyun.com/zabbix/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
# yum clean all
```

打开/etc/yum.repos.d/zabbix.repo，把所有的“https://repo.zabbix.com/zabbix/” 改成 “https://mirrors.aliyun.com/zabbix/” （除了zabbix-non-supported，其他的分支的URL都有两个zabbix）

### b. Install Zabbix server and agent（安装Zabbix server 和agent）

换了yum源应该安装很快，使用 -y 跳过选择

```
# yum install -y zabbix-server-mysql zabbix-agent
```

### c. Install Zabbix frontend（安装Zabbix 前端）

[Documentation](https://www.zabbix.com/documentation/5.0/manual/installation/frontend/frontend_on_rhel7)

Zabbix 使用php写的，所以有单独的安装过程：

Enable Red Hat Software Collections.

```
# yum install centos-release-scl
```

Edit file /etc/yum.repos.d/zabbix.repo and enable zabbix-frontend repository.

编辑该配置文件 /etc/yum.repos.d/zabbix.repo，使得enabled=1。

```
[zabbix-frontend]...enabled=1...
```

Install Zabbix frontend packages.

下载安装Zabbix 前端依赖。

```
# yum install zabbix-web-mysql-scl zabbix-nginx-conf-scl
```

### d. Create initial database（创建初始数据库）

[Documentation](https://www.zabbix.com/documentation/5.0/manual/appendix/install/db_scripts)

Run the following on your database host.

在数据库主机上运行以下代码，目的是在本地mysql数据库中创建zabbix账号和同名数据库，并且赋予相应权限。

```mysql
# mysql -uroot -p
password
mysql> create database zabbix character set utf8 collate utf8_bin;
mysql> create user zabbix@localhost identified by 'password';
mysql> grant all privileges on zabbix.* to zabbix@localhost;
mysql> quit;
```

On Zabbix server host import initial schema and data. You will be prompted to enter your newly created password.

导入初始架构和数据，系统将提示您输入新创建的密码。

```shell
zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix
```

TODO 其中遇到了问题，愿意是以上语句执行时，发生以下报错：

```shell
ERROR 1071 (42000) at line 90: Specified key was too long; max key length is 767 bytes
```

{% asset_img character-error.png %}

应该是mysql字符集的报错，所以去查了Stack Overflow后，可以在mysql命令行中，使用`show variables like 'character%'`查询默认字符集，结果发现许多编码为utf-8。

```mysql
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

MySQL的varchar主键只支持不超过767个字节，需要将mysql的字符编码设置为utf8mb4

使用`vim /etc/my.cnf `编辑mysql配置文件（不同系统、安装方式和mysql版本会造成差异）

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

现在是说索引列大小超出，这就与第一次的报错相反。这就很恶心了，未完待续。

### e. Configure the database for Zabbix server（为Zabbix server配置数据库）

Edit file /etc/zabbix/zabbix_server.conf

编辑配置文件 /etc/zabbix/zabbix_server.conf，将刚才设置的密码填入该配置文件。

```
DBPassword=password
```

### f. Configure PHP for Zabbix frontend（为Zabbix前端配置PHP）

Edit file /etc/opt/rh/rh-nginx116/nginx/conf.d/zabbix.conf, uncomment and set 'listen' and 'server_name' directives.

编辑配置文件 /etc/opt/rh/rh-nginx116/nginx/conf.d/zabbix.conf，取消其中listen和server_name的注释。此处就是我们非常熟悉的Nginx，配置反向代理监听的端口和域名。

```
# listen 80;
# server_name example.com;
```

Edit file /etc/opt/rh/rh-php72/php-fpm.d/zabbix.conf, add nginx to listen.acl_users directive.

编辑配置文件 /etc/opt/rh/rh-php72/php-fpm.d/zabbix.conf, 将nginx添加到listen.acl_users配置项中。

```
listen.acl_users = apache,nginx
```

Then uncomment and set the right timezone for you.

最后取消最后一行的注释（注意注释为分号），我们也可以将市区设置为Asia/Shanghai

```
; php_value[date.timezone] = Asia/Shanghai
```

### g. Start Zabbix server and agent processes（启动Zabbix server和agent进程）

Start Zabbix server and agent processes and make it start at system boot.

启动Zabbix server和agent进程，并为它们设置开机自启：

```
# systemctl restart zabbix-server zabbix-agent rh-nginx116-nginx rh-php72-php-fpm
# systemctl enable zabbix-server zabbix-agent rh-nginx116-nginx rh-php72-php-fpm
```

##### h. Configure Zabbix frontend

Connect to your newly installed Zabbix frontend: http://server_ip_or_name
Follow steps described in Zabbix documentation: [Installing frontend](https://www.zabbix.com/documentation/5.0/manual/installation/install#installing_frontend)