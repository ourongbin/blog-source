title: why-should-you-close-a-pipe-in-linux
date: 2016-12-15 11:31:25
tags:
- c
- linux
- pipe
---
*http://stackoverflow.com/questions/19265191/why-should-you-close-a-pipe-in-linux*

If you connect two processes - parent and child - using a pipe, you create the pipe before the fork.

The fork makes the both processes have access to both ends of the pipe. This is not desirable.

- The reading side is supposed to learn that the writer has finished if it notices an EOF condition. This can only happen if all writing sides are closed. So it is best if it closes its writing FD ASAP.

- The writer should close its reading FD just in order not to have too many FDs open and thus reaching a maybe existing limit of open FDs. Besides, if the then only reader dies, the writer gets notified about this by getting a SIGPIPE or at least an EPIPE error (depending on how signals are defined). If there are several readers, the writer cannot detect that "the real one" went away, goes on writing and gets stuck as the writing FD blocks in the hope, the "unused" reader will read something.

So here in detail what happens:

1. parent process calls pipe() and gets 2 file descriptors: let's call it rd and wr.
2. parent process calls fork(). Now both processes have a rd and a wr.
3. Suppose the child process is supposed to be the reader.
4. Then
    - the parent should close its reading end (for not wasting FDs and for proper detection of dying reader) and
    - the child must close its writing end (in order to be possible to detect the EOF condition).
