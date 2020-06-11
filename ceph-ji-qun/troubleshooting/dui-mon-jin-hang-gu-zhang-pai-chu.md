# 对MON进行故障排除

## 对MON进行故障排除

当群集遇到与监视器相关的问题时，会出现恐慌的趋势，有时甚至有充分的理由。您应该记住，丢失一台或多台监视器并不一定意味着您的群集已关闭，只要大多数已启动，正在运行且具有法定人数即可。无论情况有多严重，您都应该做的第一件事是冷静下来，屏住呼吸，然后尝试回答我们的初始故障排除脚本。

### 初始故障排除

**监视器在运行吗？**

> 首先，我们需要确保监视器正在运行。人们会经常忘记运行监视器或在升级后重新启动监视器，您会感到惊讶。这没有什么可耻的，但是让我们尽量不要浪费几个小时来追逐不存在的问题。

**您是否可以连接到显示器的服务器？**

> 不会经常发生，但有时人们确实有一些`iptables`规则会阻止对监视服务器或监视端口的访问。通常在某些时候忘记了监视器压力测试的遗留物。尝试将ssh'插入服务器，如果成功，请尝试使用您选择的工具（telnet，nc等）连接到显示器的端口。

**ceph -s是否运行并从集群获得答复？**

> 如果答案是肯定的，则您的群集已启动并正在运行。您可以想当然的一件事是，监视器只有`status` 在形成法定人数后才会回答请求。
>
> 但是，如果被阻止，而没有从群集中获得答复或显示大量消息，则您的显示器很可能已完全关闭，或者仅一部分处于启动状态–不足以形成仲裁的一部分（请记住）如果是由大多数监视器组成的法定人数）。`ceph -sfault`

**如果ceph -s没有完成怎么办？**

