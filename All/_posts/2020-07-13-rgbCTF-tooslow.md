---                                                                                                                                                                                                                                        
title: "rgbCTF: Too Slow"                                                                             
categories:                                                                                                          
  - rgbCTF                                                                                                           
tags:                                                                                                                                                                                                                                      
  - rgbCTF                                                                                                                                                                                                                                 
--- 

*I've made this flag decryptor! It's super secure, but it runs a little slow.*


*~ungato#3536*

---

For this challenge, we're given a binary that supposedly generates a key and decrypts a flag. This was a rather simple challenge once we actually pull the binary apart, since it creates a loop that causes the getKey function to take a LONG time. By removing the loop, we're able to easily grab the flag.

## Pulling the Binary Apart

To disassemble the binary, we'll use radare2.

```
$ r2 -d a.out
Process with PID 163890 started...
= attach 163890 163890
bin.baddr 0x555555554000
Using 0x555555554000
asm.bits 64
Warning: r_bin_file_hash: file exceeds bin.hashlimit
[0x7ffff7fd5090]> aaaa
[Cannot find function at 0x5555555550a0. and entry0 (aa)
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze function calls (aac)
[x] Analyze len bytes of instructions for references (aar)
[x] Check for objc references
[x] Check for vtables
[TOFIX: aaft can't run in debugger mode.ions (aaft)
[x] Type matching analysis for all functions (aaft)
[x] Propagate noreturn information
[x] Use -AA or aaaa to perform additional experimental analysis.
[x] Finding function preludes
[x] Enable constraint types analysis for variables
[0x7ffff7fd5090]> afl
0x5555555550d0    4 41   -> 34   sym.deregister_tm_clones
0x555555555100    4 57   -> 51   sym.register_tm_clones
0x555555554000    3 126  -> 181  loc.imp._ITM_deregisterTMCloneTable
```

This doesn't give us much information, so we'll start the program:

```
[0x7ffff7fd5090]> dc
Flag Decryptor v1.0
Generating key...
```

If we CTRL+C now, we can see where we're at in the execution by entering visual mode:

```
[0x55555555529b]> vv
...
        ╎╎   ;-- rip:                                                                                                                                                                                                                    │
│        ╎╎   0x55555555528d      c1ea1f         shr edx, 0x1f                                                                                                                                                                            │
│        ╎╎   0x555555555290      01d0           add eax, edx                                                                                                                                                                             │
│        ╎╎   0x555555555292      d1f8           sar eax, 1                                                                                                                                                                               │
│        ╎╎   0x555555555294      8945fc         mov dword [rbp - 4], eax                                                                                                                                                                 │
│        ╎╎   0x555555555297      837dfc01       cmp dword [rbp - 4], 1                                                                                                                                                                   │
│        └──< 0x55555555529b      75d3           jne 0x555555555270 
```

This is the interesting part that creates the loop. If we look at the register it's comparing to 1, we see the following:

```
:> px @rbp-4
- offset -       0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF
0x7fffffffdf2c  0000 0000 50df ffff ff7f 0000 e452 5555  ....P........RUU
...
```

If we let the program continue its execution again and wait for a few seconds, we can pause execution again with CTRL+C and check the register again:

```
:> px @rbp-4
- offset -       0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF
0x7fffffffdf2c  0000 0000 50df ffff ff7f 0000 e452 5555  ....P........RUU
...
```

This register doesn't seem to be changing whatsoever, which means it's either going to take a long time to be equal to 1, or it will never equal one and this is an infinite loop. To bypass this loop and make sure that it isn't hit, since we know that rbp-4 isn't going to equal 1 for some time, we can patch the binary and change the following line:

*0x55555555529b      75d3           jne 0x555555555270*
to
*0x55555555529b      75d3           je 0x555555555270*

To do this, we'll use radare2's **wa** command:

```
:> wa je 0x555555555270 @0x55555555529b
Written 2 byte(s) (je 0x555555555270) = wx 74d3
```

Now, if we go back into visual mode and take a look at that line, we'll see:

```
│        ╎╎   0x555555555294      8945fc         mov dword [rbp - 4], eax                                                                                                                                                                 │
│        ╎╎   0x555555555297      837dfc01       cmp dword [rbp - 4], 1                                                                                                                                                                   │
│        └──< 0x55555555529b      74d3           je 0x555555555270
```

We've now bypassed the loop, as it will only jump to 0x555555555270 if rbp-4 is equal to 1!

Now, we can continue the execution of the program and get our flag:

```
:> dc
Your flag: rgbCTF{pr3d1ct4bl3_k3y_n33d5_no_w41t_cab79d}
```

**rgbCTF{pr3d1ct4bl3_k3y_n33d5_no_w41t_cab79d}**

