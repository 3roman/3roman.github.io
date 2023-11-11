无论是计算机老鸟还是小白，多少都接触过点注册表，这里俨然就是 Windows 操作系统的藏宝库。注册表由单词`Registry`直译而来，不过笔者认为译作**配置信息数据库**更加贴切。注册表是一种集中存储方式，linux 系统采用配置文件存储配置信息，比如`bash.rc`、`my.cnf`等等，这种方式是松散的。

注册表包含五个根键，根键、子健、键值对的概念这里不再累述。排在第一个的根键为`HKEY_CLASSES_ROOT`，它是`HKEY_LOCAL_MACHINE\Software\Classes`子健和`HKEY_CURRENT_USER\Software\Classes`子健的合并视图，主要存储了文件类型、右键菜单和COM组件等信息。展开该根键后，出现一大堆后缀名和 GUID 序列，下面将谈谈这些内容的用途。

## 打开方式

### 剖析注册表

双击一个文本文件后，为什么张三电脑上打开的是记事本程序，而李四的电脑上确是 sublime？想要直接弄清楚注册表中的端倪，有点难度。不妨换个思路，将刚才的文本文件重命名为`test.abcd`，操作系统现在不知道该如何打开这个文件了，文件图标也变成了白色。使用 [Registry Workshop](http://www.torchsoft.com/download/RegistryWorkshop.exe) 工具给当前注册表拍个快照，然后双击打开`test.abcd`，选择**始终用**记事本打开，接着对比当前注册表和刚生成的快照。

![image-20210928224718742](https://raw.githubusercontent.com/3roman/PicBed/master/hexo/image-20210928224718742.png)

对比结果显示，新增了`HKEY_CLASSES_ROOT\.abcd`和`HKEY_CLASSES_ROOT\abcd_auto_file`两个子项；`.abcd`子项默认键值对描述了该后缀名对应的文件类型`abcd_auto_file`；而文件类型子项才真正记录了打开、编辑该类型文件的程序。您可以依葫芦画瓢，给任何已知、未知的程序设置默认打开方式，这也是很多病毒的惯用伎俩。

![image-20210928232620565](https://raw.githubusercontent.com/3roman/PicBed/master/hexo/image-20210928232620565.png)

### ShellExecute 函数

ShellExecute 的功能是对一个指定的文件执行某操作，函数原型如下：

```c++
HINSTANCE ShellExecuteA(
  HWND   hwnd, // 父窗口句柄
  LPCSTR lpOperation, // 指定操作：open、edit、print...
  LPCSTR lpFile, // 待操作的文件或程序
  LPCSTR lpParameters, // 操作参数，如果打开的是文件，该参数设为NULL
  LPCSTR lpDirectory, // 操作的工作目录
  INT    nShowCmd // 窗口显示方式
);
```

其中，lpFile 待操作文件的默认程序通过查询以上注册表项获得，lpOperation 指定的具体操作与注册表 shell 项下的子项名称对应。

做个小试验，先将`HKEY_CLASSES_ROOT\txtfile\shell\print\command`默认键值改为 calc.exe，然后编写一个非常简单的C++命令行程序，代码如下。在生成目录下新建 test.txt 文本文件，运行程序后不出意外将弹出计算器。当然，在文件上右键打印也会弹出计算器，ShellExecute 函数其实就是实现了诸如打开、编辑和打印等文件交互式操作。

```c++
#include <Windows.h>

void main()
{
    // 调用系统默认程序打印文本文件
    ShellExecute(NULL, L"print", L"test.txt", NULL, NULL, 0);
}
```

## 右键菜单

注册表项`shell`和`shellex`都与右键菜单相关联。`shell`子项对应普通右键菜单，通常用于选择某个程序打开文件；`shellex`对应级联菜单，功能相对复杂，一般只是记录了对某个 COM 组件的引用。

按作用位置不同，右键菜单至少分文件右键菜单、文件夹右键菜单、文件夹空白处右处键菜单三类，下面将逐一介绍。

// TODO

### 文件右键菜单



### 文件夹右键菜单

### 文件夹空白右键菜单

## COM 组件



