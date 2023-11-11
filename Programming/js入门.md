## 前言

 JavaScript是一种类C的脚本语言，目前在Web前端、后端都有非常广泛的应用。js其实与Java并无关联，当年为了沾上Java的热度就起名叫JavaScript。

对于大多数程序而言，程序=数据+算法，因此学习一门新的编程语言应从这两个方面学起，后续在逐渐深入该语言的特有的内容。

## 数据

### 基本数据类型

1. number

   整型、浮点型、非数字`NaN`、无穷大`Infinity`4种数值类型。

   ```javascript
   let num = 1 / 0; // Infinity
   num = "hello" - 1; // NaN
   ```

   浮点型占用整型2倍的内存空间，因此JavaScript会不失时机地将浮点型数值转换为整型数值。

   ```javascript
   var n = 2.0 // 自动转换为2
   ```

   > 任何语言任何时候都不要与浮点型数值作比较

2. string

   用字符串模板代替字符串拼接

   

   ```javascript
    let age = 20
    console.log(`I am ${age} years old`);
   ```

3. 常见类型与bool类型转换关系表

   | 数据类型  | true                     | false         |
   | :-------: | ------------------------ | ------------- |
   |  string   | 任何非空字符串           | “”（空字符串) |
   |  number   | 任何非零数值(包括无穷大) | 0和NaN        |
   |  object   | 任何对象                 | null          |
   | undefined | n/a                      | undefined     |

4. undefined

   所有仅被声明而未被赋值的变量，JavaScript会分配一个默认值`undefined`。

   ```javascript
   let slogan
   console.log(slogan); // undefined
   ```

5. null

   如果一个变量将来用于存储对象，通常将其初始化为`null`。

6. object

7. array

   - JavaScript属于弱类型编程语言，因此array不会对数据类型进行检查，即同一数组中可以不同类型的数据；
   - 出于性能考虑，不建议数组头部进行操作。

   ```javascript
   let fruits = ["apple", "banana"];
   fruits.push(2020); // apple, banana. 2020
   fruits.pop(); // 2020
   ```

### 常量与变量

变量命名必须以字母、下划线(_)或美元符号($)打头；类名使用大驼峰命名法；常量使用大写+下划线命名法；其余使用小驼峰命名法。

不建议改变变量数据类型

```javascript
let slogan = "hello javascript";
slogan = "hello world"; // 允许
slogan = 2020; // 允许但不推荐
```

除非为兼容旧版本，否则一律使用`let`关键字声明变量。`let`不支持变量提升，即变量在被使用之前必须先被声明；其次`let`不允许重复声明变量；以上特点可避免~~var~~关键字可能带来的混乱。

常量必须初始化，即声明时就赋初值。

```javascript
let slogan;
slogan = "hello javascript";
const PI = 3.1415926;
```

### 动态特性

JavaScript属于动态型编程语言，允许同一变量在运行周期里存储不同类型的数据，**注意：这不是类型推断。**

```javascript
let x = "hello js";
typeof x // => 'string'
slogan = 2021;
typeof x // => 'number'
```

## 算法

### 语句

单行语句以分号结尾；花括号包起来的几行语句即多行语句。

```javascript
{
    temp = a;
    a = b;
    b = temp;
}
```

### 运算符

1. 递增与递减

前置与后置递增、递减运算结果都相同，区别在于赋值运算次序不同。

  ```javascript
  let price1 = 100, price2 = 100;
  let newPrice1 = price1++;
  console.log(newPrice1); // 100
  let newPrice2 = ++price2;
  console.log(newPrice2); // 101
  ```

2. 比较

- 数字与字符串比较时，JavaScript会先将字符串隐式转换为数字后再进行比较。
  
  ```javascript
  "20" > 10; // Number("20") > 10 → 20 > 10  → true
  "apple" > 10; // Number("apple") > 10 → NaN > 10 → false
  ```

- 抽象相等`==`

  比较对象类型一致时，随即比较值是否相等；不一致时，尝试隐式类型转换后再比较值是否相等。
  
  ```javascript
  10 == "10" // 10 == Number("10") → 10 == 10 → true
  ```

- 严格相等`===`

  严格以上的相等（类型与值都相等），本质是**严格相等不会尝试类型转换**。
  
  ```javascript
  10 === "10" // false
  ```

> 在不清楚具体应用场合的前提下，尽量使用严格相等`===`或者严格不相等`!==`。

### 控制结构

1. 多条件判断过程

