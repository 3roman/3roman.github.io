nameof 是 C# 6.0新增的表达式，与 sizeof 不同， 它不是运算符。MSDN 上 关于 nameof 的介绍很简单，但作用却不简单。

nameof 用于获取变量、类型或成员的名称，可避免类名或属性名作为参数时使用 hardcode，以防后续重构困难。同时 nameof 也可部分代替反射，提高了程序的性能。下面介绍两个 nameof 的典型应用场景。

![](https://raw.githubusercontent.com/3roman/PicBed/master/hexo/nameof.png)

## 属性通知

MVVM 设计模式中，属性值改变时要触发相关事件，同时传递属性名称作为事件的参数，以告知 View 具体是哪个属性发生了变化，代码一般写在属性的 set 访问器中。原始写法和改进写法对比如下：

```csharp
public class Stock: INotifyPropertyChanged 
{
    private int _price;
    public int Price
    {
     	get { return _price;
	}
set 
{
    this.OnPropertyChanged("UserName");
    // 原始写法使用了hardcode
    this.OnPropertyChanged(nameof(Price));
    // 改进写法可被同步改名
    _price = value;
}
}
private void OnPropertyChanged(string propertyName) 
{
	this.PropertyChanged.Invoke(this, new PropertyChangedEventArgs(propertyName));
}
	public event PropertyChangedEventHandler PropertyChanged;
}
```

使用 hardcode 的原始写法存在两个明显弊端：

1. 排错难

   若将字符串字面量`"Price"`误写成`"Priec"`，编译能通过，运行后还是无法实现通知功能。这类错误往往很难排查。

2. 重构烦

   当修改属性名时应同步修改对应的字符串参数，改多了就容易遗漏，当然一个一个改起来本身就很繁琐。

## 数据库字段映射

使用 Dapper 这类 ORM 框架时，SQL 语句中的表名和字段名应与对应 POCO 类的类名和属性名一致，否则无法建立映射关系。以往做法是写死 SQL 语句中的表名和字段名，造成后期维护麻烦，比如：

```csharp
var record = DbConnection.QueryFirstOrDefault($"SELECT TOP 1 FROM Stock WHERE Name={tesla.Name}");
```

上例中，表名 Tesla 和 字段名 Name 都是写死的。后期若要重构 POCO 类，逐条修改 SQL 语句要累成狗。使用 nameof 改成以下写法，按按【Ctrl+R+R】轻松搞定，高下立判！

```csharp
var record = DbConnection.QueryFirstOrDefault($"SELECT TOP 1 FROM {nameof(Stock)} WHERE {nameof(Stock.Name)} = {tesla.Name}");
```

## 非完全限定名

nameof 返回的是非完全限定名，如果需要完全限定名就只能靠反射了，这点需要留意。

```csharp
Console.WriteLine(nameof(System.Console)); // 打印 Console
Console.WriteLine(typeof(System.Console).FullName); // 打印 System.Console
```

