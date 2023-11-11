
## 问题描述

有时为了测试代码，我们会用VS Code在Node环境下调试js文件。当代码中使用了**import**关键字，运行会报如下错误。

> (node:10048) Warning: To load an ES module, set "type": "module" in the package.json or use the .mjs extension.
>
> (Use `node --trace-warnings ...` to show where the warning was created)
>
> c:\repo\main.js:1
>
> import math from "./js/math"

上述大概意思是：为了能够加载ES6模块，应在**package.json**配置文件中增加**type**字段，并将值设为**module**或将后缀名改为**mjs**。

<!-- more -->

## 解决方法
经测试，第2种方法需将所有相关文件后缀名都改为**mjs**，包括直接、间接依赖的包文件。如此处理工作量太大，更麻烦的是改变了包内文件结构，可能带来潜在问题。故推荐在**package.json**文件中增加并修改**type**字段。该文件不存在则用`npm init -y`命令生成。

```js
{
  "name": "repo",
  "version": "1.0.0",
  "description": "",
  "main": "main.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "type": "module"
}
```

## 原因分析

对于模块化的实现，CommonJS使用`module.exports`导出及`require()`加载；而ES6分别使用`export`和`import`关键字导出和导入。当源码中出现`export`或`import`关键字，Node就认为使用了ES6模块；接着再去检查后缀名是否为**mjs**或者**type**字段的值是否为**module**；否则就报错。

### 文件后缀

针对不同的后缀名，Node处理原则如下：

- **.mjs**文件总是以ES6模块加载
- **.cjs**文件总是以CommonJS模块加载
- **.js**文件的加载取决于**package.json**里面**type**字段值

### type字段

该字段决定了.js文件和无后缀名文件的加载方式。值为**commonjs**时，以commonjs模式加载；值为**module**时，以ES6模式加载；字段不存在时，默认以commonjs模式加载。



