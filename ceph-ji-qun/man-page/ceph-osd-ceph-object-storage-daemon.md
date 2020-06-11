# CEPH-OSD – CEPH OBJECT STORAGE DAEMON

## CEPH-OSD – CEPH OBJECT STORAGE DAEMON

### SYNOPSIS

**ceph-osd** -i _osdnum_ \[ –osd-data _datapath_ \] \[ –osd-journal _journal_ \] \[ –mkfs \] \[ –mkjournal \] \[–flush-journal\] \[–check-allows-journal\] \[–check-wants-journal\] \[–check-needs-journal\] \[ –mkkey \]

### DESCRIPTION

**ceph-osd** is the object storage daemon for the Ceph distributed file system. It is responsible for storing objects on a local file system and providing access to them over the network.

The datapath argument should be a directory on a xfs file system where the object data resides. The journal is optional, and is only useful performance-wise when it resides on a different disk than datapath with low latency \(ideally, an NVRAM device\).

### OPTIONS

`-f, --foreground`

Foreground: do not daemonize after startup \(run in foreground\). Do not generate a pid file. Useful when run via [ceph-run](https://docs.ceph.com/docs/nautilus/man/8/ceph-run/)\(8\).`-d`

Debug mode: like `-f`, but also send all log output to stderr.`--setuser userorgid`

Set uid after starting. If a username is specified, the user record is looked up to get a uid and a gid, and the gid is also set as well, unless –setgroup is also specified.`--setgroup grouporgid`

Set gid after starting. If a group name is specified the group record is looked up to get a gid.`--osd-data osddata`

Use object store at _osddata_.`--osd-journal journal`

Journal updates to _journal_.`--check-wants-journal`

Check whether a journal is desired.`--check-allows-journal`

Check whether a journal is allowed.`--check-needs-journal`

Check whether a journal is required.`--mkfs`

Create an empty object repository. This also initializes the journal \(if one is defined\).`--mkkey`

Generate a new secret key. This is normally used in combination with `--mkfs` as it is more convenient than generating a key by hand with [ceph-authtool](https://docs.ceph.com/docs/nautilus/man/8/ceph-authtool/)\(8\).`--mkjournal`

Create a new journal file to match an existing object repository. This is useful if the journal device or file is wiped out due to a disk or file system failure.`--flush-journal`

Flush the journal to permanent store. This runs in the foreground so you know when it’s completed. This can be useful if you want to resize the journal or need to otherwise destroy it: this guarantees you won’t lose data.`--get-cluster-fsid`

Print the cluster fsid \(uuid\) and exit.`--get-osd-fsid`

Print the OSD’s fsid and exit. The OSD’s uuid is generated at –mkfs time and is thus unique to a particular instantiation of this OSD.`--get-journal-fsid`

Print the journal’s uuid. The journal fsid is set to match the OSD fsid at –mkfs time.`-c ceph.conf, --conf=ceph.conf`

Use _ceph.conf_ configuration file instead of the default `/etc/ceph/ceph.conf` for runtime configuration options.`-m monaddress[:port]`

Connect to specified monitor \(instead of looking through `ceph.conf`\).

### AVAILABILITY

**ceph-osd** is part of Ceph, a massively scalable, open-source, distributed storage system. Please refer to the Ceph documentation at [http://ceph.com/docs](http://ceph.com/docs) for more information.

### SEE ALSO

[ceph](https://docs.ceph.com/docs/nautilus/man/8/ceph/)\(8\), [ceph-mds](https://docs.ceph.com/docs/nautilus/man/8/ceph-mds/)\(8\), [ceph-mon](https://docs.ceph.com/docs/nautilus/man/8/ceph-mon/)\(8\), [ceph-authtool](https://docs.ceph.com/docs/nautilus/man/8/ceph-authtool/)\(8\)

