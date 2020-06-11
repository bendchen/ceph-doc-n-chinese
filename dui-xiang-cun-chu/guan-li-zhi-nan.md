# 管理指南

## 管理指南

一旦启动并运行了Ceph对象存储服务，就可以通过用户管理，访问控制，配额和使用情况跟踪等功能来管理该服务。

### 用户管理

Ceph对象存储用户管理是指Ceph对象存储服务的用户（即，不是Ceph存储网关用户的Ceph对象网关）。您必须创建用户，访问密钥和机密，以使最终用户能够与Ceph Object Gateway服务进行交互。

有两种用户类型：

* **用户：**术语“用户”反映了S3界面的用户。
* **子用户：**术语“子用户”反映了Swift界面的用户。子用户与用户关联。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-b4d57ecd6d1bf334f8d70e716c0870738a375d5a.png)

您可以创建，修改，查看，挂起和删除用户和子用户。除了用户和子用户ID外，您还可以为用户添加显示名称和电子邮件地址。您可以指定密钥和机密，或自动生成密钥和机密。在生成或指定密钥时，请注意，用户ID对应于S3密钥类型，子用户ID对应于快速键类型。斯威夫特键也有访问级别`read`，`write`，`readwrite`和`full`。

#### 创建用户

要创建用户（S3界面），请执行以下操作：

```text
radosgw-admin user create --uid={username} --display-name="{display-name}" [--email={email}]
```

例如：

```text
radosgw-admin user create --uid=johndoe --display-name="John Doe" --email=john@example.com
```

```text
{ "user_id": "johndoe",
  "display_name": "John Doe",
  "email": "john@example.com",
  "suspended": 0,
  "max_buckets": 1000,
  "subusers": [],
  "keys": [
        { "user": "johndoe",
          "access_key": "11BS02LGFB6AL6H1ADMW",
          "secret_key": "vzCEkuryfn060dfee4fgQPqFrncKEIkh3ZcdOANY"}],
  "swift_keys": [],
  "caps": [],
  "op_mask": "read, write, delete",
  "default_placement": "",
  "placement_tags": [],
  "bucket_quota": { "enabled": false,
      "max_size_kb": -1,
      "max_objects": -1},
  "user_quota": { "enabled": false,
      "max_size_kb": -1,
      "max_objects": -1},
  "temp_url_keys": []}
```

创建用户还会创建一个`access_key`和`secret_key`条目，以用于任何与S3 API兼容的客户端。

重要 

检查按键输出。有时会`radosgw-admin` 生成JSON转义（`\`）字符，并且某些客户端不知道如何处理JSON转义符。解决方法包括删除JSON逸出字符（`\`），将字符串封装在引号中，重新生成密钥并确保它没有JSON逸出字符或手动指定密钥和秘密。

#### 创建SUBUSER 

要为该用户创建子用户（Swift界面），必须指定用户ID（`--uid={username}`），子用户ID以及该子用户的访问级别。

```text
radosgw-admin subuser create --uid={uid} --subuser={uid} --access=[ read | write | readwrite | full ]
```

例如：

```text
radosgw-admin subuser create --uid=johndoe --subuser=johndoe:swift --access=full
```

注意 

`full`不是`readwrite`，因为它还包括访问控制策略。

```text
{ "user_id": "johndoe",
  "display_name": "John Doe",
  "email": "john@example.com",
  "suspended": 0,
  "max_buckets": 1000,
  "subusers": [
        { "id": "johndoe:swift",
          "permissions": "full-control"}],
  "keys": [
        { "user": "johndoe",
          "access_key": "11BS02LGFB6AL6H1ADMW",
          "secret_key": "vzCEkuryfn060dfee4fgQPqFrncKEIkh3ZcdOANY"}],
  "swift_keys": [],
  "caps": [],
  "op_mask": "read, write, delete",
  "default_placement": "",
  "placement_tags": [],
  "bucket_quota": { "enabled": false,
      "max_size_kb": -1,
      "max_objects": -1},
  "user_quota": { "enabled": false,
      "max_size_kb": -1,
      "max_objects": -1},
  "temp_url_keys": []}
