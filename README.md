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

I've tried to keep these modifications as minimal as possible in an effort to preserve
the editor that I learned unix on.

To compile with GCC on most platforms you can get away with:

    gcc -Os -s -ansi -w -o ed ed.c 

`ed.c.orig` is included so you can verify how minimal my changes were:

    1c1,4
    < #
    ---
    > #include <stdlib.h>
    > #include <signal.h>
    > #include <setjmp.h>
    > #include <string.h>
    7,9d9
    < #define	SIGHUP	1
    < #define	SIGINTR	2
    < #define	SIGQUIT	3
    28c28
    < #define	error	goto errlab
    ---
    > #define	error	errlab()
    31a32
    > jmp_buf setexit;
    54,56c55,57
    < int	onhup;
    < int	onquit;
    < int	vflag	1;
    ---
    > void *onhup;
    > void *onquit;
    > int	vflag =	1;
    60c61
    < int	tfile	-1;
    ---
    > int	tfile =	-1;
    62c63
    < char	*tfname;
    ---
    > char	tfname[16];
    67c68
    < int	iblock	-1;
    ---
    > int	iblock =	-1;
    69c70
    < int	oblock	-1;
    ---
    > int	oblock =	-1;
    72,74c73,75
    < int	errfunc();
    < int	*errlab	errfunc;
    < char	TMPERR[] "TMP";
    ---
    > void errfunc(void);
    > void (*errlab)(void);
    > char	TMPERR[] ="TMP";
    85,86c86,88
    < 	onquit = signal(SIGQUIT, 1);
    < 	onhup = signal(SIGHUP, 1);
    ---
    >         errlab = errfunc;
    > 	onquit = signal(SIGQUIT, SIG_IGN);
    > 	onhup = signal(SIGHUP, SIG_IGN);
    92c94
    < 			signal(SIGQUIT, 0);
    ---
    > 			signal(SIGQUIT, SIG_DFL);
    106,108c108,110
    < 	if ((signal(SIGINTR, 1) & 01) == 0)
    < 		signal(SIGINTR, onintr);
    < 	setexit();
    ---
    > 	if ((signal(SIGINT, SIG_IGN)) != SIG_IGN)
    > 		signal(SIGINT, onintr);
    > 	setjmp(setexit);
    227c229
    < 		exit();
    ---
    > 		exit(0);
    275c277
    < 		unix();
    ---
    > 		run_unix();
    298,299c300,301
    < 				n =* 10;
    < 				n =+ c - '0';
    ---
    > 				n *= 10;
    > 				n += c - '0';
    306c308
    < 			a1 =+ n;
    ---
    > 			a1 += n;
    372c374
    < 			a1 =+ minus;
    ---
    > 			a1 += minus;
    473c475
    < 	signal(SIGINTR, onintr);
    ---
    > 	signal(SIGINT, onintr);
    486c488
    < 	seek(0, 0, 2);
    ---
    > 	lseek(0, 0, 2);
    497c499
    < 	reset();
    ---
    > 	longjmp(setexit,1);
    514c516
    < 	lastc =& 0177;
    ---
    > 	lastc &= 0177;
    531c533
    < 		if ((c =& 0177) == 0)
    ---
    > 		if ((c &= 0177) == 0)
    609c611
    < 			if (sbrk(1024) == -1)
    ---
    > 			if (sbrk(4096) == -1)
    611c613
    < 			endcore.integer =+ 1024;
    ---
    > 			endcore += 4096;
    625c627
    < unix()
    ---
    > run_unix()
    635c637
    < 		exit();
    ---
    > 		exit(255);
    637c639
    < 	savint = signal(SIGINTR, 1);
    ---
    > 	savint = signal(SIGINT, SIG_IGN);
    639c641
    < 	signal(SIGINTR, savint);
    ---
    > 	signal(SIGINT, savint);
    653c655
    < 	dol =- a2 - a1;
    ---
    > 	dol -= a2 - a1;
    671c673
    < 	tl =& ~0377;
    ---
    > 	tl &= ~0377;
    674c676
    < 			bp = getblock(tl=+0400, READ);
    ---
    > 			bp = getblock(tl+=0400, READ);
    690c692
    < 	tl =& ~0377;
    ---
    > 	tl &= ~0377;
    698c700
    < 			bp = getblock(tl=+0400, WRITE);
    ---
    > 			bp = getblock(tl+=0400, WRITE);
    703c705
    < 	tline =+ (((lp-linebuf)+03)>>1)&077776;
    ---
    > 	tline += (((lp-linebuf)+03)>>1)&077776;
    720c722
    < 		ichanged =| iof;
    ---
    > 		ichanged |= iof;
    742c744
    < 	seek(tfile, b, 3);
    ---
    > 	lseek(tfile, b * 512, 0);
    758c760
    < 	tfname = "/tmp/exxxxx";
    ---
    > 	strcpy(tfname,"/tmp/exxxxx");
    763c765
    < 		pid =>> 3;
    ---
    > 		pid >>= 3;
    802c804
    < 		*a1 =& ~01;
    ---
    > 		*a1 &= ~01;
    804c806
    < 			*a1 =| 01;
    ---
    > 			*a1 |= 01;
    808c810
    < 			*a1 =& ~01;
    ---
    > 			*a1 &= ~01;
    826c828
    < 		inglob =| 01;
    ---
    > 		inglob |= 01;
    837,838c839,840
    < 		a1 =+ nl;
    < 		addr2 =+ nl;
    ---
    > 		a1 += nl;
    > 		addr2 += nl;
    902c904
    < 		} else if (c<0 && (c =& 0177) >='1' && c < NBRA+'1') {
    ---
    > 		} else if (c<0 && (c &= 0177) >='1' && c < NBRA+'1') {
    911c913
    < 	loc2 = sp + linebuf - genbuf;
    ---
    > 	loc2 = sp + sizeof(linebuf) - sizeof(genbuf);
    1062c1064
    < 			*lastep =| STAR;
    ---
    > 			*lastep |= STAR;
    1177c1179
    < 			ep =+ *ep;
    ---
    > 			ep += *ep;
    1184c1186
    < 			ep =+ *ep;
    ---
    > 			ep += *ep;
    1212c1214
    < 		ep =+ *ep;
    ---
    > 		ep += *ep;
    1248c1250,1252
    < 	extern ldivr;
    ---
    >         long x = count[0];
    > 	x <<= 16;
    > 	x += count[1];
    1250c1254
    < 	count[1] = ldiv(count[0], count[1], 10);
    ---
    > 	count[1] = x / 10;
    1252c1256
    < 	r = ldivr;
    ---
    > 	r = x % 10;
    1270c1274
    < char	*linp	line;
    ---
    > char	*linp = line;
    1302c1306
    < 			col =+ 2;
    ---
    > 			col += 2;
