Download Link: https://assignmentchef.com/product/solved-hw-2-shell-cs-162
<br>
In this homework, you’ll be building a shell, similar to the Bash shell you use on your CS 162 Virtual Machine. When you open a terminal window on your computer, you are running a shell program, which is bash on your VM. The purpose of a shell is to provide an interface for users to access an operating system’s services, which include file and process management. The Bourne shell (sh) is the original Unix shell, and there are many different flavors of shells available. Some other examples include ksh (Korn shell), tcsh (TENEX C shell), and zsh (Z shell). Shells can be interactive or non-interactive. For instance, you are using bash non-interactively when you run a bash script. It is important to note that bash is non-interactive by default; run bash -i for a interactive shell.

It also contains a little exercise to better see how threads are scheduled on real systems. You may want to do that first, as it doesn’t involve lots of reading and coding.

The operating system kernel provides well-documented interfaces for building shells. By building your own, you’ll become more familiar with these interfaces and you’ll probably learn more about other shells as well. This assignment will be due <strong>October 2, 2019 at 9:00 pm PST</strong>.

<h1><a name="_Toc8773"></a>1           Getting started</h1>

Log in to your Vagrant Virtual Machine and run:

$ cd ~/code/personal/

$ git pull staff master $ cd hw2

We have added starter code for your shell and a simple Makefile in the hw2 directory. It includes a string tokenizer, which splits a string into words. In order to run the shell:

$ make

$ ./shell

In order to terminate the shell after it starts, either type exit or press CTRL-D.

<h1><a name="_Toc8774"></a>2           Add Support for cd and pwd</h1>

The skeleton code for your shell has a <strong>dispatcher </strong>for “built-in” commands. Every shell needs to support a number of built-in commands, which are functions in the shell itself, not external programs. For example, the exit command needs to be implemented as a built-in command, because it exits the shell itself. So far, the only two built-ins supported are ?, which brings up the help menu, and exit, which exits the shell.

<strong>Add a new built-in </strong>pwd <strong>that prints the current working directory to standard output. Then, add a new built-in </strong>cd <strong>that takes one argument, a directory path, and changes the current working directory to that directory. </strong><em>Hint</em>: Use chdir and getcwd.

Once you’re done, push your code to the autograder. In your VM:

$ git add shell.c

$ git commit -m “Finished adding basic functionality into the shell.”

$ git push personal master

You should commit your code periodically and often so you can go back to a previous version of your code if you want to.

<h1><a name="_Toc8775"></a>3           Program Execution</h1>

If you try to type something into your shell that isn’t a built-in command, you’ll get a message that the shell doesn’t know how to execute programs. Modify your shell so that it can execute programs when they are entered into the shell. The first word of the command is the name of the program. The rest of the words are the command-line arguments to the program.

For this step, you can assume that the first word of the command will be <strong>the full path to the program</strong>. So instead of running wc, you would have to run /usr/bin/wc. In the next section, you will implement support for simple program names like wc. But you can pass some autograder tests by only supporting full paths.

You should use the functions defined in tokenizer.c for separating the input text into words. You do not need to support any parsing features that are not supported by tokenizer.c. Once you implement this step, you should be able to execute programs like this:

$ ./shell

0: /usr/bin/wc shell.c

77    262   1843 shell.c

1: exit

When your shell needs to execute a program, it should fork a child process, which calls one of the exec functions to run the new program. The parent process should wait until the child process completes and then continue listening for more commands.

<h1><a name="_Toc8776"></a>4           Path Resolution</h1>

You probably found that it was a pain to test your shell in the previous part because you had to type the full path of every program. Luckily, every program (including your shell program) has access to a set of “environment variables”, which is structured as a hashtable of string keys to string values. One of these environment variables is the PATH variable. You can print the PATH variable of your current environment on your Vagrant VM: (use bash for this, not your homemade shell!)

$ echo $PATH

/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:…

When bash or any other shell executes a program like wc, it looks for a program called “wc” in each directory on the PATH environment variable and runs the first one that it finds. The directories on the path are separated with a colon.

Modify your shell so that it uses the PATH variable from the environment to resolve program names. Typing in the full pathname of the executable should still be supported. <strong>Do not use “execvp”. The autograder looks for “execvp”, and you won’t receive a grade if that word is found. </strong>Use execv instead and implement your own PATH resolution.

<h1><a name="_Toc8777"></a>5           Input/Output Redirection</h1>

When running programs, it is sometimes useful to provide input from a file or to direct output to a file. The syntax “[process] &gt; [file]” tells your shell to redirect the process’s standard output to a file. Similarly, the syntax ”[process] &lt; [file]” tells your shell to feed the contents of a file to the process’s standard input.

Modfiy your shell so that it supports redirecting stdin and stdout to files. You do not need to support redirection for shell built-in commands. You do not need to support stderr redirection or appending to files (e.g. “[process] &gt;&gt; [file]”). You can assume that there will always be spaces around special characters &lt; and &gt;. Be aware that the “&lt; [file]” or “&gt; [file]” are NOT passed as arguments to the program.

