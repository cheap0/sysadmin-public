# sysadmin-public
Debian System Administrator Repository

## debian
Based on Debian Buster (current testing) for single host.

### /etc/sysctl.conf
Custom kernel flag

### /etc/apt/\*
* Leave sources.list empty, put everything on sources.list.d
* Format: [repo].[codename].list
* Disable translation (need to **rm /var/lib/apt/lists/\*-en**)

### /etc/cron.daily/ntpdate
Need to **apt install ntpdate**

### /etc/skel/\*
* Custom ps1
* ls with color

