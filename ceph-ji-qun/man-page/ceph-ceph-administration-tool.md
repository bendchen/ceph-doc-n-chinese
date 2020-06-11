# CEPH – CEPH ADMINISTRATION TOOL

## CEPH – CEPH ADMINISTRATION TOOL

### SYNOPSIS

**ceph** **auth** \[ _add_ \| _caps_ \| _del_ \| _export_ \| _get_ \| _get-key_ \| _get-or-create_ \| _get-or-create-key_ \| _import_ \| _list_ \| _print-key_ \| _print\_key_ \] …**ceph** **compactceph** **config-key** \[ _rm_ \| _exists_ \| _get_ \| _ls_ \| _dump_ \| _set_ \] …**ceph** **daemon** _&lt;name&gt;_ \| _&lt;path&gt;_ _&lt;command&gt;_ …**ceph** **daemonperf** _&lt;name&gt;_ \| _&lt;path&gt;_ \[ _interval_ \[ _count_ \] \]**ceph** **df** _{detail}_**ceph** **fs** \[ _ls_ \| _new_ \| _reset_ \| _rm_ \] …**ceph** **fsidceph** **health** _{detail}_**ceph** **heap** \[ _dump_ \| _start\_profiler_ \| _stop\_profiler_ \| _release_ \| _get\_release\_rate_ \| _set\_release\_rate_ \| _stats_ \] …**ceph** **injectargs** _&lt;injectedargs&gt;_ \[ _&lt;injectedargs&gt;_… \]**ceph** **log** _&lt;logtext&gt;_ \[ _&lt;logtext&gt;_… \]**ceph** **mds** \[ _compat_ \| _fail_ \| _rm_ \| _rmfailed_ \| _set\_state_ \| _stat_ \| _repaired_ \] …**ceph** **mon** \[ _add_ \| _dump_ \| _getmap_ \| _remove_ \| _stat_ \] …**ceph** **mon\_statusceph** **osd** \[ _blacklist_ \| _blocked-by_ \| _create_ \| _new_ \| _deep-scrub_ \| _df_ \| _down_ \| _dump_ \| _erasure-code-profile_ \| _find_ \| _getcrushmap_ \| _getmap_ \| _getmaxosd_ \| _in_ \| _ls_ \| _lspools_ \| _map_ \| _metadata_ \| _ok-to-stop_ \| _out_ \| _pause_ \| _perf_ \| _pg-temp_ \| _force-create-pg_ \| _primary-affinity_ \| _primary-temp_ \| _repair_ \| _reweight_ \| _reweight-by-pg_ \| _rm_ \| _destroy_ \| _purge_ \| _safe-to-destroy_ \| _scrub_ \| _set_ \| _setcrushmap_ \| _setmaxosd_ \| _stat_ \| _tree_ \| _unpause_ \| _unset_ \] …**ceph** **osd** **crush** \[ _add_ \| _add-bucket_ \| _create-or-move_ \| _dump_ \| _get-tunable_ \| _link_ \| _move_ \| _remove_ \| _rename-bucket_ \| _reweight_ \| _reweight-all_ \| _reweight-subtree_ \| _rm_ \| _rule_ \| _set_ \| _set-tunable_ \| _show-tunables_ \| _tunables_ \| _unlink_ \] …**ceph** **osd** **pool** \[ _create_ \| _delete_ \| _get_ \| _get-quota_ \| _ls_ \| _mksnap_ \| _rename_ \| _rmsnap_ \| _set_ \| _set-quota_ \| _stats_ \] …**ceph** **osd** **pool** **application** \[ _disable_ \| _enable_ \| _get_ \| _rm_ \| _set_ \] …**ceph** **osd** **tier** \[ _add_ \| _add-cache_ \| _cache-mode_ \| _remove_ \| _remove-overlay_ \| _set-overlay_ \] …**ceph** **pg** \[ _debug_ \| _deep-scrub_ \| _dump_ \| _dump\_json_ \| _dump\_pools\_json_ \| _dump\_stuck_ \| _getmap_ \| _ls_ \| _ls-by-osd_ \| _ls-by-pool_ \| _ls-by-primary_ \| _map_ \| _repair_ \| _scrub_ \| _stat_ \] …**ceph** **quorum** \[ _enter_ \| _exit_ \]**ceph** **quorum\_statusceph** **report** { _&lt;tags&gt;_ \[ _&lt;tags&gt;…_ \] }**ceph** **scrubceph** **statusceph** **sync** **force** {–yes-i-really-mean-it} {–i-know-what-i-am-doing}**ceph** **tell** _&lt;name \(type.id\)&gt; &lt;command&gt; \[options…\]_**ceph** **version**

### DESCRIPTION

**ceph** is a control utility which is used for manual deployment and maintenance of a Ceph cluster. It provides a diverse set of commands that allows deployment of monitors, OSDs, placement groups, MDS and overall maintenance, administration of the cluster.

### COMMANDS

#### AUTH

Manage authentication keys. It is used for adding, removing, exporting or updating of authentication keys for a particular entity such as a monitor or OSD. It uses some additional subcommands.

