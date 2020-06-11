# 用户管理

## 用户管理

本文档介绍了[Ceph客户端](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-client)用户及其在[Ceph存储群集中](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-storage-cluster)的身份验证和授权。用户是使用Ceph客户端与Ceph存储群集守护程序进行交互的个人或系统参与者（例如应用程序）。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-4e41c1197cfd86b7f36a128297660e48903315d1.png)

当Ceph在启用身份验证和授权的情况下运行（默认情况下启用）时，您必须指定用户名和包含指定用户的私钥的密钥环（通常是通过命令行）。如果您未指定用户名，则Ceph将`client.admin`用作默认用户名。如果您未指定密钥环，则Ceph将通过`keyring`Ceph配置中的设置来寻找密钥环。例如，如果在 不指定用户或密钥环的情况下执行命令：`ceph health`

```text
ceph health
```

Ceph解释如下命令：

```text
ceph -n client.admin --keyring=/etc/ceph/ceph.client.admin.keyring health
```

或者，您可以使用`CEPH_ARGS`环境变量来避免重新输入用户名和密码。

有关配置Ceph存储群集以使用身份验证的详细信息，请参阅《[Cephx配置参考》](https://docs.ceph.com/docs/nautilus/rados/configuration/auth-config-ref)。有关Cephx架构的详细信息，请参阅《 [架构-高可用性身份验证》](https://docs.ceph.com/docs/nautilus/architecture#high-availability-authentication)。

### 背景

无论Ceph客户端的类型如何（例如，块设备，对象存储，文件系统，本机API等），Ceph都将所有数据存储为对象在[池中](https://docs.ceph.com/docs/nautilus/rados/operations/pools)。Ceph用户必须有权访问池才能读取和写入数据。此外，Ceph用户必须具有执行权限才能使用Ceph的管理命令。以下概念将帮助您了解Ceph用户管理。

#### 用户

用户可以是个人，也可以是系统参与者，例如应用程序。通过创建用户，您可以控制哪些人（或什么人）可以访问您的Ceph存储群集，其池以及池中的数据。

Ceph具有`type`用户a的概念。出于用户管理的目的，类型始终为`client`。（。）在周期CEPH识别用户分隔形式由用户类型和用户ID的：例如， `TYPE.ID`，`client.admin`，或`client.user1`。用户键入的原因是Ceph监视器，OSD和元数据服务器也使用Cephx协议，但它们不是客户端。区分用户类型有助于区分客户端用户和其他用户-简化访问控制，用户监控和可追溯性。

有时Ceph的用户类型可能看起来令人困惑，因为Ceph命令行允许您根据命令行的使用情况指定使用或不使用类型的用户。如果指定`--user`或`--id`，则可以省略类型。因此`client.user1`可以简单地输入为`user1`。如果指定 `--name`或`-n`，则必须指定类型和名称，例如 `client.user1`。我们建议尽可能使用类型和名称作为最佳实践。

注意 

Ceph存储群集用户与Ceph对象存储用户或Ceph文件系统用户不同。Ceph对象网关使用Ceph存储群集用户在网关守护程序和存储群集之间进行通信，但是网关对最终用户具有自己的用户管理功能。Ceph文件系统使用POSIX语义。与Ceph文件系统关联的用户空间与Ceph存储集群用户不同。

#### 授权（功能）

Ceph使用术语“功能”（caps）来描述授权经过身份验证的用户行使监视器，OSD和元数据服务器的功能。功能还可以根据其应用程序标记来限制对池中的数据，池中的命名空间或一组池的访问。Ceph管理用户在创建或更新用户时设置用户的功能。

功能语法遵循以下形式：

```text
{daemon-type} '{cap-spec}[, {cap-spec} ...]'
```

* **监控上限：**监控功能包括`r`，`w`，`x`访问设置或。例如：`profile {name}`

  ```text
  mon 'allow {access-spec} [network {network/prefix}]'

  mon 'profile {name}'
  ```

  的`{access-spec}`语法如下：

  ```text
  * | all | [r][w][x]
  ```

  可选的`{network/prefix}`是标准网络名称和CIDR表示法中的前缀长度（例如`10.3.0.0/16`）。如果存在，则此功能的使用仅限于从该网络连接的客户端。

* **OSD上限：** OSD功能包括`r`，`w`，`x`，`class-read`， `class-write`访问设置或。此外，OSD功能还允许池和名称空间设置。`profile {name}`

  ```text
  osd 'allow {access-spec} [{match-spec}] [network {network/prefix}]'

  osd 'profile {name} [pool={pool-name} [namespace={namespace-name}]] [network {network/prefix}]'
  ```

  该`{access-spec}`语法是以下情况之一：

  ```text
  * | all | [r][w][x] [class-read] [class-write]

  class {class name} [{method name}]
  ```

  可选`{match-spec}`语法为以下任意一种：

  ```text
  pool={pool-name} [namespace={namespace-name}] [object_prefix {prefix}]

  [namespace={namespace-name}] tag {application} {key}={value}
  ```

  可选的`{network/prefix}`是标准网络名称和CIDR表示法中的前缀长度（例如`10.3.0.0/16`）。如果存在，则此功能的使用仅限于从该网络连接的客户端。

* **经理上限：**经理（`ceph-mgr`）功能包括 `r`，`w`，`x`访问设置或。例如：`profile {name}`

  ```text
  mgr 'allow {access-spec} [network {network/prefix}]'

  mgr 'profile {name} [{key1} {match-type} {value1} ...] [network {network/prefix}]'
  ```

  还可以为特定命令，内置管理器服务导出的所有命令或特定附加模块导出的所有命令指定管理器功能。例如：

  ```text
  mgr 'allow command "{command-prefix}" [with {key1} {match-type} {value1} ...] [network {network/prefix}]'

  mgr 'allow service {service-name} {access-spec} [network {network/prefix}]'

  mgr 'allow module {module-name} [with {key1} {match-type} {value1} ...] {access-spec} [network {network/prefix}]'
  ```

  的`{access-spec}`语法如下：

  ```text
  * | all | [r][w][x]
  ```

  的`{service-name}`是以下情况之一：

  ```text
  mgr | osd | pg | py
  ```

  的`{match-type}`是以下情况之一：

  ```text
  = | prefix | regex
  ```

* **元数据服务器上限：**对于管理员，请使用。对于所有其他用户，例如CephFS客户端，请查阅[CephFS客户端功能](https://docs.ceph.com/docs/nautilus/cephfs/client-auth/)`allow *`

注意 

Ceph对象网关守护程序（`radosgw`）是Ceph存储群集的客户端，因此它不表示为Ceph存储群集守护程序类型。

以下条目描述了每种访问功能。

`allow`描述

在守护程序的访问设置之前。意味着`rw` 只有MDS。

`r`描述

授予用户读取访问权限。监视器必需，以检索CRUSH映射。

`w`描述

授予用户对对象的写访问权限。

`x`描述

使用户能够调用类方法（即读和写）并`auth` 在监视器上进行操作。

`class-read`内容描述

使用户能够调用类读取方法。的子集`x`。

`class-write`描述

使用户能够调用类写入方法。的子集`x`。

`*`， `all`描述

向用户授予对特定守护程序/池的读取，写入和执行权限，以及执行管理命令的能力。

以下条目描述了有效的功能配置文件：

`profile osd` （仅显示器）描述

授予用户作为OSD连接到其他OSD或监视器的权限。授予OSD以便使OSD能够处理复制心跳流量和状态报告。

`profile mds` （仅显示器）描述

授予用户作为MDS连接到其他MDS或监视器的权限。

`profile bootstrap-osd` （仅显示器）描述

授予用户引导OSD的权限。赋予如部署工具`ceph-volume`，`ceph-deploy`等等，使他们有权自举一个OSD时添加键等。

`profile bootstrap-mds` （仅显示器）描述

向用户授予引导元数据服务器的权限。赋予部署工具（如`ceph-deploy`等），因此它们在引导元数据服务器时有权添加密钥等。

`profile bootstrap-rbd` （仅显示器）描述

授予用户引导RBD用户的权限。赋予部署工具（如`ceph-deploy`等）的权限，因此它们在引导RBD用户时有权添加密钥等。

`profile bootstrap-rbd-mirror` （仅显示器）描述

授予用户引导`rbd-mirror`守护程序用户的权限。赋予部署工具（例如`ceph-deploy`等），因此它们在引导`rbd-mirror`守护程序时有权添加密钥等。

`profile rbd` （管理器，监视器和OSD）描述

授予用户操作RBD图像的权限。当用作Monitor Cap时，它提供了RBD客户端应用程序所需的最小特权；这包括将其他客户端用户列入黑名单的能力。当用作OSD上限时，它为RBD客户端应用程序提供对指定池的读写访问。Manager上限支持可选 参数`pool`和`namespace`关键字参数。

`profile rbd-mirror` （仅显示器）描述

授予用户操作RBD图像和检索RBD镜像配置密钥机密的权限。它提供了`rbd-mirror`守护程序所需的最小特权。

`profile rbd-read-only` （经理和OSD）描述

向用户授予RBD图像的只读权限。Manager上限支持可选参数`pool`和`namespace`关键字参数。

#### 池

池是用户存储数据的逻辑分区。在Ceph部署中，通常创建一个池作为类似数据类型的逻辑分区。例如，作为用于开栈后端部署Ceph的时，一个典型的部署将具有用于卷，图像，备份和虚拟机，并且用户例如池`client.glance`，`client.cinder`等

#### 应用标签

可以将访问限制为由其应用程序元数据定义的特定池。的`*`通配符可以被用于`key`参数， `value`参数，或两者。`all`是...的代名词`*`。

#### 命名空间

池中的对象可以关联到命名空间-池中对象的逻辑组。用户对池的访问可以与名称空间相关联，这样，用户的读取和写入操作仅在名称空间内进行。写入池中名称空间的对象只能由有权访问该名称空间的用户访问。

注意 

命名空间主要用于编写在`librados`逻辑分组可以减轻创建不同池需求的应用程序 。Ceph对象网关（来自`luminous`）使用命名空间来存储各种元数据对象。

名称空间的基本原理是，出于授权单独的用户集的目的，池可能是一种计算量大的，将数据集分离的方法。例如，每个OSD池应具有约100个放置组。因此，具有1000个OSD的示例群集将为一个池具有100,000个放置组。每个池将在示例集群中创建另外100,000个放置组。相比之下，将对象写入名称空间只是将名称空间与对象名称相关联，而没有单独池的计算开销。您可以使用名称空间，而不是为一个用户或一组用户创建单独的池。**注意：目前**只能使用`librados`。

使用该`namespace` 功能，访问权限可能仅限于特定的RADOS名称空间。支持名称空间的有限遍历；如果指定名称空间的最后一个字符是`*`，则将授予从提供的参数开始的任何名称空间的访问权限。

### 管理用户

用户管理功能使Ceph Storage Cluster管理员能够直接在Ceph Storage Cluster中创建，更新和删除用户。

在Ceph Storage Cluster中创建或删除用户时，可能需要将密钥分发给客户端，以便可以将其添加到密钥环中。有关详细信息，请参见[密钥环管理](https://docs.ceph.com/docs/nautilus/rados/operations/user-management/#keyring-management)。

#### 列出用户

要列出集群中的用户，请执行以下操作：

```text
ceph auth ls
```

Ceph将列出您集群中的所有用户。例如，在两个节点的示例集群中，将输出如下内容：`ceph auth ls`

```text
installed auth entries:

osd.0
        key: AQCvCbtToC6MDhAATtuT70Sl+DymPCfDSsyV4w==
        caps: [mon] allow profile osd
        caps: [osd] allow *
osd.1
        key: AQC4CbtTCFJBChAAVq5spj0ff4eHZICxIOVZeA==
        caps: [mon] allow profile osd
        caps: [osd] allow *
client.admin
        key: AQBHCbtT6APDHhAA5W00cBchwkQjh3dkKsyPjw==
        caps: [mds] allow
        caps: [mon] allow *
        caps: [osd] allow *
client.bootstrap-mds
        key: AQBICbtTOK9uGBAAdbe5zcIGHZL3T/u2g6EBww==
        caps: [mon] allow profile bootstrap-mds
client.bootstrap-osd
        key: AQBHCbtT4GxqORAADE5u7RkpCN/oo4e5W0uBtw==
        caps: [mon] allow profile bootstrap-osd
```

请注意，`TYPE.ID`适用于用户的表示法是，是`osd.0`类型的用户，`osd`其ID为`0`，`client.admin`是类型的用户， `client`其ID为`admin`（即，默认`client.admin`用户）。还要注意，每个条目都有一个条目以及一个或多个 条目。`key: <value>caps:`

您可以使用选项with 将输出保存到文件。`-o {filename}ceph auth ls`

#### 获取用户

要检索特定的用户，键和功能，请执行以下操作：

```text
ceph auth get {TYPE.ID}
```

例如：

```text
ceph auth get client.admin
```

您也可以使用选项with 将输出保存到文件。开发人员还可以执行以下操作：`-o {filename}ceph auth get`

```text
ceph auth export {TYPE.ID}
```

该命令与相同。`auth exportauth get`

#### 添加用户

添加用户会创建一个用户名（即`TYPE.ID`），一个秘密密钥以及您用来创建用户的命令中包含的所有功能。

用户的密钥使用户可以通过Ceph存储群集进行身份验证。用户的功能授权用户在Ceph监视器（`mon`），Ceph OSD（`osd`）或Ceph Metadata Server（`mds`）上进行读取，写入或执行。

有几种添加用户的方法：

* `ceph auth add`：此命令是添加用户的规范方法。它将创建用户，生成密钥并添加任何指定功能。
* `ceph auth get-or-create`：此命令通常是创建用户的最方便的方法，因为它返回带有用户名（在方括号中）和密钥的密钥文件格式。如果用户已经存在，则此命令仅以密钥文件格式返回用户名和密钥。您可以使用该 选项将输出保存到文件中。`-o {filename}`
* `ceph auth get-or-create-key`：此命令是创建用户并返回用户密钥（仅）的便捷方法。这对于仅需要密钥的客户端（例如libvirt）很有用。如果用户已经存在，则此命令仅返回密钥。您可以使用该选项将输出保存到文件中。`-o {filename}`

创建客户端用户时，您可能会创建没有功能的用户。没有能力的用户仅凭身份验证就没有用，因为客户端无法从监视器检索群集映射。但是，如果希望以后再使用该命令推迟添加功能，则可以创建一个没有功能的用户。`ceph auth caps`

典型的用户至少在Ceph监视器上具有读取功能，并且在Ceph OSD上具有读写功能。此外，用户的OSD权限通常仅限于访问特定池。

```text
ceph auth add client.john mon 'allow r' osd 'allow rw pool=liverpool'
ceph auth get-or-create client.paul mon 'allow r' osd 'allow rw pool=liverpool'
ceph auth get-or-create client.george mon 'allow r' osd 'allow rw pool=liverpool' -o george.keyring
ceph auth get-or-create-key client.ringo mon 'allow r' osd 'allow rw pool=liverpool' -o ringo.key
```

重要 

如果您为用户提供OSD功能，但是您不限制对特定池的访问，则该用户将有权访问群集中的所有池！

#### 修改用户能力

该命令允许您指定用户并更改用户的功能。设置新功能将覆盖当前功能。要查看当前功能，请运行。要添加功能，还应在使用表单时指定现有功能：`ceph auth capsceph auth get USERTYPE.USERID`

```text
ceph auth caps USERTYPE.USERID {daemon} 'allow [r|w|x|*|...] [pool={pool-name}] [namespace={namespace-name}]' [{daemon} 'allow [r|w|x|*|...] [pool={pool-name}] [namespace={namespace-name}]']
```

例如：

```text
ceph auth get client.john
ceph auth caps client.john mon 'allow r' osd 'allow rw pool=liverpool'
ceph auth caps client.paul mon 'allow rw' osd 'allow rwx pool=liverpool'
ceph auth caps client.brian-manager mon 'allow *' osd 'allow *'
```

有关[功能](https://docs.ceph.com/docs/nautilus/rados/operations/user-management/#authorization-capabilities)的更多详细信息，请参见[授权（功能）](https://docs.ceph.com/docs/nautilus/rados/operations/user-management/#authorization-capabilities)。

#### 删除用户

要删除用户，请使用：`ceph auth del`

```text
ceph auth del {TYPE}.{ID}
```

哪里`{TYPE}`是一个`client`，`osd`，`mon`，或`mds`，并且`{ID}`是用户名或守护进程的ID。

#### 打印用户密钥

要将用户的认证密钥打印到标准输出，请执行以下操作：

```text
ceph auth print-key {TYPE}.{ID}
```

哪里`{TYPE}`是一个`client`，`osd`，`mon`，或`mds`，并且`{ID}`是用户名或守护进程的ID。

当您需要使用用户密钥（例如libvirt）填充客户端软件时，打印用户密钥非常有用。

```text
mount -t ceph serverhost:/ mountpoint -o name=client.user,secret=`ceph auth print-key client.user`
```

#### 导入用户

要导入一个或多个用户，请使用并指定一个密钥环：`ceph auth import`

```text
ceph auth import -i /path/to/keyring
```

例如：

```text
sudo ceph auth import -i /etc/ceph/ceph.keyring
```

注意 

ceph存储集群将添加新用户，其密钥和功能，并将更新现有用户，其密钥和功能。

### 匙扣管理

当您通过Ceph客户端访问Ceph时，Ceph客户端将查找本地密钥环。Ceph `keyring`默认情况下使用以下四个密钥环名称来预设设置，因此除非您想覆盖默认值（不推荐），否则您不必在Ceph配置文件中进行设置：

* `/etc/ceph/$cluster.$name.keyring`
* `/etc/ceph/$cluster.keyring`
* `/etc/ceph/keyring`
* `/etc/ceph/keyring.bin`

元`$cluster`变量是由Ceph配置文件的名称定义的Ceph群集名称（即，`ceph.conf`群集名称为`ceph`;因此，`ceph.keyring`）。元`$name`变量是用户类型和用户ID（例如`client.admin`；`ceph.client.admin.keyring`）。

注意 

当执行读取或写入的命令时`/etc/ceph`，您可能需要使用`sudo`来执行命令`root`。

创建用户（例如`client.ringo`）后，必须获取密钥并将其添加到Ceph客户端上的密钥环中，以便用户可以访问Ceph存储集群。

该[用户管理](https://docs.ceph.com/docs/nautilus/rados/operations/user-management/#id1)部分详细介绍如何列表，获取，添加，修改和删除用户直接在Ceph的存储集群。但是，Ceph还提供了 `ceph-authtool`实用程序，使您可以从Ceph客户端管理密钥环。

#### 创建一个密钥环

使用“ [管理用户”](https://docs.ceph.com/docs/nautilus/rados/operations/user-management/#managing-users)部分中的过程创建用户时，需要向Ceph客户端提供用户密钥，以便Ceph客户端可以检索指定用户的密钥并通过Ceph存储集群进行身份验证。Ceph客户端访问密钥环以查找用户名并检索用户的密钥。

该`ceph-authtool`实用程序允许您创建密钥环。要创建空钥匙圈，请使用`--create-keyring`或`-C`。例如：

```text
ceph-authtool --create-keyring /path/to/keyring
```

当创建具有多个用户的密钥环时，我们建议使用群集名称（例如`$cluster.keyring`）作为密钥环文件名并将其保存在 `/etc/ceph`目录中，以便`keyring`配置默认设置将选择文件名，而无需您在的本地副本中指定它您的Ceph配置文件。例如，`ceph.keyring`通过执行以下操作创建：

```text
sudo ceph-authtool -C /etc/ceph/ceph.keyring
```

当由一个用户创建密钥环时，我们建议使用群集名称，用户类型和用户名，并将其保存在`/etc/ceph`目录中。例如，`ceph.client.admin.keyring`对于`client.admin`用户。

要创建密钥环`/etc/ceph`，您必须按`root`。这意味着该文件将仅对用户具有`rw`权限`root`，这在密钥环包含管理员密钥时才适用。但是，如果打算将密钥环用于特定用户或一组用户，请确保执行`chown`或`chmod`建立适当的密钥环所有权和访问权。

#### 向钥匙圈添加用户

当您 [添加用户](https://docs.ceph.com/docs/nautilus/rados/operations/user-management/#add-a-user)到Ceph的存储集群，您可以用[获取用户](https://docs.ceph.com/docs/nautilus/rados/operations/user-management/#get-a-user)程序检索用户，按键和功能，并且用户保存到钥匙圈。

如果每个密钥环只希望使用一个用户，则带有选项的“ [获取用户”](https://docs.ceph.com/docs/nautilus/rados/operations/user-management/#get-a-user)过程`-o`将以密钥环文件格式保存输出。例如，要为`client.admin`用户创建密钥环，请执行以下操作：

```text
sudo ceph auth get client.admin -o /etc/ceph/ceph.client.admin.keyring
```

请注意，我们为个人用户使用推荐的文件格式。

当您要将用户导入密钥环时，可以用于`ceph-authtool` 指定目标密钥环和源密钥环。例如：

```text
sudo ceph-authtool /etc/ceph/ceph.keyring --import-keyring /etc/ceph/ceph.client.admin.keyring
```

#### 创建用户

Ceph提供了“ [添加用户”](https://docs.ceph.com/docs/nautilus/rados/operations/user-management/#add-a-user)功能，可以直接在Ceph存储集群中创建用户。但是，您也可以直接在Ceph客户端密钥环上创建用户，密钥和功能。然后，您可以将用户导入Ceph存储集群。例如：

```text
sudo ceph-authtool -n client.ringo --cap osd 'allow rwx' --cap mon 'allow rwx' /etc/ceph/ceph.keyring
```

有关[功能](https://docs.ceph.com/docs/nautilus/rados/operations/user-management/#authorization-capabilities)的更多详细信息，请参见[授权（功能）](https://docs.ceph.com/docs/nautilus/rados/operations/user-management/#authorization-capabilities)。

您还可以创建密钥环，并同时将新用户添加到密钥环。例如：

```text
sudo ceph-authtool -C /etc/ceph/ceph.keyring -n client.ringo --cap osd 'allow rwx' --cap mon 'allow rwx' --gen-key
```

在上述情况下，新用户`client.ringo`仅位于密钥环中。要将新用户添加到Ceph存储群集，您仍然必须将新用户添加到Ceph存储群集。

```text
sudo ceph auth add client.ringo -i /etc/ceph/ceph.keyring
```

#### 修改用户

要修改密钥环中用户记录的功能，请指定密钥环，然后再指定用户。例如：

```text
sudo ceph-authtool /etc/ceph/ceph.keyring -n client.ringo --cap osd 'allow rwx' --cap mon 'allow rwx'
```

要将用户更新到Ceph存储群集，您必须将密钥环中的用户更新到Ceph存储群集中的用户条目。

```text
sudo ceph auth import -i /etc/ceph/ceph.keyring
```

有关从密钥环更新Ceph存储群集用户的详细信息，请参阅[导入用户](https://docs.ceph.com/docs/nautilus/rados/operations/user-management/#import-a-user-s)。

您也可以直接在集群中[修改用户功能](https://docs.ceph.com/docs/nautilus/rados/operations/user-management/#id2)，将结果存储到密钥环文件中；然后，将密钥环导入到您的主 `ceph.keyring`文件中。

### 命令行用法

Ceph支持以下用于用户名和密码的用法：

`--id` \| `--user`描述

Ceph的识别用户类型和ID（例如，`TYPE.ID`或 `client.admin`，`client.user1`）。的`id`，`name`和 `-n`选项可以指定用户名（例如，ID部分`admin`，`user1`，`foo`等等）。您可以使用指定用户，`--id`并省略类型。例如，要指定用户，请`client.foo`输入以下内容：

```text
ceph --id foo --keyring /path/to/keyring health
ceph --user foo --keyring /path/to/keyring health
```

`--name` \| `-n`描述

Ceph的识别用户类型和ID（例如，`TYPE.ID`或 `client.admin`，`client.user1`）。在`--name`和`-n` 选项使您可以指定完全合格的用户名。您必须使用`client`用户ID 指定用户类型（通常为）。例如：

```text
ceph --name client.foo --keyring /path/to/keyring health
ceph -n client.foo --keyring /path/to/keyring health
```

`--keyring`描述

包含一个或多个用户名和密码的密钥环的路径。该`--secret`选项提供相同的功能，但不适用于`--secret`用于其他目的的Ceph RADOS网关 。您可以使用检索钥匙圈 并将其存储在本地。这是首选方法，因为您可以切换用户名而无需切换密钥环路径。例如：`ceph auth get-or-create`

```text
sudo rbd map --id foo --keyring /path/to/keyring mypool/myimage
```

### 局限性

该`cephx`协议相互验证Ceph客户端和服务器。它无意处理人类用户或代表他们运行的应用程序的身份验证。如果需要这种效果来满足您的访问控制需求，则必须具有另一种机制，该机制可能特定于用于访问Ceph对象存储的前端。这种其他机制的作用是确保只有可接受的用户和程序才能在Ceph允许访问其对象存储的机器上运行。

用于认证Ceph客户端和服务器的密钥通常存储在纯文本文件中，并在受信任的主机中具有适当的权限。

重要 

将密钥存储在纯文本文件中存在安全缺陷，但是鉴于Ceph在后台使用的基本身份验证方法，很难避免这些缺陷。那些建立Ceph系统的人应该意识到这些缺点。

特别是，不应将任意用户机器（尤其是便携式机器）配置为直接与Ceph交互，因为这种使用方式将需要在不安全的机器上存储明文身份验证密钥。窃取该计算机或对其进行秘密访问的任何人都可以获取密钥，以使他们可以向Ceph验证自己的计算机。

与其允许潜在的不安全机器直接访问Ceph对象存储，不应该要求用户使用一种为您的目的提供足够安全性的方法登录到您环境中的受信任机器。该受信任的机器将为人类用户存储纯文本Ceph密钥。Ceph的未来版本可能会更全面地解决这些特定的身份验证问题。

目前，没有任何一种Ceph身份验证协议为传输中的消息提供保密性。因此，即使无法创建或更改，窃听者仍可以在Ceph中听到并理解在客户端和服务器之间发送的所有数据。此外，Ceph不包括在对象存储中加密用户数据的选项。用户当然可以手动加密并将自己的数据存储在Ceph对象存储中，但是Ceph没有提供任何功能来执行对象加密本身。那些在Ceph中存储敏感数据的人应该在将其数据提供给Ceph系统之前考虑对其进行加密。

