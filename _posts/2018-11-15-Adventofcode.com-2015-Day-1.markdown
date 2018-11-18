---
layout: post
title:  "Solving the Advent of Code - 2015 Day 1"
date:   2018-11-15 16:46:00
comments: true
disqus: false
categories: Programming challenges AdventOfCode.com python puzzles
---


The Advent of Code (AdventOfCode.com) website hosts many CTF-like events, with a variety of programming puzzles to solve. It was developed by `Eric Wastl`, the author of Vanilla JS, PHP Sadness and other cool projects. The Events are usually hosted annually, in December. The events and the puzzles are packaged with a festive touch and in each level, the challenger will need to resolve issues relating to Christmas using his programming knowledge. These puzzles were developed for a variety of skill sets and skill levels, and are solvable using any programming language of choice. Each puzzle is unique, small and fun to solve. They do not require specialised hardware.


##### Challenge Description:
- Name: "The Advent of Code - 2015 Day 1: Not Quite Lisp"
- Difficulty: Easy
- Category: Programming
- Rating/Award: 2 Stars
- Prerequisite: Sign Up - Free
- Host: <a href="https://adventofcode.com/2015/day/1" target="_blank">AdventOfCode.com</a> - Season 2015
![fd-pwnablekr-challenge-description](/assets/images/adventofcodecom/season2015/main_art.png)

###### Rules:
No rules were specified on the challenge website.


##### Tl;dr:
Day 1 has one puzzle, but it is made up of two parts. The puzzle is based on parsing a long stream of characters. The stream of characters is comprised of two types of characters, '(' and ')'. Both are instructions, the opening parenthesis, '(' instruction is defined to move Santa 1 floor up in an apartment building. The closing parenthesis, ')' instruction does the opposite, moves Santa 1 floor down. The Apartment building have endless floors going in either direction. Thus, it is possible to be at floor -10, in the basement. The first part of the puzzle requires to find the floor that Santa ends up at when all the instructions have been completed. In the second part, it is required to find the instruction number that places Santa in floor -1 (basement) for the first time. The Python(version 3) programming language was used to solve the puzzle since it is simple and easy.


##### Solving the challenge
Like any other challenges, it is important to understand the puzzle and its parameters. The puzzle description is the best place to start, there are no binaries and source code to analyse.

A brief summary of the puzzle description would be:
+ Santa starts at ground floor, floor 0 and follows 1 instruction at a time.
+ An opening parenthesis, '(' instructs Santa to go 1 floor up.
+ A closing Parenthesis, ')' instructs Santa to go 1 floor down.
+ The apartment building has endless floors going in either direction (up and down).


For the full description please see figure 1, below.

Figure 1: The Full Challenge Description
![fd-pwnablekr-challenge-description](/assets/images/adventofcodecom/day1/challenge_description.png)

###### Obtaining the Instruction set
To get the instructions for Santa, click on the <a href="https://adventofcode.com/2015/day/1/input" target="_blank">get your puzzle input</a> link in the puzzle description. For the link to work, you will need to be signed in first. The link should open another tab and it should look similar to:


`((((()(()(((((((()))(((()((((()())(())()((...`

##### Finding a solution: Part 1
The first part of the puzzle requires to find the floor which Santa ends up at when all instructions have been completed. To be able to solve this programmatically, each instruction in the set needs to be checked whether if it is an increment or decrement of a floor and a method of keeping track of current floor. In programming we can store numbers, current floor using <a href="https://en.wikibooks.org/wiki/Python_Programming/Variables_and_Strings" target="_blank">variables</a> in memory. Also, it is possible to check each instruction one by one using the <a href="https://wiki.python.org/moin/ForLoop" target="_blank">For loop</a>. The For loop goes through each item chronologically and executes predefined code. Maybe there are many methods of solving this puzzle, however, the simplest that readily comes to mind is using addition and subtraction.

###### Solving the puzzle - part 1
To keep track of Santa when he goes up or down a floor, a variable is defined (floor) and updated on each execution of an instruction.

First, store the instruction set of the puzzle inside a variable for accessing it later.

{% highlight python %}
instruction_set = "((((()(()(((((((()))..."
{% endhighlight %}

Second, define the `floor` variable. As mentioned before, this is the variable that is used to keep track of the floor Santa is at.

{% highlight python %}
floor = 0
{% endhighlight %}

Third, create a for loop to iterate through the instruction set.

{% highlight python %}
for instruction in instruction_set:
{% endhighlight %}

Next, the `instruction` from the for loop needs to be checked if it is a opening or a closing parenthesis. This is done using the <a href="https://wiki.python.org/moin/IfStatementWithValue" target="_blank">`If`</a> and <a href="https://www.tutorialspoint.com/python/python_if_else.htm" target="_blank">`elif`</a> else if statement. Based on what the `instruction` is, the floor variable is either incremented or decremented.

{% highlight python %}
instruction_set = "((((()(()(((((((()))(((()((((()..."
floor = 0
for instruction in instruction_set:
    if instruction == "(":
        floor = floor +1
    elif instruction == ")":
        floor = floor -1
{% endhighlight %}

