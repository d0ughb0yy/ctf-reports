# Pwnlab: Init
### Users
```
kent
mike
kane
```
### Credentials
```
kane:iSv5Ym2GRo
```
## Nmap Scan
```
Nmap scan report for pwnlab (192.168.1.189)
Host is up (0.00016s latency).
Not shown: 65531 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
111/tcp   open  rpcbind 2-4 (RPC #100000)
3306/tcp  open  mysql   MySQL 5.5.47-0+deb8u1
45777/tcp open  status  1 (RPC #100024)
```
## Web Services Enumeration

* Web Server: Apache 2.4.10
* Web Technology: PHP

### Nikto Scan
```
- Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:          192.168.1.189
+ Target Hostname:    192.168.1.189
+ Target Port:        80
+ Start Time:         2025-01-07 14:42:21 (GMT1)
---------------------------------------------------------------------------
+ Server: Apache/2.4.10 (Debian)
+ /: The anti-clickjacking X-Frame-Options header is not present. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ /images: The web server may reveal its internal or real IP in the Location header via a request to with HTTP/1.0. The value is "127.0.0.1". See: http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2000-0649
+ Apache/2.4.10 appears to be outdated (current is at least Apache/2.4.54). Apache 2.2.34 is the EOL for the 2.x branch.
+ /: Web Server returns a valid response with junk HTTP methods which may cause false positives.
+ /login.php: Cookie PHPSESSID created without the httponly flag. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies
+ /config.php: PHP Config file may contain database IDs and passwords.
+ /images/: Directory indexing found.
+ /icons/README: Apache default file found. See: https://www.vntweb.co.uk/apache-restricting-access-to-iconsreadme/
+ /login.php: Admin login page/section found.
+ /#wp-config.php#: #wp-config.php# file found. This file contains the credentials.
```

**/?page=**
```
I exploited an LFI on this parameter using php:// wrappers like in the article:
https://diablohorn.com/2010/01/16/interesting-local-file-inclusion-method/

I used this LFI technique to access config.php which contained db credentials
$server	  = "localhost";
$username = "root";
$password = "H4u%QJ_H99";
$database = "Users";
```
## Database Access ##

**I accessed the db using cli commands and had to put --skip_ssl parameter because it was throwing an error.**
```
+------+------------------+
| user | pass             |
+------+------------------+
| kent | Sld6WHVCSkpOeQ== | JWzXuBJJNy
| mike | U0lmZHNURW42SQ== | SIfdsTEn6I
| kane | aVN2NVltMkdSbw== | iSv5Ym2GRo
+------+------------------+
```
## Getting a Shell ##

GIF87a --> Insert GIF format to bypass file type check
Also change content type and extension:\
*Content-Type: image/gif*

To execute the shell I found an optional cookie **lang=** which looked vulnerable to LFI so
when I used it to execute my shell I was successful.

## Foothold / Privilege Escalation
### Initial foothold
Credential reusage for user kane, same credentials as the one for the website.

### Lateral Privilege Escalation
User kane had a “sticky bit” on one of the binaries in his home directory called msg2mike which used cat binary to read message.txt inside mike’s home directory. This means that when I as kane run msg2mike it will run as the user mike.
To exploit this I tried poisioning the PATH variable with a malicious cat binary.
In the /tmp/ directory I made a cat file and wrote in /bin/bash and appended the /tmp directory to the beginning of the PATH.
Executing the msg2mike binary elevated me to mike’s shell.

### Getting root
As mike I saw another binary with a “sticky bit”, this time called msg2root. The binary would echo our message to a file inside the root directory. To exploit this I simply put $() in the message and in between the brackets I inserted a reverse shell command using netcat.

**$() —> Everything inside this will be treated as a shell command.**
Using this I got root privileges on my other listener.

## Notes
* If it’s run by PHP try the ```php://filter/convert.base64-encode/``` as an LFI payload 
* Using PHP Protocol Wrappers you tell PHP to use the HTTP POST data as the entry point for it’s include*
* Directly accessing the file was not able to execute my PHP reverse shell so a cookie vulnerable to LFI was used to execute it