Subcommand `add` adds authentication info for a particular entity from input file, or random key if no input is given and/or any caps specified in the command.

Usage:

```text
ceph auth add <entity> {<caps> [<caps>...]}
```

Subcommand `caps` updates caps for **name** from caps specified in the command.

Usage:

```text
ceph auth caps <entity> <caps> [<caps>...]
```

Subcommand `del` deletes all caps for `name`.

Usage:

```text
ceph auth del <entity>
```

Subcommand `export` writes keyring for requested entity, or master keyring if none given.

Usage:

```text
ceph auth export {<entity>}
```

Subcommand `get` writes keyring file with requested key.

Usage:

```text
ceph auth get <entity>
```

Subcommand `get-key` displays requested key.

Usage:

```text
ceph auth get-key <entity>
```

Subcommand `get-or-create` adds authentication info for a particular entity from input file, or random key if no input given and/or any caps specified in the command.

Usage:

```text
ceph auth get-or-create <entity> {<caps> [<caps>...]}
```

Subcommand `get-or-create-key` gets or adds key for `name` from system/caps pairs specified in the command. If key already exists, any given caps must match the existing caps for that key.

Usage:

```text
ceph auth get-or-create-key <entity> {<caps> [<caps>...]}
```

Subcommand `import` reads keyring from input file.

Usage:

```text
ceph auth import
```

Subcommand `ls` lists authentication state.

Usage:

```text
ceph auth ls
```

Subcommand `print-key` displays requested key.

Usage:

```text
ceph auth print-key <entity>
```

Subcommand `print_key` displays requested key.

Usage:

```text
ceph auth print_key <entity>
```

#### COMPACT

Causes compaction of monitor’s leveldb storage.

Usage:

```text
ceph compact
```

#### CONFIG-KEY

Manage configuration key. It uses some additional subcommands.

Subcommand `rm` deletes configuration key.

Usage:

```text
ceph config-key rm <key>
```

Subcommand `exists` checks for configuration keys existence.

Usage:

```text
ceph config-key exists <key>
```

Subcommand `get` gets the configuration key.

Usage:

```text
ceph config-key get <key>
```

Subcommand `ls` lists configuration keys.

Usage:

```text
ceph config-key ls
```

Subcommand `dump` dumps configuration keys and values.

Usage:

```text
ceph config-key dump
```

Subcommand `set` puts configuration key and value.

Usage:

```text
ceph config-key set <key> {<val>}
```

#### DAEMON

Submit admin-socket commands.

Usage:

```text
ceph daemon {daemon_name|socket_path} {command} ...
```

Example:

```text
ceph daemon osd.0 help
```

#### DAEMONPERF

Watch performance counters from a Ceph daemon.

Usage:

```text
ceph daemonperf {daemon_name|socket_path} [{interval} [{count}]]
```

#### DF

Show cluster’s free space status.

Usage:

```text
ceph df {detail}
```

#### FEATURES

Show the releases and features of all connected daemons and clients connected to the cluster, along with the numbers of them in each bucket grouped by the corresponding features/releases. Each release of Ceph supports a different set of features, expressed by the features bitmask. New cluster features require that clients support the feature, or else they are not allowed to connect to these new features. As new features or capabilities are enabled after an upgrade, older clients are prevented from connecting.

Usage:

```text
ceph features
```

#### FS

Manage cephfs filesystems. It uses some additional subcommands.

Subcommand `ls` to list filesystems

Usage:

```text
ceph fs ls
```

Subcommand `new` to make a new filesystem using named pools &lt;metadata&gt; and &lt;data&gt;

Usage:

```text
ceph fs new <fs_name> <metadata> <data>
```

Subcommand `reset` is used for disaster recovery only: reset to a single-MDS map

Usage:

```text
ceph fs reset <fs_name> {--yes-i-really-mean-it}
```

Subcommand `rm` to disable the named filesystem

Usage:

```text
ceph fs rm <fs_name> {--yes-i-really-mean-it}
```

#### FSID

Show cluster’s FSID/UUID.

Usage:

```text
ceph fsid
```

#### HEALTH

Show cluster’s health.

Usage:

```text
ceph health {detail}
```

#### HEAP

Show heap usage info \(available only if compiled with tcmalloc\)

Usage:

```text
ceph heap dump|start_profiler|stop_profiler|stats
```

Subcommand `release` to make TCMalloc to releases no-longer-used memory back to the kernel at once.

Usage:

```text
ceph heap release
```

Subcommand `(get|set)_release_rate` get or set the TCMalloc memory release rate. TCMalloc releases no-longer-used memory back to the kernel gradually. the rate controls how quickly this happens. Increase this setting to make TCMalloc to return unused memory more frequently. 0 means never return memory to system, 1 means wait for 1000 pages after releasing a page to system. It is `1.0` by default..

Usage:

```text
ceph heap get_release_rate|set_release_rate {<val>}
```

#### INJECTARGS

Inject configuration arguments into monitor.

Usage:

```text
ceph injectargs <injected_args> [<injected_args>...]
```

#### LOG

Log supplied text to the monitor log.

Usage:

