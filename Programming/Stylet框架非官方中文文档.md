Stylet 是一款非常优秀的 MVVM 框架，配套 wiki 写的也很棒。为了让更多的人熟悉这款框架，同时也给自己一个全面学习 Stylet 的机会，笔者决定将其 wiki 译成中文。限于外语水平和专业知识有限，难免出现不到和纰漏之处，望读者海涵。

# 简介

Stylet是一款小巧又强大的 MVVM 框架，它的诞生受到了[Caliburn.Micro](http://caliburnmicro.com/)的启发。开发本框架的意图是尽量降低 MVVM 的复杂性和神秘性，让菜鸟也能快速上手。

Stylet 的部分功能是 Caliburn.Micro 不具备的，比如：IoC  容器、简单的 ViewModel 验证，兼容 MVVM 的消息框（译者注：方便单元测试）。

Stylet 项目源码经过全面的单元测试，LOC 指标优秀，非常适合一些由于使用和验证 SOUP 导致开销很大的项目。得益于模块化的设计，本框架提供功能可按需引入。

>Stylet is a minimal, but powerful, MVVM framework inspired by [Caliburn.Micro](http://caliburnmicro.com/). Its intention is to cut down on the complexity and magic even further, to allow people (coworkers) unfamiliar with any MVVM framework to get up to speed more quickly.
>
>It also provides features not available in Caliburn.Micro, including its own IoC container, easy ViewModel validation, and even an MVVM-compatible MessageBox.
>
>The low LOC count and very comprehensive test suite makes it an attractive option for projects where using and validating/verifying SOUP comes with a high overhead, which its modular toolkit-inspired architecture means it's easy to use just the bits you like, or replace the bits you don't.
>
>A brief feature list is shown below. Follow the links on the right to learn more.

## ViewModel 优先模式

经典 MVVM 架构通常使用View优先的模式，该模式中由 View 隐式实例化 ViewModel，ViewModel 一般不被显式实例化。Stylet 反其道而行之，先实例化 ViewModel 再用 ViewModel 附加 View，形成了独具一格的ViewModel 优先模式。这样做最明显的好处是方便组织 ViewModel。

> The classic MVVM structure, where a view knows how to instantiate its ViewModel, and ViewModels typically don't communicate directly, is known as View-first. However, reversing this pattern - instantiating the ViewModels yourself and having the Views automatically attached - provides many advantages, allowing you to compose your ViewModels in a way which should feel very familiar. This ViewModel-first approach is the only one supported by Stylet.

## 动作

WPF 的 ICommand 接口很强大但也很笨重，譬如按钮点击动作用属性来实现而不是用方法，也不怎么符合直觉。 通过前端 XAML 动作绑定：`<Button Command="{s:Action DoSomething}"/>`，按钮被点击时将调用 ViewModel 中定义好的 `DoSomething()`方法。另外，还可创建一个名为 `CanDoSomething` 的监视属性用于判断该方法是否可被执行。事件也可绑定动作，比如为`MouseEnter`事件绑定动作：`<Button MouseEnter="{s:Action DoSomethingElse}"/>`。

> The ICommand interface used by WPF is powerful, but clunky when used in an MVVM architecture. It doesn't seem right that actions to be taken by your ViewModel in response to things like button clicks should be represented as properties, rather than methods. A simple `<Button Command="{s:Action DoSomething}"/>` will cause `DoSomething()` on your ViewModel to be called every time the button is clicked. Additionally, if you have a bool property called `CanDoSomething`, that will be observed and used to tell whether the button should be enabled or disabled.
>
> Actions also work with events, allowing you to do things like `<Button MouseEnter="{s:Action DoSomethingElse}"/>`.


## 页面和页面管理器

Screen 是 ViewModel 的基类，其本身实现了很多功能：事件通知、ViewModel 校验、生命周期管理（何时显示、隐藏、关闭窗口及确定窗口能否被关闭）。

>  The Screen class provides many things which make it an attractive base class for your ViewModels: PropertyChanged notifications, validation, and the ability to be notified when it's shown/hidden/closed, and the ability to control if and when it can be closed.

## 事件总线

类似于 Caliburn.Micro 的事件总线，订阅者直接从总线上获取某事件而无需了解该事件发布者的信息。事件总线有很多应用场景，比如 ViewModel 间传递消息。

>  Stylet's Event Aggregator is very similar to Caliburn.Micro's, and allows subscribers to receive messages from publishes without either having knowledge of, or retaining, the other. This is particularly useful for messaging between ViewModels, although it has plenty of other uses.

## 窗口管理器

窗口具体实现了 Stylet 的 ViewModel 优先模式，还实现了兼容 MVVM 的消息框。

> With a ViewModel-first approach, you display windows and dialogs by referencing the ViewModel to display, and the View gets attached automatically. The WindowManager allows this to be done with ease.
>
> An MVVM-compatible MessageBox implementation is also provided, so you don't have to roll your own.

## 校验

在传统 MVVM 架构下，校验 ViewModel 需要大量的模板文件，而且相关资源都是松散的。Stylet 有一个专门用于校验的框架，该框架还支持自定义校验库（比如： [FluentValidation](https://fluentvalidation.codeplex.com/)）。

> Validation in MVVM is traditionally a bit of a pain: it requires a fair amount of boilerplate in each ViewModel that requires validation, and resources on how to do this well are sparse.
>
> Stylet comes with a framework for taking your favourite validation library (e.g. [FluentValidation](https://fluentvalidation.codeplex.com/)) and handles running validations and reporting the results to the View.

## StyletIoC 容器

Stylet 提供了一个强大、快速、轻量且易于使用的 IoC 容器。

> Stylet comes with its own lightweight and extremely fast (but still powerful) IoC container, although it's easy to use another one if you'd prefer.

## MIT 许可

Stylet 使用 MIT 许可，该许可证允许您自行修改 Stylet 并将其用于商业用途，无归属者（唯一的限制是您必须包含本许可的副本）。 若有需要，我们愿意根据具体情况重新许可。

> Stylet is distributed under the MIT license, which allows you to modify Stylet, and include it in commercial projects, without attributation (the only restriction being that you must include a copy of the license). I'm open to re-licensing it on a case-by-case basis if you require this.

# 快速上手

想要上手试一下吗？那么请往下看吧！下面将创建一个简单的脚手架项目。

***注意**：本项目的 git 仓库包含示例代码，可自行下载学习。*

> Want to get up and running as quickly as possible? This is the right place!
>
> *NOTE: If you're looking for example applications, download the source code and look in the Samples folder.*
>
> The following instructions will set up a minimal skeleton project.

## 自动安装

### .NET Framework

***注意**：VS2013 及之前的版本不支持自动安装 Nuget 包，若有需要请手动安装。*

如果您刚接触 Stylet，最简单的上手步骤如下：

1. 打开Visual Studio，创建一个 WPF 项目；
2. 通过 NuGet 包管理器（右键项目 ->管理 Nuget 包）安装`Stylet.Start`。
3. 项目创建成功后，可删除 `Stylet.Start` 包。

> **Note**: This will **not** work if your project uses **PackageReference** for NuGet packages or if you are using VS2013 or earlier. Follow the "Manual Option" section below instead.
>
> If you're new to Stylet (and you're running VS2015 or later), this is the easiest way to get started.
>
> 1. Open Visual Studio, and create a new `WPF Application` project.
> 2. Open NuGet (right-click your project -> Manage NuGet Packages), and install the `Stylet.Start` package.
>
> This will give you a working skeleton project.
>
> When it has finished installing, uninstall Stylet.Start.
>
> Happy coding!

### .NET Core

建议通过 `dotnet new`命令使用 Stylet 模板创建  .NET Core 脚手架项目。

在项目目录打开命令行，输入命令安装 Stylet 模板：

```pow
dotnet new -i Stylet.Templates
```

根据新安装的模板创建项目：

```pow
dotnet new stylet -o MyStyletProject
```

> For .NET Core projects, the quickest way to get started is by using `dotnet new` with Stylet's template.
>
> Open a command window where you want to create your new project, and install the Stylet templates using:
>
> ```
> dotnet new -i Stylet.Templates
> ```
>
> Then create a new project with:
>
> ```
> dotnet new stylet -o MyStyletProject
> ```
>
> (changing `MyStyletProject` as appropriate).

## 手动安装

如果您想手动创建项目，步骤如下：

1. 打开 Visual Studio，创建一个 WPF 项目；
2. 通过 NuGet 包管理器（右键项目 ->管理 Nuget 包）安装`Stylet`包。

删除`MainWindow.xaml`和`MainWindow.xaml.cs/vb`这些以后用不到的文件。接着右键项目，新增一个名为`RootView`的 WPF 窗口，前端示例代码如下：

> If you don't want to use the `Stylet.Start` package and would prefer to create your own skeleton project, follow the instructions in this section.
>
> 1. Open Visual Studio, and create a new `WPF Application` project.
> 2. Open NuGet (right-click your project -> Manage NuGet Packages), and install the `Stylet` package.
>
> First off, delete `MainWindow.xaml` and `MainWindow.xaml.cs/vb`. You won't be needing them.
>
> Next, you'll need a root View and a ViewModel. The View has to be a `Window`, but there are no other restrictions.

```xml
<Window x:Class="Stylet.Samples.Hello.RootView"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Height="300" Width="300">
    <TextBlock>Hello, World</TextBlock>
</Window>
```

新建一个名为`RootViewModel`的 ViewModel 类。

> The ViewModel can be any old class (for now - you might want it to be a [Screen or Conductor](https://github.com/canton7/Stylet/wiki/Screens-and-Conductors)).

```c#
public class RootViewModel
{
}
```

引导器类的基本职责是实例化主窗口的 ViewModel，诸如 IoC 容器等其它功能详见后续章节。

> Next, you'll need a bootstrapper. For now, you don't need anything special - just something to identify your root ViewModel. Later, you'll be able to configure your IoC container here, as well as other application-level stuff.

```C#
public class Bootstrapper : Bootstrapper<RootViewModel>
{
}
```

删除`App.xaml`中的`StartUri`特性并引入 Stylet 命名空间后，通过该命名空间下的`ApplicationLoader`元素以加载资源的方式实例化引导器类，示例代码如下：

> Finally, this needs to be referenced as a resource in your `App.xaml`. You'll need to remove the `StartUri` attribute, and add `xmlns` entries for Stylet and your own application. Finally, you'll need to add Stylet's `ApplicationLoader` to the resources, and identify the bootstrapper you created above.
>
> It should look something like this:

```<Application x:Class="Stylet.Samples.Hello.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:s="https://github.com/canton7/Stylet"
             xmlns:local="clr-namespace:Stylet.Samples.Hello">
    <Application.Resources>
       <s:ApplicationLoader>
            <s:ApplicationLoader.Bootstrapper>
                <local:Bootstrapper/>
            </s:ApplicationLoader.Bootstrapper>
        </s:ApplicationLoader>
    </Application.Resources>
</Application>
```

好了，可以运行程序了！

> That's it! Run that, and you should get a window with 'Hello World' in it.

## 应用程序装载器

上述`<s:ApplicationLoader>`元素用于加载 Stylet 内置的资源，此类继承自资源字典类 ResourceDictionary ，用法如下：

> It's worth noting that `<s:ApplicationLoader>` above is a ResourceDictionary subclass. This allows it to load in Stylet's built-in resources (see [Screens and Conductors](https://github.com/canton7/Stylet/wiki/Screens-and-Conductors)). You can choose not to load Stylet's resources like this:

```xml
<s:ApplicationLoader LoadStyletResources="False">
   ...
</s:ApplicationLoader>

```

`<s:ApplicationLoader>`元素也可用来创建不属于Stylet 的应用程序级资源或资源字典，用法如下：

> If you want to add your own Resources / ResourceDictionaries to the Application, the simplest way is like this:

```xml
<Application.Resources>
    <s:ApplicationLoader>
        <s:ApplicationLoader.Bootstrapper>
            <local:Bootstrapper/>
        </s:ApplicationLoader.Bootstrapper>

        <Style x:Key="MyResourceKey">
            ...
        </Style>

        <s:ApplicationLoader.MergedDictionaries>
            <ResourceDictionary Source="MyResourceDictionary.xaml"/>
        </s:ApplicationLoader.MergedDictionaries>
    </s:ApplicationLoader>
</Application.Resources>
```

以上代码也写成如下习惯形式：

> If this makes you uncomfortable for some reason, you can also do this:

```xml
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.MergedDictionaries>
            <s:ApplicationLoader>
                <s:ApplicationLoader.Bootstrapper>
                    <local:Bootstrapper/>
                </s:ApplicationLoader.Bootstrapper>
            </s:ApplicationLoader>

            <ResourceDictionary Source="MyResourceDictionary.xaml"/>
        </ResourceDictionary.MergedDictionaries>

        <Style x:Key="MyResourceKey">
            ...
        </Style>
    </ResourceDictionary>
</Application.Resources>
```

# 引导器

引导器负责引导应用程序，可完成多项任务，其中两项基本任务如下：

1. 配置 IoC 容器
2. 通过窗口管理器实例化根 ViewModel

引导器类 BootStrapper 有两种父类：

1. BootstrapperBase<TRootViewModel>

   使用自定义的 IoC 容器

2. Bootstrapper<TRootViewModel>

   使用默认的 IoC 容器，示例代码如下：

> The bootstrapper is responsible for bootstrapping your application. It configures the IoC container, creates a new instance of your root ViewModel and displays it using the `WindowManager`. It also provides various other functions, described below.
>
> The bootstrapper comes in two flavours: `BootstrapperBase<TRootViewModel>`, which requires you to configure the IoC container yourself, and `Bootstrapper<TRootViewModel>`, which uses Stylet's built-in IoC container, StyletIoC.
>
> Example bootstrapper, using StyletIoC:

```c#

class Bootstrapper : Bootstrapper<MyRootViewModel>
{
   protected override void OnStart()
   {
      // This is called just after the application is started, but before the IoC container is set up.
      // Set up things like logging, etc
   }
 
   protected override void ConfigureIoC(IStyletIoCBuilder builder)
   {
      // Bind your own types. Concrete types are automatically self-bound.
      builder.Bind<IMyInterface>().To<MyType>();
   }
 
   protected override void Configure()
   {
      // This is called after Stylet has created the IoC container, so this.Container exists, but before the
      // Root ViewModel is launched.
      // Configure your services, etc, in here
   }
 
   protected override void OnLaunch()
   {
      // This is called just after the root ViewModel has been launched
      // Something like a version check that displays a dialog might be launched from here
   }
 
   protected override void OnExit(ExitEventArgs e)
   {
      // Called on Application.Exit
   }
 
   protected override void OnUnhandledException(DispatcherUnhandledExceptionEventArgs e)
   {
      // Called on Application.DispatcherUnhandledException
   }
}
```

## 自定义 IoC 容器

使用自定义 IoC 容器很简单，官方 git 仓库也提供了多个 IoC 容器供您选择。这些容器都经过了全面的单元测试。

由于管理多个 IoC 容器很麻烦，同时为了减少依赖， 本框架的发布包只内置一个 IoC 容器。若您需要使用其它容器，可从官方 git 仓库下载后手动添加引用。使用方法见上一章，示例代码如下：

> Using another IoC container with Stylet is easy. I've included bootstrappers for a number of popular IoC containers in the [Bootstrappers project](https://github.com/canton7/Stylet/tree/master/Bootstrappers). These are all unit-tested but not battle-tested: feel free to customize them.
>
> Note that the Stylet nuget package / dll don't include these, as it would add unnecessary dependencies. Similarly, I don't publish IoC container-specific packages, as that's a waste of effort.
>
> Copy the bootstrapper you want from the above link into your project somewhere. Then subclass it, as you would normally subclass `Bootstrapper<TRootViewModel>`, documented above. Then add your subclass to your App.xaml.cs, as documented in [Quick Start](https://github.com/canton7/Stylet/wiki/Quick-Start), e.g.

```c#
public class Bootstrapper : AutofacBootstrapper<ShellViewModel>
{
}
```

```xml
<Application x:Class="Stylet.Samples.Hello.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:s="https://github.com/canton7/Stylet"
             xmlns:local="clr-namespace:Stylet.Samples.Hello">
    <Application.Resources>
       <s:ApplicationLoader>
            <s:ApplicationLoader.Bootstrapper>
                <local:Bootstrapper/>
            </s:ApplicationLoader.Bootstrapper>
        </s:ApplicationLoader>
    </Application.Resources>
</Application>

```

## 添加资源字典

由于`s:ApplicationLoader `本身就是资源字典，如果再想添加其它资源字典，就必须与`s:ApplicationLoader`一起嵌套在合并资源字典`MergedDictionaries`中，示例代码如下：

> s:ApplicationLoader is itself a ResourceDictionary. If you need to add your own resource dictionary to App.xaml, you will have to nest s:ApplicationLoader inside your ResourceDictionary as a MergedDictionary, like this:

```xml
<Application.Resources>
    <ResourceDictionary>
        <ResourceDictionary.MergedDictionaries>
            <s:ApplicationLoader>
                <s:ApplicationLoader.Bootstrapper>
                    <local:Bootstrapper/>
                </s:ApplicationLoader.Bootstrapper>
            </s:ApplicationLoader>

            <ResourceDictionary Source="YourDictionary.xaml"/>
        </ResourceDictionary.MergedDictionaries>
    </ResourceDictionary>
</Application.Resources>
```

# ViewModel 优先

ViewModel 优先是 Stylet 框架的一大特点，其相对于传统的View优先更显自然。

> The ViewModel-first approach is one that's crucial to Stylet's architecture, but unintuitive if you learnt MVVM in the traditional View-first manner.
>
> Hopefully this article will make everything clear.

## View优先

让我们先了解一下什么是 View优先。该模式下视图必须了解 ViewModel 的详细情况，但反过来 ViewModel 可以不知 View 的信息。将两者联系到一起的常用方法是：在 View 的后台代码中实例化 ViewModel ，并赋给 View 的`DataContent`属性。

>Let's start by defining the View-first approach, and what exactly I mean by that. MVVM states that the ViewModel should know nothing about the View, but that View should be aware of the ViewModel. The obvious way to attach a View and ViewModel, then, is have the View construct its ViewModel in its codebehind - something like this:

```c#

public partial class MyView : Window
{
   public MyView()
   {
      InitializeComponent();
 
      this.DataContext = new MyViewModel();
   }
}
```

目前看起来都还不错！但当 View 创建了其它 View，并需按层次关系组合这些被创建的 View 时，危机来到了。一个很典型场景是窗口 View 创建了标题栏 View 和工作区 View。

> This is fine. Views can create and own other views, meaning that you can compose your views into a hierarchy. All well and good.
>
> The crunch comes when you've composed a couple of views, say something like this, where a shell contains a top bar and a frame, inside which any page can be shown:

```xml
<!-- This is a window which contains a top bar and another page -->
<Window x:Class="MyNamespace.ShellView" ....>
   <StackPanel>
      <my:TopBarView/>
      <Frame x:Name="navigationFrame"/>
   </StackPanel>
</Window>

```

三者都有各自的 ViewModel 。当标题栏 View 的某些字段变化了，比如标题改变了，窗口的 ViewModel 可以得知上述变化，但标题栏 ViewModel 却不知道。常用做法是在窗口标题栏的前端代码中，将窗口标题绑定到窗口本身的某个依赖属性上，示例代码如下：

> where `TopBarView` has its own ViewModel, `TopBarViewModel`. Fine.
>
> Now say that `TopBarView` has a field containing some data you want to update, for example the title of the current page. Now, the `ShellViewModel` knows this (it's decided what the current page is, after all), but the `TopBarViewModel` doesn't (how would it? It doesn't really know about anything). The temptation is to expose a dependency property on the `TopBarView` and bind it into the `ShellViewModel`, like so:

```xml
<Window x:Class="MyNamespace.ShellView" .... x:Name="rootObject">
   <StackPanel>
      <my:TopBarView CurrentPageTitle="{Binding CurrentPageTitle, ElementName=rootObject}"/>
      <Frame x:Name="navigationFrame"/>
   </StackPanel>
</Window>
```

后续标题栏的其它属性或者工作区的属性也许也要如此操作一番，简直太麻烦了。

另一个主要问题发生在显示窗口和对话框时，此问题也是传统 MVVM 模型中的一个痛点。在 ViewModel 中实例化并显示相应的 View 不便于单元测试，更好一点的选择是在主视图的后台代码中实例化并显示某个子视图，后续通过某种方式将创建的子视图与其子视图模型结合起来。

`Frame`元素的内容需先实例化一个 View ，该由窗口的 ViewModel 还是 View 实例化这个 View 呢？

总之，View优先弊端很多。

> but that's just nasty. You've now got one-and-a-bit views bound to the `ShellViewModel`.
>
> Another major concern is displaying windows and dialogs. In traditional MVVM, this is a bit of a pain. One option is to instantiate and display the View (using `Show()` or `ShowDialog()`) from inside the ViewModel (which makes it, or at least that bit of it, untestable). The better option is to instantiate the View you want to display in the codebehind of your View, and show it from there. This means you need to establish ways of telling to View to display this dialog, and a way of getting the results of the dialog back to the ViewModel.
>
> Indeed, setting the content for the `Frame` above requires instantiating a View to put in it. This has the same dilemma - either the ViewModel instantiates it (making it untestable), or the View does (leading to communication pain).
>
> Either way, this approach has some nastiness.

## ViewModel 优先

该模式下， ViewModel 不由 View 来实例化， ViewModel 也无需知道与其对应 View 的存在。一个第三方服务将通过命名约定的方式匹配相关联的 View 和 ViewModel （比如根据 MainView 去搜寻 MainViewModel ），匹配成功后继续由该服务实例化 ViewModel ，然后将其赋给对应 View 的 `DataContext `属性。详见后续章节。

> The ViewModel-first approach accepts that a ViewModel shouldn't know anything about its View, but does not accept that the View should be responsible for constructing the ViewModel either. Instead, a third service is responsible for locating the correct View for a given ViewModel, and setting up its DataContext correctly.
>
> The default implementation uses naming conventions to locate the correct View for a given ViewModel, replacing 'ViewModel' with 'View' in its name. That's explained in more detail in [The ViewManager](https://github.com/canton7/Stylet/wiki/The-ViewManager).
>
> **This allows ViewModels to be created by other ViewModels. Which allows ViewModels to know about, and own, other ViewModels. Which allows you to compose your ViewModels properly.**
>
> There's another part of this trick, which is best explained by example:

```c#
public class ShellViewModel
{
   public TopBarViewModel TopBar { get; private set; }
   // Stuff to instantiate and assign TopBarViewModel
}
```

```xml
<Window x:Class="MyNamespace.ShellView"
        xmlns:s="https://github.com/canton7/Stylet" .....>
   <StackPanel>
      <ContentControl s:View.Model="{Binding TopBar}"/>
      <!-- ... -->
   </StackPanel>
</Window>

```

附加属性`View.Model`将会实例化其绑定 ViewModel 的实例（本例中是`TopBarViewModel`的实例）并作为ContentControl元素的内容，然后即可定位到正确的 View（本例中是 `TopBarView`）。

诀窍在于`TopBarView`可以根据命名约定找到对应的`TopBarViewModel`，`TopBarViewModel`也可以被`ShellViewModel`知会。问题解决了！

> The `View.Model` attached property will fetch the ViewModel it's bound to (in this case it's an instance of `TopBarViewModel`), and locate the correct view (`TopBarView`). It will instantiate an instance, and set it as the content of that `ContentControl`.
>
> The upshot is that the `TopBarView` can get the name of the current page from its `TopBarViewModel`, and the `TopBarViewModel` can be told this by the `ShellViewModel`. Problem solved!
>
> The `ContentControl` trick also works well for navigation:

```xml
<Window x:Class="MyNamespace.ShellView"
        xmlns:s="https://github.com/canton7/Stylet" .....>
   <StackPanel>
      <ContentControl s:View.Model="{Binding TopBar}"/>
      <ContentControl s:View.Model="{Binding CurrentPage}"/>
   </StackPanel>
</Window>

```

当多页面窗口应用程序切换页面时，由主窗口的 ViewModel 实例化页面的 ViewModel ，并将该实例赋值给主窗口 ViewModel 的的`CurrentPage`属性。主窗口的 ViewModel 不必知道如何得到页面的 View。窗口管理器会以大致相同的方式处理对话框应用程序和窗口应用程序，在获得某个 ViewModel 实例后将由其显示对应的窗口 View 或对话框 View。

> The `ShellViewModel` will then navigate to a new page by instantiating a new instance of that page's `ViewModel`, and assigning it to the property `CurrentPage`. Note how the `ShellViewModel` no longer needs to know anything about views. *It hasn't had to instantiate a single view*. This is a very important, useful, and powerful point.
>
> Dialogs and Windows are handled in much the same way by [The WindowManager](https://github.com/canton7/Stylet/wiki/The-WindowManager). This takes a given ViewModel instance, and displays its View as a dialog or window.

## 删除后台代码

在 ViewModel 优先模式下，通过动作、转换器、附加属性、附加行为几乎可以实现所有功能，View 的后台代码也就没有存在的必要了。强烈建议您删除 View 的后台代码文件，`InitializeComponent`方法会由 Stylet 替您调用。

***注意**：VB.NET中不建议删除后台代码。*

> With this approach in place, there's nothing you actually need to do in the codebehind. You can of course do so, but there's very little you can't solve with [Actions](https://github.com/canton7/Stylet/wiki/Actions) (for handling events), Converters, Attached Properties, and (most importantly) Attached Behaviors.
>
> Stylet lets you delete the codebehind entirely (it will call `InitializeComponent` for you), and you are strongly encouraged to do so. Delete the codebehind!
>
> ***Note**: If you're using VB.NET, sometimes your XAML namespaces will stop working if you delete the codebehind. If this is the case, simply recreate the codebehind with the matching filename, give it the correct namespace and class and then leave the remainder blank. For example, `MyView.xaml.vb` :*

# 动作

点击按钮后调用 ViewModel 定义的方法，动作适用于诸如此场景。

>You have a button, and you want to click it and execute a method on your ViewModel? Actions cover this use-case.

## 动作和方法

WPF 中命令绑定步骤为：ViewModel 实现` ICommand`接口后将按钮的 Command 特性绑定到 ViewModel 的某个属性。尽管看起来有点混乱，这种机制其实也不错，至少 ViewModel 无需了解 View且View 的后台代码也可删除。但最好还是直接调用方法而不是通过某个属型间接调用。Stylet 引入了动作来解决这个问题，示例代码如下：

> In 'traditional' WPF, you'd create a property on your ViewModel which implements the [ICommand](http://msdn.microsoft.com/en-us/library/system.windows.input.icommand(v=vs.110).aspx) interface, and bind your button's Command attribute to it. This works fairly well (the ViewModel knows nothing about the View, and code-behind is not required), but it's a bit messy - you really want to be calling a method on your ViewModel, not executing a method on some property.
>
> Stylet solves this by introducing Actions. Look at this:

```c#

class ViewModel : Screen
{
   public void DoSomething()
   {
      Debug.WriteLine("DoSomething called");
   }
}
```

```xml
<UserControl x:Class="MyNamespace.View"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:s="https://github.com/canton7/Stylet">
   <Button Command="{s:Action DoSomething}">Click me</Button>
</UserControl>
```

上例中点击按钮后将直接调用`DoSomething `方法，多么简单！CommandParameter 特性还可用来给方法传递一个参数，示例代码如下：

> As you might have guessed, clicking the button called the DoSomething method on the ViewModel to be called.
>
> It's that simple.
>
> If your method accepts a single argument, the value of the button's CommandParameter property will be passed. For example:

```c#

class ViewModel : Screen
{
   public void DoSomething(string argument)
   {
      Debug.WriteLine(String.Format("Argument is {0}", argument));
   }
}
```

```xml
<UserControl x:Class="MyNamespace.View"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:s="https://github.com/canton7/Stylet">
   <Button Command="{s:Action DoSomething}" CommandParameter="Hello">Click me</Button>
</UserControl>

```

注意：动作也适用于任何 ICommand 类型的属性，比如 KeyBinding 属性。

> Note that Actions also work on any ICommand property, on anything (e.g. a [KeyBinding](http://msdn.microsoft.com/en-us/library/system.windows.input.keybinding(v=vs.110).aspx)).

## 守卫属性

1111

> You can also control whether you button is enabled just as easily, using *Guard Properties*. A guard property for a given method is a boolean property which has the name "Can<method name>", so if your method is called "DoSomething", the corresponding guard property is called "CanDoSomething".
>
> Stylet will check whether a guard property exists, and if so, will disable the button if it returns false, or enable it if it returns true. It will also watch for PropertyChanged notifications for that property, so you can change whether the button is enabled.
>
> For example:

```c#

class ViewModel : Screen
{
   private bool _canDoSomething;
   public bool CanDoSomething
   {
      get { return this._canDoSomething; }
      set { this.SetAndNotify(ref this._canDoSomething, value); }
   }
   public void DoSomething()
   {
      Debug.WriteLine("DoSomething called");
   }
}
```

## 事件

Stylet 也支持事件绑定，语法同命令绑定，但是事件绑定不涉及守卫属性的概念。

> But what about if you want to call a ViewModel method when an event occurs? Actions have that covered as well. The syntax is exactly the same, although there's no concept of a guard property here.

```xml
<UserControl x:Class="MyNamespace.View"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:s="https://github.com/canton7/Stylet">
   <Button Click="{s:Action DoSomething}">Click me</Button>
</UserControl>
```

绑定事件的方法可以没有形参，也可以有一个或两个形参，方法的签名可能如下：

> The method which is called must have zero, one, or two parameters. The possible signatures are:

```c#

public void HasNoArguments() { }
 
// This can accept EventArgs, or a subclass of EventArgs
public void HasOneSingleArgument(EventArgs e) { }
 
// Again, a subclass of EventArgs is OK
public void HasTwoArguments(object sender, EventArgs e) { }
```

## 方法返回值

动作通常不关心方法的返回类型和返回值，但返回 Task 是个例外。

> Actions don't care about the return type of the method, and the returned value is discarded.
>
> The exception to this is if a `Task` is returned (e.g. if the method being invoked is `async`). In this case, the `Task` will be awaited in an `async void` method. This means that if the method returns a `Task` which ends up containing an exception, this exception is rethrown and will bubble up to the Dispatcher, where it will terminate your application (unless you handle it, with e.g. `BootstrapperBase.OnUnhandledException`). The effect is the same as if the method being invoked was `async void`, but means it's easier to unit-test `async` ViewModel methods.

## 动作目标



> So far I've been telling a little white lie. I've been saying that the Action is invoked on the ViewModel, but that isn't strictly true. Let's go into a bit more detail.
>
> Stylet defines an inherited attached property called View.ActionTarget. When a View is bound to its ViewModel, the View.ActionTarget on the root element in the View is bound to the ViewModel, and it's then inherited by each element in the View. When you invoke an action, it's invoked on the View.ActionTarget.
>
> This means that, by default, actions are invoked on the ViewModel regardless of the current DataContext, which is probably what you want.
>
> This is a very important point, and one that's worth stressing. The DataContext will probably change at multiple points throughout the visual tree. However, the View.ActionTarget will stay the same (unless you manually change it). This means the actions will always be handled by your ViewModel, and not by whatever object is being bound to, which is almost always what you want.
>
> You can of course alter the View.ActionTarget for individual elements, for example:

## 静态方法

动作也可调用静态方法

> Actions can also invoke static methods, if the target is a `Type` object (use `{x:Type ...}` in XAML for this). You can set this using both `View.ActionTarget` and Action's `Target` property.

```c#
public static class CommonButtonTarget
{
    public static void DoSomething() { }
}
```

```xml
<Button Command="{s:Action DoSomething, Target={x:Type my:CommonButtonTarget}}">Click me</Button>
```

## 动作和样式



> Actions will not work from style setters. The classes required to do this in WPF are all internal, which means there is no way to fix the issue. You will need to use old-fashioned Commands in this (rare) case, unfortunately.

## 上下文菜单和弹出式菜单陷阱

111

>View.ActionTarget is of course an attached property, which is configured to be inherited by the children of whatever element it is set on. Like any attached property, and indeed the DataContext, there are certain boundaries it is not inherited across, such as:

- Using a ContextMenu
- Using a Popup
- Using a Frame

In these cases, Stylet will do the best it can to find a suitable ActionTarget (it may, for example, find the the ActionTarget associated with the root element in the current XAML file), but this may not be exactly what you expect (e.g. it may ignore a `s:View.ActionTarget="{Binding ...}"` line you have somewhere in the middle of your page), or it may (in rare circumstances) fail to find an ActionTarget at all.

In this case, please set `s:View.ActionTarget` to a suitable value. You may struggle to get a reference to anything outside of a ContextMenu from inside of one: I suggest the [BindingProxy](http://www.thomaslevesque.com/2011/03/21/wpf-how-to-bind-to-data-when-the-datacontext-is-not-inherited/) technique.

## 附加行为

有两种可能会导致动作停止： `View.ActionTarget`为空或者`View.ActionTarget` 中指定方法不存在。上述情况的默认处理如下：

|      | View.ActionTarget为空 | View.ActionTarget中指定方法不存在 |
| ---- | :-------------------: | :-------------------------------: |
| 命令 |       禁用控件        |        点击控件时抛出异常         |
| 事件 |       启用控件        |       事件被引发时抛出异常        |

>There are two cases which will stop an action from working properly: if the `View.ActionTarget` is null, or if the specified method on the `View.ActionTarget` doesn't exist. The default behaviour in each of these cases is as follows:
>
>|          | View.ActionTarget == null | No method on View.ActionTarget                 |
>| -------- | ------------------------- | ---------------------------------------------- |
>| Commands | Disable the control       | Throw an exception when the control is clicked |
>| Events   | Enable the control        | Throw an exception when the event is raised    |
>
>The justification for this is that if the `View.ActionTarget` is null, you must have set it yourself, so you probably know what you're doing. However, if the specified method doesn't exist on the `View.ActionTarget`, that's probably a mistake, and you deserve to know.
>
>Of course, this behaviour is configurable.
>
>To control the behaviour when `View.ActionTarget` is null, set the `NullTarget` property on the `Action` markup extension so either `Enable`, `Disable`, or `Throw`. (Note that `Disable` is invalid when the Action is linked to an event, as we have no power to disable anything).
>
>For example:

```xml
<Button Command="{s:Action MyMethod, NullTarget=Enable}"/>
<Button Click="{s:Action MyMethod, NullTarget=Throw}"/>
```

11

> Similarly, you can set the `ActionNotFound` property to the same values:

```xml
<Button Command="{s:Action MyMethod, ActionNotFound=Disable}"/>
<Button Click="{s:Action MyMethod, ActionNotFound=Enable}"/>
```

# 窗口管理器

采用传统View优先的方法，要显示一个新的窗口或对话框应首先创建视图的实例，再调用实例方法`.Show()`或`.ShowDialog()`。采用ViewModel 优先的方法时无法与视图直接交互，因此上述步骤行不通。窗口管理器正是为了解决这个问题，通过调用`IWindowManager.ShowWindow(someViewModel)`方法可定位并实例化对应的 View，再将其绑定到 ViewModel，最后显示它。示例代码如下：

> In a traditional View-first approach, if you want to display a new window or dialog, you create a new instance of the View, then call `.Show()` or `.ShowDialog()`.
>
> In a ViewModel-first approach, you can't interact directly with the views, so you can't do this. The WindowManager solves this problem - calling `IWindowManager.ShowWindow(someViewModel)` will take that ViewModel, find its view, instantiate it, bind it to that ViewModel, and display it.

```c#
class SomeViewModel
{
   private IWindowManager windowManager;
   public SomeViewModel(IWindowManager windowManager)
   {
      this.windowManager = windowManager;
   }
 
   public void ShowAWindow()
   {
      var viewModel = new OtherViewModel();
      this.windowManager.ShowWindow(viewModel);
   }
 
   public void ShowADialog()
   {
      var viewModel = new OtherViewModel();
      bool? result = this.windowManager.ShowDialog(viewModel);
      // result holds the return value of Window.ShowDialog()
      if (result.GetValueOrDefault(true))
      {
         // DialogResult was set to true
      }
   }
}
```

多么简单有效！另外，通过构造注入的方式得到`IWindowManager `的实例而不是在 ViewModel 中显式构造该实例使得单元测试方便很多。在 ViewModel 中调用父类 `Screen` 的 `RequestCLose`方法可关闭窗口或对话框。

> Nice and easy! In addition, the introduction of the IWindowManager (rather than calling methods directly on the ViewModel) makes testing a lot easier.
>
> To close a window or dialog from its ViewModel, use `Screen.RequestClose`, like this:  

```c#
class ViewModelDisplayedAsWindow
{
   // Called by pressing the 'close' button
   public void Close()
   {
      this.RequestClose();
   }
}
 
class ViewModelDisplayedAsDialog
{
   // Called by pressing the 'OK' button
   public void CloseWithSuccess()
   {
      this.RequestClose(true);
   }
}
```

# 消息框

11

> As we all know, WPF comes with its own MessageBox implementation - `System.Windows.MessageBox`. And that's fine, except that you can't call it from your ViewModel (well, you *can*, but it makes your ViewModel untestable). The usual workaround suggested online is "write your own".
>
> Well, Stylet comes with its own MessageBox clone, which looks and behaves almost identically to the WPF one (including appearance, buttons, icons, auto-sizing, sounds, alignment, etc).

## 用法

11

> To use, simply call the `ShowMessageBox` method on `IWindowManager`, like this:
>
> The MessageBox accepts all of the same options as as the WPF MessageBox, plus a few more.

```c#

public MyViewModel
{
   private readonly IWindowManager windowManager;
 
   public MyViewModel(IWindowManager windowManager)
   {
      this.windowManager = windowManager;
   }
 
   public void ShowMessagebox()
   {
      var result = this.windowManager.ShowMessageBox("Hello");
   }
}
```

## 定制消息框

111

> Stylet's MessageBox is implemented as a ViewModel, `MessageBoxViewModel`, and its corresponding View, `MessageBoxView`. The ViewModel implements an interface, `IMessageBoxViewModel`, and the `ShowMessageBox()` method uses this interface to retrieve an instance of the ViewModel.
>
> Therefore, you can provide you own custom implementation of `MessageBoxViewModel` and `MessageBoxView` by writing a ViewModel which implements `IMessageBoxViewModel`, and registering it with your IoC container. This will then be used by `ShowMessageBox()`.
>
> If you just want to tweak the behaviour of the existing `MessageBoxViewModel`, you can. The following options are available:

### 自定义按钮内容

111

> You can edit the button text for any of the buttons on a per-application basis by modifying `MessageBoxViewModel.ButtonLabels`, which is a dictionary holding the text to display for each button. If you just want to edit the text for a particular MessageBox, `ShowMessageBox` will accept a dictionary allowing you to do just that:  

```c#
MessageBoxViewModel.ButtonLabels[MessageBoxResult.No] = "No, thanks";
 
this.windowManager.ShowMessageBox("Do you want breakfast?", 
                                   buttons: MessageBoxButton.YesNo, 
                                   buttonLabels: new Dictionary<MessageBoxResult, string>()
        {
            { MessageBoxResult.Yes, "Yes please!" },
        });
 
// Will display a MessageBox with the buttons "Yes please!" and "No, thanks"
```

# 事件总线

事件总线是一个基于发布者/订阅者模型的事件管理器，其特征是去中心化和弱绑定。

>The EventAggregator is a decentralised, weakly-binding, publish/subscribe-based event manager.

## 发布者与订阅者

### 订阅者

111

> subscribers interesting in a particular event can tell the IEventAggregator of their interest, and will be notified whenever a publisher publishes that particular event to the IEventAggregator.
>
> Events are classes - do whatever you want with them. For example:

```c#
class MyEvent
{ 
  // Do something 
}
```

11

> Subscribers must implement `IHandle<T>`, where `T` is the event type they are interested in receiving (they can of course implement multiple `IHandle<T>`'s for multiple `T`'s). They must then get hold of an instance of the IEventAggregator, and subscribe themselves, for example:

```c#

class Subscriber : IHandle<MyEvent>, IHandle<MyOtherEvent>
{
   public Subscriber(IEventAggregator eventAggregator)
   {
      eventAggregator.Subscribe(this);
   }
 
   public void Handle(MyEvent message)
   {
      // ...
   }
 
   public void Handle(MyOtherEvent message)
   {
      // ...
   }
}
```

111

> For VB.NET users, the `Sub New()` passing the eventAggregator by reference will probably fail across namespaces, and can be irritating to have to define with each new subscriber. Thus, it may be easier to define your eventAggregator in a global module, then subscribe directly to it instead of passing its reference along to each new ViewModel you call.

```vb
Module Global
  Public eventAggregator as IEventAggregator
End Module

Class Subscriber : Implements IHandle(Of MyEvent)

  Public Sub New()
  Global.eventAggregator.Subscribe(Me)
  End Sub
  
  'Public Sub Handle...

End Class

```

11

> Make sure to keep the namespace for the *module* blank, so that it can be used throughout the program.

### 发布者

11

> Publishers must also get an instance of the IEventAggregator, but they don't need to subscribe themselves - they only need to call IEventAggregator.Publish every time they want to publish an event, for example:

```c#

class Publisher
{
   private IEventAggregator eventAggregator;
   public Publisher(IEventAggregator eventAggregator)
   {
      this.eventAggregator = eventAggregator;
   }
 
   public void PublishEvent()
   {
      this.eventAggregator.Publish(new MyEvent());
   }
}
```

11

> Again, for VB.NET users, if you've set up the global module then you don't need to pass the eventAggregator to the Publisher. You can just publish directly to the global eventAggregator;

```vb
Class Publisher

  Public Sub PublishEvent()
  Global.eventAggregator.Publish(New MyEvent())
  End Sub
  
End Class
```

## 取订和弱绑定

11

> Because the IEventAggregator is weakly binding, subscribers don't need to unsubscribe themselves - the IEventAggregator won't retain them. It is however possible for a subscriber to unsubscribe itself if it wants - call  

```c#
IEventAggregator.Unsubscribe(this);
```

### 同步发布和异步发布

111

> The default `IEventAggregator.Publish` method publishes the event synchronously. You can also call `PublishOnUIThread` to dispatch asynchronously to the UI thread, or `PublishWithDispatcher` and pass any action you want to act as the dispatcher (this can be useful if writing your own methods on IEventAggregator).

## 频道



# 属性通知类

PropertyChangedBase 提供了用于引发通知的方法，被用作那些要实现 INotifyPropertyChanged 接口的类型的基类。

> PropertyChangedBase is base class for types implementing INotifyPropertyChanged, and provides methods for raising PropertyChanged notifications.

## 引发通知

目的不同，引发通知的方法也不一样，最常用的方法是属性被赋值时引发通知。使用`PropertyChangedBase`自带的工具方法`SetAndNotify`，当字段被赋新值时，赋值操作才实际发生，并引发`PropertyChanged `事件通知，示例代码如下：

> There are a number of ways to raise PropertyChanged notifications, depending on what exactly you want to do.
>
> The most common case is having a property raise a notification each time it's assigned to. PropertyChangedBase provides a nice utility method to help: SetAndNotify. It takes, by reference, a field, and a value to assign to the field. If the field's value != the value, the assignment happens, and a PropertyChanged notification is raised. For example:

```c#
class MyClass : PropertyChangedBase
{
   private string _name;
   public string Name
   {
      get { return this._name; }
      set { SetAndNotify(ref this._name, value); }
   }
}
```

绑定属性前台代码如下：

> To connect to the object you have to put in the view:

```xml
<TextBox Text="{Binding Name, UpdateSourceTrigger=PropertyChanged}"/>
```

由于C#版本不同，在当前属性中引发其它属性通知的方法也不同。C# 6.0及以上版本推荐使用 `nameof()`方法，该方法非常简单并可在编译期进行安全检查；C# 6.0以下版本使用 lambda 表达式，可能相对慢一点，也可在编译期进行安全检查。如果有需要，甚至可采用原始字符串的方式。具体可结合以下代码来理解：

> If you want to raise a PropertyChanged notification for a property other than the current one, there are a few ways of achieving that, too (depending on whether you're using C#6, or below). The preferred C# 6 way is to use `nameof()`, as that is very cheap and provides compile-time safety. If you're using C# 5 or below, you can use an expression: this is slower, but also gives compile-time safety. If you *really* want, you can use a raw string as well. See below:

```c#
class MyClass : PropertyChangedBase
{
   private string _firstName;
   public string FirstName
   {
      get { return _firstName; }
      private set
      {
         SetAndNotify(ref _firstName, value);

         // Preferred if you're using C# 6, as it provides compile-time safety
         this.NotifyOfPropertyChange(nameof(this.FullName));         
      }
   }

   private string _lastName;
   public string LastName
   {
      get { return _lastName; }
      private set
      {
         SetAndNotify(ref _lastName, value);

         // Preferred for C# 5 and below, as it provides compile-time safety
         this.NotifyOfPropertyChange(() => this.FullName);
      }
   }

   public string FullName
   {
      get { return FirstName + " " + LastName; }
   }
}
```

关联通知其它属性的代码集中放到构造函数里，比如：

> You can also wire things together in the constructor, like this:

```c#
class MyClass : PropertyChangedBase
{
   private string _firstName, _lastName;

   public MyClass()
   {
      this.Bind(s => s.FirstName, (o, e) => NotifyOfPropertyChange(nameof(FullName)));
      this.Bind(s => s.LastName, (o, e) => NotifyOfPropertyChange(nameof(FullName)));
   }

   public string FirstName
   {
      get { return _firstName; }
      private set { SetAndNotify(ref _firstName, value); }
   }

   public string LastName
   {
      get { return _lastName; }
      private set { SetAndNotify(ref _lastName, value); }
   }

   public string FullName
   {
      get { return FirstName + " " + LastName; }
   }
}
```

## 分配事件

默认情况下，`PropertyChanged `事件先在当前线程被引发，再由 WPF 具体负责分配到 UI 线程。当然，直接在 UI 线程中引发事件也没问题，`PropertyChangedBase`有一个名为`PropertyChangedDispatcher`的属性，该属性有一个名为`Execute.DefaultPropertyChangedDispatcher`的默认委托，将委托值设为`Execute.OnUIThread`即可。改变`PropertyChangedBase`所有子类的事件引发方式应重写`Configure`函数，示例代码如下：

> By default, PropertyChanged events are raised on the current thread (and WPF takes care of dispatching them to the UI thread). If you do want to change this, however, you can! PropertyChangedBase has a property called PropertyChangedDispatcher, which has an `Action<Action>`, and defaults to Execute.DefaultPropertyChangedDispatcher (which you can assign) (which has the value `a => a()`).
>
> If you want to change this to execute on the UI thread, you can do the following.
>
> To change for all instances of PropertyChangedBase:

```c#
class Bootstrapper : Bootstrapper<MyRootViewModel>
{
   public override void Configure()
   {
      base.Configure();
      Execute.DefaultPropertyChangedDispatcher = Execute.OnUIThread;
   }
}
```

针对单独子类的示例代码如下：

> To change for just once instance of PropertyChangedBase:

```c#
class MyClass : PropertyChangedBase
{
   public MyClass()
   {
      this.PropertyChangedDispatcher = Execute.OnUIThread;
   }
}
```

## PropertyChanged.Fody包

[PropertyChanged.Fody](https://github.com/Fody/PropertyChanged)是一个非常神奇的 nuget 包，它能在编译期自动将用于引发`PropertyChanged `通知的代码注入到相关属性中，这样就使得代码整体很简洁。同时能够自行识别哪些属性需要引发通知，示例代码如下：

> [PropertyChanged.Fody](https://github.com/Fody/PropertyChanged) is a fantastic package, which injects code at compile-time to automatically raise PropertyChanged notifications for your properties, allowing you to write very concise code. It will also figure out dependencies between your properties and raise notifications appropriately for example:

```C#
class MyClass : PropertyChangedBase
{
   public string FirstName { get; private set; }
   public string LastName { get; private set; }
   public string FullName { get { return String.Format("{0} {1}", this.FirstName, this.LastName); } }

   public void SomeMethod()
   {
      // PropertyChanged notifications are automatically raised for both FirstName and FullName
      this.FirstName = "Fred";
   }
```

`PropertyChangedBase`也注意与`Fody.PropertyChanged`的集成，实际采用`PropertyChangedDispatcher`引发通知。因此，`Screen`、 `ValidatingModelBase`或`PropertyChangedBase`的子类使用 `Fody.PropertyChanged`无需任何特别代码。

> PropertyChangedBase also takes care to integrate with `Fody.PropertyChanged`. Notifications raised by `Fody.PropertyChanged` are raised using the `PropertyChangedDispatcher`.
>
> Therefore you do not need to do anything special in order to use `Fody.PropertyChanged` with any subclass of `Screen`, `ValidatingModelBase`, or `PropertyChangedBase`.

# 可绑定集合

## 概述

`BindableCollection<T>`集合类是`ObservableCollection`的子类，其实例一般用作 ItemsControl 族控件的数据源（增、减元素时 View 可以得到通知）。该集合类还新增了两个很有用的特性：

- 新增方法`AddRange`, `RemoveRange` 和 `Refresh`
- 线程安全

> `BindableCollection<T>` is a subclass of [`ObservableCollection`](http://msdn.microsoft.com/en-us/library/ms668604(v=vs.110).aspx). It's the class to use if you have a collection of something in your ViewModel, and want to use it as the `ItemsSource` / etc for something in your View (and have the View be notified whenever an item is added to / removed from that collection).
>
> However, it adds a couple of useful extra features:
>
> - New `AddRange`, `RemoveRange`, and `Refresh` methods
> - Is thread-safe

## 新方法

`ObservableCollection<T>`集合增加元素时需手动迭代每个元素并多次调用`collection.Add(element)`方法，如此就会引发大量新增元素事件。通过新增的方法`AddRange` 和`RemoveRange`，可以一次性批量增、减元素，同时只引发一次事件。

`Refresh` 方法不会改变集合，它只是触发`PropertyChanged` 和`CollectionChanged` 事件，通知 UI 其绑定的集合发生了变化，请 UI 元素重载数据。尽管该方法很有用，但大多数场合都用不到。

> `ObservableCollection<T>` is missing a couple of very useful methods: `AddRange` and `RemoveRange`. These do pretty much what you'd expect, allowing you to add a remove a range of elements at once, without having to manually iterate over each element and calling `collection.Add(element)` on each (while raising lots of events for each element added). `AddRange` and `RemoveRange` will raise one set of events per range added/removed only.
>
> `Refresh` is a convenience. It does not modify the collection in any way, but does cause the `PropertyChanged` and `CollectionChanged` events to be fired, indicating to any UI elements that the collection has been modified and that they should reload their data. It's not often needed, but when it is it's *really* needed.

## 线程安全

线程安全的实现方法是将集合元素的增加、删除、清空、重置等操作分配到的 UI 线程，具体分配工作由 `Execute.OnUIThreadSync`方法完成，也就是说：

- 这些都是同步操作，即方法在动作完成后才返回；
-  `PropertyChanged`和 `CollectionChanged`事件总是由 UI 线程引发；
- ？？？

属性相关的操作总是在 UI 线程中完成，因此事件也由 UI 线程引发。基于此，`BindableCollection<T>`移除了 `PropertyChangedDispatcher`、`CollectionChangedDispatcher`属性，在`PropertyChangedBase` 中这些属性仍存在。

> Thread safety is achieved by dispatching all actions (adds, removes, clear, reset, etc) to the UI thread. The dispatch uses `Execute.OnUIThreadSync`, which means that:
>
> - These operations are synchronous: the method being called won't return until the action has been completed.
> - They're free if you're already on the UI Thread - the operation will be carried out synchronously in this case.
> - All `PropertyChanged` and `CollectionChanged` events are always raised on the UI thread.
>
> That last point means that there is no `PropertyChangedDispatcher` property on `BindableCollection<T>`, as there is with `PropertyChangedBase` - the event is always raised on the UI thread, since the operation the property relates to is always performed on the UI thread. Similarly, there's no `CollectionChangedDispatcher` concept.
