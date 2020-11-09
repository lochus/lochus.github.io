---
title: "HackPack: Quote of the Day"
categories:
  - HackPackCTF
tags:
  - HackPackCTF
---


We are given a client to connect to cha.hackpack.club:41709. Loading the client up gives us a simple user interface to work with:

```
./client cha.hackpack.club 41709
server: qm9kd:v3.1.337
connected; available commands: 
        [e]cho word: example 'e hello\n'
        [q]uote: example 'q\n'
```

We can run some simple commands, and see what it prints out:

```
e hello
echo: hello

q
quote: These are not the quotes you're looking for...
```

Messing around with the input doesn't really give us anything interesting. If we pull the binary apart with ghidra, we see the following line in the main function (we can also pull it apart with r2, just depends on which you want to use):

```
          if ((char)local_80 == '~') {
            do_debug_test(local_24);
          }
          else {
            if ('~' < (char)local_80) goto LAB_080498b9;
            if ((char)local_80 == 'q') {
LAB_08049899:
              do_quote(local_24);
            }
            else {
              if ((char)local_80 < 'r') {
                if ((char)local_80 != 'e') {
                  if ((char)local_80 < 'f') {
                    if ((char)local_80 == 'E') goto LAB_08049882;
                    if ((char)local_80 == 'Q') goto LAB_08049899;
                  }
                  goto LAB_080498b9;
                }
LAB_08049882:
```

We see some familiar characters in there, like q, e, Q and E. However, ~ looks interesting, as it refers to a do_debug_test function that wasn't on the user interface.

Entering ~ into the client gives us:

```
~
result(99): badload
try:old.qotd,default.qotd
```

While this doesn't really give us anything we can use yet, it does look like this debug function might have some use.

Looking further into the do_debug_test function in Ghidra, we can see the following line:

```
send_message(param_1,0x2a,&DAT_0804a051,4);
```

It's not very clear what these mean yet, but we'll come back to that as well. We do know that the hex 0804a051 decodes to the word "test".

We can also see what echo and quote send as well:

echo:
```
sVar1 = strlen(param_2)
send_message(param_1, 10, param_2, sVar1)
```

quote:
```
send_message(param_1, 0x14, 0, 0)
```

To find out exactly what these actually mean, we'll run strace on the binary. First, we'll run "e hello":

```
"e hello\n", 1024)              = 8
send(3, "\0\0\0\n\0\0\0\6", 8, MSG_NOSIGNAL) = 8
send(3, "hello\n", 6, MSG_NOSIGNAL)     = 6
recv(3, "\0\0\0\v\0\0\0\6", 8, 0)       = 8
recv(3, "hello\n", 6, 0)                = 6
write(1, "echo: hello\n", 12echo: hello
)           = 12
write(1, "\n", 1
)                       = 1
```

We can see that we're sending a series of bytes first (\0\0\0\n\0\0\0\6) then sending our string to echo, followed by a newline character (\n). Looking at the code we grabbed for echo from ghidra, we can see that param_2 is most likely our string (hello), and sVar1 is the length. We can see the length in the first line:

\0\0\0\n\0\0\0\*6*

Next, we'll see what quote is doing:

```
read(0, q
"q\n", 1024)                    = 2
send(3, "\0\0\0\24\0\0\0\0", 8, MSG_NOSIGNAL) = 8
recv(3, "\0\0\0\25\0\0\0.", 8, 0)       = 8
recv(3, "These are not the quotes you're "..., 46, 0) = 46
write(1, "quote: These are not the quotes "..., 54quote: These are not the quotes you're looking for...
) = 54
```

Quote is passing the bytes (\0\0\0\24\0\0\0\0) to the server, and that's all. Looking at the code we grabbed from ghidra, we can see that there is no param_2 that is being passed like there was with echo, which makes sense, as the client doesn't need any user input to grab a quote from the server.

Finally, we'll look at what the ~ debug character does:

```
read(0, ~
"~\n", 1024)                    = 2
send(3, "\0\0\0*\0\0\0\4", 8, MSG_NOSIGNAL) = 8
send(3, "test", 4, MSG_NOSIGNAL)        = 4
recv(3, "\0\0\0c\0\0\0!", 8, 0)         = 8
recv(3, "badload\ntry:old.qotd,default.qot"..., 33, 0) = 33
write(1, "result(99): badload\n", 20result(99): badload
)   = 20
write(1, "try:old.qotd,default.qotd\n", 26try:old.qotd,default.qotd
) = 26
```

