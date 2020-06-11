# RADOSGW手册

## RADOSGW – RADOS REST网关

### 概要

**Radosgw**

### 描述

**radosgw**是RADOS对象存储（Ceph分布式存储系统的一部分）的HTTP REST网关。它使用libfcgi作为FastCGI模块实现，并且可以与任何具有FastCGI功能的Web服务器一起使用。

### 选项

`-c ceph.conf, --conf=ceph.conf`

使用`ceph.conf`配置文件而不是默认文件 `/etc/ceph/ceph.conf`来确定启动期间的监视器地址。`-m monaddress[:port]`

连接到指定的监视器（而不是浏览`ceph.conf`）。`-i ID, --id ID`

设置radosgw名称的ID部分`-n TYPE.ID, --name TYPE.ID`

设置网关的rados用户名（例如client.radosgw.gateway）`--cluster NAME`

设置集群名称（默认值：ceph）`-d`

在前台运行，登录到stderr`-f`

在前台运行，登录到通常的位置`--rgw-socket-path=path`

指定Unix域套接字路径。`--rgw-region=region`

radosgw运行的区域`--rgw-zone=zone`

radosgw运行的区域

### 配置

较早的RADOS网关必须使用`Apache`和进行配置`mod_fastcgi`。现在，使用`mod_proxy_fcgi`module代替`mod_fastcgi`。 `mod_proxy_fcgi`与传统的FastCGI模块的工作方式不同。该模块需要`mod_proxy`提供对FastCGI协议的支持的服务。所以，为了能够处理FastCGI协议，双方`mod_proxy` 并`mod_proxy_fcgi`有存在于服务器。不同于`mod_fastcgi`， `mod_proxy_fcgi`无法启动申请程序。一些平台具有 `fcgistarter`该目的。但是，在使用中的FastCGI应用程序框架中可以使用应用程序的外部启动或过程管理。

`Apache`可以通过`mod_proxy_fcgi`与localhost tcp或通过Unix域套接字一起使用的方式进行配置。`mod_proxy_fcgi`不支持unix域套接字（例如Apache 2.2和Apache 2.4早期版本中的套接字）的端口需要进行配置，以与localhost tcp一起使用。诸如Apache 2.4.9或更高版本的Apache的更高版本支持unix域套接字，因此它们允许使用unix域套接字而不是localhost tcp进行配置。

以下步骤显示了Ceph的配置文件（即） `/etc/ceph/ceph.conf`和网关配置文件（即， `/etc/httpd/conf.d/rgw.conf`基于RPM的发行版或 `/etc/apache2/conf-available/rgw.conf`基于Debian的发行版）与localhost tcp以及通过unix域套接字的配置：

1. 对于使用localhost TCP且不支持Unix Domain Socket的Apache 2.2和Apache 2.4的早期版本的发行版，请将以下内容附加到`/etc/ceph/ceph.conf`：

   ```text
   [client.radosgw.gateway]
   host = {hostname}
   keyring = /etc/ceph/ceph.client.radosgw.keyring
   rgw socket path = ""
   log file = /var/log/ceph/client.radosgw.gateway.log
   rgw frontends = fastcgi socket_port=9000 socket_host=0.0.0.0
   rgw print continue = false
   ```

2. 在网关配置文件中添加以下内容：

   对于Debian / Ubuntu，请添加`/etc/apache2/conf-available/rgw.conf`：

   ```text
   <VirtualHost *:80>
   ServerName localhost
   DocumentRoot /var/www/html

   ErrorLog /var/log/apache2/rgw_error.log
   CustomLog /var/log/apache2/rgw_access.log combined

   # LogLevel debug

   RewriteEngine On

   RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization},L]

   SetEnv proxy-nokeepalive 1

   ProxyPass / fcgi://localhost:9000/

   </VirtualHost>
   ```

   对于CentOS / RHEL，添加`/etc/httpd/conf.d/rgw.conf`：

   ```text
   <VirtualHost *:80>
   ServerName localhost
   DocumentRoot /var/www/html

   ErrorLog /var/log/httpd/rgw_error.log
   CustomLog /var/log/httpd/rgw_access.log combined

   # LogLevel debug

   RewriteEngine On

   RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization},L]

   SetEnv proxy-nokeepalive 1

   ProxyPass / fcgi://localhost:9000/

   </VirtualHost>
   ```

3. 对于支持Unix Domain Socket的Apache 2.4.9或更高版本的发行版，请将以下配置附加到`/etc/ceph/ceph.conf`：

   ```text
   [client.radosgw.gateway]
   host = {hostname}
   keyring = /etc/ceph/ceph.client.radosgw.keyring
   rgw socket path = /var/run/ceph/ceph.radosgw.gateway.fastcgi.sock
   log file = /var/log/ceph/client.radosgw.gateway.log
   rgw print continue = false
   ```

4. 在网关配置文件中添加以下内容：

   对于CentOS / RHEL，添加`/etc/httpd/conf.d/rgw.conf`：

   ```text
   <VirtualHost *:80>
   ServerName localhost
   DocumentRoot /var/www/html

   ErrorLog /var/log/httpd/rgw_error.log
   CustomLog /var/log/httpd/rgw_access.log combined

   # LogLevel debug

   RewriteEngine On

   RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization},L]

   SetEnv proxy-nokeepalive 1

   ProxyPass / unix:///var/run/ceph/ceph.radosgw.gateway.fastcgi.sock|fcgi://localhost:9000/

   </VirtualHost>
   ```

   请注意，它不支持Unix Domain Socket，因此必须使用localhost tcp进行配置。Unix域套接字支持在更高版本中可用。`Apache 2.4.7Apache 2.4.9`

5. 生成radosgw的密钥，以用于集群的身份验证。

   ```text
   ceph-authtool -C -n client.radosgw.gateway --gen-key /etc/ceph/keyring.radosgw.gateway
   ceph-authtool -n client.radosgw.gateway --cap mon 'allow rw' --cap osd 'allow rwx' /etc/ceph/keyring.radosgw.gateway
   ```

6. 将密钥添加到身份验证条目。

   ```text
   ceph auth add client.radosgw.gateway --in-file=keyring.radosgw.gateway
   ```

7. 启动Apache和radosgw。

   Debian / Ubuntu：

   ```text
   sudo /etc/init.d/apache2 start
   sudo /etc/init.d/radosgw start
   ```

   CentOS / RHEL：

   ```text
   sudo apachectl start
   sudo /etc/init.d/ceph-radosgw start
   ```

### 使用记录

**radosgw**维护异步使用日志。它累积有关用户操作的统计信息并定期刷新它。可以通过**radosgw-admin**访问和管理日志。

正在记录的信息包含总数据传输，总操作和总成功操作。除非在服务上完成了操作（例如，在列出存储桶时），否则数据将以每小时的分辨率在存储桶所有者下进行会计处理，在这种情况下，数据将由运行用户进行会计处理。

以下是示例配置：

```text
[client.radosgw.gateway]
    rgw enable usage log = true
    rgw usage log tick interval = 30
    rgw usage log flush threshold = 1024
    rgw usage max shards = 32
    rgw usage max user shards = 1
```

分片的总数确定保存使用日志信息的对象总数。每个用户的分片数量指定了一个对象中有多少个对象保存使用情况信息。滴答间隔配置日志刷新之间的秒数，并且刷新阈值指定在诉诸同步刷新之前可以保留多少个条目。

### 可用性

**radosgw**是Ceph的一部分，Ceph是可大规模扩展的开源分布式存储系统。有关更多信息，请参阅[http://ceph.com/docs](http://ceph.com/docs)上的Ceph文档。



