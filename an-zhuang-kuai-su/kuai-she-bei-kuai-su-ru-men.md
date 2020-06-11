# 块设备快速入门

## 块设备快速入门

要使用本指南，您必须先执行《Storage Cluster快速入门》指南中的过程。在使用Ceph块设备之前，请确保您的Ceph存储群集处于状态。`active + clean`

注意 

Ceph块设备也称为RBD或RADOS 块设备。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-921dec410b0b0c4cac42031b5829443d413c0960.png)

您可以将虚拟机用于您的`ceph-client`节点，但不要在与Ceph Storage Cluster节点相同的物理节点上执行以下过程（除非您使用VM）。有关详细信息，请参见[常见问题解答](http://wiki.ceph.com/How_Can_I_Give_Ceph_a_Try)。

### 安装CEPH

1. 验证您是否具有适当版本的Linux内核。有关详细信息，请参见操作系统建议。

   ```text
   lsb_release -a
   uname -r
   ```

2. 在管理节点上，用于`ceph-deploy`在您的`ceph-client`节点上安装Ceph 。

   ```text
   ceph-deploy install ceph-client
   ```

3. 在管理节点上，用于`ceph-deploy`将Ceph配置文件和复制`ceph.client.admin.keyring`到`ceph-client`。

   ```text
   ceph-deploy admin ceph-client
   ```

   该`ceph-deploy`实用程序将密钥环复制到`/etc/ceph` 目录中。确保密钥环文件具有适当的读取权限（例如）。`sudo chmod +r /etc/ceph/ceph.client.admin.keyring`

### 创建一个块设备池

1. 在管理节点上，使用该`ceph`工具[创建一个池](https://docs.ceph.com/docs/nautilus/rados/operations/pools/#create-a-pool) （我们建议使用名称“ rbd”）。
2. 在管理节点上，使用该`rbd`工具初始化池以供RBD使用：

   ```text
   rbd pool init <pool-name>
   ```

### 配置块设备

1. 在该`ceph-client`节点上，创建一个块设备映像。

   ```text
   rbd create foo --size 4096 --image-feature layering [-m {mon-IP}] [-k /path/to/ceph.client.admin.keyring] [-p {pool-name}]
   ```

2. 在`ceph-client`节点上，将映像映射到块设备。

   ```text
   sudo rbd map foo --name client.admin [-m {mon-IP}] [-k /path/to/ceph.client.admin.keyring] [-p {pool-name}]
   ```

3. 通过在`ceph-client` 节点上创建文件系统来使用块设备。

   ```text
   sudo mkfs.ext4 -m0 /dev/rbd/{pool-name}/foo

   This may take a few moments.
   ```

4. 在`ceph-client`节点上挂载文件系统。

   ```text
   sudo mkdir /mnt/ceph-block-device
   sudo mount /dev/rbd/{pool-name}/foo /mnt/ceph-block-device
   cd /mnt/ceph-block-device
   ```

5. （可选）将块设备配置为在引导时自动进行映射和安装（在关机时进行卸载/取消映射）-请参见[rbdmap联机帮助页](https://docs.ceph.com/docs/nautilus/man/8/rbdmap)。



