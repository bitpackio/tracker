
# enable the univention unmaintained repository

root@fs01:~# ucr set repository/online/unmaintained=yes
root@fs01:~# apt-get update

# install the tracker packages

root@fs01:~# apt-get install tracker tracker-miner-fs libtracker-sparql-1.0-dev dconf-tools

# setup the tracker user

root@fs01:~# useradd -m mdssvc
root@fs01:~# adduser mdssvc root

# setup dbus

Place the service files to /usr/share/dbus-1/system-services

root@fs01:~# ls /usr/share/dbus-1/system-services/system-tracker-*
/usr/share/dbus-1/system-services/system-tracker-extract.service     
/usr/share/dbus-1/system-services/system-tracker-miner-fs.service
/usr/share/dbus-1/system-services/system-tracker-miner-apps.service  
/usr/share/dbus-1/system-services/system-tracker-store.service

# setup systemd

Place the service files to /usr/lib/systemd/system

root@fs01:~# ls /usr/lib/systemd/system/system-tracker-*
/usr/share/dbus-1/system-services/system-tracker-extract.service     
/usr/share/dbus-1/system-services/system-tracker-miner-fs.service
/usr/share/dbus-1/system-services/system-tracker-miner-apps.service  
/usr/share/dbus-1/system-services/system-tracker-store.service

# enable systemd services

root@fs01:~# systemctl enable system-tracker-store
root@fs01:~# systemctl enable system-tracker-extract
root@fs01:~# systemctl enable system-tracker-miner-fs

# setup dbus configuration

Place the dbus tracker config to /etc/dbus-1/system.d/org.freedesktop.Tracker1.conf

# setup tracker configuration

root@fs01:~# su - mdssvc
mdssvc@fs01:~$ dbus-launch --sh-syntax > .dbus_settings
mdssvc@fs01:~$ source .dbus_settings 
mdssvc@fs01:~$ dconf write /org/freedesktop/tracker/miner/files/index-recursive-directories "['/srv/samba']"
mdssvc@fs01:~$ dconf read /org/freedesktop/tracker/miner/files/index-recursive-directories
['/srv/samba']

# start the tracker services

root@fs01:~# systemctl start system-tracker-store
root@fs01:~# systemctl start system-tracker-extract
root@fs01:~# systemctl start system-tracker-miner-fs

# verify tracker settings

mdssvc@fs01:~$ gsettings list-recursively | grep -i org.freedesktop.Tracker | sort | uniq

# enable samba4 spotlight search

UMC > Domain > Shares > Share > Advanced settings > Samba custom settings:

Key: spotlight
Value: yes

root@fs01:~# grep spotlight /etc/samba/shares.conf.d/Share
spotlight = yes

root@fs01:~# vi /etc/samba/smb.conf

fruit:model = Xserve
fruit:aapl = yes
rpc_server:mdssvc = embedded

root@fs01:~# vi /etc/init.d/samba-ad-dc 

export TRACKER_BUS_TYPE=system

root@fs01:~# systemctl daemon-reload
root@fs01:~# service samba-ad-dc restart

# verify samba4 logging

root@fs01:~# grep tracker /var/log/samba/log.smbd
pid=12530] ../../source3/rpc_server/mdssvc/mdssvc.c:729(tracker_con_cb)

root@fs01:~# grep mdssvc /var/log/samba/log.smbd
pid=12530] ../../source3/rpc_server/mdssvc/mdssvc.c:1148(slrpc_open_query)
pid=12530] ../../source3/rpc_server/mdssvc/mdssvc.c:1652(slrpc_fetch_attributes)
pid=12530] ../../source3/rpc_server/mdssvc/mdssvc.c:1357(slrpc_fetch_query_results)
pid=12530] ../../source3/rpc_server/mdssvc/mdssvc.c:1748(slrpc_close_query)

# verify system logging with syslog (or journalctl -f)

root@fs01:~# grep tracker /var/log/syslog
root@fs01:~# grep tracker-miner-fs /var/log/syslog

# tracker command line search

