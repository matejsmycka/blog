# Virus.DOS.Kuku - Recreating MS-DOS malware in Python

It's crucial to emphasize that the intention behind this guide is purely educational, and I do not endorse malware development for malicious purposes. With that out of the way, let's dive in.

## Rant

I came across this 10-year-old [YouTube video](https://www.youtube.com/watch?v=IWtcvOLsLIk) where malware from 1991 is shown. At that time, malware was as funny and annoying as it should be; nowadays, cyberspace is packed with spyware and ransomware, and there is little room for trolling. Viruses are not what they used to be, and capitalism is to blame.

## KUKU!

The video shows Eastern European malware on MS DOS, which randomly overwrites files and displays more and more colored KUKU! Popups. Kuku means in multiple Slav languages something like "I gotch you".

You can view [the original source code](https://github.com/vxunderground/MalwareSourceCode/blob/main/MSDOS/K-Index/Virus.MSDOS.Unknown.kuku.bas) in the VX Underground malware collection. However, it was written in TURBO-BASIC, which is even more painful to read than regular assembly.

See this snippet:

```vb
data"n$=string$(8,63)+chr$(46)+chr$(66)+chr$(65)+chr$(83):dim dta%(32),find%(32)
data"for a%=0% to 32%:dta%(a%)=0:next
data"for z=0 to len(n$)-2 step 2:find%(z/2)=asc(mid$(n$,z+2,1))*256+asc(mid$(n$,z+1,1)):next
data"reg 1,&h1A00:reg 8,varseg(dta%(0)):reg 4,varptr(dta%(0)):call interrupt &h21
data"reg 1,&h4e00:reg 3,attr:reg 8,varseg(find%(0)):reg 4,varptr(find%(0)):call interrupt &h21
data"if reg(1)<>0 then p$=string$(15,255):goto findfirstfile1
data"for a=0 to 32:h=dta%(a) and 255:p$=p$+chr$(h):l=(dta%(a)-h)/&h100 and 255:p$=p$+chr$(l):next
data"findfirstfile1:
data"dta$=p$:f$=mid$(dta$,&h1f,13):if f$=string$(len(f$),255) then
data"for J=1 to 1500:Sound Rnd(1)*(1500-j)+40,.01:NEXT:delay(2)
data"screen 1:def seg=&Hb800:for a=0 to 16384:poke a,rnd(1)*255:next:exit sub
data"end if
```

## Recreation

This pseudo-nostalgia, which I have not experienced for a time, has inspired me to recreate the KUKU virus on modern operating systems. I chose Python, an absolutely fantastic tool for GUI windows. At the same time, it is inherently cross-platform in the sense that scripts are cross-platform. Unfortunately, this is not true for the Python interpreter.
 
### Portability 

Malware can't rely on the victim having Python installed, so I used the **PyInstaller** library, which wraps the script, libraries, and interpreter into one binary.
Unfortunately, PyInstaller does not support cross-compilation, so it must be compiled on the OS and architecture on which the malware will be used.

### GUI

For the GUI, I used the **tkinter** library; thanks to that, I avoided using the low-level GUI API, and I can use the high-level call, which works on both Windows and Linux.

## Result

The last two binaries behave the same as the original virus; it's not the fastest implementation, and other languages would probably be better. But the speed with which I created this malware is unmatched. Python is written blazingly fast.

Thank you for reading; feedback is appreciated.

## Resources

[1]: Original YT video - https://www.youtube.com/watch?v=IWtcvOLsLIk
[2]: PyInstaller docs - https://pyinstaller.org/en/stable/
[3]: Kuku source code - https://github.com/vxunderground/MalwareSourceCode/blob/main/MSDOS/K-Index/Virus.MSDOS.Unknown.kuku.bas
[4]: Turbo BASIC - https://en.wikipedia.org/wiki/PowerBASIC

