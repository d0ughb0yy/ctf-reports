# VulnOS v2
### Users
```
webmin
vulnosadmin
```
### Credentials
```
webmin:webmin1980
```
## Nmap Scan
```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 f5:4d:c8:e7:8b:c1:b2:11:95:24:fd:0e:4c:3c:3b:3b (DSA)
|   2048 ff:19:33:7a:c1:ee:b5:d0:dc:66:51:da:f0:6e:fc:48 (RSA)
|   256 ae:d7:6f:cc:ed:4a:82:8b:e8:66:a5:11:7a:11:5f:86 (ECDSA)
|_  256 71:bc:6b:7b:56:02:a4:8e:ce:1c:8e:a6:1e:3a:37:94 (ED25519)
80/tcp   open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-title: VulnOSv2
|_http-server-header: Apache/2.4.7 (Ubuntu)
6667/tcp open  irc     ngircd
MAC Address: 08:00:27:57:4F:AA (Oracle VirtualBox virtual NIC)
Service Info: Host: irc.example.net; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
## Web Enumeration
```
/jabc/
- Company Index site
CMS: Drupal

/jabcd0cs/
CMS: OpenDocMan
DB: MySQL
Creds: guest:guest
OpenDocMan 1.2.7
This version is vulnerable to an SQL Injection on the enpoint
/jabd0cs/ajax_udf.php?q=1&add_value=odm_user
```
**webmin:b78aae356709f8c31118ea613980954b (webmin1980)**
- hash cracked from https://www.md5online.org/md5-decrypt.html
Credentials were reused for SSH password.

## Getting a foothold
I was able to get an inital foothold through a SQL Injection vulnerability in the
OpenDocMan CMS where I was able to dump the database and crack credentials for the 
webmin user.
As soon I setup my bash shell I transfered myself linpeas.sh script to 
enumerate the machine.

## Machine Enumeration
### Arch
```
/bin/bash: ELF 32-bit LSB  executable, Intel 80386, version 1 (SYSV), 
dynamically linked (uses shared libs), for GNU/Linux 2.6.24, 
BuildID[sha1]=4ead65aeca4e9f1eabf3a0d63eb1f96c225b25fd, stripped

Linux VulnOSv2 3.13.0-24-generic #47-Ubuntu SMP i686 i686 i686 GNU/Linux

DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=14.04
DISTRIB_CODENAME=trusty
DISTRIB_DESCRIPTION="Ubuntu 14.04.4 LTS"
NAME="Ubuntu"
VERSION="14.04.4 LTS, Trusty Tahr"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 14.04.4 LTS"
VERSION_ID="14.04"
HOME_URL="http://www.ubuntu.com/"
SUPPORT_URL="http://help.ubuntu.com/"
BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
```
## Privilege Escalation
I was able to get root user using the overlayfs Local Privilege Escalation vulnerability.
Read more here: https://www.exploit-db.com/exploits/37292
This exploit happens because the overlayfs filesystem does not check permissions properly
when creating new files in the upper filesystem directories.

This can be exploited by an unprivileged process in kernels with 
CONFIG_USER_NS=y and where overlayfs has the FS_USERNS_MOUNT flag, 
which allows the mounting of overlayfs inside unprivileged mount namespaces.

## Notes
* Sometimes a machine’s kernel will be vulnerable, this is where linpeas comes in, check out what output you get under exploits
