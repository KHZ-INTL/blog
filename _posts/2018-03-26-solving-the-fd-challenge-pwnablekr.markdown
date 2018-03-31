---
layout: post
title:  "Solving the fd challenge - pwnable.kr"
date:   2018-03-26 10:18:00
categories: capture-the-flag pawnablekr Writeup  
---

# writing, in progress.

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

In the description, it mentions "...What is a file descriptor in linux?". We can guess that the challenge has something to do with file descriptors. The OpenBSD/Linux <a href="https://en.wikipedia.org/wiki/File_descriptor" target="_blank">manual </a>

(section 4), describes a file descriptor (in short "fd") as a non zero integer used as a handle to acess files or input/output devices. Each process with the exception of deamons have three standard file descriptors

Every program:
    By default, on creation of each process with the exception of daemons, three stremes opened for it: input, output and error.

http://man7.org/linux/man-pages/man3/stdout.3.html#SYNOPSIS 





