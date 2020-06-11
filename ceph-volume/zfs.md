# ZFS

## `ZFS`

Implements the functionality needed to deploy OSDs from the `zfs` subcommand: `ceph-volume zfs`

The current implementation only works for ZFS on FreeBSD

**Command Line Subcommands**

* [inventory](https://docs.ceph.com/docs/nautilus/ceph-volume/zfs/inventory/#ceph-volume-zfs-inventory)

**Internal functionality**

There are other aspects of the `zfs` subcommand that are internal and not exposed to the user, these sections explain how these pieces work together, clarifying the workflows of the tool.

[zfs](https://docs.ceph.com/docs/nautilus/dev/ceph-volume/zfs/#ceph-volume-zfs-api)

