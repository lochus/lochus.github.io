---
title: "NahamCTF: Fake File"
categories:
  - NahamCTF
tags:
  - NahamCTF
---

"Wait... where is the flag?

Connect here:
nc jh2i.com 50026"

For this challenge, we're given a shell when we connect to the netcat listener. There's nothing obviously out of place, aside from an extra .. directory in /home/user/.

We can see if there's a flag in it by using a for loop and grepping for flag{:

```
user@host:/home/user$ for i in $(grep -lr flag);do cat $i |grep flag{;done

flag{we_should_have_been_worried_about_u2k_not_y2k}
```

Voila, we have our flag!

**flag{we_should_have_been_worried_about_u2k_not_y2k}**
