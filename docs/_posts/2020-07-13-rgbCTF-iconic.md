---
title: "rgbCTF: Name a more iconic band"
categories:
  - rgbCTF
tags:
  - rgbCTF
---

"*I'll wait.*

*The flag for this challenge is all the passwords in alphabetical order, each separated by a single white-space as an MD5 hash in lower case*

*md5(passwordA passwordB passwordC ...)*

*Example: if the passwords were "dog" and "cat", the flag would be*
*rgbCTF{md5("cat dog")}*
*rgbCTF{b89526a82f7ec08c202c2345fbd6aef3}*


*~Klanec#3100*"

---

For this challenge, we're given a data.7z file, containing a file called "data". Running file against this file shows us that it's the following file type:

```
$ file data
data: ELF 64-bit LSB core file, x86-64, version 1 (SYSV)
```

We'll use volatility to go through the core dump. We first need to find a useable profile:

```
$ volatility imageinfo -f data                                                                                                                                                                       
Volatility Foundation Volatility Framework 2.6                                                                                                                                                                                
INFO    : volatility.debug    : Determining profile based on KDBG search...                                                                                                                                                   
          Suggested Profile(s) : Win7SP1x64, Win7SP0x64, Win2008R2SP0x64, Win2008R2SP1x64_24000, Win2008R2SP1x64_23418, Win2008R2SP1x64, Win7SP1x64_24000, Win7SP1x64_23418                                                   
                     AS Layer1 : WindowsAMD64PagedMemory (Kernel AS)                                                                                                                                                          
                     AS Layer2 : VirtualBoxCoreDumpElf64 (Unnamed AS)                                                                                                                                                         
                     AS Layer3 : FileAddressSpace (/root/Downloads/data)                                                                                                                                                      
                      PAE type : No PAE                                                                                                                                                                                       
                           DTB : 0x187000L                                                                                                                                                                                    
                          KDBG : 0xf8000183f110L                                                                                                                                                                              
          Number of Processors : 1                                                                             
     Image Type (Service Pack) : 1                                                                             
                KPCR for CPU 0 : 0xfffff80001840d00L                                                                                                                                                                          
             KUSER_SHARED_DATA : 0xfffff78000000000L                                                           
           Image date and time : 2020-07-04 03:41:04 UTC+0000                                                  
     Image local date and time : 2020-07-04 04:41:04 +0100 
```

We'll use the profile Win7SP1x64 for the rest of this challenge. We can use volatility to find the memory locations of the SYSTEM and SAM files, in order to grab the hashes for all users:

```
$ volatility --profile Win7SP1x64 -f data hivelist                                                                                                                                                   
Volatility Foundation Volatility Framework 2.6                                                                                                                                                                                
Virtual            Physical           Name                                                                     
------------------ ------------------ ----                                                                     
0xfffff8a0000b1010 0x000000003bddd010 \SystemRoot\System32\Config\DEFAULT                                      
0xfffff8a0001bd010 0x000000003b072010 \SystemRoot\System32\Config\SECURITY                                     
0xfffff8a000935010 0x000000003cc02010 \Device\HarddiskVolume1\Boot\BCD                                                                                                                                                        
0xfffff8a000d97410 0x000000002ec00410 \SystemRoot\System32\Config\SOFTWARE                                                                                                                                                    
0xfffff8a00105c010 0x0000000034ae1010 \SystemRoot\System32\Config\SAM                                                                                                                                                         
0xfffff8a001113010 0x0000000034128010 \??\C:\Windows\ServiceProfiles\NetworkService\NTUSER.DAT                                                                                                                                
0xfffff8a001184010 0x000000003a916010 \??\C:\Windows\ServiceProfiles\LocalService\NTUSER.DAT                                                                                                                                  
0xfffff8a001324010 0x000000002d801010 \??\C:\Users\in rainbows\ntuser.dat                                                                                                                                                     
0xfffff8a0014db010 0x000000002e2ee010 \??\C:\Users\in rainbows\AppData\Local\Microsoft\Windows\UsrClass.dat                                                                                                                   
0xfffff8a001b21010 0x000000001ad82010 \??\C:\Windows\System32\config\COMPONENTS                                                                                                                                               
0xfffff8a001eee010 0x000000000734d010 \??\C:\Users\Administrator\ntuser.dat                                                                                                                                                   
0xfffff8a001ef9010 0x0000000005d52010 \??\C:\Users\Administrator\AppData\Local\Microsoft\Windows\UsrClass.dat                                                                                                                 
0xfffff8a006680010 0x000000001ce0d010 \??\C:\Users\ok computer\ntuser.dat                                                                                                                                                     
0xfffff8a006712010 0x000000000e3b1010 \??\C:\Users\ok computer\AppData\Local\Microsoft\Windows\UsrClass.dat                                                                                                                   
0xfffff8a00000e010 0x0000000001dea010 [no name]                                                                                                                                                                               
0xfffff8a000023010 0x0000000001cb5010 \REGISTRY\MACHINE\SYSTEM                                                                                                                                                                
0xfffff8a00005d010 0x0000000001d71010 \REGISTRY\MACHINE\HARDWARE
```

