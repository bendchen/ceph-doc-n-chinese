# STS lite

## STS 

Ceph对象网关为Amazon Secure Token Service（STS）API的子集提供支持。STS Lite提供对身份和访问管理的一组临时凭据的访问。

STS身份验证机制已与Ceph Object Gateway中的Keystone集成在一起。使用Keystone对一组AWS凭证进行身份验证后，将返回一组临时安全凭证。这些临时凭据可用于进行后续的S3调用，这些调用将由STS引擎进行身份验证，从而减少Keystone服务器上的负载。

### STS LITE REST 

在Ceph Object Gateway中实现了以下STS Lite REST API：

1. GetSessionToken：为一组AWS凭证返回一组临时凭证。该API可用于Keystone的初始身份验证，返回的临时凭据可用于进行后续的S3调用。临时凭证将具有与AWS凭证相同的权限。参数：

**DurationSeconds**（整数/可选）：凭据应保持有效的持续时间（以秒为单位）。其默认值为3600。其默认最大值为43200，可以使用rgw sts max会话持续时间进行配置。

**SerialNumber**（字符串/可选）：与进行GetSessionToken调用的用户相关联的MFA设备的ID号。

**TokenCode**（字符串/可选）：如果需要MFA，则由MFA设备提供的值。

最终用户需要附加一个策略，以允许使用其永久凭证来调用GetSessionToken API，并允许仅使用由GetSessionToken返回的临时凭证来进行后续的s3操作调用。以下是将策略附加到用户“ TESTER1”的示例：

```text
s3curl.pl --debug --id admin -- -s -v -X POST "http://localhost:8000/?Action=PutUserPolicy&PolicyName=Policy1&UserName=TESTER1&PolicyDocument=\{\"Version\":\"2012-10-17\",\"Statement\":\[\{\"Effect\":\"Deny\",\"Action\":\"s3:*\",\"Resource\":\[\"*\"\],\"Condition\":\{\"BoolIfExists\":\{\"sts:authentication\":\"false\"\}\}\},\{\"Effect\":\"Allow\",\"Action\":\"sts:GetSessionToken\",\"Resource\":\"*\",\"Condition\":\{\"BoolIfExists\":\{\"sts:authentication\":\"false\"\}\}\}\]\}&Version=2010-05-08"
```

附加策略的用户需要具有管理员权限。例如：

```text
radosgw-admin caps add --uid="TESTER" --caps="user-policy=*"
```

2. AssumeRole：返回一组临时凭证，可用于跨帐户访问。临时凭证将具有两个角色都允许的权限-角色附带的权限策略和AssumeRole API附带的策略。参数：

**RoleArn**（字符串/必需）：要承担的角色的ARN。

**RoleSessionName**（字符串/必需）：假定角色会话的标识符。

**策略**（字符串/可选）：JSON格式的IAM策略。

**DurationSeconds**（整数/可选）：会话的持续时间（以秒为单位）。其默认值为3600。

**ExternalId**（字符串/可选）：在另一个帐户中担任角色时可能使用的唯一ID。

**SerialNumber**（字符串/可选）：与进行AssumeRole调用的用户相关联的MFA设备的ID号。

**令牌代码**（字符串/可选）：如果假定角色的信任策略要求MFA，则MFA设备提供的值。

### STS LITE配置

以下可配置选项可用于STS Lite集成：

```text
[client.radosgw.gateway]
rgw sts key = {sts key for encrypting the session token}
rgw s3 auth use sts = true
```

如果需要将STS Lite与Keystone结合使用，则上述STS可配置项可与Keystone可配置项一起使用。完整的可配置选项集为：