<h1><a name="_Toc8778"></a>6           Signal Handling and Terminal Control</h1>

Most shells let you stop or pause processes with special keystrokes. These special keystrokes, such as Ctrl-C or Ctrl-Z, work by sending signals to the shell’s subprocesses. For example, pressing CTRL-C sends the SIGINT signal which usually stops the current program, and pressing CTRL-Z sends the SIGTSTP signal which usually sends the current program to the background. Recall that your terminal window is running a shell program itself. The shell must make sure that these keystrokes do not stop the shell program itself. If you try these keystrokes in your homemade shell, the signals are sent directly to the shell process itself. This means that attempting to CTRL-Z a subprocess of your shell, for example, will also stop the shell itself. We want to have the signals affect only the subprocesses that our shell creates.

<h2><a name="_Toc8779"></a>6.1         Example: Shells in Shells</h2>

On your Vagrant VM, you’ll be executing a short series of commands in order to better understand the correct behavior. We’ll primarily be making use of two commands, ps and jobs. Recall that ps gives you information about all processes running on the system, while jobs gives you a list of jobs that the current shell is managing. Enter the following commands in your terminal, and you should see similar behavior:

<table width="209">

 <tbody>

  <tr>

   <td width="119">$ ps</td>

   <td width="91"> </td>

  </tr>

  <tr>

   <td width="119">PID TTY</td>

   <td width="91">TIME CMD</td>

  </tr>

  <tr>

   <td width="119">20970 ttys002 $ sh sh-3.2$ ps</td>

   <td width="91">0:01.30 -bash</td>

  </tr>

  <tr>

   <td width="119">PID TTY</td>

   <td width="91">TIME CMD</td>

  </tr>

  <tr>

   <td width="119">20970 ttys002</td>

   <td width="91">0:00.63 -bash</td>

  </tr>

  <tr>

   <td width="119">22323 ttys004</td>

   <td width="91">0:00.01 sh</td>

  </tr>

 </tbody>

</table>

At this point, we have started a sh shell within our bash shell.

sh-3.2$ cat hello hello world world

^Z

[1]+ Stopped(SIGTSTP)     cat

sh-3.2$ ps

<table width="209">

 <tbody>

  <tr>

   <td width="119">PID TTY</td>

   <td width="91">TIME CMD</td>

  </tr>

  <tr>

   <td width="119">20970 ttys004</td>

   <td width="91">0:00.63 -bash</td>

  </tr>

  <tr>

   <td width="119">22323 ttys004</td>

   <td width="91">0:00.02 sh</td>

  </tr>

  <tr>

   <td width="119">22328 ttys004</td>

   <td width="91">0:00.01 cat</td>

  </tr>

 </tbody>

</table>

Notice how sending a CTRL-Z while the cat program was running did not suspend the sh nor the bash programs.

sh-3.2$ jobs

[1]+ Stopped(SIGTSTP)     cat

sh-3.2$ exit

$ ps

PID TTY     TIME CMD 20970 ttys004 0:00.65 -bash

Since exit terminates the shell program to terminate, we terminated the sh program. Enter exit again and your terminal will close.

Before we explain how you can achieve this effect, let’s discuss some more operating system concepts.

<h2><a name="_Toc8780"></a>6.2         Process Groups</h2>

We have already established that every process has a unique process ID (pid). Every process also has a (possibly non-unique) process group ID (pgid) which, by default, is the same as the pgid of its parent process. Processes can get and set their process group ID with getpgid(), setpgid(), getpgrp(), or setpgrp().

Keep in mind that, when your shell starts a new program, that program might require multiple processes to function correctly. All of these processes will inherit the same process group ID of the original process. So, it may be a good idea to put each shell subprocess in its own process group, to simplify your bookkeeping. When you move each subprocess into its own process group, the pgid should be equal to the pid.

<h2><a name="_Toc8781"></a>6.3         Foreground Terminal</h2>

Every terminal has an associated “foreground” process group ID. When you type CTRL-C, your terminal sends a signal to every process inside the foreground process group. You can change which process group is in the foreground of a terminal with “tcsetpgrp(int fd, pid_t pgrp)”. The fd should be 0 for “standard input”.

<h2><a name="_Toc8782"></a>6.4         Overview of Signals</h2>

Signals are asynchronous messages that are delivered to processes. They are identified by their signal number, but they also have somewhat human-friendly names that all start with SIG. Some common ones include:

<ul>

 <li><strong>SIGINT </strong>– Delivered when you type CTRL-C. By default, this stops the program.</li>

 <li><strong>SIGQUIT </strong>– Delivered when you type CTRL-. By default, this also stops the program, but programs treat this signal more seriously than SIGINT. This signal also attempts to produce a core dump of the program before exiting.</li>

 <li><strong>SIGKILL </strong>– There is no keyboard shortcut for this. This signal stops the program forcibly and cannot be overridden by the program. (Most other signals can be ignored by the program.)</li>

 <li><strong>SIGTERM </strong>– There is no keyboard shortcut for this either. It behaves the same way as SIGQUIT.</li>

 <li><strong>SIGTSTP </strong>– Delivered when you type CTRL-Z. By default, this pauses the program. In bash, if you type CTRL-Z, the current program will be paused and bash (which can detect that you paused the current program) will start accepting more commands.</li>

 <li><strong>SIGCONT </strong>– Delivered when you run fg or fg %NUMBER in bash. This signal resumes a paused program.</li>

 <li><strong>SIGTTIN </strong>– Delivered to a background process that is trying to read input from the keyboard. By default, this pauses the program, since background processes cannot read input from the keyboard. When you resume the background process with SIGCONT and put it in the foreground, it can try to read input from the keyboard again.</li>

 <li><strong>SIGTTOU </strong>– Delivered to a background process that is trying to write output to the terminal console, but there is another foreground process that is using the terminal. Behaves the same as SIGTTIN by default.</li>

</ul>

In your shell, you can use kill -XXX PID, where XXX is the human-friendly suffix of the desired signal, to send any signal to the process with process id PID. For example, kill -TERM PID sends a SIGTERM to the process with process id PID.

In C, you can use the sigaction system call to change how signals are handled by the current process. The shell should basically ignore most of these signals, whereas the shell’s subprocesses should respond with the default action. For example, the shell should ignore SIGTTOU, but the subprocesses should not. Beware: forked processes will inherit the signal handlers of the original process. Reading <a href="http://man7.org/linux/man-pages/man2/sigaction.2.html">man 2 sigaction</a> and <a href="http://man7.org/linux/man-pages/man7/signal.7.html">man 7 signal</a> will provide more information. Be sure to check out the SIG_DFL and SIG_IGN constants. For more information on process group and terminal signaling, please go through this tutorial <a href="https://www.usna.edu/Users/cs/aviv/classes/ic221/s16/lec/17/lec.html">here</a><a href="https://www.usna.edu/Users/cs/aviv/classes/ic221/s16/lec/17/lec.html">.</a>

<strong>Your task is to ensure that each program you start is in its own process group. When you start a process, its process group should be placed in the foreground. Stopping signals should only affect the foregrounded program, not the backgrounded shell.</strong>

<h1><a name="_Toc8783"></a>7           Threads and Concurrency</h1>

We have included a program for exploring threads and concurrency in the starter files. To build the program, run

$ make threads

The program accepts 3 arguments: a number of threads, a count, and a boolean for toggling log statements. It creates the specified number of threads, and each one tries to acquire a global lock. A thread will continue to try to acquire it until the total number of times the lock has been acquired reaches the specified count. The program will report the number of times each thread was able to acquire the lock. In a file called threads.txt, answer the following questions:

<ol>

 <li>Run ./threads 3 10 True twice and copy the output you see from each run. Is the program’s output the same each time it is run? Why?</li>

 <li>The number of times each thread increments the count is an indicator of the amount of processingtime it got and the work it got done. Run ./threads 4 100 True a few times. How fair is the allocation? Turn off the watch and run it a few times. Does it change the fairness? Explain what you think is going on.</li>

 <li>Run ./threads 4 100000 False a couple of times to get a rough idea of the fairness. How does this compare with what you saw earlier? Explain.</li>

 <li>Sometimes the program reports that there were more lock acquisitions than the number that weinitially specified. Copy the lines of code that cause this behavior and explain.</li>

 <li>Notice the average time per lock. Does it change in the unfair runs? You can see in the code howto time a section of computation. Write a little program to sum all the elements of an array. How large a problem can you do in the time to acquire and release a lock?</li>

 <li>Try building and running this on your native machine (if you’ve got a development environmentset up) or on an instructional machine. What differences do you notice?</li>

</ol>

<h1><a name="_Toc8784"></a>8           Stretch: Foreground and Background Processes</h1>

<h2><a name="_Toc8785"></a>8.1         Background Processes</h2>

So far, your shell waits for each program to finish before starting the next one. Many shells allow you run a command in the background by putting an “&amp;” at the end of the command line. After the background program is started, the shell allows you to start more processes without waiting for background process to finish.

Modify your shell so that it runs commands that end in an “&amp;” in the background. Once you’ve implemented this feature, you should be able to run programs in the background with a command such as “/bin/ls &amp;”.

You should also add a new built-in command wait, which waits until <em>all </em>background jobs have terminated before returning to the prompt.

You can assume that there will always be spaces around the &amp; character. You can assume that, if there is a &amp; character, it will be the last token on that line.

<h2><a name="_Toc8786"></a>8.2         Foreground/Background Switching</h2>

Most shells allow for running processes to be toggled between running in the foreground versus in background. You can optionally add two built-in commands to support this:

<ul>

 <li>“fg [pid]” – Move the process with id pid to the foreground. The process should resume if it was paused. If pid is not specified, then move the most recently launched process to the foreground.</li>

 <li>“bg [pid]” – Resume a paused background process. If pid is not specified, then resume the most recently launched process.</li>

</ul>

You should keep a list of all existing processes (whether they are in the foreground or background) and their pids. Inside this list, you should also keep a “struct termios” to store the terminal settings of each program.