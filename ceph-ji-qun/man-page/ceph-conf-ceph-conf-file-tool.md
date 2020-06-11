# CEPH-CONF – CEPH CONF FILE TOOL

## CEPH-CONF – CEPH CONF FILE TOOL

### SYNOPSIS

**ceph-conf** -c _conffile_ –list-all-sections**ceph-conf** -c _conffile_ -L**ceph-conf** -c _conffile_ -l _prefix_**ceph-conf** _key_ -s _section1_ …**ceph-conf** \[-s _section_ \] \[-r\] –lookup _key_**ceph-conf** \[-s _section_ \] _key_

### DESCRIPTION

**ceph-conf** is a utility for getting information from a ceph configuration file. As with most Ceph programs, you can specify which Ceph configuration file to use with the `-c` flag.

Note that unlike other ceph tools, **ceph-conf** will _only_ read from config files \(or return compiled-in default values\)–it will _not_ fetch config values from the monitor cluster. For this reason it is recommended that **ceph-conf** only be used in legacy environments that are strictly config-file based. New deployments and tools should instead rely on either querying the monitor explicitly for configuration \(e.g., `ceph config get <daemon> <option>`\) or use daemons themselves to fetch effective config options \(e.g., `ceph-osd -i 123 --show-config-value osd_data`\). The latter option has the advantages of drawing from compiled-in defaults \(which occasionally vary between daemons\), config files, and the monitor’s config database, providing the exact value that that daemon would be using if it were started.

### ACTIONS

**ceph-conf** performs one of the following actions:`-L, --list-all-sections`

list all sections in the configuration file.`-l, --list-sections *prefix*`

list the sections with the given _prefix_. For example, `--list-sections mon` would list all sections beginning with `mon`.`--lookup *key*`

search and print the specified configuration setting. Note: `--lookup` is the default action. If no other actions are given on the command line, we will default to doing a lookup.`-h, --help`

print a summary of usage.

### OPTIONS

`-c *conffile*`

the Ceph configuration file.`--filter-key *key*`

filter section list to only include sections with given _key_ defined.```--filter-key-value *key* ``=`` *value*```

filter section list to only include sections with given _key_/_value_ pair.`--name *type.id*`

the Ceph name in which the sections are searched \(default ‘client.admin’\). For example, if we specify `--name osd.0`, the following sections will be searched: \[osd.0\], \[osd\], \[global\]`-r, --resolve-search`

search for the first file that exists and can be opened in the resulted comma delimited search list.`-s, --section`

additional sections to search. These additional sections will be searched before the sections that would normally be searched. As always, the first matching entry we find will be returned.

### EXAMPLES

To find out what value osd 0 will use for the “osd data” option:

```text
ceph-conf -c foo.conf  --name osd.0 --lookup "osd data"
```

To find out what value will mds a use for the “log file” option:

```text
ceph-conf -c foo.conf  --name mds.a "log file"
```

To list all sections that begin with “osd”:

```text
ceph-conf -c foo.conf -l osd
```

To list all sections:

```text
ceph-conf -c foo.conf -L
```

To print the path of the “keyring” used by “client.0”:

```text
ceph-conf --name client.0 -r -l keyring
```

### FILES

`/etc/ceph/$cluster.conf`, `~/.ceph/$cluster.conf`, `$cluster.conf`

the Ceph configuration files to use if not specified.

### AVAILABILITY

**ceph-conf** is part of Ceph, a massively scalable, open-source, distributed storage system. Please refer to the Ceph documentation at [http://ceph.com/docs](http://ceph.com/docs) for more information.

### SEE ALSO

[ceph](https://docs.ceph.com/docs/nautilus/man/8/ceph/)\(8\),

