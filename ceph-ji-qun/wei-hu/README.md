# 维护

## CLUSTER OPERATIONS

<table>
  <thead>
    <tr>
      <th style="text-align:left">
        <p>High-level cluster operations consist primarily of starting, stopping,
          and restarting a cluster with the <code>ceph</code> service; checking the
          cluster&#x2019;s health; and, monitoring an operating cluster.</p>
        <ul>
          <li><a href="https://docs.ceph.com/docs/nautilus/rados/operations/operating/">Operating a Cluster</a>
          </li>
          <li><a href="https://docs.ceph.com/docs/nautilus/rados/operations/health-checks/">Health checks</a>
          </li>
          <li><a href="https://docs.ceph.com/docs/nautilus/rados/operations/monitoring/">Monitoring a Cluster</a>
          </li>
          <li><a href="https://docs.ceph.com/docs/nautilus/rados/operations/monitoring-osd-pg/">Monitoring OSDs and PGs</a>
          </li>
          <li><a href="https://docs.ceph.com/docs/nautilus/rados/operations/user-management/">User Management</a>
          </li>
          <li><a href="https://docs.ceph.com/docs/nautilus/rados/operations/pg-repair/">Repairing PG inconsistencies</a>
          </li>
        </ul>
      </th>
      <th style="text-align:left">
        <p>Once you have your cluster up and running, you may begin working with
          data placement. Ceph supports petabyte-scale data storage clusters, with
          storage pools and placement groups that distribute data across the cluster
          using Ceph&#x2019;s CRUSH algorithm.</p>
        <ul>
          <li><a href="https://docs.ceph.com/docs/nautilus/rados/operations/data-placement/">Data Placement Overview</a>
          </li>
          <li><a href="https://docs.ceph.com/docs/nautilus/rados/operations/pools/">Pools</a>
          </li>
          <li><a href="https://docs.ceph.com/docs/nautilus/rados/operations/erasure-code/">Erasure code</a>
          </li>
          <li><a href="https://docs.ceph.com/docs/nautilus/rados/operations/cache-tiering/">Cache Tiering</a>
          </li>
          <li><a href="https://docs.ceph.com/docs/nautilus/rados/operations/placement-groups/">Placement Groups</a>
          </li>
          <li><a href="https://docs.ceph.com/docs/nautilus/rados/operations/balancer/">Balancer</a>
          </li>
          <li><a href="https://docs.ceph.com/docs/nautilus/rados/operations/upmap/">Using the pg-upmap</a>
          </li>
          <li><a href="https://docs.ceph.com/docs/nautilus/rados/operations/crush-map/">CRUSH Maps</a>
          </li>
          <li><a href="https://docs.ceph.com/docs/nautilus/rados/operations/crush-map-edits/">Manually editing a CRUSH Map</a>
          </li>
        </ul>
      </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">
        <p>Low-level cluster operations consist of starting, stopping, and restarting
          a particular daemon within a cluster; changing the settings of a particular
          daemon or subsystem; and, adding a daemon to the cluster or removing a
          daemon from the cluster. The most common use cases for low-level operations
          include growing or shrinking the Ceph cluster and replacing legacy or failed
          hardware with new hardware.</p>
        <ul>
          <li><a href="https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-osds/">Adding/Removing OSDs</a>
          </li>
          <li><a href="https://docs.ceph.com/docs/nautilus/rados/operations/add-or-rm-mons/">Adding/Removing Monitors</a>
          </li>
          <li><a href="https://docs.ceph.com/docs/nautilus/rados/operations/devices/">Device Management</a>
          </li>
          <li><a href="https://docs.ceph.com/docs/nautilus/rados/operations/bluestore-migration/">BlueStore Migration</a>
          </li>
          <li><a href="https://docs.ceph.com/docs/nautilus/rados/operations/control/">Command Reference</a>
          </li>
        </ul>
      </td>
      <td style="text-align:left">
        <p>Ceph is still on the leading edge, so you may encounter situations that
          require you to evaluate your Ceph configuration and modify your logging
          and debugging settings to identify and remedy issues you are encountering
          with your cluster.</p>
        <ul>
          <li><a href="https://docs.ceph.com/docs/nautilus/rados/troubleshooting/community/">The Ceph Community</a>
          </li>
          <li><a href="https://docs.ceph.com/docs/nautilus/rados/troubleshooting/troubleshooting-mon/">Troubleshooting Monitors</a>
          </li>
          <li><a href="https://docs.ceph.com/docs/nautilus/rados/troubleshooting/troubleshooting-osd/">Troubleshooting OSDs</a>
          </li>
          <li><a href="https://docs.ceph.com/docs/nautilus/rados/troubleshooting/troubleshooting-pg/">Troubleshooting PGs</a>
          </li>
          <li><a href="https://docs.ceph.com/docs/nautilus/rados/troubleshooting/log-and-debug/">Logging and Debugging</a>
          </li>
          <li><a href="https://docs.ceph.com/docs/nautilus/rados/troubleshooting/cpu-profiling/">CPU Profiling</a>
          </li>
          <li><a href="https://docs.ceph.com/docs/nautilus/rados/troubleshooting/memory-profiling/">Memory Profiling</a>
          </li>
        </ul>
      </td>
    </tr>
  </tbody>
</table>

