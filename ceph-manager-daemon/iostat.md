# IOSTAT

## IOSTAT

This module shows the current throughput and IOPS done on the Ceph cluster.

### ENABLING

To check if the _iostat_ module is enabled, run:

```text
ceph mgr module ls
```

The module can be enabled with:

```text
ceph mgr module enable iostat
```

To execute the module, run:

```text
ceph iostat
```

To change the frequency at which the statistics are printed, use the `-p` option:

```text
ceph iostat -p <period in seconds>
```

For example, use the following command to print the statistics every 5 seconds:

```text
ceph iostat -p 5
```

To stop the module, press Ctrl-C.

