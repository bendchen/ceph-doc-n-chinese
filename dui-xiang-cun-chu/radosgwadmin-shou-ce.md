# RADOSGW-ADMIN手册

## RADOSGW-ADMIN – RADOS REST GATEWAY USER ADMINISTRATION UTILITY

### SYNOPSIS

**radosgw-admin** _command_ \[ _options_ _…_ \]

### DESCRIPTION

**radosgw-admin** is a RADOS gateway user administration utility. It allows creating and modifying users.

### COMMANDS

**radosgw-admin** utility uses many commands for administration purpose which are as follows:**user create**

Create a new user.**user modify**

Modify a user.**user info**

Display information of a user, and any potentially available subusers and keys.**user rm**

Remove a user.**user suspend**

Suspend a user.**user enable**

Re-enable user after suspension.**user check**

Check user info.**user stats**

Show user stats as accounted by quota subsystem.**user list**

List all users.**caps add**

Add user capabilities.**caps rm**

Remove user capabilities.**subuser create**

Create a new subuser \(primarily useful for clients using the Swift API\).**subuser modify**

Modify a subuser.**subuser rm**

Remove a subuser.**key create**

Create access key.**key rm**

Remove access key.**bucket list**

List buckets, or, if bucket specified with –bucket=&lt;bucket&gt;, list its objects. If bucket specified adding –allow-unordered removes ordering requirement, possibly generating results more quickly in buckets with large number of objects.**bucket limit check**

Show bucket sharding stats.**bucket link**

Link bucket to specified user.**bucket unlink**

Unlink bucket from specified user.**bucket stats**

Returns bucket statistics.**bucket rm**

Remove a bucket.**bucket check**

Check bucket index.**bucket rewrite**

Rewrite all objects in the specified bucket.**bucket reshard**

Reshard a bucket.**bucket sync disable**

Disable bucket sync.**bucket sync enable**

Enable bucket sync.**bi get**

Retrieve bucket index object entries.**bi put**

Store bucket index object entries.**bi list**

List raw bucket index entries.**bi purge**

Purge bucket index entries.**object rm**

Remove an object.**object stat**

Stat an object for its metadata.**object unlink**

Unlink object from bucket index.**object rewrite**

Rewrite the specified object.**objects expire**

Run expired objects cleanup.**period rm**

Remove a period.**period get**

Get the period info.**period get-current**

Get the current period info.**period pull**

Pull a period.**period push**

Push a period.**period list**

List all periods.**period update**

Update the staging period.**period commit**

Commit the staging period.**quota set**

Set quota params.**quota enable**

Enable quota.**quota disable**

Disable quota.**global quota get**

View global quota parameters.**global quota set**

Set global quota parameters.**global quota enable**

Enable a global quota.**global quota disable**

Disable a global quota.**realm create**

Create a new realm.**realm rm**

Remove a realm.**realm get**

Show the realm info.**realm get-default**

Get the default realm name.**realm list**

List all realms.**realm list-periods**

List all realm periods.**realm rename**

Rename a realm.**realm set**

Set the realm info \(requires infile\).**realm default**

Set the realm as default.**realm pull**

Pull a realm and its current period.**zonegroup add**

Add a zone to a zonegroup.**zonegroup create**

Create a new zone group info.**zonegroup default**

Set the default zone group.**zonegroup rm**

Remove a zone group info.**zonegroup get**

Show the zone group info.**zonegroup modify**

Modify an existing zonegroup.**zonegroup set**

Set the zone group info \(requires infile\).**zonegroup remove**

Remove a zone from a zonegroup.**zonegroup rename**

Rename a zone group.**zonegroup list**

List all zone groups set on this cluster.**zonegroup placement list**

List zonegroup’s placement targets.**zonegroup placement add**

Add a placement target id to a zonegroup.**zonegroup placement modify**

Modify a placement target of a specific zonegroup.**zonegroup placement rm**

Remove a placement target from a zonegroup.**zonegroup placement default**