```text
ceph log <logtext> [<logtext>...]
```

#### MDS

Manage metadata server configuration and administration. It uses some additional subcommands.

Subcommand `compat` manages compatible features. It uses some additional subcommands.

Subcommand `rm_compat` removes compatible feature.

Usage:

```text
ceph mds compat rm_compat <int[0-]>
```

Subcommand `rm_incompat` removes incompatible feature.

Usage:

```text
ceph mds compat rm_incompat <int[0-]>
```

Subcommand `show` shows mds compatibility settings.

Usage:

```text
ceph mds compat show
```

Subcommand `fail` forces mds to status fail.

Usage:

```text
ceph mds fail <role|gid>
```

Subcommand `rm` removes inactive mds.

Usage:

```text
ceph mds rm <int[0-]> <name> (type.id)>
```

Subcommand `rmfailed` removes failed mds.

Usage:

```text
ceph mds rmfailed <int[0-]>
```

Subcommand `set_state` sets mds state of &lt;gid&gt; to &lt;numeric-state&gt;.

Usage:

```text
ceph mds set_state <int[0-]> <int[0-20]>
```

Subcommand `stat` shows MDS status.

Usage:

```text
ceph mds stat
```

Subcommand `repaired` mark a damaged MDS rank as no longer damaged.

Usage:

```text
ceph mds repaired <role>
```

#### MON

Manage monitor configuration and administration. It uses some additional subcommands.

Subcommand `add` adds new monitor named &lt;name&gt; at &lt;addr&gt;.

Usage:

```text
ceph mon add <name> <IPaddr[:port]>
```

Subcommand `dump` dumps formatted monmap \(optionally from epoch\)

Usage:

```text
ceph mon dump {<int[0-]>}
```

Subcommand `getmap` gets monmap.

Usage:

```text
ceph mon getmap {<int[0-]>}
```

Subcommand `remove` removes monitor named &lt;name&gt;.

Usage:

```text
ceph mon remove <name>
```

Subcommand `stat` summarizes monitor status.

Usage:

```text
ceph mon stat
```

#### MON\_STATUS

Reports status of monitors.

Usage:

```text
ceph mon_status
```

#### MGR

Ceph manager daemon configuration and management.

Subcommand `dump` dumps the latest MgrMap, which describes the active and standby manager daemons.

Usage:

```text
ceph mgr dump
```

Subcommand `fail` will mark a manager daemon as failed, removing it from the manager map. If it is the active manager daemon a standby will take its place.

Usage:

```text
ceph mgr fail <name>
```

Subcommand `module ls` will list currently enabled manager modules \(plugins\).

Usage:

```text
ceph mgr module ls
```

Subcommand `module enable` will enable a manager module. Available modules are included in MgrMap and visible via `mgr dump`.

Usage:

```text
ceph mgr module enable <module>
```

Subcommand `module disable` will disable an active manager module.

Usage:

```text
ceph mgr module disable <module>
```

Subcommand `metadata` will report metadata about all manager daemons or, if the name is specified, a single manager daemon.

Usage:

```text
ceph mgr metadata [name]
```

Subcommand `versions` will report a count of running daemon versions.

Usage:

```text
ceph mgr versions
```

Subcommand `count-metadata` will report a count of any daemon metadata field.

Usage:

```text
ceph mgr count-metadata <field>
```

#### OSD

Manage OSD configuration and administration. It uses some additional subcommands.

Subcommand `blacklist` manage blacklisted clients. It uses some additional subcommands.

Subcommand `add` add &lt;addr&gt; to blacklist \(optionally until &lt;expire&gt; seconds from now\)

Usage:

```text
ceph osd blacklist add <EntityAddr> {<float[0.0-]>}
```

Subcommand `ls` show blacklisted clients

Usage:

```text
ceph osd blacklist ls
```

Subcommand `rm` remove &lt;addr&gt; from blacklist

Usage:

```text
ceph osd blacklist rm <EntityAddr>
```

Subcommand `blocked-by` prints a histogram of which OSDs are blocking their peers

Usage:

```text
ceph osd blocked-by
```

Subcommand `create` creates new osd \(with optional UUID and ID\).

This command is DEPRECATED as of the Luminous release, and will be removed in a future release.

Subcommand `new` should instead be used.

Usage:

```text
ceph osd create {<uuid>} {<id>}
```

Subcommand `new` can be used to create a new OSD or to recreate a previously destroyed OSD with a specific _id_. The new OSD will have the specified _uuid_, and the command expects a JSON file containing the base64 cephx key for auth entity _client.osd.&lt;id&gt;_, as well as optional base64 cepx key for dm-crypt lockbox access and a dm-crypt key. Specifying a dm-crypt requires specifying the accompanying lockbox cephx key.

Usage:

```text
ceph osd new {<uuid>} {<id>} -i {<params.json>}
```

The parameters JSON file is optional but if provided, is expected to maintain a form of the following format:

```text
{
    "cephx_secret": "AQBWtwhZdBO5ExAAIDyjK2Bh16ZXylmzgYYEjg==",
    "crush_device_class": "myclass"
}
```

Or:

