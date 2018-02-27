# sysadmin-public
OpenSwirl Debian GNU/Linux System Administrator Configuration.
Based on Debian GNU/Linux Buster (current testing) for single host.

## What To Do After A Fresh Installation?

### Modify Repositories
* Define your debian version in _/etc/apt/apt.conf_.
* Leave _/etc/apt/sources.list_ empty, then put on _/etc/apt/sources.list.d/_ for each repo. Filename format: _[repo].[codename].list_.
* Disable translation. Delete any downloaded translation files.
```console
# rm /var/lib/apt/lists/*-en
```
* Put 99translations file on _/etc/apt/apt.conf.d_.
* Update and dist-upgrade.
```console
# apt update && apt dist-upgrade
```

### Firmware
* Check if all hardware is recognized.
* Install necessary firmware.
```console
# apt install firmware-linux firmware-linux-nonfree
```
* Pin/name hardware (NIC, USB device, etc) in _udev_ if necessary.

### Text Editor
* Personally, I prefer vim-nox over vim-tiny. 
* Remove the default vim-tiny.
```console
# apt --purge remove vim-tiny
```
* Install vim-nox.
```console
# apt install vim-nox
````
* Edit _vimrc_ and/or _vimrc.local_ to your needs.

### Remove/Disable Unwanted Service
* See installed and running services.
```console
# ps aux
# netstat -nptlu
# dpkg -l
```
* Disable exim4 (and uninstall).
```console
# systemctl stop exim4
# apt --purge remove exim4
# apt autoremove
```
* Disable rpcbind.
```console
# systemctl stop rpcbind
# systemctl disable rpcbind
```

### Update Kernel Parameters
* Create a file in _/etc/sysctl.d/_.
* Edit _vm.swappiness_ to 1 for minimize swapping. Important on SSD disk.
* Edit _net.ipv4.ip_local_port_range_ if you will have many outgoing connections. 
* Execute.
```console
# sysctl -p
```

### Install Necessary Packages
* Server monitoring: sysstat, htop, iftop.
```console
# apt install sysstat htop iftop
```
* Web-related tools: elinks, curl, dig.
```console
# apt install elinks curl dnsutils
```
* Various.
```console
# apt install build-essential
```

### Modify Skeleton Directory
* Change _.bashrc_ to enable color command prompt.
* Add custom aliases.

### SSH Banner
* Create _/etc/ssh-banner_ file.
* Update _Banner_ directive in _/etc/ssh/sshd_config_.

### SSH Key
* Copy your ssh key to _~/.ssh/authorized_keys_, or using _ssh-copy-id_ command.

### Monitoring
Create a monitoring user.

## Firewall
For a single host inside a secure network (private IP address), I would recommend using ufw. Fail2ban is necessary if you forward a port from the outside.

## Web Server

### Apache
After installation:
* Add _ServerName_ directive in _conf-available/fqdn.conf_, and enable it using _a2enconf fqdn_.
* Copy and edit _security.conf_, then enable it.
* Enable modules you want to use, and disable them if you don't.
* Example, if you want to compress json, add this line in _/etc/apache/mods-available/deflate.conf_:
```
AddOutputFilterByType DEFLATE application/json
```
* Enable status module if you want to monitor your webserver via cacti.
* Disable serve-cgi-bin configuration if you don't know what it is.
* Adjust _MaxRequestWorkers_ on _mpm_prefork.conf_ to your needs (I assume you are using PHP with this apache module).
* For multiple vhost, do not edit _sites-available/default.conf_!
* If you are using cacti or phpmyadmin, add extra security by adding Auth-Basic authentification. 

### Lighttpd
*Not today*

### Nginx (Engine X)
*Not today*

## MySQL Database
Using Percona MySQL DB:

1. Add repo to _/etc/apt/sources.list.d/_ (stable release).
```
deb http://repo.percona.com/apt stretch main
```

2. Add percona key.
```console
# apt-key adv --keyserver keys.gnupg.net --recv-keys 8507EFA5
```

3. Install Percona Server MySQL and follow installation.
```console
# apt update
# apt install percona-server-server-5.7
```

4. Make sure it reads percona config file.
```console
# update-alternatives --list my.cnf
/etc/mysql/percona-server.cnf
```

5. Config in mysqld.cnf
* _innodb-file-per-table_ (default in 5.7).
* _open-files-limit_ (see #6 below).
* _innodb_thread_concurrency_ is 2 x CPU cores.
* Set _innodb-buffer-pool-size_ to 75% of server RAM.
* Disable slow query log.
* _log_timestamps = system_ to show time in your current server timezone.

6. Update systemd (under _[Service]_)
```
LimitNOFILE=102400
LimitNPROC=102400
```

7. Don't forget to reload systemd.
```console
# systemctl daemon-reload
```

8. Create client.cnf, and update its permission to 640.
```console
# chmod 640 /etc/mysql/percona-server.conf.d/client.cnf
# chown root.mysql /etc/mysql/percona-server.conf.d/client.cnf
```

9. Restart MySQL, and remove test database.
```console
# systemctl restart mysql
# mysql_secure_installation
```

10. Add a monitoring user.


## Server Monitoring

### Cacti
1. Prerequisites: 
   * web server with php (for frontend).
   * database (MySQL DB in this case).
   
2. Apt.
```console
# apt install cacti
```

3. For extra security, add http auth basic and/or restricts only for specific IP addresses.

4. Use percona cacti template. Add percona repo if you haven't.
```console
# apt install percona-cacti-templates
```

5. Import templates you want to use.
```console
# cd /usr/share/cacti/resource/percona/templates
# ls -1
cacti_host_template_percona_apache_server_ht_0.8.6i-sver1.1.7.xml
cacti_host_template_percona_galera_server_ht_0.8.6i-sver1.1.7.xml
cacti_host_template_percona_gnu_linux_server_ht_0.8.6i-sver1.1.7.xml
cacti_host_template_percona_jmx_server_ht_0.8.6i-sver1.1.7.xml
cacti_host_template_percona_memcached_server_ht_0.8.6i-sver1.1.7.xml
cacti_host_template_percona_mongodb_server_ht_0.8.6i-sver1.1.7.xml
cacti_host_template_percona_mysql_server_ht_0.8.6i-sver1.1.7.xml
cacti_host_template_percona_nginx_server_ht_0.8.6i-sver1.1.7.xml
cacti_host_template_percona_openvz_server_ht_0.8.6i-sver1.1.7.xml
cacti_host_template_percona_rds_server_ht_0.8.6i-sver1.1.7.xml
cacti_host_template_percona_redis_server_ht_0.8.6i-sver1.1.7.xml
# php /usr/share/cacti/cli/import_template.php \
--filename=cacti_host_template_percona_gnu_linux_server_ht_0.8.6i-sver1.1.7.xml \
--with-user-rras='1:2:3:4'
# php /usr/share/cacti/cli/import_template.php \
--filename=cacti_host_template_percona_mysql_server_ht_0.8.6i-sver1.1.7.xml \
--with-user-rras='1:2:3:4'
# php /usr/share/cacti/cli/import_template.php \
--filename= cacti_host_template_percona_apache_server_ht_0.8.6i-sver1.1.7.xml \
--with-user-rras='1:2:3:4'
```
6. Linux server template:
   * Copy monitoring user rsa-key (_id_rsa_ and _id_rsa.pub_) to _/etc/cacti_. Make sure _id_rsa_ file only root and poller (cacti user) have read access.
   * Copy configuration from _/usr/share/cacti/site/scripts/ss_get_by_ssh.php_ to _/etc/cacti/ss_get_by_ssh.php.cnf_ and modify to your needs (only the _CONFIGURATION_ part, delete the reset).
   * Add device and graph from frontend.

7. MySQL server template:
   * Copy configuration from _/usr/share/cacti/site/scripts/ss_get_mysql_stats.php_ to _/etc/cacti/ss_get_mysql_stats.php.cnf_ and modify to your needs (only the _CONFIGURATION_ part, delete the reset).
   * Add device and graph from frontend.

8. Apache server template:
   * Enable Apache mod-status only allowed for cacti poller IP addresses.
   * Edit _/etc/cacti/ss_get_by_ssh.php.cnf_ to your needs.
   * Add device and graph from frontend.

9. Further reads about [percona cacti templates](https://www.percona.com/doc/percona-monitoring-plugins/LATEST/cacti/index.html).


### Zabbix
1. Prerequisites: 
   * web server with php (for frontend).
   * database (I'm using MySQL).
   
2. Add zabbix repo.
```console
# echo 'deb http://repo.zabbix.com/zabbix/3.4/debian stretch main' > /etc/apt/sources.list.d/stretch.zabbix.list
# apt-key adv --keyserver keys.gnupg.net --recv-keys A14FE591
# apt update
```

3. Install zabbix (with MySQL as its database) and php frontend.
```console
# apt install zabbix-server-mysql zabbix-frontend-php
```

4. Update config to your needs.
5. For extra security, add http auth basic and/or restricts only for specific IP addresses.

## Backup
*Not today*
