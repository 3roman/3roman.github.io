Dapper 是一款 .Net 平台下的微型 ORM 框架，特点是：轻量、高效、简单、灵活并适用于 SQL Server、SQLite、MySQL等多款数据库。

Dapper 以 NuGet 包的形式提供，其扩展了 IDbConnection 接口的方法。

## 方法

### Query方法：强类型返回值

方法原型如下：

```csharp
public static IEnumerable<T> Query<T>(this IDbConnection cnn,
               string sql,
               object param = null,
               IDbTransaction transaction = null,
               bool buffered = true,
               int? commandTimeout = null,
               CommandType? commandType = null)
```

示例代码如下：

```csharp
public class Dog
{
    public int? Age { get; set; }
    public Guid Id { get; set; }
    public string Name { get; set; }
    public float? Weight { get; set; }

    public int IgnoredProperty { get { return 1; } }  // 被忽略的字段直接返回固定值
}

var guid = Guid.NewGuid();
var dog = connection.Query<Dog>("SELECT Age = @Age, Id = @Id",
                new { Age = (int?)null, Id = guid });
```

### Query方法：动态类型返回值

方法原型如下：

```csharp
public static IEnumerable<dynamic> Query (this IDbConnection cnn, 
               string sql,
               object param = null,
               IDbTransaction transaction = null,
               bool buffered = true,
               int? commandTimeout = null,
               CommandType? commandType = null)
```

示例代码如下：

```csharp
var rows = connection.Query("SELECT 1 A, 2 B UNION ALL SELECT 3, 4").AsList();
```

### Excute方法：返回受影响行数

Excute 方法返回受影响的行数，一般用于增、删、改操作。

方法原型如下：

```csharp
public static int Execute(this IDbConnection cnn,
            string sql,
            object param = null,
            IDbTransaction transaction = null,
            int? commandTimeout = null,
            CommandType? commandType = null)
```

示例代码如下：

```csharp
var count = connection.Execute(@"
      SET NOCOUNT ON
      CREATE TABLE #t(i int)
      SET NOCOUNT OFF
      INSERT #t
      SELECT @a a UNION ALL SELECT @b
      SET NOCOUNT ON
      DROP TABLE #t", 
      new {a=1, b=2 });
```

### Excute方法：反复执行

该特性支持反复执行同一 SQL 语句，传递对象数组或对象集合（实现了`IEnumerable<T>`集合）作为参数，常用于批量插数据。 

示例代码如下：

```csharp
// 示例1
var count = connection.Execute(@"INSERT MyTable VALUES (@a, @b)",
    new[] { new { a = 1, b = 1 }, new { a = 2, b = 2 }, new { a = 3, b = 3 } });
// 示例2
var foos = new List<Foo>
{
    new Foo { A = 1, B = 1 },
    new Foo { A = 2, B = 2 },
    new Foo { A = 3, B = 3 }

};
var count = connection.Execute(@"INSERT MyTable(colA, colB) VALUES (@a, @b)", foos);
Assert.Equal(foos.Count, count);
```

## 参数

### 参数化查询

SQL 语句采用匿名对象作为参数可实现参数化查询，且与 SQL Server 的参数化查询语法一致。参数化查询有两个好处：

1. 防止 SQL 注入
2. 提高代码复用率

111111111111111111111111111111111111111111

### 单个参数

Dapper 采用匿名对象作为查询参数，对象属性应与 SQL 语句中@符号打头的参数名称一致。参数化查询可以有效地抵御 **SQL注入攻击**。

```csharp
var rows = connection.Query<User>("SELECT * FROM user WHERE username = @UserName And password = @PassWord",
   new {UserName = "admin", PassWord = "xxxxxxx"})  // UserName属性值给到参数@UserName, PassWord属性值给到参数@PassWord 
```

### 多条查询

Dapper 同时支持`IEnumerable<T>`类型的多参数查询，集合中的元素被拆开后依次传入参数会被自动展开。

原始代码如下：

```csharp
var rows = connection.Query<int>("SELECT * FROM (SELECT 1 AS Id UNION ALL SELECT 2 UNION ALL SELECT 3) AS x WHERE Id IN @Ids",
  new { Ids = new int[] { 1, 2, 3 } });
```

展开后代码如下

```csharp
SELECT * FROM (SELECT 1 AS Id UNION ALL SELECT 2 UNION ALL SELECT 3) AS x WHERE Id IN (@Ids1, @Ids2, @Ids3)" // @Ids1 = 1 , @Ids2 = 2 , @Ids2 = 3
```

## 结果

### 一对多映射

Dapper 支持将一条数据库记录映射到多个对象，这个关键特性可以减少额外查询。

