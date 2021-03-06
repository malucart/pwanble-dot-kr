# fd

First at all, let's know the meaning of file descriptor, ssh command and cat command.

<p><strong>File descriptor</strong> is a number that the computer understands it as a open file. Also, this file is an unique number no-negative. File descriptor is used by modern operating systems, like: Linux, macOS X and BSD. In Microsoft Windows, it's called file handle, by the way.</p>

<p><strong>ssh command</strong> is included on Linux and it's used to start the ssh client program that it enables secure encrypted connection to the ssh server on a remote machine.</p>

<p><strong>cat command</strong> reads data from files and outputs their contents on Linux.</p>

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

<p>Which number can I choose? I found that atoi() function is responsable to convert string to int. I also found that read() function
is responsable to read bytes from a stdin with a file descriptor. As I know, stdin is represented as 0, stdout as 1 and stderr as 2.
Consequently, int fd as 0 is perfect to let me write anything because the program will read it as a stdin and move it to a char buf[32].
So, I will choose a number which minus 0x1234 is going to result in 0. This number is 0x1234 in decimal: 4660. And then, I write LETMEWIN.</p>

```
fd@prowl:~$ ./fd 4660
LETMEWIN
```

<p>Finally, I got the flag!</p>

```
good job :)
mommy! I think I know what a file descriptor is!!
```
