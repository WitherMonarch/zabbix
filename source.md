# Compile from source (AlmaLinux 9)
wget https://cdn.zabbix.com/zabbix/sources/stable/7.4/zabbix-7.4.3.tar.gz

tar -xvzf zabbix-7.4.3.tar.gz
rm zabbix-7.4.3.tar.gz
cd zabbix-7.4.3

sudo groupadd --system zabbix
sudo useradd --system -g zabbix -d /usr/lib/zabbix -s /sbin/nologin -c "Zabbix Monitoring System" zabbix

sudo mkdir -m u=rwx,g=rwx,o= -p /usr/lib/zabbix
sudo chown zabbix:zabbix /usr/lib/zabbix

sudo dnf groupinstall "Development Tools" -y
sudo dnf install -y mariadb mariadb-server
sudo systemctl enable --now mariadb
sudo mysql_secure_installation
sudo mysql -uroot

alter user 'root'@'localhost' identified by 'crosemont';
flush privileges;

mysql -uroot -p

CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'crosemont';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
FLUSH PRIVILEGES;
EXIT;

cd database/mysql

mysql -u zabbix -p zabbix < schema.sql
mysql -u zabbix -p zabbix < images.sql
mysql -u zabbix -p zabbix < data.sql

# Enable Additional Repositories (compiling dependencies)
sudo dnf install epel-release -y

sudo dnf install libxml2-devel mariadb-connector-c-devel net-snmp-devel libssh2-devel libevent-devel libcurl-devel pcre2-devel -y

export CFLAGS="-std=gnu99"

cd ../..

./configure   --enable-server   --enable-agent   --with-mysql   --with-libcurl   --with-libxml2   --with-net-snmp   --with-libpcre2   --with-ssh2 --disable-perfstat

sudo make install


# Install web server
sudo dnf install httpd php php-mysqlnd php-xml php-bcmath php-gd php-mbstring php-json php-ctype -y

sudo systemctl enable --now httpd

sudo mkdir -p /var/www/html/zabbix
sudo cp -a . /var/www/html/zabbix

sudo chown -R apache:apache /var/www/html/zabbix
sudo chmod -R 755 /var/www/html/zabbix

sudo setsebool -P httpd_can_network_connect 1

sudo chcon -R -t httpd_sys_content_t /var/www/html/zabbix
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload

sudo nano /etc/php.ini
post_max_size = 16M
max_execution_time = 300
max_input_time = 300

sudo systemctl restart httpd php-fm
sudo chcon -R -t httpd_sys_rw_content_t /var/www/html/zabbix/conf



# UNTESTED SECTION

Binaries -> /usr/lib/zabbix/sbin
Configs -> /etc/zabbix
Logs -> /var/log/zabbix
Runtime/PID -> /var/run/zabbix
Home (optional) -> /usr/lib/zabbix

./configure \
    --prefix=/usr/lib/zabbix \
    --sbindir=/usr/lib/zabbix/sbin \
    --sysconfdir=/etc/zabbix \
    --localstatedir=/var/lib/zabbix \
    --runstatedir=/var/run/zabbix \
    --logdir=/var/log/zabbix \
    --libdir=/usr/lib/zabbix/lib

# Create system group/user if not exists
sudo groupadd --system zabbix
sudo useradd --system -g zabbix -d /usr/lib/zabbix -s /sbin/nologin -c "Zabbix Monitoring System" zabbix

# Create runtime/log directories
sudo mkdir -p /var/lib/zabbix /var/run/zabbix /var/log/zabbix
sudo chown -R zabbix:zabbix /var/lib/zabbix /var/run/zabbix /var/log/zabbix

# Optional home directory for MySQL credentials
sudo mkdir -m 770 -p /usr/lib/zabbix
sudo chown zabbix:zabbix /usr/lib/zabbix

sudo -u zabbix /usr/lib/zabbix/sbin/zabbix_server -c /etc/zabbix/zabbix_server.conf
sudo -u zabbix /usr/lib/zabbix/sbin/zabbix_agentd -c /etc/zabbix/zabbix_agentd.conf

# Recommended changes
sudo cp -a frontends/php/* /var/www/html/zabbix

https://techexpert.tips/zabbix/
