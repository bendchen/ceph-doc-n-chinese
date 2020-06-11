# CEPH-SYN – CEPH SYNTHETIC WORKLOAD GENERATOR

## CEPH-SYN – CEPH SYNTHETIC WORKLOAD GENERATOR

### SYNOPSIS

**ceph-syn** \[ -m _monaddr_:_port_ \] –syn _command_ _…_

### DESCRIPTION

**ceph-syn** is a simple synthetic workload generator for the Ceph distributed file system. It uses the userspace client library to generate simple workloads against a currently running file system. The file system need not be mounted via ceph-fuse\(8\) or the kernel client.

One or more `--syn` command arguments specify the particular workload, as documented below.

### OPTIONS

`-d`

Detach from console and daemonize after startup.`-c ceph.conf, --conf=ceph.conf`

Use _ceph.conf_ configuration file instead of the default `/etc/ceph/ceph.conf` to determine monitor addresses during startup.`-m monaddress[:port]`

Connect to specified monitor \(instead of looking through `ceph.conf`\).`--num_client num`

Run num different clients, each in a separate thread.`--syn workloadspec`

Run the given workload. May be specified as many times as needed. Workloads will normally run sequentially.

### WORKLOADS

Each workload should be preceded by `--syn` on the command line. This is not a complete list.**mknap** _path_ _snapname_

Create a snapshot called _snapname_ on _path_.**rmsnap** _path_ _snapname_

Delete snapshot called _snapname_ on _path_.**rmfile** _path_

Delete/unlink _path_.**writefile** _sizeinmb_ _blocksize_

Create a file, named after our client id, that is _sizeinmb_ MB by writing _blocksize_ chunks.**readfile** _sizeinmb_ _blocksize_

Read file, named after our client id, that is _sizeinmb_ MB by writing _blocksize_ chunks.**rw** _sizeinmb_ _blocksize_

Write file, then read it back, as above.**makedirs** _numsubdirs_ _numfiles_ _depth_

Create a hierarchy of directories that is _depth_ levels deep. Give each directory _numsubdirs_ subdirectories and _numfiles_ files.**walk**

Recursively walk the file system \(like find\).

### AVAILABILITY

**ceph-syn** is part of Ceph, a massively scalable, open-source, distributed storage system. Please refer to the Ceph documentation at [http://ceph.com/docs](http://ceph.com/docs) for more information.

### SEE ALSO

[ceph](https://docs.ceph.com/docs/nautilus/man/8/ceph/)\(8\), [ceph-fuse](https://docs.ceph.com/docs/nautilus/man/8/ceph-fuse/)\(8\)

