# Setup HAProxy with Keepalived's VIP
* Download haproxy
dnf install -y haproxy

* Add the maria nodes to /etc/hosts
192.168.10.203 maria1.rosemont.com maria1
192.168.10.204 maria2.rosemont.com maria2
192.168.10.205 maria3.rosemont.com maria3

* Backup haproxy.cfg and paste the new contents of haproxy.cfg
mv /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.bak
nano /etc/haproxy/haproxy.cfg

* Edit the override of haproxy.service to only be up when keepalived and its VIP are up
systemctl edit haproxy

	### Anything between here and the comment below will become the new contents of the file
	
	[Unit]
	After=keepalived.service
	Requires=keepalived.service

	[Service]
	ExecStartPre=/usr/bin/bash -c 'until ip addr show ens18 | grep -q 192.168.10.210; do sleep 1; done'

	### Lines below this comment will be discarded

* Edit the SELinux and firewalld policies to let HAProxy pass
setsebool -P haproxy_connect_any 1
firewall-cmd --permanent --add-port=3306/tcp
firewall-cmd --reload

* Enable and start haproxy **only when keepalived is configured and working**
systemctl enable --now haproxy

* Test the connection and check the hostname of the machine
mysql -u zabbix -p -h 192.168.10.210 zabbix
