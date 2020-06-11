# 同步模块

## 同步模块

Kraken版本中的新功能。

Jewel中引入的RGW 的多[站点](https://docs.ceph.com/docs/nautilus/radosgw/multisite)功能允许创建多个区域并在它们之间镜像数据和元数据。 在多站点框架之上构建，该框架允许将数据和元数据转发到不同的外部层。同步模块允许在数据发生更改时执行一组操作（元数据操作（例如存储桶或用户创建等）也被视为数据更改）。由于rgw多站点更改最终在远程站点上是一致的，因此更改将异步传播。这将允许解锁用例，例如将对象存储备份到外部云集群或使用磁带驱动器的自定义备份解决方案，在ElasticSearch中索引元数据等。`Sync Modules`

同步模块配置在区域本地。同步模块确定区域是导出数据还是只能使用在另一个区域中修改的数据。从发光的角度来看，支持的同步插件是[elasticsearch](https://docs.ceph.com/docs/nautilus/radosgw/elastic-sync-module)， `rgw`它是在区域之间同步数据的默认同步插件，并且`log`是记录在远程区域中发生的元数据操作的简单同步插件。以下文档以使用[Elasticsearch同步模块](https://docs.ceph.com/docs/nautilus/radosgw/elastic-sync-module)的区域示例为例编写，配置任何同步插件的过程将相似

* [ElasticSearch同步模块](https://docs.ceph.com/docs/nautilus/radosgw/elastic-sync-module/)
* [云同步模块](https://docs.ceph.com/docs/nautilus/radosgw/cloud-sync-module/)
* [PubSub模块](https://docs.ceph.com/docs/nautilus/radosgw/pubsub-module/)
* [存档同步模块](https://docs.ceph.com/docs/nautilus/radosgw/archive-sync-module/)

### 要求和假设

让我们假设如在描述了一个简单的多站点配置[多站点](https://docs.ceph.com/docs/nautilus/radosgw/multisite) 文档的2个区，`us-east`以及`us-west`，让我们添加一个第三区 `us-east-es`是一个区域，只有从其他网站处理元数据。该区域可以与处于相同或不同的ceph群集中`us-east`。该区域将仅消耗其他区域的元数据，并且该区域中的RGW将不直接服务于任何最终用户请求。

### 配置同步模块

例如，创建类似于“多[站点”](https://docs.ceph.com/docs/nautilus/radosgw/multisite)文档的第三个区域

```text
# radosgw-admin zone create --rgw-zonegroup=us --rgw-zone=us-east-es \
--access-key={system-key} --secret={secret} --endpoints=http://rgw-es:80
```

可以通过以下方式为此区域配置一个同步模块

```text
# radosgw-admin zone modify --rgw-zone={zone-name} --tier-type={tier-type} --tier-config={set of key=value pairs}
```

例如在`elasticsearch`同步模块中

```text
# radosgw-admin zone modify --rgw-zone={zone-name} --tier-type=elasticsearch \
                            --tier-config=endpoint=http://localhost:9200,num_shards=10,num_replicas=1
```

有关各种受支持的tier-config选项，请参阅[elasticsearch同步模块](https://docs.ceph.com/docs/nautilus/radosgw/elastic-sync-module)文档

最后更新期间

```text
# radosgw-admin period update --commit
```

现在在区域中启动radosgw

```text
# systemctl start ceph-radosgw@rgw.`hostname -s`
# systemctl enable ceph-radosgw@rgw.`hostname -s`
```

