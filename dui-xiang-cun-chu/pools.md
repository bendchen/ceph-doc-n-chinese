# POOLS

## 池

Ceph对象网关使用多个池来满足其各种存储需求，这些池在Zone对象中列出（请参阅参考资料）。将自动创建名称为的单个区域，池名称以开头，但是多[站点配置](https://docs.ceph.com/docs/nautilus/radosgw/multisite)将具有多个区域。`radosgw-admin zone getdefaultdefault.rgw.`

### 调优

当`radosgw`第一次试图在不存在的区域池操作，它会创建的默认值是游泳池 和。这些默认值足以满足某些池的需求，但其他默认值（尤其是其中列出的存储桶索引和数据的默认值 ）将需要进行其他调整。我们建议使用[Ceph放置组的每个池计算器](http://ceph.com/pgcalc/)为这些池计算合适数量的放置组。有关[池](http://docs.ceph.com/docs/master/rados/operations/pools/#pools) 创建的详细信息，请参见 [池](http://docs.ceph.com/docs/master/rados/operations/pools/#pools)。`osd pool default pg numosd pool default pgp numplacement_pools`

### 池命名空间

发光的新版本。

区域专用的池名称遵循命名约定 `{zone-name}.pool-name`。例如，一个名为的区域`us-east`将具有以下池：

* `.rgw.root`
* `us-east.rgw.control`
* `us-east.rgw.meta`
* `us-east.rgw.log`
* `us-east.rgw.buckets.index`
* `us-east.rgw.buckets.data`

区域定义列出了更多的池，但是其中许多池是通过使用rados名称空间进行合并的。例如，以下所有池条目都使用池的名称空间`us-east.rgw.meta`：

```text
"user_keys_pool": "us-east.rgw.meta:users.keys",
"user_email_pool": "us-east.rgw.meta:users.email",
"user_swift_pool": "us-east.rgw.meta:users.swift",
"user_uid_pool": "us-east.rgw.meta:users.uid",
```

