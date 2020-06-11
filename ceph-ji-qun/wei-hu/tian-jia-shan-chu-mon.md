# 添加/删除MON

## 添加/删除MON

当群集启动并运行时，可以在运行时从群集中添加或删除监视器。要引导监视器，请参阅“ [手动部署”](https://docs.ceph.com/docs/nautilus/install/manual-deployment) 或“ [监视器引导”](https://docs.ceph.com/docs/nautilus/dev/mon-bootstrap)。

### 添加监视器

Ceph监视器是轻量级进程，维护集群图的主副本。您可以使用1个监视器运行集群。我们建议生产集群至少使用3个监视器。Ceph监视器使用[Paxos](https://en.wikipedia.org/wiki/Paxos_%28computer_science%29)协议的一种变体 来建立关于整个集群中的地图和其他关键信息的共识。由于Paxos的性质，Ceph要求大多数监视器运行以建立仲裁（从而建立共识）。

建议运行奇数个监视器，但不是强制性的。奇数个监视器比偶数个监视器具有更高的故障恢复能力。例如，在2个监视器的部署中，不能容忍任何故障以维持仲裁。配备3个监视器，可以容忍一个故障；在4个监视器的部署中，可以容忍一个故障。具有5个监视器，可以容忍两个故障。这就是为什么建议使用奇数的原因。总而言之，Ceph需要运行大多数监视器（并且能够相互通信），但是大多数可以使用单个监视器来实现，或者使用2个监视器中的2个，3个中的2个，4个中的3个，等等。 。

对于多节点Ceph集群的初始部署，建议部署三个监视器，如果确实需要三个以上的监视器，则一次增加两个监视器。

由于监视器重量轻，因此可以将它们与OSD放在同一主机上运行。但是，我们建议在单独的主机上运行它们，因为内核的fsync问题可能会降低性能。

注意 

一个_大多数_集群中的显示器必须能够到达对方，以建立仲裁。

#### 部署您的硬件

如果在添加新显示器时要添加新主机，请参阅[硬件建议，](https://docs.ceph.com/docs/nautilus/start/hardware-recommendations)以获取有关显示器硬件最低建议的详细信息。要将监视器主机添加到群集，请首先确保已安装最新版本的Linux（通常是Ubuntu 16.04或RHEL 7）。

将监视器主机添加到群集中的机架，将其连接到网络并确保其具有网络连接性。

#### 安装所需的软件

对于手动部署的群集，必须手动安装Ceph软件包。有关详细信息，请参见[安装软件包](https://docs.ceph.com/docs/nautilus/install/install-storage-cluster)。您应该为具有无密码身份验证和root权限的用户配置SSH。

#### 添加监视器（手动）

此过程将创建一个`ceph-mon`数据目录，检索监视器映射和监视器密钥环，然后将`ceph-mon`守护程序添加到集群中。如果这仅导致两个监视器守护程序，则可以通过重复此过程直到拥有足够数量的`ceph-mon` 守护程序来达到法定数量来添加更多监视器。

此时，您应该定义监视器的ID。传统上，显示器被命名为单字母（`a`，`b`，`c`，...），但是你可以自由，你认为合适定义ID。就本文档而言，请考虑`{mon-id}`该ID为您选择的ID，不带`mon.`前缀（即`{mon-id}`应为`a` on `mon.a`）。

1. 在将承载新监视器的计算机上创建默认目录。

   ```text
   ssh {new-mon-host}
   sudo mkdir /var/lib/ceph/mon/ceph-{mon-id}
   ```

2. 创建一个临时目录`{tmp}`以保留此过程中所需的文件。此目录应与上一步中创建的监视器的默认目录不同，并且在执行所有步骤后可以将其删除。

   ```text
   mkdir {tmp}
   ```

3. 检索显示器的密钥环，其中`{tmp}`是检索到的密钥环的路径，并且`{key-filename}`是包含检索到的显示器密钥的文件的名称。

   ```text
   ceph auth get mon. -o {tmp}/{key-filename}
   ```

4. 检索监视器映射，其中`{tmp}`是检索到的监视器映射的路径，并且`{map-filename}`是包含检索到的监视器映射的文件的名称。

   ```text
   ceph mon getmap -o {tmp}/{map-filename}
   ```

5. 准备在第一步中创建的监视器的数据目录。您必须指定监视器映射的路径，以便可以检索有关法定人数及其监视器的信息`fsid`。您还必须指定监视器密钥环的路径：

   ```text
   sudo ceph-mon -i {mon-id} --mkfs --monmap {tmp}/{map-filename} --keyring {tmp}/{key-filename}
   ```

6. 启动新的监视器，它将自动加入集群。守护程序需要通过或参数知道要绑定到的地址 。例如：`--public-addr {ip}--public-network {network}`

   ```text
   ceph-mon -i {mon-id} --public-addr {ip:port}
   ```

### 拆卸监视器

从群集中删除监视器时，请考虑Ceph监视器使用PAXOS建立有关主群集映射的共识。您必须具有足够数量的监视器才能建立仲裁人数，以就群集映射达成共识。

#### 卸下显示器（手动）

此过程`ceph-mon`从您的集群中删除一个守护程序。如果此过程仅导致两个监视器守护程序，则可以添加或删除另一个监视器，直到拥有许多`ceph-mon`可以实现仲裁的守护程序为止。

1. 停止显示器。

   ```text
   service ceph -a stop mon.{mon-id}
   ```

2. 从群集中删除监视器。

   ```text
   ceph mon remove {mon-id}
   ```

3. 从中删除监视器条目`ceph.conf`。

#### 从不正常的群集中删除监视器

此过程`ceph-mon`将从运行状况不佳的群集（例如，监视器无法形成仲裁的群集）中删除守护程序。

1. 停止所有`ceph-mon`监视器主机上的所有守护程序。

   ```text
   ssh {mon-host}
   service ceph stop mon || stop ceph-mon-all
   # and repeat for all mons
   ```

2. 确定尚存的监视器并登录到该主机。

   ```text
   ssh {mon-host}
   ```

3. 提取monmap文件的副本。

   ```text
   ceph-mon -i {mon-id} --extract-monmap {map-path}
   # in most cases, that's
   ceph-mon -i `hostname` --extract-monmap /tmp/monmap
   ```

4. 卸下无法幸存或有问题的显示器。举例来说，如果你有三个显示器，`mon.a`，`mon.b`，和`mon.c`，只有`mon.a`将生存，遵循下面的例子：

   ```text
   monmaptool {map-path} --rm {mon-id}
   # for example,
   monmaptool /tmp/monmap --rm b
   monmaptool /tmp/monmap --rm c
   ```

5. 将移除了监视器的生存地图注入到生存监视器中。例如，要将地图注入monitor中 `mon.a`，请遵循以下示例：

   ```text
   ceph-mon -i {mon-id} --inject-monmap {map-path}
   # for example,
   ceph-mon -i a --inject-monmap /tmp/monmap
   ```

6. 仅启动尚存的监视器。
7. 验证监视器是否达到法定人数（）。`ceph -s`
8. 您可能希望将已删除的监视器的数据目录归档 `/var/lib/ceph/mon`在一个安全的位置，或者，如果您确信其余的监视器是健康的并且足够冗余，则可以将其删除。

### 更改显示器的IP地址

重要 

现有监视器不应更改其IP地址。

监控器是Ceph集群的关键组成部分，它们需要保持仲裁，以使整个系统正常工作。要建立仲裁，监视器需要相互发现。Ceph对发现监视器有严格的要求。

Ceph客户端和其他Ceph守护程序用于`ceph.conf`发现监视器。但是，监视器使用监视器映射而不是彼此发现`ceph.conf`。例如，如果您参考[添加监视器（手动），](https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-mons/#adding-a-monitor-manual)您将看到创建新监视器时需要获取集群的当前monmap，因为它是的必需参数之一。以下各节介绍了Ceph监视器的一致性要求，以及一些更改监视器IP地址的安全方法。`ceph-mon -i {mon-id} --mkfs`

#### 一致性的要求

当发现群集中的其他监视器时，监视器始终引用monmap的本地副本。使用monmap而不是`ceph.conf`避免可能破坏群集的错误（例如，在`ceph.conf`指定监视器地址或端口时输入错误）。由于监视器使用monmap进行发现，并且它们与客户端和其他Ceph守护程序共享monmap，因此monmap为监视器提供了严格的保证，即它们的共识是有效的。

严格的一致性也适用于对monmap的更新。与监视器上的任何其他更新一样，对monmap的更改始终通过称为[Paxos](https://en.wikipedia.org/wiki/Paxos_%28computer_science%29)的分布式共识算法来运行。监视器必须同意对monmap的每次更新，例如添加或删除监视器，以确保仲裁中的每个监视器都具有相同版本的monmap。对monmap的更新是增量更新，因此监视器具有最新的议定版本和一组以前的版本，从而使具有较旧版本monmap的监视器可以赶上群集的当前状态。

如果监视器是通过Ceph配置文件而不是通过monmap彼此发现的，这将带来其他风险，因为Ceph配置文件不会自动更新和分发。监视器可能会无意中使用了较旧的`ceph.conf`文件，无法识别监视器，没有达到法定人数，或者出现了[Paxos](https://en.wikipedia.org/wiki/Paxos_%28computer_science%29)无法准确确定系统当前状态的情况。因此，必须非常小心地更改现有监视器的IP地址。

#### 更改显示器的IP地址（正确的方法）

仅更改监视器的IP地址`ceph.conf`不足以确保群集中的其他监视器将接收更新。要更改监视器的IP地址，必须添加具有您要使用的IP地址的新监视器（如[添加监视器（手动）中所述](https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-mons/#adding-a-monitor-manual)），确保新监视器成功加入了仲裁。然后，删除使用旧IP地址的监视器。然后，更新`ceph.conf`文件以确保客户端和其他守护程序知道新监视器的IP地址。

例如，假设有三个监视器，例如

```text
[mon.a]
        host = host01
        addr = 10.0.0.1:6789
[mon.b]
        host = host02
        addr = 10.0.0.2:6789
[mon.c]
        host = host03
        addr = 10.0.0.3:6789
```

要更改`mon.c`为`host04`IP地址 `10.0.0.4`，请按照通过添加新监视器[添加监视器（手动）中](https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-mons/#adding-a-monitor-manual)的步骤进行[操作](https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-mons/#adding-a-monitor-manual)`mon.d`。`mon.d`在删除之前`mon.c`，请确保其正在运行，否则它将破坏仲裁。删除`mon.c`作为描述 [删除监视器（手动）](https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-mons/#removing-a-monitor-manual)。因此，移动所有三个监视器将需要根据需要重复此过程多次。

#### 更改监视器的IP地址（麻烦的方式）

有时可能必须将监视器移到其他网络，数据中心的不同部分或完全不同的数据中心。尽管可以这样做，但过程变得更加危险。

在这种情况下，解决方案是为群集中的所有监视器生成具有更新的IP地址的新monmap，并将新的map注入每个单独的监视器。这不是最人性化的方法，但是我们不希望这每隔一周就要做一次。正如本节顶部明确指出的那样，监视器不应更改IP地址。

以以前的监视器配置为例，假设您要将所有监视器从`10.0.0.x`范围移动到`10.1.0.x`，并且这些网络无法通信。使用以下过程：

1. 检索监视器映射，其中`{tmp}`是检索到的监视器映射的路径，并且`{filename}`是包含检索到的监视器映射的文件的名称。

   ```text
   ceph mon getmap -o {tmp}/{filename}
   ```

2. 下面的示例演示monmap的内容。

   ```text
   $ monmaptool --print {tmp}/{filename}

   monmaptool: monmap file {tmp}/{filename}
   epoch 1
   fsid 224e376d-c5fe-4504-96bb-ea6332a19e61
   last_changed 2012-12-17 02:46:41.591248
   created 2012-12-17 02:46:41.591248
   0: 10.0.0.1:6789/0 mon.a
   1: 10.0.0.2:6789/0 mon.b
   2: 10.0.0.3:6789/0 mon.c
   ```

3. 删除现有的监视器。

   ```text
   $ monmaptool --rm a --rm b --rm c {tmp}/{filename}

   monmaptool: monmap file {tmp}/{filename}
   monmaptool: removing a
   monmaptool: removing b
   monmaptool: removing c
   monmaptool: writing epoch 1 to {tmp}/{filename} (0 monitors)
   ```

4. 添加新的监视器位置。

   ```text
   $ monmaptool --add a 10.1.0.1:6789 --add b 10.1.0.2:6789 --add c 10.1.0.3:6789 {tmp}/{filename}

   monmaptool: monmap file {tmp}/{filename}
   monmaptool: writing epoch 1 to {tmp}/{filename} (3 monitors)
   ```

5. 检查新内容。

   ```text
   $ monmaptool --print {tmp}/{filename}

   monmaptool: monmap file {tmp}/{filename}
   epoch 1
   fsid 224e376d-c5fe-4504-96bb-ea6332a19e61
   last_changed 2012-12-17 02:46:41.591248
   created 2012-12-17 02:46:41.591248
   0: 10.1.0.1:6789/0 mon.a
   1: 10.1.0.2:6789/0 mon.b
   2: 10.1.0.3:6789/0 mon.c
   ```

此时，我们假设监视器（和存储）已安装在新位置。下一步是将修改后的monmap传播到新的监视器，并将修改后的monmap注入每个新监视器。

1. 首先，请确保停止所有监视器。必须在守护程序未运行时完成注入。
2. 注入monmap。

   ```text
   ceph-mon -i {mon-id} --inject-monmap {tmp}/{filename}
   ```

3. 重新启动监视器。

完成此步骤后，便完成了向新位置的迁移，并且监视器应该成功运行。

