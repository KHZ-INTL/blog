---
layout: post
title:  "CrackMe Writeup - SnD Reversing Tutorial #1"
date:   2017-11-17 10:18:00
categories: Crackme SnD Writeup Tutorial 
---

CrackMe #1 - DRAFT


The main goal of this CrackMe is to get the success message by validating a license key from a file.


##### Tl;dr:
Summary:

The licensing algorithm checks the Keyfile.dat file for a valid license key. It is possible to pass the algorithm check as long as the license key complies with the following conditions:
+ The length of the license key should be at least 16 bytes long (16 letters).
+ Should contain a minimum of 8 47h(hex) or "G"s, before any 0s in the license key.

The following license key would pass the licensing check: "GGGGGGGGGG000000".

The license validation algorithm is something like (psudo python):

{% highlight python %}
ESI = 0
EBX = 0
for each_letter in licenseKey:
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
Solving a simple CrackMe challenge like this one can be achieved using several methods, including patching the Operation Codes(OP Codes), and/or devising a licensekey based on the licensing algorithm and the license key "Read/Write" API calls. In this tutorial we will focus on the later option. In order to devise a license key first we need to identify the structure and as well as other paramaters of a valid license key. These include:
+ License key length/size
+ Content type: is the license key made up of numbers, letters or both (alphanumeric)
+ if the key is stored in a file, what is the file name?

To gather information about the license key we need to identify how the program interacts with it. In a real scenario, a program may need to validate and save the licensing information when it is activated. Furthermore, it may need to read the licensing information again when the program is re-launched. Programs have many options when it comes to saving data/user licensing infromation, these include:
+ Save data to the Registry (in case of windows).
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


 If we search the disassembled CrackMe executeable for calls to <a href="https://msdn.microsoft.com/en-us/library/windows/desktop/aa363858(v=vs.85).aspx" target="_blank">CreateFile</a> function. We come up with a call to "CreateFileA" instead of "CreateFile". The "CreateFileA" is the American National Standards Institute(ANSI) version of "CreateFile", which is a standard for the charcter encoding system. The characters encoded using ANSI standard are 1 byte long compared to the 2 bytes long Unicode encoded characters. Meaning the ANSI standard has a maximum characterset of 255 characters while the UNICODE standard has a maximum characterset of 65,536 characters <a href="https://ehsanakhgari.org/article/visual-c/2008-06-21/unicode" target="_blank"> (Akhgari,  2008)</a>. The call to "CreateFileA" function is made accordingly (in red box):
 
![SnD1-CrackMe-CreateFile-annotated](/assets/images/snd1-createfile-annotated.png)

 In the screenshot above, we can see, before the CreateFileA call, the paramaters for that function are pushed onto the stack. OllyDbg has cleverly annotated them. Comparing it to the API documentation, we can see that the function paramaters are passed in reverse order. This is because when using the Win32 API, it is recomended to use the CDECL (C declaration) calling convention. Since CDECL is derived/originates from the C programming language, the function paramaters are passed in reverse order <a href="https://en.wikipedia.org/wiki/X86_calling_conventions#cdecl" target="_blank">(Wikipedia,  2018)</a>. According to  <a href="https://blogs.msdn.microsoft.com/oldnewthing/20040108-00/?p=41163/" target="_blank">Chen 2004</a>, from MSDN blogs, he describes that functions are passed in reverse order (in reference to CDEC)"...so that the first parameter is nearest to top-of-stack...". Thinking back to how the stack works, it does sort of make sense?, the Last element In would be the First to be Out (LIFO) <a href="https://en.wikipedia.org/wiki/LIFO_%28computing%29
" target="_blank">(Wikipedia, 2018)</a>. Getting back from the wild tangent, if you read the API refrence for CreateFile/A we can see that in order to create a file we need to pass several paramaters, among them is a file name (lpFileName). The filename that is passsed in for this particular call is "keyfile.dat". For the sake of the tutorial and the simpleness of this crackme, we can assume that this filename confirms that the license key is being stored locally and in this file. 

If we look further, right after the call there is a compare instruction (CMP), that compares the content of EAX register and the value "-1" and sets . The CreateFileA api refrence mentions that it returns either a file handle or a "-1" using the EAX register. Meaning, if the CreateFileA function was able to open "Keyfile.dat", it would return the handle to that file in the EAX register and if it didn't it would return "-1". Continuing onto the next operation, there is a Jump If Not Equal (JNE) operation. The JNE  
	   



When analysing/trying to figure out what a piece of code does, it is helpful to start by observing what it does in general and then the specifics. This may help in building a simple mental picture.

Calling convention - ASM
http://unixwiz.net/techtips/win32-callconv-asm.html

###### File Handling 


###### License Validation


 

