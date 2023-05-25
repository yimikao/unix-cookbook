# Process Environment

### main Function

A C program starts execution with a function called main. \
However, when a C program is executed by the kernel—by one of the exec functions,a special start-up routine is called before the main function is called. This start-up routine is not part of your actual code but is provided by the system (specifically, the kernel). When you compile and link your C program, the executable file generated specifies the start-up routine as the starting address for the program. The start-up routine performs tasks such as initializing the runtime environment, setting up command-line arguments and the program's environment variables, allocating memory, and performing other necessary setup tasks. After this initialization, the start-up routine then calls the main function, which serves as the entry point for your program's logic.

### Process Termination

8 ways for a process to terminate: \
\
Normal:
1. Return from `main`
2. Calling `exit`
3. Calling `_exit` / `_Exit`
4. Return of last thread from it's start routine <sub>(11)</sub>
5. Calling `pthread_exit` from the last thread <sub>(11)</sub>

Abnormal:
6. Calling `abort` <sub>(10)</sub>
7. Receipt of a signal <sub>(10)</sub>
8. Response of last thread to a cancellation request <sub>(11)</sub>
