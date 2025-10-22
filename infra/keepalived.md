# Setup Keepalived and its VIP
* Install keepalived
```bash
dnf install -y keepalived
```

* Backup the old keepalived and set the new keepalived.conf 
```bash
mv /etc/keepalived/keepalived.conf /etc/keepalived/keepalived.bak
```
```bash
wget -O /etc/keepalived/keepalived.conf https://raw.githubusercontent.com/WitherMonarch/zabbix/refs/heads/main/infra/keepalived.conf
```

* Edit the priority in the keepalived.conf file
```bash
nano /etc/keepalived/keepalived.conf
```

* Start keepalived
```bash
systemctl enable --now keepalived
```
