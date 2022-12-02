
# Setup CentOS 7 Server 🐧
- Web Server 🌐
- DNS Server 📗
- DHCP Server 💽
- FTP Server 📁
- MariaDB 🐬

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
save

nano /etc/php.ini
----------------------------
display_errors = Off
display_errors = On

upload_max_filesize = 2M
upload_max_filesize = 1024M
---------------------------
save
```