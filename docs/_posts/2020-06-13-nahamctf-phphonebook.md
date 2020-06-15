---
title: "NahamCTF: Phphonebook"
categories:
  - NahamCTF
tags:
  - NahamCTF
---

When we visit the website, we get this message:

*Sorry! You are in /index.php/?file=*

*The phonebook is located at phphonebook.php*

Visiting http://jh2i.com:50002/?file=phphonebook.php gives us a blank box that says "Enter Number: " and a submit button. Submitting numbers doesn't seem to work, but we can check the source using LFI in that ?file parameter:

http://jh2i.com:50002/?file=php://filter/convert.base64-encode/resource=phphonebook.php

This gives us the source, and shows us an interesting section:

``` 
<?php
  extract($_POST);

    if (isset($emergency)){
            echo(file_get_contents("/flag.txt"));
    }
?>
```

So, if we catch the request in Burp, add an &emergency=true parameter to the POST request, and send it, we get the flag:

**flag{phon3_numb3r_3xtr4ct3d}**