root@fs01:~# export TRACKER_BUS_TYPE="system"
root@fs01:~# tracker search foo

# tracker verbose command line search

root@fs01:~# TRACKER_VERBOSITY=3 tracker search foo

# tracker status

root@fs01:~# tracker daemon

# tracker logging and verbosity

root@fs01:~# tracker daemon --get-log-verbosity
root@fs01:~# tracker daemon --list-processes
root@fs01:~# tracker daemon --list-miners-running
root@fs01:~# tracker daemon --list-common-statuses

# requeue file to index

root@fs01:~# TRACKER_VERBOSITY=3 tracker index -f /srv/samba/share/file-to-requeue
root@fs01:~# tracker index -f /srv/samba/share/file-to-requeue

# reset file to index

root@fs01:~# TRACKER_VERBOSITY=3 tracker reset -f /srv/samba/share/file-to-reset 
root@fs01:~# tracker reset -f /srv/samba/share/file-to-reset

# for tracker logging add second Environment variable TRACKER_USE_LOG_FILES to each service and restart the daemon

root@fs01:~# vi /usr/lib/systemd/system/system-tracker-store.service 

Environment="TRACKER_USE_LOG_FILES="

# restart all daemons

root@fs01:~# systemctl daemon-reload
root@fs01:~# systemctl restart system-tracker-store
root@fs01:~# systemctl restart system-tracker-miner-apps.service 
root@fs01:~# systemctl restart system-tracker-miner-apps.service 
root@fs01:~# systemctl restart system-tracker-miner-fs.service 
root@fs01:~# systemctl restart system-tracker-extract.service 

# tracker log files

root@fs01:~# ls -l /srv/lib/mdssvc/.local/share/tracker/*.log
/srv/lib/mdssvc/.local/share/tracker/tracker-extract.log
/srv/lib/mdssvc/.local/share/tracker/tracker-miner-apps.log
/srv/lib/mdssvc/.local/share/tracker/tracker-miner-fs.log
/srv/lib/mdssvc/.local/share/tracker/tracker-store.log

# tracker extract modules

root@fs01:~# dpkg -L tracker-extract

root@fs01:~# ls -l /usr/lib/x86_64-linux-gnu/tracker-1.0/extract-modules/
libextract-abw.so
libextract-bmp.so
libextract-dummy.so
libextract-dvi.so
libextract-epub.so
libextract-flac.so
libextract-gif.so
libextract-gstreamer.so
libextract-html.so
libextract-icon.so
libextract-iso.so
libextract-jpeg.so
libextract-mp3.so
libextract-msoffice-xml.so
libextract-msoffice.so
libextract-oasis.so
libextract-pdf.so
libextract-playlist.so
libextract-png.so
libextract-ps.so
libextract-text.so
libextract-tiff.so
libextract-vorbis.so
libextract-xmp.so
libextract-xps.so

# use the tracker user

root@fs01:~# su - mdssvc
mdssvc@fs01:~$ export TRACKER_BUS_TYPE="system"
mdssvc@fs01:~$ tracker search foo

# location of the tracker database

mdssvc@fs01:~$ file .cache/tracker/meta.db
.cache/tracker/meta.db: SQLite 3.x database, last written using SQLite version 3016002

mdssvc@fs01:~$ du -sh .cache/tracker/meta.db
1,2G	.cache/tracker/meta.db

# Docs

Tracker

https://wiki.gnome.org/Projects/Tracker

Features

https://wiki.gnome.org/Projects/Tracker/Features

Getting Started

https://wiki.gnome.org/Projects/Tracker/Documentation/GettingStarted

Supported File Formats

https://wiki.gnome.org/Projects/Tracker/SupportedFormats

First 5 minutes with Tracker

https://wiki.gnome.org/Projects/Tracker/Documentation/First5Minutes

Debugging

https://wiki.gnome.org/Projects/Tracker/Documentation/Debugging

Samba Spotlight Start Script

https://wiki.samba.org/index.php/Samba_Spotlight_Start_Script

Samba Spotlight

https://wiki.samba.org/index.php/Spotlight

