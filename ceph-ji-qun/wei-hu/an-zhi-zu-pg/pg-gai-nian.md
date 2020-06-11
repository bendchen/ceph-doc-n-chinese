# PG概念

## PLACEMENT GROUP CONCEPTS

When you execute commands like `ceph -w`, `ceph osd dump`, and other commands related to placement groups, Ceph may return values using some of the following terms:_Peering_

The process of bringing all of the OSDs that store a Placement Group \(PG\) into agreement about the state of all of the objects \(and their metadata\) in that PG. Note that agreeing on the state does not mean that they all have the latest contents._Acting Set_

The ordered list of OSDs who are \(or were as of some epoch\) responsible for a particular placement group._Up Set_

The ordered list of OSDs responsible for a particular placement group for a particular epoch according to CRUSH. Normally this is the same as the _Acting Set_, except when the _Acting Set_ has been explicitly overridden via `pg_temp` in the OSD Map._Current Interval_ or _Past Interval_

A sequence of OSD map epochs during which the _Acting Set_ and _Up Set_ for particular placement group do not change._Primary_

The member \(and by convention first\) of the _Acting Set_, that is responsible for coordination peering, and is the only OSD that will accept client-initiated writes to objects in a placement group._Replica_

A non-primary OSD in the _Acting Set_ for a placement group \(and who has been recognized as such and _activated_ by the primary\)._Stray_

An OSD that is not a member of the current _Acting Set_, but has not yet been told that it can delete its copies of a particular placement group._Recovery_

Ensuring that copies of all of the objects in a placement group are on all of the OSDs in the _Acting Set_. Once _Peering_ has been performed, the _Primary_ can start accepting write operations, and _Recovery_ can proceed in the background._PG Info_

Basic metadata about the placement group’s creation epoch, the version for the most recent write to the placement group, _last epoch started_, _last epoch clean_, and the beginning of the _current interval_. Any inter-OSD communication about placement groups includes the _PG Info_, such that any OSD that knows a placement group exists \(or once existed\) also has a lower bound on _last epoch clean_ or _last epoch started_._PG Log_

A list of recent updates made to objects in a placement group. Note that these logs can be truncated after all OSDs in the _Acting Set_ have acknowledged up to a certain point._Missing Set_

Each OSD notes update log entries and if they imply updates to the contents of an object, adds that object to a list of needed updates. This list is called the _Missing Set_ for that `<OSD,PG>`._Authoritative History_

A complete, and fully ordered set of operations that, if performed, would bring an OSD’s copy of a placement group up to date._Epoch_

A \(monotonically increasing\) OSD map version number_Last Epoch Start_

The last epoch at which all nodes in the _Acting Set_ for a particular placement group agreed on an _Authoritative History_. At this point, _Peering_ is deemed to have been successful._up\_thru_

Before a _Primary_ can successfully complete the _Peering_ process, it must inform a monitor that is alive through the current OSD map _Epoch_ by having the monitor set its _up\_thru_ in the osd map. This helps _Peering_ ignore previous _Acting Sets_ for which _Peering_ never completed after certain sequences of failures, such as the second interval below:

* _acting set_ = \[A,B\]
* _acting set_ = \[A\]
* _acting set_ = \[\] very shortly after \(e.g., simultaneous failure, but staggered detection\)
* _acting set_ = \[B\] \(B restarts, A does not\)

_Last Epoch Clean_

The last _Epoch_ at which all nodes in the _Acting set_ for a particular placement group were completely up to date \(both placement group logs and object contents\). At this point, _recovery_ is deemed to have been completed.

