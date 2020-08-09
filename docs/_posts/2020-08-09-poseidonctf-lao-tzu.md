---
title: "PoseidonCTF: The Large Cherries"
categories:
  - PoseidonCTF
tags:
  - PoseidonCTF
---

# The Large Cherries

In a time, where only bad people with large cherries existed. Lao-Tzu said 'the five colours make a man blind, the five tones make a man deaf, because if you can only see five colours, you’re blind, and if you can only hear five tones in music, you’re deaf
nc poseidonchalls.westeurope.cloudapp.azure.com 9003
Author : t0m7r00z

---

## TLDR: 
The binary reads a flag from /home/ctf/magic_word.txt, and compares it to the user input (if certain conditions are met). Not meeting these conditions causes the input to be set to a default value, which will be incorrect on the server side. If the conditions are met, a second input is read, then compared to the first, but the binary won't accept the same eight character string as the first input, so we trick the binary into accepting 9 characters (since strncmp is only reading 8 characters) and grab our flag.

---

## Reversing Lao-Tzu

For this challenge, we're given a binary called "Lao-Tzu" that we can pull apart. We'll be using r2 and Ghidra to analyze this binary.

We can start by opening the binary with r2 in debug mode:

```
$ r2 -d Lao-Tzu
[0x7feff4253090]> aaa
[0x7feff4253090]> pdf @main
...
│           0x55fdb759ebb5      e876feffff     call sym.imp.ptrace     ; long ptrace(__ptrace_request request, pid_t pid, void*addr, void*data)                                               
│           0x55fdb759ebba      8945b4         mov dword [var_4ch], eax                                                                                                                       
│           0x55fdb759ebbd      837db400       cmp dword [var_4ch], 0                                                                                                                         
│       ┌─< 0x55fdb759ebc1      7916           jns 0x55fdb759ebd9            
│       │   0x55fdb759ebc3      488d3d3e0400.  lea rdi, qword str.No_tracing ; 0x55fdb759f008 ; "No tracing ;)"
```

In the above disassembled section, we can see that ptrace is being called, which gives us the error "No tracing ;)" if we're running it in debug mode. We can fix this rather easily by patching the binary, but we'll need to reopen it in write mode:

```
$ r2 -d -w Lao-Tzu
```

The following line is causing the program to print out "No tracing ;)" and exit if we're running in debug mode, so we can change the conditional to cause it to continue execution:

```
│       ┌─< 0x5587db804bc1      7916           jns 0x5587db804bd9            
```

We'll copy the binary to a new file, patch it with r2, and run it again:

```
$ cp Lao-Tzu trace-Tzu
$ r2 -w trace-Tzu
[0x00000a80]> pdf @main
...
│       ┌─< 0x00000bc1      7916           jns 0xbd9                                  ...       
[0x00000a80]> wa js 0xbd9 @0x00000bc1
Written 2 byte(s) (js 0xbd9) = wx 7816

[0x00000000]> ood
Cannot debug file (trace) with permissions set to 0x7.
Reopening the original file in read-only mode.
Process with PID 216834 started...
File dbg:///home/anti/Downloads/trace  reopened in read-only mode
= attach 216834 216834
```

Now that we've patched the binary, we can continue debugging it:

```
[0x7f3fffbc3090]> dc
Enter the secret for the magic word: asdf1234
```

We can see that the binary now continues its execution without exiting, but we don't know what the magic word is yet. If we restart the program, continue execution until we're prompted for the magic word, then pause execution, we can step through the assembly to see what's happening:

```
[0x7f6c5104e493]> ood
child received signal 9
Process with PID 216850 started...
= attach 216850 216850
File dbg:///home/anti/Downloads/trace  reopened in read-write mode
Unable to find filedescriptor 3
Unable to find filedescriptor 4
Unable to find filedescriptor 5
Unable to find filedescriptor 3
Unable to find filedescriptor 4
Unable to find filedescriptor 5
216850
[0x7f71a4713090]> dc
Enter the secret for the magic word:
CTRL+C
[0x7f4098ec05ce]> ds
asdf1234
[0x7f4098ec05ce]> 
```

