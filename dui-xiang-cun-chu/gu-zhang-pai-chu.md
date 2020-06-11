# 故障排除

## 故障排除

### 网关无法启动

如果您无法启动网关（即不存在`pid`），请检查是否存在`.asok`来自其他用户的文件。如果`.asok`存在另一个用户`pid`的`.asok`文件并且没有运行，请删除该文件，然后尝试重新启动该过程。当您以`root`用户身份启动进程并且启动脚本试图以a `www-data`或`apache`user 身份启动进程 并且现有`.asok`进程阻止脚本启动守护程序时，可能会发生这种情况。

radosgw初始化脚本（/etc/init.d/radosgw）也有一个冗长的参数，可以为可能的问题提供一些见解：

```text
/etc/init.d/radosgw start -v
```

要么

```text
/etc/init.d radosgw start --verbose
```

### HTTP请求错误

检查Web服务器本身的访问和错误日​​志可能是识别正在发生的事情的第一步。如果出现500错误，通常表明与`radosgw`守护程序通信出现问题 。确保守护程序正在运行，已配置其套接字路径，并且Web服务器正在正确的位置寻找它。

### 失事`RADOSGW`过程

如果该`radosgw`过程终止，通常会从Web服务器（Apache，nginx等）看到500错误。在这种情况下，只需重新启动radosgw即可恢复服务。

要诊断崩溃原因，请检查登录`/var/log/ceph` 和/或核心文件（如果已生成）。

### 阻止`RADOSGW`请求

如果某些（或全部）radosgw请求似乎被阻止，则可以`radosgw`通过后台驻留程序的管理套接字来了解该后台驻留程序的内部状态。默认情况下，将有一个套接字配置为驻留在中`/var/run/ceph`，并且可以使用以下命令查询守护进程：

```text
ceph daemon /var/run/ceph/client.rgw help

help                list available commands
objecter_requests   show in-progress osd requests
perfcounters_dump   dump perfcounters value
perfcounters_schema dump perfcounters schema
version             get protocol version
```

特别感兴趣：

```text
ceph daemon /var/run/ceph/client.rgw objecter_requests
...
```

将使用RADOS群集转储有关当前正在进行的请求的信息。这样一来，您就可以确定无响应的OSD是否阻止了任何请求。例如，可能会看到：

```text
{ "ops": [
      { "tid": 1858,
        "pg": "2.d2041a48",
        "osd": 1,
        "last_sent": "2012-03-08 14:56:37.949872",
        "attempts": 1,
        "object_id": "fatty_25647_object1857",
        "object_locator": "@2",
        "snapid": "head",
        "snap_context": "0=[]",
        "mtime": "2012-03-08 14:56:37.949813",
        "osd_ops": [
              "write 0~4096"]},
      { "tid": 1873,
        "pg": "2.695e9f8e",
        "osd": 1,
        "last_sent": "2012-03-08 14:56:37.970615",
        "attempts": 1,
        "object_id": "fatty_25647_object1872",
        "object_locator": "@2",
        "snapid": "head",
        "snap_context": "0=[]",
        "mtime": "2012-03-08 14:56:37.970555",
        "osd_ops": [
              "write 0~4096"]}],
"linger_ops": [],
"pool_ops": [],
"pool_stat_ops": [],
"statfs_ops": []}
```

在此转储中，两个请求正在进行中。该`last_sent`字段是发送RADOS请求的时间。如果这是前一阵子，则表明OSD没有响应。例如，对于请求1858，可以使用以下命令检查OSD状态：

```text
ceph pg map 2.d2041a48

osdmap e9 pg 2.d2041a48 (2.0) -> up [1,0] acting [1,0]
```

这告诉我们查看`osd.1`，此PG的主要副本：

```text
ceph daemon osd.1 ops
{ "num_ops": 651,
 "ops": [
       { "description": "osd_op(client.4124.0:1858 fatty_25647_object1857 [write 0~4096] 2.d2041a48)",
         "received_at": "1331247573.344650",
         "age": "25.606449",
         "flag_point": "waiting for sub ops",
         "client_info": { "client": "client.4124",
             "tid": 1858}},
...
```

`flag_point`在此情况下，该字段指示OSD当前正在等待副本响应`osd.0`。

### JAVA S3 API故障排除

#### 对等未认证

您可能会收到如下错误：

```text
[java] INFO: Unable to execute HTTP request: peer not authenticated
```

用于S3的Java SDK要求来自公认的证书颁发机构的有效证书，因为默认情况下它使用HTTPS。如果您仅测试Ceph对象存储服务，则可以通过以下几种方法解决此问题：

1. 在IP地址或主机名前添加`http://`。例如，更改此：

   ```text
   conn.setEndpoint("myserver");
   ```

   至：

   ```text
   conn.setEndpoint("http://myserver")
   ```

2. 设置凭据后，添加客户端配置并将协议设置为`Protocol.HTTP`。

   ```text
   AWSCredentials credentials = new BasicAWSCredentials(accessKey, secretKey);

   ClientConfiguration clientConfig = new ClientConfiguration();
   clientConfig.setProtocol(Protocol.HTTP);

   AmazonS3 conn = new AmazonS3Client(credentials, clientConfig);
   ```

#### 405 METHODNOTALLOWED 

如果收到405错误，请检查是否正确设置了S3子域。您需要在DNS记录中具有通配符设置，才能使子域功能正常工作。

另外，请检查以确保默认站点已禁用。

```text
[java] Exception in thread "main" Status Code: 405, AWS Service: Amazon S3, AWS Request ID: null, AWS Error Code: MethodNotAllowed, AWS Error Message: null, S3 Extended Request ID: null
```

### DEFAULT.RGW.META池中的许多对象

在_宝石_之前创建的群集具有默认情况下使用`default.rgw.meta`池启用的元数据存档功能。该存档保留了用户和存储区元数据的所有旧版本，从而导致`default.rgw.meta`池中有大量对象。

#### 禁用元数据堆

以后要禁用此功能的用户应将`metadata_heap`字段设置为空字符串`""`：

```text
$ radosgw-admin zone get --rgw-zone=default > zone.json
[edit zone.json, setting "metadata_heap": ""]
$ radosgw-admin zone set --rgw-zone=default --infile=zone.json
$ radosgw-admin period update --commit
```

这将阻止将新的元数据写入`default.rgw.meta`池中，但不会删除任何现有的对象或池。

#### 清理元数据堆池

在_珠宝_之前创建的群集通常`default.rgw.meta`仅用于元数据存档功能。

然而，从_发光_起，radosgw使用[池名称空间](https://docs.ceph.com/docs/nautilus/radosgw/pools/#radosgw-pool-namespaces)内`default.rgw.meta`用于完全不同的目的，即，存储`user_keys`和其它关键的元数据。

用户应在执行任何清除步骤之前检查区域配置：

```text
$ radosgw-admin zone get --rgw-zone=default | grep default.rgw.meta
[should not match any strings]
```

确认不将池用于任何目的后，用户可以安全地删除`default.rgw.meta`池中的所有对象，或者选择删除整个池本身。

