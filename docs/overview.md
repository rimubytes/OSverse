# Overview

## Goals

- Understand operating system (O/S) design and implementation
- Hands-on experience extending a small O/S
- Hands-on experience writing systems software

### What is the purpose of an O/S?

- Abstract the hardware for convinience and portability
- Multiplex the hardware among many applications
- Isolate applications inorder to contain bugs
- Allow sharing among cooperating applications
- Control sharing for security
- Don't get in the way of high-performance
- Support a wide range of applications

### Organization: user/kernel

- User applications: vi, gcc, DB, &c
- kernel services
- h/w: CPU, RAM, disk, net, &c
- we care alot about the interfaces and internal kernel structure

### What services does an O/S kernel typically provide?

- process (a running program)
  - memory allocation
  - file contents
  - file names, directories
  - access control (security)
  - many others: users, IPC, network, time, terminals

### What's the application/kernel interface?

- "System calls"
- Examples, in C, from UNIX (e.g. Linux, macOS, FreeBSD):

        fd = open("out", 1);
        write(fd, "hello\n", 6);
        pid = fork();

- These look like function calls but they aren't

### Why is O/S design+implementation hard and interesting?

- unforgiving environment: quirky h/m, hard to debug
- many design tensions:
  - efficient vs abstract/portable/general-purpose
  - powerful vs simple interfaces
  - flexible vs secure
- features interact: 'fd = open(); fork()'
- uses are varied: laptops, smart-phones, cloud, virtual machines, embedded
- evolving hardware: NVRAM, multi-core, fast networks

### For people that

- care about what goes on under the hood
- like infrastructure
- need to track down bugs or security problems
- care about high performance
