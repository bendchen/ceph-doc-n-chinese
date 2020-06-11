# 存档同步模块

## 存档同步模块

Nautilus版本中的新功能。

该同步模块利用RGW中S3对象的版本控制功能来拥有一个存档区域，该存档区域捕获随着时间的推移在其他区域中出现的S3对象的不同版本。

存档区域允许具有S3对象版本的历史记录，这些历史记录只能通过与存档区域关联的网关来消除。

此功能对于以下配置很有用：几个非版本区域通过其区域网关复制其数据和元数据（镜像配置），从而为最终用户提供高可用性，而归档区域捕获所有数据更新和元数据以将其合并为S3对象的版本。

在多区域配置中包括一个存档区域，使您可以在一个区域中拥有S3对象历史记录的灵活性，同时节省了版本化的S3对象的副本将在其余区域中占用的空间。

### 存档同步层类型配置

#### 如何配置

见[多站点配置](https://docs.ceph.com/docs/nautilus/radosgw/archive-sync-module/multisite)为如何多站点配置的说明。存档同步模块需要创建一个新区域。区域层类型需要定义为`archive`：

```text
# radosgw-admin zone create --rgw-zonegroup={zone-group-name} \
                            --rgw-zone={zone-name} \
                            --endpoints={http://fqdn}[,{http://fqdn}]
                            --tier-type=archive
```

