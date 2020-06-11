# 加密

## 加密

`dmcrypt`通过`--dmcrypt`在创建OSD时指定 标志，可以对逻辑卷进行加密。加密可以通过不同的方式完成，特别是使用LVM。`ceph-volume`对于使用逻辑卷设置加密的方式有些怀疑，因此该过程是一致且健壮的。

在这种情况下，请遵循以下约束：`ceph-volume lvm`

* 仅使用LUKS（版本1）
* 逻辑卷已加密，而其基础PV（物理卷）未加密
* 诸如分区之类的非LVM设备也使用相同的OSD密钥加密

### 陆氏

目前，LUKS有两个版本：1和2。版本2易于实现，但并非在所有Ceph发行版支持中广泛使用。不赞成使用LUKS 1，而反对使用LUKS 2，因此为了获得尽可能广泛的支持，请`ceph-volume`使用LUKS版本1。

注意 

LUKS的版本1仅被称为“ LUKS”，而版本2则被称为LUKS2

### LVM上的

加密是在现有逻辑卷之上完成的（与加密物理设备不同）。任何单个逻辑卷都可以加密，而其他卷则可以保持未加密状态。此方法还允许灵活的逻辑卷设置，因为一旦创建了LV，就会进行加密。

### 工作流程

设置OSD时，将创建一个秘密密钥，该密钥将以JSON格式传递给监视器，`stdin`以防止密钥被捕获到日志中。

JSON有效负载类似于：

```text
{
    "cephx_secret": CEPHX_SECRET,
    "dmcrypt_key": DMCRYPT_KEY,
    "cephx_lockbox_secret": LOCKBOX_SECRET,
}
```

密钥的命名约定是**strict**，它们的命名方式与所使用的ceph-disk的硬编码（旧）名称相同。

* `cephx_secret` ：用于验证的cephx密钥
* `dmcrypt_key` ：用于解锁加密设备的秘密（或私有）密钥
* `cephx_lockbox_secret`：用于检索的身份验证密钥 `dmcrypt_key`。之所以命名为_密码箱_，是因为ceph磁盘以前有一个以其命名的未加密分区，用于存储公用密钥和其他OSD元数据。

命名约定很严格，因为Monitors支持使用这些键名的ceph-disk命名约定。为了保持兼容性并防止ceph磁盘损坏，ceph-volume将使用相同的命名约定，_尽管它们对于新的加密工作流程没有意义_。

在准备阶段通过[文件存储](https://docs.ceph.com/docs/nautilus/glossary/#term-filestore)或[bluestore](https://docs.ceph.com/docs/nautilus/glossary/#term-bluestore)设置OSD的常规步骤之后，无论设备处于什么状态（加密或解密），逻辑卷都可以随时被激活。

在激活时，一旦过程正确完成，逻辑卷将被解密并启动OSD。

用于创建新OSD的加密工作流的摘要：

1. OSD已创建，密码箱和dmcrypt密钥均已创建，并与JSON一起发送到监视器，指示已加密的OSD。
2. 使用相同的OSD密钥创建并加密所有互补设备（例如日志，数据库或wal）。密钥存储在OSD的LVM元数据中
3. 通过确保安装了设备，从监视器检索dmcrypt秘密密钥并在OSD启动之前进行解密，继续激活。

