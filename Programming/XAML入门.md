本文的素材来源于刘铁猛老师的《深入浅出WPF》一书及网上收集的资料。

XMAL(E**x**tensible **A**pplication **M**arkup **L**anguage) 全称可扩展程序标记语言，是微软开发的一种基于 XML 、声明式的界面描述语言。XAML 与C#、VB.NET 同属 .NET 平台语言，也会被编译成 IL 代码。

以上所述XAML是一种声明式语言，即只要通过标签的方式声明某个元素，编译器就会为该标签创建一个与之对应的CLR对象。比如以下 XAML 语句创建了一个按钮并指定了按钮内容和 click 事件处理函数。

```xml
<Button Click="button1_Click">点我</Button>
```

对应C#代码：

```c#
var button1 = new Button();
button1.Text = "button1";
button1.Click += new EventHandler(this.button1_Click);
```

## 命名空间

为了避免命名冲突，XAML 沿用了 XML 中命名空间的概念。命名空间只能在根元素中声明，语法为`xmlns:空间名称="xxx"`，默认命名空间的名称可以省略。双引号中的字符串有两种形式：

1. 某程序集中的命名空间

   xmlns:空间名称="clr-namesapce:程序集中的空间名;assembly=程序集文件名"。比如要引用mscorlib类库中的System命名空间，写法如下：

   ```xml
   xmlns:sys = "clr-namespace:System;assembly=mscorlib"
   ```

2. URL

   每个 WPF 程序都依赖一些必备的命名空间，逐个引入既麻烦又臃肿。XAML 使用 URL 地址代表一堆同类型命名空间，形式简洁明了，常见分类如下。

   ```xml
   xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
   xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
   xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006" 
   xmlns:d="http://schemas.microsoft.com/expression/blend/2008" 
   ```



x命名空间与解析 XAML 语言相关，其下属的几个特性很常用：

- x:Name

  指定标签名称，方便后续引用。

- x:Class

  此特性只能用于根元素，告诉编译器将当前 XAML 的编译结果与后台代码的指定类合并，前提是该类必须声明为同名的`partial`类。新建窗口、页面及控件时，VS会自动生成前台 XMAL 代码和相应的后台代码，但后台代码不是必须的。

- x:key

  WPF中的资源是以“key-value”字典形式存储的，通过key检索具体资源。

- x:static

  在 XAML 中访问类的静态成员，常用于软件的多语言显示。习惯做法是将字符串保存到资源类的静态属性中。

  ```xml
  <TextBlock Text="{x:Static Local:I8n.ENG_AGE}"
  ```


## 元素

元素通常由一个开始标签和一个结束标签组成。标签体包含了其它元素的元素称为非空元素，不包含其它元素的元素称为空元素。建议统一采用非空元素的形式。

```xml
<!--非空标签-->
<StackPanel> 
   <!--空标签-->
   <Button />
</StackPanel>
```

### 元素特性

为了区分对象的属性（Property），这里将 Attribute 翻译成**特性**，特性的功能是为元素附加额外的信息。特性赋值有两种方式：

- 使用字符串简单赋值
- 使用特性元素复杂赋值

元素的特性部分对应对象的属性及各种事件，但不是一一对应。特性的简单赋值通过`Attribute=“Value”`语法，示例如下。

```xml
<TextBlock Text="用户名"></TextBlock>
```

### 特性元素

特征元素即某元素为另一个元素的某个特征，即以元素的方式展示另一个元素的特征，这有区别于上述在标签中内嵌特性。如果特征的值可以用字符串表达，就不要使用特征元素的表达方式。以下示例代码用特征元素的方式定义了窗口的资源。

```xml
<Window.Resources>
    <sys:String x:Key="motto">BBQ BOYS</sys:String>
</Window.Resources>
```

### 标记扩展

标记扩展主要扩展了特征的取值范围，简单赋值不再单单是个字符串，也可以是静态资源、对象等等。扩展语法是：`Attribute=“{value}”`，用大括号将值括起来。**标记扩展支持嵌套使用**。

