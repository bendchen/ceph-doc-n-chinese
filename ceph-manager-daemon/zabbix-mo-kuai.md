# ZABBIX模块

## ZABBIX模块

Zabbix模块主动将信息发送到Zabbix服务器，例如：

* Ceph身份
* I / O操作
* I / O带宽
* OSD状态
* 存储利用率

### 要求

该模块要求在运行_ceph_ -mgr的_所有_计算机上_都_存在_zabbix\_sender_可执行文件。可以使用包管理器将其安装在大多数发行版上。

#### 依赖

可以使用apt或dnf在Ubuntu或CentOS下完成安装zabbix\_sender。

在Ubuntu Xenial上：

```text
apt install zabbix-agent
```

在Fedora上：

```text
dnf install zabbix-sender
```

### 启用

您可以通过以下方式启用_zabbix_模块：

```text
ceph mgr module enable zabbix
```

### 配置

两个配置密钥对于模块的工作至关重要：

* zabbix\_host
* 标识符（可选）

参数_zabbix\_host_控制_zabbix\_sender_将项目发送到的Zabbix服务器的主机名 。如果您的安装需要，这可以是IP地址。

的_标识符_参数控制发送项的zabbix当标识符/主机名作为源使用。这应该与您的Zabbix服务器中的_主机_名匹配。

如果未配置_标识符_参数，则在向Zabbix发送数据时将使用集群的ceph- &lt;fsid&gt;。

例如，这将是_ceph-c4d32a99-9e80-490f-bd3a-1d22d8a7d354_

可以配置的其他配置键及其默认值：

* zabbix\_port：10051
* zabbix\_sender：/ usr / bin / zabbix\_sender
* 间隔：60

#### 配置项

可以在具有适当cephx凭据的任何计算机上设置配置密钥，这些通常是存在_client.admin_密钥的Monitors 。

```text
ceph zabbix config-set <key> <value>
```

例如：

```text
ceph zabbix config-set zabbix_host zabbix.localdomain
ceph zabbix config-set identifier ceph.eu-ams02.local
```

该模块的当前配置也可以显示：

```text
ceph zabbix config-show
```

#### 模板

一个[模板](https://raw.githubusercontent.com/ceph/ceph/9c54334b615362e0a60442c2f41849ed630598ab/src/pybind/mgr/zabbix/zabbix_template.xml)。可以在模块的源目录中找到要在Zabbix服务器上使用的（XML）。

该模板包含所有项目和一些触发器。之后，您可以自定义触发器以适合您的需求。

#### 多个ZABBIX服务器

可以指示zabbix模块将数据发送到多个Zabbix服务器。

可以使用多个主机名（用逗号分隔）设置参数_zabbix\_host_。域名（或IP地址）后可以跟冒号和端口号。如果端口号不存在，模块将使用_zabbix\_port中_定义的端口号。

例如：

```text
ceph zabbix config-set zabbix_host "zabbix1,zabbix2:2222,zabbix3:3333"
```

### 手动发送数据

如果需要，可以要求模块立即发送数据，而不是等待间隔。

可以使用以下命令完成此操作：

```text
ceph zabbix send
```

现在，该模块会将其最新数据发送到Zabbix服务器。

### 调试

如果要调试Zabbix模块，请增加ceph-mgr的日志记录级别并检查日志。

```text
[mgr]
    debug mgr = 20
```

将日志记录设置为为管理器调试时，模块将打印各种带有_mgr \[zabbix\]_前缀的日志记录行，以便于过滤。

