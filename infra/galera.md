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

# MariaDB Galera config for HAProxy / Keepalived
* Add zabbix users to the db
```bash
mysql -uroot -p
```

```bash
CREATE USER 'zabbix'@'192.168.10.201' IDENTIFIED BY 'crosemont';
CREATE USER 'zabbix'@'192.168.10.202' IDENTIFIED BY 'crosemont';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'192.168.10.201';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'192.168.10.202';
FLUSH PRIVILEGES;
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
EXIT;
```

* Add haproxy host to add haproxy user
```bash
mysql -uroot -p
```

```bash
CREATE USER 'haproxy'@'%';
EXIT;
```

### After each full cluster shutdown, refresh the cluster
* Make sure mariadb service is stopped on all nodes
```bash
systemctl stop mariadb
systemctl status mariadb
```

* Check which node is safe to bootstrap (run on each node, proceed only on the node that returns 1)
```bash
grep safe_to_bootstrap /var/lib/mysql/grastate.dat
```

* Bootstrap the cluster
```bash
galera_new_cluster
```

* Check if the **wsrep_cluster_size** is equal to 1
```bash
mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size';"
```

* If **wsrep_cluster_size** is equal to 1, start mariadb on the **2 other nodes**
```bash
systemctl start mariadb
```

* Re-run to check if the **wsrep_cluster_size** is now equal to 3
```bash
mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size';"
```