Set a zonegroup’s default placement target.**zone create**

Create a new zone.**zone rm**

Remove a zone.**zone get**

Show zone cluster params.**zone set**

Set zone cluster params \(requires infile\).**zone modify**

Modify an existing zone.**zone list**

List all zones set on this cluster.**metadata sync status**

Get metadata sync status.**metadata sync init**

Init metadata sync.**metadata sync run**

Run metadata sync.**data sync status**

Get data sync status of the specified source zone.**data sync init**

Init data sync for the specified source zone.**data sync run**

Run data sync for the specified source zone.**sync error list**

list sync error.**sync error trim**

trim sync error.**zone rename**

Rename a zone.**zone placement list**

List zone’s placement targets.**zone placement add**

Add a zone placement target.**zone placement modify**

Modify a zone placement target.**zone placement rm**

Remove a zone placement target.**pool add**

Add an existing pool for data placement.**pool rm**

Remove an existing pool from data placement set.**pools list**

List placement active set.**policy**

Display bucket/object policy.**log list**

List log objects.**log show**

Dump a log from specific object or \(bucket + date + bucket-id\). \(NOTE: required to specify formatting of date to “YYYY-MM-DD-hh”\)**log rm**

Remove log object.**usage show**

Show the usage information \(with optional user and date range\).**usage trim**

Trim usage information \(with optional user and date range\).**gc list**

Dump expired garbage collection objects \(specify –include-all to list all entries, including unexpired\).**gc process**

Manually process garbage.**lc list**

List all bucket lifecycle progress.**lc process**

Manually process lifecycle.**metadata get**

Get metadata info.**metadata put**

Put metadata info.**metadata rm**

Remove metadata info.**metadata list**

List metadata info.**mdlog list**

List metadata log.**mdlog trim**

Trim metadata log.**mdlog status**

Read metadata log status.**bilog list**

List bucket index log.**bilog trim**

Trim bucket index log \(use start-marker, end-marker\).**datalog list**

List data log.**datalog trim**

Trim data log.**datalog status**

Read data log status.**orphans find**

Init and run search for leaked rados objects**orphans finish**

Clean up search for leaked rados objects**orphans list-jobs**

List the current job-ids for the orphans search.**role create**

create a new AWS role for use with STS.**role rm**

Remove a role.**role get**

Get a role.**role list**

List the roles with specified path prefix.**role modify**

Modify the assume role policy of an existing role.**role-policy put**

Add/update permission policy to role.**role-policy list**

List the policies attached to a role.**role-policy get**

Get the specified inline policy document embedded with the given role.**role-policy rm**

Remove the policy attached to a role**reshard add**

Schedule a resharding of a bucket**reshard list**

List all bucket resharding or scheduled to be resharded**reshard process**

Process of scheduled reshard jobs**reshard status**

Resharding status of a bucket**reshard cancel**

Cancel resharding a bucket

### OPTIONS

`-c ceph.conf, --conf=ceph.conf`

Use `ceph.conf` configuration file instead of the default `/etc/ceph/ceph.conf` to determine monitor addresses during startup.`-m monaddress[:port]`

Connect to specified monitor \(instead of looking through ceph.conf\).`--tenant=<tenant>`

Name of the tenant.`--uid=uid`

The radosgw user ID.`--subuser=<name>`

Name of the subuser.`--access-key=<key>`

S3 access key.`--email=email`

The e-mail address of the user.`--secret/--secret-key=<key>`

The secret key.`--gen-access-key`

Generate random access key \(for S3\).`--gen-secret`

Generate random secret key.`--key-type=<type>`

key type, options are: swift, s3.`--temp-url-key[-2]=<key>`

Temporary url key.`--max-buckets`

max number of buckets for a user \(0 for no limit, negative value to disable bucket creation\). Default is 1000.`--access=<access>`

Set the access permissions for the sub-user. Available access permissions are read, write, readwrite and full.`--display-name=<name>`

