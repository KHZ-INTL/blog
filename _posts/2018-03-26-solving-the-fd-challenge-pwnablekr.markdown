---
layout: post
title:  "Solving the fd challenge - pwnable.kr"
date:   2018-03-26 10:18:00
categories: capture-the-flag pawnablekr Writeup  
---

##### This blog is incomplete. In progress.

The objective of this challenge (fd - pwnable.kr) is to capture the flag.

Challenge: <a href="http://www.pwnable.kr/play.php" target="_blank">fd</a>, [Toddler's Bottle section] from Pwnable.kr
<img alt="fd challenge from pwnable.kr" src="http://www.wmon.com.tw/RO/photo/card/poring.jpg"/>

Rules:
No brute forcing as mentioned on the challenge website.

Note: If you are here to simply take the flag without the intention to learn, you are not doing your self any favors. Eventually you will face the harder challenges without the necessary skills. These early challenges will teach and give you the foundational skills.

##### Tl;dr:
Summary


##### Solving the challenge
To solve Capture The Flag challenges, we can approach them using a generic routine/technique:

1. Information gathering
    + Research
    + Static Analysis
    + Dynamic Analysis
2. Identifying an attack surface

###### Information Gathering
The more knowledge we have about the target, the challenge, the more likely were are able to make an appropriate descisions and solving it. We can gather information by doing a research about the target in general, anyalyse the target statically (binary analysis) and dynamically when its running (behaviour analysis).
A good place to initate our reasearch is by reading the description of the challenge. It includes a cheesy hint, a youtube video and a command to connect to the challenge server, refer to figure 1 below.

Figure 1: Challenge Description
![fd-pwnablekr-challenge-description](/assets/images/fd-pwnablekr/fd_chall_desc.png)

In the description, it mentions "...What is a file descriptor in linux?". We can guess that the challenge has something to do with file descriptors. In simple terms, a file descriptor is a handle that can be used to access a file or an input/output device. In linux, by default on creation of a process, with the exception of daemons, three standard/default streams are opened for it: Standard Input, Standard Output and Standard Error. These streams can also be refered to numerically:
+ Standard input or stdin: 0
+ Standard output or stdout: 1
+ Standard Error or stderr: 2

With these streams a program or process can interact with its enviroment. With stdin the program can take input, for example with the use of "cin" in c++. With stdout and stderr information can be used to displayed to the console.

If you require further explanation please refer to:
+ The Linux <a href="http://man7.org/linux/man-pages/man3/stdout.3.html#SYNOPSIS " target="_blank">manual </a>(section 4) on "fd, stdin, stdout, stderr" or
+ Holidaylvr's <a href="https://www.youtube.com/watch?v=EqndHT606Tw" target="_blank">video</a> tutorial "fd, dup()..." on youtube. 

Now that we are familiar with file descriptor, lets take a look at the challenge.

##### static analysis
Lets connect to the challenge server with the given ssh command:
{% highlight bash %}
    ssh fd@pwnable.kr -p2222
{% endhighlight %}

The ssh command connects to "fd@pwnable.kr" using the 2222 port. Use the password ("guest") to sign into the server when you're asked. Refer to figure 2, below.

Figure 2: Connecting to the fd challenge server
![fd-pwnablekr-challenge-description](/assets/images/fd-pwnablekr/connecting-to-fd.png)

The server is running on linux. We can use the following commands:
+ ls: List items in the current dirrctory
+ cd: Change dirrectory
+ vim: Edit/view text and source code files using the vim editor 

If we do a listing (ls) of the current dirrectory we see the:
+ fd: Challenge binary,
+ fd.c: Challenge source code and
+ Flag

One way of analysing the challenge (binary) is to do a static analysis. With this method we try to get as much information from the binary without having it running. Sometimes  a program may behave differently when its running, for example some viruses. If we did not have the source code available we could use the "strings" command to extract the strings/texts from the binary. Sometimes with this method we can extract the function names, hardcoded texts, section names and etc. Running the strings command for the fd binary returns some useful information, refer to figure 3.

Figure 3: Strings analysis of fd binary
![strings-analysis-of-fd-binary](/assets/images/fd-pwnablekr/fd-strings_analysis.png)

The strings analysis of the fd binary shows some intresting texts. For example, "good job :)", "learn about Linux file IO" and "LETMEWIN". They sound like a message you would get if you validate a correct or an invalid password/input. Also there is a reference to "atoi". If you are not familiar with programming or atoi function, basically it extracts numbers from a given string. There is also "/bin/cat flag". The cat command prints the file its given as its argument. In this case it prints the flag file. If you scroll down, near the end there is "main". This may refer to the main function of the program. We can search for this function when debugging in our dynamic analysis. Finally, there is also "pass argv[1] a number". This essentially translates to: when launching the program, you may have to pass a number. Argv is a structure where arguments are stored. When a program is launched, in this case, the first argument, argv[0] is the name of binary it self, the second argument is argv[1] is the argument passed when launched. From before we found a call to "atoi". From this we can guess that the program tries to extract numbers from the user input or argv[1].

Since we have access to the source code of the binary we could also analyse that. Open the source code in vim:

{% highlight bash %}
    vim fd.c
{% endhighlight %}

The souce file should look like the one in figure 4.

Figure 4: Source code of fd shown in vim editor- vim fd.c
![strings-analysis-of-fd-binary](/assets/images/fd-pwnablekr/fd-source-file-vim.png)

###### A general understanding

It is easier to understand if we tried to grasp what the code does in general and then the specifics. The code imports three headers and has 1 function named "main". In the main function something is read and stored from a file descriptor to a variable and it is compared with a hardcoded string "LETMEWIN". If the variable did equal then the flag is shown . If it did not, "learn about Linux file IO" message is shown. 

###### The specifics
Now that we have an understanding of what the code does in general, let's look at what each line of code do. 

In the 3 three lines, 3 header are imported: stdio, stdlib and string.

{% highlight c++ %}
    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
{% endhighlight %}

<a href="http://www.cplusplus.com/" target="_blank"> C++.com</a> (2017) describes them accordingly. The <a href="http://www.cplusplus.com/reference/cstio/" target="_blank">stdio</a> header, it is imported to use its predefined functions to perform input/output operations using the standard input, output and error streams. The <a href="http://www.cplusplus.com/reference/csstdlib/" target="_blank">stdlib</a> header hosts several general purpose functions such as string conversions (atoi), dynamic memory management and integer arithmetics. The <a href="http://www.cplusplus.com/reference/cstring" target="_blank">string</a> header is imported for it's string manipulation functions. 



Next, an array named "buf" with a size of 32 elements is defined:

{% highlight c++ %}
    char buf[32];
{% endhighlight %}

Each elements is defined to be the type of Char. The Char property is used to identify each elements as a Character type. A Char is one byte in size (8bits) and used to represent one character for example: "A", "!", or "{" (<a href="http://www.cplusplus.com/doc/tutorial/variables/" target="_blank">C++.com</a>, 2017). To visualise an example of this please look at figure 5 below.

Figure 5: The buf array visualised
![diagram: a graphical visualisation of the buf array](/assets/images/fd-pwnablekr/buf-array-fd.jpg)

On the 5th line a function named "main" is defined. It takes argc, argv and envp as parameters and returns an integer (a number). 
