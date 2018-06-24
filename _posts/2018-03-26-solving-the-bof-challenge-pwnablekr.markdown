---
layout: post
title:  "Solving the bof challenge - pwnable.kr"
date:   2018-03-30 10:18:00
categories: capture-the-flag pawnablekr Writeup buffer-overflow  
---

This post is incomplete. I hope to finish this in my upcoming spare time.
 

The objective of this challenge (bof - pwnable.kr) is to learn about buffer overflow and capture the the flag. This challenge is an introductory to the bof principles.

Challenge: <a href="http://www.pwnable.kr/play.php" target="_blank">bof</a>, [Toddler's Bottle section] from Pwnable.kr
<img alt="bof challenge from pwnable.kr" src="http://photos1.blogger.com/blogger/3726/3243/320/Smokie_Card.png"/>

Rules:
No brute forcing as mentioned on the challenge website.

Note: If you are here to simply take the flag without the intention to learn, you are not doing your self any favors. Eventually you will face the harder challenges without the necessary skills. These early challenges will teach and give you the foundational skills.

##### Tl;dr:
Summary

The challenge revolves around learning and understanding about buffer over flow. In this challenge a variable used as a buffer is overflowed and the stack is modified as result. 

##### Solving the challenge
To solve Capture The Flag challenges, we can approach them using a generic routine/technique:

1. Information gathering
    + Research
    + Static Analysis
    + Dynamic Analysis
2. Identifying an attack surface

###### Information Gathering
We can gather information by doing a research about the target in general, analyse the target statically (binary analysis) and dynamically when its running (behavior analysis).
A good place to initiate our research is by reading the description of the challenge. It includes a cheesy hint, a YouTube video and a command to connect to the challenge server, refer to figure 1 below.

Figure 1: Challenge Description
![fd-pwnablekr-challenge-description](/assets/images/fd-pwnablekr/fd_chall_desc.png)

In the description, it mentions "...What is a file descriptor in linux?". We can guess that the challenge has something to do with file descriptors. In simple terms, a file descriptor is a handle that can be used to access a file or an input/output device. In linux, by default on creation of a process, with the exception of daemons, three standard/default streams are opened for it: Standard Input, Standard Output and Standard Error. These streams can also be referred to numerically:
+ Standard input or stdin: 0
+ Standard output or stdout: 1
+ Standard Error or stderr: 2

With these streams a program or process can interact with its environment. With stdin the program can take input, for example with the use of "cin" in c++. With stdout and stderr information can be used to displayed to the console.

If you require further explanation please refer to:
+ The Linux <a href="http://man7.org/linux/man-pages/man3/stdout.3.html#SYNOPSIS " target="_blank">manual </a>(section 4) on "fd, stdin, stdout, stderr" or
+ Holidaylvr's <a href="https://www.youtube.com/watch?v=EqndHT606Tw" target="_blank">video</a> tutorial "fd, dup()..." on YouTube. 

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
+ ls: List items in the current directory
+ cd: Change directory
+ vim: Edit/view text and source code files using the vim editor 

If we do a listing (ls) of the current directory we see the:
+ fd: Challenge binary,
+ fd.c: Challenge source code and
+ Flag

One way of analysing the challenge (binary) is to do a static analysis. With this method we try to get as much information from the binary without having it running. Sometimes  a program may behave differently when its running, for example some viruses. If we did not have the source code available we could use the "strings" command to extract the strings/texts from the binary. Sometimes with this method we can extract the function names, hard coded texts, section names and etc. Running the strings command for the fd binary returns some useful information, refer to figure 3.

Figure 3: Strings analysis of fd binary
![strings-analysis-of-fd-binary](/assets/images/fd-pwnablekr/fd-strings_analysis.png)

The strings analysis of the fd binary shows some interesting texts. For example, "good job :)", "learn about Linux file IO" and "LETMEWIN". They sound like a message you would get if you validate a correct or an invalid password/input. Also there is a reference to "atoi". If you are not familiar with programming or atoi function, basically it extracts numbers from a given string. There is also "/bin/cat flag". The cat command prints the file its given as its argument. In this case it prints the flag file. If you scroll down, near the end there is "main". This may refer to the main function of the program. We can search for this function when debugging in our dynamic analysis. Finally, there is also "pass argv[1] a number". This essentially translates to: when launching the program, you may have to pass a number. Argv is a structure where arguments are stored. When a program is launched, in this case, the first argument, argv[0] is the name of binary it self, the second argument is argv[1] is the argument passed when launched. From before we found a call to "atoi". From this we can guess that the program tries to extract numbers from the user input or argv[1].

Since we have access to the source code of the binary we could also analyse that. Open the source code in vim:

{% highlight bash %}
    vim fd.c
{% endhighlight %}

The source file should look like the one in figure 4.

Figure 4: Source code of fd shown in vim editor- vim fd.c
![strings-analysis-of-fd-binary](/assets/images/fd-pwnablekr/fd-source-file-vim.png)

###### A general understanding

It is easier to understand if we tried to grasp what the code does in general and then the specifics. The code imports three headers and has 1 function named "main". In the main function something is read and stored from a file descriptor to a variable and it is compared with a hard coded string "LETMEWIN". If the variable did equal then the flag is shown . If it did not, "learn about Linux file IO" message is shown. 

###### The specifics
Now that we have an understanding of what the code does in general, let's look at what each line of code do. 

In the first 3 three lines, 3 header are imported: stdio, stdlib and string.

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

Figure 5: The buf array visualised - diagram design inspired by c++.com
![diagram: a graphical visualisation of the buf array](/assets/images/fd-pwnablekr/buf-array-fd.jpg)

On the 5th line a function named "main" is defined. It takes argc, argv and envp as parameters and returns an integer (a number). Argc (argument count) is an integer that holds the count, the total number of arguments passed from the command line. By default the name of the program is considered an argument thus argc is incremented by 1 and pre-appended to argv. Argv (argument vector) is an array that holds the arguments passed from command line. The first argument passed by the user will be the 2nd element of argv. This is because the name of the program is automatically pre-pended. Envp is an array where environment strings are stored/referenced (<a href="https://docs.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-6.0/aa299386(v=vs.60)" target="_blank">Microsoft</a>, 2006).

The main function consist of many operations. They are described chronologically: 

The first operation:
{% highlight c++ %}

    if(argc<2){
        printf("pass argv[1] a number\n");
        return 0;
    }
{% endhighlight %}

 The first operation checks if argc is less than 2. This is to check if the user has passed a password or the required input. To get argc to equal 2 or more, one or more arguments needs to be passed when launching the program. Keeping in mind that by default argc=1 since the name of the program is considered an argument. If argc is less than 2 or the user did not pass an argument, then a message is printed "pass argv[1] a number" and the program exits.

{% highlight c++ %}

    int fd = atoi(argv[1]) - 0x1234;    
{% endhighlight %}

In this operation two things occur. First a variable named fd with a type of integer is defined. Immediately it is initialised with a subtraction operation. The first operand of subtraction is atoi(argv[1] or the user input) and the second operand is a number in hex. The first operand is the returning value to a call to atoi function. The atoi function takes a string (argv[1]) and extracts the integers. The second parameter is in hex since it has the "0x" prefix. To convert this to decimal for subtraction, we can use the int() conversion function from python. This function takes in a string and its base and returns it in base10 form. The hex number equates to 4660 in decimal, see below:

{% highlight python %}

    int("0x1234", 16)
    = 4660
{% endhighlight %}

If you remember reading about file descriptors, they were referred to as fd for short. This variable might be used as a reference to a fd.

{% highlight c++ %}

    int len = 0;
    len = read(fd, buf, 32);

{% endhighlight %}

Next, a variable named len is defined. It is initialised with zero but on the next line it is set to the returning value of a call to the read function. According to the Linux Programmer's <a href="http://man7.org/linux/man-pages/man2/read.2.html" target="_blank">Manual</a> (man 2 read) the read function takes in the following parameters:
+ fd: The file descriptor to read from.
+ buf*: Pointer to buffer, where read bytes are stored temporarily.
+ Count: Number of bytes to read.

And attempts to read up to $Count number of bytes from $fd file descriptor and stores it in $buf* buffer. In this case the read function is called to read using the following parameters:
+ fd: fd = atoi(argv[1]) - 4660
+ buf: buf[32]
+ count: 32

From this we can confirm that the first argument passed by the user (argv[1]) is used to determine the file descriptor to be used in the read function call.

Next, the buf variable is compared with a hard-coded string "LETMEWIN" using the strcmp function.

{% highlight c++ %}
    if(!strcmp("LETMEWIN\n", buf)){
        print("good job :)\n");
        system("/bin/cat flag");
        exit(0);
}
{% endhighlight %}


The strcmp function takes in two strings, compares them and return an integer indicating their relationship. If both strings are the same, the returned integer would be 0 and if they are not the same the returned integer would be a non-zero value. If the strings are the same then:
+ A message is printed to the console, "good job :)".
+ The flag is printed also to the console using the cat command.
+ The program exits.

Else:
{% highlight c++ %}
    print("learn about file IO\n");
    return 0;
{% endhighlight %}
If the strings are not the same:
+ A message is printed to the console, "learn about file IO".
+ The program exits by returning 0.

##### Identifying an attack surface
Since we have the source code and know what the program does exactly, it is not necessary to conduct a dynamic analysis. Now that we understand what the program does exactly:
+ Extracts the integer from the user input.
+ Subtracts 4660 from that.
+ Use that as a file descriptor to read 32 bytes.  
+ Compares the read bytes to "LETMEWIN".
+ If both strings are the same, the flag is printed to the console.

To get the flag, the the buf variable must be the same as "LETMEWIN". The only thing that modifies that variable is the read function. The read function reads 32 bytes from the file descriptor passed as one of its parameters. Since we have control of which file descriptor are the bytes read from we have to figure out a way to write to it. Going back to the basics, three file descriptors/streams are opened for each process. The only one that applies to our problem is Standard Input, 0. This is because we need to write to a file descriptor, the other two are for output purposes. Now that we have a file descriptor that we can write to, we need the fd passed to read function to equal to 0 (Standard Input file descriptor= 0). 

The math is simple:
{% highlight c++ %}
    X - 4660 = 0
    
    Therefore X must equal 4660:
    4660 - 4660 = 0

{% endhighlight %}

If we launch the program from the command line with an argument of 4660 we should be able to write the required text "LETMEWIN" to the Standard Input:
{% highlight c++ %}
    ./fd 4660
    LETMEWIN
{% endhighlight %}

##### Congratulations
If you did everything right, you should be greeted with a success message and the flag (figure 6):

Figure 6: The success message and the flag printed out to the console.
![The success message: 'Good job :)', the flag 'mommy! I think I know what a file descriptor is!!'](/assets/images/fd-pwnablekr/fd-success.png)

Dont forget to validate your flag on <a href="http://www.pwnable.kr/play.php" target="_blank">Pwnable.kr</a>:

Figure 7: Validating the flag on Pwnable.kr (first challenge):
![](/assets/images/fd-pwnablekr/fd-validate.png)




