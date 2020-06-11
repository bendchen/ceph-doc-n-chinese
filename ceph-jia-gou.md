# CEPH架构

## 架构

[Ceph](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph)在一个统一的系统中唯一地提供**对象，块和文件存储**。Ceph高度可靠，易于管理且免费。Ceph的功能可以改变您公司的IT基础架构以及管理大量数据的能力。Ceph提供了非凡的可伸缩性-成千上万的客户端访问PB到EB的数据。一个[ Ceph节点](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-node)利用商品硬件和智能守护程序，一个[ Ceph存储集群](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-storage-cluster) 容纳大量节点，它们相互通信以动态复制和重新分配数据。![../\_images/stack.png](https://docs.ceph.com/docs/nautilus/_images/stack.png)

### 一个CEPH存储集群

Ceph提供了一个基于 RADOS的无限扩展的[Ceph存储集群](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-storage-cluster)，您可以在[RADOS中](https://ceph.com/wp-content/uploads/2016/08/weil-rados-pdsw07.pdf)阅读它[-PB级存储集群的可扩展，可靠的存储服务](https://ceph.com/wp-content/uploads/2016/08/weil-rados-pdsw07.pdf)。

Ceph存储集群由两种类型的守护程序组成：

* [Ceph监控器](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-monitor)
* [Ceph OSD守护程序](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-osd-daemon)

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-4cf6d0983521ea66cd16f98b7ce624e6666eed77.png)

Ceph监控器维护集群图的主副本。如果监视器守护程序发生故障，则Ceph监视器群集可确保高可用性。存储群集客户端从Ceph监控器检索群集映射的副本。

Ceph OSD守护程序检查其自身的状态以及其他OSD的状态，并向监视器报告。

存储集群客户端和每个[Ceph OSD守护程序都](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-osd-daemon)使用CRUSH算法来有效地计算有关数据位置的信息，而不必依赖中央查找表。Ceph的高级功能包括通过提供与Ceph存储集群的本机接口`librados`，以及在之上构建的许多服务接口`librados`。

#### 存储数据

Ceph存储群集从[Ceph客户端](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-clients)接收数据，无论它是通过[Ceph块设备](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-block-device)，[Ceph对象存储](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-object-storage)， [Ceph文件系统](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-filesystem)还是使用您创建的自定义实现 传递的`librados`，它都将数据存储为对象。每个对象对应于文件系统中的一个文件，该文件存储在[对象存储设备上](https://docs.ceph.com/docs/nautilus/glossary/#term-object-storage-device)。Ceph OSD守护程序处理存储磁盘上的读/写操作。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-518f1eba573055135eb2f6568f8b69b4bb56b4c8.png)

Ceph OSD守护程序将所有数据作为对象存储在平面命名空间中（例如，没有目录层次结构）。对象具有标识符，二进制数据和由一组名称/值对组成的元数据。语义完全取决于[Ceph Clients](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-clients)。例如，CephFS使用元数据存储文件属性，例如文件所有者，创建日期，上次修改日期等。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-ae8b394e1d31afd181408bab946ca4a216ca44b7.png)

注意 

对象ID在整个集群中都是唯一的，而不仅仅是本地文件系统。

#### 可伸缩性和高可用性

在传统体系结构中，客户端与集中式组件（例如，网关，代理，API，Facade等）进行通信，该组件充当复杂子系统的单个入口点。这在引入单点故障的同时，对性能和可伸缩性都施加了限制（即，如果集中式组件出现故障，那么整个系统也会出现故障）。

Ceph消除了集中式网关，使客户端能够直接与Ceph OSD守护程序进行交互。Ceph OSD守护进程在其他Ceph节点上创建对象副本，以确保数据安全和高可用性。Ceph还使用监视器集群来确保高可用性。为了消除集中化，Ceph使用了一种称为CRUSH的算法。

**CRUSH简介**

Ceph客户端和Ceph OSD守护程序都使用CRUSH算法来有效地计算有关对象位置的信息，而不必依赖中央查找表。与旧方法相比，CRUSH提供了更好的数据管理机制，并且通过将工作干净地分配给群集中的所有客户端和OSD守护程序来实现大规模扩展。CRUSH使用智能数据复制来确保弹性，该弹性更适合于超大规模存储。以下各节提供有关CRUSH如何工作的其他详细信息。有关CRUSH的详细讨论，请参见[CRUSH控制，可伸缩，去中心化复制数据的放置](https://ceph.com/wp-content/uploads/2016/08/weil-crush-sc06.pdf)。

**集群地图**

Ceph依赖于具有集群拓扑知识的Ceph客户端和Ceph OSD守护程序，其中包括5个映射（统称为“集群映射”）：

1. **监视器映射：**包含群集`fsid`，每个监视器的位置，名称地址和端口。它还指示当前纪元，创建地图的时间以及上次更改的时间。要查看监视器地图，请执行。`ceph mon dump`
2. **OSD映射：**包含集群`fsid`，创建和最后修改映射时，池列表，副本大小，PG号，OSD列表及其状态（例如`up`，`in`）。要查看OSD映射，请执行 。`ceph osd dump`
3. **PG地图：**包含PG版本，其时间戳，最后一个OSD地图时代，完整比例以及每个放置组的详细信息，例如PG ID，Up Set，Acting Set，PG的状态（例如， ）以及每个池的数据使用情况统计信息。`active + clean`
4. **CRUSH Map：**包含存储设备，故障域层次结构（例如，设备，主机，机架，行，房间等）的列表，以及在存储数据时遍历层次结构的规则。要查看CRUSH映射，请执行 ; 然后，通过执行反编译 。您可以在文本编辑器中或使用查看反编译的地图。`ceph osd getcrushmap -o {filename}crushtool -d {comp-crushmap-filename} -o {decomp-crushmap-filename}cat`
5. **MDS映射：**包含当前MDS映射纪元，创建映射的时间以及上次更改的时间。它还包含存储元数据，元数据服务器的列表池，以及元数据服务器`up`和`in`。要查看MDS地图，请执行。`ceph fs dump`

每个映射都维护其操作状态更改的迭代历史记录。Ceph监控器维护群集映射的主副本，包括群集成员，状态，更改以及Ceph存储群集的整体运行状况。

**高可用性监视器**

在Ceph客户端可以读取或写入数据之前，他们必须联系Ceph监视器以获取群集映射的最新副本。一个Ceph Storage Cluster可以在一个监视器上运行；但是，这会引入单点故障（即，如果监视器关闭，则Ceph客户端无法读取或写入数据）。

为了提高可靠性和容错能力，Ceph支持一系列监视器。在监视器集群中，延迟和其他故障可能导致一个或多个监视器落在集群的当前状态之后。因此，Ceph必须在各个监视实例之间就集群状态达成协议。Ceph始终使用大多数监视器（例如1、2：3、3：[5、4](https://en.wikipedia.org/wiki/Paxos_%28computer_science%29)：6等）和[Paxos](https://en.wikipedia.org/wiki/Paxos_%28computer_science%29)算法在监视器之间建立有关集群当前状态的共识。

有关配置监视器的详细信息，请参见《[监视器配置参考》](https://docs.ceph.com/docs/nautilus/rados/configuration/mon-config-ref)。

**高可用性认证**

为了识别用户并防止中间人攻击，Ceph提供了其`cephx`身份验证系统来对用户和守护程序进行身份验证。

注意 

该`cephx`协议未解决传输中的数据加密（例如SSL / TLS）或静态加密。

Cephx使用共享密钥进行身份验证，这意味着客户端和监视器群集都具有客户端密钥的副本。身份验证协议使得双方都可以彼此证明自己拥有密钥的副本，而无需实际透露它。这提供了相互认证，这意味着群集确保用户拥有密钥，而用户确保群集具有密钥的副本。

Ceph的关键可伸缩性功能是避免与Ceph对象存储的集中接口，这意味着Ceph客户端必须能够直接与OSD进行交互。为了保护数据，Ceph提供了其`cephx`身份验证系统，该系统对运行Ceph客户端的用户进行身份验证。该`cephx`协议以类似于[Kerberos的](https://en.wikipedia.org/wiki/Kerberos_%28protocol%29)行为方式运行。

用户/演员调用Ceph客户端来联系监视器。与Kerberos不同，每个监视器都可以对用户进行身份验证并分发密钥，因此使用时不会出现单点故障或瓶颈`cephx`。监视器返回类似于Kerberos票证的身份验证数据结构，其中包含用于获取Ceph服务的会话密钥。此会话密钥本身已使用用户的永久秘密密钥加密，因此只有用户可以从Ceph监视器请求服务。然后，客户端使用会话密钥从监视器请求其所需的服务，并且监视器向客户端提供票证，该票证将向实际处理数据的OSD认证客户端。Ceph监视器和OSD共享一个秘密，因此客户端可以将监视器提供的票证与群集中的任何OSD或元数据服务器一起使用。像Kerberos一样，`cephx`票证过期，因此攻击者无法使用秘密获得的过期票证或会话密钥。这种身份验证形式将防止有权访问通信介质的攻击者以其他用户的身份创建虚假消息，或更改其他用户的合法消息，只要该用户的密钥在过期之前不被泄露即可。

要使用`cephx`，管理员必须首先设置用户。在下图中，`client.admin`用户从命令行调用 以生成用户名和密钥。Ceph的 子系统生成用户名和密钥，将其副本与监视器一起存储，并将用户的机密发送回用户。这意味着客户端和监视器共享一个密钥。`ceph auth get-or-create-keyauthclient.admin`

注意 

该`client.admin`用户必须提供用户ID和秘密密钥的用户以安全的方式。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-6b1dafb6d8f177ab2beb3325857f1e98e4593ec6.png)

为了向监视器进行身份验证，客户端将用户名传递给监视器，监视器将生成一个会话密钥，并使用与该用户名关联的秘密密钥对其进行加密。然后，监视器将加密的票证发送回客户端。然后，客户端使用共享密钥解密有效负载以检索会话密钥。会话密钥标识当前会话的用户。然后，客户端代表由会话密钥签名的用户请求票证。监视器生成票证，使用用户的密钥对其进行加密，然后将其发送回客户端。客户端解密票证，并使用它对整个集群中的OSD和元数据服务器签署请求。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-56e3a72e085f9070289331d64453b84ab1e9510b.png)

该`cephx`协议对客户端计算机和Ceph服务器之间正在进行的通信进行身份验证。初始身份验证之后，在客户端和服务器之间发送的每个消息都使用票证进行签名，监视器，OSD和元数据服务器可以使用其共享密码进行验证。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-f97566f2e17ba6de07951872d259d25ae061027f.png)

此身份验证提供的保护在Ceph客户端和Ceph服务器主机之间。身份验证不会扩展到Ceph客户端之外。如果用户从远程主机访问Ceph客户端，则Ceph身份验证不会应用于用户主机和客户端主机之间的连接。

有关配置的详细信息，请参阅《[Cephx配置指南》](https://docs.ceph.com/docs/nautilus/rados/configuration/auth-config-ref)。有关用户管理的详细信息，请参阅[用户管理](https://docs.ceph.com/docs/nautilus/rados/operations/user-management)。

**智能守护程序启用超大规模**

在许多集群体系结构中，集群成员资格的主要目的是使集中式接口知道其可以访问哪些节点。然后，集中式接口通过双重调度为客户端提供服务，这是PB级到EB级的**巨大**瓶颈。

Ceph消除了瓶颈：Ceph的OSD守护进程和Ceph客户端都知道群集。像Ceph客户端一样，每个Ceph OSD守护程序都知道集群中的其他Ceph OSD守护程序。这使Ceph OSD守护程序可以直接与其他Ceph OSD守护程序和Ceph监视器进行交互。此外，它使Ceph客户端可以直接与Ceph OSD守护程序进行交互。

Ceph客户端，Ceph监视器和Ceph OSD守护程序相互交互的能力意味着Ceph OSD守护程序可以利用Ceph节点的CPU和RAM轻松执行使集中式服务器陷入困境的任务。利用这种计算能力的能力带来了几个主要好处：

1. **OSD直接为客户服务：**由于任何网络设备都对其支持的并发连接数有限制，因此集中式系统在大规模上的物理限制较低。通过使Ceph客户端直接联系Ceph OSD守护程序，Ceph可以同时提高性能和系统总容量，同时消除单点故障。Ceph客户端可以在需要时维护会话，并使用特定的Ceph OSD守护进程而不是集中式服务器维护会话。
2. **OSD成员资格和状态**：Ceph OSD守护程序加入集群并报告其状态。在最低级别，Ceph OSD守护程序状态为`up` 或`down`反映它是否正在运行并能够满足Ceph客户端请求。如果Ceph的OSD守护程序`down`和`in`一个Ceph存储集群，这种状态可能表明一个Ceph OSD守护进程的失败。如果Ceph OSD守护进程未运行（例如，崩溃），则Ceph OSD守护进程无法通知Ceph监视器它正在运行`down`。OSD定期将消息发送到Ceph监视器（`MPGStats`发光前，`MOSDBeacon`在发光）。如果Ceph Monitor在一段可配置的时间后没有看到该消息，则它将OSD标记为down。但是，此机制是故障安全的。通常，Ceph OSD守护程序将确定相邻的OSD是否已关闭并将其报告给Ceph监视器。这确保了Ceph Monitors是轻量级进程。有关其他详细信息，请参见[监视OSD](https://docs.ceph.com/docs/nautilus/rados/operations/monitoring-osd-pg/#monitoring-osds)和[心跳](https://docs.ceph.com/docs/nautilus/rados/configuration/mon-osd-interaction)。
3. **数据清理：**作为保持数据一致性和整洁性的一部分，Ceph OSD守护进程可以**清理**放置组中的对象。也就是说，Ceph OSD守护程序可以将一个放置组中的对象元数据与其存储在其他OSD上的放置组中的副本进行比较。清理（​​通常每天执行一次）会捕获错误或文件系统错误。Ceph OSD守护程序还通过逐位比较对象中的数据来执行更深层的清理。深度清理（通常每周执行一次）会发现驱动器上的坏扇区在轻清理中不明显。有关配置清理的详细信息，请参见[数据](https://docs.ceph.com/docs/nautilus/rados/configuration/osd-config-ref#scrubbing)清理。
4. **复制：**与Ceph客户端一样，Ceph OSD守护程序也使用CRUSH算法，但是Ceph OSD守护程序使用它来计算对象副本的存储位置（并进行重新平衡）。在典型的写场景中，客户端使用CRUSH算法计算对象的存储位置，将对象映射到池和放置组，然后查看CRUSH映射以识别放置组的主要OSD。

   客户端将对象写入主OSD中标识的放置组。然后，具有自己的CRUSH映射副本的主OSD标识用于复制目的的辅助OSD和第三OSD，并将对象复制到辅助OSD和第三OSD中的适当放置组（与其他副本一样多的OSD），并响应客户端一旦确认对象已成功存储。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-54719cc959473e68a317f6578f9a2f0f3a8345ee.png)

Ceph OSD守护程序具有执行数据复制的功能，可减轻Ceph客户端的职责，同时确保高数据可用性和数据安全性。

#### 动态群集管理

在“ [可伸缩性和高可用性”](https://docs.ceph.com/docs/nautilus/architecture/#scalability-and-high-availability)部分，我们解释了Ceph如何使用CRUSH，集群感知和智能守护程序来扩展和维护高可用性。Ceph设计的关键是自主，自我修复和智能的Ceph OSD守护程序。让我们更深入地了解CRUSH如何使现代云存储基础架构能够放置数据，重新平衡集群并动态地从故障中恢复。

**关于池**

Ceph存储系统支持“池”的概念，“池”是用于存储对象的逻辑分区。

Ceph客户端从Ceph监控器检索[集群映射](https://docs.ceph.com/docs/nautilus/architecture/#cluster-map)，并将对象写入池中。池的`size`副本数或副本数，CRUSH规则和放置组的数量决定了Ceph将如何放置数据。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-778c68aa73d182d88236b73e1a4a40bd78fb32d6.png)

池至少设置以下参数：

* 所有权/对对象的访问
* 展示位置组数，以及
* 使用的CRUSH规则。

有关详细信息，请参见[设置池值](https://docs.ceph.com/docs/nautilus/rados/operations/pools#set-pool-values)。

**前列腺素映射到屏上显示**

每个池都有许多放置组。CRUSH将PG动态映射到OSD。当Ceph客户端存储对象时，CRUSH会将每个对象映射到一个放置组。

将对象映射到放置组会在Ceph OSD守护程序和Ceph客户端之间创建一个间接层。Ceph存储群集必须能够增长（或收缩）并在动态存储对象的地方重新平衡。如果Ceph客户端“知道”哪个Ceph OSD守护程序具有哪个对象，那将在Ceph客户端和Ceph OSD守护程序之间建立紧密的联系。相反，CRUSH算法将每个对象映射到一个放置组，然后将每个放置组映射到一个或多个Ceph OSD守护程序。当新的Ceph OSD守护进程和底层OSD设备联机时，此间接层允许Ceph动态重新平衡。下图描述了CRUSH如何将对象映射到放置组，以及如何将放置组映射到OSD。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-c7fd5a4042a21364a7bef1c09e6b019deb4e4feb.png)

借助群集映射和CRUSH算法的副本，客户端可以准确计算在读取或写入特定对象时要使用的OSD。

**计算PG** 

当Ceph客户端绑定到Ceph监视器时，它将检索[集群映射](https://docs.ceph.com/docs/nautilus/architecture/#cluster-map)的最新副本 。使用群集映射，客户端可以了解群集中的所有监视器，OSD和元数据服务器。**但是，它对对象位置一无所知。**

> 计算对象位置。

客户端所需的唯一输入是对象ID和池。很简单：Ceph将数据存储在命名池中（例如“ liverpool”）。当客户要存储命名对象（例如“ john”，“ paul”，“ george”，“ ringo”等）时，它将使用对象名称，哈希码和其中的PG数量来计算展示位置组池和池名称。Ceph客户端使用以下步骤来计算PG ID。

1. 客户端输入池名称和对象ID。（例如，pool =“ liverpool”，object-id =“ john”）
2. Ceph获取对象ID并对其进行哈希处理。
3. Ceph以PG的数量为模来计算哈希值。（例如`58`）以获取PG ID。
4. Ceph获取给定池名称的池ID（例如，“ liverpool” = `4`）
5. Ceph将池ID附加到PG ID（例如`4.58`）之前。

计算对象位置比通过健谈会话执行对象位置查询要快得多。所述CRUSH算法允许客户端计算，其中对象_应该_被存储，并且使得客户端能够联系主OSD存储或检索的对象。

**对等和集合**

在前面的部分中，我们注意到Ceph OSD守护程序会互相检查心跳并报告给Ceph Monitor。Ceph OSD守护进程所做的另一件事称为“对等”，这是使存储一个放置组（PG）的所有OSD对该PG中所有对象（及其元数据）的状态达成一致的过程。实际上，Ceph OSD守护进程向Ceph监视器[报告对等失败](https://docs.ceph.com/docs/nautilus/rados/configuration/mon-osd-interaction#osds-report-peering-failure)。对等问题通常可以解决。但是，如果问题仍然存在，则可能需要参考“ [对等失败故障排除”](https://docs.ceph.com/docs/nautilus/rados/troubleshooting/troubleshooting-pg#placement-group-down-peering-failure)部分。

注意 

同意状态并不意味着PG具有最新的内容。

Ceph存储集群旨在存储至少两个对象的副本（即），这是数据安全的最低要求。为了实现高可用性，Ceph存储群集应存储一个对象（例如和）的两个以上副本，以便它可以在保持数据安全性的状态下继续运行 。`size = 2size = 3min size = 2degraded`

返回参照图[智能守护程序启用超大规模](https://docs.ceph.com/docs/nautilus/architecture/#smart-daemons-enable-hyperscale)，我们不命名Ceph的OSD守护专（如`osd.0`，`osd.1`等），而是把它们称为_原发性_，_继发_，等等。按照惯例，_主要_是在第一OSD _代理设置_，并负责协调它充当各贴装组的对等过程中_主_，而且是**唯一的** OSD，这将接受客户端发起的写入对象的给定的展示位置组（充当_主要）_。

当一系列OSD负责一个放置组时，我们将该系列OSD称为“ _代理集”_。一个_代理设置_可参考Ceph的OSD守护程序是目前负责安置组，或Ceph的OSD守护进程负责一个特定的贴装组的一些时代。

_代理集的_一部分的Ceph OSD守护进程可能并非始终是 `up`。当一个OSD _代理设置_是`up`，它是的一部分 _时置_。该_时置_是一个重要的区别，因为当OSD失败Ceph的可以重新映射的PG到其他Ceph的OSD守护进程。

注意 

在一个_代理集_用于容纳PG `osd.25`，`osd.32`和 `osd.61`，第一OSD， `osd.25`，是_主_。如果OSD出现故障，辅助，`osd.32`，成为_小学_，并`osd.25`会从被删除_时置_。

**再平衡**

当您将Ceph OSD守护程序添加到Ceph存储群集时，群集映射将使用新的OSD更新。再次参考[计算PG ID](https://docs.ceph.com/docs/nautilus/architecture/#calculating-pg-ids)，这将更改群集映射。因此，它更改了对象放置，因为它更改了计算的输入。下图描述了重新平衡过程（尽管相当粗略，因为它对大型集群的影响较小），其中一些但不是全部PG从现有OSD（OSD 1和OSD 2）迁移到新OSD（OSD 3） ）。即使重新平衡，破碎也很稳定。许多放置组保持其原始配置，并且每个OSD都有一些增加的容量，因此重新平衡完成后，新OSD上不会出现负载峰值。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-b31e1f646135b9706000fa0799d572563dffac81.png)

**数据一致性**

作为保持数据一致性和整洁度的一部分，Ceph OSD还可以清理放置组中的对象。也就是说，Ceph OSD可以将一个放置组中的对象元数据与其存储在其他OSD中的放置组中的副本进行比较。清理（​​通常每天执行一次）会捕获OSD错误或文件系统错误。OSD还可以通过逐位比较对象中的数据来执行更深入的清理。深度清理（通常每周执行一次）会发现磁盘上的坏扇区在轻度清理中不明显。

有关配置清理的详细信息，请参见[数据](https://docs.ceph.com/docs/nautilus/rados/configuration/osd-config-ref#scrubbing)清理。

#### 删除编码

擦除编码池将每个对象存储为`K+M`块。它分为 `K`数据块和`M`编码块。池的大小配置为，`K+M`以便每个块都存储在代理集中的OSD中。块的等级被存储为对象的属性。

例如，创建一个擦除编码池以使用五个OSD（）并维持其中两个OSD（）的丢失。`K+M = 5M = 2`

**读写编码块**

将包含**NYAN**的对象`ABCDEFGHI`写入池时，擦除编码功能只需将内容分为三部分即可将内容分为三个数据块：第一个包含`ABC`，第二个`DEF`和最后一个`GHI`。如果内容长度不是的倍数，则将填充内容`K`。该函数还创建两个编码块：第四个带有`YXY` ，第五个带有`QGC`。每个块都存储在代理集中的OSD中。这些块存储在具有相同名称（**NYAN**）但位于不同OSD上的对象中。`shard_t`除了名称之外，必须保留创建块的顺序并将其存储为对象（）的属性。块1包含`ABC`并且存储在**OSD5上，**而块4包含 `YXY`并存储在**OSD3上**。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-96fe8c3c73e5e54cf27fa8a4d64ed08d17679ba3.png)

从擦除编码池中读取对象**NYAN时**，解码功能将读取三个块：包含的块1，包含的`ABC`块3 `GHI`和包含的 块4 `YXY`。然后，它重建对象的原始内容`ABCDEFGHI`。通知解码功能缺少块2和5（它们称为“擦除”）。由于**OSD4**耗尽，无法读取块5 。一旦读取了三个块，就可以调用解码功能：**OSD2**是最慢的，并且不考虑其块。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-1f3acf28921568db86bb22bb748cbf42c9db7059.png)

**中断的完整写操作**

在擦除编码池中，向上集合中的主OSD接收所有写操作。它负责将有效负载编码为`K+M`块并将其发送到其他OSD。它还负责维护放置组日志的权威版本。

在下图中，已使用三个OSD 创建了一个擦除编码的放置组，并为其提供 支持，两个OSD 和一个 OSD 。放置组的动作集由**OSD 1**，**OSD 2**和 **OSD 3组成**。对象已被编码并存储在OSD中：块 （即数据块编号1，版本1）在**OSD 1上**，在 **OSD 2上，**以及（即，编码块编号1，版本1）在**OSD 3上**。每个OSD上的放置组日志都是相同的（即对于时期1，版本1）。`K = 2 + M = 1KMD1v1D2v1C1v11,1`

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-a60e808835cf8860e19b9f2a9c83691c2a4f0218.png)

**OSD 1**是主要对象，它从客户端接收**WRITE FULL**（写满），这意味着有效负载将完全替换对象，而不是覆盖对象的一部分。创建对象的版本2（v2）以覆盖版本1（v1）。**OSD 1**将有效载荷编码为三个块：（`D1v2`即，数据块编号1版本2）将在**OSD 1上**，`D2v2`在**OSD 2上，**以及 `C1v2`（即，编码块编号1版本2）在**OSD 3上。**。每个块都发送到目标OSD，包括主要OSD，该主OSD除了处理写操作和维护放置组日志的权威版本外，还负责存储块。当OSD收到指示其写入块的消息时，它还会在放置组日志中创建一个新条目以反映更改。例如，一旦**OSD 3**存储 `C1v2`，它就会将条目`1,2`（即epoch 1，版本2）添加到其日志中。由于OSD是异步工作的，因此某些块可能仍在运行中（例如`D2v2`），而其他一些块已经被确认并在磁盘上（例如`C1v1`和 `D1v1`）。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-513e0558c5877884d43ffc9e7b792a5f77466831.png)

如果一切顺利，则在动作集中的每个OSD上确认这些块，并且日志的`last_complete`指针可以从`1,1`移至`1,2`。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-8db474f2d1f9a795067c4aef26c0530072cfa77f.png)

最后，可以删除用于存储对象先前版本的块的文件：`D1v1`在**OSD 1上**，`D2v1`在**OSD 2上**和`C1v1` 在**OSD 3上**。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-4f9c14e4c28edb4b34e0eaa33253231a4d96b11f.png)

但是意外发生了。如果**OSD 1**`D2v2`仍在飞行中而下降，则对象的版本2将被部分写入：**OSD 3**有一个块，但不足以恢复。它丢失了两个块：`D1v2`和`D2v2`和擦除编码参数，要求至少有两个块可用于重建第三个块。**OSD 4**成为新的主要日志，并且发现日志条目（即，已知该条目之前的所有对象在以前的操作集中的所有OSD上都可用）是并且它将成为新的权威日志的头。`K = 2M = 1last_complete1,1`

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-e211bf856cdbbf5d055980e95d39a4b60113c954.png)

在**OSD 3**上找到的日志条目1,2 与**OSD 4**提供的新权威日志不同：将其丢弃，并`C1v2` 删除包含块的文件。所述`D1v1`块与所述重建`decode`擦洗期间擦除编码文库的功能，并存储在新的主 **OSD 4**。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-77b8a9b262ce5e9cbd7030c5da9ed7ab0edffc8a.png)

有关更多详细信息，请参见[擦除代码注释](https://github.com/ceph/ceph/blob/40059e12af88267d0da67d8fd8d9cd81244d8f93/doc/dev/osd_internals/erasure_coding/developer_notes.rst)。

#### 高速缓存分层

缓存层为Ceph客户端提供了更好的I / O性能，用于存储在后备存储层中的部分数据。缓存分层涉及创建配置为充当缓存层的相对较快/昂贵的存储设备（例如，固态驱动器）的池，以及配置为充当经济存储的擦除编码或相对较慢/更便宜的设备的后备池层。Ceph反对者处理放置对象的位置，并且分层代理确定何时将对象从缓存刷新到后备存储层。因此，缓存层和后备存储层对Ceph客户端是完全透明的。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-2982c5ed3031cac4f9e40545139e51fdb0b33897.png)

有关其他详细信息，请参见[缓存分层](https://docs.ceph.com/docs/nautilus/rados/operations/cache-tiering)。

#### 扩展头孢

您可以通过创建称为“ Ceph类”的共享对象类来扩展Ceph。Ceph 动态地（即默认情况下）加载`.so`存储在目录中的类。实现类时，可以创建能够调用Ceph Object Store中的本机方法的新对象方法，或通过库合并或创建自己的其他类方法。`osd class dir$libdir/rados-classes`

写入时，Ceph类可以调用本机或类方法，对入站数据执行任何一系列操作，并生成结果写入事务，Ceph将自动应用该事务。

读取时，Ceph类可以调用本机或类方法，对出站数据执行任何一系列操作，然后将数据返回给客户端。

Ceph类示例

内容管理系统的Ceph类可以显示特定大小和纵横比的图片，可以拍摄入站位图图像，将其裁剪为特定的纵横比，调整其大小并嵌入不可见的版权或水印以帮助保护知识产权；然后，将生成的位图图像保存到对象存储中。

有关示例性实现`src/objclass/objclass.h`，请参见`src/fooclass.cc`和`src/barclass`。

#### 总结

Ceph存储集群是动态的，就像生物一样。鉴于许多存储设备无法完全利用典型商品服务器的CPU和RAM，而Ceph却可以。从心跳，对等，重新平衡群集或从故障中恢复，Ceph都从客户端（以及从Ceph架构中不存在的集中式网关）卸载工作，并使用OSD的计算能力来执行工作。在参考《[硬件建议》](https://docs.ceph.com/docs/nautilus/start/hardware-recommendations)和《[网络配置参考》时](https://docs.ceph.com/docs/nautilus/rados/configuration/network-config-ref)，请了解上述概念，以了解Ceph如何利用计算资源。

### CEPH协议

Ceph客户端使用本机协议与Ceph存储集群进行交互。Ceph将此功能打包到`librados`库中，以便您可以创建自己的自定义Ceph客户端。下图描述了基本架构。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-1a91351293f441ce0238c21f2c432331a0f5a9d3.png)

#### 本机协议和`LIBRADOS`

现代应用程序需要具有异步通信功能的简单对象存储接口。Ceph存储集群提供了具有异步通信功能的简单对象存储接口。该接口提供对整个集群中对象的直接，并行访问。

* 池操作
* 快照和写时复制克隆
* 读取/写入对象-创建或删除-整个对象或字节范围-追加或截断
* 创建/设置/获取/删除XATTR
* 创建/设置/获取/删除键/值对
* 复合运算和双重确认语义
* 对象类

#### 对象监视/通知

客户端可以向对象注册持久兴趣，并使与主要OSD的会话保持打开状态。客户端可以向所有观察者发送通知消息和有效负载，并在观察者收到通知时接收通知。这使客户端可以将任何对象用作同步/通信通道。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-afd50e13a81128d0a2c38fadcd27dfc8b7ac523b.png)

#### 数据条带化

存储设备具有吞吐量限制，这会影响性能和可伸缩性。因此，存储系统通常支持[条带化](https://en.wikipedia.org/wiki/Data_striping)（在多个存储设备之间存储顺序的信息），以提高吞吐量和性能。数据条带化最常见的形式来自[RAID](https://en.wikipedia.org/wiki/RAID)。与Ceph的条带化最相似的RAID类型是[RAID 0](https://en.wikipedia.org/wiki/RAID_0#RAID_0)或“条带化卷”。Ceph的条带化提供RAID 0条带化的吞吐量，n路RAID镜像的可靠性和更快的恢复速度。

Ceph提供三种客户端：Ceph块设备，Ceph文件系统和Ceph对象存储。Ceph客户端将其数据从提供给用户的表示格式（块设备映像，RESTful对象，CephFS文件系统目录）转换为对象，以存储在Ceph存储集群中。

小费 

Ceph存储集群中Ceph存储的对象没有条带化。Ceph对象存储，Ceph块设备和Ceph文件系统在多个Ceph存储集群对象上分条数据。通过直接写入Ceph存储群集的Ceph客户端`librados`必须自己执行分拆（和并行I / O）操作才能获得这些好处。

最简单的Ceph条带化格式涉及1个对象的条带数。Ceph客户端将条带单元写入Ceph存储群集对象，直到该对象达到最大容量，然后为另一个数据条带创建另一个对象。条带化的最简单形式对于小型块设备映像，S3或Swift对象以及CephFS文件可能就足够了。但是，这种简单的形式并没有最大程度地利用Ceph在各个展示位置组之间分布数据的能力，因此并不能显着提高性能。下图描述了最简单的条带化形式：

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-deb861a26cf89e008006b63d95885b4ed88ba608.png)

如果您预期较大的图像大小，较大的S3或Swift对象（例如，视频）或较大的CephFS目录，则可以通过在一个对象集中的多个对象上剥离客户端数据来看到可观的读/写性能改进。当客户端将条带单元并行写入其对应的对象时，将产生显着的写入性能。由于对象被映射到不同的放置组并进一步映射到不同的OSD，因此每次写入均以最大写入速度并行发生。对单个磁盘的写操作将受到该设备的磁头移动（例如，每次搜索6毫秒）和带宽（例如100MB / s）的限制。

注意 

条带化与对象副本无关。由于CRUSH跨OSD复制对象，因此条带会自动复制。

在下图中，客户数据在由4个对象组成的对象集（在下图中）中进行条带化，其中第一个条带单元位于中，第四个条带单元位于中。写入第四个条带后，客户端确定对象集是否已满。如果对象集不完整，则客户端将再次开始将条带写入第一个对象（如下图所示）。如果对象集已满，则客户端将创建一个新的对象集（如下图所示），并开始写入新对象集（下图中的第一个对象）中的第一个条带（）。`object set 1stripe unit 0object 0stripe unit 3object 3object 0object set 2stripe unit 16object 4`

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-92220e0223f86eb33cfcaed4241c6680226c5ce2.png)

三个重要变量决定了Ceph如何划分数据：

* **对象大小：** Ceph存储群集中的对象具有最大可配置大小（例如2MB，4MB等）。对象大小应足够大以容纳许多条带单元，并且应为条带单元的倍数。
* **条带宽度：条带**具有可配置的单位大小（例如64kb）。Ceph客户端将将要写入对象的数据划分为大小相等的条带单元，最后一个条带单元除外。条带宽度应为“对象大小”的一部分，以便一个对象可以包含许多条带单元。
* **条带计数：** Ceph客户端在条带计数确定的一系列对象上写入条带单元序列。对象系列称为对象集。Ceph客户端写入对象集中的最后一个对象后，它返回对象集中的第一个对象。

重要 

在将群集投入生产之前，请测试条带化配置的性能。条带化数据并将其写入对象后，您将无法更改这些条带化参数。

一旦Ceph Client将数据条带化到条带单元并将条带单元映射到对象，Ceph的CRUSH算法就将对象映射到放置组，并将放置组映射到Ceph OSD守护程序，然后将对象作为文件存储在存储磁盘上。

注意 

由于客户端写入单个池，因此将条带化为对象的所有数据映射到同一池中的放置组。因此，他们使用相同的CRUSH映射和相同的访问控制。

### CEPH客户端

Ceph客户端包括许多服务接口。这些包括：

* **块设备：**该[Ceph的块设备](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-block-device)（又名，RBD）服务提供快照和克隆可调整大小，精简配置块的装置。Ceph在整个群集上对块设备进行条带化以实现高性能。Ceph支持内核对象（KO）和`librbd`直接使用的QEMU管理程序-避免了虚拟化系统的内核对象开销。
* **对象存储：**在[Ceph的对象存储](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-object-storage)（又名，RGW）服务提供RESTful API中有能与Amazon S3和OpenStack的斯威夫特兼容的接口。
* **文件系统**：[Ceph文件系统](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-filesystem)（CephFS）服务提供了与POSIX兼容的文件系统，可与`mount`用户空间（FUSE）中的文件系统一起使用或用作文件系统。

Ceph可以运行OSD，MDS的其他实例，并监视可伸缩性和高可用性。下图描述了高级体系结构。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-804f45fe5a789fa161d7b4100740adf992a0dc07.png)

#### CEPH的对象存储

Ceph对象存储守护程序，`radosgw`是一项FastCGI服务，它提供[RESTful](https://en.wikipedia.org/wiki/RESTful) HTTP API来存储对象和元数据。它以自己的数据格式在Ceph存储群集的顶层，并维护自己的用户数据库，身份验证和访问控制。RADOS网关使用统一的名称空间，这意味着您可以使用OpenStack Swift兼容的API或Amazon S3兼容的API。例如，您可以在一个应用程序中使用兼容S3的API写入数据，然后在另一个应用程序中使用兼容Swift的API读取数据。

S3 / Swift对象和存储群集对象的比较

Ceph的对象存储使用术语_对象_来描述其存储的数据。S3和Swift对象与Ceph写入Ceph存储集群的对象不同。Ceph对象存储对象被映射到Ceph存储集群对象。S3和Swift对象不一定与存储集群中存储的对象以1：1方式对应。S3或Swift对象可能映射到多个Ceph对象。

有关详细信息，请参见[Ceph对象存储](https://docs.ceph.com/docs/nautilus/radosgw/)。

#### CEPH块设备

Ceph块设备在Ceph存储集群中的多个对象上划分块设备映像，其中每个对象都映射到一个放置组并进行分布，并且放置组分布在`ceph-osd` 整个群集的各个守护进程中。

重要 

条带化使RBD块设备的性能优于单个服务器！

精简配置的快照式Ceph块设备是虚拟化和云计算的一个有吸引力的选择。在虚拟机场景中，人们通常使用`rbd`QEMU / KVM中的网络存储驱动程序部署Ceph块设备，在此主机将主机用于`librbd`向来宾提供块设备服务。许多云计算堆栈用于`libvirt`与管理程序集成。您可以将精简配置的Ceph块设备与QEMU一起使用， `libvirt`并在其他解决方案中支持OpenStack和CloudStack。

虽然我们目前不提供`librbd`其他管理程序的支持，但您也可以使用Ceph块设备内核对象为客户端提供块设备。其他虚拟化技术（例如Xen）可以访问Ceph块设备内核对象。这是通过命令行工具完成的`rbd`。

#### 头孢文件系统

Ceph文件系统（CephFS）提供了POSIX兼容文件系统作为服务，该系统位于基于对象的Ceph存储群集之上。CephFS文件被映射到Ceph存储在Ceph存储集群中的对象。Ceph客户端在用户空间（FUSE）中将CephFS文件系统挂载为内核对象或文件系统。

![](https://docs.ceph.com/docs/nautilus/_images/ditaa-9f5c6a9a2a7cc576498e3922cb03fafed4347fb9.png)

Ceph文件系统服务包括与Ceph存储群集一起部署的Ceph元数据服务器（MDS）。MDS的目的是将所有文件系统元数据（目录，文件所有权，访问模式等）存储在元数据驻留在内存中的高可用性Ceph元数据服务器中。究其原因，MDS（被称为守护进程`ceph-mds`）是象列出一个目录或更改目录简单的文件系统操作（`ls`，`cd`）将征税Ceph的OSD守护程序不必要的。因此，将元数据与数据分开意味着Ceph文件系统可以提供高性能服务，而无需增加Ceph存储集群的负担。

CephFS将元数据与数据分离，将元数据存储在MDS中，并将文件数据存储在Ceph存储群集中的一个或多个对象中。Ceph文件系统旨在实现POSIX兼容性。`ceph-mds`可以作为单个进程运行，也可以分发到多个物理机，以实现高可用性或可伸缩性。

* **高可用性**：额外的`ceph-mds`实例可以待命，随时准备接管的任何失败的职责`ceph-mds`，这是 积极的。这很容易，因为所有数据（包括日志）都存储在RADOS上。转换由自动触发`ceph-mon`。
* **可伸缩性**：多个`ceph-mds`实例可以处于活动状态，它们会将目录树拆分为子树（以及单个繁忙目录的分片），从而有效平衡所有活动 服务器之间的负载。

备用和活动等的组合是可能的，例如，运行3个活动 `ceph-mds`实例以进行扩展，并运行一个备用 实例以实现高可用性。

