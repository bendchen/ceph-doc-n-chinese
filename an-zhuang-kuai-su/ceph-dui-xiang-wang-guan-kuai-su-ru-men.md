# CEPH对象网关快速入门

## CEPH对象网关快速入门

从firefly（v0.80）开始，Ceph Storage大大简化了Ceph对象网关的安装和配置。网关守护程序嵌入了Civetweb，因此您不必安装Web服务器或配置FastCGI。此外， `ceph-deploy`可以安装网关软件包，生成密钥，配置数据目录并为您创建网关实例。

小费 

Civetweb `7480`默认使用端口。您必须在Ceph配置文件中打开port `7480`或将其设置为首选端口（例如port `80`）。

要启动Ceph对象网关，请执行以下步骤：

### 安装CEPH对象网关

1. 在上执行预安装步骤`client-node`。如果打算使用Civetweb的默认端口`7480`，则必须使用`firewall-cmd`或来打开它 `iptables`。有关更多信息，请参见[飞行前检查清单](https://docs.ceph.com/docs/nautilus/start/quick-start-preflight)。
2. 从管理服务器的工作目录中，在`client-node`节点上安装Ceph对象网关软件包。例如：

   ```text
   ceph-deploy install --rgw <client-node> [<client-node> ...]
   ```

### 创建CEPH对象网关实例

在管理服务器的工作目录中，在上创建Ceph对象网关的实例`client-node`。例如：

```text
ceph-deploy rgw create <client-node>
```

网关运行后，您应该可以在port上访问它`7480`。（例如`http://client-node:7480`）。

### 配置CEPH对象网关实例

1. 要更改默认端口（例如，更改为port `80`），请修改您的Ceph配置文件。添加标题为的部分`[client.rgw.<client-node>]`，替换`<client-node>`为您的Ceph客户端节点的短节点名称（即）。例如，如果您的节点名称为 ，则在该部分之后添加如下部分：`hostname -sclient-node[global]`

   ```text
   [client.rgw.client-node]
   rgw_frontends = "civetweb port=80"
   ```

   注意 

   请确保你离开之间没有空格`port=<port-number>` 的`rgw_frontends`键/值对。

   重要 

   如果打算使用端口80，请确保Apache服务器未运行，否则它将与Civetweb冲突。我们建议在这种情况下删除Apache。

2. 要使新的端口设置生效，请重新启动Ceph对象网关。在Red Hat Enterprise Linux 7和Fedora上，运行以下命令：

   ```text
   sudo systemctl restart ceph-radosgw.service
   ```

   在Red Hat Enterprise Linux 6和Ubuntu上，运行以下命令：

   ```text
   sudo service radosgw restart id=rgw.<short-hostname>
   ```

3. 最后，检查以确保所选的端口在节点的防火墙上是打开的（例如port `80`）。如果未打开，请添加端口并重新加载防火墙配置。例如：

   ```text
   sudo firewall-cmd --list-all
   sudo firewall-cmd --zone=public --add-port 80/tcp --permanent
   sudo firewall-cmd --reload
   ```

   有关使用或配置防火墙的更多信息， 请参见[预检检查表](https://docs.ceph.com/docs/nautilus/start/quick-start-preflight)。`firewall-cmdiptables`

   您应该能够发出未经身份验证的请求，并收到响应。例如，没有这样的参数的请求：

   ```text
   http://<client-node>:80
   ```

   应导致这样的响应：

   ```text
   <?xml version="1.0" encoding="UTF-8"?>
   <ListAllMyBucketsResult xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
     <Owner>
       <ID>anonymous</ID>
       <DisplayName></DisplayName>
     </Owner>
       <Buckets>
     </Buckets>
   </ListAllMyBucketsResult>
   ```

有关其他管理和API详细信息，请参阅《配置Ceph对象网关》指南。

