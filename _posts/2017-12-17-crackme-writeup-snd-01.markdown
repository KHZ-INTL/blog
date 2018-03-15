---
layout: post
title:  "Crack-Me Writeup/Personal notes - SnD Reversing Tutorial #2"
date:   2017-11-17 10:18:00
categories: Crackme SnD Writeup  
---

The objective of this Crack-Me is to get the success message by either validating a license key from a file or patching the execution flow of the program. 

Crack-Me: Crack-Me from Search and Destroy (SnD) Reversing Tutorial by Lena #2

###### A personal note
At the time of writing of this write-up, I had moved from Windows to Linux OS. Some resources were not available. However, necessary resources are included to compensate. The pictures/screenshots may look different but please use memory addresses as a reference for navigating in between them.   

##### Tl;dr:
Summary

The licensing algorithm checks the Keyfile.dat file for a valid license key. A license key that complies with the following conditions would pass the algorithm:
+ The license key should be at least 16 Bytes long (16 Characters).
+ Have a minimum of 8(47h or "G"), before any zeroes in the license key.

A valid license key: "GGGGGGGGGG000000".

The license validation algorithm in psudo python:

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

##### Solving the Crac-kMe
Solving a simple Crack-Me challenge like this can be achieved using several methods. Methods such as patching the execution flow of the program by altering the operation codes (opcode) and/or devising a license key based on the licensing algorithm. This write-up will focus on the latter option. 

In devising a license key the structure and as well as relevant parameters needs to be considered. These include:
+ License key length/size
+ Content type: made up of numbers, letters or both (alphanumeric) and/or special characters e.g "!@#$..."
+ if the key is stored in a file, what is the file name?

To gather such information about the license key its required to identify how the software interacts with it. In a real scenario, a commercial software may validate, save and re-validate the licensing information when it is activated and re-launched. Programs have many options when it comes to saving data/user licensing information, these include:
+ Save data to the Registry (in case of Windows OS).
+ Write to a file.
+ Handle licensing information using a remote server.

This is a simple Crack-Me challenge, thus it would not handle its licensing data using a remote server. Assuming it is is handled locally, the remaining two options are saving the licensing data in the registry or in a file.

##### CreateFileA

Understanding how the software interacts with the OS helps in identifiying how the licensing data is stored. Programs do not have direct access to the storage device for reading and writing data and thus it will need to go through the Kernal. In this scenario, the kernel is simply an interface that takes in write/read requests and manages the underlying low-level operations. To access the kernal for reading/writing operations, the Windows 32 (win32) API library allows users to call and utilize its predefined functions and classes. The win32 API library documentation can be found:
+ <a href="https://msdn.microsoft.com/en-us/library/windows/desktop/dn933214(v=vs.85).aspx" target="_blank">View on Microsoft Developers Network</a>
+ <a href="http://laurencejackson.com/win32/index.html" target="_blank">Download Win32 API documentation from Laurence Jackson website</a>
+ 
<a href="#win32api.chm" target="_blank">Download Win32 API documentation from this blog (Thanks to Laurence Jackson)</a>

If a program needs to write to and read from a file, it needs to call a predefined win32 function. In this case the "CreateFile" function. We can use this as a signature to find whether if the license key is being stored in a file or a registry since different functions handle different operations. We can search the disassembled CrackMe executable for this call by either:
+ Reading and finding it manually or,
+ Utilizing available plugins.

If you are using Olly Debugger there are some plugins that can help with this, such as <a href="http://www.openrce.org/downloads/details/211/APIFinder" target="_blank">APIFinder</a> by Tomisslav Pericin. There are plugins available for other debuggers, you will need to do a google search. For example debuggerName API finder/set breakpoint.

 If we search the disassembled CrackMe executable for calls to <a href="https://msdn.microsoft.com/en-us/library/windows/desktop/aa363858(v=vs.85).aspx" target="_blank">CreateFile</a> function. We come up with a call to the "CreateFileA" instead of "CreateFile". The "CreateFileA" is the American National Standards Institute(ANSI) version of "CreateFile", which is a standard for the character encoding system. The characters encoded using ANSI standard are 1 byte long compared to the 2 bytes long Unicode encoded characters. Meaning the ANSI standard has a smaller characterset compared to Unicode<a href="https://ehsanakhgari.org/article/visual-c/2008-06-21/unicode" target="_blank"> (Akhgari,  2008)</a>.
 
