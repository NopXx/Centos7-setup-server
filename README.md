
# Setup CentOS 7 Server 🐧
- [Web Server](#setup-webserver) 🌐
- [DNS Server](#setup-dns) 📗
- [DHCP Server](#setup-dhcp) 💽
- [FTP Server](#setup-ftp) 📁
- [MariaDB](#setup-mariadb) 🐬
- [Firewall](#setup-firewall) 🧱
- [Route](#setup-route) 🛣
- [Service](#setup-service) 🧰

# Get Started 🚀  
## `Install Package use USB`

### `mount usb`
```javascript
mkdir /media/cdrom
fdisk -l
|Device      |Boot  |
|------------|------|
|  /dev/sdb1 |  *   |

mount /dev/sdb1 /media/cdrom
```
### `Install Package`
```javascript
yum --disablerepo=\* --enablerepo=c7-media install lynx -y
yum --disablerepo=\* --enablerepo=c7-media install bind -y
yum --disablerepo=\* --enablerepo=c7-media install dhcp -y
yum --disablerepo=\* --enablerepo=c7-media install mariadb-server -y
yum --disablerepo=\* --enablerepo=c7-media install php-mysql -y
yum --disablerepo=\* --enablerepo=c7-media install vsftpd -y

```
## `Setup WebServer`
```javascript
nano /etc/httpd/conf/httpd.conf
---------------------------------
#Add
#Listen 12.34.56.78:80
Listen 192.168.1.2:80

#ServerName www.example.com:80
ServerName www.ltc.com:80

<IfModule dir_module>
    DirectoryIndex index.html index.php index.htm
</IfModule>

----------------------------------
save & exit
```

```javascript
nano /etc/php.ini
----------------------------
display_errors = Off
display_errors = On

upload_max_filesize = 2M
upload_max_filesize = 1024M
---------------------------
save & exit
```

```javascript
chcon -Rv -t public_content_rw_t /var/www
chown webmaster -R /var/www
chmod -R 777 /var/www
setsebool -P httpd_anon_write 1
setsebool -P httpd_can_network_connect_db 1
```

## `Setup DNS`
```javascript
nano /etc/named.conf
------------------------
listen-on port 53 { 127.0.0.1; };
listen-on port 53 { 127.0.0.1; 192.168.1.2;};

allow-query     { localhost; };
allow-query     { localhost; any;};
------------------------
save & exit
```

```javascript
------------------------
nano /etc/named.rfc1912.zones
-------------------------
#edit

zone "localhost.localdomain" IN {
        type master;
        file "named.localhost";
        allow-update { none; };
};
zone "ltc.com" IN {
        type master;
        file "for.zone";
        allow-update { none; };
};

zone "0.in-addr.arpa" IN {
        type master;
        file "named.empty";
        allow-update { none; };
};
zone "1.168.192.in-addr.arpa" IN {
        type master;
        file "re.zone";
        allow-update { none; };
};
-------------------
save & exit
```

```javascript
cd /var/named
cp named.localhost for.zone
nano for.zone
-----------------------------
#edit
$TTL 1D
@       IN SOA  @ rname.invalid. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      @
        A       127.0.0.1
        AAAA    ::1
-------------------------------------------------------------------
$TTL 1D
@       IN SOA  it.ltc.com. root.ltc.com. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
@       IN NS      it.ltc.com.
it      IN A       192.168.1.2

-------------------------
savve
```

```javascript
cp for.zone re.zone
nano re.zone
---------------------------
$TTL 1D
@       IN SOA  it.ltc.com. root.ltc.com. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
@       IN NS      it.ltc.com.
2       IN PTR     it.ltc.com.
--------------------------
save & exit
```

```javascript
chown named: for.zone
chown named: re.zone
```

```javascript
#checkconfig
named-checkconf /etc/named.conf

#checkzone
------
named-checkzone for.zone for.zone
zone for.zone/IN: loaded serial 0
OK
named-checkzone re.zone re.zone
zone re.zone/IN: loaded serial 0
OK
```

## `Setup DHCP`

```javascript
cp /usr/share/doc/dhcp-4.2.5/dhcpd.conf.example /etc/dhcp/dhcpd.conf
Enter
cp: overwrite ‘/etc/dhcp/dhcpd.conf’? Y
nano /etc/dhcp/dhcpd.conf
--------------------------
#ลบหมดยกเว้น
------------------
# dhcpd.conf
#
# Sample configuration file for ISC dhcpd

option domain-name "example.org";
option domain-name-servers ns1.example.org, ns2.example.org;

default-lease-time 600;
max-lease-time 7200;

#ddns-update-style none;

#authoritative;

log-facility local7;

subnet 10.254.239.32 netmask 255.255.255.224 {
  range dynamic-bootp 10.254.239.40 10.254.239.60;
  option broadcast-address 10.254.239.31;
  option routers rtr-239-32-1.example.org;
}

#edit
------------------
# dhcpd.conf
#
# Sample configuration file for ISC dhcpd

option domain-name "ltc.com";
option domain-name-servers 192.168.1.2;

default-lease-time 600;
max-lease-time 7200;

#ddns-update-style none;

authoritative;

log-facility local7;

subnet 192.168.1.0 netmask 255.255.255.0 {
  range dynamic-bootp 192.168.1.100 192.168.1.200;
  option broadcast-address 192.168.1.255;
  option routers 192.168.1.1;
}

subnet 192.168.10.0 netmask 255.255.255.0 {
  range dynamic-bootp 192.168.10.100 192.168.10.200;
  option broadcast-address 192.168.10.255;
  option routers 192.168.10.1;
}

subnet 192.168.20.0 netmask 255.255.255.0 {
  range dynamic-bootp 192.168.20.100 192.168.20.200;
  option broadcast-address 192.168.20.255;
  option routers 192.168.20.1;
}
-----------------------
save & exit
```

```javascript
nano /etc/sysconfig/dhcpd
---------------
#Add
DHCPDAGS=enp2s0
DHCPDAGS=enp2s0.10
DHCPDAGS=enp2s0.20
------------
save & exit
```

```javascript
nano /etc/hosts
-------------
#Add
192.168.1.2 it  ltc.com
-----------
save & exit
```

## `Setup FTP`

```javascript
nano /etc/passwd
-----------------
#edit
webmaster:x:1000:1000:webmaster:/home/webmaster:/bin/bash

webmaster:x:1000:1000:webmaster:/var/www:/bin/bash
-----------------
save & exit
```

```javascript
nano /etc/vsftpd/vsftpd.conf
------------
#edit
anonymous_enable=YES
anonymous_enable=NO

#chroot_local_user=YES
chroot_local_user=YES

#Add
userlist_deny=NO
userlist_file=/etc/vsftpd/user_list
allow_writeable_chroot=YES
----------------
save & edit
```

```javascript
nano /etc/vsftpd/user_list
-------------------------
#Add
webmaster
-------------------------
save & exit
```

```javascript
setsebool -P ftpd_full_access 1
```

## `Setup MariaDB`

```javascript
systemctl start mariadb
mysql_secure_installation
---------------------------
1. Enter
2. y
3. input password
4. y
5. n
6. n
7. y
-------------------------
mysql -u root -p
input password
------------------------
create user 'admin'@'%';
grant all on *.* to 'admin'@'%' with grant option;
exit
```

## `Setup Firewall`

```javascript
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=dns
firewall-cmd --permanent --add-service=ftp
firewall-cmd --permanent --add-service=dhcp
firewall-cmd --permanent --add-port=21/tcp
firewall-cmd --permanent --add-port=3306/tcp
firewall-cmd --reload
```

## `Setup Route`

```javascript
nano /etc/init.d/network
--------------------------
#ไปที่บันทัดสุดท้าย

esac

exit $rc

esac
ip r d 192.168.1.0/24
ip r d 192.168.10.0/24
ip r d 192.168.20.0/24

exit $rc
-------------------------
save & edit
```

## `Setup Service`
```
systemctl enable httpd
systemctl enable named
systemctl enable dhcpd
systemctl enable vsftpd
systemctl enable mariadb
systemctl enable network
```

## `End`

```javascript
reboot
```