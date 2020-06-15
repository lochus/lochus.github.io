---
title: "NahamCTF: Lucky"
categories:
  - NahamCTF
tags:
  - NahamCTF
---

"Oooh, an encrypted filesystem, lucky you!

Download the file below."

For this challenge, we're given a LUKS encrypted image. To bruteforce the password needed to decrypt it, we'll grab bruteforce-luks from the following Github repo:

https://github.com/glv2/bruteforce-luks

Once we've built the binary, we can use it to bruteforce the password: 

```
./bruteforce-luks lucky.img -v 10 -f /usr/share/wordlists/rockyou.txt

Password found: iloveyou
```

Now that we have the password, we can decrypt the image:

```
sudo modprobe dm-crypt
sudo cryptsetup open --type luks lucky.img luck
Password: iloveyou
```

Now that it's decrypted, we can mount it and grab the flag:

```
sudo mount /dev/mapper/luck /mnt
```

In mount, we find our flag.txt:

**flag{lucky_it_was_an_easy_password}**