```text
{
    "cephx_secret": "AQBWtwhZdBO5ExAAIDyjK2Bh16ZXylmzgYYEjg==",
    "cephx_lockbox_secret": "AQDNCglZuaeVCRAAYr76PzR1Anh7A0jswkODIQ==",
    "dmcrypt_key": "<dm-crypt key>",
    "crush_device_class": "myclass"
}
```

Or:

```text
{
    "crush_device_class": "myclass"
}
```

The “crush\_device\_class” property is optional. If specified, it will set the initial CRUSH device class for the new OSD.

Subcommand `crush` is used for CRUSH management. It uses some additional subcommands.

Subcommand `add` adds or updates crushmap position and weight for &lt;name&gt; with &lt;weight&gt; and location &lt;args&gt;.

Usage:

```text
ceph osd crush add <osdname (id|osd.id)> <float[0.0-]> <args> [<args>...]
```

Subcommand `add-bucket` adds no-parent \(probably root\) crush bucket &lt;name&gt; of type &lt;type&gt;.

Usage:

```text
ceph osd crush add-bucket <name> <type>
```

Subcommand `create-or-move` creates entry or moves existing entry for &lt;name&gt; &lt;weight&gt; at/to location &lt;args&gt;.

Usage:

```text
ceph osd crush create-or-move <osdname (id|osd.id)> <float[0.0-]> <args>
[<args>...]
```

Subcommand `dump` dumps crush map.

Usage:

```text
ceph osd crush dump
```

Subcommand `get-tunable` get crush tunable straw\_calc\_version

Usage:

```text
ceph osd crush get-tunable straw_calc_version
```

Subcommand `link` links existing entry for &lt;name&gt; under location &lt;args&gt;.

Usage:

```text
ceph osd crush link <name> <args> [<args>...]
```

Subcommand `move` moves existing entry for &lt;name&gt; to location &lt;args&gt;.

Usage:

```text
ceph osd crush move <name> <args> [<args>...]
```

Subcommand `remove` removes &lt;name&gt; from crush map \(everywhere, or just at &lt;ancestor&gt;\).

Usage:

```text
ceph osd crush remove <name> {<ancestor>}
```

Subcommand `rename-bucket` renames bucket &lt;srcname&gt; to &lt;dstname&gt;

Usage:

```text
ceph osd crush rename-bucket <srcname> <dstname>
```

Subcommand `reweight` change &lt;name&gt;’s weight to &lt;weight&gt; in crush map.

Usage:

```text
ceph osd crush reweight <name> <float[0.0-]>
```

Subcommand `reweight-all` recalculate the weights for the tree to ensure they sum correctly

Usage:

```text
ceph osd crush reweight-all
```

Subcommand `reweight-subtree` changes all leaf items beneath &lt;name&gt; to &lt;weight&gt; in crush map

Usage:

```text
ceph osd crush reweight-subtree <name> <weight>
```

Subcommand `rm` removes &lt;name&gt; from crush map \(everywhere, or just at &lt;ancestor&gt;\).

Usage:

```text
ceph osd crush rm <name> {<ancestor>}
```

Subcommand `rule` is used for creating crush rules. It uses some additional subcommands.

Subcommand `create-erasure` creates crush rule &lt;name&gt; for erasure coded pool created with &lt;profile&gt; \(default default\).

Usage:

```text
ceph osd crush rule create-erasure <name> {<profile>}
```

Subcommand `create-simple` creates crush rule &lt;name&gt; to start from &lt;root&gt;, replicate across buckets of type &lt;type&gt;, using a choose mode of &lt;firstn\|indep&gt; \(default firstn; indep best for erasure pools\).

Usage:

```text
ceph osd crush rule create-simple <name> <root> <type> {firstn|indep}
```

Subcommand `dump` dumps crush rule &lt;name&gt; \(default all\).

Usage:

```text
ceph osd crush rule dump {<name>}
```

Subcommand `ls` lists crush rules.

Usage:

```text
ceph osd crush rule ls
```

Subcommand `rm` removes crush rule &lt;name&gt;.

Usage:

```text
ceph osd crush rule rm <name>
```

Subcommand `set` used alone, sets crush map from input file.

Usage:

```text
ceph osd crush set
```

Subcommand `set` with osdname/osd.id update crushmap position and weight for &lt;name&gt; to &lt;weight&gt; with location &lt;args&gt;.

Usage:

```text
ceph osd crush set <osdname (id|osd.id)> <float[0.0-]> <args> [<args>...]
```

Subcommand `set-tunable` set crush tunable &lt;tunable&gt; to &lt;value&gt;. The only tunable that can be set is straw\_calc\_version.

Usage:

```text
ceph osd crush set-tunable straw_calc_version <value>
```

Subcommand `show-tunables` shows current crush tunables.

Usage:

```text
ceph osd crush show-tunables
```

Subcommand `tree` shows the crush buckets and items in a tree view.

Usage:

```text
ceph osd crush tree
```

Subcommand `tunables` sets crush tunables values to &lt;profile&gt;.

Usage:

```text
ceph osd crush tunables legacy|argonaut|bobtail|firefly|hammer|optimal|default
```

