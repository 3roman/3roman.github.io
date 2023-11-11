也许您能够熟练编写 C 程序，但是您未必了解如何由 C 代码得到二进制文件，本文将系统介绍下这个过程。

## 编译器介绍

### 编译器的定义

维基百科关于编译器的定义如下：

> 编译器（compiler）是一种计算机程序，它会将某种编程语言写成的源代码（原始语言）转换成另一种编程语言（目标语言）。它的主要目的是将便于人类编写、阅读、维护的高级计算机语言所写作的源代码程序，翻译为计算机能解读、运行的低级机器语言的程序，也就是可执行文件。



### 编译器的分类

在正式讲编译过程之前，有必要提前了解下 C 编译器的分类。

#### GCC

GCC 是 **GNU Compiler Collection**  的缩写，也是 GNU 项目的重要一员。只要遵循 **GPL** 协议，任何人都可以免费使用它。本文将使用该编译器作演示。

#### Clang

Clang 编译器采用 LLVM 后端，可以编译C、C++、Object-C、Object-C++四种 C 系程序。该编译器基本用于 macOS 平台。

> 当程序较大、较复杂时，GCC 在 macOS 上的表现欠佳，因此想到了 Clang。   —[EDUCBA](https://www.educba.com/best-c-compilers/)

您可在该[维基词条](https://en.wikipedia.org/wiki/List_of_compilers)下找到包括 C 编译器在内的市面上所有的编译器。下面让我们来剖析一下 C 程序的编译流程。

## 编译流程详解

写好代码之后，第一件要做的事就是编译代码，此过程往往耗费数秒。代码在这期间经过一系列转换，得到了二进制文件。另外，这个过程是顺序进行的，就好比工厂里的流水线，上一工序的输出是下一工序的输入。

进一步讲解之前，先声明以下两个约定：

- 仅编译源文件
- 分开编译源文件

C 程序的编译流程分四大步骤：

1. 预处理
2. 编译
3. 汇编
4. 链接

只有顺利通过上述所有步骤，才能输出产物。操作系统不同，输出产物也不同。在 Windows 系统下，通常得到 **.exe** 文件，而在 Linux 系统下默认生成 **a.out**。在上述任何步骤中，即使发生了一个细微的错误也会造成编译或链接失败。

下图展示了整个编译流程，您可以快速总览一下。

![](https://nerdyelectronics.com/wp-content/uploads/2017/07/GCC_CompilationProcess.png)

以下命令可得到编译过程产生的所有中间文件和可执行文件：

```bash
$ gcc –Wall –save-temps cprogram.c –o cprogram
```

参数说明：

- -Wall

  显示所有错误信息和警告信息

- -save-temps

  保存编译过程产生的中间文件

- -o

  指定输出文件名

中间文件：

- cprogram.i

  预处理器生成物

- cprogram.s

  编译器生成物

- cprogram.o

  汇编器生成物

- cprogram.out

  最终产物—可执行文件

### 预处理

预处理是整个编译流程的第一步，经过预处理的代码称为**翻译单元**。

任何 C 代码中都不可避免地要包含一些库、定义一些宏，有时还要设一些条件编译指令。所有这些指令被称为预处理指令。经过预处理后，预处理指令将被替换成原始值。

#### 头文件

C 代码的开头一般会包含一个或多个头文件。预处理器根据`#include`指令将相关头文件的内容替换进来。

#### 宏

C 语言中使用`#define`指令定义宏，预处理器会将这些宏替换成相应值。

#### 条件编译

为了提高编译效率，我们期望需要编译的代码尽可能少，这时就用到了条件编译。预处理器会根据条件编译指令去除那些无需编译的代码。常见条件编译指令如下：

- **#undef**
- **#ifdef**
- **#ifndef**
- **#if**
- **#else**
- **#elif**
- **#endif**

#### 注释

为了提高程序的可读性，代码中经常要写注释。对于计算机而言，这些注释都是多余的，因此预处理器会移除代码中的注释。

#### GCC 命令

一个简单 C 程序的示例代码如下：

```c
// Header File
#include <stdio.h>

#define Max 10

int main()
{
    printf("Hello World"); // Print Hello World
    printf("%d", Max);
    return 0;
}
```

使用`-E`参数将源文件转换成编译单元。由于包含了 stdio.h 头文件，翻译后的代码行数多达上千，以下只截取关键部分展示：

```bash
$ gcc -E cprogram.c


# 1 "cprogram.c"
# 1 ""
# 1 ""
# 1 "cprogram.c"
.......
.......
.......
typedef __time64_t time_t;
# 435 "C:/msys64/mingw64/x86_64-w64-mingw32/include/corecrt.h" 3

typedef struct localeinfo_struct {
  pthreadlocinfo locinfo;
  pthreadmbcinfo mbcinfo;
} _locale_tstruct,*_locale_t;

typedef struct tagLC_ID {
  unsigned short wLanguage;
  unsigned short wCountry;
  unsigned short wCodePage;
} LC_ID,*LPLC_ID;

.......
.......
.......
# 1582 "C:/msys64/mingw64/x86_64-w64-mingw32/include/stdio.h" 2 3
# 3 "cprogram.c" 2


# 5 "cprogram.c"
int main()
{
    printf("Hello World");
    printf("%d", 10);
    return 0;
}
```

C 开发工具箱提供了一个专门用于预处理的工具 cpp，其用法如下：

```bash
cpp cprogram.c
```

综上，预处理器只是做了一些很基础的字符替换、移除等工作。预处理得到的文件扩展名为`.i`。如果将其直接作为编译器的输入，编译器假定此文件已完成预处理，自动跳过预处理步骤。

### 编译

编译是很关键的一步，此步骤的输入为翻译单元，输出为汇编代码。汇编代码接近硬件，但人类仍可读。

先用`-S`参数将源文件预处理、编译成汇编代码，再用`cat`命令查看结果。

```bash
$ gcc -S cprogram.c

$ cat cprogram.s

	.file	"cprogram.c"
	.text
	.def	printf;	.scl	3;	.type	32;	.endef
	.seh_proc	printf
printf:
	pushq	%rbp
	.seh_pushreg	%rbp
	pushq	%rbx
	.seh_pushreg	%rbx
	subq	$56, %rsp
	.seh_stackalloc	56
	leaq	128(%rsp), %rbp
	.seh_setframe	%rbp, 128
	.seh_endprologue
	movq	%rcx, -48(%rbp)
	movq	%rdx, -40(%rbp)
	movq	%r8, -32(%rbp)
	movq	%r9, -24(%rbp)
	leaq	-40(%rbp), %rax
	movq	%rax, -96(%rbp)
	movq	-96(%rbp), %rbx
	movl	$1, %ecx
	movq	__imp___acrt_iob_func(%rip), %rax
	call	*%rax
	movq	%rbx, %r8
	movq	-48(%rbp), %rdx
	movq	%rax, %rcx
	call	__mingw_vfprintf
	movl	%eax, -84(%rbp)
	movl	-84(%rbp), %eax
	addq	$56, %rsp
	popq	%rbx
	popq	%rbp
	ret
	.seh_endproc
	.def	__main;	.scl	2;	.type	32;	.endef
	.section .rdata,"dr"
.LC0:
	.ascii "Hello World\0"
.LC1:
	.ascii "%d\0"
	.text
	.globl	main
	.def	main;	.scl	2;	.type	32;	.endef
	.seh_proc	main
main:
	pushq	%rbp
	.seh_pushreg	%rbp
	movq	%rsp, %rbp
	.seh_setframe	%rbp, 0
	subq	$32, %rsp
	.seh_stackalloc	32
	.seh_endprologue
	call	__main
	leaq	.LC0(%rip), %rcx
	call	printf
	movl	$10, %edx
	leaq	.LC1(%rip), %rcx
	call	printf
	movl	$0, %eax
	addq	$32, %rsp
	popq	%rbp
	ret
	.seh_endproc
	.ident	"GCC: (Rev3, Built by MSYS2 project) 10.2.0"
	.def	__mingw_vfprintf;	.scl	2;	.type	32;	.endef
```

编译结果与 CPU 架构相关。即使同一份代码用同一个编译器编译，在不同 CPU 架构的机器上得到汇编结果也不同。编译得到的汇编代码属于低级语言，下一步骤将被转换为可重定位对象文件。

### 汇编

汇编器用上一步得到的汇编代码生成机器代码，输出可重定位对象文件。CPU 架构不同，汇编器也不同。不同架构下的汇编器将汇编代码转换成匹配本架构的机器代码。

#### 生成可重定位对象文件

`as`工具可由汇编文件生成可重定位对象文件，用法如下：

```bash
$ as cprogram.s -o cprogram.o
```

gcc 使用`-c`参数合并预处理、编译、汇编三个操作，直接从源文件生成对象文件。多个源文件要重复执行以下命令。

```bash
$ gcc -c cprogram.c	
```

机器指令人类不可读。截止目前，我们学会了如何从汇编文件或者源文件生成对象文件，下面仅就剩最后一步链接。

### 链接

链接也是至关重要的一步，这一步我们将合并/链接多个对象文件，得到另一种可执行的对象文件。

假设有以下两个文件：htd.h 中声明了`printHTD()`函数，htd.c 中定义了`printHTD()`函数。

```c
// htd.h
void printHTD();
```

```c
#include <stdio.h>
#include "htd.h"

void printHTD()
{
    printf("Hack The Developer");
}
```

cprogram.c 中的`main`()调用`printHTD`()输出字符串。

```c
// cprogram.c
#include "htd.h"
int main()
{
    printHTD();
    return 0;
}
```

先将源文件分别生成两个对象文件，然后使用`ld`工具将生成的对象文件链接起来。由于未链接标准库，链接器报错**未定义的引用**，链接失败！

```bash
$ gcc -c cporgram.c
$ gcc -c htd.c
$ ld htd.o cprogram.o -o cprogram.out

ld: warning: cannot find entry symbol _start; defaulting to 00010074
ld: htd.o: in function `printHTD':
htd.c:(.text+0xc): undefined reference to `printf'
```

gcc 内置的链接器可自动链接标准库，用法如下。链接成功，终于得到梦寐以求的可执行文件。

```bash
$ gcc cprogram.o htd.o -o cprogram.out
```

## 对象文件分析

`nm`工具用于显示对象文件里的各种符号。

```bash
$ nm htd.o
         U printf
00000000 T printHTD
$ nm cprogram.o
00000000 T main
         U printHTD
```

以上结果中，00000000为符号在对象文件中的偏移量，U 代表未定义(undefined)的符号，T 代表该符号位于代码段(Text)。链接器会从一起链接的标准库和其它对象文件中搜寻这些未定义符号，如果在别处都找到就能成功链接。下图展示了可重定位对象文件的文件结构。

![](https://hackthedeveloper.com/wp-content/uploads/2020/10/Object-File-Structure.png.webp)

众多节中，以下4个节非常关键：

- .text

  此节可读、可执行但不可写，存储了机器代码。

- .data

  此节可读，可写，存储了所有已初始化的全局变量和已初始化的静态变量。

- .rodata

  此节仅可读，储存了常量和字面量。

- .bss

  此节可读，可写，存储了所有未初始化的全局变量和未初始化的静态变量。

ELF 是可执行文件、可重定位文件、共享库文件、核心转储文件的共同文件标准。在 Linux 下可用`readelf`工具读取标准 ELF 文件的信息。限于篇幅原因，这里不再演示。

可执行文件中的主要节的结构如下：

![](https://hackthedeveloper.com/wp-content/uploads/2020/10/Structure.png.webp)

感谢您的阅读，希望您喜欢这些有趣的知识。

[原文链接：C Program Compilation Process](https://hackthedeveloper.com/c-program-compilation-process/)

 