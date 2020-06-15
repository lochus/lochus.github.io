---
title: "NahamCTF: Rotten"
categories:
  - NahamCTF
tags:
  - NahamCTF
---

"Ick, this salad doesn't taste too good!

Connect with:
nc jh2i.com 50034"

For this challenge, when we connect to the nc server, we get the following line:

*send back this line exactly. no flag here, just filler.*

If we copy the line and send it back, we gt another line like the following:

*jveu srtb kyzj czev vortkcp. tyrirtkvi 17 fw kyv wcrx zj '_'*

This is encoded with a caesar cipher, which is easily breakable and decodes to:

*send back this	line exactly. character	17 of the flag	is '_'*

While we could do this by hand, it would be a pain, so we'll write a python script to connect to the listener, receive input, decrypt it, and send it back:

cipher.py:
```
#!/usr/bin/env python3
import sys, socket

HOST = 'jh2i.com'  # The server's hostname or IP address
PORT = 50034        # The port used by the server

key = 'abcdefghijklmnopqrstuvwxyz'
sendback = ''
output = open('output.csv','w')

def decrypt(n, ciphertext):
    """Decrypt the string and return the plaintext"""
    result = ''

    for l in ciphertext:
        try:
            i = (key.index(l) - n) % 26
            result += key[i]
        except ValueError:
            result += l

    return result

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
    s.connect((HOST, PORT))
    
    s.sendall(b'')
    data = s.recv(1024)    
    print('Received: ', repr(data))
    s.sendall(b'send back this line exactly. no flag here, just filler.')
    while data:
        data = s.recv(1024)
        print('Received: ', repr(data))
        data2 = str(data)
        data3 = data2[2:-3]
        for offset in range(len(key)):
            decrypted = decrypt(offset, data3)
            if "send" in decrypted:
                output.write(decrypted+'\n')
                sendback = str.encode(decrypted)
                s.sendall(sendback)
```

This gives us a CSV file that we can sort through, grab unique lines from, open in LibreOffice Calc, sort by column, and get our flag:

```
sort < output.csv | uniq > unique.csv
```

In LibreOffice calc, we can use the space separator option, then sort by Column G. Once we have it sorted, we can copy column G to a new file and remove the apostrophes and newlines to get our flag:

```
tr -d "'" < final.txt | tr -d "\n"

flag{now_you_know_your_caesars}
```

**flag{now_you_know_your_caesars}**
