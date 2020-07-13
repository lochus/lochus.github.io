---
title: "rgbCTF: Hallo?"
categories:
  - rgbCTF
tags:
  - rgbCTF
---

*The flag is exactly as decoded. No curly brackets.*

*NOTE: keep the pound symbols as pound symbols!*

*~BobbaTea#6235, Klanec#3100*

---

For this challenge, we're given a file called hmm.mp3. When we listen to the file, we can hear keypad tones, like we would hear when hitting numbers on a telephone.

To decode the keypad tones, or DTMF Tones, we can use multimon:

```
$ multimon-ng -t wav -a DTMF x.wav
multimon-ng 1.1.8
  (C) 1996/1997 by Tom Sailer HB9JNX/AE4WA
  (C) 2012-2019 by Elias Oenal
Available demodulators: POCSAG512 POCSAG1200 POCSAG2400 FLEX EAS UFSK1200 CLIPFSK FMSFSK AFSK1200 AFSK2400 AFSK2400_2 AFSK2400_3 HAPN4800 FSK9600 DTMF ZVEI1 ZVEI2 ZVEI3 DZVEI PZVEI EEA EIA CCIR MORSE_CW DUMPCSV X10 SCOPE
Enabled demodulators: DTMF
DTMF: 7
DTMF: 7
DTMF: 7
DTMF: 4
DTMF: 2
DTMF: 2
DTMF: 2
DTMF: 2
DTMF: 2
DTMF: 8
DTMF: 3
DTMF: 3
DTMF: 3
DTMF: #
DTMF: 9
DTMF: 9
DTMF: 9
DTMF: 3
DTMF: 3
DTMF: 3
DTMF: 3
DTMF: 8
DTMF: #
DTMF: 3
DTMF: 8
DTMF: 6
DTMF: 3
DTMF: 3
DTMF: 3
DTMF: #
DTMF: 8
DTMF: 6
DTMF: 6
DTMF: 6
DTMF: 6
DTMF: 6
DTMF: 3
DTMF: 3
DTMF: 7
DTMF: 7
DTMF: 7
DTMF: 7
DTMF: #
```

We can clean this up a bit:

```
$ multimon-ng -t wav -a DTMF x.wav | cut -d ":" -f2 | tr -d "\n" | tr -d " " | tr -d "DTMF"
...
7774222228333#99933338#386333#866666337777#
```

Now that we have the numbers being entered, we can make educated guesses based on the numbers being entered. Multimon-ng gave us some repeating numbers, but we can reasonably be sure that the first section evaluates to "rgbctf".

The second section doesn't make sense yet, but the third section being "dtmf" and the fourth section being "tones" makes sense. 

There are only three things that the first section could be: wet, yet or yeet. "wet" and "yet" don't make much sense, but yeet is plausable.

Voila, we have the flag!

**rbgctf#yeet#dtmf#tones#**
