# Setup HAProxy with Keepalived's VIP
* Download haproxy
```bash
dnf install -y haproxy
```

* Add the maria nodes to /etc/hosts
```bash
192.168.10.203 maria1.rosemont.com maria1
192.168.10.204 maria2.rosemont.com maria2
192.168.10.205 maria3.rosemont.com maria3
```

* Backup haproxy.cfg and paste the new contents of haproxy.cfg
```bash
mv /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.bak
wget -O /etc/haproxy/haproxy.cfg https://raw.githubusercontent.com/WitherMonarch/zabbix/refs/heads/main/infra/haproxy.cfg
```


* Edit the override of haproxy.service to only be up when keepalived and its VIP are up
```bash
systemctl edit haproxy
```
	[Unit]
	After=keepalived.service
	Requires=keepalived.service

	[Service]
	ExecStartPre=/usr/bin/bash -c 'until ip addr show ens18 | grep -q 192.168.10.210; do sleep 1; done'

* Edit the SELinux and firewalld policies to let HAProxy pass
```bash
setsebool -P haproxy_connect_any 1
setsebool -P httpd_can_network_connect 1
setsebool -P zabbix_can_network 1
firewall-cmd --permanent --add-service=mysql
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=zabbix-server
firewall-cmd --permanent --add-service=zabbix-agent (if the agent is configured on this machine)
firewall-cmd --reload
```

* Enable and start haproxy **only when keepalived is configured and working**
```bash
systemctl enable --now haproxy
```

* Test the connection and check the hostname of the machine
```bash
mysql -u zabbix -p -h 192.168.10.210 zabbix
```