The Win32 API documentation for "CreateFileA" states that it "Creates or opens a file...(and) The function returns a handle that can be used to access the file...". The call to "CreateFileA" function is made accordingly (in the red box) refer to figure 1:
 
Figure 1: CreateFileA - Opening "Keyfile.dat" and Returning a file handle
![SnD1-CrackMe-CreateFile-annotated](/assets/images/snd1/snd1-createfile-annotated.png)

 In the screenshot above, we can see a call to CreateFileA. Before the call, parameters for that function are pushed onto the stack. Comparing it to the API documentation, we can see that the function parameters are passed in reverse order. <a href="https://blogs.msdn.microsoft.com/oldnewthing/20040108-00/?p=41163/" target="_blank">Chen 2004</a>, from MSDN blogs, describes that parameters are passed in reverse order (in reference to CDECL calling convention)"...so that the first parameter is nearest to top-of-stack...". Thinking back to how the stack works, it does sort of make sense?, the Last element In would be the First to be Out (LIFO) <a href="https://en.wikipedia.org/wiki/LIFO_%28computing%29
" target="_blank">(Wikipedia, 2018)</a>. If you read the API reference for CreateFileA we can see that in order to create a file we need to pass several parameters, among them is a file name (lpFileName).  The filename that is passed to this particular call is "keyfile.dat". This filename confirms that the license key is being stored locally and in this file. The CreateFileA API reference mentions that it returns either a file handle or a "-1" using the EAX register. Meaning, if the CreateFileA function was able to open "Keyfile.dat", it would return the handle to that file in the EAX register and if it did not, it would return "-1".

Next, there is a compare (CMP) instruction , taking the EAX register and "-1" as operands. It compares the content of the operands, specifically it performs a logical AND operation (please refer to an X86 Opcode <a href="https://c9x.me/x86/" target="_blank">manual</a>). Based on the result of the operation, flags such as the ZF(Zero), SF(Signed) and PF(Parity) flags are set in the EFLAGs register. 

In reference to the mentioned x86 opcode manual (JCC section), see figure 2, the JNE - Jump if Not Equal (opcode 75) instruction jumps to the location passed as its operand (0040109A), if the Zf flag equal to 0 or the CMP/previous operation changed the state of the Zf flag to 0. Simply, in this case, if EAX did not equal to "-1" in the previous CMP operation then it jumps to 0040109A. This is done to check if the license key file is present.

Figure 2: <a href="https://c9x.me/x86/html/file_module_x86_id_146.html" target="_blank">JNE</a> - Jump if Not Equal. Courtesy of c9x.me
![SnD1-CrackMe-CreateFile-annotated](/assets/images/snd1/snd1-jne.png)

Continuing on, few things are pushed on to the stack before calling MessageBoxA (refer to the Win32 API if need to). Observing these parameters, the text for the MessageBoxA says "Evaluation period out of date...". To avoid getting this error message we need to take the jump at JNE operation to change the flow of the program. To accomplish this, at the JNE operation(at 0x40107B), the state of the Zero flag needs to be 0. The Zero flag is set based on the CMP instruction. In order for CreateFileA to return a file handle and not "-1" it has to be able to open the specified file "Keyfile.dat". In the same directory as the crackme executable create a file named "Keyfile.dat". With the license key file created, debugging again, the execution flow of the program has changed and we no longer see the evaluation error message. 