If we enter visual mode with "v" we just see a lot of random instructions, but if we take another look at main, we find another interesting section of assembly that the execution will hit:

```
│       │   0x55f15af53cc0      4889c2         mov rdx, rax                                                                                                                          [41/1569]
│       │   0x55f15af53cc3      488d45ed       lea rax, qword [dest]                                                                                                                          
│       │   0x55f15af53cc7      4889d6         mov rsi, rdx            ; const char *src       
│       │   0x55f15af53cca      4889c7         mov rdi, rax            ; char *dest                                                                                                           
│       │   0x55f15af53ccd      e8befcffff     call sym.imp.strcpy     ; char *strcpy(char *dest, const char *src)                                                                            
│       │   0x55f15af53cd2      488d35690300.  lea rsi, qword [0x55f15af54042] ; "r" ; const char *mode                                                                                       
│       │   0x55f15af53cd9      488d3d640300.  lea rdi, qword [0x55f15af54044] ; "/home/ctf/magic_word.txt" ; const char *filename                                                            
│       │   0x55f15af53ce0      e86bfdffff     call sym.imp.fopen      ; file*fopen(const char *filename, const char *mode)                                                                   
│       │   0x55f15af53ce5      488945b8       mov qword [stream], rax                                                                                                                        
│       │   0x55f15af53ce9      488b55b8       mov rdx, qword [stream] ; FILE *stream                                                                                                         
│       │   0x55f15af53ced      488d45e2       lea rax, qword [s1]                             
│       │   0x55f15af53cf1      be09000000     mov esi, 9              ; int size              
│       │   0x55f15af53cf6      4889c7         mov rdi, rax            ; char *s               
│       │   0x55f15af53cf9      e802fdffff     call sym.imp.fgets      ; char *fgets(char *s, int size, FILE *stream)                                                                         
│       │   0x55f15af53cfe      488d4ded       lea rcx, qword [dest]                                                                                                                          
│       │   0x55f15af53d02      488d45e2       lea rax, qword [s1]
│       │   0x55f15af53d06      ba08000000     mov edx, 8              ; size_t n     
│       │   0x55f15af53d0b      4889ce         mov rsi, rcx            ; const char *s2
│       │   0x55f15af53d0e      4889c7         mov rdi, rax            ; const char *s1    
│       │   0x55f15af53d11      e86afcffff     call sym.imp.strncmp    ; int strncmp(const char *s1, const char *s2, size_t n)                                                                
│       │   0x55f15af53d16      85c0           test eax, eax                      
│       │   0x55f15af53d18      0f85b4000000   jne 0x55f15af53dd2                     
│      ┌──< 0x55f15af53d1e      488b45b8       mov rax, qword [stream]                                                                                                                        
│      ││   0x55f15af53d22      4889c7         mov rdi, rax            ; FILE *stream
│      ││   0x55f15af53d25      e886fcffff     call sym.imp.fclose     ; int fclose(FILE *stream)                                          
```

We'll set a breakpoint before the strcpy and strncmp functions to see what's being passed to the comparison:

```
:> db 0x55f15af53cca
:> db 0x55f15af53d0e
```

Once we hit our first breakpoint, we can examine the registers:

```
[0x55f15af53ccd]> drr
role reg     value            ref
―――――――――――――――――――――――――――――――――
R0   rax     7ffe55aa68bd      ([stack]) stack R W 0x0 -->  0
     rbx     0                 0
A3   rcx     0                 0
A2   rdx     55f15af5409c      (/home/anti/Downloads/trace) (.rodata) program R X 'outsb dx, byte [rsi]' 'trace' (nopassyo)
A4   r8      0                 0
A5   r9      25                37 ascii ('%')
     r10     9                 9
     r11     246               582
     r12     55f15af53a80      (/home/anti/Downloads/trace) (.text) entry0 program R X 'xor ebp, ebp' 'trace'
     r13     7ffe55aa69b0      ([stack]) stack R W 0x1 -->  1
     r14     0                 0
     r15     0                 0
A1   rsi     55f15af5409c      (/home/anti/Downloads/trace) (.rodata) program R X 'outsb dx, byte [rsi]' 'trace' (nopassyo)
A0   rdi     7ffe55aa68bd      ([stack]) stack R W 0x0 -->  0
SP   rsp     7ffe55aa6880      ([stack]) stack R W 0xffffffff55aa69c8 -->  -2857735736
BP   rbp     7ffe55aa68d0      ([stack]) stack R W 0x55f15af53f80 -->  (/home/anti/Downloads/trace) (.text) sym.__libc_csu_init sym.__libc_csu_init program R X 'push r15' 'trace'
PC   rip     55f15af53ccd      (/home/anti/Downloads/trace) (.text) main program R X 'call 0x55f15af53990' 'trace'
     cs      33                51 ascii ('3')
     rflags  212               530
SN   orax    ffffffffffffffff 
     ss      2b                43 ascii ('+')
     fs_base 7f4098fe1540      (unk2) R W 0x7f4098fe1540
     gs_base 0                 0
     ds      0                 0
     es      0                 0
     fs      0                 0
     gs      0                 0
```

We can see our input being passed in the rdi register, and the string "nopassyo" being passed in rdx and rsi. Nothing of much value happens here, and we're returned to the next function in main:

```
│ │           0x5582fb2eccd2      488d35690300.  lea rsi, qword [0x5582fb2ed042]    ; "r" ; const char *mode                                                                                 │
│ │           0x5582fb2eccd9      488d3d640300.  lea rdi, qword [0x5582fb2ed044]    ; "/home/ctf/magic_word.txt" ; const char *filename 
```

This is the interesting part, as we can see that /home/ctf/magic_word.txt is being called and the contents are being read in the assembly below:

```
│ │           0x5582fb2ecce0      e86bfdffff     call sym.imp.fopen      ;[1] ; file*fopen(const char *filename, const char *mode)                                                           │
│ │           0x5582fb2ecce5      488945b8       mov qword [stream], rax                                                                                                                     │
│ │           0x5582fb2ecce9      488b55b8       mov rdx, qword [stream]    ; FILE *stream                                                                                                   │
│ │           0x5582fb2ecced      488d45e2       lea rax, qword [s1]                                                                                                                         │
│ │           0x5582fb2eccf1      be09000000     mov esi, 9              ; int size                                                                                                          │
│ │           0x5582fb2eccf6      4889c7         mov rdi, rax            ; char *s                                                                                                           │
│ │           0x5582fb2eccf9      e802fdffff     call sym.imp.fgets      ;[2] ; char *fgets(char *s, int size, FILE *stream)                                                                 │
│ │           0x5582fb2eccfe      488d4ded       lea rcx, qword [dest]                                                                                                                       │
│ │           0x5582fb2ecd02      488d45e2       lea rax, qword [s1]                                                                                                                         │
│ │           0x5582fb2ecd06      ba08000000     mov edx, 8              ; size_t n                                                                                                          │
│ │           0x5582fb2ecd0b      4889ce         mov rsi, rcx            ; const char *s2                                                                                                    │
│ │           0x5582fb2ecd0e b    4889c7         mov rdi, rax            ; const char *s1  
```

Right after this function, two strings are compared. Since we've already set a breakpoint before this comparison, we'll continue execution:

```
:> dc
child stopped with signal 28
[+] SIGNAL 28 errno=0 addr=0x00000000 code=128 ret=0
child stopped with signal 11
[+] SIGNAL 11 errno=0 addr=0x00000000 code=1 ret=0
```

