# Install zabbix from docker (AlmaLinux 9)
* Remove older docker packages and native alma container packages (conflict with docker)
  * `sudo dnf remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine podman runc`
* Setup the repository
  * `sudo dnf -y install dnf-plugins-core`
  * `sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo`
* Install the docker engine
  * `sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`
* Start the docker engine
  * `sudo systemctl enable --now docker`
* Curl the docker-compose file from here
  * `curl -L -O https://github.com/WitherMonarch/zabbix/raw/main/docker-compose.yml`
* Add rosemont to docker group and log back in
  * `sudo usermod -aG docker rosemont`
* Start the containers
  * `docker compose up -d`
* Wait a couple minutes for the link to the database to be correctly established and go to http://ip:8080
  * Login: **Admin, zabbix**

# Adding the agent to the server
* The docker compose file already installed an agent as a container, all thats left is adding it in zabbix. Go to **Data collection > Hosts > Zabbix-server** (edit the default host)
* Set the hostname to **zabbix-server-agent** (same as in the zabbix-agent config in the .yml)
* Set the templates to **Templates/Operating systems > Linux by Zabbix agent active** and **Zabbix server health** (using an active agent so the agent pushes the metrics to the server not the other way around. Same as the .yml config)
* Set the Host groups to **Linux servers** and **Zabbix servers**
* Add an interface: **Agent**, ip address 127.0.0.1 can be used since all dockers are in the same docker network but we'll be using DNS name **zabbix-agent**, (name of the agent service in the .yml file), select **DNS** and port **10050**
* The host should appear in the dashboard section with some metrics like cpu utilisation details




