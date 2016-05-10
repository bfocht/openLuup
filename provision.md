# Provision steps on a debian system

1. Create HyperV
2. Create Legacy Network
3. Install Debian  and check version `cat /etc/debian_version`
4. `apt-get install openssh-server`
5. get ip address `ifconfig | grep inet`
6. modify your /etc/apt/sources.list as follows:
   `deb http://ftp.us.debian.org/debian sid main`

## Install lua
```
apt-get install lua5.1
apt-get install lua5.1-socket2
apt-get install lua5.1-filesystem
apt-get install lua5.1-sec1
apt-get install liblua5.1-0
apt-get install curl
```


## Download and install vera firmware
```
mkdir /vera && cd /vera
wget http://dl.mios.com/firmware/mt7621/mios/release/mt7621_Luup_ui7-1.7.1754-en-mios.squashfs
mkdir /mnt/vera
mount -t squashfs mt7621_Luup_ui7-1.7.1754-en-mios.squashfs /mnt/vera
cp -r /mnt/vera/* /vera
sed -i '1 s|^#!/usr/bin/haserl$|#!/usr/bin/haserl --shell=/bin/bash|g' www/cgi-bin/cmh/*
echo '123456789' > /etc/cmh/PK_Account
export REMOTE_USER=aaa
find /vera -name '*.lzo' -exec sudo pluto-lzo {} \;

```

## Setup directories
```
mkdir -p /usr/local/share/lua/
mkdir -p /vera/www/cmh/skins/default/icons/
mkdir -p /vera/www/cmh/skins/default/img/devices/device_states
ln -s /vera/www/cmh/skins/default/img/devices/device_states /www/cmh/skins/default/img/devices/device_states/images

ln -s /vera/etc/cmh /etc/cmh
ln -s /vera/etc/cmh-ludl /etc/cmh-ludl
ln -s /vera/etc/cmh-lu /etc/cmh-lu
ln -s /vera/etc/cmh-ra /etc/cmh-ra
ln -s /vera/etc/cmh-static /etc/cmh-static
ln -s /vera/etc/mios /etc/mios
ln -s /vera/usr/lib/lua /usr/local/share/lua/5.1
ln /vera/mios_constants.sh /mios_constants.sh
ln /vera/nvram.sh /usr/sbin/nvram
ln -s /vera/usr/lib/cmh /usr/lib/cmh

usermod -a -G www-data bryan
chgrp -R www-data /vera/www
chmod -R 755 /vera/*.sh

```

## Bugfix https module installs in the wrong location.
```
mkdir /usr/share/lua/5.1/ssl/
ln -s /usr/share/lua/5.1/https.lua /usr/share/lua/5.1/ssl/https.lua
```

## Setup apache web server
```
apt-get install apache2.2
apt-get install libapache2-mod-proxy-html libxml2-dev
a2enmod #enable apache mod
proxy proxy_ajp proxy_http rewrite deflate headers proxy_balancer proxy_connect proxy_html
```
### copy /etc/apache2/sites-available/default
### add the following lines to  `/etc/apache2/apache2.conf`
```
<Directory /vera/www/>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
</Directory>

apachectl -k restart
```

```
apt-get install haserl_0.9.29-3_i386.deb
dpkg -i haserl_0.9.29-3_i386.deb
```

## extract lzo files
1. run pluto-lzo on /etc/chm-lu folder

## Install openLuup
1. ftp openLuup folder to `/etc/cmh-ludl/openLuup`
2. ftp openLuupExtensions to `/etc/cmh-ludl/`
3. ftp Utilities to `/etc/chm-ludl/`

## Starting openLuup
```
chmod 755 openLuup_reload
cd /etc/cmh-ludl/ && ./openLuup_reload reset
cd /etc/cmh-ludl/ && ./openLuup_reload startup.lua
cd /etc/cmh-ludl/ && ./openLuup_reload &
```

## Useful comments
```
date -s "May 5 21:16:13 MST 2016"
tail -F -n500 /var/log/cmh/LuaUPnP.log
tail -n500 -f /var/log/monit.log
dos2unix
cd /etc/cmh-ludl/ && ./openLuup_reload startup.lua
kill $(ps aux | grep 'openLuup' | awk '{print $2}')
```

## install and configure monit
```
apt-get install monit
```
copy the following to `/etc/monit/conf.d/openLuup`

```
check process op6yenLuup with pidfile /etc/cmh-ludl/openLuup.pid
    start program = "/bin/bash -c 'cd /etc/cmh-ludl/ && ./openLuup_load'"
    stop program = "/bin/bash"

monit reload

```
