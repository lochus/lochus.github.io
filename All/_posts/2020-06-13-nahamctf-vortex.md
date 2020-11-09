---
title: "NahamCTF: Vortex"
categories:
  - NahamCTF
tags:
  - NahamCTF
---

*Will you find the flag, or get lost in the vortex?*

*Connect here:*
*nc jh2i.com 50017*

---

For this challenge, we just need to connect to the server, redirect the output to a file, and then run strings on it:

```
nc jh2i.com 50017 >> out
strings out | grep flag
```

**flag{more_text_in_the_vortex}**
