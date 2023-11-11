WinDbg 是 Windows 平台下的一款多用途调试器，由微软官方维护。区别于 OD，WinDbg  既可调试用户态代码，又能调试内核态代码及驱动程序。

## 安装

Win10及以上系统可通过[微软商店](https://www.microsoft.com/en-us/p/windbg-preview/9pgjgd53tn86#activetab=pivot:overviewtab)安装最新 Preview 程序，Win10以下只能下载安装包安装。

## 符号表

PDB(Program Database Files) 符号表文件由链接器生成，它是调试器与源文件之间的桥梁，记录了变量及函数的名称与地址、参数和局部变量的堆栈偏移量、源码的文件名和行号等信息，一般用于调试目的。PDB 文件默认搜索路径如下：

1. 当前被调试文件所在目录
2. 当前被调试文件硬编码记录的 build 目录
3. symbol server 的本地缓存
4. 远程 symbol server

指定3路径、4路径命令如下：

```shell
.sympath SRV*c:\windows\localsymbols*http://msdl.microsoft.com/download/symbols
```

## 插件

### 加载插件

强大的程序大都支持插件，WinDbg 亦然。加载插件很简单，一句命令即可：

```shell
.load C:\Windows\Microsoft.NET\Framework\v4.0.30319\SOS.dll
```

上述命令只能一次性加载，下次还得重新执行。解决方法是将加载命令添加到启动命令组中。

![image-20220203165342328](https://raw.githubusercontent.com/3roman/PicBed/master/hexo/202202031653402.png)

### 查看插件

`.chain`命令查看当前已加载插件列表

### 常用插件

SOS 插件随 .NET Framework 一同发布，提供了查看托管堆的方法。其对应 dll 文件既分 x86 和 x64 版本，又分 .NET 版本，因此应根据具体调试对象区分加载。

-  .NET 4.0及上32位程序集

  C:\Windows\Microsoft.NET\Framework\v4.0.30319\clr.dll

-  .NET 4.0及上64位程序集

  C:\Windows\Microsoft.NET\Framework64\v4.0.30319\clr.dll

该插件常用方法如下：

1. Name2EE

   根据名称查找类型、属性及方法，其原理是遍历对象的 `MethodTable`和`EEClass`结构体。

   基本语法：`Name2EE <module name> !<type or method name>`，通配符 * 代表所有模块，名称区分大小写。

   典型用法：`!name2ee *!Program.Main`

