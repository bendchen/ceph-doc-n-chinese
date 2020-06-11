# RGW多租户

## RGW多租户

J新版本。

多租户功能允许将桶和相同名称的用户同时使用，方法是将它们隔离在所谓的之下`tenants`。例如，这可能对允许Swift API的用户创建具有容易冲突的名称（例如“ test”或“ trove”）的存储桶很有用。

从Jewel发行开始，每个用户和存储桶都位于一个租户之下。为了兼容，提供了一个具有空名称的“旧式”租户。每当在没有显式租户的情况下引用存储桶时，都会使用从执行操作的用户那里获取的隐式租户。由于先前存在的用户位于旧租户下，因此他们继续像以前一样创建和访问存储桶。RADOS中对象的布局以兼容的方式扩展，确保顺利升级到Jewel。

### 管理具有明确租户的用户

租户本身没有任何操作。在管理用户时，它们会根据需要显示和消失。为了创建，修改和删除具有显式租户的用户，可以提供一个附加选项–tenant，或者在radosgw-admin命令的参数中使用语法“ &lt;tenant&gt; $ &lt;user&gt;”。

#### 例子

创建要通过S3访问的用户testx $ tester：

```text
# radosgw-admin --tenant testx --uid tester --display-name "Test User" --access_key TESTER --secret test123 user create
```

创建一个用户testx $ tester以使用Swift进行访问：

```text
# radosgw-admin --tenant testx --uid tester --display-name "Test User" --subuser tester:test --key-type swift --access full user create
# radosgw-admin --subuser 'testx$tester:test' --key-type swift --secret test123
```

注意 

具有显式租户的子用户必须在外壳程序中引用。

租户名称只能包含字母数字字符和下划线。

### 使用显式租户访问存储桶

客户端应用程序访问存储桶时，它始终以特定用户的凭据运行。如上所述，每个用户都属于一个租户。因此，每个操作在其上下文中都有一个隐式租户，如果未明确指定任何租户，则使用该隐式租户。因此，只要所引用的存储桶和所引用的用户属于同一租户，就可以与以前的版本保持完全的兼容性。换句话说，_仅_访问另一个租户的存储桶时，会发生任何异常情况。

用于指定显式租户的扩展名根据所使用的协议和身份验证系统而有所不同。

#### S3 

在S3的情况下，冒号用于分隔租户和存储桶。因此，示例URL为：

```text
https://ep.host.dom/tenant:bucket
```

这是一个简单的Python示例：

|  |  |
| :--- | :--- |


请注意，使用主机名无法提供明确的租户。主机名不能包含冒号或存储桶名称中尚未有效的任何其他分隔符。使用句点会产生歧义的语法。因此，必须使用“ URL中的存储桶”格式。

由于本机S3 API不能处理多租户，而radosgw的实现可以处理，因此在处理签名URL和公共读取ACL时会涉及一些麻烦。

* 一个**标识的URL**不会包含`AWSAccessKeyId`查询参数，从中radosgw能够辨别正确的用户和租户拥有桶。换句话说，生成签名URL的应用程序应该能够仅采用未加前缀的存储桶名称，并生成一个签名URL，该URL本身包含没有租户前缀的存储桶名称。但是，_可以_选择是否包含前缀。

  因此，取决于签名生成时是否传入了租户前缀， 可以通过 或通过 访问属于租户`bar`的容器 中的对象的签名URL 。`foo7188e165c0ae4424ac68ae2e89a05c50http://<host>:<port>/foo/bar?AWSAccessKeyId=b200fb6634c547199e436a0f93c0c46e&Expires=1542890806&Signature=eok6CYQC%2FDwmQQmqvY5jTg6ehXU%3Dhttp://<host>:<port>/7188e165c0ae4424ac68ae2e89a05c50:foo/bar?AWSAccessKeyId=b200fb6634c547199e436a0f93c0c46e&Expires=1542890806&Signature=eok6CYQC%2FDwmQQmqvY5jTg6ehXU%3D`

* 具有**公共读取ACL的**存储桶旨在由HTTP客户端读取，_而不_包含任何允许radosgw识别租户的查询参数。因此，必须始终使用带有租户前缀的存储桶名称来访问公共可读对象。

  因此，如果您`bar`在`foo`属于租户 的容器中的对象上设置了公共读取ACL，则`7188e165c0ae4424ac68ae2e89a05c50`需要通过public URL访问该对象 `http://<host>:<port>/7188e165c0ae4424ac68ae2e89a05c50:foo/bar`。

#### 带有内置身份验证器的

待定-尚未在test\_multen.py中

#### SWIFT WITH KEYSTONE

在默认配置中，尽管本机Swift具有固有的多租户功能，但radosgw并未为Swift API启用多租户功能。这是为了确保使用旧版存储桶（即在radosgw支持多租户之前创建的存储桶）进行设置，这些存储桶保留了其双API功能，可以使用S3或Swift查询和修改。

如果要为Swift启用多租户，特别是如果您的用户仅通过OpenStack Keystone进行身份验证，则应使用以下`ceph.conf` 配置选项启用基于Keystone的多租户：

```text
rgw keystone implicit tenants = true
```

启用此选项后，任何新连接的用户（无论他们使用的是Swift API还是使用Keystone认证的S3）都将提示radosgw创建一个名为的用户`<tenant_id>$<tenant_id`，其中 `<tenant_id>`是Keystone租户（项目）UUID（例如） `7188e165c0ae4424ac68ae2e89a05c50$7188e165c0ae4424ac68ae2e89a05c50`。

每当该用户随后创建Swift容器时，radosgw就会在内部将给定的容器名称转换为 `<tenant_id>/<container_name>`，例如 `7188e165c0ae4424ac68ae2e89a05c50/foo`。这样可以确保，如果有两个或多个不同的租户，都创建了一个名为的容器 `foo`，则radosgw可以通过其租户前缀透明地识别它们。

通过设置 为或，也可以将隐式租户的影响限制为仅适用于swift或s3 。这可能主要适用于以前使用隐式租户和较旧版本的ceph的用户，其中隐式租户仅适用于swift协议。`rgw keystone implicit tenantss3swift`

#### 注意事项和已知问题

请注意，目前无法在其他租户中创建存储桶。从身份验证信息中提取新创建的存储桶的所有者。