Subcommand `unlink` unlinks &lt;name&gt; from crush map \(everywhere, or just at &lt;ancestor&gt;\).

Usage:

```text
ceph osd crush unlink <name> {<ancestor>}
```

Subcommand `df` shows OSD utilization

Usage:

```text
ceph osd df {plain|tree}
```

Subcommand `deep-scrub` initiates deep scrub on specified osd.

Usage:

```text
ceph osd deep-scrub <who>
```

Subcommand `down` sets osd\(s\) &lt;id&gt; \[&lt;id&gt;…\] down.

Usage:

```text
ceph osd down <ids> [<ids>...]
```

Subcommand `dump` prints summary of OSD map.

Usage:

```text
ceph osd dump {<int[0-]>}
```

Subcommand `erasure-code-profile` is used for managing the erasure code profiles. It uses some additional subcommands.

Subcommand `get` gets erasure code profile &lt;name&gt;.

Usage:

```text
ceph osd erasure-code-profile get <name>
```

Subcommand `ls` lists all erasure code profiles.

Usage:

```text
ceph osd erasure-code-profile ls
```

Subcommand `rm` removes erasure code profile &lt;name&gt;.

Usage:

```text
ceph osd erasure-code-profile rm <name>
```

Subcommand `set` creates erasure code profile &lt;name&gt; with \[&lt;key\[=value\]&gt; …\] pairs. Add a –force at the end to override an existing profile \(IT IS RISKY\).

Usage:

```text
ceph osd erasure-code-profile set <name> {<profile> [<profile>...]}
```

Subcommand `find` find osd &lt;id&gt; in the CRUSH map and shows its location.

Usage:

```text
ceph osd find <int[0-]>
```

Subcommand `getcrushmap` gets CRUSH map.

Usage:

```text
ceph osd getcrushmap {<int[0-]>}
```

Subcommand `getmap` gets OSD map.

Usage:

```text
ceph osd getmap {<int[0-]>}
```

Subcommand `getmaxosd` shows largest OSD id.

Usage:

```text
ceph osd getmaxosd
```

Subcommand `in` sets osd\(s\) &lt;id&gt; \[&lt;id&gt;…\] in.

Usage:

```text
ceph osd in <ids> [<ids>...]
```

Subcommand `lost` marks osd as permanently lost. THIS DESTROYS DATA IF NO MORE REPLICAS EXIST, BE CAREFUL.

Usage:

```text
ceph osd lost <int[0-]> {--yes-i-really-mean-it}
```

Subcommand `ls` shows all OSD ids.

Usage:

```text
ceph osd ls {<int[0-]>}
```

Subcommand `lspools` lists pools.

Usage:

```text
ceph osd lspools {<int>}
```

Subcommand `map` finds pg for &lt;object&gt; in &lt;pool&gt;.

Usage:

```text
ceph osd map <poolname> <objectname>
```

Subcommand `metadata` fetches metadata for osd &lt;id&gt;.

Usage:

```text
ceph osd metadata {int[0-]} (default all)
```

Subcommand `out` sets osd\(s\) &lt;id&gt; \[&lt;id&gt;…\] out.

Usage:

```text
ceph osd out <ids> [<ids>...]
```

Subcommand `ok-to-stop` checks whether the list of OSD\(s\) can be stopped without immediately making data unavailable. That is, all data should remain readable and writeable, although data redundancy may be reduced as some PGs may end up in a degraded \(but active\) state. It will return a success code if it is okay to stop the OSD\(s\), or an error code and informative message if it is not or if no conclusion can be drawn at the current time.

Usage:

```text
ceph osd ok-to-stop <id> [<ids>...]
```

Subcommand `pause` pauses osd.

Usage:

```text
ceph osd pause
```

Subcommand `perf` prints dump of OSD perf summary stats.

Usage:

```text
ceph osd perf
```

Subcommand `pg-temp` set pg\_temp mapping pgid:\[&lt;id&gt; \[&lt;id&gt;…\]\] \(developers only\).

Usage:

```text
ceph osd pg-temp <pgid> {<id> [<id>...]}
```

Subcommand `force-create-pg` forces creation of pg &lt;pgid&gt;.

Usage:

```text
ceph osd force-create-pg <pgid>
```

Subcommand `pool` is used for managing data pools. It uses some additional subcommands.

Subcommand `create` creates pool.

Usage:

```text
ceph osd pool create <poolname> <int[0-]> {<int[0-]>} {replicated|erasure}
{<erasure_code_profile>} {<rule>} {<int>}
```

Subcommand `delete` deletes pool.

Usage:

```text
ceph osd pool delete <poolname> {<poolname>} {--yes-i-really-really-mean-it}
```

Subcommand `get` gets pool parameter &lt;var&gt;.

Usage:

```text
ceph osd pool get <poolname> size|min_size|pg_num|pgp_num|crush_rule|write_fadvise_dontneed
```

Only for tiered pools:

