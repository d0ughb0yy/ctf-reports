# Gallery
### Users
```
mike
ubuntu
```
### Credentials
```
mike:b3stpassw0rdbr0xx
```
## Recon
### Nmap Scan
```
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
8080/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
```
* Web servers: Apache 2.4.29
## Foothold / Privilege Escalation
Gained a foothold through a chain of exploits starting with an SQL injection on the admin panel.
Using the ```admin’ OR ‘1’=’1’#``` payload I was able to login as admin.
Inside I saw that its running a Simple Image Gallery CMS which was vulnerable to a file upload vulnerability in a way that it didnt check the types of files so I was able to upload a php reverse shell.
### Privilege Escalation
I was able to make a lateral move in the machine thanks to the backups folder under /var directory, which contained the .bash_history file of a user “mike” and inside I found his password in clear text.

I was able to elevate my privileges with the sudo permission which allowed me to execute a binary as the root user and I could use nano with that binary, search on gtfo bins gave me a payload:
```reset; sh 1>&0 2>&0``` 
which when executed in a nano terminal gave me a root shell.