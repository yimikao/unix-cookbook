# Signals

Signals are software interrupts.\
\
They provide a way to handle asynchronous events. \ 
\
Signal names are all defined by positive integer constants (signal number) in `<sigal.h>`. None has a sig num 0, POSIX.1 calls it *null signal*. \
\
Many conditions can generate a si9gnal:
- Terminal-generated when users press certain terminal keys. e.g CTRL+C normally causes SIGINT
- Hardware exceptions: divide by 0, invalid mem reference etc . These are usually detected by the hardware.
- `kill(2)` function allows a process send any signals to other process or process group. We have to be the owner of the process we're sending to, or be the superuser.
- `kill(1)` command allows to send signals to other processes. This program is just an interface to `kill` function. Its often used to terminate a runaway background process.
- software conditions can generate signals when a process should be notifued of various events. These arent hardware-generated conditions (as in div-by-o condition), but software conditions. E.G \
SIGURG (gened when out-of-band data arrives over a net conn) \
SIGIPE (when a process writes to a pipe that has no reader) \
SIGALRM (when an alarm clock set by process expires) \
\
Signals are async events. THey occur at random times to the process. Process cant simply test a variable (e.g `errno`) to see whether a signal has occured. It has to tell kernel **if and when this signal occurs, do the following**. \
\
We can tell kernel to do one of these at signal (i.e disposition of a signal):
1. Ignore signal: works for most signal except `SIGKILL` and `SIGSTOP` (This is to provide kernel and superuser a sure way to kill/stop any proc). Also, ignoring signals gened by hardware exception, process behavior will be undefined.
2. Catch signal: Tell kernel to call a function of ours whenever signal occurs. `SIGKILL` and `SIGSTOP` cant be caught.
3. Let default action apply. Every signal has a default action. FOr most signals is to terminate process.

## signal Function

Simplest interface to the signal features of UNIX system is `signal` function. \
\
```c
#include <signal.h>
void (*signal(int signo, void (*func)(int))) (int);
// returns previous disposition of signal if OK, SIG_ERR on error
```

It returns a function pointer, which is the previous signal handler. \
\
**signo** is the name of the signal \
\
**func** is either:
1. constant `SIG_IGN` (ignore)
2. constant `SIG_DFL` (default) 
3. address of a function to be called when signal occurs.

Many systems call the handler with additional, implementation-dependent arguments.

The `signal` function protoype above can be simplified through use of the following `typedef`:
```c
typedef void Sigfunc(int);
```
Then the prototype becomes
```c
Sigfunc *signal(int, Sigfunc*);
```

---
CONFUSES ABOUT THIS page 324 \
#define SIG_IGN (void (*)())1

---

## Program startup

When a program is executed, the status of all signals is either default or ignore. Normally, all signals are set to their default action, unless the process that calls `exec` is ignoringn the signal.\
\
`exec` changes the disposition of any signals being caught to their default action and leave the status of all other signals alone. Naturally, a signal that us being caught by a process that calls `exec` cannt be caught by the same functipn in the new program, since the address of the catching func in the caller probably has no meaning in the new program file that is executed. \
\
Example is how an interactiive shell treats the interrupt and quit signals for a background process. With a shell that doesnt support job control, when we execute a process in the background, as in
```c
cc main.c &
```
shell automatically sets the disposition of the interrupt (`SIGINT`) and quit (`SIGQUIT`) signals in the bg process to be ignored. So that interrupt character for example doesnt affect bg proc. If not both fore and bag process are terminated. \
interractive programs that catch these two signals have code like ths:
```c
// catching SIGINT and SIGQUIT in a child process with new program. The process catches the signal only if the signal is not currently ignored

void sig_int(int), sig_quit(int);

if (signal(SIGINT, SIG_IGN) != SIG_IGN)
    signal(SIGINT, sig_int)
if (signal(SIGQUIT, SIG_IGN) != SIG_IGN)
    signal(SIGQUIT, sig_quit)
```
These two calls to signal also show a limitation of `signal`: we are not able to determine the current disposition of a signal without changing the disposition. `sigaction` function allows us to determine a signal’s disposition without changing it.

## Process Creation

When a process calls `fork`, the child inherits the parent's signal dispositions. Here, since the child starts off with a copy of parent's memory image, the address of a signal-catching function has meaning in the child.

## Reentrant Functions

When a signal that is being caught is handled by a process, the normal sequence of instructions being executed by the process is temporarily interrupted by the signal. 