```text
ceph osd pool get <poolname> hit_set_type|hit_set_period|hit_set_count|hit_set_fpp|
target_max_objects|target_max_bytes|cache_target_dirty_ratio|cache_target_dirty_high_ratio|
cache_target_full_ratio|cache_min_flush_age|cache_min_evict_age|
min_read_recency_for_promote|hit_set_grade_decay_rate|hit_set_search_last_n
```

Only for erasure coded pools:

```text
ceph osd pool get <poolname> erasure_code_profile
```

Use `all` to get all pool parameters that apply to the pool’s type:

```text
ceph osd pool get <poolname> all
```

Subcommand `get-quota` obtains object or byte limits for pool.

Usage:

```text
ceph osd pool get-quota <poolname>
```

Subcommand `ls` list pools

Usage:

```text
ceph osd pool ls {detail}
```

Subcommand `mksnap` makes snapshot &lt;snap&gt; in &lt;pool&gt;.

Usage:

```text
ceph osd pool mksnap <poolname> <snap>
```

Subcommand `rename` renames &lt;srcpool&gt; to &lt;destpool&gt;.

Usage:

```text
ceph osd pool rename <poolname> <poolname>
```

Subcommand `rmsnap` removes snapshot &lt;snap&gt; from &lt;pool&gt;.

Usage:

```text
ceph osd pool rmsnap <poolname> <snap>
```

Subcommand `set` sets pool parameter &lt;var&gt; to &lt;val&gt;.

Usage:

```text
ceph osd pool set <poolname> size|min_size|pg_num|
pgp_num|crush_rule|hashpspool|nodelete|nopgchange|nosizechange|
hit_set_type|hit_set_period|hit_set_count|hit_set_fpp|debug_fake_ec_pool|
target_max_bytes|target_max_objects|cache_target_dirty_ratio|
cache_target_dirty_high_ratio|
cache_target_full_ratio|cache_min_flush_age|cache_min_evict_age|
min_read_recency_for_promote|write_fadvise_dontneed|hit_set_grade_decay_rate|
hit_set_search_last_n
<val> {--yes-i-really-mean-it}
```

Subcommand `set-quota` sets object or byte limit on pool.

Usage:

```text
ceph osd pool set-quota <poolname> max_objects|max_bytes <val>
```

Subcommand `stats` obtain stats from all pools, or from specified pool.

Usage:

```text
ceph osd pool stats {<name>}
```

Subcommand `application` is used for adding an annotation to the given pool. By default, the possible applications are object, block, and file storage \(corresponding app-names are “rgw”, “rbd”, and “cephfs”\). However, there might be other applications as well. Based on the application, there may or may not be some processing conducted.

Subcommand `disable` disables the given application on the given pool.

Usage:

```text
ceph osd pool application disable <pool-name> <app> {--yes-i-really-mean-it}
```

Subcommand `enable` adds an annotation to the given pool for the mentioned application.

Usage:

```text
ceph osd pool application enable <pool-name> <app> {--yes-i-really-mean-it}
```

Subcommand `get` displays the value for the given key that is assosciated with the given application of the given pool. Not passing the optional arguments would display all key-value pairs for all applications for all pools.

Usage:

```text
ceph osd pool application get {<pool-name>} {<app>} {<key>}
```

Subcommand `rm` removes the key-value pair for the given key in the given application of the given pool.

Usage:

```text
ceph osd pool application rm <pool-name> <app> <key>
```

Subcommand `set` assosciates or updates, if it already exists, a key-value pair with the given application for the given pool.

Usage:

```text
ceph osd pool application set <pool-name> <app> <key> <value>
```

Subcommand `primary-affinity` adjust osd primary-affinity from 0.0 &lt;=&lt;weight&gt; &lt;= 1.0

Usage:

```text
ceph osd primary-affinity <osdname (id|osd.id)> <float[0.0-1.0]>
```

Subcommand `primary-temp` sets primary\_temp mapping pgid:&lt;id&gt;\|-1 \(developers only\).

Usage:

```text
ceph osd primary-temp <pgid> <id>
```

Subcommand `repair` initiates repair on a specified osd.

Usage:

```text
ceph osd repair <who>
```

Subcommand `reweight` reweights osd to 0.0 &lt; &lt;weight&gt; &lt; 1.0.

Usage:

```text
osd reweight <int[0-]> <float[0.0-1.0]>
```

Subcommand `reweight-by-pg` reweight OSDs by PG distribution \[overload-percentage-for-consideration, default 120\].

Usage:

```text
ceph osd reweight-by-pg {<int[100-]>} {<poolname> [<poolname...]}
{--no-increasing}
```

Subcommand `reweight-by-utilization` reweight OSDs by utilization \[overload-percentage-for-consideration, default 120\].

Usage:

```text
ceph osd reweight-by-utilization {<int[100-]>}
{--no-increasing}
```

Subcommand `rm` removes osd\(s\) &lt;id&gt; \[&lt;id&gt;…\] from the OSD map.

Usage:

```text
ceph osd rm <ids> [<ids>...]
```

Subcommand `destroy` marks OSD _id_ as _destroyed_, removing its cephx entity’s keys and all of its dm-crypt and daemon-private config key entries.

