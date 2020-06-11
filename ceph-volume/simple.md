# SIMPLE

## `SIMPLE`

Implements the functionality needed to manage OSDs from the `simple` subcommand: `ceph-volume simple`

**Command Line Subcommands**

* [scan](https://docs.ceph.com/docs/nautilus/ceph-volume/simple/scan/#ceph-volume-simple-scan)
* [activate](https://docs.ceph.com/docs/nautilus/ceph-volume/simple/activate/#ceph-volume-simple-activate)
* [systemd](https://docs.ceph.com/docs/nautilus/ceph-volume/simple/systemd/#ceph-volume-simple-systemd)

By _taking over_ management, it disables all `ceph-disk` systemd units used to trigger devices at startup, relying on basic \(customizable\) JSON configuration and systemd for starting up OSDs.

This process involves two steps:

1. [Scan](https://docs.ceph.com/docs/nautilus/ceph-volume/simple/scan/#ceph-volume-simple-scan) the running OSD or the data device
2. [Activate](https://docs.ceph.com/docs/nautilus/ceph-volume/simple/activate/#ceph-volume-simple-activate) the scanned OSD

The scanning will infer everything that `ceph-volume` needs to start the OSD, so that when activation is needed, the OSD can start normally without getting interference from `ceph-disk`.

As part of the activation process the systemd units for `ceph-disk` in charge of reacting to `udev` events, are linked to `/dev/null` so that they are fully inactive.

