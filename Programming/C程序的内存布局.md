理解 C 程序及其进程的内存布局对于程序员学习内存管理至关重要。Java、Python、C#这类高级语言由 GC 自动释放堆内存，但 C 和 C++ 没有 GC 机制，因此需要程序员手动申请、释放。C 代码先被编译成可执行文件，运行后可执行文件被载入内存，然后 CPU 开始逐条读取、执行指令。注意：在被载入前，可执行文件不占用内存空间。如果您想了解从 C 代码到二进制文件的编译过程，可阅读此文章[C Program Compilation Process - Source To Binary](https://hackthedeveloper.com/c-program-compilation-process/)。

C 程序的典型内存布局含以下几个区段：

- 命令行参数

- 栈

- 堆

- 未初始化数据段（BSS）

- 已初始化数据段

- 代码段

  ![](https://hackthedeveloper.com/wp-content/uploads/2020/11/Memory-layout.png.webp)



按内存分配方式不同，上述区段可分为：

1. 静态内存区域：代码段、已初始化数据段、未初始化数据段
2. 动态内存区域：堆、栈

在编译期静态内存区域就已划分，动态内存区域必须等到程序跑起来才会形成。下面具体学习下各个区段。

## 静态内存区域

静态内存区域由代码段、已初始化数据段和未初始化数据段组成。这些区段最初划分在可执行文件中，加载时再由操作系统加载器将其从硬盘映射到内存。

最简单的 C 程序代码如下：

```c
// 1.c
int main()
{
    return 0;
}
```

**size **是 Linux 下用于查看目标文件、库文件和可执行文件中各个区段信息及其大小的工具。将上述代码编译后用该工具查看生成的可执行文件。

```bash
$ gcc 1.c -o 1.out

$ size 1.out
text    data     bss     dec     hex filename
785     276       4      1065     429 1.out 
```

### 代码段

代码段是静态内存区域的核心段，该区段包含了程序的机器指令、常量和字面量，这些指令实现了程序的逻辑。

代码段在堆和数据段之下，可读、可执行。为了避免遭受**堆栈溢出**的破坏，防止代码被意外修改，代码段不可写。

**objdump **是 Linux 下反汇编目标文件和可执行文件的工具。以下反汇编代码中的`<main>`函数对应了示例代码中的`main`函数。限于篇幅原因，只截取了部分代码展示。

```bash
$ objdump -d 1.out


1.out:     file format elf32-littlearm


Disassembly of section .init:

0001029c <_init>:
   1029c:       e92d4008        push    {r3, lr}
   102a0:       eb00001d        bl      1031c <call_weak_fn>
   102a4:       e8bd8008        pop     {r3, pc}

Disassembly of section .plt:

000102a8 <.plt>:
   102a8:       e52de004        push    {lr}            ; (str lr, [sp, #-4]!)
   102ac:       e59fe004        ldr     lr, [pc, #4]    ; 102b8 <.plt+0x10>
   102b0:       e08fe00e        add     lr, pc, lr
   102b4:       e5bef008        ldr     pc, [lr, #8]!
   102b8:       00010d48        .word   0x00010d48

000102bc <__libc_start_main@plt>:
   102bc:       e28fc600        add     ip, pc, #0, 12
   102c0:       e28cca10        add     ip, ip, #16, 20 ; 0x10000
   102c4:       e5bcfd48        ldr     pc, [ip, #3400]!        ; 0xd48

000102c8 <__gmon_start__@plt>:
   102c8:       e28fc600        add     ip, pc, #0, 12
   102cc:       e28cca10        add     ip, ip, #16, 20 ; 0x10000
   102d0:       e5bcfd40        ldr     pc, [ip, #3392]!        ; 0xd40

000102d4 <abort@plt>:
   102d4:       e28fc600        add     ip, pc, #0, 12
   102d8:       e28cca10        add     ip, ip, #16, 20 ; 0x10000
   102dc:       e5bcfd38        ldr     pc, [ip, #3384]!        ; 0xd38

Disassembly of section .text:

000102e0 <_start>:
   102e0:       e3a0b000        mov     fp, #0
   102e4:       e3a0e000        mov     lr, #0
   102e8:       e49d1004        pop     {r1}            ; (ldr r1, [sp], #4)
   102ec:       e1a0200d        mov     r2, sp
   102f0:       e52d2004        push    {r2}            ; (str r2, [sp, #-4]!)
   102f4:       e52d0004        push    {r0}            ; (str r0, [sp, #-4]!)
   102f8:       e59fc010        ldr     ip, [pc, #16]   ; 10310 <_start+0x30>
   102fc:       e52dc004        push    {ip}            ; (str ip, [sp, #-4]!)
   10300:       e59f000c        ldr     r0, [pc, #12]   ; 10314 <_start+0x34>
   10304:       e59f300c        ldr     r3, [pc, #12]   ; 10318 <_start+0x38>
   10308:       ebffffeb        bl      102bc <__libc_start_main@plt>
   1030c:       ebfffff0        bl      102d4 <abort@plt>
   10310:       0001044c        .word   0x0001044c
   10314:       000103d0        .word   0x000103d0
   10318:       000103ec        .word   0x000103ec

0001031c <call_weak_fn>:
   1031c:       e59f3014        ldr     r3, [pc, #20]   ; 10338 <call_weak_fn+0x1c>
   10320:       e59f2014        ldr     r2, [pc, #20]   ; 1033c <call_weak_fn+0x20>
   10324:       e08f3003        add     r3, pc, r3
   10328:       e7932002        ldr     r2, [r3, r2]
   1032c:       e3520000        cmp     r2, #0
   10330:       012fff1e        bxeq    lr
   10334:       eaffffe3        b       102c8 <__gmon_start__@plt>
   10338:       00010cd4        .word   0x00010cd4
   1033c:       00000018        .word   0x00000018

00010340 <deregister_tm_clones>:
   10340:       e59f0018        ldr     r0, [pc, #24]   ; 10360 <deregister_tm_clones+0x20>
   10344:       e59f3018        ldr     r3, [pc, #24]   ; 10364 <deregister_tm_clones+0x24>
   10348:       e1530000        cmp     r3, r0
   1034c:       012fff1e        bxeq    lr
   10350:       e59f3010        ldr     r3, [pc, #16]   ; 10368 <deregister_tm_clones+0x28>
   10354:       e3530000        cmp     r3, #0
   10358:       012fff1e        bxeq    lr
   1035c:       e12fff13        bx      r3
   10360:       00021024        .word   0x00021024
   10364:       00021024        .word   0x00021024
   10368:       00000000        .word   0x00000000

0001036c <register_tm_clones>:
   1036c:       e59f0024        ldr     r0, [pc, #36]   ; 10398 <register_tm_clones+0x2c>
   10370:       e59f1024        ldr     r1, [pc, #36]   ; 1039c <register_tm_clones+0x30>
   10374:       e0411000        sub     r1, r1, r0
   10378:       e1a01141        asr     r1, r1, #2
   1037c:       e0811fa1        add     r1, r1, r1, lsr #31
   10380:       e1b010c1        asrs    r1, r1, #1
   10384:       012fff1e        bxeq    lr
   10388:       e59f3010        ldr     r3, [pc, #16]   ; 103a0 <register_tm_clones+0x34>
   1038c:       e3530000        cmp     r3, #0
   10390:       012fff1e        bxeq    lr
   10394:       e12fff13        bx      r3
   10398:       00021024        .word   0x00021024
   1039c:       00021024        .word   0x00021024
   103a0:       00000000        .word   0x00000000

000103a4 <__do_global_dtors_aux>:
   103a4:       e92d4010        push    {r4, lr}
   103a8:       e59f4018        ldr     r4, [pc, #24]   ; 103c8 <__do_global_dtors_aux+0x24>
   103ac:       e5d43000        ldrb    r3, [r4]
   103b0:       e3530000        cmp     r3, #0
   103b4:       18bd8010        popne   {r4, pc}
   103b8:       ebffffe0        bl      10340 <deregister_tm_clones>
   103bc:       e3a03001        mov     r3, #1
   103c0:       e5c43000        strb    r3, [r4]
   103c4:       e8bd8010        pop     {r4, pc}
   103c8:       00021024        .word   0x00021024

000103cc <frame_dummy>:
   103cc:       eaffffe6        b       1036c <register_tm_clones>

000103d0 <main>:
   103d0:       e52db004        push    {fp}            ; (str fp, [sp, #-4]!)
   103d4:       e28db000        add     fp, sp, #0
   103d8:       e3a03000        mov     r3, #0
   103dc:       e1a00003        mov     r0, r3
   103e0:       e28bd000        add     sp, fp, #0
   103e4:       e49db004        pop     {fp}            ; (ldr fp, [sp], #4)
   103e8:       e12fff1e        bx      lr
```

### 已初始化数据段

已初始化数据段存储了代码中所有已初始化的全局变量和已初始化的静态变量。

已初始化数据段可读、可写，储存在该段的数据可被实时修改。在上节代码中各增加一个全局变量和一个静态局部变量。

```c
// 2.c
int a = 123;
int main()
{
    static int b = 456;
    return 0;
}
```

对比可知，新文件的 data 段增加了8个字节，正好等于32位平台下2个 `int`型变量的字节数。

```bash
$ size 1.out 2.out  

   text    data     bss     dec     hex filename
    785     276       4    1065     429 1.out
    785     284       4    1073     431 2.out 
```

### 未初始化数据段（BSS）

未初始化数据段也叫`BSS`段，其名称来源于一个古老的汇编操作符，该操作符代表 **block started by the symbol**（以符号开始的块）。

未初始化数据段存储了代码中所有未初始化的全局变量和未初始化的静态变量，本段同样可读、可写。

去掉上节代码中的初始化语句，只留下变量的声明。

```c
// 3.c
int a;
int main()
{
    static int b;
    return 0;
}
```

对比可知，新文件的 data 段减少了8个字节，bss 段恰好增加了8个字节。

```bash
$ size 1.out  2.out 3.out 
	 
  text    data     bss     dec     hex filename
    785     276       4    1065     429 1.out
    785     284       4    1073     431 2.out
    785     276      12    1073     431 3.out  
```

## 动态分配的内存

下图是在程序运行期间对应进程的内存情况：

![](https://hackthedeveloper.com/wp-content/uploads/2020/11/Memory-Management-in-C-768x251.png.webp)



### 栈

栈是进程虚拟地址空间中的一块区域，该区域按先进后出、后进先出的方式增加、移除数据，并且增加数据时地址从高往低增长。程序运行时可以没有堆，但不能没有栈，由此可见栈的重要性。

调用新的一个函数将创建一个新栈帧，函数返回后对应栈帧将被自动清除。每个函数都有一个独立的栈帧，也称为**活动记录**。栈帧的总大小是浮动的，这取决于局部变量、形参和函数调用情况。

每个进程都有自己固定或可配置的栈空间，栈空间中可容纳多个栈帧，操作系统会在进程结束后回收这些栈空间。使用`ulimit -s`命令查看当前 Linux 系统支持的最大栈空间。

```bas
$ ulimit -s
8192
```

`cat /proc/xxx/limits`命令可查看某具体进程的栈空间限额。

```bash
$ cat /proc/6673/limits
Limit                     Soft Limit           Hard Limit           Units
Max cpu time              unlimited            unlimited            seconds
Max file size             unlimited            unlimited            bytes
Max data size             unlimited            unlimited            bytes
Max stack size            8388608              unlimited            bytes
Max core file size        0                    unlimited            bytes
Max resident set          unlimited            unlimited            bytes
Max processes             6865                 6865                 processes
Max open files            1024                 1048576              files
Max locked memory         67108864             67108864             bytes
Max address space         unlimited            unlimited            bytes
Max file locks            unlimited            unlimited            locks
Max pending signals       6865                 6865                 signals
Max msgqueue size         819200               819200               bytes
Max nice priority         0                    0
Max realtime priority     2                    2
Max realtime timeout      unlimited            unlimited            us
```

#### 内存布局

栈帧存储了4类数据：

- 传给被调用者的形参（按从右到左的顺序入栈）
- 调用者的返回地址
- 调用者的帧指针
- 被调用者中的局部变量

32位操作系统中，调用者的返回地址和帧指针各占4个字节，而64位操作系统中各占8个字节。

以下代码用于展示栈的内存布局：

```c
// stack_layout.c
#include <stdio.h>
int sum(int a, int b)
{
    return a + b;
}

float avg(int a, int b)
{
    int s = sum(a, b);
    return (float)s / 2;
}

int main()
{
    int a = 10;
    int b = 20;
    printf("Average of %d, %d = %f\n", a, b, avg(a, b));
    return 0;
}
```

上述代码生成的程序运行起来后栈布局如下：

![](https://hackthedeveloper.com/wp-content/uploads/2020/11/Stack-Layout.png.webp)

当前函数的栈帧总是位于进程栈区段的顶部。指向当前栈帧起始的指针称为帧指针或基指针；指向当前栈帧末尾的指针称为栈指针。被调用者帧针指向的4字节的内存空间存储了调用者的帧指针，其作用是被调用者返回后能够重新定位调用者的帧指针，以此恢复调用者的栈帧。

操作系统内存管理模块负责栈空间的分配与回收，程序员无法干涉。函数的栈帧被构造后，局部变量方能在此获得内存空间；若函数的栈帧被操作系统回收，局部变量也将随之消失；这就是所谓的变量作用域。

#### 错误类型

接下来谈一谈可能发生的栈错误。

1. 栈溢出

这种错误通常发生在一长串函数调用时，相应的栈帧不断累加，直至超过程序的栈内存限额。具体分以下两种情况：

- 函数的递归调用
- 函数中声明了长数组

因此，不推荐在栈中储存大对象。

2. 栈损坏

当向目标空间拷贝了过量的数据往往就会发生栈损坏。

以下代码中先声明了一个10个元素的字符数组，再将命令行参数拷贝进来。命令行参数是人为给定的，当参数的字符总数超出了数组大小，多余字符将非法覆盖与数组相邻的栈空间，这就造成了栈损坏。

```c
#include <stdio.h>
#include <string.h>

int copy(char *argv)
{
    char name[10];
    strcpy(name, argv);
}

int main(int argc, char **argv)
{
    copy(argv[1]);
    printf("Exit\n");
    return 0;
}
```

### 堆

从上文可知，栈空间的大小存在限制，因此不能用来存储大数据，并且程序员无法控制。堆不存在这些问题，也不被操作系统管理，您可以随时申请或释放一段连续的内存地址空间。

**Glibc** 运行库提供了多个堆操作函数，详见 stdlib.h 头文件中相关函数的声明。其中，`malloc()`和`calloc()`函数用于向操作系统申请堆块，再通过返回指针访问堆块；`free()`函数用于释放堆块；这些函数底层都调用了`brk()`和`sbrk()`系统调用。以下代码展示了堆块申请和释放过程：

```c
#include <stdio.h>
#include <stdlib.h>

void func()
{
    int a = 10;
    int* aptr = &a;
    int* ptr = (int*)malloc(sizeof(int));
    *ptr = 20;
    printf("Pointing in Stack = %d\n",  *aptr);
    printf("Heap Memory Value = %d\n",  *ptr);
    free(ptr);
}

int main()
{
    func();
    
    return 0;
}
```

`malloc()`函数申请了4字节的堆块并返回一个指针（虚拟地址），通过该指针定位到区块，再将整型数20存储入其中。内存管理单元（MMU）会将虚拟地址转换成物理地址，因此20实际就存储在计算机的物理内存中。生成程序的堆、栈布局如下图。

![](https://hackthedeveloper.com/wp-content/uploads/2020/11/heap-layout.png.webp)

堆中存储的数据不涉及作用域的概念，也无法被自动释放，要求程序员及时、自行释放，否则会造成内存泄漏。

感谢您的阅读，希望您喜欢这些有趣的知识。

[原文链接：Memory Layout Of A C Program](https://hackthedeveloper.com/memory-layout-c-program/)
