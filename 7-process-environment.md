# Process Environment

## main Function

A C program starts execution with a function called main. \
However, when a C program is executed by the kernel—by one of the exec functions,a special start-up routine is called before the main function is called. This start-up routine is not part of your actual code but is provided by the system (specifically, the kernel). When you compile and link your C program, the executable file generated specifies the start-up routine as the starting address for the program. The start-up routine performs tasks such as initializing the runtime environment, setting up command-line arguments and the program's environment variables, allocating memory, and performing other necessary setup tasks. After this initialization, the start-up routine then calls the main function, which serves as the entry point for your program's logic.

## Process Termination and exit()

8 ways for a process to terminate: \
\
**Normal:**
1. Return from `main`
2. Calling `exit`
3. Calling `_exit` / `_Exit`
4. Return of last thread from it's start routine <sub>(11)</sub>
5. Calling `pthread_exit` from the last thread <sub>(11)</sub>

**Abnormal:**
1. Calling `abort` <sub>(10)</sub>
2. Receipt of a signal <sub>(10)</sub>
3. Response of last thread to a cancellation request <sub>(11)</sub>

The start-up routine is also coded to call `exit` if `main` returns. It in assembly but it's C variant could have been `exit(main(argc, argv));`

**Note:** Three **functions** terminate a program normally: (These are that are explicit functions) \
`_exit` and `_Exit`, return to the kernel immediately, while `exit`, performs certain cleanup processing and then returns to the kernel. (clean shutdown of the standard I/O library: the `fclose` function is called for all open streams, causing all buffered output data to be flushed (written to the file)). \
**Any process can cause a program to be executed, wait for the process to complete, and then fetch its exit status.**


## Exit Handlers with atExit()

A process can register at least 32 functions that are automatically called by
exit. These are called `exit handlers` and are registered by calling the `atexit` function. The exit function calls these functions in reverse order of their registration. Each function is called as many times as it was registered. With ISO C and POSIX.1, exit first calls the exit handlers and then closes (via `fclose`) all open streams. POSIX.1 extends the ISO C standard by specifying that any exit handlers installed will be cleared if the program calls any of the `exec` family of functions.

The only way a program can be executed by the kernel is if one of the exec functions is called. The only way a process can voluntarily terminate is if `_exit` or `_Exit` is called, either explicitly or implicitly (by calling `exit`). A process can also be involuntarily terminated by a signal.

**Note that we don’t call exit in main; instead, we return from main. (C startup routine)**


## Command Line Arguments

## Environment List

## Memory Layout of a C Program

C program consists of the ffl pieces:

- Text Segment:
consisting of instructions the CPU executes. Usually, the text segment is sharable so that only a single copy needs to be in memory for frequently executed programs, such as text editors, the C compiler, the shells etc. It is `READ-ONLY`.

- Initialized Data Segment:
AKA Data Segment. Contains vars specifically initialized in the program, e.g \


```c
int maxv = 100; 
``` 
This declaration **appearing outside any function** causes this variable to be stored in the initialized data segment with its initial value.

- Uninitialized Data Segment:
AKA "bss segment" . Data in this segment is initialized by the kernel to arithmetic 0 or null pointers before the program starts executing. 

```c
int sum[100];
```

This decalration **appearing outside any function** causes this variable to be stored in the uninitialized data segment.

- Stack:
Where automatic variables are stored, along with information that is saved each time a function is called. **Each time a function is called, the address of where to return to and certain information about the caller’s environment, such as some of the machine registers, are saved on the stack. The newly called function then allocates room on the stack for its automatic and temporary variables.** This is how recursive functions in C can work. Each time a recursive function calls itself, a new stack frame is used, so one set of variables doesn’t interfere with the variables from another instance of the function.

- Heap:
where dynamic memory allocation usually takes place. located btw UDS and Stack.


## Typical Memory Layout

(Low Address) Text -> IDS -> UDS -> Heap -> Stack ->  (High Address)

## Shared Libs

## Memory Allocation

ISO C three functions: `malloc`, `calloc`, `realloc`. \
`free` call deallocated the space pointed to. \
This freed space **is usually put into a pool of available memory** and can be allocated in a later call to one of the three alloc functions. \
They are implemented with `sbrk()` syscall, which expands(or contracts) the heap of the process. Although `sbrk` can expand or contract the memory of a process, most versions of `malloc` and `free` never decrease their memory size. The space that we free is available for a later allocation, but the freed space is not usually returned to the kernel; instead, that space is kept in the malloc pool.

## Environment Variables

## setjmp() and longjmp() Functions

## getrlimit() and setrlimit() Functions

