# Heap Smash v1 --- Solution Write-up

## Challenge Summary

The challenge description explicitly stated that the flag was loaded
into heap memory and freed before user interaction:

> Heap exploitation warmup: the flag is loaded into memory and freed at
> startup. Reclaim the uninitialized memory to leak the flag.

This indicates an information disclosure vulnerability caused by reuse
of an uninitialized heap allocation.

## Analysis

The service exposed the following commands:

-   `A idx size` --- Allocate a note.
-   `W idx off len` --- Write data.
-   `R idx` --- Read a note.
-   `D idx` --- Delete a note.
-   `E` --- Exit.

The intended behavior is equivalent to:

``` c
char *flag = malloc(size);
read(flag_fd, flag, ...);
free(flag);
```

Since `free()` does not erase heap memory, the contents remain intact
until that chunk is reused.

glibc's tcache allocator tends to return the most recently freed chunk
of the same size. Therefore, allocating a note with the matching size
should reclaim the same heap chunk that previously stored the flag.

## Exploitation Strategy

Rather than manually guessing the allocation size, I wrote a small
pwntools automation script.

For each candidate allocation size, the script:

1.  Connected to the challenge service.
2.  Allocated a note.
3.  Did **not** write any data into it.
4.  Immediately issued the `R` command.
5.  Scanned the returned bytes for a string matching `athena{...}`.

This process was repeated across common heap sizes until the correct
allocation size was found.

## Result

When the correct allocation size was used, the allocator returned the
previously freed heap chunk. Because the memory had not been initialized
after allocation, reading it immediately revealed the residual contents,
including the flag.

No heap metadata corruption, arbitrary write, or code execution was
required. The challenge was solved solely by reclaiming and reading an
uninitialized heap allocation.

## Root Cause

The vulnerability arises because sensitive heap memory is freed without
being cleared, and later returned by `malloc()` without initialization.

Conceptually:

``` c
flag = malloc(...);
read(flag_fd, flag, ...);
free(flag);

/* later */

buf = malloc(...);
/* buf still contains previous contents */
```

Reading `buf` before writing to it exposes stale heap data.

## Conclusion

The intended solution is an information disclosure attack based on heap
reuse. By reclaiming the freed allocation and immediately reading the
uninitialized buffer, the previously stored flag can be recovered.
