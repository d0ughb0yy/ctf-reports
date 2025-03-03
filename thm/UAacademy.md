# UA Academy
### Users
```
deku
```
### Credentials
```
deku:One?For?All_!!one1/A
AllmightForEver!!! -- passphrase for .jpg file
```
## Recon
### Nmap Scan
```
Nmap scan report for 10.10.111.143
Host is up (0.072s latency).
Not shown: 65175 closed tcp ports (reset), 358 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
### Web Recon
* **Server: Apache 2.4.41** 
```
 :: Method           : GET
 :: URL              : http://10.10.111.143/FUZZ/
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/raft-small-directories.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response status: 404
________________________________________________

assets                  [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 68ms]
```
## Foothold / Privilege Escalation
Found index.php in the assets directory. 
I fuzzed for possible parameters and found that the cmd= parameter was present and made the page vulnerable to command injection.
Using a python reverse shell I was able to get a foothold in the machine.
### Lateral Move
As the www user I found a .jpg file and downloaded it. The file is corrupted and can't be opened, so I had to fix it with hexedit.
You just need to check the file signature for .jpg files from here https://en.wikipedia.org/wiki/List_of_file_signatures, and repair
the broken photo file.
When fixed it shows up successfully, so I thought maybe steganography was being used to hide something inside.
Using steghide you can see what is hidden within the jpg, but in this case it needed a password to extract.
After a bit more enumeration I found a passphrase.txt file with some base64 encoded string and when I decoded it,
it was the passphrase for the .jpeg file.
As I got the passphrase for the stego file, I could extract it and saw that the jpg file contains user credentials for user deku.
### Privilege Escalation
Now as a real user I could run sudo -l and see if I can run something as root. And as it turns out I could run /opt/NewComponent/feedback.sh
A custom script that I had permission to run as root without the use of a password. The script was vulnerable because it used the eval command
without any proper input sanitization,  the eval function is able to execute any command on linux, so to exploit this I can just add the deku user
to the sudoers file with:
```deku ALL=NOPASSWD: ALL >> /etc/sudoers```
And now run sudo -l again and you can use sudo for everything now without the use of the password.