Figure 3: ReadFile 
![SnD1-CrackMe-CreateFile-annotated](/assets/images/snd1/snd1-readfile.png)

Jumping past the evaluation error message we land at 0x40109A where the stack is populated before a call to the ReadFile function, refer to figure 3, above. The windows API reference for <a href="https://msdn.microsoft.com/en-us/library/windows/desktop/aa365467(v=vs.85).aspx" target="_blank">ReadFile</a> mentions that the function reads data from a specified position within a specified file or an input/output device. The function takes the following parameters:
+ hFile (HANDLE): The handle to the file that needs to be read from,
+ lpBuffer (LPVOID): The pointer to the buffer, a pointer that points to the location in memory where data to be stored temporarily,
+ nNumberOfBytesToRead (DWORD): The amount of data (bytes) to be read.
+ etc...

Looking at what is actually being pushed on to the stack before the ReadFile call, keeping in mind that the parameters are pushed in reverse order. We can see the following:
+ lpOverlapped: 0 ,
+ nNumberOfBytesRead: offset [00402173],
+ nNumberOfBytesToRead : 46 hex (bytes),
+ lpBuffer: offset [0040211A],
+ hFile Handle: EAX

In reference to the Win32 API manual and the pushed parameters, we can identify that it tries to open a file using the handle in the EAX register, which was set by the CreateFileA call earlier. Then it reads a maximum of 46h ("h" for hex) or 70 bytes from the file and stores them at the location pointed by the lpBuffer ([0040211A]). Try adding some text in the "Keyfile.dat" you had created earlier and you should be able to view them in the memory/hex dump at lpBuffer location (40211A). For example, figure 3 - hex dump section. Keeping in mind that with CDECL calling convention it is standard to return values from a call in the EAX register. 

Essentially the ReadFile function is called to read the serial key from the Keyfile.dat into the memory.
Looking at it from a practical point of view, this function is being called to read the serial key from the "Keyfile.dat" file. 
The next operation "TEST EAX, EAX" conducts a logical AND operation on the returned value in the EAX register and sets the state of ZF based on the result. The win32 API manual mentions that if the function succeeds then it would return a true value, a non-zero value. Thus if the function returns a non-zero value then the TEST operation on a non zero value would be 1, see <a href="http://xcprod.com/titan/XCSB-DOC/binary_and.html/" target="_blank">AND</a> documentation. Based on this the ZF is set to 0 and vice versa for zero value. If you do an TEST (or an AND operation) on 0 and 0 the result would be 0 and the zero flag would be set to 1. In the next operation, JNE SHORT 04010B4, a jump will take place based on the ZF status. If the ZF equals to 0 then it would jump to 04010B4 and if it did not it would continue to the next instruction, JMP 4010F7, to an invalid license key error message. It is necessary to take the jump at JNE 0x004010B0 to avoid the invalid license error. 

Figure 4: ReadFile - Test eax:eax = 1; zf=0, jmp taken
![SnD1-CrackMe-CreateFile-annotated](/assets/images/snd1/snd1-readfile2.png)


##### License Validation algorithm
Successfully taking the jump at JNE after the Test operation, we land at 0x4010B4, and avoided the licensing error. This section may look chunky and complex. To get a sense of what it does we can use some basic analysis strategy. It is helpful to start by observing what this section of code does in general and then the specifics. We can do this by first taking a look where the jumps leads to and what sort of functions are called. Refering to figure 3, the first jump, JL (Jump if Lower) at 0x04010BF and as well as the JL at 0x04010D6 leads to another licensing error. 


Figure 3: License Validation Algorithm
![SnD1-CrackMe-CreateFile-annotated](/assets/images/snd1/snd1-licensing-algoriythm.png)
   

Figure 4: License Validation Algorithm Jump Outline
![SnD1-CrackMe-License Validation-algorithm](/assets/images/snd1/LIC-alg.png)




Calling convention - ASM
http://unixwiz.net/techtips/win32-callconv-asm.html



 

