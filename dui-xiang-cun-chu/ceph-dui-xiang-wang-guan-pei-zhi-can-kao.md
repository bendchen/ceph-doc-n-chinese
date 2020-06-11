# CEPH对象网关配置参考

## CEPH对象网关配置参考

以下设置可能会添加到Ceph配置文件中（即通常为 `ceph.conf`）下的`[client.radosgw.{instance-name}]`部分。这些设置可能包含默认值。如果您未在Ceph配置文件中指定每个设置，则默认值将自动设置。

`[client.radosgw.{instance-name}]` 没有在该命令中指定实例名称的情况下，在该部分下设置的配置变量将不适用于rgw或radosgw-admin命令。因此，可以将要应用于所有RGW实例或所有radosgw-admin命令的变量放入`[global]`或 `[client]`部分中，以避免指定实例名称。

`rgw frontends`描述

配置HTTP前端。可以在多个逗号分隔的列表中提供多个前端的配置。每个前端配置可以包括由空格分隔的选项列表，其中每个选项的形式为“键=值”或“键”。有关支持的选项的更多信息，请参见 [HTTP前端](https://docs.ceph.com/docs/nautilus/radosgw/frontends)。类型

串默认

`beast port=7480`

`rgw data`描述

设置Ceph对象网关的数据文件的位置。类型

串默认

`/var/lib/ceph/radosgw/$cluster-$id`

`rgw enable apis`描述

启用指定的API。

注意 

`s3`对于要参与[多站点](https://docs.ceph.com/docs/nautilus/radosgw/multisite) 配置的任何radosgw实例，都必须启用API 。类型

串默认

`s3, swift, swift_auth, admin` 所有API。

`rgw cache enabled`描述

是否启用了Ceph对象网关缓存。类型

布尔型默认

`true`

`rgw cache lru size`描述

Ceph对象网关缓存中的条目数。类型

整数默认

`10000`

`rgw socket path`描述

域套接字的套接字路径。`FastCgiExternalServer` 使用此套接字。如果您未指定套接字路径，则Ceph Object Gateway将不会作为外部服务器运行。您在此处指定的路径必须与`rgw.conf`文件中指定的路径相同 。类型

串默认

不适用

`rgw fcgi socket backlog`描述

fcgi的套接字积压。类型

整数默认

`1024`

`rgw host`描述

Ceph对象网关实例的主机。可以是IP地址或主机名。类型

串默认

`0.0.0.0`

`rgw port`描述

实例的端口监听请求。如果未指定，则Ceph Object Gateway运行外部FastCGI。类型

串默认

没有

`rgw dns name`描述

服务域的DNS名称。另请参阅`hostnames`区域内的设置。类型

串默认

没有

`rgw script uri`描述

`SCRIPT_URI`请求中未设置的替代值。类型

串默认

没有

`rgw request uri`描述

`REQUEST_URI`请求中未设置的替代值。类型

串默认

没有

`rgw print continue`描述

`100-continue`如果可以运行，则启用。类型

布尔型默认

`true`

`rgw remote addr param`描述

远程地址参数。例如，包含远程地址的HTTP字段，或者`X-Forwarded-For` 如果反向代理可操作的地址。类型

串默认

`REMOTE_ADDR`

`rgw op thread timeout`描述

开放线程的超时时间（以秒为单位）。类型

整数默认

600

`rgw op thread suicide timeout`描述

`timeout`Ceph对象网关进程终止之前的时间（以秒为单位）。如果设置为，则禁用`0`。类型

整数默认

`0`

`rgw thread pool size`描述

线程池的大小。类型

整数默认

100个线程。

`rgw num control oids`描述

用于不同`rgw`实例之间的高速缓存同步的通知对象的数量。类型

整数默认

`8`

`rgw init timeout`描述

Ceph对象网关放弃初始化之前的秒数。类型

整数默认

`30`

`rgw mime types file`描述

MIME类型的路径和位置。用于对象类型的Swift自动检测。类型

串默认

`/etc/mime.types`

`rgw gc max objs`描述

一个垃圾回收处理周期中垃圾回收可以处理的最大对象数。类型

整数默认

`32`

`rgw gc obj min wait`描述

可以通过垃圾回收处理删除和处理对象之前的最短等待时间。类型

整数默认

`2 * 3600`

`rgw gc processor max time`描述

两个连续的垃圾回收处理周期开始之间的最长时间。类型

整数默认

`3600`

`rgw gc processor period`描述

垃圾收集处理的周期时间。类型

整数默认

`3600`

`rgw s3 success create obj status`描述

的备用成功状态响应`create-obj`。类型

整数默认

`0`

`rgw resolve cname`描述

是否`rgw`应使用请求主机名字段的DNS CNAME记录（如果主机名不等于）。`rgw dns name`类型

布尔型默认

`false`

`rgw obj stripe size`描述

Ceph对象网关对象的对象条带大小。有关条带化的详细信息，请参见[体系结构](https://docs.ceph.com/docs/nautilus/architecture#data-striping)。类型

整数默认

`4 << 20`

`rgw extended http attrs`描述

添加可以在实体（用户，存储桶或对象）上设置的新属性集。在放置实体或使用POST方法修改实体时，可以通过HTTP标头字段设置这些额外的属性。如果设置，则在实体上执行GET / HEAD时，这些属性将作为HTTP字段返回。类型

串默认

没有例

“ content\_foo，content\_bar，x-foo-bar”

`rgw exit timeout secs`描述

无条件退出之前等待进程的秒数。类型

整数默认

`120`

`rgw get obj window size`描述

单个对象请求的窗口大小（以字节为单位）。类型

整数默认

`16 << 20`

`rgw get obj max req size`描述

发送到Ceph存储集群的单个get操作的最大请求大小。类型

整数默认

`4 << 20`

`rgw relaxed s3 bucket names`描述

为美国区域存储桶启用宽松的S3存储桶名称规则。类型

布尔型默认

`false`

`rgw list buckets max chunk`描述

列出用户存储桶时，在一次操作中要检索的最大存储桶数。类型

整数默认

`1000`

`rgw override bucket index max shards`描述

表示存储区索引对象的分片数，值为零表示没有分片。不建议将值设置得太大（例如，千），因为这会增加存储区列表的成本。应该在客户端或全局部分中设置此变量，以便将其自动应用于radosgw-admin命令。类型

整数默认

`0`

`rgw curl wait timeout ms`描述

某些`curl`呼叫的超时时间（以毫秒为单位）。类型

整数默认

`1000`

`rgw copy obj progress`描述

在长时间复制操作期间启用对象进度的输出。类型

布尔型默认

`true`

`rgw copy obj progress every bytes`描述

复制进度输出之间的最小字节。类型

整数默认

`1024 * 1024`

`rgw admin entry`描述

管理员请求URL的入口点。类型

串默认

`admin`

`rgw content length compat`描述

启用同时设置了CONTENT\_LENGTH和HTTP\_CONTENT\_LENGTH的FCGI请求的兼容性处理。类型

布尔型默认

`false`

`rgw bucket quota ttl`描述

缓存的配额信息以秒为单位的时间是受信任的。超时后，将从群集中重新获取配额信息。类型

整数默认

`600`

`rgw user quota bucket sync interval`描述

同步到群集之前，存储桶配额信息的时间（以秒为单位）已累积。在此期间，其他RGW实例将不会从该实例的操作中看到存储区配额统计信息的更改。类型

整数默认

`180`

`rgw user quota sync interval`描述

在同步到群集之前，会累积用户配额信​​息的时间（以秒为单位）。在此期间，其他RGW实例将不会看到对该实例的操作导致的用户配额统计信息的更改。类型

整数默认

`180`

`rgw bucket default quota max objects`描述

每个存储桶的默认最大对象数。如果未指定其他配额，则在新用户上设置。对现有用户没有影响。应该在客户端或全局部分中设置此变量，以便将其自动应用于radosgw-admin命令。类型

整数默认

`-1`

`rgw bucket default quota max size`描述

每个存储分区的默认最大容量，以字节为单位。如果未指定其他配额，则在新用户上设置。对现有用户没有影响。类型

整数默认

`-1`

`rgw user default quota max objects`描述

用户的默认最大对象数。这包括用户拥有的所有存储桶中的所有对象。如果未指定其他配额，则在新用户上设置。对现有用户没有影响。类型

整数默认

`-1`

`rgw user default quota max size`描述

如果未指定其他配额，则为新用户设置的用户最大大小配额的值（以字节为单位）。对现有用户没有影响。类型

整数默认

`-1`

`rgw verify ssl`描述

发出请求时验证SSL证书。类型

布尔型默认

`true`

### 多站点设置

珠宝的新版本。

您可以在每个`[client.radosgw.{instance-name}]`实例下的Ceph配置文件中包括以下设置。

`rgw zone`描述

网关实例的区域名称。如果未设置区域，则可以使用命令配置群集范围的默认值 。`radosgw-admin zone default`类型

串默认

没有

`rgw zonegroup`描述

网关实例的区域组的名称。如果未设置zonegroup，则可以使用命令配置群集范围的默认值。`radosgw-admin zonegroup default`类型

串默认

没有

`rgw realm`描述

网关实例的领域的名称。如果未设置领域，则可以使用命令配置群集范围的默认值 。`radosgw-admin realm default`类型

串默认

没有

`rgw run sync thread`描述

如果领域中还有其他要同步的区域，请生成线程以处理数据和元数据的同步。类型

布尔型默认

`true`

`rgw data log window`描述

数据日志条目窗口（以秒为单位）。类型

整数默认

`30`

`rgw data log changes size`描述

数据更改日志要保留的内存中条目数。类型

整数默认

`1000`

`rgw data log obj prefix`描述

数据日志的对象名称前缀。类型

串默认

`data_log`

`rgw data log num shards`描述

保留数据更改日志的分片（对象）的数量。类型

整数默认

`128`

`rgw md log max shards`描述

元数据日志的最大分片数。类型

整数默认

`64`

重要 

开始同步后，不得更改和 的值。`rgw data log num shardsrgw md log max shards`

### 快捷设置

`rgw enforce swift acls`描述

强制执行“快速访问控制列表”（ACL）设置。类型

布尔型默认

`true`

`rgw swift token expiration`描述

Swift令牌到期的时间（以秒为单位）。类型

整数默认

`24 * 3600`

`rgw swift url`描述

Ceph对象网关Swift API的URL。类型

串默认

没有

`rgw swift url prefix`描述

Swift API的URL前缀，以区别于S3 API端点。默认值为`swift`，这使Swift API可以在URL上使用 `http://host:port/swift/v1`（ `http://host:port/swift/v1/AUTH_%(tenant_id)s`如果 已启用）。`rgw swift account in url`

为了兼容性，将此配置变量设置为空字符串会导致使用默认值`swift`；如果您确实需要一个空前缀，请将此选项设置为 `/`。

警告 

如果将此选项设置为`/`，则必须通过修改为exclude 来禁用S3 API 。不能同时使用radosgw 和S3和Swift API来支持它们。如果确实需要同时支持两个不带前缀的API，请部署多个radosgw实例以侦听不同的主机（或端口），从而为S3和Swift启用某些实例。`rgw enable apiss3rgw swift url prefix = /`默认

`swift`例

“ /快速测试”

`rgw swift auth url`描述

用于验证v1身份验证令牌的默认URL（如果未使用内部Swift身份验证）。类型

串默认

没有

`rgw swift auth entry`描述

Swift身份验证URL的入口点。类型

串默认

`auth`

`rgw swift account in url`描述

Swift API URL中是否应包含Swift帐户名称。

如果设置为`false`（默认值），则Swift API将侦听类似组成的URL `http://host:port/<rgw_swift_url_prefix>/v1`，并且将从请求标头中推断帐户名（如果radosgw配置了[Keystone集成，](https://docs.ceph.com/docs/nautilus/radosgw/keystone)则通常为Keystone项目UUID ）。

如果设置为`true`，则Swift API URL将 改为`http://host:port/<rgw_swift_url_prefix>/v1/AUTH_<account_name>` （或 `http://host:port/<rgw_swift_url_prefix>/v1/AUTH_<keystone_project_id>`），并且`object-store`必须相应地将Keystone 端点配置为包括 `AUTH_%(tenant_id)s`后缀。

您**必须**将此选项设置为`true`（和更新梯形服务目录），如果你想radosgw支持公开可读容器和[临时网址](https://docs.ceph.com/docs/nautilus/radosgw/swift/tempurl)。类型

布尔型默认

`false`

`rgw swift versioning enabled`描述

启用OpenStack对象存储API的对象版本控制。这允许客户端将`X-Versions-Location`属性放在应版本化的容器上。该属性指定存储归档版本的容器的名称。由于访问控制验证，该用户必须与版本容器相同的用户拥有-不考虑ACL。那些容器不能通过S3对象版本控制机制进行版本控制。

该`X-History-Location`属性也由OpenStack的斯威夫特处理了解`DELETE`操作 [略有不同](https://docs.openstack.org/swift/latest/overview_object_versioning.html) 的`X-Versions-Location`，目前不支持。类型

布尔型默认

`false`

`rgw trust forwarded https`描述

当radosgw前面的代理用于ssl终止时，radosgw不知道传入的HTTP连接是否安全。在确定连接是否安全时，启用此选项可信任代理发送的`Forwarded`和`X-Forwarded-Proto`标头。这是某些功能（例如服务器端加密）所必需的。类型

布尔型默认

`false`

### 记录设置

`rgw log nonexistent bucket`描述

使Ceph Object Gateway可以记录对不存在的存储桶的请求。类型

布尔型默认

`false`

`rgw log object name`描述

对象名称的日志记录格式。有关格式说明符的详细信息，请参见联机帮助 _日期_。类型

日期默认

`%Y-%m-%d-%H-%i-%n`

`rgw log object name utc`描述

记录的对象名称是否包含UTC时间。如果为`false`，则使用当地时间。类型

布尔型默认

`false`

`rgw usage max shards`描述

使用率日志记录的最大分片数。类型

整数默认

`32`

`rgw usage max user shards`描述

用于单个用户的使用记录的最大分片数。类型

整数默认

`1`

`rgw enable ops log`描述

为每个成功的Ceph Object Gateway操作启用日志记录。类型

布尔型默认

`false`

`rgw enable usage log`描述

启用使用情况日志。类型

布尔型默认

`false`

`rgw ops log rados`描述

是否将操作日志写入Ceph存储群集后端。类型

布尔型默认

`true`

`rgw ops log socket path`描述

用于写入操作日志的Unix域套接字。类型

串默认

没有

`rgw ops log data backlog`描述

写入Unix域套接字的操作日志的最大数据积压数据大小。类型

整数默认

`5 << 20`

`rgw usage log flush threshold`描述

同步刷新之前，使用率日志中的脏合并条目数。类型

整数默认

1024

`rgw usage log tick interval`描述

每秒钟刷新待处理的使用情况日志数据`n`。类型

整数默认

`30`

`rgw log http headers`描述

以逗号分隔的HTTP标头列表，包含在ops日志条目中。标题名称不区分大小写，并使用完整的标题名称，并用下划线分隔单词。类型

串默认

没有例

“ http\_x\_forwarded\_for，http\_x\_special\_k”

`rgw intent log object name`描述

意向日志对象名称的日志记录格式。有关格式说明符的详细信息，请参见联机帮助 _日期_。类型

日期默认

`%Y-%m-%d-%i-%n`

`rgw intent log object name utc`描述

意向日志对象名称是否包含UTC时间。如果为`false`，则使用当地时间。类型

布尔型默认

`false`

### 梯形校正设置

`rgw keystone url`描述

Keystone服务器的URL。类型

串默认

没有

`rgw keystone api version`描述

应用于与Keystone服务器通信的OpenStack Identity API版本（2或3）。类型

整数默认

`2`

`rgw keystone admin domain`描述

使用OpenStack Identity API v3时具有管理员特权的OpenStack域的名称。类型

串默认

没有

`rgw keystone admin project`描述

使用OpenStack Identity API v3时具有管理员特权的OpenStack项目的名称。如果未指定，则将使用的值 。`rgw keystone admin tenant`类型

串默认

没有

`rgw keystone admin token`描述

Keystone管理员令牌（共享机密）。在与令牌认证上与管理员凭据具有优先级的管理Ceph的RadosGW认证（，， ，， ）。Keystone管理令牌已被弃用，但可用于与较早的环境集成。最好 避免暴露令牌。`rgw keystone admin userrgw keystone admin passwordrgw keystone admin tenantrgw keystone admin projectrgw keystone admin domainrgw keystone admin token path`类型

串默认

没有

`rgw keystone admin token path`描述

包含Keystone管理令牌（共享机密）的文件的路径。在与令牌认证上与管理员凭据具有优先级的管理Ceph的RadosGW认证（，， ，， ）。Keystone管理令牌已被弃用，但可用于与较早的环境集成。`rgw keystone admin userrgw keystone admin passwordrgw keystone admin tenantrgw keystone admin projectrgw keystone admin domain`类型

串默认

没有

`rgw keystone admin tenant`描述

使用OpenStack Identity API v2时具有管理员特权的OpenStack租户的名称（服务租户）类型

串默认

没有

`rgw keystone admin user`描述

OpenStack Identity API v2时具有Keystone身份验证（服务用户）的管理员特权的OpenStack用户的名称类型

串默认

没有

`rgw keystone admin password`描述

使用OpenStack Identity API v2时OpenStack管理员用户的密码。最好 避免暴露令牌。`rgw keystone admin password path`类型

串默认

没有

`rgw keystone admin password path`描述

使用OpenStack Identity API v2时包含OpenStack管理员用户密码的文件的路径。类型

串默认

没有

`rgw keystone accepted roles`描述

角色需要服务请求。类型

串默认

`Member, admin`

`rgw keystone token cache size`描述

每个Keystone令牌缓存中的最大条目数。类型

整数默认

`10000`

`rgw keystone revocation interval`描述

令牌吊销检查之间的秒数。类型

整数默认

`15 * 60`

`rgw keystone verify ssl`描述

向基石发出令牌请求时，请验证SSL证书。类型

布尔型默认

`true`

### 外堡设置

`rgw barbican url`描述

Barbican服务器的URL。类型

串默认

没有

`rgw keystone barbican user`描述

可以访问 用于[加密](https://docs.ceph.com/docs/nautilus/radosgw/encryption)的[Barbican](https://docs.ceph.com/docs/nautilus/radosgw/barbican)机密的OpenStack用户的名称。类型

串默认

没有

`rgw keystone barbican password`描述

与[Barbican](https://docs.ceph.com/docs/nautilus/radosgw/barbican)用户关联的密码。类型

串默认

没有

`rgw keystone barbican tenant`描述

使用OpenStack Identity API v2时与[Barbican](https://docs.ceph.com/docs/nautilus/radosgw/barbican)用户关联的OpenStack租户的名称。类型

串默认

没有

`rgw keystone barbican project`描述

使用OpenStack Identity API v3时，与[Barbican](https://docs.ceph.com/docs/nautilus/radosgw/barbican)用户关联的OpenStack项目的名称。类型

串默认

没有

`rgw keystone barbican domain`描述

使用OpenStack Identity API v3时，与[Barbican](https://docs.ceph.com/docs/nautilus/radosgw/barbican)用户关联的OpenStack域的名称。类型

串默认

没有

