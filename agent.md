# Zabbix agent setup (AlmaLinux 9)
Install the zabbix agent on the machine
edit the zabbix_agentd.conf (sometimes zabbix-agent.conf)
Set the ip of the Server field (zabbix server ip, not the ServerActive field unless active check)
Set the Hostname field to whatever hostname (has to be the same in the zabbix-server host value)
Allow port 10050 through the firewall:
```bash
sudo firewall-cmd --add-port=10050/tcp --permanent
```
```bash
sudo firewall-cmd --reload
```
Start (and enable) the zabbix-agent service
