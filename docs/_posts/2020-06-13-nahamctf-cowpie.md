---
title: "NahamCTF: Cow Pie"
categories:
  - NahamCTF
tags:
  - NahamCTF
---

*Ew. Some cow left this for us. It's gross... but something doesn't seem right...*

*Download the file below.*

---

This is another really easy one. We are given a QEMU image for this challenge named manure. If we run strings on it and grep for flag, we get the flag instantly:

```
strings manure | grep flag
```

**flag{this_flag_says_mooo_what_say_you}**
