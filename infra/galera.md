# MariaDB Galera Cluster Setup on AlmaLinux 9

Follow the instructions on this page first:
[ServerWorld AlmaLinux 9 MariaDB Galera Guide](https://www.server-world.info/en/note?os=AlmaLinux_9&p=mariadb&f=5)

## Important Additions and Notes

### 1. Add Node Name

In `/etc/my.cnf.d/galera.cnf`, for each node, add:

```
wsrep_node_name=maria[#]
```

Replace `[#]` with the node number (e.g., `maria1`, `maria2`, `maria3`).

### 2. Set SST Authentication

Also in `/etc/my.cnf.d/galera.cnf`, add:

```
wsrep_sst_auth=root:crosemont
```

This sets the root password for SST transfers between nodes.

### 3. Root User Setup After mysql_secure_installation

After running `mysql_secure_installation` on the first node, log in and alter the root user for localhost to ensure proper access:

```bash
mysql -u root -p
ALTER USER 'root'@'localhost' IDENTIFIED BY 'crosemont';
FLUSH PRIVILEGES;
EXIT;
```

This step ensures the Galera SST authentication works correctly.

### Summary

* Follow the ServerWorld guide for general setup.
* Add `wsrep_node_name` and `wsrep_sst_auth` in `galera.cnf`.
* Do not forget to set the root password after `mysql_secure_installation` on the first node.
* Then run systemctl enable --now mariadb on node1 before node2 and 3

# After each full cluster shutdown, refresh the cluster
* Make sure mariadb service is stopped on all nodes
systemctl stop mariadb
systemctl status mariadb

* Check which node is safe to bootstrap (run on each node, proceed only on the node that returns 1)
grep safe_to_bootstrap /var/lib/mysql/grastate.dat

* Bootstrap the cluster
galera_new_cluster

* Check if the **wsrep_cluster_size** is equal to 1
mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size';"

* If **wsrep_cluster_size** is equal to 1, start mariadb on the other 2 nodes
systemctl start mariadb

* Re-run to check if the **wsrep_cluster_size** is now equal to 3
mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size';"

