Download Link: https://assignmentchef.com/product/solved-cs107-lab2-c-strings
<br>
During this lab, you will:

<ol>

 <li>read and analyze C code that operates on chars and C-strings</li>

 <li>use Valgrind to detect and debug memory errors</li>

 <li>write some of your own code to wrangle C-strings</li>

</ol>

First things rst! Find an open computer to share with a partner. Introduce yourself and tell them about your favorite music to listen to while coding.

<h1>Get started</h1>

Clone the lab repo by using the command below. This command creates a lab2 directory containing the project les.

<h1>Lab exercises</h1>

<h2>1) Code study</h2>

The code excerpts for this week’s lab are taken from the standard C library. By studying this code, we can learn how the library functions are implemented, as well as become better informed about how to properly use these functions as a client and what consequences to expect when we don’t.

<h3>ASCII and ctype</h3>

&lt;ctype.h&gt; o ers small utility functions such as isdigit and tolower that operate on single characters. The ctype function isxdigit below reports whether a given character is a valid hex digit.

Here are a few questions to work out with your partner. If you need an ASCII refresher, use man ascii .

<ol>

 <li>It seems the rst test checks for a digit character, so the second test must be in charge of validating hex letters, but its expression is rather mystifying. What is the purpose of the bitwise OR with 32 and the character subtraction?</li>

 <li>The cast to unsigned is deliberate and necessary. Get into gdb and print the expression where c is ‘#’ both with and without the cast, e.g.:</li>

</ol>

You’ll see that the result is di erent without the cast. Why?

<ol start="3">

 <li>Use man string to see the list of string functions in the standard library. You could reimplement ixdigit as a single call to one of them. Describe how.</li>

</ol>

<h3>strcpy and strncpy</h3>

Below are simple implementations for strcpy and its cousin strncpy . For variety, I chose a strcpy that operates via pointer arithmetic as a contrast to the strncpy that uses arrayindexing.

Points to ponder:

<ol>

 <li>Dissect the expression used as while loop test in strcpy . That single statement copies a char, advances the two pointers, and tests the stopping condition all in one! How does it work?</li>

 <li>What happens if you call strcpy passing a src string that is missing its null terminator? What if src is NULL or an invalid address?</li>

 <li>Neither function checks if dst is large enough to hold a copy of the string. Such protection against user error would be very welcome, but sadly is not possible in C. Why not? So what is going to happen if dst is too small?</li>

 <li>strcpy appends always and exactly one null-terminator to the characters copied to dst . How many null-terminators are written by strncpy ? (Be careful!)</li>

</ol>

Aside: In the days of yore, pointer arithmetic had a performance advantage over array indexing, but smart modern compilers make this largely moot. A lot of old code and old programmers haven’t yet gotten that memo. Pointer arithmetic is especially rife in code that wrangles C-strings where it is perhaps tolerable, but for arrays of type other than char, array indexing is almost always going to be more readable.

<h2>2) Tools interlude</h2>

Each lab will devote some time to hands-on practice with the tools. Last week, it was sanitycheck and gdb. This week, we’ll do a little more with gdb and introduce Valgrind.

<h3>gdb x command</h3>

<ol>

 <li>Load the code program under the debugger, set a breakpoint at main and run the program.</li>

 <li>When you hit the breakpoint, try out the gdb x command which is used to examine raw memory:</li>

</ol>

x shows the contents of memory starting at a given address. In the command above, the

modi ers /8bc tell gdb to print 8 bytes interpreting each byte as character format (ascii). Use help x to read about other available modi ers. What modi ers would print 3 bytes in hex format instead?

<ol start="3">

 <li>Use next to step execution through the strcpy calls and use x to show the contents after each line to see how buf is changing. Verify that what you observe corroborates with your understanding of the behavior of strcpy and strncpy .</li>

</ol>

<h3>Using Valgrind</h3>

Now that we are moving on to more signi cant use of pointers, our programs become more at risk for memory errors. These can be di          cult bugs to track down and Valgrind is a supremely helpful tool to have in your arsenal. However, it takes some practice to learn how to interpret a Valgrind report. If you haven’t already, review the our guide to valgrind (/class/cs107/guide/valgrind.html) and let’s try it out now.

