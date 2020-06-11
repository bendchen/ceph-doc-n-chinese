# 多站点

## 多站点

J版本新功能。

一个区域配置通常由一个区域组组成，该区域组包含一个区域和一个或多个ceph-radosgw实例，您可以在这些实例之间负载均衡网关客户端请求。在单个区域配置中，通常多个网关实例指向单个Ceph存储群集。但是，Kraken支持Ceph Object Gateway的多个多站点配置选项：

* **多区域：**更高级的配置由一个区域组和多个区域组成，每个区域具有一个或多个ceph-radosgw实例。每个区域均由其自己的Ceph存储群集支持。如果区域之一出现严重故障，则区域组中的多个区域可为区域组提供灾难恢复。在Kraken中，每个区域都处于活动状态，并且可以接收写操作。除灾难恢复外，多个活动区域还可以用作内容交付网络的基础。
* **多区域组：** Ceph Object Gateway以前称为“区域”，也可以支持多个区域组，每个区域组具有一个或多个区域。存储到与另一个区域组在同一领域内的一个区域组中的区域的对象将共享全局对象名称空间，从而确保跨区域组和区域的唯一对象ID。
* **多个领域：**在Kraken中，Ceph对象网关支持领域的概念，领域可以是单个区域组或多个区域组以及该领域的全局唯一名称空间。多个领域提供了支持众多配置和名称空间的能力。