> 如果您到目前为止尚未完成所有步骤，请返回并执行。
>
> 对于在Emperor 0.72-rc1或更高版本上运行的计算机，无论是否形成法定人数，您都可以单独联系每台显示器，询问其状态。这可以通过使用ID作为监视器的标识符来实现。您应该为集群中的每个监视器执行此操作。在“ [了解mon\_status”](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/troubleshooting-mon/#understanding-mon-status)部分中，我们将解释如何解释此命令的输出。`ceph ping mon.ID`
>
> 对于其余的那些不愿意冒险的人，您将需要进入服务器并使用监视器的管理套接字。请跳至 [使用显示器的管理套接字](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/troubleshooting-mon/#using-the-monitor-s-admin-socket)。

对于其他特定问题，请继续阅读。

### 使用监视器的管理套接字

管理员套接字允许您直接使用Unix套接字文件与给定的守护程序进行交互。可以在监视器的`run`目录中找到此文件。默认情况下，管理套接字将保留在其中，`/var/run/ceph/ceph-mon.ID.asok` 但是如果您另行定义，则可以有所不同。如果在此处找不到，请检查您`ceph.conf`的替代路径或运行：

```text
ceph-conf --name mon.ID --show-config-value admin_socket
```

请记住，管理套接字仅在监视器运行时可用。正确关闭监视器后，将删除管理套接字。但是，如果监视器未运行，并且管理套接字仍然存在，则可能是监视器未正确关闭。无论如何，如果监视器未运行，您将无法使用admin套接字，`ceph`可能会返回。`Error 111: Connection Refused`

访问管理套接字就像告诉`ceph`工具使用`asok`文件一样简单。在预包销Ceph中，可以通过以下方式实现：

```text
ceph --admin-daemon /var/run/ceph/ceph-mon.<id>.asok <command>
```

而在饺子及更高版本中，您可以使用备用（推荐）格式：

```text
ceph daemon mon.<id> <command>
```

使用`help`的命令到`ceph`工具将显示您可以通过管理员插座支持的命令。请看一看，，和，如故障排除显示器的时候那些能够启发。`config getconfig showmon_statusquorum_status`

### 了解MON\_STATUS 

`mon_status`可以在`ceph`形成法定人数后通过该工具获得，如果没有，则可以通过管理套接字获得。此命令将输出有关监视器的大量信息，包括您将获得的相同输出`quorum_status`。

以以下示例为例`mon_status`：

```text
{ "name": "c",
  "rank": 2,
  "state": "peon",
  "election_epoch": 38,
  "quorum": [
        1,
        2],
  "outside_quorum": [],
  "extra_probe_peers": [],
  "sync_provider": [],
  "monmap": { "epoch": 3,
      "fsid": "5c4e9d53-e2e1-478a-8061-f543f8be4cf8",
      "modified": "2013-10-30 04:12:01.945629",
      "created": "2013-10-29 14:14:41.914786",
      "mons": [
            { "rank": 0,
              "name": "a",
              "addr": "127.0.0.1:6789\/0"},
            { "rank": 1,
              "name": "b",
              "addr": "127.0.0.1:6790\/0"},
            { "rank": 2,
              "name": "c",
              "addr": "127.0.0.1:6795\/0"}]}}
```

有几件事很明显：我们在monmap中有三个监视器（_a_，_b_ 和_c_），仲裁仅由两个监视器构成，而_c_在_pone_中作为仲裁。

哪个监视器超出法定人数？

> 答案是**一个**。

为什么？

> 看一下`quorum`集合。我们在这个集合中有两个监视器：_1_ 和_2_。这些不是监视器名称。这些是在当前monmap中建立的监视器等级。我们缺少排名为0的显示器，并且根据monmap会是`mon.a`。

顺便问一下，队伍如何建立？

> 秩是（重新）计算的，只要你添加或删除监控，并按照一个简单的规则：在**更大**的`IP:PORT`结合，**降低**秩。在这种情况下，考虑到`127.0.0.1:6789`它低于所有其余`IP:PORT`组合，`mon.a`则其等级为0。

### 最常见的显示器问题

#### 有法定人数，但至少有一个监控器已关闭

发生这种情况时，根据您所运行的Ceph的版本，您应该会看到类似于以下内容：

```text
$ ceph health detail
[snip]
mon.a (rank 0) addr 127.0.0.1:6789/0 is down (out of quorum)
```

如何解决这个问题？

> 首先，确保`mon.a`正在运行。
>
> 其次，请确保您能够`mon.a`从其他监视器的服务器连接到“服务器”。还要检查端口。检查`iptables`所有监视器节点，并确保您没有断开/拒绝连接。
>
> 如果最初的故障排除不能解决您的问题，那么该是更深入的时候了。
>
> 首先，`mon_status`按照[使用监视器的管理套接字](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/troubleshooting-mon/#using-the-monitor-s-admin-socket)和 [了解mon\_status中的](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/troubleshooting-mon/#understanding-mon-status)说明，通过管理套接字检查有问题的监视器。
>
> 考虑到监控器是仲裁的进行，它的状态应该是一个 `probing`，`electing`或`synchronizing`。如果碰巧是 `leader`或`peon`，则监视器认为是仲裁，而其余群集确定不是。或者在我们对显示器进行故障排除时，它进入了法定人数，因此请再次检查您以确保。如果监视器尚未达到法定人数，请继续。`ceph -s`

如果状态是`probing`什么？

> 这意味着该监视器仍在寻找其他监视器。每次启动监视器时，该监视器将在此状态中保持一段时间，同时尝试查找中指定的其余监视器`monmap`。监视器在此状态下花费的时间可能会有所不同。例如，在单监视器群集上时，由于周围没有其他监视器，因此该监视器将几乎立即通过探测状态。在多监视器群集上，监视器将一直保持这种状态，直到找到足够的监视器以构成仲裁为止–这意味着，如果您有3个监视器中有2个处于关闭状态，则剩下的一台监视器将无限期保持此状态，直到您将其他监视器之一。
>
> 但是，如果达到法定人数，则只要可以访问其余监视器，监视器就应该能够很快找到它们。如果您的显示器一直处于探测状态，并且您已经完成了所有的通信故障排除，那么该显示器很可能会尝试通过错误的地址到达其他显示器。`mon_status`将`monmap`已知信息输出 到监视器：检查另一台监视器的位置是否符合实际情况。如果没有，请跳至“ [恢复监视器的损坏的monmap”](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/troubleshooting-mon/#recovering-a-monitor-s-broken-monmap)；如果确实如此，则可能与监视器节点之间的严重时钟偏斜有关，您应该先参考“ [时钟偏斜”](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/troubleshooting-mon/#clock-skews)，但是，如果这不能解决您的问题，那么是时候准备一些日志并联系社区了（请参阅[准备](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/troubleshooting-mon/#preparing-your-logs)有关如何最好地准备日志的日志）。

如果状态是`electing`什么？

> 这意味着监控器正在选举中。这些应该很快完成，但是有时监视器可能会卡住选择。这通常是监视器节点之间时钟偏斜的迹象。跳转至 [Clock Skews](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/troubleshooting-mon/#clock-skews)以获得更多信息。如果所有时钟都正确同步，则最好准备一些日志并与社区联系。这不是一个可能会持续的状态，除了（_确实_）旧错误之外，除了时钟偏斜之外，没有其他明显的原因会导致这种情况的发生。

如果状态是`synchronizing`什么？

> 这意味着监视器正在与集群的其余部分进行同步，以加入仲裁。同步过程的速度与较小的监视器存储区一样快，因此，如果存储区很大，则可能需要一段时间。不用担心，它应该尽快完成。
>
> 但是，如果你注意到监视器从跳跃`synchronizing`到 `electing`，然后回`synchronizing`，那么你有一个问题：群集状态前进（即生成新地图）太快同步过程跟上。在早期的墨鱼中，这曾经是一回事，但是从那时起，同步过程就得到了充分的重构和增强，从而避免了这种行为。如果以后的版本中发生这种情况，请通知我们。并带来一些日志（请参阅[准备日志](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/troubleshooting-mon/#preparing-your-logs)）。

如果状态为`leader`或`peon`怎么办？

> 这不应该发生。但是，这种情况有可能会发生，并且与时钟偏斜有很大关系-请参阅“ [时钟偏斜”](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/troubleshooting-mon/#clock-skews)。如果您没有时钟偏斜的困扰，那么请准备您的日志（请参阅 [准备日志](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/troubleshooting-mon/#preparing-your-logs)）并与我们联系。

#### 恢复监视器的破碎MONMAP 

这是`monmap`通常的样子，具体取决于监视器的数量：

```text
epoch 3
fsid 5c4e9d53-e2e1-478a-8061-f543f8be4cf8
last_changed 2013-10-30 04:12:01.945629
created 2013-10-29 14:14:41.914786
0: 127.0.0.1:6789/0 mon.a
1: 127.0.0.1:6790/0 mon.b
2: 127.0.0.1:6795/0 mon.c
```

但是，这可能不是您所拥有的。例如，在早期乌贼的某些版本中，存在一个可能导致您`monmap` 无效的错误。完全填充零。这意味着甚至 `monmaptool`无法读取它，因为它将很难理解只有零的含义。在另一些时间，您可能最终得到了带有严重过时的monmap的监视器，因此无法找到剩余的监视器（例如，`mon.c`已关闭；添加新的监视器`mon.d`，然后删除`mon.a`，然后添加新的监视器`mon.e`并删除 `mon.b`；您最终将获得与已知的完全不同的monmap `mon.c`）。

在这种情况下，您有两种可能的解决方案：

报废显示器并创建一个新显示器

> 仅当您确定不会丢失该监视器保存的信息时，才应采用此路线；您有其他监视器，并且它们运行良好，以便您的新监视器能够与其余监视器同步。请记住，销毁监视器（如果没有其内容的其他副本）可能会导致数据丢失。

将monmap注入监视器

> 通常是最安全的路径。您应该从其余监视器中获取monmap，并将其与损坏/丢失的monmap一起注入到监视器中。
>
> 这些是基本步骤：
>
> 1. 是否有法定人数？如果是这样，请从仲裁中获取monmap：
>
>    ```text
>    $ ceph mon getmap -o /tmp/monmap
>    ```
>
> 2. 没有法定人数？直接从另一台监视器获取monmap（假定您要从其抓取monmap的监视器具有ID ID-FOO且已停止）：
>
>    ```text
>    $ ceph-mon -i ID-FOO --extract-monmap /tmp/monmap
>    ```
>
> 3. 停止将monmap注入到的监视器。
> 4. 注入monmap：
>
>    ```text
>    $ ceph-mon -i ID --inject-monmap /tmp/monmap
>    ```
>
> 5. 启动显示器
>
> 请记住，注入monmap的功能是一项强大的功能，如果使用不当，可能会对显示器造成破坏，因为它将覆盖显示器保存的最新的现有monmap。

#### 时钟偏斜

监视器可能会受到跨监视器节点的明显时钟偏差的严重影响。这通常转化为无明显原因的奇怪行为。为避免此类问题，应在监视器节点上运行时钟同步工具。

容许的最大时钟偏斜是多少？

> 默认情况下，监视器将允许时钟偏移到最大。`0.05 seconds`

我可以增加最大容许时钟偏斜吗？

> 该值是通过可配置的`mon-clock-drift-allowed`选项，虽然你_CAN_，这并不意味着你_SHOULD_。由于时钟偏斜监视器可能无法正常运行，因此存在时钟偏斜机制。作为开发人员和质量检查专家，我们对当前的默认值感到满意，因为它会在监视器失去控制之前提醒用户。更改此值而不先对其进行测试，尽管不会造成数据丢失的风险，但可能会对监视器的稳定性和整个群集的健康状况产生无法预料的影响。

我怎么知道有时钟偏斜？

> 监视器将以警告形式警告您`HEALTH_WARN`。应该以以下形式显示：`ceph health detail`
>
> ```text
> mon.c addr 10.10.0.1:6789/0 clock skew 0.08235s > max 0.05s (latency 0.0045s)
> ```
>
> 这意味着`mon.c`已被标记为存在时钟偏斜。

如果出现时钟偏斜怎么办？

> 同步时钟。运行NTP客户端可能会有所帮助。如果您已经在使用一个NTP服务器，并且已经遇到此类问题，请检查您是否正在使用远程网络上的某些NTP服务器，并考虑在网络上托管自己的NTP服务器。最后一个选项可以减少监视器时钟偏斜的问题。

#### 客户端无法连接或挂载

检查您的IP表。某些OS安装实用程序会向添加`REJECT`规则 `iptables`。该规则将拒绝所有尝试连接到主机的客户端`ssh`。如果监视器主机的IP表具有这样的`REJECT`规则，则从单独节点连接的客户端将无法挂载，并出现超时错误。您需要解决`iptables`拒绝客户端尝试连接到Ceph守护程序的规则。例如，您需要适当地解决如下所示的规则：

```text
REJECT all -- anywhere anywhere reject-with icmp-host-prohibited
```

您可能还需要向Ceph主机上的IP表添加规则，以确保客户端可以访问与Ceph监视器关联的端口（即默认情况下为端口6789）和Ceph OSD（即默认情况下为6800至7300）。例如：

```text
iptables -A INPUT -m multiport -p tcp -s {ip-address}/{netmask} --dports 6789,6800:7300 -j ACCEPT
```

### 监视存储故障

#### 商店损坏的症状

Ceph监视器将[群集映射](https://docs.ceph.com/docs/nautilus/glossary/#term-cluster-map)存储在键/值存储（例如LevelDB）中。如果监视器由于键/值存储损坏而失败，则可能在监视器日志中找到以下错误消息：

```text
Corruption: error in middle of record
```

要么：

```text
Corruption: 1 missing files; e.g.: /var/lib/ceph/mon/mon.foo/store.db/1234567.ldb
```

#### 使用健康监视器进行恢复

如果有幸存者，我们总是可以用新的幸存者[替换](https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-mons/#adding-and-removing-monitors)掉。启动后，新的加入程序将与正常运行的同级同步，一旦完全同步，便可以为客户端提供服务。

#### 使用OSD恢复

但是，如果所有监视器都同时出现故障怎么办？由于鼓励用户在Ceph集群中部署至少三台（最好是五台）监视器，因此同时发生故障的机会很少。但是，如果磁盘/ fs设置配置不正确，则数据中心内的计划外关机可能会使基础文件系统失效，从而杀死所有监视器。在这种情况下，我们可以使用OSD中存储的信息来恢复监视器存储。

```text
ms=/root/mon-store
mkdir $ms

# collect the cluster map from OSDs
for host in $hosts; do
  rsync -avz $ms/. user@host:$ms.remote
  rm -rf $ms
  ssh user@host <<EOF
    for osd in /var/lib/ceph/osd/ceph-*; do
      ceph-objectstore-tool --data-path \$osd --op update-mon-db --mon-store-path $ms.remote
    done
  EOF
  rsync -avz user@host:$ms.remote/. $ms
done

# rebuild the monitor store from the collected map, if the cluster does not
# use cephx authentication, we can skip the following steps to update the
# keyring with the caps, and there is no need to pass the "--keyring" option.
# i.e. just use "ceph-monstore-tool $ms rebuild" instead
ceph-authtool /path/to/admin.keyring -n mon. \
  --cap mon 'allow *'
ceph-authtool /path/to/admin.keyring -n client.admin \
  --cap mon 'allow *' --cap osd 'allow *' --cap mds 'allow *'
# add one or more ceph-mgr's key to the keyring. in this case, an encoded key
# for mgr.x is added, you can find the encoded key in
# /etc/ceph/${cluster}.${mgr_name}.keyring on the machine where ceph-mgr is
# deployed
ceph-authtool /path/to/admin.keyring --add-key 'AQDN8kBe9PLWARAAZwxXMr+n85SBYbSlLcZnMA==' -n mgr.x \
  --cap mon 'allow profile mgr' --cap osd 'allow *' --cap mds 'allow *'
# if your monitors' ids are not single characters like 'a', 'b', 'c', please
# specify them in the command line by passing them as arguments of the "--mon-ids"
# option. if you are not sure, please check your ceph.conf to see if there is any
# sections named like '[mon.foo]'. don't pass the "--mon-ids" option, if you are
# using DNS SRV for looking up monitors.
ceph-monstore-tool $ms rebuild -- --keyring /path/to/admin.keyring --mon-ids alpha beta gamma

# make a backup of the corrupted store.db just in case!  repeat for
# all monitors.
mv /var/lib/ceph/mon/mon.foo/store.db /var/lib/ceph/mon/mon.foo/store.db.corrupted

# move rebuild store.db into place.  repeat for all monitors.
mv $ms/store.db /var/lib/ceph/mon/mon.foo/store.db
chown -R ceph:ceph /var/lib/ceph/mon/mon.foo/store.db
```

以上步骤

1. 从所有OSD主机收集地图，
2. 然后重建商店
3. 用适当的上限填充密钥环文件中的实体
4. 用`mon.foo`恢复的副本替换损坏的存储。

**已知限制**

使用上述步骤无法恢复以下信息：

* **一些添加的密钥环**：使用命令添加的所有OSD密钥环均从OSD的副本中恢复。并使用导入密钥环。但是，MDS密钥环和其他密钥环在恢复的监视器存储中丢失。您可能需要手动重新添加它们。`ceph auth addclient.adminceph-monstore-tool`
* **创建池**：如果正在创建任何RADOS池，则该状态将丢失。恢复工具假定已创建所有池。如果部分创建的池恢复后有PG卡在“未知”中，则可以使用命令强制创建_空_ PG 。请注意，这将创建一个_空的_ PG，因此仅当您知道该池为空时，才执行此操作。`ceph osd force-create-pg`
* **MDS映射**：MDS映射丢失。

### 一切都失败了！怎么办？

#### 寻求帮助

您可以在\#ceph和＃头孢-devel的在OFTC（服务器irc.oftc.net）和上找到我们在IRC `ceph-devel@vger.kernel.org`和`ceph-users@lists.ceph.com`。确保您已抓取日志并在有人询问时准备好日志：交互速度越快，响应延迟越短，每个人的时间被优化的机会就越大。

#### 准备日志

监视器日志默认情况下保存在中`/var/log/ceph/ceph-mon.FOO.log*`。我们可能想要它们。但是，您的日志可能没有必要的信息。如果找不到监视日志的默认位置，则可以通过运行以下命令检查它们的位置：

```text
ceph-conf --name mon.FOO --show-config-value log_file
```

日志中的信息量取决于配置文件强制执行的调试级别。如果您尚未强制执行特定的调试级别，则Ceph使用的是默认级别，并且您的日志可能不包含重要信息来跟踪您的问题。将相关信息输入日志的第一步是提高调试级别。在这种情况下，我们将对监视器提供的信息感兴趣。与在其他组件上发生的情况类似，监视器的不同部分将在不同的子系统上输出其调试信息。

您将不得不提高与您的问题密切相关的那些子系统的调试级别。对于不熟悉Ceph故障排除的人来说，这可能不是一件容易的事。在大多数情况下，在监视器上设置以下选项就足以找出问题的潜在根源：

```text
debug mon = 10
debug ms = 1
```

如果我们发现这些调试级别还不够，我们可能会要求您提高它们甚至定义其他调试子系统来从中获取信息，但至少我们从一些有用的信息开始，而不是大量的空日志而没有还有很多。

#### 我是否需要重新启动监视器以调整调试级别？

不可以。您可以通过以下两种方式之一进行操作：

你有法定人数

> 将debug选项注入到要调试的监视器中：
>
> ```text
> ceph tell mon.FOO config set debug_mon 10/10
> ```
>
> 或一次进入所有监视器：
>
> ```text
> ceph tell mon.* config set debug_mon 10/10
> ```

没有法定人数

> 使用监视器的管理套接字并直接调整配置选项：
>
> ```text
> ceph daemon mon.FOO config set debug_mon 10/10
> ```

返回默认值就像使用调试级别重新运行上述命令一样容易`1/10`。您可以使用管理套接字和以下命令检查当前值：

```text
ceph daemon mon.FOO config show
```

要么：

```text
ceph daemon mon.FOO config get 'OPTION_NAME'
```

#### 重现了具有适当调试级别的问题。怎么办？

理想情况下，您只向我们发送日志的相关部分。我们认识到找出对应的部分可能不是最简单的任务。因此，如果您提供完整的日志，我们不会保留给您，但应使用常识。如果您的日志有成千上万的行，那么遍历整个过程可能会很棘手，特别是如果我们不知道在什么时候发生了问题，无论您遇到什么问题。例如，在复制时，请记住写下当前时间和日期，并据此提取日志的相关部分。

最后，您应该在IRC上的邮件列表中与我们联系，或者在[tracker](http://tracker.ceph.com/projects/ceph/issues/new)上提出新的问题。

