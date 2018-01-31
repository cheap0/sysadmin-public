# sysadmin-public
OpenSwirl Debian GNU/Linux System Administrator Configuration.
Based on Debian GNU/Linux Buster (current testing) for single host.

## What To Do After A Fresh Installation?

### Modify Repositories
* Define your debian version in _/etc/apt/apt.conf_.
* Leave _/etc/apt/sources.list_ empty, then put on _/etc/apt/sources.list.d/_ for each repo. Filename format: _[repo].[codename].list_.
* Disable translation. Delete any downloaded translation files.
```console
# rm /var/lib/apt/lists/\*-en
```
* Put 99translations file on _/etc/apt/apt.conf.d_.
* Update and dist-upgrade
```console
# apt update && apt dist-upgrade
```

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
* Edit vimrc/vimrc.local to your needs.

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

## Web Server

### Apache

### Lighttpd

## MySQL Database
Using Percona MySQL DB:

1. Add repo to _/etc/apt/sources.list.d/_ (stable release).
```
deb http://repo.percona.com/apt stretct main
```

2. Add percona key.
```console
# apt-key adv --keyserver keys.gnupg.net --recv-keys 8507EFA5
```

3. Install Percona Server MySQL.
```console
# apt update
# apt install percona-server-server-5.7
```

   Follow installation.

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

6. Update systemd (under _[Service]_
```
LimitNOFILE=102400
LimitNPROC=102400
```

   Don't forget to reload systemd.
```console
# systemctl daemon-reload
```

7. Create client.cnf, and update its permission to 640.


## Server Monitoring

### Cacti

### Zabbix

## Backup

