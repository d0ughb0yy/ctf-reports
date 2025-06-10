# Anthem

**Hostname:**
`WIN-LU09299160F`

**OS:**
`Windows Server`

**Users:**
`JD@anthem.com`
`SG@anthem.com`

**Credentials:**

Umbraco admin: `SG@anthem.com:UmbracoIsTheBest!`

Machine:
`SG:UmbracoIsTheBest!`
`Administrator:ChangeMeBaby1MoreTime`

## Port Scan:

```
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: WIN-LU09299160F
|   NetBIOS_Domain_Name: WIN-LU09299160F
|   NetBIOS_Computer_Name: WIN-LU09299160F
|   DNS_Domain_Name: WIN-LU09299160F
|   DNS_Computer_Name: WIN-LU09299160F
|   Product_Version: 10.0.17763
|_  System_Time: 2025-06-10T13:18:26+00:00
| ssl-cert: Subject: commonName=WIN-LU09299160F
| Not valid before: 2025-06-09T13:12:52
|_Not valid after:  2025-12-09T13:12:52
|_ssl-date: 2025-06-10T13:19:31+00:00; -1s from scanner time.
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

## Recon:

### Web Recon:

**robots.txt:**
```
UmbracoIsTheBest!

# Use for all search robots
User-agent: *

# Define the directories not to crawl
Disallow: /bin/
Disallow: /config/
Disallow: /umbraco/
Disallow: /umbraco_client/

```

## Foothold / Privilege Escalation:

### Initial Foothold

I was able to get initial access first due to an exposed password in robots.txt then credential reusage. Umbraco admin used the same password as for his server credentials and I was able to RDP in.

A post with a poem hinted to the name of the admin 'Solomon Grundy' and another post leaked the email of another user JD@anthem.com which indicates to email templating of the company leading to an admin email of SG@anthem.com

---

### Privilege Escalation:

I was able to get Administrator due to misconfigured backup folder which allowed access to anyone. I checked the file inside and found the Administrator password but first I had to give myself permissions for the file.


## Journal:

**When connecting with RDP to a machine remember to enable viewing of hidden files**