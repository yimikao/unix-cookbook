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
