---
title: "HackPack: Paster"
categories:
  - HackPackCTF
tags:
  - HackPackCTF
---


Visiting https://paster.cha.hackpack.club gives us a text box to enter input into. All input is truncated, and returned to the screen, giving us a pretty good idea that this'll be a XSS vulnerability. Trying a variable of difficult payloads (<script src=, <iframe src=, <script>alert('123') etc) shows that these are all too long, and are truncated.

However, we can use a super short payload to trigger an alert:

```
<svg/onload=alert(1)>
```

Using this payload, we get an alert with the flag!

**flag{x55_i5Nt_7hA7_bAD_R1Gh7?}**
