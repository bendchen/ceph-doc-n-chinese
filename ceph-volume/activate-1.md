# ACTIVATE

## `ACTIVATE`

一旦[扫描](https://docs.ceph.com/docs/nautilus/ceph-volume/simple/scan/#ceph-volume-simple-scan)已经完成，并且所有捕获的OSD元数据被保存到`/etc/ceph/osd/{id}-{uuid}.json` 的OSD现在准备好“激活”。

此激活过程通过屏蔽所有systemd单元来**禁用**`ceph-disk`它们，以防止UDEV / ceph-disk交互将试图在引导时启动它们。

`ceph-disk`仅在直接调用时才禁用单元，但在系统启动时由systemd调用时可以避免禁用。`ceph-volume simple activate`

激活过程需要同时使用[OSD ID](https://docs.ceph.com/docs/nautilus/glossary/#term-osd-id)和[OSD uuid](https://docs.ceph.com/docs/nautilus/glossary/#term-osd-uuid) 来激活已解析的OSD：

```text
ceph-volume simple activate 0 6cc43680-4f6e-4feb-92ff-9c7ba204120e
```

上面的命令将假定可以在以下位置找到JSON配置：

```text
/etc/ceph/osd/0-6cc43680-4f6e-4feb-92ff-9c7ba204120e.json
```

另外，也可以直接使用JSON文件的路径：

```text
ceph-volume simple activate --file /etc/ceph/osd/0-6cc43680-4f6e-4feb-92ff-9c7ba204120e.json
```

### 需要的UUID 

该[UUID OSD](https://docs.ceph.com/docs/nautilus/glossary/#term-osd-uuid)被要求作为一个额外的步骤，以确保正确的OSD被激活。先前存在具有相同ID的OSD很有可能会导致激活错误的OSD。

#### 发现

对于先前已扫描过的OSD `ceph-volume`，使用和执行_发现_过程。当前仅支持具有GPT分区和LVM逻辑卷的设备。`blkidlvm`

GPT分区将具有一个`PARTUUID`可以通过调用来查询的`blkid`，逻辑卷将具有`lv_uuid`可以对其进行查询的`lvs`（用于列出逻辑卷的LVM工具）。

此发现过程可确保即使将设备重新用于其他系统或更改其名称（例如，如非持久性名称`/dev/sda1`）也能正确检测到设备

JSON配置文件用于将哪些设备映射到什么OSD，然后将其作为激活的一部分来协调安装和符号链接。

为了确保符号链接始终正确，如果它们存在于OSD目录中，则将重新完成符号链接。

一个systemd单元将捕获[OSD id](https://docs.ceph.com/docs/nautilus/glossary/#term-osd-id)和[OSD uuid](https://docs.ceph.com/docs/nautilus/glossary/#term-osd-uuid)并将其持久化。在内部，激活将启用它，例如：

```text
systemctl enable ceph-volume@simple-$id-$uuid
```

例如：

```text
systemctl enable ceph-volume@simple-0-8715BEB4-15C5-49DE-BA6F-401086EC7B41
```

将以ID为`0`且UUID 为的OSD启动发现过程`8715BEB4-15C5-49DE-BA6F-401086EC7B41`。

systemd进程将调出激活以传递识别OSD及其设备所需的信息，并且它将继续进行：＃将设备安装在相应的位置（按照惯例，这是

`/var/lib/ceph/osd/<cluster name>-<osd id>/`）

＃确保所有必需的设备都已准备好用于该OSD，并且已正确链接，无论使用了什么对象存储（文件存储或bluestore）。符号链接将 **始终**被重做以确保链接了正确的设备。

＃启动`ceph-osd@0`系统单元