Read over the code in buggy.c to see its 3 planted memory errors. Consider just error #1 to start.

Run ./buggy 1 and you should be rewarded with a Segmentation fault (core dumped) . A segfault results from an attempt to read/write an inaccessible/invalid address.

Ok, so now you know your program as a memory error but not much else.

Run buggy under gdb. You observe the same segfault, but now can use the gdb backtrace command to learn where execution was at the time of the crash. That’s

helpful!

Exit gdb and run under Valgrind: valgrind ./buggy 1 . The Valgrind report gives more detail including type of memory error, execution callstack, and the value of the invalid address. Furthermore, Valgrind also detected an earlier problem that preceded the seg fault — an uninitialized value of size 8 (sizeof pointer) right before the invalid read of size 1 (char). This is the very clue need to lead you to the right place to x!

Repeat steps 1-4 for buggy 2 and buggy 3 and dig into the Valgrind report for each. Note the terminology that Valgrind uses for di erent kinds of errors. Your goal is to learn how to relate the error as reported by Valgrind back to the root cause.

All three buggy programs have a memory error, but the visibility of the error di ers. buggy 1 crashes with a seg fault: every time, every run. buggy 2 has a visible aw in that its output is garbage/unpredictable (did you note the change in running with Valgrind and without?), but it does not crash. buggy 3 appears work correctly, the output looks just ne and program successfully runs to completion. A memory error may cause no visible harm, but that doesn’t mean the program is correct, it just “got lucky” this time. By keeping Valgrind at your side, you get a vigilant partner that can detect these errors that are lying in wait, even before they produce an observable e ect.

We recommend that you run Valgrind early and often during your development cycle. Focus on the rst error reported (it often cascades from there), nd, and x, and repeat any remaining errors. Don’t move on until you get a clean report from Valgrind. Note that memory leaks don’t demand the immediate attention that errors do. Leaks can (and should) be safely ignored until the nal phase of polishing a working program. When grading assignments, we will run your submission under Valgrind and expect to see clean bill of health, so be sure you have checked it yourself before calling it done.

<h2>3) Write, test, debug, repeat</h2>

In this part of the lab, you will write a version of the printenv program, used to show the “environment variables” that pertain to your terminal session. The environment is a list of keyvalue pairs that provide information about the context for your terminal session and con gure the way processes behave. Type echo $USER $HOST $HOME $SHELL to a few of your environment variables. You have already used the USER environment variable when you cloned your assignments; the USER environment variable is set to your SUNet ID when you log into myth. Other environment variables include PATH (where the system looks for programs to run), HOME (path to your home directory),and SHELL (what command line interpreter you are using; most people should be using /bin/bash in cs107).

You will write your own myprintenv and will use env to test it. First, practice using the standard commands:

<ol>

 <li>printenv will show your environment variables, env allows you to change them. Read the man pages for printenv and env .</li>

 <li>Run printenv with no arguments and skim its output. Then try printenv USER and printenv SHELL . What is the output from a bad request printenv BOGUS ?</li>

 <li>Run printenv , then run env BINKY=1 WINKY=2 printenv . What changes between the two? Run printenv again with no arguments to determine whether changes to the environment were transient or permanent.</li>

</ol>

Now go on to coding the implementation of myprintenv .

<ol>

 <li>Review the code in c . The program is mostly complete, save the implementation of the get_env_value helper function.. As you review the given code, be sure to take note of the 3-argument variant of the main function it uses to access the environment. Read the comment in myprintenv.c for more info.</li>

 <li>Implement the get_env_value function to search the environment for a variable by name. Take advantage of the handy &lt;string.h&gt; library functions to help with this task.</li>

 <li>Use sanitycheck (/class/cs107/sanitycheck.html) to test your work. Edit the custom_tests le for additional test coverage.</li>

</ol>

<h1>Check o with TA</h1>

At the end of the lab period, submit the checko form and ask your lab TA to approve your submission. Take stock of your understanding with our selfcheck (/class/cs107/selfcheck.html#lab2).





