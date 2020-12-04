---
title: "NahamCTF: Time Keeper"
categories:
  - NahamCTF
tags:
  - NahamCTF
---

*There is some interesting stuff on this website. Or at least, I thought there was...*

*Connect here:*
*https://apporima.com/*

---

Visiting this website doesn't show us anything interesting, but if we take a look at the hint it's giving us, we see that we need to look at an older version of the website. 

We'll use wayback machine for this:

https://web.archive.org/web/20200418213402/https://apporima.com/

On this page, we see the following message:

*Today, I created my first CTF challenge. The flag can be found at forward slash flag dot txt.*

Visiting that url gives us the flag:

https://web.archive.org/web/20200418213402/https://apporima.com/flag.txt

**JCTF{the_wayback_machine}**
