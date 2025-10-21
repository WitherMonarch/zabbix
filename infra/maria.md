# MariaDB Galera config for HA-KA
* Add zabbix users to the db

mysql -uroot -p

CREATE USER 'zabbix'@'192.168.10.201' IDENTIFIED BY 'crosemont';
CREATE USER 'zabbix'@'192.168.10.202' IDENTIFIED BY 'crosemont';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'192.168.10.201';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'192.168.10.202';
FLUSH PRIVILEGES;
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
EXIT;

* Add haproxy host to add haproxy user
mysql -uroot -p

CREATE USER 'haproxy'@'%';
EXIT;
