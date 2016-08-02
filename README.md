this is V6 ed, _lightly_ modified to:

1. compile with a K&R compiler (not even ansified); replacing =+ with += and using "=" for default assignment
2. assume 32-bits or better (ldiv and ldivr, increasing the sbrk to 4096)
3. use setjmp+longjmp instead of sysexit+reset
4. use lseek() instead of seek() (although still assumes 512 bytes)
5. copies the tempfilename to work on systems without writable strings
6. use SIG_IGN instead of the numeric value "1"
7. remove the goto errlab since that doesn't work in ANSI C
8. commented out getpid() in favor of a local copy
9. renamed unix() to run_unix() because some systems (at least some linux) annoyingly do a -Dunix=1
10. use sizeof(buf) instead of buf
11. renamed putchar() to putc() because gcc and clang generate errors if you create a function called putchar() with a different signature.
12. added a prototype for errfunc() because clang refuses to honor `-ansi`

I've tried to keep these modifications as minimal as possible in an effort to preserve
the editor that I learned unix on.

To compile with GCC on most platforms you can get away with:

    gcc -Os -s -ansi -w -o ed ed.c 

CLANG will begrudgingly compile with:

    clang -Wno-return-type -w -Os -o ed ed.c

`ed.c.orig` is included so you can verify how minimal my changes were.
