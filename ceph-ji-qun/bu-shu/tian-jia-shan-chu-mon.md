# 添加/删除MON

## 添加/删除MON

使用`ceph-deploy`，添加和删除监视器是一项简单的任务。您只需使用一个命令在命令行上添加或删除一个或多个监视器。在此之前 `ceph-deploy`，[添加和删​​除监视器](https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-mons)的过程涉及许多手动步骤。使用`ceph-deploy`有一个限制： **每个主机只能安装一个监视器。**

注意 

我们不建议在同一主机上合并监视器和OSD。

为了获得高可用性，您应该使用**至少**三个监视器来运行生产Ceph集群。Ceph使用Paxos算法，该算法需要在法定人数的大多数监视器之间达成共识。使用Paxos，监视器无法确定仅使用两个监视器来建立仲裁的多数。大多数监视器必须这样计数：1：1、2：3、3：4、3：5、4：6等。

有关配置监视器的详细信息，请参见《[监视器配置参考](https://docs.ceph.com/docs/nautilus/rados/configuration/mon-config-ref)》。

### 添加监视器

创建集群并将Ceph软件包安装到监视器主机后，可以将监视器部署到监视器主机。使用时`ceph-deploy`，该工具为每个主机强制使用一个监视器。

```text
ceph-deploy mon create {host-name [host-name]...}
```

注意 

确保添加监视器，以使它们在大多数监视器之间达成共识，否则其他步骤（如）将失败。`ceph-deploy gatherkeys`

注意 

在不使用命令最初定义的主机中的主机上添加监视器时，需要在ceph.conf文件中添加一条语句。`ceph-deploy newpublic network`

### 删除监视器

如果您的集群中有要删除的监视器，则可以使用该`destroy`选项。

```text
ceph-deploy mon destroy {host-name [host-name]...}
```

注意 

确保如果删除监视器，其余监视器将能够建立共识。如果不可能，请在删除要脱机的显示器之前考虑添加显示器。