This command will not remove the OSD from crush, nor will it remove the OSD from the OSD map. Instead, once the command successfully completes, the OSD will show marked as _destroyed_.

In order to mark an OSD as destroyed, the OSD must first be marked as **lost**.

Usage:

```text
ceph osd destroy <id> {--yes-i-really-mean-it}
```

Subcommand `purge` performs a combination of `osd destroy`, `osd rm` and `osd crush remove`.

Usage:

```text
ceph osd purge <id> {--yes-i-really-mean-it}
```

Subcommand `safe-to-destroy` checks whether it is safe to remove or destroy an OSD without reducing overall data redundancy or durability. It will return a success code if it is definitely safe, or an error code and informative message if it is not or if no conclusion can be drawn at the current time.

Usage:

```text
ceph osd safe-to-destroy <id> [<ids>...]
```

Subcommand `scrub` initiates scrub on specified osd.

Usage:

```text
ceph osd scrub <who>
```

Subcommand `set` sets &lt;key&gt;.

Usage:

```text
ceph osd set full|pause|noup|nodown|noout|noin|nobackfill|
norebalance|norecover|noscrub|nodeep-scrub|notieragent
```

Subcommand `setcrushmap` sets crush map from input file.

Usage:

```text
ceph osd setcrushmap
```

Subcommand `setmaxosd` sets new maximum osd value.

Usage:

```text
ceph osd setmaxosd <int[0-]>
```

