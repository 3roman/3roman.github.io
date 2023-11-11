## AOP编程范式

世界观与方法论是一个宏大的话题，世界观决定了方法论，方法论影响了世界观。在计算机的世界里，编程范式就是程序员的方法论。从时间维度看待编程，面向过程是自然而然的选择。从空间角度去看，必然会提炼出面对对象的编程方法。

面向过程、面向对象、函数式编程等都是常见的编程范式，AOP 也属于编程范式的一种，笔者认为这是一种面向**补充需求**的编程范式。AOP 的字面翻译为面向切片编程，它其实是对面向对象编程的补充。AOP 的核心思想是将程序的功能点独立出来，使程序具备更高的模块化特性。就好比提供了红点瞄准器、倍镜、消音器、枪口制退器、扩容弹夹等枪械部件，根据任务需要搭配，最终组成一把功能齐备的步枪。

AOP 常见实现方式有三种：

- 中间件
- 过滤器
- 代码织入

本文将只谈代码织入，代码织入分动态织入和静态织入。静态织入在编译时就将各种涉及 AOP 拦截的代码注入到符合一定规则的类中，最终效果与直接在源码中给类增加属性或方法相同，只是这个工作交由编译器和框架完成。动态织入借助反射特性，动态生成代理类达成 AOP。

## Fody框架

[Fody](https://github.com/FODY) 是 .NET 平台下一款优秀的开源静态织入框架，以插件的方式灵活地为程序增加功能。

Fody 并没有对织入源码，而是直接织入编译好的程序集，其中至关重要的 ILRepack 功能由 [Ceil](https://github.com/Fody/cecil) 库实现。这样处理使得框架更具通用性，毕竟 .Net 平台下的编程语言包括 C#、VB.Net和 F#三种，不过它们生成的 IL 代码格式一致。只针对 IL 代码，不针对具体语言，无形中降低了框架的开发难度。织入流程图如下。

![Code weaving using Fody](https://raw.githubusercontent.com/3roman/PicBed/master/hexo/202204122108221.png)

打包资源，参数 Null 检查，通知属性等个性化需求以往都在源码层面通过大量的 plumbing code 完成，这些代码通常高度一致。编写这些模板代码工作量很大，要求程序员掌握诸如反射、编译等多方面知识，如此重复性的工作意义不大，还造成代码冗余。插件解决了上述痛点，程序员可以用一种傻瓜化的方式实现各种需求，还避免了造轮子。目前官方提供10多个插件，具体用法见[示例项目](https://github.com/Fody/FodyAddinSamples)。

## PropertyChanged插件

PropertyChanged 是一款非常方便的插件，省去了 MVVM 编程中大量冗余代码。众所周知，属性要具备通知功能，传统做法是所属类先继承`INotifyPropertyChanged`接口，再在`setter`访问器中引发通知事件。事件的第一个参数是`this`指针，第二个参数是对应属性名（ C#6.0新增的`nameof`关键字可代替硬编码属性名）。示例代码如下：

```csharp
class Person : INotifyPropertyChanged
{
    public event PropertyChangedEventHandler PropertyChanged;

    private string _fullName;

    public string FullName
    {
        get { return _fullName; }
        set 
        { 
            if (!string.Equals(value, _fullName))
            {
                _fullName = value;
                OnPropertyChanged(nameof(FullName)); // C#6.0新增关键字
            }
        }
    }

    protected void OnPropertyChanged(string propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }
}
```

试想一下，如果有几十个通知属性，写起来多么麻烦！在没有`nameof`关键字的情况下，一旦属性名拼写错误，排错也不是件易事。这些弊端在引入 PropertyChanged 插件后将完全根除，插件安装命令如下：

```bash
Install-Package PropertyChanged.Fody
```

随之将上述代码简化为：

```csharp
class Person : INotifyPropertyChanged
{
    public event PropertyChangedEventHandler PropertyChanged;

    public string FullName { get; set; }
}
```

只要某个类实现了`INotifyPropertyChanged`接口，插件就可根据此规则自动注入相关 IL 代码，将其所属全部属性升级为通知属性。如无必要的话，也可给属性单独设置`[DoNotNotifyAttribute]`特性以跳过代码注入，更多细节用法见[官方wiki](https://github.com/Fody/PropertyChanged/wiki)。

知其然而知其所有然，我们用 ILSpy 查看一下反编译后的代码：

```csharp
internal class Person : INotifyPropertyChanged
{
	public string FullName
	{
		[CompilerGenerated]
		get
		{
			return <FullName>k__BackingField;
		}
		[CompilerGenerated]
		set
		{
			if (!string.Equals(<FullName>k__BackingField, value, StringComparison.Ordinal))
			{
				<FullName>k__BackingField = value;
				<>OnPropertyChanged(<>PropertyChangedEventArgs.FullName);
			}
		}
	}

	public event PropertyChangedEventHandler PropertyChanged;

	[GeneratedCode("PropertyChanged.Fody", "3.4.0.0")]
	[DebuggerNonUserCode]
	protected void <>OnPropertyChanged(PropertyChangedEventArgs eventArgs)
	{
		this.PropertyChanged?.Invoke(this, eventArgs);
	}
}
```

上述自动生成的代码与传统方式编写的代码并无二致，笔者认为这是目前实现通知属性最优雅的方式。

## 参考文章

1、https://codingcanvas.com/code-weaving-using-fody/
2、https://www.cnblogs.com/weidaorisun/p/15627183.html
3、https://github.com/Fody/Home

