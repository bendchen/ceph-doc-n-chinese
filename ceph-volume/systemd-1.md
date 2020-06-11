# SYSTEMD

## SYSTEMD 

启动后，它将使用[LVM标记](https://docs.ceph.com/docs/nautilus/glossary/#term-lvm-tags)识别逻辑卷，找到匹配的ID，然后再使用[OSD uuid](https://docs.ceph.com/docs/nautilus/glossary/#term-osd-uuid)确保它是正确的。

识别正确的卷后，它将继续使用OSD目标约定来挂载它，即：

```text
/var/lib/ceph/osd/<cluster name>-<osd id>
```

对于我们的ID为的示例OSD `0`，这意味着将在以下位置挂载标识的设备：

```text
/var/lib/ceph/osd/ceph-0
```

该过程完成后，将进行调用以启动OSD：

```text
systemctl start ceph-osd@0
```

此过程的systemd部分由子命令处理，该子命令仅负责解析来自systemd和启动的元数据，然后将其分派到激活过程中。`ceph-volume lvm triggerceph-volume lvm activate`

