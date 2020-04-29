# collision

<strong>MD5</strong> is a hash function that checks data integrity.

```
Input (data) -> Function -> Output (hash)
```

<p>Also, MD5 has one limitation called collision. It happens when two different inputs have the same output.</p>

<p>Now, let's start the challenge!</p>

```
:~$ ssh fd@pwnable.kr -p2222
col@pwnable.kr's password: guest
```

```
col@prowl:~$ ls
col col.c flag
col@prowl:~$ cat col.c
```

```
#include <string.h>
#include <string.h>
unsigned long hashcode = 0x21DD09EC;
unsigned long check_password(const char* p){
    int* ip = (int*)p;
    int i;
    int res=0;
    for(i=0; i<5; i++){
        res += ip[i];
    }
    return res;
}

int main(int argc, char* argv[]){
    if(argc<2){
        printf("usage : %s [passcode]\n", argv[0]);
        return 0;
    }
    if(strlen(argv[1]) != 20){
        printf("passcode length should be 20 bytes\n");
    }
    if(hashcode == check_password(argv[1])){
        system("/bin/cat flag");
        return 0;
    }
    else
        print("wrong passcode.\n");
        return 0;
}
```

<p>Well, the code says I have to choose one password which has to follow this: </p>

```
0 - 255 (dec)
0 - FF  (hex)
int -> 4 bytes
```

```
res += ip[i]

i = 0;
i = 1;
i = 2;
i = 3;
i = 4;
```

For example, If I have ```01 01 01 01``` ```01 01 01 01``` ```01 01 01 01``` ```01 01 01 01``` ```01 01 01 01```, it means that each i has 4 bytes or  ```01 01 01 01```.

Now, following ```res```, I have to sum all of these i and the final result have to be the same than ```hashcode = 0x21DD09EC```. However, 0x21DD09EC in decimal is 568134124, and it is not divisible by 5 because the result is 113626824.8. Let's use just 113626824 which in hex is 6C5CEC8. So, 113626824 * 4 is 454507296. If we do ```0x21DD09EC``` - 454507296, the result is 113626828 which in hex is 6C5CECC.

Finally, we have 6C5CECC + 6C5CEC8 * 4 == ```0x21DD09EC```

Let's find the flag using python to pass it as an input, and also we have to respect endiannes because the terminal understands this way.

```
col@prowl:~$ ./col $(python -c 'print "\xCC\xCE\xC5\x06" + "\xC8\xCE\xC5\x06" * 4')
```

```
flag here!
```
