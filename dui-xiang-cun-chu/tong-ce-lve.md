# 桶策略

## 桶策略

Luminous新版本。

Ceph对象网关支持应用于存储桶的Amazon S3策略语言的子集。

### 创建和删除

桶策略是通过标准S3操作而不是radosgw-admin管理的。

例如，可以使用s3cmd设置或删除策略，因此：

```text
$ cat > examplepol
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {"AWS": ["arn:aws:iam::usfolks:user/fred:subuser"]},
    "Action": "s3:PutObjectAcl",
    "Resource": [
      "arn:aws:s3:::happybucket/*"
    ]
  }]
}

$ s3cmd setpolicy examplepol s3://happybucket
$ s3cmd delpolicy s3://happybucket
```

### 局限性

目前，我们仅支持以下操作：

* s3：AbortMultipartUpload
* s3：CreateBucket
* s3：DeleteBucketPolicy
* s3：DeleteBucket
* s3：DeleteBucket网站
* s3：DeleteObject
* s3：DeleteObjectVersion
* s3：DeleteReplicationConfiguration
* s3：GetAccelerateConfiguration
* s3：GetBucketAcl
* s3：GetBucketCORS
* s3：GetBucketLocation
* s3：GetBucketLogging
* s3：GetBucketNotification
* s3：GetBucketPolicy
* s3：GetBucketRequestPayment
* s3：GetBucketTagging
* s3：GetBucketVersioning
* s3：GetBucketWebsite
* s3：GetLifecycleConfiguration
* s3：GetObjectAcl
* s3：GetObject
* s3：GetObjectTorrent
* s3：GetObjectVersionAcl
* s3：GetObjectVersion
* s3：GetObjectVersionTorrent
* s3：GetReplicationConfiguration
* s3：ListAllMyBuckets
* s3：ListBucketMultiPartUploads
* s3：ListBucket
* s3：ListBucketVersions
* s3：ListMultipartUploadParts
* s3：PutAccelerateConfiguration
* s3：PutBucketAcl
* s3：PutBucketCORS
* s3：PutBucketLogging
* s3：PutBucketNotification
* s3：PutBucketPolicy
* s3：PutBucketRequestPayment
* s3：PutBucketTagging
* s3：PutBucketVersioning
* s3：PutBucket网站
* s3：PutLifecycleConfiguration
* s3：PutObjectAcl
* s3：PutObject
* s3：PutObjectVersionAcl
* s3：PutReplicationConfiguration
* s3：RestoreObject

我们尚不支持针对用户，组或角色设置策略。

我们使用RGW的“租户”标识符代替亚马逊的十二位数帐户ID。将来，我们可能会允许您为租户分配账户ID，但是现在，如果您想在AWS S3和RGW S3之间使用策略，则在创建用户时必须使用Amazon账户ID作为租户ID。

在AWS下，所有租户共享一个名称空间。RGW为每个租户提供其自己的存储桶名称空间。在将来的版本中，可能会有一个选项来启用类似AWS的“扁平”存储空间名称空间。目前，要访问属于另一个租户的存储桶，请在S3请求中将其命名为“ tenant：bucket”。

在AWS中，存储桶策略可以授予对另一个帐户的访问权限，然后该帐户所有者可以向具有用户权限的单个用户授予访问权限。由于我们尚不支持用户，角色和组权限，因此帐户所有者当前需要直接向单个用户授予访问权限，向整个帐户授予存储桶访问权限将向该帐户中的所有用户授予访问权限。

存储桶策略尚不支持字符串插值。

对于所有请求，我们支持的条件键为：-aws：CurrentTime-aws：EpochTime-aws：PrincipalType-aws：Referer-aws：SecureTransport-aws：SourceIp-aws：UserAgent-aws：username

我们为存储桶和对象请求支持某些s3条件键。

Mimic版本中的新功能。

#### BUCKET RELATED OPERATIONS

<table>
  <thead>
    <tr>
      <th style="text-align:left">Permission</th>
      <th style="text-align:left">Condition Keys</th>
      <th style="text-align:left">Comments</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">s3:createBucket</td>
      <td style="text-align:left">s3:x-amz-acl s3:x-amz-grant-&lt;perm&gt; where perm is one of read/write/read-acp
        write-acp/ full-control</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>s3:ListBucket &amp;</p>
        <p>s3:ListBucketVersions</p>
      </td>
      <td style="text-align:left">s3:prefix</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">s3:delimiter</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">s3:max-keys</td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">s3:PutBucketAcl</td>
      <td style="text-align:left">s3:x-amz-acl s3:x-amz-grant-&lt;perm&gt;</td>
      <td style="text-align:left"></td>
    </tr>
  </tbody>
</table>

#### OBJECT RELATED OPERATIONS

| Permission | Condition Keys | Comments |
| :--- | :--- | :--- |
| s3:PutObject | s3:x-amz-acl & s3:x-amz-grant-&lt;perm&gt; |  |
| s3:x-amz-copy-source |  |  |
| s3:x-amz-server-side-encryption |  |  |
| s3:x-amz-server-side-encryption-aws-kms-key-id |  |  |
| s3:x-amz-metadata-directive | PUT & COPY to overwrite/preserve metadata in COPY requests |  |
| s3:RequestObjectTag/&lt;tag-key&gt; |  |  |
| s3:PutObjectAcl s3:PutObjectVersionAcl | s3:x-amz-acl & s3-amz-grant-&lt;perm&gt; |  |
| s3:ExistingObjectTag/&lt;tag-key&gt; |  |  |
| s3:PutObjectTagging & s3:PutObjectVersionTagging | s3:RequestObjectTag/&lt;tag-key&gt; |  |
| s3:ExistingObjectTag/&lt;tag-key&gt; |  |  |
| s3:GetObject & s3:GetObjectVersion | s3:ExistingObjectTag/&lt;tag-key&gt; |  |
| s3:GetObjectAcl & s3:GetObjectVersionAcl | s3:ExistingObjectTag/&lt;tag-key&gt; |  |
| s3:GetObjectTagging & s3:GetObjectVersionTagging | s3:ExistingObjectTag/&lt;tag-key&gt; |  |
| s3:DeleteObjectTagging & s3:DeleteObjectVersionTagging | s3:ExistingOBjectTag/&lt;tag-key&gt; |  |

我们与最近重写的Authentication / Authorization子系统集成后，可能会很快支持更多功能。

### SWIFT

无法在Swift下设置存储桶策略，但是已设置的存储桶策略将控制Swift和S3操作。

Swift凭证以特定于所使用后端的方式与策略中指定的主体匹配。

