# MGR开发指导

## CEPH-MGR MODULE DEVELOPER’S GUIDE

Warning 

This is developer documentation, describing Ceph internals that are only relevant to people writing ceph-mgr modules.

### CREATING A MODULE

In pybind/mgr/, create a python module. Within your module, create a class that inherits from `MgrModule`. For ceph-mgr to detect your module, your directory must contain a file called module.py.

The most important methods to override are:

* a `serve` member function for server-type modules. This function should block forever.
* a `notify` member function if your module needs to take action when new cluster data is available.
* a `handle_command` member function if your module exposes CLI commands.

Some modules interface with external orchestrators to deploy Ceph services. These also inherit from `Orchestrator`, which adds additional methods to the base `MgrModule` class. See [Orchestrator modules](https://docs.ceph.com/docs/nautilus/mgr/orchestrator_modules/#orchestrator-modules) for more on creating these modules.

### INSTALLING A MODULE

Once your module is present in the location set by the `mgr module path` configuration setting, you can enable it via the `ceph mgr module enable` command:

```text
ceph mgr module enable mymodule
```

Note that the MgrModule interface is not stable, so any modules maintained outside of the Ceph tree are liable to break when run against any newer or older versions of Ceph.

### LOGGING

`MgrModule` instances have a `log` property which is a logger instance that sends log messages into the Ceph logging layer where they will be recorded in the mgr daemon’s log file.

Use it the same way you would any other python logger. The python log levels debug, info, warn, err are mapped into the Ceph severities 20, 4, 1 and 0 respectively.

### EXPOSING COMMANDS

Set the `COMMANDS` class attribute of your module to a list of dicts like this:

```text
COMMANDS = [
    {
        "cmd": "foobar name=myarg,type=CephString",
        "desc": "Do something awesome",
        "perm": "rw",
        # optional:
        "poll": "true"
    }
]
```

The `cmd` part of each entry is parsed in the same way as internal Ceph mon and admin socket commands \(see mon/MonCommands.h in the Ceph source for examples\). Note that the “poll” field is optional, and is set to False by default; this indicates to the `ceph` CLI that it should call this command repeatedly and output results \(see `ceph -h` and its `--period` option\).

Each command is expected to return a tuple `(retval, stdout, stderr)`. `retval` is an integer representing a libc error code \(e.g. EINVAL, EPERM, or 0 for no error\), `stdout` is a string containing any non-error output, and `stderr` is a string containing any progress or error explanation output. Either or both of the two strings may be empty.

Implement the `handle_command` function to respond to the commands when they are sent:`MgrModule.handle_command`\(_inbuf_, _cmd_\)

Called by ceph-mgr to request the plugin to handle one of the commands that it declared in self.COMMANDS

Return a status code, an output buffer, and an output string. The output buffer is for data results, the output string is for informative text.Parameters

* **inbuf** \(_str_\) – content of any “-i &lt;file&gt;” supplied to ceph cli
* **cmd** \(_dict_\) – from Ceph’s cmdmap\_t

Returns

HandleCommandResult or a 3-tuple of \(int, str, str\)

### CONFIGURATION OPTIONS

Modules can load and store configuration options using the `set_module_option` and `get_module_option` methods.

Note 

Use `set_module_option` and `get_module_option` to manage user-visible configuration options that are not blobs \(like certificates\). If you want to persist module-internal data or binary configuration data consider using the [KV store](https://docs.ceph.com/docs/nautilus/mgr/modules/#kv-store).

You must declare your available configuration options in the `MODULE_OPTIONS` class attribute, like this:

```text
MODULE_OPTIONS = [
    {
        "name": "my_option"
    }
]
```

If you try to use set\_module\_option or get\_module\_option on options not declared in `MODULE_OPTIONS`, an exception will be raised.

You may choose to provide setter commands in your module to perform high level validation. Users can also modify configuration using the normal ceph config set command, where the configuration options for a mgr module are named like mgr/&lt;module name&gt;/&lt;option&gt;.

If a configuration option is different depending on which node the mgr is running on, then use _localized_ configuration \( `get_localized_module_option`, `set_localized_module_option`\). This may be necessary for options such as what address to listen on. Localized options may also be set externally with `ceph config set`, where they key name is like `mgr/<module name>/<mgr id>/<option>`

If you need to load and store data \(e.g. something larger, binary, or multiline\), use the KV store instead of configuration options \(see next section\).

Hints for using config options:

* Reads are fast: ceph-mgr keeps a local in-memory copy, so in many cases you can just do a get\_module\_option every time you use a option, rather than copying it out into a variable.
* Writes block until the value is persisted \(i.e. round trip to the monitor\), but reads from another thread will see the new value immediately.
* If a user has used config set from the command line, then the new value will become visible to get\_module\_option immediately, although the mon-&gt;mgr update is asynchronous, so config set will return a fraction of a second before the new value is visible on the mgr.
* To delete a config value \(i.e. revert to default\), just pass `None` to set\_module\_option.

`MgrModule.get_module_option`\(_key_, _default=None_\)

Retrieve the value of a persistent configuration settingParameters

* **key** \(_str_\) –
* **default** \(_str_\) –

Returns

str`MgrModule.set_module_option`\(_key_, _val_\)

Set the value of a persistent configuration settingParameters

**key** \(_str_\) –`MgrModule.get_localized_module_option`\(_key_, _default=None_\)

Retrieve localized configuration for this ceph-mgr instance :param str key: :param str default: :return: str`MgrModule.set_localized_module_option`\(_key_, _val_\)

Set localized configuration for this ceph-mgr instance :param str key: :param str val: :return: str

### KV STORE

Modules have access to a private \(per-module\) key value store, which is implemented using the monitor’s “config-key” commands. Use the `set_store` and `get_store` methods to access the KV store from your module.

The KV store commands work in a similar way to the configuration commands. Reads are fast, operating from a local cache. Writes block on persistence and do a round trip to the monitor.

This data can be access from outside of ceph-mgr using the `ceph config-key [get|set]` commands. Key names follow the same conventions as configuration options. Note that any values updated from outside of ceph-mgr will not be seen by running modules until the next restart. Users should be discouraged from accessing module KV data externally – if it is necessary for users to populate data, modules should provide special commands to set the data via the module.

Use the `get_store_prefix` function to enumerate keys within a particular prefix \(i.e. all keys starting with a particular substring\).`MgrModule.get_store`\(_key_, _default=None_\)

Get a value from this module’s persistent key value store`MgrModule.set_store`\(_key_, _val_\)

Set a value in this module’s persistent key value store. If val is None, remove key from storeParameters

* **key** \(_str_\) –
* **val** \(_str_\) –

`MgrModule.get_localized_store`\(_key_, _default=None_\)`MgrModule.set_localized_store`\(_key_, _val_\)`MgrModule.get_store_prefix`\(_key\_prefix_\)

Retrieve a dict of KV store keys to values, where the keys have the given prefixParameters

**key\_prefix** \(_str_\) –Returns

str

### ACCESSING CLUSTER DATA

Modules have access to the in-memory copies of the Ceph cluster’s state that the mgr maintains. Accessor functions as exposed as members of MgrModule.

Calls that access the cluster or daemon state are generally going from Python into native C++ routines. There is some overhead to this, but much less than for example calling into a REST API or calling into an SQL database.

There are no consistency rules about access to cluster structures or daemon metadata. For example, an OSD might exist in OSDMap but have no metadata, or vice versa. On a healthy cluster these will be very rare transient states, but modules should be written to cope with the possibility.

Note that these accessors must not be called in the modules `__init__` function. This will result in a circular locking exception.`MgrModule.get`\(_data\_name_\)

Called by the plugin to fetch named cluster-wide objects from ceph-mgr.Parameters

**data\_name** \(_str_\) – Valid things to fetch are osd\_crush\_map\_text, osd\_map, osd\_map\_tree, osd\_map\_crush, config, mon\_map, fs\_map, osd\_metadata, pg\_summary, io\_rate, pg\_dump, df, osd\_stats, health, mon\_status, devices, device &lt;devid&gt;, pg\_stats, pool\_stats, pg\_ready, osd\_ping\_times.Note:

All these structures have their own JSON representations: experiment or look at the C++ `dump()` methods to learn about them.`MgrModule.get_server`\(_hostname_\)

Called by the plugin to fetch metadata about a particular hostname from ceph-mgr.

This is information that ceph-mgr has gleaned from the daemon metadata reported by daemons running on a particular server.Parameters

**hostname** – a hostname`MgrModule.list_servers`\(\)

Like `get_server`, but gives information about all servers \(i.e. all unique hostnames that have been mentioned in daemon metadata\)Returns

a list of information about all serversReturn type

list`MgrModule.get_metadata`\(_svc\_type_, _svc\_id_\)

Fetch the daemon metadata for a particular service.

ceph-mgr fetches metadata asynchronously, so are windows of time during addition/removal of services where the metadata is not available to modules. `None` is returned if no metadata is available.Parameters

* **svc\_type** \(_str_\) – service type \(e.g., ‘mds’, ‘osd’, ‘mon’\)
* **svc\_id** \(_str_\) – service id. convert OSD integer IDs to strings when calling this

Return type

dict, or None if no metadata found`MgrModule.get_daemon_status`\(_svc\_type_, _svc\_id_\)

Fetch the latest status for a particular service daemon.

This method may return `None` if no status information is available, for example because the daemon hasn’t fully started yet.Parameters

* **svc\_type** – string \(e.g., ‘rgw’\)
* **svc\_id** – string

Returns

dict, or None if the service is not found`MgrModule.get_perf_schema`\(_svc\_type_, _svc\_name_\)

Called by the plugin to fetch perf counter schema info. svc\_name can be nullptr, as can svc\_type, in which case they are wildcardsParameters

* **svc\_type** \(_str_\) –
* **svc\_name** \(_str_\) –

Returns

list of dicts describing the counters requested`MgrModule.get_counter`\(_svc\_type_, _svc\_name_, _path_\)

Called by the plugin to fetch the latest performance counter data for a particular counter on a particular service.Parameters

* **svc\_type** \(_str_\) –
* **svc\_name** \(_str_\) –
* **path** \(_str_\) – a period-separated concatenation of the subsystem and the counter name, for example “mds.inodes”.

Returns

A list of two-tuples of \(timestamp, value\) is returned. This may be empty if no data is available.`MgrModule.get_mgr_id`\(\)

Retrieve the name of the manager daemon where this plugin is currently being executed \(i.e. the active manager\).Returns

str

### EXPOSING HEALTH CHECKS

Modules can raise first class Ceph health checks, which will be reported in the output of `ceph status` and in other places that report on the cluster’s health.

If you use `set_health_checks` to report a problem, be sure to call it again with an empty dict to clear your health check when the problem goes away.`MgrModule.set_health_checks`\(_checks_\)

Set the module’s current map of health checks. Argument is a dict of check names to info, in this form:

```text
{
  'CHECK_FOO': {
    'severity': 'warning',           # or 'error'
    'summary': 'summary string',
    'detail': [ 'list', 'of', 'detail', 'strings' ],
   },
  'CHECK_BAR': {
    'severity': 'error',
    'summary': 'bars are bad',
    'detail': [ 'too hard' ],
  },
}
```

Parameters

**list** – dict of health check dicts

### WHAT IF THE MONS ARE DOWN?

The manager daemon gets much of its state \(such as the cluster maps\) from the monitor. If the monitor cluster is inaccessible, whichever manager was active will continue to run, with the latest state it saw still in memory.

However, if you are creating a module that shows the cluster state to the user then you may well not want to mislead them by showing them that out of date state.

To check if the manager daemon currently has a connection to the monitor cluster, use this function:`MgrModule.have_mon_connection`\(\)

Check whether this ceph-mgr daemon has an open connection to a monitor. If it doesn’t, then it’s likely that the information we have about the cluster is out of date, and/or the monitor cluster is down.

### REPORTING IF YOUR MODULE CANNOT RUN

If your module cannot be run for any reason \(such as a missing dependency\), then you can report that by implementing the `can_run` function._static_ `MgrModule.can_run`\(\)

Implement this function to report whether the module’s dependencies are met. For example, if the module needs to import a particular dependency to work, then use a try/except around the import at file scope, and then report here if the import failed.

This will be called in a blocking way from the C++ code, so do not do any I/O that could block in this function.

:return a 2-tuple consisting of a boolean and explanatory string

Note that this will only work properly if your module can always be imported: if you are importing a dependency that may be absent, then do it in a try/except block so that your module can be loaded far enough to use `can_run` even if the dependency is absent.

### SENDING COMMANDS

A non-blocking facility is provided for sending monitor commands to the cluster.`MgrModule.send_command`\(_\*args_, _\*\*kwargs_\)

Called by the plugin to send a command to the mon cluster.Parameters

* **result** \(_CommandResult_\) – an instance of the `CommandResult` class, defined in the same module as MgrModule. This acts as a completion and stores the output of the command. Use `CommandResult.wait()` if you want to block on completion.
* **svc\_type** \(_str_\) –
* **svc\_id** \(_str_\) –
* **command** \(_str_\) – a JSON-serialized command. This uses the same format as the ceph command line, which is a dictionary of command arguments, with the extra `prefix` key containing the command name itself. Consult MonCommands.h for available commands and their expected arguments.
* **tag** \(_str_\) – used for nonblocking operation: when a command completes, the `notify()` callback on the MgrModule instance is triggered, with notify\_type set to “command”, and notify\_id set to the tag of the command.

### RECEIVING NOTIFICATIONS

The manager daemon calls the `notify` function on all active modules when certain important pieces of cluster state are updated, such as the cluster maps.

The actual data is not passed into this function, rather it is a cue for the module to go and read the relevant structure if it is interested. Most modules ignore most types of notification: to ignore a notification simply return from this function without doing anything.`MgrModule.notify`\(_notify\_type_, _notify\_id_\)

Called by the ceph-mgr service to notify the Python plugin that new state is available.Parameters

* **notify\_type** – string indicating what kind of notification, such as osd\_map, mon\_map, fs\_map, mon\_status, health, pg\_summary, command, service\_map
* **notify\_id** – string \(may be empty\) that optionally specifies which entity is being notified about. With “command” notifications this is set to the tag `from send_command`.

### ACCESSING RADOS OR CEPHFS

If you want to use the librados python API to access data stored in the Ceph cluster, you can access the `rados` attribute of your `MgrModule` instance. This is an instance of `rados.Rados` which has been constructed for you using the existing Ceph context \(an internal detail of the C++ Ceph code\) of the mgr daemon.

Always use this specially constructed librados instance instead of constructing one by hand.

Similarly, if you are using libcephfs to access the filesystem, then use the libcephfs `create_with_rados` to construct it from the `MgrModule.rados` librados instance, and thereby inherit the correct context.

Remember that your module may be running while other parts of the cluster are down: do not assume that librados or libcephfs calls will return promptly – consider whether to use timeouts or to block if the rest of the cluster is not fully available.

### IMPLEMENTING STANDBY MODE

For some modules, it is useful to run on standby manager daemons as well as on the active daemon. For example, an HTTP server can usefully serve HTTP redirect responses from the standby managers so that the user can point his browser at any of the manager daemons without having to worry about which one is active.

Standby manager daemons look for a subclass of `StandbyModule` in each module. If the class is not found then the module is not used at all on standby daemons. If the class is found, then its `serve` method is called. Implementations of `StandbyModule` must inherit from `mgr_module.MgrStandbyModule`.

The interface of `MgrStandbyModule` is much restricted compared to `MgrModule` – none of the Ceph cluster state is available to the module. `serve` and `shutdown` methods are used in the same way as a normal module class. The `get_active_uri` method enables the standby module to discover the address of its active peer in order to make redirects. See the `MgrStandbyModule` definition in the Ceph source code for the full list of methods.

For an example of how to use this interface, look at the source code of the `dashboard` module.

### COMMUNICATING BETWEEN MODULES

Modules can invoke member functions of other modules.`MgrModule.remote`\(_module\_name_, _method\_name_, _\*args_, _\*\*kwargs_\)

Invoke a method on another module. All arguments, and the return value from the other module must be serializable.

Limitation: Do not import any modules within the called method. Otherwise you will get an error in Python 2:

```text
RuntimeError('cannot unmarshal code objects in restricted execution mode',)
```

Parameters

* **module\_name** – Name of other module. If module isn’t loaded, an ImportError exception is raised.
* **method\_name** – Method name. If it does not exist, a NameError exception is raised.
* **args** – Argument tuple
* **kwargs** – Keyword argument dict

Raises

* **RuntimeError** – **Any** error raised within the method is converted to a RuntimeError
* **ImportError** – No such module

Be sure to handle `ImportError` to deal with the case that the desired module is not enabled.

If the remote method raises a python exception, this will be converted to a RuntimeError on the calling side, where the message string describes the exception that was originally thrown. If your logic intends to handle certain errors cleanly, it is better to modify the remote method to return an error value instead of raising an exception.

At time of writing, inter-module calls are implemented without copies or serialization, so when you return a python object, you’re returning a reference to that object to the calling module. It is recommend _not_ to rely on this reference passing, as in future the implementation may change to serialize arguments and return values.

### LOGGING

Use your module’s `log` attribute as your logger. This is a logger configured to output via the ceph logging framework, to the local ceph-mgr log files.

Python log severities are mapped to ceph severities as follows:

* DEBUG is 20
* INFO is 4
* WARN is 1
* ERR is 0

### SHUTTING DOWN CLEANLY

If a module implements the `serve()` method, it should also implement the `shutdown()` method to shutdown cleanly: misbehaving modules may otherwise prevent clean shutdown of ceph-mgr.

### LIMITATIONS

It is not possible to call back into C++ code from a module’s `__init__()` method. For example calling `self.get_module_option()` at this point will result in an assertion failure in ceph-mgr. For modules that implement the `serve()` method, it usually makes sense to do most initialization inside that method instead.

### IS SOMETHING MISSING?

The ceph-mgr python interface is not set in stone. If you have a need that is not satisfied by the current interface, please bring it up on the ceph-devel mailing list. While it is desired to avoid bloating the interface, it is not generally very hard to expose existing data to the Python code when there is a good reason.

