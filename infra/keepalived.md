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
nano /etc/keepalived/keepalived.conf
```

* Start keepalived
```bash
systemctl enable --now keepalived
```
