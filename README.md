# sysadmin-public
Debian System Administrator Repository

## debian
Based on Debian Buster (current testing) for single host.

### /etc/sysctl.conf
Custom kernel flag.

### /etc/apt/\*
* Leave _sources.list_ empty, put everything on _sources.list.d_.
* Format: _[repo].[codename].list_.
* Disable translation (need to **_rm /var/lib/apt/lists/\*-en_**).

### /etc/cron.daily/ntpdate
Auto update time by cron daily. (I personally) prefer using cron instead of ntpd if time is not crucial.
Prerequisite: **_apt install ntpdate_**.

### /etc/skel/\*
* Custom ps1
* ls with color

### /usr/local/bin/backupserv
Backup config to another host using rsync. 
Prerequisite: **_apt install rsync_**.
