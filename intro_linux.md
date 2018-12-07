# Very quick intro to Linux
This content is to help students enjoy using Linux, starting with the very basics. There are many Linux tutorials already written, this one is tailored to our blockchain development course. Corrections and additional requests are welcomed.
Code blocks represent commands and their outputs are in new lines.

## Contents
[OS commands](#basic-os-commands)
[Monitoring the OS](#monitoring-the-os)
[Manipulating processes](#manipulating-processes)
[Manipulating files and folders](#manipulating-files-and-folders)
[Access the Internet from the command line](#access-the-internet-from-command-line)

# Basic OS commands
Manuals for most commands are available via `man`. Example for finding documentation about the command `pwd`:
```
> man pwd
NAME       pwd - print name of current/working directory
SYNOPSIS   pwd [OPTION]...
...
```
Where am I? To find out your current directory, use `pwd`:
```
> pwd
/home/user
```
Which Linux release and version am I using? This information is available in the `os-release` file available in all Linux distributions:
```
> cat /etc/os-release
NAME=Fedora
VERSION="28 (Twenty Eight)"
ID=fedora
VERSION_ID=28
...
```
More info about Linux kernel name and release, hostname, date, time, hardware platform and such is available with `uname` command:
```
> uname -a
Linux work 4.14.74-1.pvops.qubes.x86_64 #1 SMP Mon Oct 8 17:14:24 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```
For CPU architecture, there is a standard `lscpu` command:
```
> lscpu
Architecture:        x86_64
CPU op-mode(s):      32-bit, 64-bit
Byte Order:          Little Endian
CPU(s):              6
On-line CPU(s) list: 0-5
Thread(s) per core:  1
Core(s) per socket:  6
Socket(s):           1
NUMA node(s):        1
Vendor ID:           GenuineIntel
CPU family:          6
Model:               142
Model name:          Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz
Stepping:            10
CPU MHz:             2112.058
...
```
Redirect output of one command to the next (a.k.a. pipe it) with `|`. 
Redirect output to a file with `>`.
Example: let's issue the command pwd and save its output in a file pwd.txt:
```
> pwd > pwd.txt
```
Redirect output to a black hole (null device file), without any way of getting the data back - just redirect it to `/dev/null`. It is often used when we want to discard some parts of data streams.
Example: let's issue the command pwd, save its output in a file and send it to a location where it is permanently disposed:
```
> pwd > pwd.txt 
> ls -a
.  ..  pwd.txt
> pwd.txt > /dev/null
> ls
```
Standard input (stdin, 0), standard output (stdout, 1) and standard error (stderr, 2) are special descriptors, used for routing data streams of commands. Say we want to redirect all errors from an application to the standard output on the console - we would redirect stderr to stdout with `2>&1`.

Limit output of commands with `grep`, very useful if you are only concerned with a particular substring in each output line. grep can be used to output lines matching a pattern in a more general way, from reading of any input, e.g. files etc.
Let's issue a command with many lines of output and filter its output with grep to only display lines which contain the word "python":
```
> ps -aux | grep python
user       710  0.0  0.7 252204 13080 ?        S    19:22   0:00 /usr/bin/python3 /usr/lib/qubes/icon-sender
user      2178  0.0  0.0 215640  1016 pts/0    S+   21:21   0:00 grep --color=auto python
```
Copy & Paste in command line with `Ctrl + Insert` and `Shift + Insert`. This comes handy often, so do remember it :)

# Monitoring the OS
Monitor processes, memory usage, CPU states etc. with `top`:
```
> top
top - 20:43:05 up  1:21,  0 users,  load average: 0.00, 0.00, 0.00
Tasks: 146 total,   1 running,  97 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.1 us,  0.1 sy,  0.0 ni, 99.8 id,  0.0 wa,  0.0 hi,  0.0 si,  0.1 st
KiB Mem :  1617448 total,   424416 free,   435332 used,   757700 buff/cache
KiB Swap:  1048572 total,  1048572 free,        0 used.   937756 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                 
  576 user      20   0  806708  75532  34920 S   0.7  4.7   0:04.72 Xorg                                    
 1243 user      20   0  831912  37824  27704 S   0.7  2.3   0:02.25 gnome-terminal-                         
    1 root      20   0  236012   9008   6884 S   0.0  0.6   0:00.74 systemd
...
```
Check disk space with `df` with an optional parameter `-h` to be more human-readable:
```
> df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda3      9.6G  7.3G  1.9G  80% /
/dev/xvdd       477M  291M  161M  65% /usr/lib/modules/4.14.74-1.pvops.qubes.x86_64
devtmpfs        3.9G     0  3.9G   0% /dev
tmpfs           1.0G     0  1.0G   0% /dev/shm
...
```

# Manipulating processes
Kill a process by gently sending it a SIGINT signal, so that it can intercept it and shutdown gracefully - use `Ctrl + c`.

List current processes with `ps`, you can add `-aux` to see all of them:
```
> ps -aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.5 236012  9008 ?        Ss   19:21   0:00 /sbin/init
root         2  0.0  0.0      0     0 ?        S    19:21   0:00 [kthreadd]
root         4  0.0  0.0      0     0 ?        I<   19:21   0:00 [kworker/0:0H]
root         6  0.0  0.0      0     0 ?        I<   19:21   0:00 [mm_percpu_wq]
root         7  0.0  0.0      0     0 ?        S    19:21   0:00 [ksoftirqd/0]
root         8  0.0  0.0      0     0 ?        I    19:21   0:00 [rcu_sched]
...
```
Kill a process with a particular PID. To help you finding the correct PID to kill, remember to use `ps -aux | grep YOUR_PROCESS_NAME` or similar. Say we want to kill a process with ID 123:
```
> kill -KILL 123
```
Run a process in the background, so that it doesn't clutter your console, by putting `&` after the command. If you want that the process continues even after you close the terminal or logout, use `nohup` (no-hang-up). For example, let's run geth in the background and be sure that it runs *(yeah, I know)* when we logout from the server:
```
> nohup geth &
```

# Manipulating files and folders

Create a folder "tmpfolder" with `mkdir` (make directory). Wait, what is a directory? A directory means the same as folder, actually folders used to be called directories, but then came Windows with a graphical user interface and it seemed fancy to organize files in a more intuitive way, never mind.
Check where we are and then create a subfolder at current location:
```
> pwd
/home/user
> mkdir tmpfolder
```
Go to that folder with `cd` (change directory) and check that our current location really changed:
```
> cd tmpfolder
> pwd
/home/user/tmpfolder
```
Create a file "tmpfile.txt" with `touch`:
```
> touch tmpfile.txt
```
List files at current locations with `ls`, but first let's create another directory "t" just to see different outputs:
```
> mkdir t
> ls
t  tmpfile.txt
```
Prettier list of files is available with using a switch `-lpa` (-l for more descriptive output, -p to append / in front of directories, -a to include objects starting with .):
```
> ls -lpa
total 12
drwxrwxr-x  3 user user 4096 Dec  3 21:08 ./
drwx------ 41 user user 4096 Dec  3 21:08 ../
drwxrwxr-x  2 user user 4096 Dec  3 21:08 t/
-rw-rw-r--  1 user user    0 Dec  3 21:08 tmpfile.txt
```
Modify the file from the command line with `vim`. Once in vim, press `i` to activate the "insert mode". After you're done modifying the file, press `Esc` to leave the insert mode and type `:wq` to "write" the changes to a file and "quit".
```
> vim tmpfile.txt
```
Change file permissions with `chmod`. For a dummy example, let's make a file executable with `+x`:
```
> chmod +x tmpfile.txt
```
Want to run an application without specifying the full path? Put an alias into `~/.bashrc`. 
An example to specify all executables in the directory /Programs be immediately accessible:
```
> vim ~/.bashrc
```
Then add at the end:
```
# User specific aliases and functions
export PATH=$PATH:$HOME/Programs
```

# Access the Internet from the command line
Download a file from the Internet with `wget`:
```
> wget https://bitcoin.org/bin/bitcoin-core-0.17.0.1/bitcoin-0.17.0.1-x86_64-linux-gnu.tar.gz
-2018-12-03 22:16:58--  https://bitcoin.org/bin/bitcoin-core-0.17.0.1/bitcoin-0.17.0.1-x86_64-linux-gnu.tar.gz
Resolving bitcoin.org (bitcoin.org)... 138.68.248.245
Connecting to bitcoin.org (bitcoin.org)|138.68.248.245|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 28263053 (27M) [application/octet-stream]
Saving to: ‘bitcoin-0.17.0.1-x86_64-linux-gnu.tar.gz’
...
```

Browse websites with `links`. After issuing the command, the URL opens with the webpage and you can use keys for navigation, it is pretty neat.
```
> links https://bitcoin.org/en/developer-reference
```
