# 存储设备

## 存储设备

有两个在磁盘上存储数据的Ceph守护程序：

* **Ceph OSD**（或对象存储守护程序）是大多数数据存储在Ceph中的地方。一般而言，每个OSD都由单个存储设备（例如传统硬盘（HDD）或固态磁盘（SSD））支持。OSD也可以由多种设备组合来支持，例如用于大多数数据的HDD和用于某些元数据的SSD（或SSD的分区）。群集中OSD的数量通常取决于存储的数据量，每个存储设备的容量以及冗余（复制或擦除编码）的级别和类型。
* **Ceph Monitor**守护程序管理关键的群集状态，例如群集成员身份和身份验证信息。对于较小的群集，只需要几GB的容量，尽管对于较大的群集，监控器数据库可以达到数十或数百GB的容量。

### OSD后端

OSD可以通过两种方式管理它们存储的数据。从Luminous 12.2.z版本开始，新的默认（推荐）后端是 _BlueStore_。在Luminous之前，默认（也是唯一的选项）是 _FileStore_。

#### BLUESTORE 

BlueStore是专用于存储的后端，专门用于管理Ceph OSD工作负载的磁盘上的数据。它是由过去十年中使用FileStore支持和管理OSD的经验所激发的。BlueStore的主要功能包括：

* 直接管理存储设备。BlueStore使用原始块设备或分区。这避免了可能影响性能或增加复杂性的任何中间抽象层（例如XFS之类的本地文件系统）。
* RocksDB的元数据管理。我们嵌入RocksDB的键/值数据库是为了管理内部元数据，例如从对象名称到磁盘上块位置的映射。
* 完整的数据和元数据校验和。默认情况下，所有写入BlueStore的数据和元数据都受到一个或多个校验和的保护。未经验证，不会从磁盘读取任何数据或元数据或将其返回给用户。
* 内联压缩。写入的数据在写入磁盘之前可以选择压缩。
* 多设备元数据分层。BlueStore允许将其内部日志（预写日志）写入单独的高速设备（例如SSD，NVMe或NVDIMM），以提高性能。如果有大量的更快的存储空间，内部元数据也可以存储在更快的设备上。
* 高效的写时复制。RBD和CephFS快照依赖于在BlueStore中有效实现的写时_复制克隆_机制。这将为常规快照和擦除代码池（依赖克隆实现高效的两阶段提交）提供高效的IO。

有关更多信息，请参阅[BlueStore配置参考](https://docs.ceph.com/docs/nautilus/rados/configuration/bluestore-config-ref/)和[BlueStore迁移](https://docs.ceph.com/docs/nautilus/rados/operations/bluestore-migration/)。

#### FILESTORE的

FileStore是在Ceph中存储对象的传统方法。它依赖于标准文件系统（通常为XFS）以及键/值数据库（传统为LevelDB，现在为RocksDB）来获取某些元数据。

FileStore经过了充分的测试，并在生产中广泛使用，但是由于其总体设计以及对用于存储对象数据的传统文件系统的依赖，因此存在许多性能缺陷。

尽管FileStore通常能够在大多数POSIX兼容文件系统（包括btrfs和ext4）上运行，但是我们仅建议使用XFS。btrfs和ext4都有已知的错误和不足，使用它们可能会导致数据丢失。默认情况下，所有Ceph设置工具都将使用XFS。

有关更多信息，请参阅[Filestore Config Reference](https://docs.ceph.com/docs/nautilus/rados/configuration/filestore-config-ref/)。

