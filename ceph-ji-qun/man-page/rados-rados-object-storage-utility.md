# RADOS – RADOS OBJECT STORAGE UTILITY

## RADOS – RADOS OBJECT STORAGE UTILITY

### SYNOPSIS

**rados** \[ _options_ \] \[ _command_ \]

### DESCRIPTION

**rados** is a utility for interacting with a Ceph object storage cluster \(RADOS\), part of the Ceph distributed storage system.

### OPTIONS

`-p pool, --pool pool`

Interact with the given pool. Required by most commands.`--pgid`

As an alternative to `--pool`, `--pgid` also allow users to specify the PG id to which the command will be directed. With this option, certain commands like `ls` allow users to limit the scope of the command to the given PG.`-N namespace, --namespace namespace`

Specify the rados namespace to use for the object.`-s snap, --snap snap`

Read from the given pool snapshot. Valid for all pool-specific read operations.`-i infile`

will specify an input file to be passed along as a payload with the command to the monitor cluster. This is only used for specific monitor commands.`-o outfile`

will write any payload returned by the monitor cluster with its reply to outfile. Only specific monitor commands \(e.g. osd getmap\) return a payload.`-c ceph.conf, --conf=ceph.conf`

Use ceph.conf configuration file instead of the default /etc/ceph/ceph.conf to determine monitor addresses during startup.`-m monaddress[:port]`

Connect to specified monitor \(instead of looking through ceph.conf\).`-b block_size`

Set the block size for put/get/append ops and for write benchmarking.`--striper`

Uses the striping API of rados rather than the default one. Available for stat, stat2, get, put, append, truncate, rm, ls and all xattr related operation`-O object_size`

Set the object size for put/get ops and for write benchmarking

### GLOBAL COMMANDS

**lspools**

List object pools**df**

Show utilization statistics, including disk usage \(bytes\) and object counts, over the entire system and broken down by pool.**list-inconsistent-pg** _pool_

List inconsistent PGs in given pool.**list-inconsistent-obj** _pgid_

List inconsistent objects in given PG.**list-inconsistent-snapset** _pgid_

List inconsistent snapsets in given PG.

### POOL SPECIFIC COMMANDS

**get** _name_ _outfile_

Read object name from the cluster and write it to outfile.**put** _name_ _infile_ \[–offset offset\]

Write object name with start offset \(default:0\) to the cluster with contents from infile. **Warning:** The put command creates a single RADOS object, sized just as large as your input file. Unless your objects are of reasonable and consistent sizes, that is probably not what you want – consider using RGW/S3, CephFS, or RBD instead.**append** _name_ _infile_

Append object name to the cluster with contents from infile.**rm** _name_

Remove object name.**listwatchers** _name_

List the watchers of object name.**ls** _outfile_

List objects in the given pool and write to outfile. Instead of `--pool` if `--pgid` will be specified, `ls` will only list the objects in the given PG.**lssnap**

List snapshots for given pool.**clonedata** _srcname_ _dstname_ –object-locator _key_

Clone object byte data from _srcname_ to _dstname_. Both objects must be stored with the locator key _key_ \(usually either _srcname_ or _dstname_\). Object attributes and omap keys are not copied or cloned.**mksnap** _foo_

Create pool snapshot named _foo_.**rmsnap** _foo_

Remove pool snapshot named _foo_.**bench** _seconds_ _mode_ \[ -b _objsize_ \] \[ -t _threads_ \]

Benchmark for _seconds_. The mode can be _write_, _seq_, or _rand_. _seq_ and _rand_ are read benchmarks, either sequential or random. Before running one of the reading benchmarks, run a write benchmark with the _–no-cleanup_ option. The default object size is 4 MB, and the default number of simulated threads \(parallel writes\) is 16. The _–run-name &lt;label&gt;_ option is useful for benchmarking a workload test from multiple clients. The _&lt;label&gt;_ is an arbitrary object name. It is “benchmark\_last\_metadata” by default, and is used as the underlying object name for “read” and “write” ops. Note: -b _objsize_ option is valid only in _write_ mode. Note: _write_ and _seq_ must be run on the same host otherwise the objects created by _write_ will have names that will fail _seq_.**cleanup** \[ –run-name _run\_name_ \] \[ –prefix _prefix_ \]

Clean up a previous benchmark operation. Note: the default run-name is “benchmark\_last\_metadata”**listxattr** _name_

List all extended attributes of an object.**getxattr** _name_ _attr_

Dump the extended attribute value of _attr_ of an object.**setxattr** _name_ _attr_ _value_

Set the value of _attr_ in the extended attributes of an object.**rmxattr** _name_ _attr_

Remove _attr_ from the extended attributes of an object.**stat** _name_

Get stat \(ie. mtime, size\) of given object**stat2** _name_

Get stat \(similar to stat, but with high precision time\) of given object**listomapkeys** _name_

List all the keys stored in the object map of object name.**listomapvals** _name_

List all key/value pairs stored in the object map of object name. The values are dumped in hexadecimal.**getomapval** \[ –omap-key-file _file_ \] _name_ _key_ \[ _out-file_ \]

Dump the hexadecimal value of key in the object map of object name. If the optional _out-file_ argument is not provided, the value will be written to standard output.**setomapval** \[ –omap-key-file _file_ \] _name_ _key_ \[ _value_ \]

Set the value of key in the object map of object name. If the optional _value_ argument is not provided, the value will be read from standard input.**rmomapkey** \[ –omap-key-file _file_ \] _name_ _key_

Remove key from the object map of object name.**getomapheader** _name_

Dump the hexadecimal value of the object map header of object name.**setomapheader** _name_ _value_

Set the value of the object map header of object name.**export** _filename_

Serialize pool contents to a file or standard output.n”**import** \[–dry-run\] \[–no-overwrite\] &lt; filename \| - &gt;

Load pool contents from a file or standard input

### EXAMPLES

To view cluster utilization:

```text
rados df
```

To get a list object in pool foo sent to stdout:

```text
rados -p foo ls -
```

To get a list of objects in PG 0.6:

```text
rados --pgid 0.6 ls
```

To write an object:

```text
rados -p foo put myobject blah.txt
```

To create a snapshot:

```text
rados -p foo mksnap mysnap
```

To delete the object:

```text
rados -p foo rm myobject
```

To read a previously snapshotted version of an object:

```text
rados -p foo -s mysnap get myobject blah.txt.old
```

To list inconsistent objects in PG 0.6:

```text
rados list-inconsistent-obj 0.6 --format=json-pretty
```

### AVAILABILITY

**rados** is part of Ceph, a massively scalable, open-source, distributed storage system. Please refer to the Ceph documentation at [http://ceph.com/docs](http://ceph.com/docs) for more information.

### SEE ALSO

[ceph](https://docs.ceph.com/docs/nautilus/man/8/ceph/)\(8\)