在区域组内的区域之间复制对象数据看起来像这样：![../../\_images/zone-sync2.png](https://docs.ceph.com/docs/nautilus/_images/zone-sync2.png)

有关设置集群的更多详细信息，请参阅《[用于生产的Ceph对象网关》](https://access.redhat.com/documentation/en-us/red_hat_ceph_storage/2/html/ceph_object_gateway_for_production/)。

### INFERNALIS的功能变化

在Kraken中，您可以配置每个Ceph对象网关以在主动/主动区域配置中工作，从而允许写入非主区域。

多站点配置存储在称为“领域”的容器中。领域存储区域组，区域和具有多个时期的时间“期间”，用于跟踪配置更改。在Kraken中，`ceph-radosgw`守护程序处理同步，从而无需单独的同步代理。此外，新的同步方法允许Ceph对象网关以“主动-主动”配置而不是“主动-被动”运行。

### 要求和假设

多站点配置需要至少两个Ceph存储群集，最好给定一个不同的群集名称。至少两个Ceph对象网关实例，每个Ceph存储集群一个。

本指南假定至少两个Ceph存储群集位于地理位置不同的位置；但是，该配置可以在同一站点上工作。本指南还假定了两个名为`rgw1`和的Ceph对象网关服务器 `rgw2`。

重要 

除非您具有低延迟的WAN连接，否则不建议运行单个Ceph存储群集。

多站点配置需要一个主区域组和一个主区域。此外，每个区域组都需要一个主区域。区域组可以具有一个或多个辅助或非主区域。

在本指南中，`rgw1`主机将充当主控区域组的主控区域。并且，`rgw2`主机将充当主区域组的辅助区域。

有关为Ceph对象存储创建和调整[池](https://docs.ceph.com/docs/nautilus/radosgw/pools)的说明，请参阅[池](https://docs.ceph.com/docs/nautilus/radosgw/pools)。

### 配置主区域

多站点配置中的所有网关将从`ceph-radosgw`主区域组和主区域中主机上的守护程序检索其配置。要在多站点配置中配置网关，请选择一个`ceph-radosgw`实例来配置主区域组和主区域。

#### 创建一个领域

领域包含区域组和区域的多站点配置，并且还用于在领域内强制执行全局唯一的名称空间。

通过在标识为在主区域组和区域中使用的主机上打开命令行界面，为多站点配置创建新领域。然后，执行以下操作：

```text
# radosgw-admin realm create --rgw-realm={realm-name} [--default]
```

例如：

```text
# radosgw-admin realm create --rgw-realm=movies --default
```

如果集群只有一个领域，请指定该`--default`标志。如果`--default`指定，`radosgw-admin`将默认使用此领域。如果`--default`未指定，则添加区域组和区域需要在添加区域组和区域时指定`--rgw-realm`标志或 `--realm-id`标记以标识领域。

创建领域之后，`radosgw-admin`将回显领域配置。例如：

```text
{
    "id": "0956b174-fe14-4f97-8b50-bb7ec5e1cf62",
    "name": "movies",
    "current_period": "1950b710-3e63-4c41-a19e-46a715000980",
    "epoch": 1
}
```

注意 

Ceph为领域生成唯一的ID，如果需要，它可以重命名领域。

#### 创建一个主区域组

一个域必须至少有一个区域组，它将用作该域的主区域组。

通过在标识为可在主区域组和区域中使用的主机上打开命令行界面，为多站点配置创建新的主区域组。然后，执行以下操作：

```text
# radosgw-admin zonegroup create --rgw-zonegroup={name} --endpoints={url} [--rgw-realm={realm-name}|--realm-id={realm-id}] --master --default
```

例如：

```text
# radosgw-admin zonegroup create --rgw-zonegroup=us --endpoints=http://rgw1:80 --rgw-realm=movies --master --default
```

如果该领域只有一个区域组，请指定该 `--default`标志。如果`--default`指定，`radosgw-admin` 则在添加新区域时默认使用此区域组。如果 `--default`未指定，则添加或修改区域时，添加区域将需要该 `--rgw-zonegroup`标志或该`--zonegroup-id`标志来标识区域组。

创建主区域组后，`radosgw-admin`将回显区域组配置。例如：

```text
{
    "id": "f1a233f5-c354-4107-b36c-df66126475a6",
    "name": "us",
    "api_name": "us",
    "is_master": "true",
    "endpoints": [
        "http:\/\/rgw1:80"
    ],
    "hostnames": [],
    "hostnames_s3webzone": [],
    "master_zone": "",
    "zones": [],
    "placement_targets": [],
    "default_placement": "",
    "realm_id": "0956b174-fe14-4f97-8b50-bb7ec5e1cf62"
}
```

#### 创建一个主区域

重要 

必须在区域内的Ceph对象网关节点上创建区域。

通过在标识为在主区域组和区域中使用的主机上打开命令行界面，为多站点配置创建新的主区域。然后，执行以下操作：

```text
# radosgw-admin zone create --rgw-zonegroup={zone-group-name} \
                            --rgw-zone={zone-name} \
                            --master --default \
                            --endpoints={http://fqdn}[,{http://fqdn}]
```

例如：

```text
# radosgw-admin zone create --rgw-zonegroup=us --rgw-zone=us-east \
                            --master --default \
                            --endpoints={http://fqdn}[,{http://fqdn}]
```

注意 

该`--access-key`和`--secret`未指定。在下一部分中创建用户后，这些设置将被添加到区域中。

重要 

以下步骤假定使用尚未安装数据的新安装系统进行多站点配置。`default`如果已使用区域及其池来存储数据，请不要删除 它，否则数据将被删除且无法恢复。

#### 删除默认区域组和区域

删除`default`区域（如果存在）。确保首先将其从默认区域组中删除。

```text
# radosgw-admin zonegroup remove --rgw-zonegroup=default --rgw-zone=default
# radosgw-admin period update --commit
# radosgw-admin zone rm --rgw-zone=default
# radosgw-admin period update --commit
# radosgw-admin zonegroup delete --rgw-zonegroup=default
# radosgw-admin period update --commit
```

最后，删除`default`Ceph存储群集中的池（如果存在）。

重要 

以下步骤假设使用当前未存储数据的新安装系统进行多站点配置。`default`如果已经使用区域组存储数据，则不要删除它。

```text
# ceph osd pool rm default.rgw.control default.rgw.control --yes-i-really-really-mean-it
# ceph osd pool rm default.rgw.data.root default.rgw.data.root --yes-i-really-really-mean-it
# ceph osd pool rm default.rgw.gc default.rgw.gc --yes-i-really-really-mean-it
# ceph osd pool rm default.rgw.log default.rgw.log --yes-i-really-really-mean-it
# ceph osd pool rm default.rgw.users.uid default.rgw.users.uid --yes-i-really-really-mean-it
```

#### 创建系统用户

该`ceph-radosgw`拉境界和周期信息之前，必须守护进程验证。在主区域中，创建一个系统用户以促进守护程序之间的认证。

```text
# radosgw-admin user create --uid="{user-name}" --display-name="{Display Name}" --system
```

例如：

```text
# radosgw-admin user create --uid="synchronization-user" --display-name="Synchronization User" --system
```

记下`access_key`和`secret_key`，因为辅助区域将要求它们与主区域进行身份验证。

最后，将系统用户添加到主区域。

```text
# radosgw-admin zone modify --rgw-zone=us-east --access-key={access-key} --secret={secret}
# radosgw-admin period update --commit
```

#### 更新期间

更新主区域配置后，请更新周期。

```text
# radosgw-admin period update --commit
```

注意 

更新时间段会更改纪元，并确保其他区域将接收更新的配置。

#### 更新CEPH配置文件

通过`rgw_zone`在实例条目中添加配置选项和主区域的名称来更新主区域主机上的Ceph配置文件 。

```text
[client.rgw.{instance-name}]
...
rgw_zone={zone-name}
```

例如：

```text
[client.rgw.rgw1]
host = rgw1
rgw frontends = "civetweb port=80"
rgw_zone=us-east
```

#### 启动网关

在对象网关主机上，启动并启用Ceph对象网关服务：

```text
# systemctl start ceph-radosgw@rgw.`hostname -s`
# systemctl enable ceph-radosgw@rgw.`hostname -s`
```

### 配置辅助区域

区域组中的区域将复制所有数据，以确保每个区域都具有相同的数据。创建辅助区域时，请在标识为服务于辅助区域的主机上执行以下所有操作。

注意 

要添加第三个区域，请遵循与添加第二个区域相同的步骤。使用其他区域名称。

重要 

您必须在主区域内的主机上执行元数据操作，例如用户创建。主区域和辅助区域可以接收存储桶操作，但是辅助区域将存储桶操作重定向到主区域。如果主区域关闭，则存储桶操作将失败。

#### 拉领域

使用主区域组中主区域的URL路径，访问密钥和机密，将领域配置拉到主机。要拉出非默认领域，请使用`--rgw-realm`或 `--realm-id`配置选项指定领域。

```text
# radosgw-admin realm pull --url={url-to-master-zone-gateway} --access-key={access-key} --secret={secret}
```

注意 

拉域还可以检索远程服务器的当前时间段配置，并使其也成为此主机上的当前时间段。

如果此领域是默认领域或唯一领域，请将该领域设置为默认领域。

```text
# radosgw-admin realm default --rgw-realm={realm-name}
```

#### 创建一个辅助区域

重要 

必须在区域内的Ceph对象网关节点上创建区域。

通过在标识为该辅助区域提供服务的主机上打开命令行界面，为多站点配置创建辅助区域。指定区域组ID，新的区域名称和该区域的端点。**请勿**使用`--master`或`--default`标志。在Kraken中，默认情况下，所有区域都以双活配置运行。也就是说，网关客户端可以将数据写入任何区域，并且该区域会将数据复制到区域组内的所有其他区域。如果辅助区域不接受写操作，请指定 `--read-only`标志以在主区域和辅助区域之间创建主动-被动配置。此外，提供存储在主区域组的主区域中的生成的系统用户的 `access_key`和`secret_key`。执行以下命令：

```text
# radosgw-admin zone create --rgw-zonegroup={zone-group-name}\
                            --rgw-zone={zone-name} --endpoints={url} \
                            --access-key={system-key} --secret={secret}\
                            --endpoints=http://{fqdn}:80 \
                            [--read-only]
```

例如：

```text
# radosgw-admin zone create --rgw-zonegroup=us --rgw-zone=us-west \
                            --access-key={system-key} --secret={secret} \
                            --endpoints=http://rgw2:80
```

重要 

以下步骤假定使用不存储数据的新安装系统进行多站点配置。**不要删除**的 `default`，如果你已经在使用它来存储数据区和游泳池，或者数据将会丢失且无法恢复。

如果需要，请删除默认区域。

```text
# radosgw-admin zone rm --rgw-zone=default
```

最后，根据需要删除Ceph存储集群中的默认池。

```text
# ceph osd pool rm default.rgw.control default.rgw.control --yes-i-really-really-mean-it
# ceph osd pool rm default.rgw.data.root default.rgw.data.root --yes-i-really-really-mean-it
# ceph osd pool rm default.rgw.gc default.rgw.gc --yes-i-really-really-mean-it
# ceph osd pool rm default.rgw.log default.rgw.log --yes-i-really-really-mean-it
# ceph osd pool rm default.rgw.users.uid default.rgw.users.uid --yes-i-really-really-mean-it
```

#### 更新CEPH配置文件

通过`rgw_zone`在实例条目中添加配置选项和辅助区域的名称来更新辅助区域主机上的Ceph配置文件。

```text
[client.rgw.{instance-name}]
...
rgw_zone={zone-name}
```

例如：

```text
[client.rgw.rgw2]
host = rgw2
rgw frontends = "civetweb port=80"
rgw_zone=us-west
```

#### 更新期间

更新主区域配置后，请更新周期。

```text
# radosgw-admin period update --commit
```

注意 

更新时间段会更改纪元，并确保其他区域将接收更新的配置。

#### 启动网关

在对象网关主机上，启动并启用Ceph对象网关服务：

```text
# systemctl start ceph-radosgw@rgw.`hostname -s`
# systemctl enable ceph-radosgw@rgw.`hostname -s`
```

#### 检查同步状态

辅助区域启动并运行后，检查同步状态。同步会将在主区域中创建的用户和存储桶复制到辅助区域。

```text
# radosgw-admin sync status
```

输出将提供同步操作的状态。例如：

```text
realm f3239bc5-e1a8-4206-a81d-e1576480804d (earth)
    zonegroup c50dbb7e-d9ce-47cc-a8bb-97d9b399d388 (us)
         zone 4c453b70-4a16-4ce8-8185-1893b05d346e (us-west)
metadata sync syncing
              full sync: 0/64 shards
              metadata is caught up with master
              incremental sync: 64/64 shards
    data sync source: 1ee9da3e-114d-4ae3-a8a4-056e8a17f532 (us-east)
                      syncing
                      full sync: 0/128 shards
                      incremental sync: 128/128 shards
                      data is caught up with source
```

注意 

辅助区域接受铲斗操作；但是，辅助区域将存储桶操作重定向到主区域，然后与主区域同步以接收存储桶操作的结果。如果主区域关闭，则在辅助区域上执行的存储桶操作将失败，但对象操作应成功。

### 维护

#### 检查同步状态

可以通过以下方式查询有关区域复制状态的信息：

```text
$ radosgw-admin sync status
        realm b3bc1c37-9c44-4b89-a03b-04c269bea5da (earth)
    zonegroup f54f9b22-b4b6-4a0e-9211-fa6ac1693f49 (us)
         zone adce11c9-b8ed-4a90-8bc5-3fc029ff0816 (us-2)
        metadata sync syncing
              full sync: 0/64 shards
              incremental sync: 64/64 shards
              metadata is behind on 1 shards
              oldest incremental change not applied: 2017-03-22 10:20:00.0.881361s
    data sync source: 341c2d81-4574-4d08-ab0f-5a2a7b168028 (us-1)
                      syncing
                      full sync: 0/128 shards
                      incremental sync: 128/128 shards
                      data is caught up with source
              source: 3b5d1a3f-3f27-4e4a-8f34-6072d4bb1275 (us-3)
                      syncing
                      full sync: 0/128 shards
                      incremental sync: 128/128 shards
                      data is caught up with source
```

#### 更改元数据主区域

重要 

更改元数据主文件所在的区域时必须小心。如果某个区域尚未完成从当前主区域的元数据同步，则当提升为主区域时，该区域将无法提供任何剩余条目，并且这些更改将丢失。因此，建议先等待区域赶上元数据同步，然后再将其升级为主节点。`radosgw-admin sync status`

同样，如果当前主区域正在处理对元数据的更改，而另一个区域正被提升为主区域，则这些更改很可能会丢失。为避免这种情况，建议关闭`radosgw`前一个主区域上的所有实例。升级另一个区域后，可以获取其新的时间段，并可以重新启动网关。`radosgw-admin period pull`

要将区域（例如`us-2`zonegroup中的zone `us`）提升为元数据主服务器，请在该区域上运行以下命令：

```text
$ radosgw-admin zone modify --rgw-zone=us-2 --master
$ radosgw-admin zonegroup modify --rgw-zonegroup=us --master
$ radosgw-admin period update --commit
```

这将产生一个新的周期，区域中的radosgw实例`us-2` 会将这个周期发送到其他区域。

### 故障转移和灾难恢复

如果主区域应发生故障，请故障转移到辅助区域以进行灾难恢复。

1. 将辅助区域设置为主区域和默认区域。例如：

   ```text
   # radosgw-admin zone modify --rgw-zone={zone-name} --master --default
   ```

   默认情况下，Ceph对象网关将以双活配置运行。如果将群集配置为在主动-被动配置中运行，则辅助区域是只读区域。删除`--read-only`状态以允许区域接收写操作。例如：

   ```text
   # radosgw-admin zone modify --rgw-zone={zone-name} --master --default \
                               --read-only=false
   ```

2. 更新期限以使更改生效。

   ```text
   # radosgw-admin period update --commit
   ```

3. 最后，重新启动Ceph对象网关。

   ```text
   # systemctl restart ceph-radosgw@rgw.`hostname -s`
   ```

如果以前的主控区域恢复，请还原操作。

1. 从恢复的区域中，从当前的主区域中提取最新的领域配置。

   ```text
   # radosgw-admin realm pull --url={url-to-master-zone-gateway} \
                              --access-key={access-key} --secret={secret}
   ```

2. 将恢复的区域设置为主区域和默认区域。

   ```text
   # radosgw-admin zone modify --rgw-zone={zone-name} --master --default
   ```

3. 更新期限以使更改生效。

   ```text
   # radosgw-admin period update --commit
   ```

4. 然后，在恢复的区域中重新启动Ceph对象网关。

   ```text
   # systemctl restart ceph-radosgw@rgw.`hostname -s`
   ```

5. 如果辅助区域需要为只读配置，请更新辅助区域。

   ```text
   # radosgw-admin zone modify --rgw-zone={zone-name} --read-only
   ```

6. 更新期限以使更改生效。

   ```text
   # radosgw-admin period update --commit
   ```

7. 最后，在辅助区域中重新启动Ceph对象网关。

   ```text
   # systemctl restart ceph-radosgw@rgw.`hostname -s`
   ```

### 将单站点系统迁移到多站点

要将具有`default`区域组和区域的单站点系统迁移到多站点系统，请使用以下步骤：

1. 创建一个领域。用`<name>`领域名称替换。

   ```text
   # radosgw-admin realm create --rgw-realm=<name> --default
   ```

2. 重命名默认区域和区域组。替换`<name>`为zonegroup或zone名称。

   ```text
   # radosgw-admin zonegroup rename --rgw-zonegroup default --zonegroup-new-name=<name>
   # radosgw-admin zone rename --rgw-zone default --zone-new-name us-east-1 --rgw-zonegroup=<name>
   ```

3. 配置主区域组。替换`<name>`为领域或区域组名称。替换`<fqdn>`为zonegroup中的完全限定域名。

   ```text
   # radosgw-admin zonegroup modify --rgw-realm=<name> --rgw-zonegroup=<name> --endpoints http://<fqdn>:80 --master --default
   ```

4. 配置主区域。替换`<name>`为领域，区域组或区域名称。替换`<fqdn>`为zonegroup中的完全限定域名。

   ```text
   # radosgw-admin zone modify --rgw-realm=<name> --rgw-zonegroup=<name> \
                               --rgw-zone=<name> --endpoints http://<fqdn>:80 \
                               --access-key=<access-key> --secret=<secret-key> \
                               --master --default
   ```

5. 创建一个系统用户。替换`<user-id>`为用户名。用`<display-name>`显示名称替换。它可能包含空格。

   ```text
   # radosgw-admin user create --uid=<user-id> --display-name="<display-name>"\
                               --access-key=<access-key> --secret=<secret-key> --system
   ```

6. 提交更新的配置。

   ```text
   # radosgw-admin period update --commit
   ```

7. 最后，重新启动Ceph对象网关。

   ```text
   # systemctl restart ceph-radosgw@rgw.`hostname -s`
   ```

完成此过程之后，继续进行“ [配置](https://docs.ceph.com/docs/nautilus/radosgw/multisite/#configure-secondary-zones)辅助区域”以在主区域组中创建辅助区域。

### 多站点配置参考

以下各节提供了有关领域，句点，区域组和区域的其他详细信息和命令行用法。

#### 领域

领域表示全局唯一的名称空间，该名称空间由一个或多个包含一个或多个区域的区域组和包含存储桶的区域组成，而存储桶又包含对象。领域使Ceph对象网关能够在同一硬件上支持多个名称空间及其配置。

领域包含句点的概念。每个时间段代表区域组的状态和时间上的区域配置。每次更改区域组或区域时，请更新时间段并提交。

默认情况下，Ceph对象网关不会创建与Infernalis和早期版本向后兼容的领域。但是，作为最佳实践，我们建议为新集群创建领域。

**创建一个领域**

要创建领域，请执行并指定领域名称。如果领域是默认领域，请指定。`realm create--default`

```text
# radosgw-admin realm create --rgw-realm={realm-name} [--default]
```

例如：

```text
# radosgw-admin realm create --rgw-realm=movies --default
```

通过指定`--default`，每次`radosgw-admin`调用都会隐式调用`--rgw-realm`领域，除非明确提供了领域名称。

**将领域设为默认设置**

领域列表中的一个领域应该是默认领域。可能只有一个默认领域。如果只有一个领域，并且在创建时未将其指定为默认领域，请使其成为默认领域。或者，要更改默认领域，请执行：

```text
# radosgw-admin realm default --rgw-realm=movies
```

注意 

当领域是默认域时，命令行将 `--rgw-realm=<realm-name>`作为参数。

**删除领域**

要删除领域，请执行并指定领域名称。`realm delete`

```text
# radosgw-admin realm delete --rgw-realm={realm-name}
```

例如：

```text
# radosgw-admin realm delete --rgw-realm=movies
```

**获取领域**

要获取领域，请执行并指定领域名称。`realm get`

```text
#radosgw-admin realm get --rgw-realm=<name>
```

例如：

```text
# radosgw-admin realm get --rgw-realm=movies [> filename.json]
```

CLI将回显具有领域属性的JSON对象。

```text
{
    "id": "0a68d52e-a19c-4e8e-b012-a8f831cb3ebc",
    "name": "movies",
    "current_period": "b0c5bbef-4337-4edd-8184-5aeab2ec413b",
    "epoch": 1
}
```

使用`>`和输出文件名将JSON对象输出到文件。

**设置领域**

要设置领域，请执行，指定领域名称，并 输入文件名。`realm set--infile=`

```text
#radosgw-admin realm set --rgw-realm=<name> --infile=<infilename>
```

例如：

```text
# radosgw-admin realm set --rgw-realm=movies --infile=filename.json
```

**列表领域**

要列出领域，请执行。`realm list`

```text
# radosgw-admin realm list
```

**列出领域期间**

要列出领域期间，请执行。`realm list-periods`

```text
# radosgw-admin realm list-periods
```

**拉一个领域**

要将领域从包含主区域组和主区域的节点拉到包含辅助区域组或区域的节点，请在将接收领域配置的节点上执行 。`realm pull`

```text
# radosgw-admin realm pull --url={url-to-master-zone-gateway} --access-key={access-key} --secret={secret}
```

**重命名领域**

领域不是期间的一部分。因此，重命名领域仅在本地应用，不会被使用拖拉。当重命名具有多个区域的领域时，请在每个区域上运行命令。要重命名领域，请执行以下操作：`realm pull`

```text
# radosgw-admin realm rename --rgw-realm=<current-name> --realm-new-name=<new-realm-name>
```

注意 

请勿用于更改参数。这仅更改内部名称。指定仍将使用旧的领域名称。`realm setname--rgw-realm`

#### 区域群组

Ceph对象网关通过使用区域组的概念来支持多站点部署和全局名称空间。区域组以前称为Infernalis中的一个区域，它定义了一个或多个区域内一个或多个Ceph对象网关实例的地理位置。

配置区域组不同于典型的配置过程，因为并非所有设置最终都存储在Ceph配置文件中。您可以列出区域组，获取区域组配置并设置区域组配置。

**创建区域组**

创建区域组包括指定区域组名称。除非`--rgw-realm=<realm-name>`指定，否则创建一个区域将假定它会存在于默认领域 中。如果zonegroup是默认的zonegroup，请指定该`--default`标志。如果区域组是主区域组，请指定`--master`标志。例如：

```text
# radosgw-admin zonegroup create --rgw-zonegroup=<name> [--rgw-realm=<name>][--master] [--default]
```

注意 

使用修改现有的区组的设置。`zonegroup modify --rgw-zonegroup=<zonegroup-name>`

**将区域组设置为默认**

区域组列表中的一个区域组应为默认区域组。可能只有一个默认区域组。如果只有一个区域组，并且在创建时未将其指定为默认区域组，则将其设为默认区域组。或者，要更改默认的区域组，请执行：

```text
# radosgw-admin zonegroup default --rgw-zonegroup=comedy
```

注意 

当zonegroup为默认值时，命令行假定 `--rgw-zonegroup=<zonegroup-name>`为参数。

然后，更新期间：

```text
# radosgw-admin period update --commit
```

**将区域添加到区域组**

要将区域添加到区域组，请执行以下操作：

```text
# radosgw-admin zonegroup add --rgw-zonegroup=<name> --rgw-zone=<name>
```

然后，更新期间：

```text
# radosgw-admin period update --commit
```

**从区域组中删除区域**

要从区域组中删除区域，请执行以下操作：

```text
# radosgw-admin zonegroup remove --rgw-zonegroup=<name> --rgw-zone=<name>
```

然后，更新期间：

```text
# radosgw-admin period update --commit
```

**重新命名区域集团**

要重命名区域组，请执行以下操作：

```text
# radosgw-admin zonegroup rename --rgw-zonegroup=<name> --zonegroup-new-name=<name>
```

然后，更新期间：

```text
# radosgw-admin period update --commit
```

**删除区域集团**

要删除区域组，请执行以下操作：

```text
# radosgw-admin zonegroup delete --rgw-zonegroup=<name>
```

然后，更新期间：

```text
# radosgw-admin period update --commit
```

**列表区域组**

一个Ceph集群包含一个区域组列表。要列出区域组，请执行：

```text
# radosgw-admin zonegroup list
```

在`radosgw-admin`返回区组的JSON格式的列表。

```text
{
    "default_info": "90b28698-e7c3-462c-a42d-4aa780d24eda",
    "zonegroups": [
        "us"
    ]
}
```

**获取区域组地图**

要列出每个区域组的详细信息，请执行：

```text
# radosgw-admin zonegroup-map get
```

注意 

如果您收到一个错误，运行 的第一个。`failed to read zonegroup mapradosgw-admin zonegroup-map updateroot`

**获取区域集团**

要查看区域组的配置，请执行：

```text
radosgw-admin zonegroup get [--rgw-zonegroup=<zonegroup>]
```

区域组配置如下所示：

```text
{
    "id": "90b28698-e7c3-462c-a42d-4aa780d24eda",
    "name": "us",
    "api_name": "us",
    "is_master": "true",
    "endpoints": [
        "http:\/\/rgw1:80"
    ],
    "hostnames": [],
    "hostnames_s3website": [],
    "master_zone": "9248cab2-afe7-43d8-a661-a40bf316665e",
    "zones": [
        {
            "id": "9248cab2-afe7-43d8-a661-a40bf316665e",
            "name": "us-east",
            "endpoints": [
                "http:\/\/rgw1"
            ],
            "log_meta": "true",
            "log_data": "true",
            "bucket_index_max_shards": 0,
            "read_only": "false"
        },
        {
            "id": "d1024e59-7d28-49d1-8222-af101965a939",
            "name": "us-west",
            "endpoints": [
                "http:\/\/rgw2:80"
            ],
            "log_meta": "false",
            "log_data": "true",
            "bucket_index_max_shards": 0,
            "read_only": "false"
        }
    ],
    "placement_targets": [
        {
            "name": "default-placement",
            "tags": []
        }
    ],
    "default_placement": "default-placement",
    "realm_id": "ae031368-8715-4e27-9a99-0c9468852cfe"
}
```

**设置区域集团**

定义区域组包括创建一个JSON对象，至少指定所需的设置：

1. `name`：区域组的名称。需要。
2. `api_name`：区域组的API名称。可选的。
3. `is_master`：确定区域组是否是主区域组。需要。**注意：**您只能有一个主区域组。
4. `endpoints`：区域组中所有端点的列表。例如，您可以使用多个域名来引用同一区域组。请记住不要使用正斜杠（`\/`）。您也可以`fqdn:port`为每个端点指定一个端口（）。可选的。
5. `hostnames`：区域组中所有主机名的列表。例如，您可以使用多个域名来引用同一区域组。可选的。该设置将自动包含在此列表中。更改此设置后，应重新启动网关守护程序。`rgw dns name`
6. `master_zone`：区域组的主区域。可选的。如果未指定，则使用默认区域。**注意：**每个区域组只能有一个主区域。
7. `zones`：区域组中所有区域的列表。每个区域都有一个名称（必填），一个端点列表（可选）以及网关是否将记录元数据和数据操作（默认情况下为false）。
8. `placement_targets`：展示位置定位条件列表（可选）。每个放置目标均包含该放置目标的名称（必填）和标签列表（可选），以便只有具有标签的用户才能使用该放置目标（即，`placement_tags`用户信息中的用户字段）。
9. `default_placement`：对象索引和对象数据的默认放置目标。`default-placement`默认设置为。您还可以在每个用户的用户信息中设置每个用户的默认展示位置。

要设置区域组，请创建一个包含必填字段的JSON对象，然后将该对象保存到文件中（例如`zonegroup.json`）；然后，执行以下命令：

```text
# radosgw-admin zonegroup set --infile zonegroup.json
```

`zonegroup.json`您创建的JSON文件在哪里。

重要 

默认情况下，`default`区域组`is_master`设置为`true`。如果创建一个新的区域组并想使其成为主区域组，则必须将`default`区域组 `is_master`设置设置为`false`，或删除该`default`区域组。

最后，更新期间：

```text
# radosgw-admin period update --commit
```

**设置区域组地图**

设置区域组映射包括创建一个由一个或多个区域组组成的JSON对象，并`master_zonegroup`为集群设置。区域组映射中的每个区域组都由一个键/值对组成，其中`key`设置等同于`name`单个区域组配置的设置，并且`val`是一个由单个区域组配置组成的JSON对象。

您可能只有一个`is_master`等于的区域组`true`，并且必须将其指定为`master_zonegroup`区域组图的末尾。以下JSON对象是默认区域组映射的示例。

```text
{
    "zonegroups": [
        {
            "key": "90b28698-e7c3-462c-a42d-4aa780d24eda",
            "val": {
                "id": "90b28698-e7c3-462c-a42d-4aa780d24eda",
                "name": "us",
                "api_name": "us",
                "is_master": "true",
                "endpoints": [
                    "http:\/\/rgw1:80"
                ],
                "hostnames": [],
                "hostnames_s3website": [],
                "master_zone": "9248cab2-afe7-43d8-a661-a40bf316665e",
                "zones": [
                    {
                        "id": "9248cab2-afe7-43d8-a661-a40bf316665e",
                        "name": "us-east",
                        "endpoints": [
                            "http:\/\/rgw1"
                        ],
                        "log_meta": "true",
                        "log_data": "true",
                        "bucket_index_max_shards": 0,
                        "read_only": "false"
                    },
                    {
                        "id": "d1024e59-7d28-49d1-8222-af101965a939",
                        "name": "us-west",
                        "endpoints": [
                            "http:\/\/rgw2:80"
                        ],
                        "log_meta": "false",
                        "log_data": "true",
                        "bucket_index_max_shards": 0,
                        "read_only": "false"
                    }
                ],
                "placement_targets": [
                    {
                        "name": "default-placement",
                        "tags": []
                    }
                ],
                "default_placement": "default-placement",
                "realm_id": "ae031368-8715-4e27-9a99-0c9468852cfe"
            }
        }
    ],
    "master_zonegroup": "90b28698-e7c3-462c-a42d-4aa780d24eda",
    "bucket_quota": {
        "enabled": false,
        "max_size_kb": -1,
        "max_objects": -1
    },
    "user_quota": {
        "enabled": false,
        "max_size_kb": -1,
        "max_objects": -1
    }
}
```

要设置区域组映射，请执行以下操作：

```text
# radosgw-admin zonegroup-map set --infile zonegroupmap.json
```

`zonegroupmap.json`您创建的JSON文件在哪里。确保已为区域组映射中指定的区域创建了区域。最后，更新期间。

```text
# radosgw-admin period update --commit
```

#### 区域

Ceph对象网关支持区域的概念。区域定义了一个逻辑组，该逻辑组由一个或多个Ceph对象网关实例组成。

配置区域不同于典型的配置过程，因为并非所有设置最终都存储在Ceph配置文件中。您可以列出区域，获取区域配置并设置区域配置。

**创建区域**

要创建区域，请指定区域名称。如果是主区域，请指定`--master`选项。区域组中只有一个区域可以是主区域。要将区域添加到区域组，请使用区域`--rgw-zonegroup` 组名称指定选项。

```text
# radosgw-admin zone create --rgw-zone=<name> \
                [--zonegroup=<zonegroup-name]\
                [--endpoints=<endpoint>[,<endpoint>] \
                [--master] [--default] \
                --access-key $SYSTEM_ACCESS_KEY --secret $SYSTEM_SECRET_KEY
```

然后，更新期间：

```text
# radosgw-admin period update --commit
```

**删除区域**

要删除区域，请首先将其从区域组中删除。

```text
# radosgw-admin zonegroup remove --zonegroup=<name>\
                                 --zone=<name>
```

然后，更新期间：

```text
# radosgw-admin period update --commit
```

接下来，删除区域。执行以下命令：

```text
# radosgw-admin zone rm --rgw-zone<name>
```

最后，更新期间：

```text
# radosgw-admin period update --commit
```

重要 

在未首先将其从区域组中删除之前，请勿删除该区域。否则，更新期间将失败。

如果已删除区域的池将不会在其他任何地方使用，请考虑删除池。`<del-zone>`在下面的示例中，将其替换为已删除区域的名称。

重要 

仅删除带有前置区域名称的池。删除根池，例如，`.rgw.root`将删除系统的所有配置。

重要 

删除池后，池中的所有数据都将以不可恢复的方式删除。仅在不再需要池内容时才删除池。

```text
# ceph osd pool rm <del-zone>.rgw.control <del-zone>.rgw.control --yes-i-really-really-mean-it
# ceph osd pool rm <del-zone>.rgw.data.root <del-zone>.rgw.data.root --yes-i-really-really-mean-it
# ceph osd pool rm <del-zone>.rgw.gc <del-zone>.rgw.gc --yes-i-really-really-mean-it
# ceph osd pool rm <del-zone>.rgw.log <del-zone>.rgw.log --yes-i-really-really-mean-it
# ceph osd pool rm <del-zone>.rgw.users.uid <del-zone>.rgw.users.uid --yes-i-really-really-mean-it
```

**修改区域**

要修改区域，请指定区域名称和要修改的参数。

```text
# radosgw-admin zone modify [options]
```

哪里`[options]`：

* `--access-key=<key>`
* `--secret/--secret-key=<key>`
* `--master`
* `--default`
* `--endpoints=<list>`

然后，更新期间：

```text
# radosgw-admin period update --commit
```

**列表区域**

作为`root`，要列出集群中的区域，请执行：

```text
# radosgw-admin zone list
```

**获取区域**

作为`root`，要获取区域的配置，请执行：

```text
# radosgw-admin zone get [--rgw-zone=<zone>]
```

该`default`区域如下所示：

```text
{ "domain_root": ".rgw",
  "control_pool": ".rgw.control",
  "gc_pool": ".rgw.gc",
  "log_pool": ".log",
  "intent_log_pool": ".intent-log",
  "usage_log_pool": ".usage",
  "user_keys_pool": ".users",
  "user_email_pool": ".users.email",
  "user_swift_pool": ".users.swift",
  "user_uid_pool": ".users.uid",
  "system_key": { "access_key": "", "secret_key": ""},
  "placement_pools": [
      {  "key": "default-placement",
         "val": { "index_pool": ".rgw.buckets.index",
                  "data_pool": ".rgw.buckets"}
      }
    ]
  }
```

**设置区域**

配置区域涉及指定一系列Ceph对象网关池。为了保持一致，我们建议使用与区域名称相同的池前缀。有关 配置[池](http://docs.ceph.com/docs/master/rados/operations/pools/#pools)的详细信息，请参见 [池](http://docs.ceph.com/docs/master/rados/operations/pools/#pools)。

要设置区域，请创建一个由池组成的JSON对象，然后将该对象保存到文件中（例如`zone.json`）；然后，执行以下命令，将其替换`{zone-name}`为区域名称：

```text
# radosgw-admin zone set --rgw-zone={zone-name} --infile zone.json
```

`zone.json`您创建的JSON文件在哪里。

然后，作为`root`，更新周期：

```text
# radosgw-admin period update --commit
```

**重命名区域**

要重命名区域，请指定区域名称和新的区域名称。

```text
# radosgw-admin zone rename --rgw-zone=<name> --zone-new-name=<name>
```

然后，更新期间：

```text
# radosgw-admin period update --commit
```

#### 区域组和区域设置

配置默认区域组和区域时，池名称包括区域名称。例如：

* `default.rgw.control`

要更改默认值，请在每个`[client.radosgw.{instance-name}]` 实例下的Ceph配置文件中包含以下设置。

| 名称 | 描述 | 类型 | 默认 |
| :--- | :--- | :--- | :--- |
| `rgw_zone` | 网关实例的区域名称。 | 串 | 没有 |
| `rgw_zonegroup` | 网关实例的区域组的名称。 | 串 | 没有 |
| `rgw_zonegroup_root_pool` | 区域组的根池。 | 串 | `.rgw.root` |
| `rgw_zone_root_pool` | 区域的根池。 | 串 | `.rgw.root` |
| `rgw_default_zone_group_info_oid` | 用于存储默认区域组的OID。我们不建议更改此设置。 | 串 | `default.zonegroup` |

