# RADOS网关数据布局

尽管源代码是最终指南，但本文档可帮助新开发人员快速掌握实施细节。

### 简介

Swift提供了一种称为容器的容器，我们可以将其与“桶”一词互换使用。可以说RGW的存储桶实现了Swift容器。

本文档未考虑RGW如何在这些结构上运行，例如，如何使用encode（）和encode（）方法进行序列化等等。

### 概念视图

尽管RADOS只知道具有xattrs和omap \[1\]的池和对象，但从概念上讲，RGW将其数据分为三种不同的类型：元数据，存储桶索引和数据。

#### 元数据

我们具有元数据的3个“部分”：“用户”，“存储桶”和“ bucket.instance”。您可以使用以下命令来内省元数据条目：

```text
$ radosgw-admin metadata list
$ radosgw-admin metadata list bucket
$ radosgw-admin metadata list bucket.instance
$ radosgw-admin metadata list user

$ radosgw-admin metadata get bucket:<bucket>
$ radosgw-admin metadata get bucket.instance:<bucket>:<bucket_id>
$ radosgw-admin metadata get user:<user>   # get or set
```

上面的命令中使用了一些变量，它们是：

* 用户：保存用户信息
* bucket：保存存储桶名称和存储桶实例ID之间的映射
* bucket.instance：保存桶实例信息\[2\]

每个元数据条目都保存在单个rados对象上。有关实现失败的信息，请参见下文。

请注意，元数据未编制索引。当列出元数据部分时，我们在包含池中执行rados pgls操作。

#### 斗指数

它是另一种元数据，并分别保存。bucket索引在rados对象中保存键-值映射。默认情况下，每个存储桶只有一个rados对象，但是由于Hammer可以分片映射到多个rados对象。地图本身保存在与每个rados对象相关联的omap中。每个omap的键都是对象的名称，该值包含该对象的一些基本元数据-列出存储桶时显示的元数据。另外，每个omap都有一个标头，并且我们在该标头中保留了一些存储段记帐元数据（对象数，总大小等）。

请注意，我们还在存储区索引中保存了其他信息，并将其保存在其他关键名称空间中。我们可以在其中保存存储桶索引日志，对于版本化的对象，我们还有其他键上保留的更多信息。

#### 数据

对象数据被保存在每个rgw对象的一个​​或多个rados对象中。

### 对象查找路径

当访问对象时，ReST API会通过三个参数进入RGW：帐户信息（S3中的访问密钥或Swift中的帐户名称），存储桶或容器名称以及对象名称（或密钥）。目前，RGW仅使用帐户信息来查找用户ID和进行访问控制。仅存储区名称和对象键用于寻址池中的对象。

RGW中的用户ID是一个字符串，通常是用户凭据中的实际用户名，而不是哈希或映射的标识符。

访问用户数据时，将从具有名称空间“ users.uid”的池“ default.rgw.meta”中的对象“ &lt;user\_id&gt;”加载用户记录。

存储区名称在池“ default.rgw.meta”中以命名空间“ root”表示。加载存储桶记录以获得所谓的标记，该标记用作存储桶ID。

该对象位于池“ default.rgw.buckets.data”中。对象名称是“ &lt;marker&gt; \_ &lt;key&gt;”，例如“ default.7593.4\_image.png”，其中标记是“ default.7593.4”，键是“ image.png”。由于不解析这些串联的名称，仅传递给RADOS，因此分隔符的选择并不重要，也不会造成歧义。出于相同的原因，对象名称（键）中允许使用斜线。

也可以创建多个数据池并使其成为默认数据库，从而在不同的rados池中创建不同的用户存储桶，从而提供必要的扩展。这些池的布局和命名由“策略”设置控制。\[3\]

一个RGW对象可能由几个RADOS对象组成，第一个对象是包含元数据的头，例如清单，ACL，内容类型，ETag和用户定义的元数据。元数据存储在xattrs中。为了提高效率和原子性，磁头还可能包含多达512 KB的对象数据。清单描述了如何在RADOS对象中布置每个对象。

### 桶和对象清单

在名称空间为“ users.uid”的池“ default.rgw.meta”中名为“ &lt;user\_id&gt; .buckets”（例如“ foo.buckets”）的对象的omap中列出了属于给定用户的存储桶。在列出存储桶，更新存储桶内容以及更新和检索存储桶统计信息（例如用于配额）时，将访问这些对象。

有关这些omap整体的值，请参见用户可见的编码类'cls\_user\_bucket\_entry'及其嵌套类'cls\_user\_bucket'。

这些列表与“ .rgw”池中的存储桶保持一致。

桶索引中列出了属于给定桶的对象，如上面“桶索引”小节所述。索引对象的默认命名为“ default.rgw.buckets.index”池中的“ .dir。&lt;marker&gt;”。

### 脚注

\[1\] Omap是与对象关联的键值存储，其方式类似于扩展属性与POSIX文件的关联方式。对象的omap不在物理上位于对象的存储中，但它的精确实现对RADOS Gateway不可见且无关紧要。在Hammer中，一个LevelDB用于在每个OSD中存储omap。

\[2\]在Dumpling版本之前，“ bucket.instance”元数据不存在，并且“ bucket”元数据包含其信息。在旧的安装中可能会遇到这种铲斗。

\[3\]从Infernalis版本开始，池名称已更改。如果您使用的是较旧的设置，则某些细节可能会有所不同。特别是，现在在default.root.meta池中使用的每个名称空间都有一个不同的池。

### 附录：纲要

已知池：.rgw.root

未指定的区域，区域和全局信息记录，每个对象一个。&lt;区域&gt; .rgw.control

通知。&lt;N&gt;&lt;区域&gt; .rgw.meta

具有不同类型的元数据的多个名称空间：命名空间：根

&lt;bucket&gt; .bucket.meta。&lt;bucket&gt;：&lt;marker&gt;＃参见put\_bucket\_instance\_info（）

租户用于消除存储桶的歧义，但不能消除存储桶实例的歧义。例：

```text
.bucket.meta.prodtx:test%25star:default.84099.6
.bucket.meta.testcont:default.4126.1
.bucket.meta.prodtx:testcont:default.84099.4
prodtx/testcont
prodtx/test%25star
testcont
```

命名空间：users.uid

在“ &lt;user&gt;”对象中包含每用户信息（RGWUserInfo），并在“ &lt;user&gt; .buckets”对象的omap中包含每个用户的存储桶列表。“ &lt;user&gt;”可能包含租户（如果非空的话），例如：

```text
prodtx$prodt
test2.buckets
prodtx$prodt.buckets
test2
```

命名空间：users.email

不重要命名空间：users.keys

47UA98JSTJZ9YAN3OS3O

这允许radosgw在身份验证期间通过用户的访问密钥查找用户。命名空间：users.swift

测试：测试人员&lt;区域&gt; .rgw.buckets.index

对象名为“ .dir。&lt;marker&gt;”，每个对象都包含一个存储区索引。如果索引是分片的，则每个分片都会在标记后附加分片索引。&lt;区域&gt; .rgw.buckets.data

default.7593.4\_\_shadow\_.488urDFerTYXavx4yAd-Op8mxehnvTI\_1 &lt;marker&gt; \_ &lt;key&gt;

标记的示例为“ default.16004.1”或“ default.7593.4”。当前格式为“ &lt;zone&gt;。&lt;instance\_id&gt;。&lt;bucket\_id&gt;”。但是一旦生成标记，就不会再次对其进行解析，因此其格式将来可能会自由更改。

