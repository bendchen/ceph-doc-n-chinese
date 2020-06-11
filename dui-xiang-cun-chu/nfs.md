# NFS

## NFS 

珠宝的新版本。

现在，可以通过基于文件的访问协议（例如NFSv3和NFSv4）以及传统的HTTP访问协议（S3和Swift）将Ceph对象网关名称空间导出。

特别是，现在可以将Ceph对象网关配置为在嵌入NFS-Ganesha NFS服务器时提供基于文件的访问。

### LIBRGW 

librgw.so共享库（Unix）为Ceph对象网关服务提供了一个可加载的接口，并在初始化时实例化了完整的Ceph对象网关实例。

反过来，librgw.so导出rgw\_file，这是一个有状态的API，用于对RGW存储桶和对象进行面向文件的访问。该API是通用的，但其设计受到NFS-Ganesha的文件系统抽象层（FSAL）API的强烈影响，而该API最初是针对该API设计的。

还提供了一组Python绑定。

### 命名空间约定

该实现符合Amazon Web Services（AWS）分层名称空间约定，该约定将UNIX样式的路径名映射到S3存储桶和对象。

附加名称空间的顶层由S3存储桶组成，表示为NFS目录。遵循S3前缀和定界符约定，存储桶所属的文件和目录均以对象表示，其中'/'是唯一受支持的路径定界符[1](https://docs.ceph.com/docs/nautilus/radosgw/nfs/#id2)。

例如，如果NFS客户端已在“ / nfs”处安装了RGW名称空间，则NFS名称空间中的文件“ /nfs/mybucket/www/index.html”对应于RGW对象“ www / index.html”。一个水桶/容器“ mybucket”。

尽管它通常对于客户端是不可见的，但是NFS命名空间是通过串联命名空间中对象所隐含的相应路径来组装的。叶子对象（无论是文件还是目录）将始终在相应键名的RGW对象中实现，如果是文件，则为“ &lt;name&gt;”，如果是目录，则为“ &lt;name&gt; /”。非叶子目录（例如，上面的“ www”）可能仅通过在一个或多个叶子对象的名称中出现来暗示。在NFS中创建的目录或由NFS客户端直接操作的目录（例如，通过诸如chown或chmod之类的属性设置操作）始终具有用于存储物化属性（例如Unix所有权和权限）的叶对象表示。

### 支持的操作

RGW NFS接口支持对文件和目录的大多数操作，但有以下限制：

* 不支持链接，包括符号链接
* 不支持NFS ACL
  * Unix用户和组的所有权和权限_都_支持
* 目录可能无法移动/重命名
  * 文件可以在目录之间移动
* 仅支持完整的顺序_写入_ I / O
  * 即写操作被限制为上**载**
  * 许多典型的I / O操作（例如，在适当位置编辑文件）必然会因执行非顺序存储而失败
  * 由于不频繁的非顺序存储，某些_显然_按顺序写入的文件实用程序（例如，某些版本的GNU tar）可能会失败
  * 通过NFS挂载时，通常可以限制顺序应用程序I / O通过同步挂载选项（例如Linux中的-osync）顺序写入NFS服务器。
  * 无法同步安装的NFS客户端（例如，MS Windows）将无法上传文件

### 安全

RGW NFS接口提供具有以下特征的混合安全模型：

* 由NFS服务器和客户端协商，由NFS-Ganesha服务器提供NFS协议安全性
  * 例如，客户端可以通过信任（AUTH\_SYS），或要求提供Kerberos用户凭据（RPCSEC\_GSS）
  * RPCSEC\_GSS电线安全性可以是仅完整性（krb5i）或完整性和私密性（加密，krb5p）
  * 各种NFS特定的安全和权限规则可用
    * 例如，压根
* 一组RGW / S3安全凭证（NFS未知）与每个RGW NFS挂载（即NFS-Ganesha EXPORT）相关联
  * 通过NFS服务器执行的所有RGW对象操作将由与存储在正在访问的导出中的凭据关联的RGW用户执行（当前仅支持RGW和RGW LDAP凭据）
    * 当前不支持其他RGW身份验证类型，例如Keystone

### 配置NFS-GANESHA实例

每个NFS RGW实例都是一个NFS-Ganesha服务器实例，其中_嵌入_ 了完整的Ceph RGW实例。

因此，RGW NFS配置包括本地ceph.conf中特定于Ceph和Ceph对象网关的配置，以及NFS-Ganesha配置文件ganesha.conf中特定于NFS-Ganesha的配置。

#### CEPH.CONF 

RGW NFS所需的ceph.conf配置包括：

* 有效的\[client.radosgw。{instance-name}\]部分
* 最小实例配置的有效值，尤其是已安装且正确的值 `keyring`

其他配置变量是可选的，特定于前端的和前端选择变量（例如和）是可选的，在某些情况下会被忽略。`rgw datargw frontends`

少量的配置变量（例如`rgw_nfs_namespace_expire_secs`）对于RGW NFS是唯一的。

#### GANESHA.CONF 

与RGW NFS一起使用的严格最小的ganesha.conf包含一个EXPORT块和RGW类型的嵌入式FSAL块：

```text
EXPORT
{
     Export_ID={numeric-id};
     Path = "/";
     Pseudo = "/";
     Access_Type = RW;
     SecType = "sys";
     NFS_Protocols = 4;
     Transport_Protocols = TCP;

     # optional, permit unsquashed access by client "root" user
     #Squash = No_Root_Squash;

     FSAL {
             Name = RGW;
             User_Id = {s3-user-id};
             Access_Key_Id ="{s3-access-key}";
             Secret_Access_Key = "{s3-secret}";
     }
}
```

`Export_ID` 必须具有整数值，例如“ 77”

`Path` （对于RGW）应为“ /”

`Pseudo` 定义一个NFSv4伪根名称（仅NFSv4）

`SecType = sys;` 允许客户端连接而无需Kerberos身份验证

`Squash = No_Root_Squash;`使客户机root用户可以覆盖权限（Unix约定）。启用root-squashing后，由root用户尝试的操作就像由NFS-Ganesha服务器上的本地“ nobody”（和“ nogroup”）用户执行一样

RGW FSAL在RGW config部分中还支持特定于RGW的配置变量：

```text
RGW {
    cluster = "{cluster name, default 'ceph'}";
    name = "client.rgw.{instance-name}";
    ceph_conf = "/opt/ceph-rgw/etc/ceph/ceph.conf";
    init_args = "-d --debug-rgw=16";
}
```

`cluster` 设置一个Ceph集群名称（必须与正在导出的集群匹配）

`name` 设置RGW实例名称（必须与正在导出的集群匹配）

`ceph_conf` 给出了要使用的非默认ceph.conf文件的路径

**其他有用的NFS-GANESHA配置：**

任何应支持NFSv3的EXPORT块都应在NFS\_Protocols设置中包括版本3。此外，NFSv3是支持UDP传输的最后一个主要版本。要启用UDP，请将其包括在Transport\_Protocols设置中。例如：

```text
EXPORT {
 ...
   NFS_Protocols = 3,4;
   Transport_Protocols = UDP,TCP;
 ...
}
```

一个重要的选项系列涉及与Linux idmapping服务的交互，该服务用于标准化系统中的用户名和组名。这里没有提供idmapper集成的详细信息。

借助Linux NFS客户端，可以将NFS-Ganesha配置为使用NFSv4接受客户端提供的数字用户和组标识符，默认情况下，这些标识符会进行字符串化处理-这在小型设置和实验中可能很有用：

```text
NFSV4 {
    Allow_Numeric_Owners = true;
    Only_Numeric_Owners = true;
}
```

**故障排除**

NFS-Ganesha配置问题通常通过使用调试选项运行服务器来调试，该选项由LOG config部分控制。

NFS-Ganesha日志消息分为多个组件，可以分别为每个组件启用日志记录。组件日志记录的有效值包括：

```text
*FATAL* critical errors only
*WARN* unusual condition
*DEBUG* mildly verbose trace output
*FULL_DEBUG* verbose trace output
```

例：

```text
 LOG {

       Components {
               MEMLEAKS = FATAL;
               FSAL = FATAL;
               NFSPROTO = FATAL;
               NFS_V4 = FATAL;
               EXPORT = FATAL;
               FILEHANDLE = FATAL;
               DISPATCH = FATAL;
               CACHE_INODE = FATAL;
               CACHE_INODE_LRU = FATAL;
               HASHTABLE = FATAL;
               HASHTABLE_CACHE = FATAL;
               DUPREQ = FATAL;
               INIT = DEBUG;
               MAIN = DEBUG;
               IDMAPPER = FATAL;
               NFS_READDIR = FATAL;
               NFS_V4_LOCK = FATAL;
               CONFIG = FATAL;
               CLIENTID = FATAL;
               SESSIONS = FATAL;
               PNFS = FATAL;
               RW_LOCK = FATAL;
               NLM = FATAL;
               RPC = FATAL;
               NFS_CB = FATAL;
               THREAD = FATAL;
               NFS_V4_ACL = FATAL;
               STATE = FATAL;
               FSAL_UP = FATAL;
               DBUS = FATAL;
       }
       # optional: redirect log output
#      Facility {
#              name = FILE;
#              destination = "/tmp/ganesha-rgw.log";
#              enable = active;
       }
}
```

### 运行多个NFS网关

每个NFS-Ganesha实例都充当一个完整的网关端点，但有一个限制，即当前无法将NFS-Ganesha实例配置为导出HTTP服务。与普通网关实例一样，可以启动任意数量的NFS-Ganesha实例，从群集中导出相同或不同的资源。这将启用NFS-Ganesha实例的群集。但是，这并不意味着高可用性。

当常规网关实例和NFS-Ganesha实例重叠相同的数据资源时，可以从标准S3 API或通过导出的NFS-Ganesha实例访问它们。您可以将NFS-Ganesha实例与Ceph对象网关实例放置在同一主机上。

### RGW与RGW 

当前不支持从同一程序实例导出NFS名称空间和其他RGW名称空间（例如，通过Civetweb HTTP前端的S3或Swift）。

在NFS之外添加对象和存储桶时，这些对象将以设置的时间出现在NFS命名空间中 `rgw_nfs_namespace_expire_secs`，默认时间为300秒（5分钟）。覆盖`rgw_nfs_namespace_expire_secs`Ceph配置文件中的默认值以更改刷新率。

如果导出不符合有效S3存储桶命名要求的Swift容器，请`rgw_relaxed_s3_bucket_names`在Ceph配置文件的\[client.radosgw\]部分中将其设置为true。例如，如果Swift容器名称包含下划线，则它不是有效的S3存储桶名称，除非`rgw_relaxed_s3_bucket_names` 设置为true，否则将被拒绝。

### 配置NFSV4客户端

要访问名称空间，请将已配置的NFS-Ganesha导出安装到本地POSIX名称空间中的所需位置。如前所述，此实现有一些独特的限制：

* 首选NFS 4.1和更高版本的协议
  * NFSv4 OPEN和CLOSE操作用于跟踪上传事务
* 要成功上传数据，客户端必须保留写顺序
  * 在Linux和许多Unix NFS客户端上，请使用-osync挂载选项

装入NFS资源的约定是特定于平台的。以下约定在Linux和某些Unix平台上有效：

在命令行中：

```text
mount -t nfs -o nfsvers=4.1,noauto,soft,sync,proto=tcp <ganesha-host-name>:/ <mount-point>
```

在/ etc / fstab中：

```text
<ganesha-host-name>:/ <mount-point> nfs noauto,soft,nfsvers=4.1,sync,proto=tcp 0 0
```

指定NFS-Ganesha主机名以及客户端上安装点的路径。

### 配置NFSV3客户端

通过提供`nfsvers=3`和`noacl`作为安装选项，可以将Linux客户端配置为使用NFSv3进行 安装。要将UDP用作传输，请添加`proto=udp`到安装选项。但是，TCP是首选的传输方式：

```text
<ganesha-host-name>:/ <mount-point> nfs noauto,noacl,soft,nfsvers=3,sync,proto=tcp 0 0
```

如果安装将使用版本3和UDP，则将NFS Ganesha EXPORT块协议设置配置为版本3，将传输设置配置为UDP。

#### NFSV3语义

由于NFSv3不会将客户端的OPEN和CLOSE操作传达给文件服务器，因此RGW NFS无法使用这些操作来标记文件上传事务的开始和结束。相反，当第一次写入发送到偏移量为0的文件时，RGW NFS将开始新的上载，并且在一段时间（默认为10秒）内未看到对文件的新写入时，RGW NFS将最终完成上载。要更改此超时，请`rgw_nfs_write_completion_interval_s` 在Ceph配置文件的RGW部分中为设置替代值。

### 参考

[1个](https://docs.ceph.com/docs/nautilus/radosgw/nfs/#id1)

[http://docs.aws.amazon.com/AmazonS3/latest/dev/ListingKeysHierarchy.html](http://docs.aws.amazon.com/AmazonS3/latest/dev/ListingKeysHierarchy.html)