相关联的两个实体类如下：

```csharp
class Post
{
    public int Id { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }
    public User Owner { get; set; }
}

class User
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```

// TODO

### 区别映射

一般情况下，同一个表的返回数据都被映射到同一个实体类，但偶尔也需要映射到不同的实体类，这时`IDataReader.GetRowParser`方法就派上用场了。

Shapes 表包含 Id、Type 和 Data 三个字段，依据 Type 字段值分别映射给 `Circle`、`Square`、`Triangle`三个实体类。

示例代码如下：

```csharp
var shapes = new List<IShape>();
using (var reader = connection.ExecuteReader("SELECT * FROM Shapes"))
{
    // Generate a row parser for each type you expect.
    // The generic type <IShape> is what the parser will return.
    // The argument (typeof(*)) is the concrete type to parse.
    var circleParser = reader.GetRowParser<IShape>(typeof(Circle));
    var squareParser = reader.GetRowParser<IShape>(typeof(Square));
    var triangleParser = reader.GetRowParser<IShape>(typeof(Triangle));

    var typeColumnIndex = reader.GetOrdinal("Type");

    while (reader.Read())
    {
        IShape shape;
        var type = (ShapeType)reader.GetInt32(typeColumnIndex);
        switch (type)
        {
            case ShapeType.Circle:
            	shape = circleParser(reader);
            	break;
            case ShapeType.Square:
            	shape = squareParser(reader);
            	break;
            case ShapeType.Triangle:
            	shape = triangleParser(reader);
            	break;
            default:
            	throw new NotImplementedException();
        }

      	shapes.Add(shape);
    }
}
```

## 其他

### 多参数重复执行

Dapper 支持一次函数调用中执行多条查询语句，并将返回结果分别映射到对应的实体类。

示例代码如下：

```csharp
var sql =
@"
SELECT * FROM Customers WHERE CustomerId = @id
SELECT * FROM Orders WHERE CustomerId = @id
SELECT * FROM Returns WHERE CustomerId = @id";

using (var multi = connection.QueryMultiple(sql, new {id=selectedId}))
{
   var customer = multi.Read<Customer>().Single();
   var orders = multi.Read<Order>().ToList();
   var returns = multi.Read<Return>().ToList();
}
```

### 字面量替换

// TODO

### 流式读取

首先了解一下数据适配器 **SqlDataAdapter** 和 数据读取器 **SqlDataReader **的主要区别：

1. SqlDataAdapter 在建立数据库连接后一次性读取所有查询数据到内存，然后就关闭了连接。适用于少量查询结果。
2. SqlDataReader  在建立数据库连接后流式读取数据，待发送缓存装满再返回查询数据，如此重复多次直至所有数据被返回。期间数据库连接一直处于 open 状态，且被当前 SqlDataReader 独占。适用于大量查询结果。

Dapper 默认一次性读取所有查询结果，这样有利于最小化共享锁的占用时长和数据库的连接时长。这种读取方式的弊端也很明显：返回大量结果时内存耗费巨大。未解决此弊端，调用`Query`方法时`buffered`形参应传递`false`，改用流式读取方式。

### 存储过程

Dapper 支持使用存储过程。

示例代码如下：

```csharp
var user = cnn.Query<User>("spGetUser", new {Id = 1},
        commandType: CommandType.StoredProcedure).SingleOrDefault();
```

花哨点的写法如下：

```csharp
var p = new DynamicParameters();
p.Add("@a", 11);
p.Add("@b", dbType: DbType.Int32, direction: ParameterDirection.Output);
p.Add("@c", dbType: DbType.Int32, direction: ParameterDirection.ReturnValue);

cnn.Execute("spMagicProc", p, commandType: CommandType.StoredProcedure);

var b = p.Get<int>("@b");
var c = p.Get<int>("@c");
```

### varchar 类型支持

Dapper 支持 varchar 类型，当对 varchar 类型字段执行 `WHERE ` 字句时，应这样传递参数：

```csharp
var things = Query<Thing>("SELECT * FROM Thing WHERE Name = @Name", new {Name = new DbString { Value = "abcde", IsFixedLength = true, Length = 10, IsAnsi = true });
```

## 注意事项

正如上文所述，Dapper 默认一次性缓存所有查询结果，这样可加快映射到实体类的进度。当前缓存机制依赖于`ConcurrentDictionary`对象，一次性的查询语句会经常刷新该对象缓存。另外，如果没有查询字句的约束，一下子取回整表数据可能造成内存暴涨。

Dapper 之所以简约是因为剥离了传统 ORM 框架的很多功能。即便如此，也能应付大约95%的应用场景，千万不要期望本框架能处理所有问题。

