# sysadmin-public
Debian System Administrator Repository

## debian
Based on Debian Buster (current testing) for single host.

### sysctl
Custom kernel flag

### apt
* Leave sources.list empty, put everything on sources.list.d
* Format: [repo].[codename].list
* Disable translation (need to **rm /var/lib/apt/lists/\*-en**)

### ntpdate
Need to **apt install ntpdate**

### skel
* Custom ps1
* ls with color

