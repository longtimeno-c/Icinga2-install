# How to setup Icinga2

`Sudo -s`

## Debian Repository

```
apt update
apt -y install apt-transport-https wget gnupg

wget -O - https://packages.icinga.com/icinga.key | gpg --dearmor -o /usr/share/keyrings/icinga-archive-keyring.gpg

DIST=$(awk -F"[)(]+" '/VERSION=/ {print $2}' /etc/os-release); \
 echo "deb [signed-by=/usr/share/keyrings/icinga-archive-keyring.gpg] https://packages.icinga.com/debian icinga-${DIST} main" > \
 /etc/apt/sources.list.d/${DIST}-icinga.list
 echo "deb-src [signed-by=/usr/share/keyrings/icinga-archive-keyring.gpg] https://packages.icinga.com/debian icinga-${DIST} main" >> \
 /etc/apt/sources.list.d/${DIST}-icinga.list

```

`apt update`


## Install Icinga 2

`apt install icinga2`

### Set up Check Plugins

`apt install monitoring-plugins`

### Set up Icinga 2 API

See the API chapter for details, or follow the steps below to set up the API quickly:
Run the following command to:
* enable the api feature,
* set up certificates, and
* add the API user root with an auto-generated password in the configuration file:
 `/etc/icinga2/conf.d/api-users.conf`

`icinga2 api setup`

Restart Icinga 2 for these changes to take effect.

`systemctl restart icinga2`

## Set up Icinga DB Redis

### Install Icinga DB Redis Package

`apt install icingadb-redis`

### Run Icinga DB Redis

`systemctl enable --now icingadb-redis`

_Note: This is usually already running so you may receive an error here
Run V to see if it is._

`systemctl status icingadb-redis`

_To manage Redis settings:_

`nano /etc/icingadb-redis/icingadb-redis.conf`

## Enable Icinga DB Feature

`icinga2 feature enable icingadb`
 
`systemctl restart icinga2`

_Link for referencing so far_ - https://icinga.com/docs/icinga-2/latest/doc/02-installation/01-Debian/


## Installing Icinga DB

```
apt update
apt -y install apt-transport-https wget gnupg

wget -O - https://packages.icinga.com/icinga.key | apt-key add -

DIST=$(awk -F"[)(]+" '/VERSION=/ {print $2}' /etc/os-release); \
 echo "deb https://packages.icinga.com/debian icinga-${DIST} main" > \
 /etc/apt/sources.list.d/${DIST}-icinga.list
 echo "deb-src https://packages.icinga.com/debian icinga-${DIST} main" >> \
 /etc/apt/sources.list.d/${DIST}-icinga.list
```

`apt update`


#### Installing Icinga DB Package

`apt install icingadb`

### Install SQL

`apt install default-mysql-server default-mysql-client`

### Setting up a MySQL or MariaDB Database

`mysql -u root -p`

```
CREATE DATABASE icingadb;
CREATE USER 'icingadb'@'localhost' IDENTIFIED BY 'CHANGEME';
GRANT ALL ON icingadb.* TO 'icingadb'@'localhost';
```

`Ctrl c`

`mysql -u root -p icingadb </usr/share/icingadb/schema/mysql/schema.sql`

**DO NOT SET A PASSWORD FOR ROOT YET!**


## Configuring Icinga DB
Icinga DB installs its configuration file to -> 
**MAKE SURE THIS MATCHES YOUR SQL CONFIG!!**
`nano /etc/icingadb/config.yml`

### Running Icinga DB

systemctl enable --now icingadb

https://icinga.com/docs/icinga-db/latest/doc/02-Installation/03-Debian/#installing-icinga-db-package


## Installing Icinga DB Web on Debian

```
apt update
apt -y install apt-transport-https wget gnupg

wget -O - https://packages.icinga.com/icinga.key | gpg --dearmor -o /usr/share/keyrings/icinga-archive-keyring.gpg

DIST=$(awk -F"[)(]+" '/VERSION=/ {print $2}' /etc/os-release); \
 echo "deb [signed-by=/usr/share/keyrings/icinga-archive-keyring.gpg] https://packages.icinga.com/debian icinga-${DIST} main" > \
 /etc/apt/sources.list.d/${DIST}-icinga.list
 echo "deb-src [signed-by=/usr/share/keyrings/icinga-archive-keyring.gpg] https://packages.icinga.com/debian icinga-${DIST} main" >> \
 /etc/apt/sources.list.d/${DIST}-icinga.list

apt update
```

### Installing the Package

