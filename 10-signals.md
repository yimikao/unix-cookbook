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
We can tell kernel to do one of these at signal:
1. Ignore signal: works for most signal except `SIGKILL` and `SIGSTOP` (This is to provide kernel and superuser a sure way to kill/stop any proc). Also, ignoring signals gened by hardware exception, process behavior will be undefined.
2. Catch signal: Tell kernel to call a function of ours whenever signal occurs. `SIGKILL` and `SIGSTOP` cant be caught.
3. Let default action apply. Every signal has a default action. FOr most signals is to terminate process.

## signal Function

Simplest interface to the signal features of UNIX system is `signal` function. \
\
```c
#include <signal.h>
void (*signal(int signo, void (*func)(int))) (int);
```