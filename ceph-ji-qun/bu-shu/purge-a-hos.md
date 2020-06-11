# PURGE A HOS

## PURGE A HOST[¶](https://docs.ceph.com/docs/nautilus/rados/deployment/ceph-deploy-purge/#purge-a-host)

When you remove Ceph daemons and uninstall Ceph, there may still be extraneous data from the cluster on your server. The `purge` and `purgedata` commands provide a convenient means of cleaning up a host.

### PURGE DATA

To remove all data from `/var/lib/ceph` \(but leave Ceph packages intact\), execute the `purgedata` command.

> ceph-deploy purgedata {hostname} \[{hostname} …\]

### PURGE

To remove all data from `/var/lib/ceph` and uninstall Ceph packages, execute the `purge` command.

> ceph-deploy purge {hostname} \[{hostname} …\]

