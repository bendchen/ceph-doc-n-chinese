# CEPH-MON – CEPH MONITOR DAEMON

## CEPH-MON – CEPH MONITOR DAEMON

### SYNOPSIS

**ceph-mon** -i _monid_ \[ –mon-data _mondatapath_ \]

### DESCRIPTION

**ceph-mon** is the cluster monitor daemon for the Ceph distributed file system. One or more instances of **ceph-mon** form a Paxos part-time parliament cluster that provides extremely reliable and durable storage of cluster membership, configuration, and state.

The _mondatapath_ refers to a directory on a local file system storing monitor data. It is normally specified via the `mon data` option in the configuration file.

### OPTIONS

`-f, --foreground`

Foreground: do not daemonize after startup \(run in foreground\). Do not generate a pid file. Useful when run via [ceph-run](https://docs.ceph.com/docs/nautilus/man/8/ceph-run/)\(8\).`-d`

Debug mode: like `-f`, but also send all log output to stderr.`--setuser userorgid`

Set uid after starting. If a username is specified, the user record is looked up to get a uid and a gid, and the gid is also set as well, unless –setgroup is also specified.`--setgroup grouporgid`

Set gid after starting. If a group name is specified the group record is looked up to get a gid.`-c ceph.conf, --conf=ceph.conf`

Use _ceph.conf_ configuration file instead of the default `/etc/ceph/ceph.conf` to determine monitor addresses during startup.`--mkfs`

Initialize the `mon data` directory with seed information to form and initial ceph file system or to join an existing monitor cluster. Three pieces of information must be provided:

* The cluster fsid. This can come from a monmap \(`--monmap <path>`\) or explicitly via `--fsid <uuid>`.
* A list of monitors and their addresses. This list of monitors can come from a monmap \(`--monmap <path>`\), the `mon host` configuration value \(in _ceph.conf_ or via `-m host1,host2,...`\), or \(for backward compatibility\) the deprecated `mon addr` lines in _ceph.conf_. If this monitor is to be part of the initial monitor quorum for a new Ceph cluster, then it must be included in the initial list, matching either the name or address of a monitor in the list. When matching by address, either the `public addr` or `public subnet` options may be used.
* The monitor secret key `mon.`. This must be included in the keyring provided via `--keyring <path>`.

`--keyring`

Specify a keyring for use with `--mkfs`.

### AVAILABILITY

**ceph-mon** is part of Ceph, a massively scalable, open-source, distributed storage system. Please refer to the Ceph documentation at [http://ceph.com/docs](http://ceph.com/docs) for more information.

### SEE ALSO

[ceph](https://docs.ceph.com/docs/nautilus/man/8/ceph/)\(8\), [ceph-mds](https://docs.ceph.com/docs/nautilus/man/8/ceph-mds/)\(8\), [ceph-osd](https://docs.ceph.com/docs/nautilus/man/8/ceph-osd/)\(8\)