The display name of the user.`--admin`

Set the admin flag on the user.`--system`

Set the system flag on the user.`--bucket=bucket`

Specify the bucket name.`--pool=<pool>`

Specify the pool name. Also used with orphans find as data pool to scan for leaked rados objects.`--object=object`

Specify the object name.`--date=yyyy-mm-dd`

The date in the format yyyy-mm-dd.`--start-date=yyyy-mm-dd`

The start date in the format yyyy-mm-dd.`--end-date=yyyy-mm-dd`

The end date in the format yyyy-mm-dd.`--bucket-id=<bucket-id>`

Specify the bucket id.`--shard-id=<shard-id>`

Optional for mdlog list, data sync status. Required for `mdlog trim`.`--max-entries=<entries>`

Optional for listing operations to specify the max entires`--purge-data`

When specified, user removal will also purge all the user data.`--purge-keys`

When specified, subuser removal will also purge all the subuser keys.`--purge-objects`

When specified, the bucket removal will also purge all objects in it.`--metadata-key=<key>`

Key to retrieve metadata from with `metadata get`.`--remote=<remote>`

Zone or zonegroup id of remote gateway.`--period=<id>`

Period id.`--url=<url>`

url for pushing/pulling period or realm.`--epoch=<number>`

Period epoch.`--commit`

Commit the period during ‘period update’.`--staging`

Get the staging period info.`--master`

Set as master.`--master-zone=<id>`

Master zone id.`--rgw-realm=<name>`

The realm name.`--realm-id=<id>`

The realm id.`--realm-new-name=<name>`

New name of realm.`--rgw-zonegroup=<name>`

The zonegroup name.`--zonegroup-id=<id>`

The zonegroup id.`--zonegroup-new-name=<name>`

The new name of the zonegroup.`--rgw-zone=<zone>`

Zone in which radosgw is running.`--zone-id=<id>`

The zone id.`--zone-new-name=<name>`

The new name of the zone.`--source-zone`

The source zone for data sync.`--default`

Set the entity \(realm, zonegroup, zone\) as default.`--read-only`

Set the zone as read-only when adding to the zonegroup.`--placement-id`

Placement id for the zonegroup placement commands.`--tags=<list>`

The list of tags for zonegroup placement add and modify commands.`--tags-add=<list>`

The list of tags to add for zonegroup placement modify command.`--tags-rm=<list>`

The list of tags to remove for zonegroup placement modify command.`--endpoints=<list>`

The zone endpoints.`--index-pool=<pool>`

The placement target index pool.`--data-pool=<pool>`

The placement target data pool.`--data-extra-pool=<pool>`

The placement target data extra \(non-ec\) pool.`--placement-index-type=<type>`

