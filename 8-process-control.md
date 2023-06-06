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

Both the child and the parent continue executing wuth the following instructions after `fork`. **The child is a copy of the parent. (Gets a copy of parent's data space, heap, stack).** This is a separate copy for the child, the parent and the child do not share these portions of memory. They share the text segment however. \
\
Modern implementations don’t perform a complete copy of the parent’s data, stack, and heap, since a `fork` is often followed by an `exec`. Instead, a technique called `copy-on-write` (COW) is used. These regions are shared by the parent and the child and have their protection changed by the kernel to read-only. If either process tries to modify these regions, the kernel then makes a copy of that piece of memory only, typically a ‘‘page’’ in a virtual memory system. \
\
If its required for child and parent to sync, some form of interprocess communication is requyired. 

## File Sharing
One characteristic of `fork` is that **all file descriptors that are open in the parent are duplicated in the child.** They share a file table entry for every **open descriptor**. Meaning even though, the fd are duplicates the file pinters refer to the same files accross both processes. 

- parent
fd 0 -> 0x1
- child
fd 0 -> 0x1 

The parent anf chilf **share the same file offset.** This is because if child write into STDOUT while parent waits for example, parent can continue writing to it knowing that it will be appended to what child wrote. If they didnt share the same file offset, this would be difficult, requiring explicit actions. \
\
Besides open files, other properties of parent are inherited by the child; process group ID, session ID etc \
\
**Differences between parent and child are:** diff fork return values, process IDs, parent procees ID, child's tms_utime and its variants, file locks set by parent are not inherited, pending alarms and pending signals are emptied. \
\
**`fork` can fail for two reasons:**
1. too many processes in system
2. total number of processes for this real yser ID exceeds system's limit i.e CHILD_MAX(max no of simul procs per real user ID)

**Two uses for fork:**
1. Process want to duplicate itself e.g hand an operation to a child e.g network request handling
2. Process intends to execute a different program.

## vFork Function

## exit Function

`exit` is associated with **normal program termination**. \
\
a program can terminate normally in five ways:
1. `return` from `main()
2. calling `exit()`. it calls all regiustered **exit handlers** by calling `atexit` and closing all standard I/O streams. It is defined by ISO C and Because ISO C does not deal with file descriptors, multiple processes (parents and children), and job control, the definition of this function is incomplete for a UNIX system.
3. calling `_exit` or `_Exit`. On UNIX, they are synonymoud and do not flush standard IO streams. `_exit` is called by `exit` and handles the UNIX system-specific details since its defined by POSIX.1
4. `return` from start routine of last thread in process. The prcess exits with termination status of 0, not return value of thread.
5. calling `pthread_exit` frm the last thread in process. exit status same as above.

**Abnormal termination occurs in three forms:**
1. calling `abort`. Its a special case of next form, as it generates the `SIGABRT` sugnal.
2. process recieves certain signals. sig could be gened by process e.g abort, other process, or kernel (e.g zero division by process)
3. last thread responds to **cancelllation request** from another.


**Regardless of how process terminates, the same code in the kernel is eventually executed. The code closes all open descriptors for the process, releases the memory it was using etc** \
\
For all termination cases, the terminating process notifies parent how it did so, in any case, parent can get the termination status from `wait` or `waitpid`:
1. For the exit functions, by passing an exit status as argument.
2. For abnormal, kernel generates a termination status.

Diff btw exit status(arg to exit or main return) and termination status is **exit stat is converted to term stat by kernel at `_exit` in `exit`** \
\
If parent teminates before child, `init` process inherits child (kernel changes parent ID to 1). In the other case, kernel keeps child info for when parent calls `wait` (unzombieing a child process. NOTE: init also calls wait for its created and inherited procs).

## wait and waitpid Functions
waitpid is a variant of wait. We tell which child terminated, because process ID is returned by them. 

```c
#include <sys/wait.h>
pid_t wait(int *status);
```
\
**When a process terminates normally/abnormally kernel notifies the parent by sending the `SIGCHLD` signal to the parent.** Child termination is async so is signal. Parent can ignore or provide signal handler function. \
\
Process that calls `wait[pid]` can:
1. block if all its children are still running (has nada to report)
2. return at once with termination status of a child, if a child has terminated and is waiting for its termination status to be fetched
3. return at once with error if no child