Instead of continuing, the binary crashed, since the file /home/ctf/magic_word.txt doesn't exist. We'll create this file with a random string in it and try again:

magic_word.txt:
```
12345678
```

Note: You may need to reset your breakpoint here.

```
[0x7f66d85e3090]> dc
Enter the secret for the magic word: asdf1234
hit breakpoint at: 557f6180bd0e
[0x557f6180bd0e]> drr
role reg     value            ref
―――――――――――――――――――――――――――――――――
R0   rax     7ffd823fb462      ([stack]) stack R W 0x3837363534333231 (12345678) -->  ascii ('1') sequence
     rbx     0                 0
A3   rcx     7ffd823fb46d      ([stack]) stack R W 0x6f79737361706f6e (nopassyo) -->  ascii ('n')
A2   rdx     8                 8
A4   r8      7ffd823fb462      ([stack]) stack R W 0x3837363534333231 (12345678) -->  ascii ('1') sequence
A5   r9      70                112 ascii ('p')
     r10     0                 0
     r11     7f66d8593840     
     r12     557f6180ba80      (/home/anti/Downloads/trace) (.text) entry0 program R X 'xor ebp, ebp' 'trace'
     r13     7ffd823fb560      ([stack]) stack R W 0x1 -->  1
     r14     0                 0
     r15     0                 0
A1   rsi     7ffd823fb46d      ([stack]) stack R W 0x6f79737361706f6e (nopassyo) -->  ascii ('n')
A0   rdi     557f62831380     
SP   rsp     7ffd823fb430      ([stack]) stack R W 0xffffffff823fb578 -->  -2109753992
BP   rbp     7ffd823fb480      ([stack]) stack R W 0x557f6180bf80 -->  (/home/anti/Downloads/trace) (.text) sym.__libc_csu_init sym.__libc_csu_init program R X 'push r15' 'trace'
PC   rip      557f6180bd0e       (/home/anti/Downloads/trace) (.text) main program R X 'mov rdi, rax' 'trace'
     cs      33                51 ascii ('3')
     rflags  202               514
SN   orax    ffffffffffffffff 
     ss      2b                43 ascii ('+')
     fs_base 7f66d8610540     
     gs_base 0                 0
     ds      0                 0
     es      0                 0
     fs      0                 0
     gs      0                 0
```

We can see the contents of our "magic_word.txt" being passed in rax and r8, which is then compared to the string "nopassyo" in the strncmp function. If we change the contents of our magic_word.txt file to "nopassyo" we're able to pass this step.

```
[0x557f6180bd0e]> dc
child stopped with signal 28
[+] SIGNAL 28 errno=0 addr=0x00000000 code=128 ret=0
hit breakpoint at: 557f6180bd0e
[0x557f6180bd0e]> ood
child received signal 9
...
[0x7f70c584b090]> dc
Enter the secret for the magic word: asdf1234
Enter the thing that's in my mind: 
```

Now, before entering anything, we'll look at where we are in main:

```
│      ││   0x56039c6a5d2a      488d3d2f0300.  lea rdi, qword str.Enter_the_thing_that_s_in_my_mind: ; 0x56039c6a6060 ; "Enter the thing that's in my mind: "
│      ││   0x56039c6a5d31      b800000000     mov eax, 0
│      ││   0x56039c6a5d36      e895fcffff     call sym.imp.printf     ; int printf(const char *format)
│      ││   0x56039c6a5d3b      488d45cc       lea rax, qword [var_34h]
│      ││   0x56039c6a5d3f      4889c6         mov rsi, rax
│      ││   0x56039c6a5d42      488d3d3b0300.  lea rdi, qword str.10s  ; 0x56039c6a6084 ; "%10s"
│      ││   0x56039c6a5d49      b800000000     mov eax, 0
│      ││   0x56039c6a5d4e      e80dfdffff     call sym.imp.__isoc99_scanf ; int scanf(const char *format)
│      ││   0x56039c6a5d53      488d55ed       lea rdx, qword [var_13h]
│      ││   0x56039c6a5d57      488d45cc       lea rax, qword [var_34h]
│      ││   0x56039c6a5d5b      4889d6         mov rsi, rdx
│      ││   0x56039c6a5d5e      4889c7         mov rdi, rax
│      ││   0x56039c6a5d61      e8c2010000     call sym.Strcmp
│      ││   0x56039c6a5d66      85c0           test eax, eax
│     ┌───< 0x56039c6a5d68      756f           jne 0x56039c6a5dd9
│     │││   0x56039c6a5d6a      488d55ed       lea rdx, qword [var_13h]
│     │││   0x56039c6a5d6e      488d45cc       lea rax, qword [var_34h]
│     │││   0x56039c6a5d72      4889d6         mov rsi, rdx
│     │││   0x56039c6a5d75      4889c7         mov rdi, rax
│     │││   0x56039c6a5d78      e893fcffff     call sym.imp.strcmp     ; int strcmp(const char *s1, const char *s2)
│     │││   0x56039c6a5d7d      85c0           test eax, eax
│    ┌────< 0x56039c6a5d7f      7558           jne 0x56039c6a5dd9
│    ││││   0x56039c6a5d81      488d35ba0200.  lea rsi, qword [0x56039c6a6042] ; "r"
│    ││││   0x56039c6a5d88      488d3dfa0200.  lea rdi, qword str.home_ctf_flag.txt ; 0x56039c6a6089 ; "/home/ctf/flag.txt"
```

We can see two strcmp functions, then the flag.txt being called, so we'll create a flag in /home/ctf/:

flag.txt:
```
flag{test}
```

Now, we'll try entering the same string in our magic_word.txt for the second input:

```
Enter the thing that's in my mind: nopassyo
flag{test}
[0x7f2fd495d146]> 
```

So, it works on our end, let's try it against the server!

```
$ nc poseidonchalls.westeurope.cloudapp.azure.com 9003
Enter the secret for the magic word: nopassyo
$
```

What worked on our end won't work against the server, meaning that the /home/ctf/magic_word.txt file probably either doesn't exist, or doesn't contain the string "nopassyo".

We need to find a way to bypass this function, so we'll open up the binary in Ghidra and see exactly what it's doing with our first input:

main:
```

undefined8 main(void)

{
  int iVar1;
  long lVar2;
  undefined8 uVar3;
  char *__src;
  FILE *__stream;
  long in_FS_OFFSET;
  char local_3c [11];
  undefined local_31 [11];
  char local_26 [11];
  char local_1b [11];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  lVar2 = ptrace(PTRACE_TRACEME,0,0);
  if ((int)lVar2 < 0) {
    setvbuf(stdout,(char *)0x0,2,0);
    setvbuf(stdin,(char *)0x0,2,0);
    setvbuf(stderr,(char *)0x0,2,0);
    memset(local_3c,0,0xb);
    memset(local_26,0,0xb);
    memset(local_31,0,0xb);
    memset(local_1b,0,0xb);
    printf("Enter the secret for the magic word: ");
    __isoc99_scanf(&DAT_0010103e,local_31);
    __src = (char *)magic_word(local_31);
    strcpy(local_1b,__src);
    __stream = fopen("/home/ctf/magic_word.txt","r");
    fgets(local_26,9,__stream);
    iVar1 = strncmp(local_26,local_1b,8);
    if (iVar1 == 0) {
      fclose(__stream);
      printf("Enter the thing that\'s in my mind: ");
      __isoc99_scanf(&DAT_00101084,local_3c);
      iVar1 = Strcmp(local_3c,local_1b,local_1b);
      if (iVar1 == 0) {
        iVar1 = strcmp(local_3c,local_1b);
        if (iVar1 == 0) {
          __stream = fopen("/home/ctf/flag.txt","r");
          while( true ) {
            iVar1 = feof(__stream);
            if (iVar1 != 0) break;
            iVar1 = fgetc(__stream);
            putchar((int)(char)iVar1);
          }
          fclose(__stream);
        }
      }
      uVar3 = 0;
    }
    else {
      uVar3 = 0x539;
    }
  }
  else {
    puts("No tracing ;)");
    uVar3 = 0x539;
  }
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return uVar3;
}
```