The debug function is passing a series of bytes to the server (\0\0\0*\0\0\0\4), followed by the word test (remember the &DAT_0804a051 variable that's being passed to send_message in the debug function from ghidra?). It looks like the param_2 that would be sent if we were calling the echo function is instead overwritten by the string 'test'. In order to get around this, we'll write a python script:

```
#!/usr/bin/env python3

import socket, sys

HOST = '152.14.93.208'  # The server's hostname or IP address
PORT = 41709        # The port used by the server

userinput = sys.argv[1]
userinput_enc = str.encode(userinput)


with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.connect((HOST, PORT))
    
    s.sendall(b'\0\0\0*\0\0\0*')
    s.sendall(userinput_enc)
    data = s.recv(1024)
    
    print('Received: ', repr(data))
    
```

Running this script gives us the following output:

```
python3 clientscript.py test

Received:  b'\x00\x00\x00\x01\x00\x00\x00\x0eqm9kd:v3.1.337\x00\x00\x00c\x00\x00\x00!badload\ntry:old.qotd,default.qotd'
```

Looks like the original client, which is a good sign. Now, using the information this gives us, we'll try sending the strings old and default to the server:

```
python3 clientscript.py old

Received:  b'\x00\x00\x00\x01\x00\x00\x00\x0eqm9kd:v3.1.337\x00\x00\x00+\x00\x00\x00\tcha-ching'

python3 clientscript.py default

Received:  b'\x00\x00\x00\x01\x00\x00\x00\x0eqm9kd:v3.1.337\x00\x00\x00+\x00\x00\x00\tcha-ching'
```

Looks a bit different from the original client! That cha-ching is a pretty good sign, as it means we're on the right track. Considering the strings we're sending, and the old.qotd/default.qotd string that's returned to us, we can infer that sending these strings to the server is loading a different version of the qotd program.

Since the client probably uses the default version, we'll try the old version, and try loading a quote:

```
#!/usr/bin/env python3

import socket, sys

HOST = '152.14.93.208'  # The server's hostname or IP address
PORT = 41709        # The port used by the server

userinput = sys.argv[1]
userinput_enc = str.encode(userinput)


with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.connect((HOST, PORT))
    
    s.sendall(b'\0\0\0*\0\0\0*')
    s.sendall(userinput_enc)
    data = s.recv(1024)
    
    print('Received: ', repr(data))
    
    s.sendall(b'\0\0\0\24\0\0\0\0')
    data2 = s.recv(1024)
    print('Quote: ', repr(data2))
```

We'll run it and see what we get:

```
python3 clientscript.py

Received:  b'\x00\x00\x00\x01\x00\x00\x00\x0eqm9kd:v3.1.337\x00\x00\x00+\x00\x00\x00\tcha-ching'
Quote:  b"\x00\x00\x00\x15\x00\x00\x00)These are the quotes you're looking for!\n"
```

Nice! Unlike the first quote with the client, this one says these "are" the quotes we're looking for! Now, to cycle through the quotes, we'll set up a for loop:


```
#!/usr/bin/env python3

import socket, sys

HOST = '152.14.93.208'  # The server's hostname or IP address
PORT = 41709        # The port used by the server

userinput = sys.argv[1]
userinput_enc = str.encode(userinput)


with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.connect((HOST, PORT))
    
    s.sendall(b'\0\0\0*\0\0\0*')
    s.sendall(userinput_enc)
    data = s.recv(1024)
    
    print('Received: ', repr(data))
    
    for i in range(16):
        s.sendall(b'\0\0\0\24\0\0\0\0')
        data2 = s.recv(1024)
        print('Quote ' + str(i) + ': ', repr(data2))
```

Finally, we'll run it one last time and see what we get:

```
Received:  b'\x00\x00\x00\x01\x00\x00\x00\x0eqm9kd:v3.1.337\x00\x00\x00+\x00\x00\x00\tcha-ching'
Quote 0:  b"\x00\x00\x00\x15\x00\x00\x00)These are the quotes you're looking for!\n"
Quote 1:  b"\x00\x00\x00\x15\x00\x00\x00\x9eDon't criticize a woman until you've walked a mile in her shoes.  Unless she's wearing heels--then you should just cut your losses and not critize her, ever.\n"
Quote 2:  b"\x00\x00\x00\x15\x00\x00\x00/No, really, you're looking in the right place!\n"
Quote 3:  b'\x00\x00\x00\x15\x00\x00\x00lA programmer has a problem.  He thinks: "I know, I\'ll use regular expressions!"  Now he has two problems...\n'
Quote 4:  b'\x00\x00\x00\x15\x00\x00\x00\x12Getting warmer...\n'
Quote 5:  b'\x00\x00\x00\x15\x00\x00\x00YA computer without FORTRAN and COBOL is like chocolate cake without ketchup and mustard.\n'
Quote 6:  b'\x00\x00\x00\x15\x00\x00\x008This program is not written in INTERCAL.  For a reason.\n'
Quote 7:  b'\x00\x00\x00\x15\x00\x00\x00\x15You are SOOOO close!\n'
Quote 8:  b'\x00\x00\x00\x15\x00\x00\x004This program is not written in Java.  For a reason.\n'
Quote 9:  b'\x00\x00\x00\x15\x00\x00\x00<This program is not written in Visual Basic.  For a reason.\n'
Quote 10:  b'\x00\x00\x00\x15\x00\x00\x008Oops, you missed it!  (Kidding, kidding, keep going...)\n'
Quote 11:  b'\x00\x00\x00\x15\x00\x00\x00\x9eVisual Basic.NET bore so little resemblance to Visual Basic "classic" that it was suggested Microsoft should have named it "Visual Fred" to reduce confusion.\n'
Quote 12:  b'\x00\x00\x00\x15\x00\x00\x00\x80It was the best of times, it was the worst of times.  Unless it was YOU on the guillotine--then it was just the worst of times.\n'
Quote 13:  b'\x00\x00\x00\x15\x00\x00\x00nA tisket a tasket, a green and yellow basket, I something-something-something, on my way I forgot the rest...\n'
Quote 14:  b'\x00\x00\x00\x15\x00\x00\x00 flag{h3r3s_y3r_pr1z3_4_pl@y1ng}\n'
Quote 15:  b'\x00\x00\x00\x15\x00\x00\x00\x01\n'
```

There's our flag!

**flag{h3r3s_y3r_pr1z3_4_pl@y1ng}**
