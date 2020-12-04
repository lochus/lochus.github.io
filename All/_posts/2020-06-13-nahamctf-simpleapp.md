---
title: "NahamCTF: Simple App"
categories:
  - NahamCTF
tags:
  - NahamCTF
---

*Here's a simple Android app. Can you get the flag?*

*Download the file below.*

---

For this challenge, we're given a file called "simple-app.apk".

This challenge can be completed by unpacking the apk with apktool, and grepping through it for "flag{":

```
apktool d simple-app.apk

cd simple-app
for i in $(grep -lr flag);do cat $i|grep flag{;done
    const-string v0, "flag{3asY_4ndr0id_r3vers1ng}"
```

**flag{3asY_4ndr0id_r3vers1ng}**