We can see our input being taken, then a magic_word function being run on it here:

```
    __isoc99_scanf(&DAT_0010103e,local_31);
    __src = (char *)magic_word(local_31);
```

Looking at the magic_word function makes it pretty clear why our input seems to be ignored:

```

char * magic_word(char *param_1)

{
  if ((int)param_1[3] + (int)*param_1 == 0xab) {
    if (param_1[3] == '7') {
      if ((byte)(param_1[2] ^ param_1[1]) == 0x5d) {
        if ((int)param_1[4] - (int)param_1[2] == 5) {
          if ((int)param_1[6] + (int)param_1[4] == 0xa2) {
            if (param_1[5] == param_1[6]) {
              if (param_1[6] == '0') {
                if (param_1[7] != 'z') {
                  param_1 = "nopassyo";
                }
              }
              else {
                param_1 = "nopassyo";
              }
            }
            else {
              param_1 = "nopassyo";
            }
          }
          else {
            param_1 = "nopassyo";
          }
        }
        else {
          param_1 = "nopassyo";
        }
      }
      else {
        param_1 = "nopassyo";
      }
    }
    else {
      param_1 = "nopassyo";
    }
  }
  else {
    param_1 = "nopassyo";
  }
  return param_1;
}
```

Our input has to pass a series of tests in order to be passed to the strncmp function, and any time it fails a test, it's set to "nopassyo".

Taking a look at the tests, it seems easiest to start from the end and work our way forward:

```
            if (param_1[5] == param_1[6]) {
              if (param_1[6] == '0') {
                if (param_1[7] != 'z') {
```

We can see here that the last three characters are going to be 00z:

*****00z

We see in the following section that the (fourth, not third, as the index starts at 0) character has to be "7":

```
    if (param_1[3] == '7') {
```

***7*00z

We can get the first character from the following section:

```
  if ((int)param_1[3] + (int)*param_1 == 0xab)
```

0xab is « in ascii, which is equivalent to the ascii number 171. We know that param_1[3] is 7, so we'll subtract the ascii number for "7" (55) from 171 to find that param_1 needs to be a "t" (116)

t**7*00z

Starting to look familiar? It's the challenge creator's name. If we try the string "t0m7r00z" as our first input, we'll see that we reach the next step:

```
$ nc poseidonchalls.westeurope.cloudapp.azure.com 9003
Enter the secret for the magic word: t0m7r00z
Enter the thing that's in my mind: 
```

No matter what we enter for the second input, it exits, so we'll change the string in /home/ctf/magic_word.txt to "t0m7r00z" and look back through r2 to see what's happening.

We can see that another comparison is made before flag.txt is called:

```
│     │││   0x55aaf37f9d6a      488d55ed       lea rdx, qword [var_13h]
│     │││   0x55aaf37f9d6e      488d45cc       lea rax, qword [var_34h]
│     │││   0x55aaf37f9d72      4889d6         mov rsi, rdx
│     │││   0x55aaf37f9d75      4889c7         mov rdi, rax
│     │││   0x55aaf37f9d78      e893fcffff     call sym.imp.strcmp     ; int strcmp(const char *s1, const char *s2)
│     │││   0x55aaf37f9d7d      85c0           test eax, eax
│    ┌────< 0x55aaf37f9d7f      7558           jne 0x55aaf37f9dd9
│    ││││   0x55aaf37f9d81      488d35ba0200.  lea rsi, qword [0x55aaf37fa042] ; "r"
│    ││││   0x55aaf37f9d88      488d3dfa0200.  lea rdi, qword str.home_ctf_flag.txt ; 0x55aaf37fa089 ; "/home/ctf/flag.txt"
```

