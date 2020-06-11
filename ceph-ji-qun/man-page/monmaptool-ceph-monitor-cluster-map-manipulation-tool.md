# MONMAPTOOL – CEPH MONITOR CLUSTER MAP MANIPULATION TOOL

## MONMAPTOOL – CEPH MONITOR CLUSTER MAP MANIPULATION TOOL

### SYNOPSIS

**monmaptool** _mapfilename_ \[ –clobber \] \[ –print \] \[ –create \] \[ –add _ip_:_port_ _…_ \] \[ –rm _ip_:_port_ _…_ \]

### DESCRIPTION

**monmaptool** is a utility to create, view, and modify a monitor cluster map for the Ceph distributed storage system. The monitor map specifies the only fixed addresses in the Ceph distributed system. All other daemons bind to arbitrary addresses and register themselves with the monitors.

When creating a map with –create, a new monitor map with a new, random UUID will be created. It should be followed by one or more monitor addresses.

The default Ceph monitor port is 6789.

### OPTIONS

`--print`

will print a plaintext dump of the map, after any modifications are made.`--clobber`

will allow monmaptool to overwrite mapfilename if changes are made.`--create`

will create a new monitor map with a new UUID \(and with it, a new, empty Ceph file system\).`--generate`

generate a new monmap based on the values on the command line or specified in the ceph configuration. This is, in order of preference,

> 1. `--monmap filename` to specify a monmap to load
> 2. `--mon-host 'host1,ip2'` to specify a list of hosts or ip addresses
> 3. `[mon.foo]` sections containing `mon addr` settings in the config. Note that this method is not recommended and support will be removed in a future release.

`--filter-initial-members`

filter the initial monmap by applying the `mon initial members` setting. Monitors not present in that list will be removed, and initial members not present in the map will be added with dummy addresses.`--add name ip:port`

will add a monitor with the specified ip:port to the map.`--rm name`

will remove the monitor with the specified ip:port from the map.`--fsid uuid`

will set the fsid to the given uuid. If not specified with –create, a random fsid will be generated.

### EXAMPLE

To create a new map with three monitors \(for a fresh Ceph file system\):

```text
monmaptool  --create  --add  mon.a 192.168.0.10:6789 --add mon.b 192.168.0.11:6789 \
  --add mon.c 192.168.0.12:6789 --clobber monmap
```

To display the contents of the map:

```text
monmaptool --print monmap
```

To replace one monitor:

```text
monmaptool --rm mon.a --add mon.a 192.168.0.9:6789 --clobber monmap
```

### AVAILABILITY

**monmaptool** is part of Ceph, a massively scalable, open-source, distributed storage system. Please refer to the Ceph documentation at [http://ceph.com/docs](http://ceph.com/docs) for more information.

### SEE ALSO

[ceph](https://docs.ceph.com/docs/nautilus/man/8/ceph/)\(8\), [crushtool](https://docs.ceph.com/docs/nautilus/man/8/crushtool/)\(8\),

