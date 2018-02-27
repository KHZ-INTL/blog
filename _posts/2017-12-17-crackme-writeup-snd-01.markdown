---
layout: post
title:  "CrackMe Writeup - SnD Reversing Tutorial #1"
date:   2017-11-17 10:18:00
categories: Crackme SnD Writeup Tutorial 
---

CrackMe #1 - DRAFT

The main goal of this CrackMe is to get the success message by either validating a license key from a file or patching the execution flow of the program.


##### Tl;dr:
Summary:

The licensing algorithm checks the Keyfile.dat file for a valid license key. It is possible to pass the licensing check if the license key complies with the following conditions:
+ The length of the license key should be at least 16 bytes long (16 letters).
+ Should contain a minimum of 8 47h(hex) or "G"s, before any zeroes in the license key.

The following license key should pass the licensing check: "GGGGGGGGGG000000".

The license validation algorithm is something like (psudo python):

{% highlight python %}
ESI = 0
EBX = 0
for each_letter/character in licenseKey:
	if letter == 0:
		if ESI < 8:
			JUMP [Invalid LicenseKeyMessage]
		elif ESI >=8:
			JUMP [Valid LicenseKeyMessage]
	if letter != 0x47 or "g":
		EBX = EBX + 1
	elif letter == 0x47 or "g":
		ESI = ESI + 1
{% endhighlight %}

##### Solving the CrackMe
Solving a simple CrackMe challenge like this one can be achieved using several methods, including patching the execution flow of the program by patching the operation codes (opcode), and/or devising a licensekey based on the licensing algorithm and the license key file "Read/Write" operations. This write-up will focus on the later option. 

To devise a license key first we need to identify the structure and as well as other parameters of a valid license key. These include:
+ License key length/size
+ Content type: is the license key made up of numbers, letters or both (alphanumeric) and/or special characters(e.g "!@#$...")
+ if the key is stored in a file, what is the file name?

To gather information about the license key we need to identify how the program interacts with it. In a real scenario, a program may need to validate and save the licensing information when it is activated. Furthermore, it may need to read the licensing information again when the program is re-launched. Programs have many options when it comes to saving data/user licensing infromation, these include:
+ Save data to the Registry (in case of windows OS).
+ Write to a file.
+ Handle licensing information using a remote server.

Keeping in mind that this is a simple CrackMe challenge, we can safely assume it would not handle its licenssing data using a webserver. It would not make sense to waste resources and cpu cycles just for a simple challenge for beginners? Knowing that it is handled locally, we now need to identify wether if data is stored in the registry or in a file.

Programs do not have direct access to the storage device for reading and writing data and thus it will need to go through the Kernal. In this scenario, the kernel is simply an interface that takes in write/read requests and manages the underlying low level operations. To access the kernal for read/write operations, the Windows 32 (win32) API library allow users to call and utilize its predefined functions and classes. The win32 API library documentation can be found:
+ <a href="https://msdn.microsoft.com/en-us/library/windows/desktop/dn933214(v=vs.85).aspx" target="_blank">View on Microsoft Developers Network</a>
+ <a href="http://laurencejackson.com/win32/index.html" target="_blank">Download Win32 API documentation from Laurence Jackson website</a>
+ 
<a href="#win32api.chm" target="_blank">Download Win32 API documentation from this blog (Thanks to Laurence Jackson)</a>

If a program needs to write to file, it needs to call a predifined win32 function. In this case the "WriteFile" function. We can use this as a signature to find wether if the license key is being stored to a file or a registry since different functions handle different operations. We can search the disassembled CrackMe executeable for this call by either:
+ Read and find it manually or,
+ Utilize available plugins.

If you are using Olly Debuger there are some plugins that can help with this, such as <a href="http://www.openrce.org/downloads/details/211/APIFinder" target="_blank">APIFinder</a> by Tomisslav Pericin. There are plugins available for other debugers, you will need do a google search. For example: debuggerName API finder/set breakpoint.


 If we search the disassembled CrackMe executeable for calls to <a href="https://msdn.microsoft.com/en-us/library/windows/desktop/aa363858(v=vs.85).aspx" target="_blank">CreateFile</a> function. We come up with a call to "CreateFileA" instead of "CreateFile". The "CreateFileA" is the American National Standards Institute(ANSI) version of "CreateFile", which is a standard for the charcter encoding system. The characters encoded using ANSI standard are 1 byte long compared to the 2 bytes long Unicode encoded characters. Meaning the ANSI standard has a maximum characterset of 255 characters while the UNICODE standard has a maximum characterset of 65,536 characters <a href="https://ehsanakhgari.org/article/visual-c/2008-06-21/unicode" target="_blank"> (Akhgari,  2008)</a>. The Win32 API documentation for "CreateFileA" states that it "Creates or opens a file...(and) The function returns a handle that can be used to access the file...". The call to "CreateFileA" function is made accordingly (in red box):
 
