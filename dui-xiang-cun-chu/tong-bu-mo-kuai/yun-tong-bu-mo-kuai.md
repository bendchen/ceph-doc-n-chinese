# 云同步模块

## 云同步模块

Mimic版本中的新功能。

该模块将区域数据同步到远程云服务。同步是单向的；数据不会从远程区域同步回去。该模块的目的是使数据同步到多个云提供商。当前支持的云提供程序是与AWS（S3）兼容的云提供程序。

需要配置远程云对象存储服务的用户凭据。由于许多云服务对每个用户可以创建的存储桶数量施加了限制，因此源对象和存储桶的映射是可配置的。可以为不同的存储桶和存储桶前缀配置不同的目标。请注意，将不会保留源ACL。可以将特定源用户的权限映射到特定目标用户。

由于API的限制，无法保留原始对象的修改时间和ETag。云同步模块将这些作为元数据属性存储在目标对象上。

### 云同步层类型配置

#### 简单配置：

```text
{
  "connection": {
    "access_key": <access>,
    "secret": <secret>,
    "endpoint": <endpoint>,
    "host_style": <path | virtual>,
  },
  "acls": [ { "type": <id | email | uri>,
              "source_id": <source_id>,
              "dest_id": <dest_id> } ... ],
  "target_path": <target_path>,
}
```

#### 非平凡的配置：

```text
{
  "default": {
    "connection": {
        "access_key": <access>,
        "secret": <secret>,
        "endpoint": <endpoint>,
        "host_style" <path | virtual>,
    },
    "acls": [
    {
      "type" : <id | email | uri>,   #  optional, default is id
      "source_id": <id>,
      "dest_id": <id>
    } ... ]
    "target_path": <path> # optional
  },
  "connections": [
      {
        "connection_id": <id>,
        "access_key": <access>,
        "secret": <secret>,
        "endpoint": <endpoint>,
        "host_style" <path | virtual>,  # optional
      } ... ],
  "acl_profiles": [
      {
        "acls_id": <id>, # acl mappings
        "acls": [ {
            "type": <id | email | uri>,
            "source_id": <id>,
            "dest_id": <id>
          } ... ]
      }
  ],
  "profiles": [
      {
       "source_bucket": <source>,
       "connection_id": <connection_id>,
       "acls_id": <mappings_id>,
       "target_path": <dest>,          # optional
      } ... ],
}
```

注意 

平凡的配置可以与非平凡的配置相吻合。

* `connection` （容器）

表示与远程云服务的连接。包括， ，，和。```conection_id`, ``access_keysecretendpointhost_style```

* `access_key` （串）

用于特定连接的远程云访问密钥。

* `secret` （串）

远程云服务的密钥。

* `endpoint` （串）

远程云服务端点的URL。

* `host_style` （路径\|虚拟）

访问远程云端点时要使用的主机样式类型（默认值：）`path`。

* `acls` （数组）

包含的列表`acl_mappings`。

* `acl_mapping` （容器）

每个`acl_mapping`结构包含`type`，`source_id`和`dest_id`。这些将定义将在每个对象上完成的ACL突变。ACL突变允许将源用户ID转换为目标ID。

* `type` （id \|电子邮件\| uri）

ACL类型：`id`定义用户ID，`email`通过电子邮件`uri`定义用户和`uri`（组）定义用户。

* `source_id` （串）

源区域中用户的ID。

* `dest_id` （串）

目标用户的ID。

* `target_path` （串）

一个字符串，定义如何创建目标路径。目标路径指定源对象名称附加到的前缀。目标路径配置可以包括以下任意变量： - `sid`：代表同步实例ID唯一的字符串- `zonegroup`：在zonegroup名字- `zonegroup_id`：在zonegroup ID - `zone`：区域名称- `zone_id`：区域ID - `bucket`：源桶名称- `owner`：源值区拥有者编号

例如： `target_path = rgwx-${zone}-${sid}/${owner}/${bucket}`

* `acl_profiles` （数组）

的数组`acl_profile`。

* `acl_profile` （容器）

每个配置文件都包含`acls_id`代表配置文件的（字符串），以及`acls`包含的列表的数组`acl_mappings`。

* `profiles` （数组）

配置文件列表。每个配置文件都包含以下内容：- `source_bucket`：存储区名称或`*`定义此配置文件的源存储区的存储区前缀（如果以结尾）-- `target_path`如上所定义- `connection_id`：将用于此连接的连接ID profile- `acls_id`：将用于此配置文件的ACL配置文件的ID

#### S3特定的可配置项：

当前，云同步仅适用于与AWS S3兼容的后端。访问这些云服务时，可以使用一些可配置项来调整其行为：

```text
{
  "multipart_sync_threshold": {object_size},
  "multipart_min_part_size": {part_size}
}
```

* `multipart_sync_threshold` （整数）

大小更大的对象将使用分段上传同步到云中。

* `multipart_min_part_size` （整数）

使用分段上传同步对象时要使用的最小分段大小。

#### 如何配置

见[多站点配置](https://docs.ceph.com/docs/nautilus/radosgw/cloud-sync-module/multisite)为如何多站点配置的说明。云同步模块需要创建一个新区域。区域层类型需要定义为`cloud`：

```text
# radosgw-admin zone create --rgw-zonegroup={zone-group-name} \
                            --rgw-zone={zone-name} \
                            --endpoints={http://fqdn}[,{http://fqdn}]
                            --tier-type=cloud
```

然后可以使用以下命令完成层配置

```text
# radosgw-admin zone modify --rgw-zonegroup={zone-group-name} \
                            --rgw-zone={zone-name} \
                            --tier-config={key}={val}[,{key}={val}]
```

该`key`配置中的指定配置变量需要被更新，并且`val`指定其新值。可以使用句点访问嵌套值。例如：

```text
# radosgw-admin zone modify --rgw-zonegroup={zone-group-name} \
                            --rgw-zone={zone-name} \
                            --tier-config=connection.access_key={key},connection.secret={secret}
```

可以通过在方括号中指定要引用的特定条目来访问配置数组条目，并可以使用\[\]添加新的数组条目。索引值为-1引用数组中的最后一个条目。目前，无法在同一命令中创建新条目并再次引用它。例如，为以{prefix}开头的存储桶创建新的配置文件：

```text
# radosgw-admin zone modify --rgw-zonegroup={zone-group-name} \
                            --rgw-zone={zone-name} \
                            --tier-config=profiles[].source_bucket={prefix}'*'

# radosgw-admin zone modify --rgw-zonegroup={zone-group-name} \
                            --rgw-zone={zone-name} \
                            --tier-config=profiles[-1].connection_id={conn_id},profiles[-1].acls_id={acls_id}
```

可以使用删除条目`--tier-config-rm={key}`。

