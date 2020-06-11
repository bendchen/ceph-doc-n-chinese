# CEPHFS快速入门

## CEPHFS快速入门

要使用《CephFS快速入门》指南，您必须先执行《Storage Cluster快速入门》指南中的过程。在管理主机上执行此快速入门。

### 前提条件

1. 验证您是否具有适当版本的Linux内核。有关详细信息，请参见`操作系统建议`。

   ```text
   lsb_release -a
   uname -r
   ```

2. 在管理节点上，用于`ceph-deploy`在您的`ceph-client`节点上安装Ceph 。

   ```text
   ceph-deploy install ceph-client
   ```

3. 确保[Ceph存储群集](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-storage-cluster)正在运行并且处于 状态。另外，请确保至少有一个[Ceph Metadata Server](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-metadata-server)正在运行。`active + clean`

   ```text
   ceph -s [-m {monitor-ip-address}] [-k {path/to/ceph.client.admin.keyring}]
   ```

### 创建文件系统

您已经创建了MDS（[存储群集快速入门](https://docs.ceph.com/docs/nautilus/start/quick-ceph-deploy)），但是直到创建了一些池和文件系统后，该MDS 才会变为活动状态。请参阅[创建Ceph文件系统](https://docs.ceph.com/docs/nautilus/cephfs/createfs/)。

```text
ceph osd pool create cephfs_data <pg_num>
ceph osd pool create cephfs_metadata <pg_num>
ceph fs new <fs_name> cephfs_metadata cephfs_data
```

### 创建保密文件

Ceph Storage Cluster在默认情况下启用身份验证的情况下运行。您应该有一个包含密钥的文件（即，不是密钥环本身）。要获取特定用户的密钥，请执行以下过程：

1. 在密钥环文件中为用户标识密钥。例如：

   ```text
   cat ceph.client.admin.keyring
   ```

2. 复制将使用已安装的CephFS文件系统的用户的密钥。它看起来应该像这样：

   ```text
   [client.admin]
      key = AQCj2YpRiAe6CxAA7/ETt7Hcl9IyxyYciVs47w==
   ```

3. 打开一个文本编辑器。
4. 将密钥粘贴到一个空文件中。它看起来应该像这样：

   ```text
   AQCj2YpRiAe6CxAA7/ETt7Hcl9IyxyYciVs47w==
   ```

5. 使用用户将文件保存`name`为属性（例如`admin.secret`）。
6. 确保文件权限适合该用户，但对其他用户不可见。

### 内核驱动程序

将CephFS挂载为内核驱动程序。

```text
sudo mkdir /mnt/mycephfs
sudo mount -t ceph {ip-address-of-monitor}:6789:/ /mnt/mycephfs
```

默认情况下，Ceph存储群集使用身份验证。在[创建机密文件](https://docs.ceph.com/docs/nautilus/start/quick-cephfs/#create-a-secret-file)部分中指定用户`name` 和`secretfile`您创建的用户。例如：

```text
sudo mount -t ceph 192.168.0.1:6789:/ /mnt/mycephfs -o name=admin,secretfile=admin.secret
```

注意 

将CephFS文件系统挂载在管理节点上，而不是服务器节点上。有关详细信息，请参见[常见问题解答](http://wiki.ceph.com/How_Can_I_Give_Ceph_a_Try)。

### 用户空间中的文件系统（FUSE）

将CephFS挂载为用户空间（FUSE）中的文件系统。

```text
sudo mkdir ~/mycephfs
sudo ceph-fuse -m {ip-address-of-monitor}:6789 ~/mycephfs
```

默认情况下，Ceph存储群集使用身份验证。如果钥匙圈不在默认位置（即），请指定它`/etc/ceph`：

```text
sudo ceph-fuse -k ./ceph.client.admin.keyring -m 192.168.0.1:6789 ~/mycephfs
```

### 其他信息[¶](https://docs.ceph.com/docs/nautilus/start/quick-cephfs/#additional-information)

有关更多信息，请参见[CephFS](https://docs.ceph.com/docs/nautilus/cephfs/)。CephFS的稳定性不如Ceph块设备和Ceph对象存储。 如果遇到麻烦，请参阅[故障排除](https://docs.ceph.com/docs/nautilus/cephfs/troubleshooting)。

