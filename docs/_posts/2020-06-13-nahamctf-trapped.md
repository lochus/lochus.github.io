---
title: "NahamCTF: Trapped"
categories:
  - NahamCTF  
tags:
  - NahamCTF  
---

*Help! I'm trapped!*

*Connect here:*
*nc jh2i.com 50019*

---

For this challenge, we're given a bash shell without job control when we connect to the netcat listener. When running any command, we receive the following message:

*You're stuck in the trap!*

In order to escape the trap and run a command, we'll have to set up a trap that executes a specific command when it encounters an event. We'll do this for the event "exit":

```
trap 'cat flag.txt' err exit
trap 'cat flag.txt' err exit
user@host:/home/user$ exit
exit
You're stuck in the trap!
flag{you_activated_my_trap_card}
```

**flag{you_activated_my_trap_card}**
