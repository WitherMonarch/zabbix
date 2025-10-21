# Setup Keepalived and its VIP
* Install keepalived
dnf install -y keepalived

* Backup the old keepalived and set the new keepalived.conf 
mv /etc/keepalived/keepalived.conf /etc/keepalived/keepalived.bak
nano /etc/keepalived/keepalived.conf

* Start keepalived
systemctl enable --now keepalived
