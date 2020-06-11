# ELASTICSEARCH同步模块

## ELASTICSEARCH同步模块

Kraken版本中的新功能。

这个同步模块将其他区域的元数据写入[ElasticSearch](https://github.com/elastic/elasticsearch)。从发光的角度来看，这是我们当前存储在ElasticSearch中的数据字段的json。

```text
{
     "_index" : "rgw-gold-ee5863d6",
     "_type" : "object",
     "_id" : "34137443-8592-48d9-8ca7-160255d52ade.34137.1:object1:null",
     "_score" : 1.0,
     "_source" : {
       "bucket" : "testbucket123",
       "name" : "object1",
       "instance" : "null",
       "versioned_epoch" : 0,
       "owner" : {
         "id" : "user1",
         "display_name" : "user1"
       },
       "permissions" : [
         "user1"
       ],
       "meta" : {
         "size" : 712354,
         "mtime" : "2017-05-04T12:54:16.462Z",
         "etag" : "7ac66c0f148de9519b8bd264312c4d64"
       }
     }
   }
```

### ELASTICSEARCH层类型可配置

* `endpoint`

指定要访问的Elasticsearch服务器端点

* `num_shards` （整数）

数据同步初始化时将为Elasticsearch配置的分片数。请注意，初始化后无法更改。此处的任何更改都需要重建Elasticsearch索引并重新初始化数据同步过程。

* `num_replicas` （整数）

数据同步初始化时将为Elasticsearch配置的副本数。

* `explicit_custom_meta` （正确\|错误）

指定是否将对所有用户自定义元数据编制索引，或者用户是否需要配置（在存储桶级别）应为哪些自定义元数据条目编制索引。默认为假

* `index_buckets_list` （以逗号分隔的字符串列表）

如果为空，则将为所有存储桶建立索引。否则，将仅索引此处指定的存储桶。可以提供存储桶前缀（例如foo \*）或存储桶后缀（例如\* bar）。

* `approved_owners_list` （以逗号分隔的字符串列表）

如果为空，将为所有所有者的存储桶建立索引（受其他限制），否则，将仅对指定所有者拥有的存储桶建立索引。也可以提供后缀和前缀。

* `override_index_path` （串）

如果不为空，则此字符串将用作elasticsearch索引路径。否则，索引路径将在同步初始化时确定并生成。

### 用户元数据查询

Luminous的新版本。

由于ElasticSearch集群现在存储对象元数据，因此对ElasticSearch端点不要公开，只有集群管理员可以访问，这一点很重要。为了将元数据查询公开给最终用户本身，这会带来一个问题，因为我们希望用户仅查询其元数据，而不查询任何其他用户，这将需要ElasticSearch集群以类似于RGW的方式对用户进行身份验证一个问题。

从元数据主区域中的发光RGW开始，现在可以为最终用户请求提供服务。由于RGW本身可以对最终用户请求进行身份验证，因此这可以避免在公共场所公开Elasticsearch端点，还解决了身份验证和授权问题。为此，RGW在存储区api中引入了一个新查询，该查询可以为elasticsearch请求提供服务。所有这些请求都必须发送到元数据主区域。

#### 语法

**得到ELASTICSEARCH查询**

```text
GET /{bucket}?query={query-expr}
```

请求参数：

* max-keys：要返回的最大条目数
* 标记：分页标记

`expression := [(]<arg> <op> <value> [)][<and|or> ...]`

op是以下之一：&lt;，&lt;=，==，&gt; =，&gt;

例如

```text
GET /?query=name==foo
```

将返回用户具有读取权限的所有索引键，并将其命名为“ foo”。

将返回用户具有读取权限的所有索引键，并将其命名为“ foo”。

输出将是XML中的键列表，该列表类似于S3列表存储桶响应。

**配置自定义元数据字段**

定义应为哪些自定义元数据条目建立索引（在指定存储桶下），以及这些键的类型是什么。如果配置了显式自定义元数据索引，则需要这样做，以便rgw将为指定的自定义元数据值建立索引。否则，在索引的元数据键是字符串以外的类型的情况下，则需要它。

```text
POST /{bucket}?mdsearch
x-amz-meta-search: <key [; type]> [, ...]
```

多个元数据字段必须用逗号分隔，对于带有；的字段，可以强制使用一种类型。。当前允许的类型是字符串（默认），整数和日期

例如。如果要将自定义对象元数据x-amz-meta-year索引为int，将x-amz-meta-date索引为date类型，将x-amz-meta-title索引为字符串，则可以

```text
POST /mybooks?mdsearch
x-amz-meta-search: x-amz-meta-year;int, x-amz-meta-release-date;date, x-amz-meta-title;string
```

**删除自定义元数据配置**

删除自定义元数据存储桶配置。

```text
DELETE /<bucket>?mdsearch
```

**获取自定义元数据配置**

检索自定义元数据存储桶配置。

```text
GET /<bucket>?mdsearch
```

