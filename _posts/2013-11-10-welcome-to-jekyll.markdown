---
layout: post
title:  "CrackMe Writeup - SnD Reversing Tutorial #1"
date:   2017-11-17 10:18:00
categories: Crackme SnD Writeup Tutorial 
---

CrackMe #1 - DRAFT


The main goal of this CrackMe is to get the success message by validating a license key from a file.


##### TLDR:
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


 If we search the disassembled CrackMe executeable for external calls to <a href="https://msdn.microsoft.com/en-us/library/windows/desktop/aa363858(v=vs.85).aspx" target="_blank">CreateFile</a> function. We come up with a call to "CreateFileA" instead of "CreateFile". The "CreateFileA" is the American National Standards Institute(ANSI) version of "CreateFile", which is a standard for the charcter encoding system. The characters encoded using ANSI standard are 1 byte long compared to the 2 bytes long Unicode encoded characters. The early windows releases: Windows 95 and 98 were developed using Windows API 3.1 which only supported ANSI. However, unicode was later implemented from Windows NT and later windows releases. With the help of "translation layer" characters encoded using ANSI standard are translated into Unicode or the other way around and by this programs from windows 98 are able to run on NT and newer versions of windows <a href="https://ehsanakhgari.org/article/visual-c/2008-06-21/unicode" target="_blank"> (Akhgari,  2008)</a>. The call to "CreateFileA" function is made accordingly:
![SnD1-CrackMe-CreateFile-annotated](/assets/images/snd1-createfile-annotated.png)




The paramaters that are passed to "CreateFileA" function call are inverse compared to the Win32 API Reference, this is because how the call stack work?????.




change:
heating and cooling - qoute

Only fix/replace cooling unit. Qoute cost.
Adelaide Gas heating for heating - cooling. Cost about 10K

example


###### File Handling 


###### License Validation


 