```
apt install icingadb-web

https://icinga.com/docs/icinga-db-web/latest/doc/02-Installation/Debian/#installing-icinga-db-web-package
```

## Wizard setup:
_Notes:
To change web settings ->
`nano /etc/icingaweb2/resources.ini`_

**Go to webpage:**
`http://'hostaddress'/icingaweb2`

**To get token key:**

`icingacli setup config directory --group icingaweb2;`

`icingacli setup token create;`

Now copy this token and paste into setup wizard.

**Install Imagick (missing module)**
`apt install php-imagick`

**Restart Apache2**
`systemctl restart apache2`

**Enter SQL information…**
This was set previously

**Set a password for root:**

`mysql -u root -p`

`ALTER USER 'root'@'localhost' IDENTIFIED BY '1234'; flush privileges;`

`Ctrl c`


Icinga DB info -> `nano /etc/icingadb/config.yml`

Icinga API info -> `nano /etc/icinga2/conf.d/api-users.conf`


## IDO

This part is where it can go wrong! make sure to change root password when told! 

**Remove root password**

`mysql -u root -p`

`ALTER USER 'root'@'localhost' IDENTIFIED BY ''; flush privileges;`

`Ctrl C`

`apt-get install icinga2-ido-mysql`

**Use defaults and remember set password **

`icinga2 feature enable ido-mysql`

`icinga2 feature enable command`

`service icinga2 restart`

**Set root password:**

`mysql -u root -p`

`ALTER USER 'root'@'localhost' IDENTIFIED BY 'PASSWORD'; flush privileges;`

`Ctrl C`

Now try validate IDO


**API username:**

nano /etc/icinga2/conf.d/api-users.conf



# Clients VIA nrpe 

sudo -s 

apt install nagios-nrpe-plugin

apt install nagios-nrpe-server

nano /etc/nagios/nrpe_local.cfg

Update file with -> 
_The ones commented out are custom scripts to add nrpe scripts go to:
`/usr/lib/nagios/plugins/`_


```
allowed_hosts= `icinga host address`

command[check_users]=/usr/lib/nagios/plugins/check_users -w 5 -c 10
command[check_load]=/usr/lib/nagios/plugins/check_load -r -w .15,.10,.05 -c .30,.25,.20
command[check_zombie_procs]=/usr/lib/nagios/plugins/check_procs -w 5 -c 10 -s Z
command[check_total_procs]=/usr/lib/nagios/plugins/check_procs -w 150 -c 200
command[check_root]=/usr/lib/nagios/plugins/check_disk -w 20% -c 10% -p /
command[check_boot]=/usr/lib/nagios/plugins/check_disk -w 5% -c 2% -p /boot
command[check_apt]=/usr/lib/nagios/plugins/check_apt
#command[check_reboot_required]=/usr/lib/nagios/plugins/check_reboot_required
command[check_load]=/usr/lib/nagios/plugins/check_load -r -w 2,1,.5 -c 4,3,2
command[check_v4l2rtspserver]=/usr/lib/nagios/plugins/check_procs -c 1:1 -C v4l2rtspserver
command[check_motioneye]=/usr/lib/nagios/plugins/check_procs -C meyectl -c1:1
command[check_motion]=/usr/lib/nagios/plugins/check_procs -C motion -c 1:1
command[checkomxplayer]=/usr/lib/nagios/plugins/check_procs -c 1:1 -C omxplayer
command[check_ntp]=/usr/lib/nagios/plugins/chexk_ntp -H localhost

```

### Save and restart NRPE 

`systemctl restart nagios-nrpe-server`


### Back over to -> Icinga2 host machine 

`apt install nagios-nrpe-plugin`

**Run a test**

/usr/lib/nagios/plugins/check_nrpe -H 'address' -c check_load




## Setup services and config commands

`nano /etc/icinga2/conf.d/"machine name"`

object Host "CHANGE TO NAME" {
        import "generic-host"
        address = "ADRESSOFDEVICE"
        vars.nrpe = 1
        vars.static_ip = 1
        vars.os = "Linux"
}

**Copy over these configs to: **
`/etc/icinga2/conf.d/`
```
000services.conf 
000template.conf
000commands.conf
```
**Reboot icinga to apply changes**
`systemctl restart icinga2`

## To install ntp:
_`/etc/icinga2/conf.d/`_

`apt install ntp `

**Add to host or groups **

`groups.ini`

Add another group and add to the devices file:


## Optional - Grafana

_https://github.com/chrisss404/icinga2-influxdb-grafana_

**Install packages:**
```
wget -O - https://packages.grafana.com/gpg.key | apt-key add -

echo "deb https://packages.grafana.com/oss/deb stable main" > /etc/apt/sources.list.d/grafana.list
```

