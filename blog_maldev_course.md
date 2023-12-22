# Malware Development Essentials Course criticism - Proving an alternative.

///RAW

I recently completed the RED TEAM Operator: Malware Development Essentials Course from Sektor7 Institute. Here is my honest opinion about this course, and I will try to provide resources that are at least equal in quality to the mentioned course.

I don't have a problem with the overall content of the material provided. I have a problem with the course format, the scripts, and the price. I hope this review will prompt a change in the authors' approach.

Positives

Don't get me wrong, the course will teach you what they say it will teach you, and it's really good for beginners. The course is a great intro to the Windows API. Even if you never want to create malware, the material is mostly very high quality, and  I only criticize minor parts. If your employer offers you this course, definitely take it.

Criticism

first, the course contains 4 hours of material in videos. Some parts of these videos only repeat previously mentioned things and do not contain any informational value. For example, the narrator shows the disassembled code  x64dbg several times and spends much time looking for different parts in the assembler. This reflects an actual debugging process, but it is unsuitable for video. If text and screenshots were provided, the material would be much shorter and equally educational. Many things are duplicated, and I don't think the viewer must see compile.bat run 15 times throughout the course.

The course would benefit if it were in text form or supplemented by short videos explaining the complex parts.

Secondly, the code is OK, but the compilation scripts are not broken down to the necessary details and work more automagically. You won't know what they do if you want to do something more custom. On the other hand, the code is broken down to the smallest detail, sometimes to the detriment. I know it is an essential course, but there is no need to explain the elementary operations C++. The viewer should be able to program; things like main functions  getchar() do not need to be explained.

I would emphasize learning to search for information in the Windows API documentation and explaining what rights a given user must have to run them.

I know this is a personal preference, but why are demos shown inside note++ I prefer to show you how to set the maldev coding environment on Windows properly. Resources on how to get all needed components like compiler, make, required libraries, and how to generate shellcode.

And lastly, the price. This is a criticism not only of these materials but also of the certification business in general. The course costs $199, an industry-wide reasonable price for the material. I can see the value if you don't want to put effort into finding your materials and want to get them on a platter, but then I would debate whether you are even capable of learning how to create malware with that mindset.

If you are a third-world country citizen, student, or someone who tries to learn it just because of passion for a subject, I don’t think this is a good investment.

Providing an alternative

To support my arguments, I put together several sources based on the course's table of contents, trying to show that finding the same quality sources is not difficult and all the information contained in the course is freely available.

TOC of the Maldev course

Portable Executable

Droppers

Obfuscation and Hiding

Backdoors and Trojans

Code Injection

My alternative resources:

https://gamehacking.academy - Although this page is designed for learning how to develop cheats for games, it does a great job of explaining how code injection, DLL injection, code caves, and other concepts work.

https://github.com/chvancooten/maldev-for-dummies/tree/main/Exercises/Exercise%201%20-%20Basic%20Shellcode%20Loader - This Repo provides simple shellcode dropper exercises and also examples, and it is not limited to `C++` only, You can choose Go, Rust or Csharp. EDR and AV evasion exercises are also provided.

https://github.com/cr-0w/maldev and https://www.youtube.com/@crr0ww - This brilliant YouTuber covers a lot of MalDev subjects, and his repo has the following topics covered:   Shellcode Injection, NTDLL, Indirect Syscalls, Direct Syscalls and DLL Injection.

https://www.ired.team/offensive-security/code-injection-process-injection/backdooring-portable-executables-pe-with-shellcode - Another great resource on Trojan malware development, aka embedding a shellcode inside the legitimate binary.

These resources are free and cover all sections of the course mentioned above.

I hope I have provided only constructive criticism and nobody will get offended,

thank you for reading, and feel free to leave feedback.

