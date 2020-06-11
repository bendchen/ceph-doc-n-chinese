# 池放置和存储类

## 池放置和存储类

### 放置目标

珠宝的新版本。

放置目标控制哪些[池](https://docs.ceph.com/docs/nautilus/radosgw/pools)与特定存储桶相关联。桶的放置目标在创建时已选择，无法修改。该命令将显示其 。`radosgw-admin bucket statsplacement_rule`

zonegroup配置包含一个放置目标列表，其初始目标名为`default-placement`。然后，区域配置会将每个区域组放置目标名称映射到其本地存储上。该区域放置信息包括`index_pool`存储桶索引的`data_extra_pool`名称，有关不完整分段上传的元数据的`data_pool`名称以及每个存储类的名称。

### 存储类

Nautilus版本中的新功能。

存储类用于自定义对象数据的放置。S3存储桶生命周期规则可以自动化存储类之间的对象转换。

存储类别是根据放置目标定义的。每个zonegroup放置目标都列出了其可用的存储类别，并带有名为的初始类别`STANDARD`。区域配置负责为`data_pool`每个区域组的存储类提供 池名称。

### [区域组/区域配置](https://docs.ceph.com/docs/nautilus/radosgw/placement/#id6)

使用`radosgw-admin`区域组和区域上的命令执行放置配置。

可以通过以下方式查询区域组放置配置：

```text
$ radosgw-admin zonegroup get
{
    "id": "ab01123f-e0df-4f29-9d71-b44888d67cd5",
    "name": "default",
    "api_name": "default",
    ...
    "placement_targets": [
        {
            "name": "default-placement",
            "tags": [],
            "storage_classes": [
                "STANDARD"
            ]
        }
    ],
    "default_placement": "default-placement",
    ...
}
```

可以通过以下方式查询区域放置配置：

```text
$ radosgw-admin zone get
{
    "id": "557cdcee-3aae-4e9e-85c7-2f86f5eddb1f",
    "name": "default",
    "domain_root": "default.rgw.meta:root",
    ...
    "placement_pools": [
        {
            "key": "default-placement",
            "val": {
                "index_pool": "default.rgw.buckets.index",
                "storage_classes": {
                    "STANDARD": {
                        "data_pool": "default.rgw.buckets.data"
                    }
                },
                "data_extra_pool": "default.rgw.buckets.non-ec",
                "index_type": 0
            }
        }
    ],
    ...
}
```

注意 

如果您之前未进行任何多站点[配置](https://docs.ceph.com/docs/nautilus/radosgw/multisite)，`default`则会为您创建一个区域和区域组，并且在重新启动Ceph对象网关之前，对区域/区域组的更改将不会生效。如果您已经为多站点创建了一个领域，则使用提交更改后，区域/区域组更改将生效。`radosgw-admin period update --commit`

#### [添加放置目标](https://docs.ceph.com/docs/nautilus/radosgw/placement/#id7)

要创建名为的新放置目标`temporary`，请先将其添加到zonegroup中：

```text
$ radosgw-admin zonegroup placement add \
      --rgw-zonegroup default \
      --placement-id temporary
```

然后提供该目标的区域放置信息：

```text
$ radosgw-admin zone placement add \
      --rgw-zone default \
      --placement-id temporary \
      --data-pool default.rgw.temporary.data \
      --index-pool default.rgw.temporary.index \
      --data-extra-pool default.rgw.temporary.non-ec
```

#### [添加存储类](https://docs.ceph.com/docs/nautilus/radosgw/placement/#id8)

要将新的存储类添加`COLD`到`default-placement`目标，请先将其添加到zonegroup中：

```text
$ radosgw-admin zonegroup placement add \
      --rgw-zonegroup default \
      --placement-id default-placement \
      --storage-class COLD
```

然后提供该存储类的区域放置信息：

```text
$ radosgw-admin zone placement add \
      --rgw-zone default \
      --placement-id default-placement \
      --storage-class COLD \
      --data-pool default.rgw.cold.data \
      --compression lz4
```

### [自定义布局](https://docs.ceph.com/docs/nautilus/radosgw/placement/#id9)

#### [默认放置](https://docs.ceph.com/docs/nautilus/radosgw/placement/#id10)

默认情况下，新存储桶将使用区域组的`default_placement`目标。可以通过以下方式更改此区域组设置：

```text
$ radosgw-admin zonegroup placement default \
      --rgw-zonegroup default \
      --placement-id new-placement
```

#### [用户放置](https://docs.ceph.com/docs/nautilus/radosgw/placement/#id11)

Ceph对象网关用户可以通过`default_placement`在用户信息中设置非空字段来覆盖区域组的默认放置目标。同样， 默认情况下，`default_storage_class`可以覆盖`STANDARD`应用于对象的存储类。

```text
$ radosgw-admin user info --uid testid
{
    ...
    "default_placement": "",
    "default_storage_class": "",
    "placement_tags": [],
    ...
}
```

如果区域组的放置目标包含`tags`，则用户将无法使用该放置目标创建存储桶，除非其用户信息在其`placement_tags`字段中至少包含一个匹配标签。这对于限制对某些类型的存储的访问很有用。

该`radosgw-admin`命令无法直接修改这些字段，因此必须手动编辑json格式：

```text
$ radosgw-admin metadata get user:<user-id> > user.json
$ vi user.json
$ radosgw-admin metadata put user:<user-id> < user.json
```

#### [S3存储桶放置](https://docs.ceph.com/docs/nautilus/radosgw/placement/#id12)

使用S3协议创建存储区时，可以提供放置目标作为LocationConstraint的一部分，以覆盖用户和区域组中的默认放置目标。

通常，LocationConstraint必须与区域组的匹配`api_name`：

```text
<LocationConstraint>default</LocationConstraint>
```

可以将自定义展示位置目标添加到`api_name`以下冒号中：

```text
<LocationConstraint>default:new-placement</LocationConstraint>
```

#### [SWIFT BUCKET放置](https://docs.ceph.com/docs/nautilus/radosgw/placement/#id13)

使用Swift协议创建存储桶时，可以在HTTP标头中提供放置目标`X-Storage-Policy`：

```text
X-Storage-Policy: new-placement
```

### [使用存储类](https://docs.ceph.com/docs/nautilus/radosgw/placement/#id14)

所有放置目标都有一个`STANDARD`存储类，默认情况下该存储类将应用于新对象。用户可以使用覆盖此默认设置 `default_storage_class`。

要在非默认存储类中创建对象，请在请求的HTTP标头中提供该存储类名称。S3协议使用 `X-Amz-Storage-Class`标头，而Swift协议使用 `X-Object-Storage-Class`标头。

然后，可以使用操作使用S3对象生命周期管理在存储类之间移动对象数据`Transition`。

