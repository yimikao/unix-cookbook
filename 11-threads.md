# Threads

We can use multiple *threads* to perform multiple tasks within the environment of a single process. \
\
**All threads in a single process have access to the same process components. e.g file descriptors and memory address space.
Each thread has its own program counter, stack, and local variables, but they share the same memory space, file descriptors, and other resources of the process.**

## Thread Concepts

A thread consists of the information necessary to represent an execution context within a process. This includes the **thread ID** that identifies it within a process, a set of register values, a stack, a schedulig priority and policy, a signal mask, an **errno** variable (sec 1.7), and thread-specific data (12.6)\
\
Evrything within a process is sharable among the threads in a process, including the text of the executable program, the program's global and heap memory, the stacks, and file desciptors.

## Thread Indentification

Unlike PID which is unique in system, thread ID has significance only within the context of the process it belongs. \
\
Recall PID(`pid_t`) is a non-negative integer. FOr a thread ID(`pthread_t`), implementations are alowed to use a structure to rep the `pthread_t` data type, so portable implementations cant treat them as integers. Therefore, a fucntion must be used to compare two thread IDs. \
\
A thread obtains its own thread ID calling `pthread_self`. \
\

This function can be used with pthread_equal when a thread needs to identify data structures that are tagged with its thread ID. For example, a master thread might place work assignments on a queue and use the thread ID to control which jobs go to each worker thread. This situation is illustrated in Figure 11.1. A single master thread places new jobs on a work queue. A pool of three worker threads removes jobs from the queue. Instead of allowing each thread to process whichever job is at the head of the queue, the master thread controls job assignment by placing the ID of the thread that should process the job in each job structure. Each worker thread then removes only jobs that are tagged with its own thread ID. 

## Thread Creation (REVISIT)

Wuth `pthreads`, when a program runs, it starts out as a single process with a single thread of control. As the program runs, its behavior should be indistinguishable from the traditional process, until it creates more threads of control. \
\
Additional threads cn be created by calling the `pthread_create` function. \
\

```c
int pthread_create(pthread_t *tipd,
                    const pthread_attr_t *attr,
                    void *(*start_rtn)(void *),
                    void *arg);

// returns 0 if OK, error num on failure
```
The newly created thread starts running at the address of the **start_rtn** routine. This function takes a single argument, *arg*, a typeless pointer. To pass multiple args to *start_trhn*, you store them in a struct. \
\
When a thread is created, theres no quarantee if it runs first or the calling thread. **Newly created thread has access to the process address space and inherits calling thread's floating-point environment and signal mask; however, the set of pending signals for the thread is cleared.** \
\
**NOTE**: pthread funcs return an error code when they fail. They dont set `errno` like other POSIX functions. The per-thread copy of `errno` is provided only for compatibility with existing functions that use it. With threads, it is cleaner to return the error code from the func, therby restricting the scope of the error to the func that caused it, instread of relying on some global state that is changes as a side effect of the function. \
\
Altough theres no portable way to print thread ID, heres a small test prog that does, for insight:
```c
#include <pthread.h>
```

## Thread Termination (REVISIT)

If any thread in a process calls `exit`, `_Exit` or `_exit`, the entire process terminates. \
\
Similarly, when the default action is to terminate the process, a signal sent to a thread will terminate the entire process. \
\
**A single thread can exit in three ways**, thereby stopping its flow of control, without terminating the entire process.
1. The thread can simply return from the start routine. The return value is the thread's exit code.
2. The thread can be cancled by another thread in a same process
3. The thread can call `pthread_exit`.

```c
#include <pthread.h>
void pthread_exit(void *rval_ptr)
```
The *rval_ptr* arg is a typeless pointer, similar to the single arg passed to the start routine. This pointer is avaiable to other threads in the process by calling the `pthread_join` function.

```c
#include <pthread.h>
int pthread_join(pthread_t thread, void **rval_ptr)
// eturns 0 if OK, err on failure
```

The calling thread will block until the specified thread calls `pthread_exit`, returns from its start routine, or is canceled. If the thread simply returned from its start routine,  *rval_ptr* will contain the return code. If the thread was cancelled, the memory locatuon specified by *rval_ptr* is set to `PTHREAD_CANCELED`.\
By callinf `pthread_join`, we automatically place the thread with which we're joining in the detached state so that its resources can be recovered. If the thread was already in the detached state, `pthread_join` can fail, returning `EINVAL`, although this behavior is implemetation specific. \
If we are not interested in a thread return value, we can set *rval_ptr* to `NULL`. In this case, calling `pthread_join` allows us wait for the specified thread, but does not retureve the thread's termination status.

## Thread Synchronization

When multiple threads of control share the same memory, we need to make sure each thread sees a consistent view of its data. \
\
When one thread modifies a variable, other threads can potentially see inconsistencies when reading the value of that variable. On processor architectures in which the modification takes more than one memory cycle, this can happen when the memory read is interleaved between the memory write cycles. This is arch dependent, but portable progs cant make assumptions which processor arch is used.\
\
If thread A reads a var and then writes to it, but the write op takes two memory cycles. If thread B reads the same var between the two write cycles, it will see an incosistent value. **To solve this problem, the threads have to use a lock that will allow only one thread access the var at a time(read or write)**


## Mutexes

protecting our data ans ensure access only by one thread at a time by using the pthreads mutual-exclusion interfaces. A *mutex* is basically a lock that we set (lock) before accessing a shared resource and release (unlock) when done. \
While it is set, any other thread that tries to set it will block until we release it. Also, only one waiting thread (for lock) proceeds at at time.\
\
Mutex var is represented by the `pthread_mutex_t` data type. \
\
Before we can use a mutex variable, first initialize it by either:
1. setting it to constant `PTHREAD_MUTEX_INITIALIZER` (for statically allocated mutexes only)
2. calling `pthread_mutex_init`.
If we allocate the mutex dynamically (e.g by calling `malloc`, then we need to call `pthread_mutex_destroy` before freeing the memory).

To lock a mutex, we call pthread_mutex_lock. If the mutex is already locked, the calling thread will block until the mutex is unlocked. To unlock a mutex, we call pthread_mutex_unlock. \
\
** If a thread can’t afford to block, it can use `pthread_mutex_trylock` to lock the mutex conditionally. If the mutex is unlocked at the time `pthread_mutex_trylock` is called, then it will lock the mutex without blocking and return 0. Otherwise, it will fail, returning `EBUSY` without locking the mutex. **