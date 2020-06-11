# 压缩

## 压缩

Kraken版本中的新功能。

Ceph对象网关支持使用Ceph现有的任何压缩插件在服务器端对上传的对象进行压缩。

### 配置

通过为`--compression=<type>`命令提供选项，可以在 区域的放置目标中的存储类上启用压缩。`radosgw-admin zone placement modify`

压缩`type`是指在写入新对象数据时要使用的压缩插件的名称。每个压缩的对象都会记住使用了哪个插件，因此更改此设置不会影响解压缩现有对象的能力，也不会迫使现有对象被重新压缩。

此压缩设置适用于使用此放置目标上传到存储桶的所有新对象。可以通过将设置`type`为空字符串或来禁用压缩`none`。

例如：

```text
$ radosgw-admin zone placement modify \
      --rgw-zone default \
      --placement-id default-placement \
      --storage-class STANDARD \
      --compression zlib
{
...
    "placement_pools": [
        {
            "key": "default-placement",
            "val": {
                "index_pool": "default.rgw.buckets.index",
                "storage_classes": {
                    "STANDARD": {
                        "data_pool": "default.rgw.buckets.data",
                        "compression_type": "zlib"
                    }
                },
                "data_extra_pool": "default.rgw.buckets.non-ec",
                "index_type": 0,
            }
        }
    ],
...
}
```

注意 

`default`如果您之前未进行任何多[站点配置，](https://docs.ceph.com/docs/nautilus/radosgw/multisite)则会为您创建一个区域。

### 统计

尽管所有现有命令和API都会继续根据其未压缩数据报告对象和存储桶的大小，但给定存储桶的压缩统计信息包括在其中：`bucket stats`

```text
$ radosgw-admin bucket stats --bucket=<name>
{
...
    "usage": {
        "rgw.main": {
            "size": 1075028,
            "size_actual": 1331200,
            "size_utilized": 592035,
            "size_kb": 1050,
            "size_kb_actual": 1300,
            "size_kb_utilized": 579,
            "num_objects": 104
        }
    },
...
}
```

的`size_utilized`和`size_kb_utilized`字段分别表示压缩数据的总大小，以字节为单位和字节。