We'll set a breakpoint right before the function call, and step through it:

```
[0x7f2c7170b090]> db 0x55aaf37f9d75
[0x7f28e65fb090]> dc
Enter the secret for the magic word: t0m7r00z
Enter the thing that's in my mind: aaaaaaaa
hit breakpoint at: 55fc0a6d9d75
```

We can view our registers at this point, and see our inputs being passed:

```
R0   rax     7ffe48f25bec      ([stack]) stack R W 0x6161616161616161 (aaaaaaaa) -->  ascii ('a')
     rbx     0                 0
A3   rcx     0                 0
A2   rdx     7ffe48f25c0d      ([stack]) stack R W 0x7a303072376d3074 (t0m7r00z) -->  ascii ('t')
A4   r8      0                 0
A5   r9      7fffffff          2147483647
     r10     9                 9
     r11     246               582
     r12     55fc0a6d9a80      (/home/anti/Downloads/trace) (.text) entry0 program R X 'xor ebp, ebp' 'trace'
     r13     7ffe48f25d00      ([stack]) stack R W 0x1 -->  1
     r14     0                 0
     r15     0                 0
A1   rsi     7ffe48f25c0d      ([stack]) stack R W 0x7a303072376d3074 (t0m7r00z) -->  ascii ('t')
A0   rdi     7ffe48f25bec      ([stack]) stack R W 0x6161616161616161 (aaaaaaaa) -->  ascii ('a')
```

We'll enter visual mode with "v" and see what's happening:

```
│             0x7f8d5ed7f950      89f8           mov eax, edi                                                                                                                                │
│             0x7f8d5ed7f952      31d2           xor edx, edx                                                                                                                                │
│             0x7f8d5ed7f954      660fefff       pxor xmm7, xmm7                                                                                                                             │
│             0x7f8d5ed7f958      09f0           or eax, esi                                                                                                                                 │
│             0x7f8d5ed7f95a      25ff0f0000     and eax, 0xfff                                                                                                                              │
│             0x7f8d5ed7f95f      3dc00f0000     cmp eax, 0xfc0          ; 4032                                                                                                              │
│         ┌─< 0x7f8d5ed7f964      0f8f78020000   jg 0x7f8d5ed7fbe2                                                                                                                           │
│         │   0x7f8d5ed7f96a      f30f6f0f       movdqu xmm1, xmmword [rdi]                                                                                                                  │
│         │   0x7f8d5ed7f96e      f30f6f06       movdqu xmm0, xmmword [rsi]                                                                                                                  │
│         │   0x7f8d5ed7f972      660f74c1       pcmpeqb xmm0, xmm1                                                                                                                          │
│         │   0x7f8d5ed7f976      660fdac1       pminub xmm0, xmm1                                                                                                                           │
│         │   0x7f8d5ed7f97a      660fefc9       pxor xmm1, xmm1                                                                                                                             │
│         │   0x7f8d5ed7f97e      660f74c1       pcmpeqb xmm0, xmm1                                                                                                                          │
│         │   0x7f8d5ed7f982      660fd7c0       pmovmskb eax, xmm0                                                                                                                          │
│         │   0x7f8d5ed7f986      4885c0         test rax, rax                                                                                                                               │
│        ┌──< 0x7f8d5ed7f989      7415           je 0x7f8d5ed7f9a0                                                                                                                           │
│        ││   0x7f8d5ed7f98b      480fbcd0       bsf rdx, rax                                                                                                                                │
│        ││   0x7f8d5ed7f98f      0fb60417       movzx eax, byte [rdi + rdx]                                                                                                                 │
│        ││   0x7f8d5ed7f993      0fb61416       movzx edx, byte [rsi + rdx]                                                                                                                 │
│        ││   0x7f8d5ed7f997      29d0           sub eax, edx                                                                                                                                │
│        ││   0x7f8d5ed7f999      c3             ret
```