Now that we know the Virtual memory locations, we'll dump the passwords:

```
$ volatility --profile Win7SP1x64 -f data hashdump -y 0xfffff8a000023010 -s 0xfffff8a00105c010 > hashes.txt
...
cat hashes.txt

Administrator:500:aad3b435b51404eeaad3b435b51404ee:756c599880f6a618881a49a9dc733627:::                                                                                                                                        
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::                                                                                                                                                
pablo honey:1001:aad3b435b51404eeaad3b435b51404ee:7aaedbb86a165e8711cb46ee7ee2d475:::                                                                                                                                         
the bends:1002:aad3b435b51404eeaad3b435b51404ee:dcbd05aa2cdd6d00b9afd4db537c4aa3:::                                                                                                                                           
ok computer:1003:aad3b435b51404eeaad3b435b51404ee:74dea9ee338c5e0750e000d6e7df08e1:::                                                                                                                                         
kid a:1004:aad3b435b51404eeaad3b435b51404ee:c9c55dd07a88b589774879255b982602:::                                                                                                                                               
amnesiac:1005:aad3b435b51404eeaad3b435b51404ee:7b05eefac901e1c852baa61f86c4376f:::                                                                                                                                            
hail to the thief:1006:aad3b435b51404eeaad3b435b51404ee:8eb2b699eeff088135befbde0880cfd4:::                                                                                                                                   
in rainbows:1007:aad3b435b51404eeaad3b435b51404ee:419c1a5a5de0b529406fb04ad5e81d39:::                                                                                                                                         
the king of limbs:1008:aad3b435b51404eeaad3b435b51404ee:a82da1c9a00f83a2ca303dc32daf1198:::                                                                                                                                   
a moon shaped pool:1009:aad3b435b51404eeaad3b435b51404ee:4353a584830ede0e7d9e405ae0b11ea4::: 
```

Instead of trying to crack these ourselves, we'll use Crackstation, since they have a much bigger wordlist. We just need to clean up the hashes a bit to get them into a suitable NTLM format:

```
$ cat short.txt | cut -d ":" -f 4                                                                                                                                                                    
756c599880f6a618881a49a9dc733627                                                                                                                                                                                              
31d6cfe0d16ae931b73c59d7e0c089c0                                                                                                                                                                                              
7aaedbb86a165e8711cb46ee7ee2d475                                                                                                                                                                                              
dcbd05aa2cdd6d00b9afd4db537c4aa3                                                                                                                                                                                              
74dea9ee338c5e0750e000d6e7df08e1                                                                                                                                                                                              
c9c55dd07a88b589774879255b982602                                                                                                                                                                                              
7b05eefac901e1c852baa61f86c4376f                                                                                                                                                                                              
8eb2b699eeff088135befbde0880cfd4                                                                                                                                                                                              
419c1a5a5de0b529406fb04ad5e81d39                                                                                                                                                                                              
a82da1c9a00f83a2ca303dc32daf1198                                                                                                                                                                                              
4353a584830ede0e7d9e405ae0b11ea4 
```

Pasting these hashes into CrackStation gives us the following passwords:

```
supercollider
anyone can play guitar 
my iron lung 
karma police 
idioteque 
pyramid song 
there, there 
weird fishes/arpeggi 
lotus flower
burn the witch
```

We need to sort these, and clean them up a bit:

```
$ sort pass | tr '\n' ' ' | tr -s " " > final.txt
...
cat final.txt 

anyone can play guitar burn the witch idioteque karma police lotus flower my iron lung pyramid song supercollider there, there weird fishes/arpeggi
```

Now, we can just go in and remove the double spaces, and get the md5 hash of these passwords to get the flag!

**rgbCTF{cf271c074989f6073af976de00098fc4}**
