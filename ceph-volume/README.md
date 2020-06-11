# CEPH-VOLUME

## CEPH-VOLUME

Deploy OSDs with different device technologies like lvm or physical disks using pluggable tools \([lvm](https://docs.ceph.com/docs/nautilus/ceph-volume/lvm/) itself is treated like a plugin\) and trying to follow a predictable, and robust way of preparing, activating, and starting OSDs.

[Overview](https://docs.ceph.com/docs/nautilus/ceph-volume/intro/#ceph-volume-overview) \| [Plugin Guide](https://docs.ceph.com/docs/nautilus/dev/ceph-volume/plugins/#ceph-volume-plugins) \|

**Command Line Subcommands**

There is currently support for `lvm`, and plain disks \(with GPT partitions\) that may have been deployed with `ceph-disk`.

`zfs` support is available for running a FreeBSD cluster.

* [lvm](https://docs.ceph.com/docs/nautilus/ceph-volume/lvm/#ceph-volume-lvm)
* [simple](https://docs.ceph.com/docs/nautilus/ceph-volume/simple/#ceph-volume-simple)
* [zfs](https://docs.ceph.com/docs/nautilus/ceph-volume/zfs/#ceph-volume-zfs)

**Node inventory**

The [inventory](https://docs.ceph.com/docs/nautilus/ceph-volume/inventory/#ceph-volume-inventory) subcommand provides information and metadata about a nodes physical disk inventory.

### MIGRATING

Starting on Ceph version 13.0.0, `ceph-disk` is deprecated. Deprecation warnings will show up that will link to this page. It is strongly suggested that users start consuming `ceph-volume`. There are two paths for migrating:

1. Keep OSDs deployed with `ceph-disk`: The [simple](https://docs.ceph.com/docs/nautilus/ceph-volume/simple/#ceph-volume-simple) command provides a way to take over the management while disabling `ceph-disk` triggers.
2. Redeploy existing OSDs with `ceph-volume`: This is covered in depth on [Replacing an OSD](https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-osds/#rados-replacing-an-osd)

For details on why `ceph-disk` was removed please see the [Why was ceph-disk replaced?](https://docs.ceph.com/docs/nautilus/ceph-volume/intro/#ceph-disk-replaced) section.

#### NEW DEPLOYMENTS

For new deployments, [lvm](https://docs.ceph.com/docs/nautilus/ceph-volume/lvm/#ceph-volume-lvm) is recommended, it can use any logical volume as input for data OSDs, or it can setup a minimal/naive logical volume from a device.

#### EXISTING OSDS

If the cluster has OSDs that were provisioned with `ceph-disk`, then `ceph-volume` can take over the management of these with [simple](https://docs.ceph.com/docs/nautilus/ceph-volume/simple/#ceph-volume-simple). A scan is done on the data device or OSD directory, and `ceph-disk` is fully disabled. Encryption is fully supported.