```

#### 获取用户信息

要获取有关用户的信息，必须指定和用户ID（）。`user info--uid={username}`

```text
radosgw-admin user info --uid=johndoe
```

#### 修改用户信息

要修改有关用户的信息，必须指定用户ID（`--uid={username}`）和要修改的属性。典型的修改是对密钥和机密，电子邮件地址，显示名称和访问级别。例如：

```text
radosgw-admin user modify --uid=johndoe --display-name="John E. Doe"
```

要修改子用户值，请指定和子用户ID。例如：`subuser modify`

```text
radosgw-admin subuser modify --subuser=johndoe:swift --access=full
```

#### 用户启用/挂起

创建用户时，默认情况下会启用该用户。但是，您可以暂停用户特权并在以后重新启用它们。要挂起用户，请指定 和用户ID。`user suspend`

```text
radosgw-admin user suspend --uid=johndoe
```

要重新启用暂停的用户，请指定和用户ID。`user enable`

```text
radosgw-admin user enable --uid=johndoe
```

注意 

禁用用户将禁用子用户。

#### 删除用户

删除用户时，该用户和子用户将从系统中删除。但是，您可以根据需要仅删除子用户。要删除用户（和子用户），请指定和用户ID。`user rm`

```text
radosgw-admin user rm --uid=johndoe
```

要仅删除子用户，请指定和子用户ID。`subuser rm`

```text
radosgw-admin subuser rm --subuser=johndoe:swift
```

选项包括：

* **清除数据：**该`--purge-data`选项清除与UID相关的所有数据。
* **清除密钥：**该`--purge-keys`选项清除与UID相关的所有密钥。

#### 删除SUBUSER 

删除子用户时，将删除对Swift界面的访问。用户将保留在系统中。要删除子用户，请指定 和子用户ID。`subuser rm`

```text
radosgw-admin subuser rm --subuser=johndoe:swift
```

选项包括：

* **清除密钥：**该`--purge-keys`选项清除与UID相关的所有密钥。

#### 添加/删除键

用户和子用户都需要密钥来访问S3或Swift界面。为了使用S3，用户需要由访问密钥和秘密密钥组成的密钥对。另一方面，要使用Swift，用户通常需要一个秘密密钥（密码），并将其与关联的用户ID一起使用。您可以创建一个密钥，然后指定或生成访问密钥和/或秘密密钥。您也可以删除密钥。选项包括：

* `--key-type=<type>`指定密钥类型。选项包括：s3，swift
* `--access-key=<key>` 手动指定一个S3访问密钥。
* `--secret-key=<key>` 手动指定S3密钥或Swift密钥。
* `--gen-access-key` 自动生成一个随机的S3访问密钥。
* `--gen-secret` 自动生成一个随机的S3密钥或一个随机的Swift密钥。

如何为用户添加指定的S3密钥对的示例。

```text
radosgw-admin key create --uid=foo --key-type=s3 --access-key fooAccessKey --secret-key fooSecretKey
```

```text
{ "user_id": "foo",
  "rados_uid": 0,
  "display_name": "foo",
  "email": "foo@example.com",
  "suspended": 0,
  "keys": [
    { "user": "foo",
      "access_key": "fooAccessKey",
      "secret_key": "fooSecretKey"}],
}
```

请注意，您可以为一个用户创建多个S3密钥对。

为子用户附加指定的快速秘密密钥。

```text
radosgw-admin key create --subuser=foo:bar --key-type=swift --secret-key barSecret
```

```text
{ "user_id": "foo",
  "rados_uid": 0,
  "display_name": "foo",
  "email": "foo@example.com",
  "suspended": 0,
  "subusers": [
     { "id": "foo:bar",
       "permissions": "full-control"}],
  "swift_keys": [
    { "user": "foo:bar",
      "secret_key": "asfghjghghmgm"}]}