Figure 1: CreateFileA - Opening "Keyfile.dat" and Returning a file handle
![SnD1-CrackMe-CreateFile-annotated](/assets/images/snd1/snd1-createfile-annotated.png)

 
 In the screenshot above, we can see that before the CreateFileA call, the parameters for that function are pushed onto the stack. OllyDbg has cleverly annotated them. Comparing it to the API documentation, we can see that the function parameters are passed in reverse order. <a href="https://blogs.msdn.microsoft.com/oldnewthing/20040108-00/?p=41163/" target="_blank">Chen 2004</a>, from MSDN blogs, describes that parameters are passed in reverse order (in reference to CDECL calling convention)"...so that the first parameter is nearest to top-of-stack...". Thinking back to how the stack works, it does sort of make sense?, the Last element In would be the First to be Out (LIFO) <a href="https://en.wikipedia.org/wiki/LIFO_%28computing%29
" target="_blank">(Wikipedia, 2018)</a>. If you read the API refrence for CreateFileA we can see that in order to create a file we need to pass several parameters, among them is a file name (lpFileName).  The filename that is passsed for this particular call is "keyfile.dat". For the sake of the tutorial and the simpleness of this crackme, we can assume that this filename confirms that the license key is being stored locally and in this file. The CreateFileA api refrence mentions that it returns either a file handle or a "-1" using the EAX register. Meaning, if the CreateFileA function was able to open "Keyfile.dat", it would return the handle to that file in the EAX register and if it did not, it would return "-1".

Next, there is a compare instruction (CMP), taking the EAX register and "-1" as operands. It compares the content of the operands, specifically it performs a logical AND operation (please refer to an X86 Opcode <a href="https://c9x.me/x86/" target="_blank">manual</a>).Based on the result of the operation, flags such as the ZF(Zero), SF(Signed) and PF(Parity) flags are set in the EFLAGs register. In refrence to the mentioned x86 opcode manual (JCC section) the JNE - Jump if Not Equal (opcode 75) instruction jumps to the location passed as its operand (0040109A), if the Zf(zero) flag equal to 0 or the CMP/previous operation changed the state of the Z(zero) flag to 0(zero). Simply, in this case, if EAX did not equal to "-1" in the previous CMP operartion then it jumps to 0040109A. This is done to check if the license key is present.

Continuing on, few things are pushed on to the stack before calling MessageBoxA (refer to the win32 api if need to). Observing these parameters, the text for the MessageBoxA says "Evaluation period out of date...". To avoid getting this error we need to take the jump at JNE operation to change the flow of the program. To accomplish this, at the JNE operation(at 0x40107B), the state of the Zero flag needs to be 0. The Zero flag is set based on the CMP instruction. In order for CreateFileA to return a file handle and not "-1" it has to be able to open the specified file "Keyfile.dat". In the same directory as the crackme executeable ceate a file named "Keyfile.dat". With the licensekey file created, debugging again, the execution flow of the program has changed and we no longer see the evaluation error message. 

Jumping past the evaluation error message we land at 0x40109A where the stack is populated before a call to ReadFile function. The windows API refrence for <a href="https://msdn.microsoft.com/en-us/library/windows/desktop/aa365467(v=vs.85).aspx" target="_blank">ReadFile</a> mentions that the function reads data from a specified position within a specified file or an input/output device. The function takes the following parameters:
+ hFile (HANDLE): The handle to the file that needs to be read from,
+ lpBuffer (LPVOID): The pointer to the buffer, a pointer that points to the location in memory where data to be stored temporarly,
+ nNumberOfBytesToRead (DWORD): The amount of data (bytes) to be read.
+ ...




Essentially the ReadFile function is called to read the serial key from the Keyfile.dat into the memory.
Looking at it from a practical point of view, this function is being called to read the serial key from the "Keyfile.dat" file. 


Figure 2: ReadFile 
![SnD1-CrackMe-CreateFile-annotated](/assets/images/snd1/snd1-readfile.png)


Figure 3: License Algorithm
![SnD1-CrackMe-CreateFile-annotated](/assets/images/snd1/snd1-licensing-algoriythm.png)
	   



When analysing/trying to figure out what a piece of code does, it is helpful to start by observing what it does in general and then the specifics. This may help in building a simple mental picture.

Calling convention - ASM
http://unixwiz.net/techtips/win32-callconv-asm.html

###### File Handling 


###### License Validation


 

