# STANDARD I/O LIBRARY

### Streams and FILE Objects

Instead of working with `fd`s directly, we use `FILE` streams instead. \ 
`fopen` returns a `FILE` object pointer. It contains a structure containing all info required by the standard I/O library to manage the stream:
- the `fd` for actual I/O,
- a pointer to a buffer for the stream,
- the size of the buffer,
- a count of the number of chars currently in the buffer,
- an error flag, etc

### Standard Input, Output, Error

The streams that refer to corresponding fds:

- `stdin -> STDIN_FILENO`
- `stdout -> STDOUT_FILENO`
- `stderr -> STDERR_FILENO`

### Buffering

The goal of buffering provided by Standard I/O is to use minimum number of `read` and `write` calls. \
Three types of buffering:
- Fully buffered
- Line buffered
- Unbuffered

### Opening Stream

`fopen` and it's variants open a standard I/O stream \

### Closing Stream

`fclose` is used \
Any buffered output data is flushed before file is closed. Any input data that may be buffered is discarded. If the standard I/O library had automatically allocated a buffer for the stream, that buffer is released.
When a process terminates normally, either by calling the exit function directly or by returning from the main function, all standard I/O streams with unwritten buffered data are flushed and all open standard I/O streams are closed.

### Reading and Writing Stream


