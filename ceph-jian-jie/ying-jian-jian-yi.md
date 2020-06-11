# 硬件建议

硬件建议

Ceph被设计为在商品硬件上运行，这使得在经济上构建和维护PB级数据集群成为可能。在规划群集硬件时，您需要权衡许多注意事项，包括故障域和潜在的性能问题。硬件规划应包括在许多主机之间分发Ceph守护进程和其他使用Ceph的进程。通常，我们建议在为该类型的守护程序配置的主机上运行特定类型的Ceph守护程序。我们建议将其他主机用于利用您的数据集群的进程（例如，OpenStack，CloudStack等）。

_提示： 也可以查看_[_Ceph博客_](https://ceph.com/community/blog/)_。_

### CPU 

Ceph元数据服务器动态地重新分配其负载，这会占用大量CPU。因此，您的元数据服务器应具有强大的处理能力（例如，四核或更好的CPU）。Ceph OSD运行[RADOS](https://docs.ceph.com/docs/nautilus/glossary/#term-rados)服务，使用[CRUSH](https://docs.ceph.com/docs/nautilus/glossary/#term-crush)计算数据放置，复制数据并维护自己的集群图副本。因此，OSD应具有合理数量的处理能力（例如，双核处理器）。监控器仅维护集群映射的主副本，因此它们不占用大量CPU。您还必须考虑主机除Ceph守护程序外是否还将运行CPU密集型进程。例如，如果您的主机将运行计算VM（例如，OpenStack Nova），则需要确保这些其他进程为Ceph守护程序留有足够的处理能力。我们建议在单独的主机上运行其他占用大量CPU的进程。

### RAM 

通常，内存越多越好。

#### 监视器和管理器（CEPH-MON和CEPH-MGR）

监视程序和管理程序守护程序的内存使用量通常随群集的大小而扩展。对于小型群集，通常1-2 GB就足够了。对于大型群集，您应该提供更多（5-10 GB）。您可能还需要考虑调整设置，例如`mon_osd_cache_size`或 `rocksdb_cache_size`。

#### 元数据服务器（CEPH-MDS）

元数据守护程序的内存利用率取决于其缓存配置为消耗多少内存。对于大多数系统，我们建议最小为1 GB。请参阅 `mds_cache_memory`。

#### OSD（CEPH-OSD）

默认情况下，使用BlueStore后端的OSD需要3-5 GB的RAM。您可以`osd_memory_target`在使用BlueStore时通过配置选项来调整OSD占用的内存量。使用旧版FileStore后端时，操作系统页面缓存用于缓存数据，因此通常不需要进行调整，并且OSD内存消耗通常与系统中每个守护程序的PG数量有关。

### 数据存储

仔细计划数据存储配置。在计划数据存储时，需要考虑大量的成本和性能折衷。同时执行OS操作，以及从多个守护程序同时请求对单个驱动器的读写操作，可能会大大降低性能。

**重要 ：由于Ceph必须先将所有数据写入日志，然后才能发送ACK（至少对于XFS），因此平衡日志和OSD性能非常重要！**

#### 硬盘驱动器

OSD应该具有足够的硬盘驱动器空间来存储对象数据。我们建议最小硬盘驱动器大小为1 TB。考虑更大磁盘的每GB成本优势。我们建议将硬盘驱动器的价格除以千兆字节的数量，以得出每千兆字节的成本，因为更大的驱动器可能会对每千兆字节的成本产生重大影响。例如，一个定价为75.00美元的1 TB硬盘的成本为每GB 0.07美元（即75美元/ 1024 = 0.0732）。相比之下，定价为150.00美元的3 TB硬盘的成本为每GB 0.05美元（即150美元/ 3072 = 0.0488）。在前面的示例中，使用1 TB磁盘通常会使每GB的成本增加40％，从而使群集的成本效率大大降低。而且，存储驱动器容量越大，每个Ceph OSD守护程序所需的内存更多，尤其是在重新平衡，回填和恢复期间。一般的经验法则是1TB的存储空间需要约1GB的RAM。

_建议：在单个磁盘上运行多个OSD（与分区无关） **不是**一个好主意。_

_建议：运行OSD和监视器或单个磁盘，无论在元数据服务器的分区，是**不是**一个好主意，无论是。_

存储驱动器受到查找时间，访问时间，读写时间以及总吞吐量的限制。这些物理限制会影响整个系统的性能，尤其是在恢复过程中。我们建议为操作系统和软件使用专用的驱动器，并为主机上运行的每个Ceph OSD守护程序使用一个驱动器。大多数“ OSD慢”问题是由于在同一驱动器上运行操作系统，多个OSD和/或多个日志而引起的。由于对小型群集上的性能问题进行故障排除的成本可能超出了额外磁盘驱动器的成本，因此您可以避免对OSD存储驱动器施加过多负担的诱惑，从而加快群集设计计划。

您可以在每个硬盘驱动器上运行多个Ceph OSD守护程序，但这可能会导致资源争用并降低整体吞吐量。您可以将日志和对象数据存储在同一驱动器上，但是这可能会增加将写和ACK日志记录到客户端所需的时间。Ceph必须先写入日志，然后才能确认写入。

Ceph最佳实践规定您应在单独的驱动器上运行操作系统，OSD数据和OSD日志。

#### 固态硬盘

一种提高性能的机会是使用固态驱动器（SSD）来减少随机访问时间和读取延迟，同时加快吞吐量。与硬盘驱动器相比，SSD的每千兆字节价格通常高出10倍以上，但是SSD的访问时间通常比硬盘驱动器快至少100倍。

SSD没有可移动的机械部件，因此它们不一定受到与硬盘驱动器相同类型的限制。SSD确实有很大的局限性。在评估SSD时，重要的是要考虑顺序读取和写入的性能。当存储多个OSD的多个日志时，具有400MB / s顺序写入吞吐量的SSD可能比具有120MB / s顺序写入吞吐量的SSD更好。

**重要 ：我们建议探索使用SSD来提高性能。但是，在对SSD进行大量投资之前，我们强烈建议您既检查SSD的性能指标，又在测试配置中测试SSD以评估性能。**

由于SSD没有移动的机械零件，因此在不占用大量存储空间（例如日志）的Ceph区域中使用它们是有意义的。相对便宜的SSD可能会吸引您的经济感觉。谨慎使用。选择用于Ceph的SSD时，可接受的IOPS不够。对于期刊和SSD，有一些重要的性能注意事项：

* **写密集型语义：**日志记录涉及写密集型语义，因此在写数据时，应确保选择部署的SSD的性能等于或优于硬盘驱动器。廉价的SSD可能会增加写入延迟，即使它们加快了访问时间，因为有时高性能硬盘的写入速度可能比市场上某些更经济的SSD快或快！
* **顺序写入：**在SSD上存储多个日志时，还必须考虑SSD的顺序写入限制，因为它们可能正在处理同时写入多个OSD日志的请求。
* **分区对齐：** SSD性能的一个常见问题是人们喜欢对驱动器进行分区，这是最佳做法，但他们通常忽略了与SSD进行正确的分区对齐，这可能导致SSD传输数据的速度变慢得多。确保SSD分区正确对齐。

尽管SSD禁止对象存储成本，但OSD可能会通过将OSD的日志存储在SSD上以及将OSD的对象数据存储在单独的硬盘驱动器上而获得显着的性能提升。所述配置设置默认为。您可以将此路径安装到SSD或SSD分区，以便它不仅仅是与对象数据在同一磁盘上的文件。`osd journal/var/lib/ceph/osd/$cluster-$id/journal`

Ceph加速CephFS文件系统性能的一种方法是将CephFS元数据的存储与CephFS文件内容的存储区分开。Ceph `metadata`为CephFS元数据提供了一个默认池。您将不必为CephFS元数据创建池，但可以为CephFS元数据池创建仅指向主机的SSD存储介质的CRUSH映射层次结构。有关详细信息，请参见将 [池映射到不同类型的OSD](https://docs.ceph.com/docs/nautilus/rados/operations/crush-map#placing-different-pools-on-different-osds)。

#### 控制器

磁盘控制器对写入吞吐量也有重大影响。仔细考虑您选择的磁盘控制器，以确保它们不会造成性能瓶颈。

建议： [Ceph的博客](https://ceph.com/community/blog/)往往是对性能问题的信息的极好来源。有关更多详细信息，请参见[Ceph写吞吐量1](http://ceph.com/community/ceph-performance-part-1-disk-controller-write-throughput/)和[Ceph写吞吐量2](http://ceph.com/community/ceph-performance-part-2-write-throughput-without-ssd-journals/)。

#### 其他注意事项

您可以在每个主机上运行多个OSD，但应确保OSD硬盘的总吞吐量之和不超过满足客户端读取或写入数据所需的网络带宽。您还应该考虑群集在每个主机上存储的全部数据的百分比。如果特定主机上的百分比很大且主机发生故障，则可能导致诸如超过的问题，这会导致Ceph停止操作，这是一种安全预防措施，可防止数据丢失。`full ratio`

当每个主机运行多个OSD时，还需要确保内核是最新的。请参阅[操作系统建议，](https://docs.ceph.com/docs/nautilus/start/os-recommendations)以获取注释，`glibc`并 `syncfs(2)`确保每个主机运行多个OSD时硬件能够按预期运行。

OSD数量很多（例如&gt; 20）的主机可能会产生很多线程，尤其是在恢复和重新平衡期间。许多Linux内核默认使用相对较小的最大线程数（例如32k）。如果在具有大量OSD的主机上启动OSD时遇到问题，请考虑设置`kernel.pid_max`为更多的线程。理论上最大为4,194,303线程。例如，您可以将以下内容添加到`/etc/sysctl.conf`文件中：

```text
kernel.pid_max = 4194303
```

### 网络

我们建议每个主机至少有两个1Gbps网络接口控制器（NIC）。由于大多数商用硬盘驱动器的吞吐量约为100MB /秒，因此您的NIC应该能够处理主机上OSD磁盘的流量。我们建议至少使用两个NIC来说明一个公共（前端）网络和一个群集（后端）网络。群集网络（最好不连接到Internet）可以处理数据复制的额外负载，并有助于阻止拒绝服务攻击，从而阻止群集实现`active + clean`OSD在整个群集中复制数据时，放置组的状态。考虑从机架中的10Gbps网络开始。跨1Gbps网络复制1TB数据需要3个小时，而3TB（典型的驱动器配置）则需要9个小时。相反，对于10Gbps网络，复制时间分别为20分钟和1小时。在PB级群集中，OSD磁盘故障应该是一种期望，而不是例外。系统管理员将欣赏PG从`degraded`状态恢复到正常 状态的过程。`active + clean`在考虑价格/性能折衷的情况下尽可能快地进行陈述。此外，某些部署工具（例如Dell的Crowbar）可部署五个不同的网络，但使用VLAN使硬件和网络布线更易于管理。使用802.1q协议的VLAN需要具有VLAN功能的NIC和交换机。增加的硬件支出可能会因网络设置和维护的运营成本节省而被抵销。当使用VLAN处理群集和计算堆栈（例如OpenStack，CloudStack等）之间的VM通信时，还值得考虑使用10G以太网。每个网络的架顶式路由器还需要能够与吞吐量甚至更高（例如40Gbps至100Gbps）的主干路由器通信。

您的服务器硬件应具有基板管理控制器（BMC）。管理和部署工具也可能会广泛使用BMC，因此请考虑带外网络进行管理的成本/收益之间的权衡。系统管理程序SSH访问，VM映像上传，OS映像安装，管理套接字等可能会对网络造成很大的负担。运行三个网络似乎有些过头，但是每个流量路径都代表一个潜在的容量，吞吐量和/或性能瓶颈，您在部署大规模数据集群之前应仔细考虑。

### 失败域

故障域是阻止访问一个或多个OSD的任何故障。那可能是主机上停止的守护程序。硬盘故障，操作系统崩溃，网卡故障，电源故障，网络中断，电源中断等。在计划硬件需求时，您必须权衡诱惑以降低成本，方法是将过多的职责分配到过少的故障域中，以及增加隔离每个潜在故障域的成本。

### 最低硬件建议

Ceph可以在廉价的商品硬件上运行。小型生产集群和开发集群可以使用适度的硬件成功运行。

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x5904;&#x7406;</th>
      <th style="text-align:left">&#x6807;&#x51C6;</th>
      <th style="text-align:left">&#x6700;&#x4F4E;&#x63A8;&#x8350;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><code>ceph-osd</code>
      </td>
      <td style="text-align:left">&#x5904;&#x7406;&#x5668;</td>
      <td style="text-align:left">
        <ul>
          <li>1&#x4E2A;64&#x4F4D;AMD-64</li>
          <li>1&#x4E2A;32&#x4F4D;ARM&#x53CC;&#x6838;&#x6216;&#x66F4;&#x9AD8;</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">&#x5185;&#x5B58;</td>
      <td style="text-align:left">&#x6BCF;&#x4E2A;&#x5B88;&#x62A4;&#x7A0B;&#x5E8F;1TB&#x7684;&#x5B58;&#x50A8;&#x7A7A;&#x95F4;&#x7EA6;&#x4E3A;1GB</td>
      <td
      style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">&#x5377;&#x5B58;&#x50A8;</td>
      <td style="text-align:left">&#x6BCF;&#x4E2A;&#x5B88;&#x62A4;&#x7A0B;&#x5E8F;1&#x4E2A;&#x5B58;&#x50A8;&#x9A71;&#x52A8;&#x5668;</td>
      <td
      style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">&#x65E5;&#x5FD7;</td>
      <td style="text-align:left">&#x6BCF;&#x4E2A;&#x5B88;&#x62A4;&#x7A0B;&#x5E8F;1&#x4E2A;SSD&#x5206;&#x533A;&#xFF08;&#x53EF;&#x9009;&#xFF09;</td>
      <td
      style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">&#x7F51;&#x7EDC;</td>
      <td style="text-align:left">2&#x4E2A;1GB&#x4EE5;&#x592A;&#x7F51;NIC</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"><code>ceph-mon</code>
      </td>
      <td style="text-align:left">&#x5904;&#x7406;&#x5668;</td>
      <td style="text-align:left">
        <ul>
          <li>1&#x4E2A;64&#x4F4D;AMD-64</li>
          <li>1&#x4E2A;32&#x4F4D;ARM&#x53CC;&#x6838;&#x6216;&#x66F4;&#x9AD8;</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">&#x5185;&#x5B58;</td>
      <td style="text-align:left">&#x6BCF;&#x4E2A;&#x5B88;&#x62A4;&#x7A0B;&#x5E8F;1 GB</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">&#x78C1;&#x76D8;&#x7A7A;&#x95F4;</td>
      <td style="text-align:left">&#x6BCF;&#x4E2A;&#x5B88;&#x62A4;&#x7A0B;&#x5E8F;10 GB</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">&#x7F51;&#x7EDC;</td>
      <td style="text-align:left">2&#x4E2A;1GB&#x4EE5;&#x592A;&#x7F51;NIC</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"><code>ceph-mds</code>
      </td>
      <td style="text-align:left">&#x5904;&#x7406;&#x5668;</td>
      <td style="text-align:left">
        <ul>
          <li>1&#x4E2A;64&#x4F4D;AMD-64&#x56DB;&#x6838;</li>
          <li>1&#x4E2A;32&#x4F4D;ARM&#x56DB;&#x6838;</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">&#x5185;&#x5B58;</td>
      <td style="text-align:left">&#x6BCF;&#x4E2A;&#x5B88;&#x62A4;&#x7A0B;&#x5E8F;&#x81F3;&#x5C11;1 GB</td>
      <td
      style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">&#x78C1;&#x76D8;&#x7A7A;&#x95F4;</td>
      <td style="text-align:left">&#x6BCF;&#x4E2A;&#x5B88;&#x62A4;&#x7A0B;&#x5E8F;1 MB</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">&#x7F51;&#x7EDC;</td>
      <td style="text-align:left">2&#x4E2A;1GB&#x4EE5;&#x592A;&#x7F51;NIC</td>
      <td style="text-align:left"></td>
    </tr>
  </tbody>
</table>

小费 

如果使用单个磁盘运行OSD，请为卷存储创建一个分区，该分区与包含OS的分区分开。通常，我们建议为操作系统和卷存储使用单独的磁盘。

### 生产集群的例子

用于PB级数据存储的生产集群也可以使用商用硬件，但应具有更多的内存，处理能力和数据存储以解决繁重的流量负载。

#### 戴尔示例

最近（2012年）的Ceph集群项目正在为Ceph OSD使用两个相当健壮的硬件配置，为显示器使用一个较轻的配置。

| 组态 | 标准 | 最低推荐 |
| :--- | :--- | :--- |
| 戴尔PE R510 | 处理器 | 2个64位四核Xeon CPU |
| 内存 | 16 GB |  |
| 卷存储 | 8个2TB驱动器。1个操作系统，7个存储 |  |
| 客户网络 | 2个1GB以太网NIC |  |
| OSD网络 | 2个1GB以太网NIC |  |
| Mgmt。网络 | 2个1GB以太网NIC |  |
| 戴尔PE R515 | 处理器 | 1个六核Opteron CPU |
| 内存 | 16 GB |  |
| 卷存储 | 12个3TB驱动器。存储 |  |
| 操作系统存储 | 1个500GB驱动器。操作系统。 |  |
| 客户网络 | 2个1GB以太网NIC |  |
| OSD网络 | 2个1GB以太网NIC |  |
| Mgmt。网络 | 2个1GB以太网NIC |  |

