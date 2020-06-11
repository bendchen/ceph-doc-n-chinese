# ZAP

## `ZAP`

此子命令用于替换ceph OSD使用的lvs，分区或原始设备，以便可以重用它们。如果指定了逻辑卷的路径，则必须采用vg / lv的格式。给定lv或分区上存在的任何文件系统都将被删除，所有数据将被清除。

注意 

lv或分区将保持不变。

注意 

如果逻辑卷，原始设备或分区用于任何与ceph相关的安装点，则将它们卸载。

更改逻辑卷：

```text
ceph-volume lvm zap {vg name/lv name}
```

分割分区：

```text
ceph-volume lvm zap /dev/sdc1
```

### 删除设备

切换时，如果要完全删除设备（lv，vg或分区），请使用该`--destroy`标志。一个常见的用例是使用整个原始设备简单地部署OSD。如果这样做，然后希望将该设备重用于另一个OSD，则`--destroy`在切换时必须使用该标志，以便删除在原始设备上创建的ceph-volume的vg和lvs。

注意 

一次可以接收多个设备，以将它们全部打包

更换原始设备并销毁存在的任何vg或lvs：

```text
ceph-volume lvm zap /dev/sdc --destroy
```

可以在分区和逻辑卷上执行此操作：

```text
ceph-volume lvm zap /dev/sdc1 --destroy
ceph-volume lvm zap osd-vg/data-lv --destroy
```

最后，如果按OSD ID和/或OSD FSID进行过滤，则可以检测到多个设备。可以使用一个标识符，也可以同时使用两个标识符。这在需要清除与特定ID关联的多个设备的情况下很有用。使用FSID时，过滤更加严格，并且可能与与ID关联的其他（可能无效）设备不匹配。

仅凭ID：

```text
ceph-volume lvm zap --destroy --osd-id 1
```

通过FSID：

```text
ceph-volume lvm zap --destroy --osd-fsid 2E8FBE58-0328-4E3B-BFB7-3CACE4E9A6CE
```

两者：

```text
ceph-volume lvm zap --destroy --osd-fsid 2E8FBE58-0328-4E3B-BFB7-3CACE4E9A6CE --osd-id 1
```

警告 

如果检测到与要切换的OSD ID关联的systemd单元正在运行，则该工具将拒绝切换，直到守护程序停止为止。

