# fd

First at all, let's know what file descriptor, ssh command and cat command means. 

<p><strong>File descriptor</strong> is a number which the computer understands as a open file. Also, this file is a unique number no-negative. File descriptor is used by modern operating systems: Linux, macOS X and BSD. In Microsoft Windows, it's called file handle.</p>

<p><strong>ssh command</strong> is included on Linux and it's used to start the ssh client program that enables secure encrypted connection to the ssh server on a remote machine.</p>

<p><strong>cat command</strong> on Linux reads data from files and outputs their contents.</p>

```
:~$ ssh fd@pwnable.kr -p2222
fd@pwnable.kr's password: guest
```

```
fd@prowl:~$ ls
fd fd.c flag
fd@prowl:~$ cat fd.c
```

<strong>fd.c</strong>
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
char buf[32];
int main(int argc, char* argv[], char* envp[]){
    if(argc<2){
        printf("pass argv[1] a number\n");
        return 0;
    }
    int fd = atoi(argv[1]) - 0x1234;
    int len = 0;
    len = read(fd, buf, 32);
    if(!strcmp("LETMEWIN\n", buf)){
        printf("good job :)\n");
        system("/bin/cat flag");
        exit(0);
    }
    printf("learn about Linux file IO\n");
    return 0;
}
```

```
fd@prowl:~$ ./fd
pass argv[1] a number
```

<p>I'm going ton choose 0x1234 in decimal, which is: 4660.</p>

```
fd@prowl:~$ ./fd 4660
LETMEWIN
```

<p>Finally, I got the flag!</p>

```
good job :)
mommy! I think I know what a file descriptor is!!
```


