# Install from package source
* Follow the steps here and come back before creating the database: https://www.zabbix.com/download?zabbix=7.4&os_distribution=alma_linux&os_version=9&components=server_frontend_agent&db=mysql&ws=apache
* Before creating initial database, run `dnf search mysql` and install **mysql-server** depending on the versions available
* Enable the mysqld module to start mysql
* To enter the db, enter `mysql -uroot` (no -p since the account has no password)
* **IMPORTANT, DO THIS STEP BEFORE CREATING THE INITIAL DATABSE!!!!!!!!!!!!** Set a password to the root account in the database to login with *mysql -uroot -p* later:
   `ALTER USER 'root'@'localhost' IDENTIFIED BY 'password';`
   `FLUSH PRIVILEGES;`
   * Go back to the zabbix documentation until the end and come back here
* After enabling the zabbix-server, zabbix-agent, http and php-fpm services, change the firewall settings to allow http(s):
  * `sudo firewall-cmd --permanent --add-service=http`
  * `sudo firewall-cmd --permanent --add-service=https` (optional)
* Make sure to reload the firewall-cmd
  * `sudo firewall-cmd --reload` 
* Access the webpage from http://ip:80/zabbix
  * Default credentials: **Admin**, **zabbix**