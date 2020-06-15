---
title: "DefenitCTF: MoM's Touch"
categories:
  - DefenitCTF
tags:
  - DefenitCTF
---

We're given a binary called momsTouch for this challenge. When we run this binary, we get the following message:

```
Mom Give Me The FLAG!
```

When we enter data, we get a message about the length:

```
Mom Give Me The FLAG!
test
Mom, Check the legnth..
```

We can fuzz to see how many characters we need:

```
for i in $(seq 0 100);do echo $i; python -c "print 'A'*$i" | ./momsTouch;done
```

At 73 characters, we get the following message:

```
Try Again..
```

We'll open up the binary in radare2 and step through the functions to see exactly what the binary is doing:

```
r2 -d ./momsTouch
aaa
afl

0x08048590    1 33           entry0
0x08048550    1 6            sym.imp.__libc_start_main
0x080484b0    1 6            sym.imp.read
0x080484c0    1 6            sym.imp.free
0x080484d0    1 6            sym.imp.signal
0x080484e0    1 6            sym.imp.alarm
0x080484f0    1 6            sym.imp.perror
0x08048500    1 6            sym.imp.malloc
0x08048510    1 6            sym.imp.puts
0x08048520    1 6            sym.imp.exit
0x08048530    1 6            sym.imp.srand
0x08048540    1 6            sym.imp.strlen
0x08048560    1 6            sym.imp.setvbuf
0x08048570    1 6            sym.imp.rand
0x08048660    8 139  -> 93   entry.init0
0x08048730    3 110          entry.init1
0x08048640    3 30           entry.fini0
0x080485d0    4 43           fcn.080485d0
0x08048840   13 200          main
```

In the main function, we see that sym.imp.strlen is being called to check the length of the user input, which we already know needs to be 73. 

From here, an unnamed function (0x80487a0) is called. At this address, we see the a bunch of assembly, but following comparison is interesting:

```
cmp ecx, dword [esi*4 + 0x8049144]
jne 0x8048825
```

The reason that this is so interesting is that if that comparison fails, that jne command jumps to 0x8048825, which queues up the "Try Again.." message to print out. Each time the character queued into ecx is correct, the program loops, queues the next character and checks it, until the string is either finished, or a wrong character is finished.

Viewing the text at that location with "px @esi*4+0x8049144" doesn't help us at all, but we can be sneaky and change the jne command to a je command, so that the process only stops when a correct character is found, thereby allowing us to bruteforce the flag one character at a time, with all of the other characters being incorrect:

```
wa je 0x8048825 @0x08048819
```

We first need a file named "fuzz" that contains all characters [a-zA-Z0-9] and the characters { and }.

Now, we'll run the following fuzzer:

fuzz.sh
```
#!/bin/bash


# Bruteforce Section
for n in $(seq 0 72)
        do for i in $(cat fuzz)
                do ONE=$(python -c "print 'A'*$n") 
                TWO=$(python -c "print 'A'*(72-$n)")
                echo $i >> output.txt
                echo $ONE$i$TWO | ./back >> output.txt
        done
done

# Clean up the output
cat output.txt | grep -v Correct |grep -v FLAG |grep Try -B 1 |grep -v Try |grep -v '\--' >> output2.txt
tr -d '\n' < output2.txt > flag.txt

# Print the flag
cat flag.txt

# Housekeeping
rm output.txt
rm output2.txt
```

Or, if we're so inclined, a one liner:

```
for n in $(seq 0 72);do for i in $(cat fuzz);do ONE=$(python -c "print 'A'*$n"); TWO=$(python -c "print 'A'*(72-$n)");echo $i >> output.txt; echo $ONE$i$TWO | ./back >> output.txt ;done;done; cat output.txt | grep -v Correct | grep -v FLAG |grep Try -B 1 |grep -v Try |grep -v '\--' >> output2.txt; tr -d '\n' < output2.txt > flag.txt;rm output.txt; rm output2.txt; cat flag.txt
```

We get:
```
Definit{ea40d42bfaf7d1f599abf284a35c535c607ccadbff38f7c39d6d57e238c4425eea40d42bfaf7d1f599abf284a35c535c607ccadbff38f7c39d6d57e238c4425e}
```

Voila, we have the flag!

**Defenit{ea40d42bfaf7d1f599abf284a35c535c607ccadbff38f7c39d6d57e238c4425eea40d42bfaf7d1f599abf284a35c535c607ccadbff38f7c39d6d57e238c4425e}**

