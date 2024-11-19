# Introduction to UNIX system calls

Applications see the O/S via system calls; that interface will be a big focus.
Let's start by looking at how programs use system calls.

**some examples, and how they run on xv6**.
- xv6 has similar structure to UNIX systems such as Linux but much simpler -- you'll be able to digest all of xv6 accompanying book explains how xv6 works, and why UNIX?
- opensource, well documented, clean design, widely used studying xv6 will help you ever need to look inside Linux xv6 has two roles:
    - example of core functions: virtual memory, multi-core, interrupts, &c starting point for most of the labs. xv6 runs on RISC-V, as in 6.004 you'll run xv6 under the qemu machine emulator.
    - example: copy.c, copy input to output $ copy. copy.c is written in C
        `Kerninghan and Ritchie (K&R) book is good for learning C`. First read()/write() argument is a "file descriptor" (fd) must previosly have been opened, an FD connects to a file/device/socket/&c a UNIX convention: fd 0 is "standard input", 1 is "standard output" third argument is the maximum number of bytes to read, read() may read less, but not more. return value: number of bytes actually read, or -1 for error note: copy.c does not care about the format of the data. UNIX I/O is 8-bit bytes. Interpretation is application specific, e.g. database records, C source, &c where do file descriptors come from?
    - example: open.c, create a file
        $ open
        $ cat output.txt
        open() creates a file, returns a file descriptor (or -1 for error) FD is a small integer. FD indexes into a per-process table maintained by kernel. different processes have different FD name-spaces i.e. FD 1 often means different things to different processes these examples ignore errors -- don't be this sloppy

## what happens when a program calls a system call like open()?

- Looks like a function call, but it's actually a special instruction. hardware saves some user registers. hardware increases privilege level. hardware jumps to a known "entry point" in the kernel now running C code in the kernel. kernel calls system call implementation. open() looks up name in file system, it might wait for the disk, it updates kernel data structures (cache, FD table) restores user registers. reduce privilege level, jump back to calling point in the program, which resumes.

- I've been typing to UNIX's command-line interface, the shell.
  the shell prints the "$" prompts.
  the shell lets you run UNIX command-line utilities
    useful for system management, messing with files, development, scripting
    $ ls
    $ ls > out
    $ grep x < out
  UNIX supports other styles of interaction too
    window systems, GUIs, servers, routers, &c.
  but time-sharing via the shell was the original focus of UNIX.
  we can exercise many system calls via the shell.

- example: fork.c, create a new process
  the shell creates a new process for each command you type, e.g. for
    $ echo hello
  the fork() system call creates a new process
    $ fork
  the kernel makes a copy of the calling process
    instructions, data, registers, file descriptors, current directory
    "parent" and "child" processes
  only difference: fork() returns a pid in parent, 0 in child
  a pid (process ID) is an integer, kernel gives each process a different pid
  thus:
    fork.c's "fork() returned" executes in *both* processes
    the "if(pid == 0)" allows code to distinguish
  ok, fork lets us create a new process
    how can we run a program in that process?

* example: exec.c, replace calling process with an executable file
  how does the shell run a program, e.g.
    $ echo a b c
  a program is stored in a file: instructions and initial memory
    created by the compiler and linker
  so there's a file called echo, containing instructions
  $ exec
  exec() replaces current process with an executable file
    discards instruction and data memory
    loads instructions and memory from the file
    preserves file descriptors
  exec(filename, argument-array)
    argument-array holds command-line arguments; exec passes to main()
    cat user/echo.c
    echo.c shows how a program looks at its command-line arguments

- example: forkexec.c, fork() a new process, exec() a program
  $ forkexec
  forkexec.c contains a common UNIX idiom:
    fork() a child process
    exec() a command in the child
    parent wait()s for child to finish
  the shell does fork/exec/wait for every command you type
    after wait(), the shell prints the next prompt
    to run in the background -- & -- the shell skips the wait()
  exit(status) -> wait(&status)
    status convention: 0 = success, 1 = command encountered an error
  note: the fork() copies, but exec() discards the copied memory
    this may seem wasteful
    you'll transparently eliminate the copy in the "copy-on-write" lab

- example: redirect.c, redirect the output of a command
  what does the shell do for this?
    $ echo hello > out
  answer: fork, change FD 1 in child, exec echo
  $ redirect
  $ cat output.txt
  note: open() always chooses lowest unused FD; 1 due to close(1).
  fork, FDs, and exec interact nicely to implement I/O redirection
    separate fork-then-exec give child a chance to change FDs before exec
    FDs provide indirection
      commands just use FDs 0 and 1, don't have to know where they go
    exec preserves the FDs that sh set up
  thus: only sh has to know about I/O redirection, not each program

- It's worth asking "why" about design decisions:
  Why these I/O and process abstractions? Why not something else?
  Why provide a file system? Why not let programs use the disk their own way?
  Why FDs? Why not pass a filename to write()?
  Why are files streams of bytes, not disk blocks or formatted records?
  Why not combine fork() and exec()?
  The UNIX design works well, but we will see other designs!

- example: pipe1.c, communicate through a pipe
  how does the shell implement
    $ ls | grep x
  $ pipe1
  an FD can refer to a "pipe", as well as a file
  the pipe() system call creates two FDs
    read from the first FD
    write to the second FD
  the kernel maintains a buffer for each pipe
    [u/k diagram]
    write() appends to the buffer
    read() waits until there is data

- example: pipe2.c, communicate between processes
  pipes combine well with fork() to implement ls | grep x
    shell creates a pipe,
    then forks (twice),
    then connects ls's FD 1 to pipe's write FD,
    and grep's FD 0 to the pipe
    [diagram]
  $ pipe2 -- a simplified version 
  pipes are a separate abstraction, but combine well w/ fork()

- example: list.c, list files in a directory
  how does ls get a list of the files in a directory?
  you can open a directory and read it -> file names
  "." is a pseudo-name for a process's current directory
  see ls.c for more details

### Summary

- We've looked at UNIX's I/O. file system, and process abstractions
- The interfaces are simple -- just integers and I/O buffers
- The abstractions combine well, e.g. for I/O redirection.