```text
[client.radosgw.gateway]
rgw sts key = {sts key for encrypting/ decrypting the session token}
rgw s3 auth use sts = true

rgw keystone url = {keystone server url:keystone server admin port}
rgw keystone admin project = {keystone admin project name}
rgw keystone admin tenant = {keystone service tenant name}
rgw keystone admin domain = {keystone admin domain name}
rgw keystone api version = {keystone api version}
rgw keystone implicit tenants = {true for private tenant for each new user}
rgw keystone admin password = {keystone service tenant user name}
rgw keystone admin user = keystone service tenant user password}
rgw keystone accepted roles = {accepted user roles}
rgw keystone token cache size = {number of tokens to cache}
rgw keystone revocation interval = {number of seconds before checking revoked tickets}
rgw s3 auth use keystone = true
rgw nss db path = {path to nss db}
```

注意：默认情况下，STS和S3 API共存于同一名称空间中，并且S3和STS API均可通过Ceph Object Gateway中的同一端点进行访问。

### 显示如何将KEYSTONE和STS LITE一起使用的示例

以下是将STS Lite与Keystone结合使用所需的步骤。Boto 3.x已被用来编写示例代码，以显示STS Lite与Keystone的集成。

1. 生成EC2凭证：

```text
openstack ec2 credentials create
+------------+--------------------------------------------------------+
| Field      | Value                                                  |
+------------+--------------------------------------------------------+
| access     | b924dfc87d454d15896691182fdeb0ef                       |
| links      | {u'self': u'http://192.168.0.15/identity/v3/users/     |
|            | 40a7140e424f493d8165abc652dc731c/credentials/          |
|            | OS-EC2/b924dfc87d454d15896691182fdeb0ef'}              |
| project_id | c703801dccaf4a0aaa39bec8c481e25a                       |
| secret     | 6a2142613c504c42a94ba2b82147dc28                       |
| trust_id   | None                                                   |
| user_id    | 40a7140e424f493d8165abc652dc731c                       |
+------------+--------------------------------------------------------+
```

1. 使用在步骤1中创建的凭据，使用GetSessionToken API来获取一组临时凭据。

```text
import boto3

access_key = <ec2 access key>
secret_key = <ec2 secret key>

client = boto3.client('sts',
aws_access_key_id=access_key,
aws_secret_access_key=secret_key,
endpoint_url=<STS URL>,
region_name='',
)

response = client.get_session_token(
    DurationSeconds=43200
)
```

1. 在步骤2中获得的临时证书可用于进行S3调用：

```text
s3client = boto3.client('s3',
  aws_access_key_id = response['Credentials']['AccessKeyId'],
  aws_secret_access_key = response['Credentials']['SecretAccessKey'],
  aws_session_token = response['Credentials']['SessionToken'],
  endpoint_url=<S3 URL>,
  region_name='')

bucket = s3client.create_bucket(Bucket='my-new-shiny-bucket')
response = s3client.list_buckets()
for bucket in response["Buckets"]:
    print "{name}\t{created}".format(
                name = bucket['Name'],
                created = bucket['CreationDate'],
)
```

1. 以下是AssumeRole API调用的示例：

```text
import boto3

access_key = <ec2 access key>
secret_key = <ec2 secret key>

client = boto3.client('sts',
aws_access_key_id=access_key,
aws_secret_access_key=secret_key,
endpoint_url=<STS URL>,
region_name='',
)

response = client.assume_role(
RoleArn='arn:aws:iam:::role/application_abc/component_xyz/S3Access',
RoleSessionName='Bob',
DurationSeconds=3600
)
```

注意：在调用AssumeRole API之前，需要创建角色“ S3Access”。

### 限制和解决方法

1. Keystone当前仅支持S3请求，因此，为了成功验证STS请求，需要将以下变通办法添加到boto的以下文件中-botocore / auth.py

下面的代码块中添加了第13-16行作为解决方法：

```text
class SigV4Auth(BaseSigner):
  """
  Sign a request with Signature V4.
  """
  REQUIRES_REGION = True

  def __init__(self, credentials, service_name, region_name):
      self.credentials = credentials
      # We initialize these value here so the unit tests can have
      # valid values.  But these will get overriden in ``add_auth``
      # later for real requests.
      self._region_name = region_name
      if service_name == 'sts':
          self._service_name = 's3'
      else:
          self._service_name = service_name
```

