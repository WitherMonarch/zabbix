# =========================================================
# Zabbix 7.4.3 Source Installation on AlmaLinux 9
# =========================================================

# -------------------------------
# Set base directory
# -------------------------------
# All compilation and source steps will be done here
cd /usr/local/src || exit

# -------------------------------
# Create Zabbix user and group
# -------------------------------
sudo groupadd --system zabbix
sudo useradd --system -g zabbix -d /usr/lib/zabbix -s /sbin/nologin -c "Zabbix Monitoring System" zabbix
sudo usermod -aG wheel zabbix
sudo passwd zabbix

sudo mkdir -m u=rwx,g=rwx,o= -p /usr/lib/zabbix
sudo chown zabbix:zabbix /usr/lib/zabbix

# -------------------------------
# Download and extract Zabbix source
# -------------------------------
sudo wget https://cdn.zabbix.com/zabbix/sources/stable/7.4/zabbix-7.4.3.tar.gz
sudo tar -xvzf zabbix-7.4.3.tar.gz
sudo rm zabbix-7.4.3.tar.gz
cd zabbix-7.4.3

# -------------------------------
# Install build dependencies
# -------------------------------
sudo dnf groupinstall "Development Tools" -y && sudo dnf install -y mariadb mariadb-server epel-release && sudo dnf install -y libxml2-devel mariadb-connector-c-devel net-snmp-devel \
    libssh2-devel libevent-devel libcurl-devel pcre2-devel openssl-devel \
    php php-mysqlnd php-xml php-bcmath php-gd php-mbstring php-json php-ctype \
    httpd mod_ssl

sudo systemctl enable --now mariadb httpd

# -------------------------------
# Secure MariaDB and create database
# -------------------------------
sudo mysql_secure_installation

# Connect as root
sudo mysql -uroot

# Inside MariaDB shell:
# ---------------------
alter user 'root'@'localhost' identified by 'crosemont';
flush privileges;

CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'crosemont';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
FLUSH PRIVILEGES;
EXIT;
# ---------------------

# Import schema and data
cd database/mysql
mysql -u zabbix -p zabbix < schema.sql
mysql -u zabbix -p zabbix < images.sql
mysql -u zabbix -p zabbix < data.sql
cd ../..

# -------------------------------
# Compile and install Zabbix
# -------------------------------
sudo chown -R zabbix:zabbix /usr/local/src/zabbix-7.4.3
sudo -u zabbix -s
export CFLAGS="-std=gnu99"
cd /usr/local/src/zabbix-7.4.3

./configure --enable-server --enable-agent --with-mysql \
    --with-libcurl --with-libxml2 --with-net-snmp \
    --with-libpcre2 --with-ssh2 --with-openssl

sudo make install
exit
sudo chown -R rosemont:rosemont /usr/local/src/zabbix-7.4.3

# Lock zabbix user and remove from wheel
sudo passwd -d zabbix
sudo gpasswd -d zabbix wheel

# -------------------------------
# Configure PHP
# -------------------------------
sudo sed -i 's/post_max_size = .*/post_max_size = 16M/' /etc/php.ini
sudo sed -i 's/max_execution_time = .*/max_execution_time = 300/' /etc/php.ini
sudo sed -i 's/max_input_time = .*/max_input_time = 300/' /etc/php.ini
sudo systemctl restart php-fpm

# -------------------------------
# Install Zabbix frontend
# -------------------------------
sudo mkdir -p /var/www/html/zabbix
sudo cp -a /usr/local/src/zabbix-7.4.3/ui/. /var/www/html/zabbix

sudo chown -R apache:apache /var/www/html/zabbix
sudo chmod -R 755 /var/www/html/zabbix
sudo setsebool -P httpd_can_network_connect 1
sudo chcon -R -t httpd_sys_content_t /var/www/html/zabbix
sudo chcon -R -t httpd_sys_rw_content_t /var/www/html/zabbix/conf

# -------------------------------
# Generate self-signed certificate in /etc/ssl/zabbix > Put server ip as CN
# -------------------------------
sudo mkdir -p /etc/ssl/zabbix
sudo openssl req -x509 -nodes -days 365 \
  -newkey rsa:2048 \
  -keyout /etc/ssl/zabbix/selfsigned.key \
  -out /etc/ssl/zabbix/selfsigned.crt 

# Temporary ownership so Zabbix frontend wizard can access/write
sudo chown -R root:apache /etc/ssl/zabbix
sudo chmod 640 /etc/ssl/zabbix/selfsigned.key
sudo chmod 644 /etc/ssl/zabbix/selfsigned.crt
sudo chmod 750 /etc/ssl/zabbix
sudo chcon -R -t httpd_sys_rw_content_t /etc/ssl/zabbix
sudo usermod -aG apache zabbix # Add zabbix to apache group so it can read certs

# -------------------------------
# Configure Apache HTTPS VirtualHost
# -------------------------------
sudo tee /etc/httpd/conf.d/zabbix-ssl.conf > /dev/null <<'EOF'
<VirtualHost *:443>
    ServerAdmin admin@localhost
    DocumentRoot /var/www/html/zabbix
    ServerName localhost

    SSLEngine on
    SSLCertificateFile /etc/ssl/zabbix/selfsigned.crt
    SSLCertificateKeyFile /etc/ssl/zabbix/selfsigned.key

    <Directory "/var/www/html/zabbix">
        Options FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog /var/log/httpd/zabbix-ssl-error.log
    CustomLog /var/log/httpd/zabbix-ssl-access.log combined
</VirtualHost>
EOF

# -------------------------------
# Firewall and Apache restart
# -------------------------------
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
sudo systemctl restart httpd

# -------------------------------
# Zabbix frontend TLS settings (wizard)
# -------------------------------
# Open browser: https://<server-ip>/zabbix
# Use the following TLS settings:
# TLS CA file: /etc/ssl/zabbix/selfsigned.crt
# TLS key file: /etc/ssl/zabbix/selfsigned.key
# TLS certificate file: /etc/ssl/zabbix/selfsigned.crt
# Verify server certificate issuer and subject: unchecked

# -------------------------------
# Create systemd service files
# -------------------------------
# Zabbix Server
sudo tee /etc/systemd/system/zabbix-server.service > /dev/null <<'EOF'
[Unit]
Description=Zabbix Server
After=network.target mariadb.service
Requires=mariadb.service

[Service]
Type=simple
User=zabbix
Group=zabbix
ExecStart=/usr/local/sbin/zabbix_server -c /usr/local/etc/zabbix_server.conf -f
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

# Zabbix Agent (optional)
sudo tee /etc/systemd/system/zabbix-agent.service > /dev/null <<'EOF'
[Unit]
Description=Zabbix Agent
After=network.target

[Service]
Type=simple
User=zabbix
Group=zabbix
ExecStart=/usr/local/sbin/zabbix_agentd -c /usr/local/etc/zabbix_agentd.conf -f
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

# Edit zabbix_server.conf file (zabbix_agentd.conf optional)
Include:
* DBHost=localhost
* DBName=zabbix
* DBUser=zabbix
* DBPassword=crosemont
* TLSCAFile=/etc/ssl/zabbix/selfsigned.crt
* TLSCertFile=/etc/ssl/zabbix/selfsigned.crt
* TLSKeyFile=/etc/ssl/zabbix/selfsigned.key
* TLSFrontendAccept=unencrypted,cert

# Edit zabbix_agentd.conf
Hostname=SameAsZabbixHostnameOnDashboard
Server=127.0.0.1

# Enable and start zabbix-server
sudo systemctl daemon-reload
sudo systemctl enable --now zabbix-server (zabbix-agent)
