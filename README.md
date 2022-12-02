
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
