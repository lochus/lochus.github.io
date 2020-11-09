---
title: "NahamCTF: Localghost"
categories:
  - NahamCTF
tags:
  - NahamCTF
---

*BooOooOooOOoo! This spooOoOooky client-side cooOoOode sure is scary! What spoOoOoOoky secrets does he have in stooOoOoOore??*

*Connect here:*
*http://jh2i.com:50003*

*Note, this flag is not in the usual format.*

---

Visiting this page gives us pictures of ghosts. We can open "Inspect element" by right clicking on the page, go to the debugger, look at jquery.jscroll2.js and click pretty print at the bottom to find the flag.

It's base64 encoded, so we'll just decode it:

```
echo SkNURntzcG9vb29va3lfZ2hvc3RzX2luX3N0b3JhZ2V9 | base64 -d
```

**JCTF{spoooooky_ghosts_in_storage}**
