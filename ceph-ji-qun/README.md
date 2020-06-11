# CEPH集群

## CEPH STORAGE CLUSTER

The [Ceph Storage Cluster](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-storage-cluster) is the foundation for all Ceph deployments. Based upon RADOS, Ceph Storage Clusters consist of two types of daemons: a [Ceph OSD Daemon](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-osd-daemon) \(OSD\) stores data as objects on a storage node; and a [Ceph Monitor](https://docs.ceph.com/docs/nautilus/glossary/#term-ceph-monitor) \(MON\) maintains a master copy of the cluster map. A Ceph Storage Cluster may contain thousands of storage nodes. A minimal system will have at least one Ceph Monitor and two Ceph OSD Daemons for data replication.

The Ceph Filesystem, Ceph Object Storage and Ceph Block Devices read data from and write data to the Ceph Storage Cluster.