**More packages:**
```
apt-get update
apt-get install grafana influxdb influxdb-client
```
```
systemctl daemon-reload
systemctl enable grafana-server.service
systemctl start grafana-server.service
```

`influx`
```
CREATE DATABASE icinga2;
CREATE USER icinga2 WITH PASSWORD '1234';
```

**Now enable Influx**

`icinga2 feature enable influxdb`

`systemctl restart icinga2`


**Update influx config**
—> 
```
/**
 * The InfluxdbWriter type writes check result metrics and
 * performance data to an InfluxDB v1 HTTP API
 */

object InfluxdbWriter "influxdb" {
  host = "127.0.0.1"
  port = 8086
  database = "icinga2"
  username = "icinga2"
  password = "your-icinga2-pwd"
  enable_send_thresholds = true
  enable_send_metadata = true
  flush_threshold = 1024
  flush_interval = 10s
  host_template = {
    measurement = "$host.check_command$"
    tags = {
      hostname = "$host.name$"
    }
  }
  service_template = {
    measurement = "$service.check_command$"
    tags = {
      hostname = "$host.name$"
      service = "$service.name$"
    }
  }
}
```

`systemctl restart icinga2.service`

### Navigate to Grafana web interface

Url: `http://localhost:3000` Username: admin Password: admin

**Create new Grafana datasource**

Add Datasource: http://hostname:3000/datasources/new?gettingstarted
Name: InfluxDB Type: InfluxDB 
Default: Yes
Url: http://localhost:8086 
Access: Server (Default)
Database: icinga2 User: root Password: 1234

http://hostname:3000/d/ddg2mzt6ren7kd/base-metrics?orgId=1&refresh=30s


**Import Grafana dashboard**

Import Dashboard: http://hostname:3000/dashboard/import
Paste JSON from https://raw.githubusercontent.com/Mikesch-mp/icingaweb2-module-grafana/v1.1.8/dashboards/influxdb/base-metrics.json
Select influx DB

**NAME MUST BE SET TO** ‘base-metrics’ 


## Add Icinga Web Grafana module
Make sure to update the version numbers in these scripts:


```
cd /usr/share/icingaweb2/modules
wget -qO- https://github.com/Mikesch-mp/icingaweb2-module-grafana/archive/refs/tags/v2.0.3.tar.gz | tar xvz
mv icingaweb2-module-grafana-2.0.3 grafana
mkdir /etc/icingaweb2/modules/grafana
```
```
MODULE_VERSION="2.0.3"
ICINGAWEB_MODULEPATH="/usr/share/icingaweb2/modules"
REPO_URL="https://github.com/Mikesch-mp/icingaweb2-module-grafana"
TARGET_DIR="${ICINGAWEB_MODULEPATH}/grafana"
URL="${REPO_URL}/archive/v${MODULE_VERSION}.tar.gz"
install -d -m 0755 "${TARGET_DIR}"
wget -q -O - "$URL" | tar xfz - -C "${TARGET_DIR}" --strip-components 1
```

**Config these files:**

`nano /etc/icingaweb2/modules/grafana/config.ini`
https://github.com/chrisss404/icinga2-influxdb-grafana/blob/master/configs/icingaweb2-grafana-config.ini
Add this to the config:
defaultdashboarduid = "_This uid can be found in grafana dashboard url_"

`nano /etc/icingaweb2/modules/grafana/graphs.ini`
https://github.com/chrisss404/icinga2-influxdb-grafana/blob/master/configs/icingaweb2-grafana-graphs.ini

**Enable anonymous access**
_`vim /etc/grafana/grafana.ini`_
OR 
_`nano /etc/grafana/grafana.ini`_

```
[auth.basic]
enabled = true
```
```
[auth.anonymous]
# enable anonymous access
enabled = true

org_name = Viewer
```
```
# set to true if you want to allow browsers to render Grafana in a <frame>, <iframe>, <embed> or <object>. default is false.
-;allow_embedding = false
+allow_embedding = true
```

_Tip:
This will search and change words required in vim without having to scroll forever! 
`:%s/word/newword`_

`systemctl restart grafana-server.service`


**Enable Icinga Web Grafana module**
```
icingacli module enable grafana
chown -R www-data:icingaweb2 /etc/icingaweb2
```

Disabling not needed ones services to graph:
_`nano /etc/icinga2/conf.d/services.conf`_
e.g. ssh, http, disk, icinga 

+  vars.grafana_graph_disable = true

`systemctl restart icinga2.service`
