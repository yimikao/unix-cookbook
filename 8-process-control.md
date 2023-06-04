# Process Control

**Process Creation, Program Execution and Process Termination.**

## Process Identifier

Every process has a unique id. \
\
Process ID 0 is usually the scheduler process , AKA **swapper**. No program on disk corresponds to this process; its a part of the kernel and known as a system process. \
\
Process ID 1 is usually the init process, its invoked by the kernel at end of the bootstrap procedure. The program file is `/sbin/init`. Its a normal user process, not a system process within the kernel. \
\
UNIX implentationshave their own set of **kernel process that provide OS services.** \
\
Functions that return process identifiers: 
```c
#include <unistd.h>
// This function has other variants
pid_t getpid(void) // returns PID of calling process
```
<br>

## fork Function
An existing process can create a new one with `fork`. \
```c
pid_t fork(void) // It is called once, but returns twice. returns 0 in child, ID in parent, -1 error
```
<br/>
Both the child and the parent