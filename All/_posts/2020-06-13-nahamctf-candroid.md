---
title: "NahamCTF: Candroid"
categories:
  - NahamCTF
tags:
  - NahamCTF
---

*I think I can, I think I can!*

*Download the file below.*

---

For this challenge we're given an apkfile called candroid.apk. We'll unpack this file using apktool:

```
apktool d candroid.apk
```

Now, we'll cd into the candroid folder that's created, and grep for flag{:

```
cd candroid

for i in $(grep -lr flag);do cat $i|grep flag{;done
    <string name="flag">flag{4ndr0id_1s_3asy}</string>
```

**flag{4ndr0id_1s_3asy}**
