诸如Visual Studio这类集成开发环境使得编程越发便捷，但也失去很多“捯饬”的乐趣。我很喜欢侯捷先生的一段话，简单朴素却又发人深省。

> 使用一个东西，却不明白它的道理，*不高明*！

当然，凡事没有绝对。发一封电子邮件却要事先把CPU的工作原理搞清楚，不把人逼疯才怪！

从C源代码到可执行程序一般经历四个主要过程：预处理(preprocess)、编译(compile)、汇编(assemble)、链接(link)，下面使用gcc工具详细剖析下。在集成开发环境中，这4个过程直接合为**构建**（Build）操作。

![](https://raw.githubusercontent.com/3roman/PicBed/master/hexo/1_wHKe6W4opLmk6pb7sxZz6w.png)


gcc(GNU C Compiler)是一套开源、跨平台的C/C++编译器，它是Linux平台编译器的事实标准。gcc通过后台调用各子程序完成各项具体任务，比如汇编子程序as、链接子程序ldd。本文中用到的的命令如下：

- 预处理

  gcc默认不保存预处理后的文件（很少用到），可使用重定向保存结果。

  -E

  > Preprocess only; do not compile, assemble or link.

  ```bash
  gcc -E main.cpp > main.i
  ```

- 编译

  -S 

  > Compile only; do not assemble or link.

  ```bash
  gcc -S main.cpp
  ```

- 编译+汇编

  -c

  > Compile and assemble, but do not link.  

  ```bash
  gcc -c main.cpp
  ```

  

- AIO

  ```bash
  gcc main.cpp base.cpp
  ```

  

## 预处理

### 过程描述

预处理的主要任务是处理源码中除编译器指令`#pragma`外其它以“#”打头的指令，具体如下：

- 宏替换

  替换`#define`自定义宏及编译器内置调试宏，比如将`PI`替换成`3.1415926`

- 文件包含

  插入头文件的内容到当前文件，注意文件包含是递归进行的，即`main.c`包含了`utility.h`，`utility.h`包含了`base.h`，最终`base.h`也会出现在`main.c`预编译结果中，可能带来的问题稍后再议。

  > 带后缀名的**iostream.h**和不带后缀名的**iostream**是不同的两个头文件，C++程序建议使用**iostream**，后者引入了命名空间且更现代、更安全。

- 条件编译

  根据`#if、#ifndef、#ifdef、#endif、#undef`条件编译指令，保留部分代码，抛弃一些代码，以实现版本控制、防止重复包含等目的。

- 添加行号和文件名标识符

  供编译器生成pdb调试文件及输出错误或警告的具体位置。比如`main.c`包含的头文件`base.h`中`int`关键字笔误，编译器给出了详细的错误定位，以便查找修改。

  ![image-20210915211625676](https://raw.githubusercontent.com/3roman/PicBed/master/hexo/image-20210915211625676.png)

- 移除注释

  移除所有块注释和行注释，因此注释的多少并不不影响可执行文件的体积。

### 代码清单

*mian.cpp*

```c++
#include "base.h"
#define PI 3.1415926

int main()
{
    // 地球体积
    double earth = 4 / 3 * PI * 6378.137e3 * 6378.137e3 * 6378.137e3;
    return 0;
}
```

*base.h*

```c++
void LoveWithFreedom();
```

处理后的文件内容如下：

![](https://raw.githubusercontent.com/3roman/PicBed/master/hexo/image-20210904212117898.png)

### 防卫式声明

设想这么一个场景，main.cpp直接包含了animal.h和person.h，person.h也包含了animal.h，这就意味着main.cpp包含animal.h两次，造成animal.h声明的变量在main.cpp中重复声明。

```cpp
// main.cpp
#include "animal.h"
#include "person.h"

int main()
{
    return 0;
}

// person.h
# include "animal.h"

// animal.h
class Animal
{

};
```

预处理阶段只是简单的替换，因此此阶段不会报错，但后续必然无法通过编译。为了避免这种情况，头文件应采用**防卫式声明**。因为部分编译器不兼容，所以不推荐使用`#pragma once`指令的方法，推荐方法如下：

```cpp
#ifndef __animal__
#define __animal__

class Animal
{

};

#endif
```

## 编译

编译是整个过程最核心、最复杂的步骤，可分为两个子阶段：预编译和优化。

### 预编译

预编译阶段执行词法分析、语法分析、语义分析并生成汇编文件。预编译大致流程如下：

> 基本词法元素包括：标识符（identifier）、关键字（keyword）、操作符（operator）、分隔符（delimiter）、字面量（literal）。

![image-20210915211033509](https://raw.githubusercontent.com/3roman/PicBed/master/hexo/image-20210915211033509.png)

1. 词法分析将整个源代码拆分成独立的记号，然后给这些记号归类、编码。当碰到无法归类的记号时，编译器就会报错，典型例子就是不合法的变量名。

![](https://raw.githubusercontent.com/3roman/PicBed/master/hexo/image-20210913221603348.png)

2. 语法分析根据语法规则识别出语法单位（赋值语句、条件语句之类），并检查语法单位结构的正确性。诸如行末缺少分号等错误都会被查出。

3. 语义分析是对语法单位进行静态的语义审查（比如声明与类型是否匹配），动态的在运行时才可确定（比如除0的错误），分析其含义，下一步就会用另一种接近目标语言（比如汇编语言）或直接用目标语言去描述这个含义。此阶段要求语句的含义和使用规则正确。

### 优化

优化目的是提高代码执行效率，过程非常复杂、艰深。常见优化有：将乘法循环转换为多次加法循环；避免函数频繁调用造成入栈、出栈开销过大，使用内联直接将函数调用替换为函数体本身；等等。

### 跨平台

CPU架构不同指令集可能不同。即使同一份代码，在X86和ARM架构下生成的汇编代码就不同，这直接造成了纯编译型语言无法实现跨平台。

![image-20210905222202793](https://raw.githubusercontent.com/3roman/PicBed/master/hexo/image-20210905222202793.png)

## 汇编

汇编语言实际就是用助记符代替机器指令，用符号代替内存地址。汇编过程只是机械地将每一条汇编指令翻译成相对应的机器指令，通俗点讲就是将人看的懂的汇编代码转换成机器看的懂的二进制代码，因此过程并不复杂。汇编生成后缀名为.o的目标文件（object file）。

## 链接

### 重定位

链接是整个流程的最后一步，主要任务是处理模块间的符号引用问题。如果当前文件引用了外部函数或全局变量，编译及汇编阶段无法确定其偏移量，只能暂保留原始符号。链接器将多个目标文件的各个段捆绑到一起后就能确定所有符号的最终段内偏移，这个过程称为重定位（Relocation）。CRT标准静态库也会打包到最终可执行程序中，文件大小对比见下图：

![image-20210915221225990](https://raw.githubusercontent.com/3roman/PicBed/master/hexo/image-20210915221225990.png)

以下示例代码中，main函数分别调用了内部的`foo`和`bar`函数，以及外部`help`函数、标准库函数`puts`，使用`gcc -c main.c`命令编译、汇编生成目标文件main.o。

```c
#include <stdio.h>
#include "help.h"

void foo()
{
    return;
}

void bar()
{
    return;
}

int main()
{
    foo();
    bar();
    help();
    puts("hello world");

    return 0;
}
```

通过命令`nm -n main.o`查看目标文件的符号信息，结果一目了然，内部函数都具有**.text**段内偏移地址，而`help`和`puts`函数显示为未定义的符号**U**。

![image-20210914224528225](https://raw.githubusercontent.com/3roman/PicBed/master/hexo/image-20210914224528225.png)

> nm工具可列出目标文件的符号信息，包括：段内偏移地址，段名，符号

根据类型和时机不同，重定位分为三类：

- 程序内部跨文件引用，链接时重定位（其它目标文件、静态库）
- 引用外部库文件，程序装载时重定位（静态调用动态链接库 #pragma comment）
- 引用外部库文件，程序运行期重定位（动态调用动态链接库 LoadLibrary）

### 符号表

为了便于链接阶段重定位，汇编器在生成的目标文件中嵌入了一个符号表（sysmbol table），表里保存了两部分信息：

- 该目标文件**本身定义的**函数和全局变量
- 该目标文件**引用的外部**函数和全局变量

### 入口函数

从代码角度而言，`main`是大家公认的入口函数，程序从硬盘映射到内存后应该从`main`函数的第一句代码开始执行，但是大家有没有想过：是谁给main函数传递了命令行参数？

事实上，程序被加载后，首先执行的是入口函数`mainCRTStartup`（针对MSVC控制台程序），随之控制权也由操作系统转移过来。接下来该函数需完成以下工作：

1. 初始化CRT库和程序运行环境，包括初始化堆、线程、IO、全局变量等；
2. 调用main函数（可自定义）并传递命令行参数；
3. main函数执行完毕后，返回到入口函数，完成善后清理工作，包括销毁堆、关闭I/O、全局变量析构等；
4. 呼叫操作系统结束进程。

使用olldbg动态反汇编一个简单的C语言helloword程序，程序加载后停在了`0x0044125B`的位置，此处即调用了入口点函数`mainCRTStartup`。

![image-20210915223026614](https://raw.githubusercontent.com/3roman/PicBed/master/hexo/image-20210915223026614.png)

## 番外篇

节选了网上的一篇文章，比较全面的描述了PE文件的加载、运行过程：

> WinMain函数被执行之前，有一系列复杂的加载动作，还要执行一大段启动代码。运行程序MyApp.exe时，操作系统的加载程序首先为进程分配一个4GB的虚拟地址空间，然后把程序MyApp.exe所占用的磁盘空间作为虚拟内存映射到这个4GB的虚拟地址空间中。一般情况下，映射到虚拟地址空间中0X00400000的位置。加载一个应用程序所花的时间比想象的要少，因为加载一个PE文件并不是把整个文件一次性的从磁盘读到内存中，而是简单的做一下内存映射，因此映射一个大文件和映射一个小文件所花费的时间相差无几。当然，真正执行文件中的代码时，操作系统还是要把存在于磁盘上的虚拟内存中的代码交换到物理内存(RAM)中。但是，这种交换也不是把整个文件所占用的虚拟地址空间一次性的全部从磁盘交换到物理内存中，操作系统会根据需要和内存占用情况交换一页或多页。当然，这种交换是双向的，即存在于物理内存中的一部分当前没有被使用的页也可能被交换到磁盘中。接着，系统在内核中创建进程对象和主线程对象以及其它内容。然后操作系统的加载程序搜索PE文件中的输入表，加载所有应用程序所使用的动态链接库。动态链接库的加载与对应用程序的加载完全类似。
>
> 所有加载完成后，操作系统执行PE文件头部所指定地址（入口点）处的代码，开始应用程序主线程的执行。首先被执行的代码并不是MyApp.exe中的WinMain函数，而是被称为C Runtime startup code的WinMainCRTStartup函数，该函数是链接时由链接程序附加到MyApp.exe中的。在MSVC下，链接器对控制台程序设置的入口函数是 mainCRTStartup，mainCRTStartup 再调用main 函数；对图形界面程序设置的入口函数是 WinMainCRTStartup，WinMainCRTStartup 调用你自己写的 WinMain 函数。具体设置哪个入口点函数由链接器的“/subsystem:”选项确定的，它告诉操作系统如何加载运行EXE文件。可指定四种方式：CONSOLE|WINDOWS|NATIVE|POSIX。
>
> WinMainCRTStartup函数获得本进程的全部命令行指针和环境变量的指针，完成一些全局变量以及C运行时内存分配函数的初始化工作。如果使用C++，还要执行全局对象的构造函数。最后，WinMainCRTStartup函数调用WinMain函数， 并传给WinMain函数的4个参数：hInstance、hPrevInstance、lpCmdline、nCmdShow。
>
> 