We can see in the third section (directly before the ret call) that bytes from eax and edx (our two inputs) are being compared, then subtracted. 

Looking back at the condition to call the flag, we need the result of eax and edx being subtraced to equal zero:

```
│     │││   0x55aaf37f9d7d      85c0           test eax, eax
│    ┌────< 0x55aaf37f9d7f      7558           jne 0x55aaf37f9dd9
│    ││││   0x55aaf37f9d81      488d35ba0200.  lea rsi, qword [0x55aaf37fa042] ; "r"
│    ││││   0x55aaf37f9d88      488d3dfa0200.  lea rdi, qword str.home_ctf_flag.txt ; 0x55aaf37fa089 ; "/home/ctf/flag.txt"
```

If eax isn't equal to zero, the test call above will avaluate to false, and the flag won't be read.

If we continue execution until right before the comparison is made and read our registers, we see that the following values are being compared:

```
R0   rax     61                97 ascii ('a')
     rbx     0                 0
A3   rcx     0                 0
A2   rdx     74                116 ascii ('t')
```

Of course, this isn't going to equal zero. We can't just enter "t0m7r00z" for the second input, as the program 
exits if we do. If we look back at the Ghidra output, we'll see that we need to have two strings that are equal to each other, in order to have this evaluate to zero:

```
iVar1 = strcmp(local_3c,local_1b);
if (iVar1 == 0) {
  __stream = fopen("/home/ctf/flag.txt","r");
```
However, if we look back at the first strncmp function in Ghidra, we see that we can trick the program into thinking we have the correct input, by adding a single letter onto the end of both inputs, since the first strncmp function only reads the first eight characters:

```
iVar1 = strncmp(local_26,local_1b,8);
```

Now, we'll run it again:

```
:> dc                                                                                                                                                                                 [82/882]
Enter the secret for the magic word: t0m7r00zb                                                                                                                                                
Enter the thing that's in my mind: t0m7r00zb                                                                                                                                                  
hit breakpoint at: 55f2849d5d75
```

We'll continue execution until the sub call, and look at our registers again:

```
R0   rax     0                 0                                                                                                                                                              
     rbx     0                 0                                                                                                                                                              
A3   rcx     0                 0                                                                                                                                                              
A2   rdx     0                 0                                                                                                                                                              
A4   r8      0                 0                                                                                                                                                              
A5   r9      7fffffff          2147483647                                                                                                                                                     
     r10     a                 10                                                                                                                                                             
     r11     246               582                                                                                                                                                            
     r12     55f2849d5a80      (/home/anti/Downloads/trace) (.text) entry0 program R X 'xor ebp, ebp' 'trace'                                                                                 
     r13     7ffc3db92a60      ([stack]) stack R W 0x1 -->  1                                                                                                                                 
     r14     0                 0                                                                                                                                                              
     r15     0                 0                                                                                                                                                              
A1   rsi     7ffc3db9296d      ([stack]) stack R W 0x7a303072376d3074 (t0m7r00zb) -->  ascii ('t')                                                                                            
A0   rdi     7ffc3db9294c      ([stack]) stack R W 0x7a303072376d3074 (t0m7r00zb) -->  ascii ('t')  
...
```

0 minus 0 = 0!

If we continue execution now, we'll get our temporary flag:

```
:> dc
flag{test}

```
Now, we'll run it against the server to get our flag:

```
$ nc poseidonchalls.westeurope.cloudapp.azure.com 9003
Enter the secret for the magic word: t0m7r00zb
Enter the thing that's in my mind: t0m7r00zb
Poseidon{Its_L3_t3Mp5_DeS_C3r1s35}
```

There's our flag: **Poseidon{Its_L3_t3Mp5_DeS_C3r1s35}**