对于多条件判断语句（逻辑与`&&`和逻辑或`||`），当根据条件1就可以得到整体判断结果，后续条件将不再执行。

```javascript
let dog = null;
if (2 > 1 || dog.age > 5) {
    console.log("old dog");
} // 正常输出：old dog，不会报错
if (dog.age > 5 || 2 > 1 ) {
    console.log("old dog");
} // 报错：Cannot read property 'age' of null
```

2. 分支语句break关键字

如果缺少`break`关键字，条件成立部分语句及后续无`break`关键字语句都将会被执行；此时`case`关键字起到的是`goto`的作用，这与`if...else if...`语句功能不同。

```javascript
let flag = 2;
switch (flag) {
    case 1: {
        console.log(1);
    }
    case 2: {
        console.log(2);
    }
    case 3: {
        console.log(3);
    }
    default: {
        break;
    }
} // 输出2、3
```

3. 遍历

`for...in`语句用于遍历数组的元素或者对象的属性。不同于C#语言，JavaScript语言中可对集合进行读写。

```javascript
var stuff = {name:"wanger", age:30, married:false}; 
for (let x in stuff) {
    console.log(`${x}:${stuff[x]}`);
}
```

## 其它

### 注释

虽然JavaScript支持行注释和块注释，但是编辑器都是以行注释的型式生成注释。

   ```javascript
// function add(x, y){
//     return x + y;
// }
   ```

## ES6

### 填坑

1. 大胆抛弃`var`关键字，请用`let`和`const`代替
2. `import`模块时不建议省略后缀名，主要怕引起诸如`math.js`和`math\index.js`的歧义。


### 简化

1. 简化对象方法声明
   
   ```javascript
   const coder = {
       // sayHello: function(){
       //     console.log("hello js");
       // }
       // 简化写法
       sayHello(){
           console.log("hello js");
       }
   }	
   ```
   
2. 使用箭头函数简化方法定义
   
   ```javascript
   numbers.sort(function(a, b){return a - b});
   // 简化写法
   numbers.sort((a,b) => a - b);
   
   // 注意：用箭头函数定义实例方法时，this一般不指向实例
   const tommy = {
       name:'tommy',
       sayHello() {
           console.log(`this is ${this.name}`);
       },
   
       sayBye: () => {
           console.log(`this is ${this.name}`);
       }
   
   }
   
   tommy.sayHello(); // this is tommy
   tommy.sayBye(); // this is undefined 
   ```


### 新意思

1. 高阶函数
   
   ```javascript
   var diameters = [3, -1, 0, 2, 1];
   diameters = diameters.filter(d => d > 0);
   console.log(diameters); // [ 1, 2, 3 ]
   areas = diameters.map(d => 0.785 * d ** 2);
   console.log(areas); // [ 0.785, 3.14, 7.065 ]
   var totalAreas = areas.reduce((x,y) => x + y);
   console.log(totalAreas); // 10.99 = 0.785+ 3.14 + 7.065
   ```


2. 解构
   
   ```javascript
   function SteamPro(press, temp) {
       var density, entropy, enthalpy, viscosity;
       // balabala
       return { density, entropy, enthalpy, viscosity };
   }
   
   const { density, entropy, enthalpy, viscosity } = SteamPro(press, temp);
   ```

3. 展开运算符
   
   ```javascript
   // 支持数组和对象
   const numbers1 = [1, 2, 3];
   const numbers2 = [4, 5, 6];
   const all = [...numbers1, ...numbers2];
   console.log(all); // [ 1, 2, 3, 4, 5, 6 ]
   ```

4. 类的声明和继承
   
   ```javascript
   class Person{
       constructor(name){
           this.name = name;
       }
   
       sayHello(){
           console.log(`hi, i am ${this.name}`);
           
       }
   }
   
   class Teacher extends Person{
       constructor(name, course){
           super(name);
           this.course = course;
       }
   
       slogan(){
           console.log(`${this.name} isn't a ${this.course} teacher, but a farmer!`);
       }
   }
   
   const wanger = new Teacher("wanger", "math");
   wanger.slogan();
   
   ```

5. 模块化
   
   ```javascript
   function add(x, y) {
       return x + y;
   }
   
   function square(x) {
       return x ** 2;
   }
   
   export default { add, square };
   
   // 调用模块
   import math from './math.js'
   
   console.log(math); // { add: [Function: add], square: [Function: square] }
   math.add(1,1);
   ```