```

请注意，子用户只能具有一个快捷密钥。

如果子用户与S3密钥对关联，则子用户也可以与S3 API一起使用。

```text
radosgw-admin key create --subuser=foo:bar --key-type=s3 --access-key barAccessKey --secret-key barSecretKey
```

```text
{ "user_id": "foo",
  "rados_uid": 0,
  "display_name": "foo",
  "email": "foo@example.com",
  "suspended": 0,
  "subusers": [
     { "id": "foo:bar",
       "permissions": "full-control"}],
  "keys": [
    { "user": "foo:bar",
      "access_key": "barAccessKey",
      "secret_key": "barSecretKey"}],
}
```

要删除S3密钥对，请指定访问密钥。

```text
radosgw-admin key rm --uid=foo --key-type=s3 --access-key=fooAccessKey
```

删除快速密钥。

```text
radosgw-admin key rm -subuser=foo:bar --key-type=swift
```

#### 添加/删除管理员功能

Ceph Storage Cluster提供了一个管理API，使用户可以通过REST API执行管理功能。默认情况下，用户无权访问此API。为了使用户能够行使管理功能，请为用户提供管理功能。

要向用户添加管理功能，请执行以下操作：

```text
radosgw-admin caps add --uid={uid} --caps={caps}
```

您可以向用户，存储桶，元数据和使用情况（使用率）添加读取，写入或所有功能。例如：

```text
--caps="[users|buckets|metadata|usage|zone]=[*|read|write|read, write]"
```

例如：

```text
radosgw-admin caps add --uid=johndoe --caps="users=*;buckets=*"
```

要从用户删除管理功能，请执行以下操作：

```text
radosgw-admin caps rm --uid=johndoe --caps={caps}
```

### 配额管理

通过Ceph Object Gateway，您可以在用户和用户拥有的存储桶上设置配额。配额包括存储桶中的最大对象数和存储桶可容纳的最大存储大小。

* **桶：**该`--bucket`选项可让您为用户拥有的桶指定配额。
* **最大对象数：**该`--max-objects`设置允许您指定最大对象数。负值将禁用此设置。
* **Maximum Size（最大大小）：**此`--max-size`选项允许您以B / K / M / G / T指定配额大小，其中B为默认值。负值将禁用此设置。
* **配额范围：**该`--quota-scope`选项设置配额的范围。选项是`bucket`和`user`。存储桶配额适用于用户拥有的存储桶。用户配额适用于用户。

#### 设置用户配额

在启用配额之前，必须首先设置配额参数。例如：

```text
radosgw-admin quota set --quota-scope=user --uid=<uid> [--max-objects=<num objects>] [--max-size=<max size>]
```

例如：

```text
radosgw-admin quota set --quota-scope=user --uid=johndoe --max-objects=1024 --max-size=1024B
```

num对象和/或max size的负值表示特定的配额属性检查已禁用。

#### 启用/禁用用户配额

设置用户配额后，您可以启用它。例如：

```text
radosgw-admin quota enable --quota-scope=user --uid=<uid>
```

您可以禁用已启用的用户配额。例如：

```text
radosgw-admin quota disable --quota-scope=user --uid=<uid>
```

#### 设置桶配额

桶配额适用于指定所拥有的桶`uid`。它们独立于用户。

```text
radosgw-admin quota set --uid=<uid> --quota-scope=bucket [--max-objects=<num objects>] [--max-size=<max size]
```

num对象和/或max size的负值表示特定的配额属性检查已禁用。

#### 启用/禁用桶配额

设置存储分区配额后，您可以启用它。例如：

```text
radosgw-admin quota enable --quota-scope=bucket --uid=<uid>
```

您可以禁用已启用的存储桶配额。例如：

```text
radosgw-admin quota disable --quota-scope=bucket --uid=<uid>
```

#### 获取配额设置

您可以通过用户信息API访问每个用户的配额设置。要通过CLI界面读取用户配额设置信息，请执行以下操作：

```text
radosgw-admin user info --uid=<uid>
```

#### 更新配额统计

配额统计信息会异步更新。您可以手动更新所有用户和所有存储桶的配额统计信息，以获取最新的配额统计信息。

```text
radosgw-admin user stats --uid=<uid> --sync-stats
```

#### 获取用户使用情况统计信息

要查看用户已消耗了多少配额，请执行以下操作：

```text
radosgw-admin user stats --uid=<uid>
```

注意 

您应该执行该 选项以接收最新数据。`radosgw-admin user stats--sync-stats`

#### 默认配额

您可以在配置中设置默认配额。这些默认值在创建新用户时使用，对现有用户没有影响。如果在config中设置了相关的默认配额，则在新用户上设置该配额，并启用该配额。见， ，，和 在[Ceph的对象网关配置参考](https://docs.ceph.com/docs/nautilus/radosgw/config-ref/)`rgw bucket default quota max objectsrgw bucket default quota max sizergw user default quota max objectsrgw user default quota max size`

#### 配额缓存

配额统计信息缓存在每个RGW实例上。如果有多个实例，则高速缓存可以防止配额得到完美执行，因为每个实例将具有不同的配额视图。控制此操作的选项，以及 。这些值越高，配额操作越有效，但是多个实例将不同步。这些值越低，多个实例将越接近完美的实施。如果所有三个都为0，则将有效禁用配额缓存，并且多个实例将具有完美的配额强制执行。请参阅《[Ceph对象网关配置参考》](https://docs.ceph.com/docs/nautilus/radosgw/config-ref/)`rgw bucket quota ttlrgw user quota bucket sync intervalrgw user quota sync interval`

#### 读/写全局配额

您可以在期间配置中读取和写入全局配额设置。要查看全局配额设置：

```text
radosgw-admin global quota get
```

全球配额设置可以与被操纵 的对口，和 命令。`global quotaquota setquota enablequota disable`

```text
radosgw-admin global quota set --quota-scope bucket --max-objects 1024
radosgw-admin global quota enable --quota-scope bucket
```

注意 

在多站点配置中，如果存在领域和期间，则必须使用来提交对全局配额的更改。如果不存在任何时间段，则必须重新启动rados网关才能使更改生效。`period update --commit`

### 用法

Ceph对象网关记录每个用户的使用情况。您也可以跟踪日期范围内的用户使用情况。

* 添加在\[client.rgw\] ceph.conf的部分，并重新启动radosgw服务。`rgw enable usage log = true`

选项包括：

* **开始日期：**该`--start-date`选项可让您从特定开始日期（**格式** ：）过滤使用情况统计信息`yyyy-mm-dd[HH:MM:SS]`。
* **结束日期：**该`--end-date`选项可让您过滤特定日期之前的使用情况（**格式：）** `yyyy-mm-dd[HH:MM:SS]`。
* **日志条目：**该`--show-log-entries`选项允许您指定是否在使用情况统计信息中包括日志条目（选项：`true`\| `false`）。

注意 

您可以用分钟和秒来指定时间，但是它以1小时的分辨率存储。

#### 显示用法

要显示使用情况统计信息，请指定。要显示特定用户的使用情况，必须指定用户ID。您还可以指定开始日期，结束日期以及是否显示日志条目。：`usage show`

```text
radosgw-admin usage show --uid=johndoe --start-date=2012-03-01 --end-date=2012-04-01
```

您也可以通过省略用户ID来显示所有用户的使用情况信息摘要。

```text
radosgw-admin usage show --show-log-entries=false
```

#### 修剪用法

大量使用后，使用日志可能会开始占用存储空间。您可以为所有用户和特定用户修剪使用日志。您也可以指定修剪操作的日期范围。

```text
radosgw-admin usage trim --start-date=2010-01-01 --end-date=2010-12-31
radosgw-admin usage trim --uid=johndoe
radosgw-admin usage trim --uid=johndoe --end-date=2013-12-31
```

