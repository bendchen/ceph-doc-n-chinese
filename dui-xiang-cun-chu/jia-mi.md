# 加密

## 加密

Luminous的新版本。

Ceph对象网关支持上传对象的服务器端加密，并带有3个用于管理加密密钥的选项。服务器端加密意味着数据以未加密形式通过HTTP发送，并且Ceph对象网关以加密形式将该数据存储在Ceph存储集群中。

注意 

必须通过安全的HTTPS连接发送对服务器端加密的请求，以避免以明文形式发送秘密。如果将代理用于SSL终止，则必须先启用代理，然后转发的请求才能被视为安全。`rgw trust forwarded https`

### 客户提供的密钥

在这种模式下，客户端将加密密钥与每个读取或写入加密数据的请求一起传递。客户有责任管理这些密钥，并记住用于加密每个对象的密钥。

这是根据[Amazon SSE-C](https://docs.aws.amazon.com/AmazonS3/latest/dev/ServerSideEncryptionCustomerKeys.html)规范在S3中实现的。

由于所有密钥管理均由客户端处理，因此无需特殊配置即可支持此加密模式。

### 密钥管理服务

此模式允许将密钥存储在安全密钥管理服务中，并由Ceph对象网关按需进行检索，以处理加密或解密数据的请求。

这是根据[Amazon SSE-KMS](http://docs.aws.amazon.com/AmazonS3/latest/dev/UsingKMSEncryption.html)规范在S3中实现的。

原则上，这里可以使用任何密钥管理服务，但目前仅实现与[Barbican的](https://wiki.openstack.org/wiki/Barbican)集成。

请参阅[OpenStack Barbican集成](https://docs.ceph.com/docs/nautilus/radosgw/barbican)。

### 自动加密（仅用于测试）

可以在ceph.conf中设置A 来强制对所有未指定加密模式的对象进行加密。`rgw crypt default encryption key`

该配置要求使用base64编码的256位密钥。例如：

```text
rgw crypt default encryption key = 4YSmvJtBv0aZ7geVgAsdpRnLBEwWSWlMIGnRS8a9TSA=
```

重要 

此模式仅用于诊断目的！ceph配置文件不是用于存储加密密钥的安全方法。以这种方式意外暴露的密钥应视为已泄露。

