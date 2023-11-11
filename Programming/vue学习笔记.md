## 前言

### 简介

[官网](https://cn.vuejs.org)对vue的定义如下：

> Vue (读音 /vjuː/，类似于 view) 是一套用于**构建用户界面**的**渐进式框架**。

关键词：

- 构建用户界面

  vue扮演了MVVM编程范式中的ViewModel角色

- 渐进式

  按需逐步引入vue提供的功能，不必夯不啷当全部引入。

<!-- more -->

### 特点

1. 组件化

   组件化是个很老的概念，vue组件定义在一个单独的**.vue**文件中。

2. 声明式

   只需明确需求而不必具体实现，比如遍历一个数组，`v-for`指令用起来就很简单：

   ```vue
   <li v-for="student in students">
     student.name
   </li>
   ```

   对于大多数编程语言，代码可能如下：

   ```c++
   std::vector<std::string> students = { "小明", "丽丽", "大黄" };
   for (int i = 0; i < students.size(); i++)
   {
   	std::cout << students[i] << std::endl;
   }
   ```

3. 虚拟DOM

   真实DOM结构复杂，频繁操作性能消耗大，直观感受是网页卡。于是vue发明了虚拟DOM，变化后生成新的虚拟DOM，再用**diff**算法对比新旧虚拟DOM，最后将变化的部分一次性放到真实DOM上。

### 术语

1. property

   对象属性，比如以下代码中的name属性。

   ```javascript
   const person = {
       name: 'wanger'
   }
   ```

2. attribute

   html元素属性，比如以下代码中的src属性。

   ```html
   <script src="www.google.com"></script>
   ```

3. 函数（function）

   实现特定功能的代码段

4. 方法（method）

   对象或类中的的函数

   ```javascript
   const person = {
       eat: function(){...},
       // ES6中方法简写
       run(){...}
   }
   ```

5. 配置对象

   用于传递参数的对象，一般都是匿名对象。

   ```javascript
   const vm = new Vue({
       el: '#app',
       data: {
           message: 'hello vue'
       }
   })
   ```

6. 数据对象

   用于存储数据的对象，比如以下代码中的data对象。

   ```javascript
   const vm = new Vue({
       el: '#app',
       data: {
           name: 'jerry',
           age: '5'
       }
   })
   ```

7. 表达式

   变量和操作符的结合，一般至少返回一个值，比如：`1+1`

8. 语句

   一种操作或命令，会对周围产生影响，比如：`year = 2021`
   
9. 元素

   HTMl元素通常由一对开始标签和结束标签组成，中间部分称为标签体。标签体为空的标签称为空标签，除script标签除外，空标签可以简写成自闭标签的型式。

   ```html
   <button type="button">点击我</button>
   <u>
   ```
   
10. 

## 基础

### 入门示例

新建html文档，使用script标签远程引入vue.js，这里要知道：

> 一旦成功引入，将创建一个全局Vue函数，实际也是对象构造方法。

以下是一段hello world示例：

```html
<html>
  <head>
    <script src="https://unpkg.com/vue/dist/vue.js"></script>
  </head>

  <body>
    <div id="app">
      <strong>{{ message }}</strong>
    </div>

    <script>
      new Vue({
        el: "#app",
        data: {
          message: "hello word",
        },
      });
    </script>
  </body>
</html>

```

#### 挂载

配置对象的el属性指定了vue实例将挂载的节点（一般为容器元素），也可通过原型方法`$mount`挂载，并且更加灵活。一旦某节点被挂载了vue实例，当前节点及其子节点将由vue接管。当数据对象发生变化时，相关节点将被重新渲染。

```html
<script>
  const vm = new Vue({
        // el: "#app",
  });
  vm.$mount("#app");
</script>
```

本例使用了id选择器定位目标节点，也可以使用类选择器，或者直接调用`Document.getElementById()`函数获取。

#### 数据

data属性值也是一个对象，称之为数据对象，里面定义了模板中会用到的数据。数据对象的所有属性会自动附加到vue实例上。为了避免全局作用域污染，组件中使用函数返回数据对象。

### 版本区别

形式上的区别：生产版去除了注释、警告信息并采用gzip压缩，因此体积更小；开发版文件名为**vue.js**，生产版文件名为**vue.min.js**。最重要区别在于：开发版错误提示更加详细、友好。上节例子中把`new`关键字拿掉后，错误提示如下：

![image-20210827232259448](https://gitee.com/romango/picbed/raw/master/picgo/image-20210827232259448.png)

### 模板语法

通过vue提供的专有模板语法，可以将html元素相关内容与vue实例建立关联。

#### 标签体插值

插值只用在标签体中，采用双大括号的**mtstache**语法。双大括号内可以插入js表达式，并且可以引用数据对象的属性。

```html
<h1>my name is {{ name }}</h1>
<h1>{{ number }} is {{ 0 === number%2 ? 'even' : 'odd' }} number</h1>
```

#### 属性绑定

1. v-bind

v-bind指令用于绑定元素属性，由于该指令经常用到，可以省略指令名称。同标签体插值一样，属性中也可以插入js表达式。

```html
<body>
    <div id="app" style="display: flex;flex-flow: column;">
      <a href="https://3roman.github.io">我的主页</a>
      <a v-bind:href="site">我的主页</a>
      <a :href="site">我的主页</a>
    </div>

    <script>
      new Vue({
        el: "#app",
        data: {
          site: 'https://3roman.github.io'
        },
      });
    </script>
  </body>
```



2. v-model

数据绑定一般是单向的，即数据从vue实例流向html元素。v-model指令实现了表单类元素的双向绑定，**但该指令只能用于表单类元素。**v-model实际就是一个语法糖，它实现了对如下代码的包装。

`<input v-bind:value="name" v-on:input="name = $event.target.value">`

v-model指令默认收集的是value属性值，因此可将`v-model:value="name"`简写成`v-model="name"`。

#### 计算属性

计算属性具备缓存功能，这是方法调用无法比拟的，频繁操作时性能更好。

只有当计算属性首次被读取或依赖的属性发生变动时才会重新调用get访问器。比如以下示例代码中，控制台只输出了一次信息。

```html
<body>
    <div id="app">
        <p> {{ fullName }}</p>
        <p> {{ fullName }}</p>
        <p> {{ fullName }}</p>
    </div>

    <script>
        const vm = new Vue({
            el: "#app",
            data: {
                firstName: "王",
                lastName: "二"
            },
            computed: {
                fullName: {
                    get() {
                        console.log('get访问器');
                        return this.firstName + this.lastName;
                    }
                }
            }
        })
    </script>
```

计算属性中的this指向vue示例，这是vue自动帮我们实现的。

get访问器只关注依赖的属性。set访问器实际很少用到，当只有get访问器时，可简写成方法的型式，方法名即为属性名。

```javascript
computed: {
    fullName() {
        console.log('get访问器');
        return this.firstName + this.lastName;
    }
}
```

#### 监视属性

TODO

#### 条件渲染

`v-show`和`v-if`都能控制节点是否在页面中出现。`v-if`控制当前节点是否渲染，而`v-show`始终渲染节点，只是改变了样式的**display**属性值。如果频繁切换显示，建议使用`v-show`，性能更优。

值得注意的是：多个if条件与if-else语法结构存在差异。示例1代码中会将3个条件依次判断一遍，不管上一个条件是否已满足；示例2代码中当某个分支条件满足后，将跳过后续判断。

```html
<!-- 示例1 -->
<p v-if="2021 === year">牛年</p>
<p v-if="2020 === year">鼠年</p>
<p v-if="2019 === year">豬年</p>
<!-- 示例2 -->
<p v-if="gender">男</p>
<p v-else-if="!gender">女</p>
<p v-else>人妖</p>
```

if-else结构中，多个分支必须连续，否则将不起作用。

单个if控制多个标签时，建议使用`template`标签，不推荐使用`div`标签，因为后者会在DOM结构中增加额外节点，但`template`只能配合`v-if`指令使用。

```html
<!-- 不推荐 -->
<div v-if="isAdmin">
  <a>发表文章</a>
  <a>管理用户</a>
</div>
<!-- 推荐 -->
<template v-if="isAdmin">
  <a>发表文章</a>
  <a>管理用户</a>
</template>
```

`v-if`配合`div`和`template`的区别：

![image-20210829215041119](https://gitee.com/romango/picbed/raw/master/picgo/image-20210829215041119.png)

#### 循环渲染

#### 事件绑定

## 组件

## 路由

## vuex