Subcommand `set-require-min-compat-client` enforces the cluster to be backward compatible with the specified client version. This subcommand prevents you from making any changes \(e.g., crush tunables, or using new features\) that would violate the current setting. Please note, This subcommand will fail if any connected daemon or client is not compatible with the features offered by the given &lt;version&gt;. To see the features and releases of all clients connected to cluster, please see [ceph features](https://docs.ceph.com/docs/nautilus/man/8/ceph/#ceph-features).

Usage:

```text
ceph osd set-require-min-compat-client <version>
```

Subcommand `stat` prints summary of OSD map.

Usage:

```text
ceph osd stat
```

Subcommand `tier` is used for managing tiers. It uses some additional subcommands.

Subcommand `add` adds the tier &lt;tierpool&gt; \(the second one\) to base pool &lt;pool&gt; \(the first one\).

Usage:

```text
ceph osd tier add <poolname> <poolname> {--force-nonempty}
```

Subcommand `add-cache` adds a cache &lt;tierpool&gt; \(the second one\) of size &lt;size&gt; to existing pool &lt;pool&gt; \(the first one\).

Usage:

```text
ceph osd tier add-cache <poolname> <poolname> <int[0-]>
```

Subcommand `cache-mode` specifies the caching mode for cache tier &lt;pool&gt;.

Usage:

```text
ceph osd tier cache-mode <poolname> none|writeback|forward|readonly|
readforward|readproxy
```

Subcommand `remove` removes the tier &lt;tierpool&gt; \(the second one\) from base pool &lt;pool&gt; \(the first one\).

Usage:

```text
ceph osd tier remove <poolname> <poolname>
```

Subcommand `remove-overlay` removes the overlay pool for base pool &lt;pool&gt;.

Usage:

```text
ceph osd tier remove-overlay <poolname>
```

Subcommand `set-overlay` set the overlay pool for base pool &lt;pool&gt; to be &lt;overlaypool&gt;.

Usage:

```text
ceph osd tier set-overlay <poolname> <poolname>
```

Subcommand `tree` prints OSD tree.

Usage:

```text
ceph osd tree {<int[0-]>}
```

Subcommand `unpause` unpauses osd.

Usage:

```text
ceph osd unpause
```

Subcommand `unset` unsets &lt;key&gt;.

Usage:

```text
ceph osd unset full|pause|noup|nodown|noout|noin|nobackfill|
norebalance|norecover|noscrub|nodeep-scrub|notieragent
```

#### PG

It is used for managing the placement groups in OSDs. It uses some additional subcommands.

Subcommand `debug` shows debug info about pgs.

Usage:

```text
ceph pg debug unfound_objects_exist|degraded_pgs_exist
```

Subcommand `deep-scrub` starts deep-scrub on &lt;pgid&gt;.

Usage:

```text
ceph pg deep-scrub <pgid>
```

Subcommand `dump` shows human-readable versions of pg map \(only ‘all’ valid with plain\).

Usage:

```text
ceph pg dump {all|summary|sum|delta|pools|osds|pgs|pgs_brief} [{all|summary|sum|delta|pools|osds|pgs|pgs_brief...]}
```

Subcommand `dump_json` shows human-readable version of pg map in json only.

Usage:

```text
ceph pg dump_json {all|summary|sum|delta|pools|osds|pgs|pgs_brief} [{all|summary|sum|delta|pools|osds|pgs|pgs_brief...]}
```

Subcommand `dump_pools_json` shows pg pools info in json only.

Usage:

```text
ceph pg dump_pools_json
```

Subcommand `dump_stuck` shows information about stuck pgs.

Usage:

```text
ceph pg dump_stuck {inactive|unclean|stale|undersized|degraded [inactive|unclean|stale|undersized|degraded...]}
{<int>}
```

Subcommand `getmap` gets binary pg map to -o/stdout.

Usage:

```text
ceph pg getmap
```

Subcommand `ls` lists pg with specific pool, osd, state

Usage:

```text
ceph pg ls {<int>} {<pg-state> [<pg-state>...]}
```

Subcommand `ls-by-osd` lists pg on osd \[osd\]

Usage:

```text
ceph pg ls-by-osd <osdname (id|osd.id)> {<int>}
{<pg-state> [<pg-state>...]}
```

Subcommand `ls-by-pool` lists pg with pool = \[poolname\]

Usage:

```text
ceph pg ls-by-pool <poolstr> {<int>} {<pg-state> [<pg-state>...]}
```

Subcommand `ls-by-primary` lists pg with primary = \[osd\]

Usage:

```text
ceph pg ls-by-primary <osdname (id|osd.id)> {<int>}
{<pg-state> [<pg-state>...]}
```

Subcommand `map` shows mapping of pg to osds.

Usage:

```text
ceph pg map <pgid>
```

Subcommand `repair` starts repair on &lt;pgid&gt;.

Usage:

```text
ceph pg repair <pgid>
```

Subcommand `scrub` starts scrub on &lt;pgid&gt;.

Usage:

```text
ceph pg scrub <pgid>
```

Subcommand `stat` shows placement group status.

Usage:

```text
ceph pg stat
```

#### QUORUM

Cause MON to enter or exit quorum.

Usage:

```text
ceph quorum enter|exit
```

Note: this only works on the MON to which the `ceph` command is connected. If you want a specific MON to enter or exit quorum, use this syntax:

```text
ceph tell mon.<id> quorum enter|exit
```

#### QUORUM\_STATUS

Reports status of monitor quorum.

Usage:

```text
ceph quorum_status
```

#### REPORT

Reports full status of cluster, optional title tag strings.

Usage:

```text
ceph report {<tags> [<tags>...]}
```

#### SCRUB

Scrubs the monitor stores.

Usage:

```text
ceph scrub
```

#### STATUS

Shows cluster status.

Usage:

```text
ceph status
```

#### SYNC FORCE

Forces sync of and clear monitor store.

Usage:

```text
ceph sync force {--yes-i-really-mean-it} {--i-know-what-i-am-doing}
```

#### TELL

Sends a command to a specific daemon.

Usage:

```text
ceph tell <name (type.id)> <command> [options...]
```

List all available commands.

Usage:

```text
ceph tell <name (type.id)> help
```

#### VERSION

Show mon daemon version

Usage:

```text
ceph version
```

### OPTIONS

`-i infile`

will specify an input file to be passed along as a payload with the command to the monitor cluster. This is only used for specific monitor commands.`-o outfile`

will write any payload returned by the monitor cluster with its reply to outfile. Only specific monitor commands \(e.g. osd getmap\) return a payload.`--setuser user`

will apply the appropriate user ownership to the file specified by the option ‘-o’.`--setgroup group`

will apply the appropriate group ownership to the file specified by the option ‘-o’.`-c ceph.conf, --conf=ceph.conf`

Use ceph.conf configuration file instead of the default `/etc/ceph/ceph.conf` to determine monitor addresses during startup.`--id CLIENT_ID, --user CLIENT_ID`

Client id for authentication.`--name CLIENT_NAME, -n CLIENT_NAME`

Client name for authentication.`--cluster CLUSTER`

Name of the Ceph cluster.`--admin-daemon ADMIN_SOCKET, daemon DAEMON_NAME`

Submit admin-socket commands via admin sockets in /var/run/ceph.`--admin-socket ADMIN_SOCKET_NOPE`

You probably mean –admin-daemon`-s, --status`

Show cluster status.`-w, --watch`

Watch live cluster changes.`--watch-debug`

Watch debug events.`--watch-info`

Watch info events.`--watch-sec`

Watch security events.`--watch-warn`

Watch warning events.`--watch-error`

Watch error events.`--version, -v`

Display version.`--verbose`

Make verbose.`--concise`

Make less verbose.`-f {json,json-pretty,xml,xml-pretty,plain}, --format`

Format of output.`--connect-timeout CLUSTER_TIMEOUT`

Set a timeout for connecting to the cluster.`--no-increasing`

`--no-increasing` is off by default. So increasing the osd weight is allowed using the `reweight-by-utilization` or `test-reweight-by-utilization` commands. If this option is used with these commands, it will help not to increase osd weight even the osd is under utilized.`--block`

block until completion \(scrub and deep-scrub only\)

### AVAILABILITY

**ceph** is part of Ceph, a massively scalable, open-source, distributed storage system. Please refer to the Ceph documentation at [http://ceph.com/docs](http://ceph.com/docs) for more information.

### SEE ALSO

[ceph-mon](https://docs.ceph.com/docs/nautilus/man/8/ceph-mon/)\(8\), [ceph-osd](https://docs.ceph.com/docs/nautilus/man/8/ceph-osd/)\(8\), [ceph-mds](https://docs.ceph.com/docs/nautilus/man/8/ceph-mds/)\(8\)