The placement target index type \(normal, indexless, or \#id\).`--tier-type=<type>`

The zone tier type.`--tier-config=<k>=<v>[,...]`

Set zone tier config keys, values.`--tier-config-rm=<k>[,...]`

Unset zone tier config keys.`--sync-from-all[=false]`

Set/reset whether zone syncs from all zonegroup peers.`--sync-from=[zone-name][,...]`

Set the list of zones to sync from.`--sync-from-rm=[zone-name][,...]`

Remove the zones from list of zones to sync from.`--fix`

Besides checking bucket index, will also fix it.`--check-objects`

bucket check: Rebuilds bucket index according to actual objects state.`--format=<format>`

Specify output format for certain operations. Supported formats: xml, json.`--sync-stats`

Option for ‘user stats’ command. When specified, it will update user stats with the current stats reported by user’s buckets indexes.`--show-log-entries=<flag>`

Enable/disable dump of log entries on log show.`--show-log-sum=<flag>`

Enable/disable dump of log summation on log show.`--skip-zero-entries`

Log show only dumps entries that don’t have zero value in one of the numeric field.`--infile`

Specify a file to read in when setting data.`--categories=<list>`

Comma separated list of categories, used in usage show.`--caps=<caps>`

List of caps \(e.g., “usage=read, write; user=read”.`--compression=<compression-algorithm>`

Placement target compression algorithm \(lz4\|snappy\|zlib\|zstd\)`--yes-i-really-mean-it`

Required for certain operations.`--min-rewrite-size`

Specify the min object size for bucket rewrite \(default 4M\).`--max-rewrite-size`

Specify the max object size for bucket rewrite \(default ULLONG\_MAX\).`--min-rewrite-stripe-size`

Specify the min stripe size for object rewrite \(default 0\). If the value is set to 0, then the specified object will always be rewritten for restriping.`--warnings-only`

When specified with bucket limit check, list only buckets nearing or over the current max objects per shard value.`--bypass-gc`

When specified with bucket deletion, triggers object deletions by not involving GC.`--inconsistent-index`

When specified with bucket deletion and bypass-gc set to true, ignores bucket index consistency.

### QUOTA OPTIONS

`--max-objects`

Specify max objects \(negative value to disable\).`--max-size`

Specify max size \(in B/K/M/G/T, negative value to disable\).`--quota-scope`

The scope of quota \(bucket, user\).

### ORPHANS SEARCH OPTIONS

`--num-shards`

Number of shards to use for keeping the temporary scan info`--orphan-stale-secs`

Number of seconds to wait before declaring an object to be an orphan. Default is 86400 \(24 hours\).`--job-id`

Set the job id \(for orphans find\)`--max-concurrent-ios`

Maximum concurrent ios for orphans find. Default is 32.

### ORPHANS LIST-JOBS OPTIONS

`--extra-info`

Provide extra info in the job list.

### ROLE OPTIONS

`--role-name`

The name of the role to create.`--path`

The path to the role.`--assume-role-policy-doc`

The trust relationship policy document that grants an entity permission to assume the role.`--policy-name`

The name of the policy document.`--policy-doc`

The permission policy document.`--path-prefix`

The path prefix for filtering the roles.

### EXAMPLES

Generate a new user:

```text
$ radosgw-admin user create --display-name="johnny rotten" --uid=johnny
{ "user_id": "johnny",
  "rados_uid": 0,
  "display_name": "johnny rotten",
  "email": "",
  "suspended": 0,
  "subusers": [],
  "keys": [
        { "user": "johnny",
          "access_key": "TCICW53D9BQ2VGC46I44",
          "secret_key": "tfm9aHMI8X76L3UdgE+ZQaJag1vJQmE6HDb5Lbrz"}],
  "swift_keys": []}
```

Remove a user:

```text
$ radosgw-admin user rm --uid=johnny
```

Remove a user and all associated buckets with their contents:

```text
$ radosgw-admin user rm --uid=johnny --purge-data
```

Remove a bucket:

```text
$ radosgw-admin bucket rm --bucket=foo
```

Link bucket to specified user:

```text
$ radosgw-admin bucket link --bucket=foo --bucket_id=<bucket id> --uid=johnny
```

Unlink bucket from specified user:

```text
$ radosgw-admin bucket unlink --bucket=foo --uid=johnny
```

Show the logs of a bucket from April 1st, 2012:

```text
$ radosgw-admin log show --bucket=foo --date=2012-04-01-01 --bucket-id=default.14193.1
```

Show usage information for user from March 1st to \(but not including\) April 1st, 2012:

```text
$ radosgw-admin usage show --uid=johnny \
                --start-date=2012-03-01 --end-date=2012-04-01
```

Show only summary of usage information for all users:

```text
$ radosgw-admin usage show --show-log-entries=false
```

Trim usage information for user until March 1st, 2012:

```text
$ radosgw-admin usage trim --uid=johnny --end-date=2012-04-01
```

### AVAILABILITY

**radosgw-admin** is part of Ceph, a massively scalable, open-source, distributed storage system. Please refer to the Ceph documentation at [http://ceph.com/docs](http://ceph.com/docs) for more information.