标记扩展是一系列派生类，其基类为抽象类 MarkupExtension。所有扩展可分为两大类：XAML 标记扩展和 WPF 标记扩展（绑定），先罗列下 XAML 的标记扩展，下一章节介绍绑定相关。

- x:Null

  显示地设置特性值为 NULL，常用于放弃相关特性的继承。

  ```xml
  <Window.Resources>
      <Style TargetType="Button">
          <Setter>
              <Setter.Property>Button.Background</Setter.Property>
              <Setter.Value>SeaGreen</Setter.Value>
          </Setter>
      </Style>
  </Window.Resources>
  
  <Button Style="{x:Null}">Click Me</Button>
  ```

- x:Static 静态属性扩展

  在 XAML 中访问静态成员

- x:Type 对象类型扩展

  在 XAML 中标记数据类型

- x:Array 静态数组扩展

  在 XAML 中创建静态数组

  ```xml
  <Window.Resources>
      <x:Array x:Key="boys" Type="{x:Type sys:String}">
          <sys:String>猴子</sys:String>
          <sys:String>肉肉</sys:String>
          <sys:String>二哈</sys:String>
      </x:Array>
  </Window.Resources>
  <StackPanel>
      <ListBox ItemsSource="{StaticResource boys}"></ListBox>
  </StackPanel>
  ```

## 绑定

- Binding

  数据绑定扩展，将自身的某个特性与任意数据绑定，MVVM中频繁用到此标记扩展。

  ```xml
  ```

- TemplateBinding

  为了方便复用，可以将控件的样式设置集中到一个控件模板里，然后通过模板绑定的方式应用到具体控件。

  ```xml
  <Application.Resources>
      <ControlTemplate x:Key="RedButton" TargetType="Button">
          <ContentControl Content="Click" Foreground="Red"></ContentControl>
      </ControlTemplate>
  </Application.Resources>
  
  <Button Template="{StaticResource RedButton}"></Button>
  ```

## 资源

通常所谓的资源（二进制资源）指程序代码用到的一些附件内容，比如图标、版权字符串等。为了防止遗失或被破坏，可将资源编译进程序的主体，与程序主体浑然一体。WPF 另外引入了对象资源的概念，每个元素都可携带能被共享的对象资源，比如样式、控件模板、动画组件及画刷等。对象资源在元素的 Resources 特性元素中定义，以“键—值”对的形式存储。

在检索资源时，先查找当前元素的 Resources 特性，如果没有找到再沿着逻辑树去父元素查找，最终直到Application.Resources 元素。如果搜索完所有路径都没发现，就会抛出异常。

根据使用资源的方式不同，分静态绑定资源和动态绑定资源。绑定资源通过前面提到的标记扩展语法实现。

### 静态绑定

当程序初始化时一次性地使用某资源，后续不再关注该资源可能的变化时，使用静态绑定扩展标记。

```xml
<Window.Resources>
    <sys:String x:Key="motto">Good Good Study,Day Day Up</sys:String>
</Window.Resources>

<TextBlock Text="{StaticResource motto}"></TextBlock>
```

### 动态绑定

资源在程序运行后可能发生变化，且被绑定的特性需同步响应变化时，使用动态绑定扩展标记。

```xml
<Window.Resources>
    <sys:String x:Key="motto">Good Good Study,Day Day Up</sys:String>
</Window.Resources>

<TextBlock Text="{DynamicResource motto}"></TextBlock>

// 后台改变资源
private void Button_Click(object sender, RoutedEventArgs e)
{
    Resources["motto"] = "so boring";
}
```

### 资源文件

为了实现工程化及便于复用，可将资源定义在一个单独的资源字典文件中。该文件类似于前端常见的 CSS 样式文件，后缀名为 .xaml。大家熟悉的 MaterialDesign 界面库就是以资源文件的方式提供的，其引入方式如下：

```xml
<Application.Resources>
    <ResourceDictionary Source="pack://application:,,,/MaterialDesignThemes.Wpf;component/Themes/MaterialDesignTheme.Defaults.xaml" />
</Application.Resources>
```

使用示例代码如下：

```xml
<Button Style="{DynamicResource MaterialDesignFlatMidBgButton}">Start</Button>
```















