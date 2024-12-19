# Introduction to UNIX system calls

- Applications interact with the Operating System through system calls, which will be a major focus. These system calls are fundamental to understanding UNIX systems and will be used extensively in labs.

## The xv6 Operating System

- xv6 serves as a simplified UNIX-like system with similar structure to Linux but is more digestible. It was chosen because it is:

  - Open source with clear documentation
  - Has a clean design and is widely used
  - Provides insight into Linux internals
  - Runs on RISC-V architecture
  - Can be run under the qemu machine emulator

- xv6 serves two main purposes in 6.S081:

1. Demonstrates core functions like virtual memory, multi-core processing, and interrupts
2. Provides the foundation for most lab assignments

## System Call Examples

### Copy Example (copy.c)

- This program demonstrates basic input/output operations using read() and write() system calls. Key points:

  - Uses file descriptors (FDs) to specify input/output targets
  - FD 0 is standard input, FD 1 is standard output by convention
  - Read/write operations work with 8-bit bytes
  - Return values indicate bytes processed or errors (-1)

### File Creation (open.c)

- The open() system call:

  - Creates files and returns a file descriptor
  - Uses per-process FD tables maintained by the kernel
  - Different processes maintain separate FD namespaces
  - Documentation available in Figure 1.2 of xv6 book or UNIX man pages

## System Call Mechanics

- When a program makes a system call:

  1. Special instruction triggers hardware response
  2. Hardware saves user registers
  3. Privilege level is increased
  4. Execution jumps to kernel entry point
  5. Kernel executes system call implementation
  6. User registers are restored
  7. Privilege level is reduced
  8. Program execution resumes

## Process Management

### Process Creation (fork.c)

- The fork() system call:

  - Creates a new process by copying the calling process
  - Duplicates instructions, data, registers, file descriptors, and current directory
  - Returns different values in parent (pid) and child (0) processes
  - Each process gets a unique process ID (pid)

### Program Execution (exec.c)

- The exec() system call:

  - Replaces current process with an executable file
  - Preserves file descriptors
  - Loads new instructions and memory
  - Passes command-line arguments to the new program

### Combined Usage (forkexec.c)

- Common UNIX pattern:

  1. Fork child process
  2. Execute command in child
  3. Parent waits for child completion
  4. Exit status indicates success (0) or failure (1)

## I/O Redirection and Pipes

### Redirection (redirect.c)

- Implements output redirection (e.g., echo hello > out):

  - Uses fork and exec with FD manipulation
  - Takes advantage of FD inheritance across exec
  - Allows programs to remain unaware of redirection

### Pipes (pipe1.c, pipe2.c)

- Pipes enable inter-process communication:

  - Created using pipe() system call
  - Provide read and write file descriptors
  - Kernel maintains pipe buffers
  - Combine well with fork for command pipelines

## Directory Operations

- Directory contents can be accessed through system calls:

  - Directories can be opened and read like files
  - "." represents current directory
  - See ls.c for implementation details

### Summary

- UNIX provides core abstractions for I/O, file systems, and processes
- Interfaces use simple integers and I/O buffers
- Abstractions are designed to work together effectively
- These fundamentals form the basis for the first lab assignment
