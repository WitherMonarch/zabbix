# Configure the zabbix servers
* Add the repository
```bash
rpm -Uvh https://repo.zabbix.com/zabbix/7.4/release/alma/9/noarch/zabbix-release-latest-7.4.el9.noarch.rpm
dnf clean all
```

```bash
dnf install -y zabbix-server-mysql zabbix-web-mysql zabbix-apache-conf zabbix-sql-scripts zabbix-selinux-policy zabbix-agent mariadb
```

* Edit the zabbix_server.conf to add the following
```bash
nano /etc/zabbix/zabbix_server.conf
```
    DBHost=192.168.10.210
    DBName=zabbix
    DBUser=zabbix
    DBPassword=crosemont
    HANodeName=zabbix[#]
    NodeAddress=[nodeIP]

* Edit the zabbix_agentd.conf to add the following (optional)
```bash
nano /etc/zabbix/zabbix_agentd.conf
```
    Server=192.168.10.201,192.168.10.202
    Hostname=SameAsWebUIHostname

* If not done already, set SELinux and firewall policies
```bash
setsebool -P haproxy_connect_any 1
setsebool -P httpd_can_network_connect 1
setsebool -P zabbix_can_network 1
firewall-cmd --permanent --add-service=mysql
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=zabbix-server
firewall-cmd --permanent --add-service=zabbix-agent # (if the agent is configured on this machine)
firewall-cmd --reload
```

* Start the services for the zabbix server
```bash
systemctl enable --now httpd php-fpm zabbix-server
systemctl enable --now zabbix-agent # optional
```

### Useful command to check SELinux policies

    getsebool -a | grep