Finally, when the iteration is complete, the value of `floor` is printed to the screen. The string <a href="https://docs.python.org/2/library/string.html#custom-string-formatting" target="_blank">format</a> function replaces the place holders ({}) with the value defined, in this case the floor variable.

{% highlight python %}
instruction_set = "((((()(()(((((((()))(((()((((()..."
for instruction in instruction_set:
    if instruction == "(":
        floor = floor +1
    elif instruction == ")":
        floor = floor -1
print("Santa is at floor: {}".format(floor))

{% endhighlight %}


To get the answer, the script needs to be executed. In terminal execute python3 with the script as its argument. Your answer may be different since the website provides a unique instruction set per user. See figure 2, below.

Figure 2: script executed and the result
![script executed and the result](/assets/images/adventofcodecom/day1/1st_method1_output_result.png)

##### Validating the output

To check if the output is correct, it needs to be validated. Validating is done in puzzle description page. If it was correct, you should be awarded a star and unlocked the second part of the puzzle. See figure 3, below.

Figure 3: Validated answer:
![validating output and unlocking 2nd part of puzzle](/assets/images/adventofcodecom/day1/validating_unlocking_2nd_part.png)

##### Solving the second part

Now the second part is unlocked and able to proceed to the second part of the puzzle. A brief summary of the description for the second part:
+ The first instruction has position 1. The second has position 2, and the rest of the instructions also follows the same order.
+ Find the position of the instruction which places Santa in the `-1` floor for the first time.

For the full description please see figure 3, above.


##### finding a solution: Part 2
To find the position of the instruction that places Santa in `-1` floor for the first time, we need to keep track of floor. The code is already implemented to keep track of Santa. The only thing that is left is to find the instruction number when Santa is at floor -1. This can be accomplished by keeping track of total instructions executed using another variable.


###### Solving the puzzle - part 2

First, create one new variable, instructions_executed. Initialise instructions_executed to 1. Instructions_executed is set to 1 because the first instruction has position 1, is number 1.

{% highlight python %}
floor = 0
instructions_executed = 1
{% endhighlight %}


To check if Santa has arrived at the basement (floor -1), the floor variable needs to be compared to `-1`. This can be accomplished using the `if` statement. If Santa did arrive at the basement, the loop needs to be stopped and the instruction number printed, `break` command is used to do so.
{% highlight python %}
for instruction in instruction_set:
    if instruction == "(":
        floor = floor +1
    elif instruction == ")":
        floor = floor -1
    if floor == -1:
        print("Santa has arrived to the basement!\nFloor: {}\nInstruction Number: {}".format(floor, instructions_executed))
        break
{% endhighlight %}


The instructions_executed variable needs to be updated/incremented on each iteration, on the execution of each instruction. The variable needs to be increased after the if conditions in the for loop, this is because we do not want to change it before checking the floor as it will give actual code number +1.
{% highlight python %}
for instruction in instruction_set:
    if instruction == "(":
        floor = floor +1
    elif instruction == ")":
        floor = floor -1
    if floor == -1:
        print("Santa has arrived to the basement!\nFloor: {}\nInstruction Number: {}".format(floor, instructions_executed))
        break
    instructions_executed = instructions_executed + 1
{% endhighlight %}

Altogether, with the modifications the code should like similar to:
{% highlight python %}
instruction_set = "((((()(()(((((((()))(((()((((()..."
floor = 0
instructions_executed = 1
for instruction in instruction_set:
    if instruction == "(":
        floor = floor +1
    elif instruction == ")":
        floor = floor -1
    if floor == -1:
        print("Santa has arrived to the basement!\nFloor: {}\nInstruction Number: {}".format(floor, instructions_executed))
        break
    instructions_executed = instructions_executed + 1
{% endhighlight %}


To get the answer, in the terminal execute python3 with the script as its argument. see figure 4, below. Your results may be different since the website provide a unique instruction set per user.

Figure 4: script executed and the result
![script_2nd part executed and the result](/assets/images/adventofcodecom/day1/exec_2nd_part.png)

##### Validating the output
To check if the output is correct, it needs to be validated. Validating is done in puzzle description page. If it was correct, you should be awarded a star and should have a total of two stars for the first day.

##### Congratulation
Congratulation for completing the first puzzle. If you liked the puzzle or wish to support Eric or Advent of Code, you can support him indirectly by sharing the website (<a href="https://www.adventofcode.com" target="_blank">AdventOfCode.com</a>) with others or directly via PayPal or Coinbase. See Advent Of Code about page for more information.
![fd-pwnablekr-challenge-description](/assets/images/adventofcodecom/season2015/main_art.png)



##### Sources - Helpful links
Python general reference information

<a href="https://docs.python.org/3/tutorial/index.html" target="_blank">Python.org</a>

<a href="https://www.tutorialspoint.com/python/" target="_blank">TutorialPoint.com</a>

<a href="https://www.w3schools.com/python/default.asp" target="_blank">W3Schools.com</a>

Python For loop

<a href="https://wiki.python.org/moin/ForLoopindex.htm" target="_blank">Wiki.Python.org</a>

<a href="https://www.w3schools.com/python/python_for_loops.asp" target="_blank">W3Schools.com</a>

