---
title: "NahamCTF: Microsooft"
categories:
  - NahamCTF
tags:
  - NahamCTF
---

*We have to use Microsoft Word at the office!? Oof...*

*Download the file below.*

---

For this challenge, we're given a .docx file. We'll download the file, unzip it, and run grep -lr on it to see if we can find the flag:

```
unzip Microsooft.docx
grep -lr flag
*src/oof,txt*

cat src/oof.txt
```

**flag{oof_is_right_why_gfxdata_though}**
