## 阻塞与非阻塞

“阻塞”一词的字面意思是被迫停滞，无法前进，很容易联想到日常生活中的堵车。大家都碰到过程序“假死”，光标一直转圈圈，键鼠也不听使唤。这种现象背后的原因是 UI 线程被阻塞，此刻后台大多在进行下载文件、查询数据库等I/O密集的操作。

![img](https://i.pinimg.com/originals/e0/9c/bc/e09cbc00e6992f3c31c78200b3a05df1.jpg)



**阻塞与非阻塞的区别在于应用层在等待请求结果期间的状态。**阻塞模式下应用层发出请求后一直死等结果，直到结果全部返回，这期间啥也不干。举个例子：网格长在小区群里发了条消息，要求各家各户上报假期出行情况，随后她就去买菜做饭了，这是非阻塞。如果她一直傻乎乎地捧着手机，等待所有家庭的答复，外面有人敲门也不管，这是阻塞。

代码默认就是按阻塞模式执行的，因为语句总是一句接一句往下走。只不过大多数语句执行的很快，看不出阻塞来。只有在执行耗时语句时，才体会到阻塞的存在。MFC 程序为防止主线程被阻塞，进而导致程序“假死”，通常会新建工作线程来执行耗时操作。 JS 本身是单线程的，其它线程只能由宿主（浏览器、NodeJS等）创建，这些线程不属于 JS。 

## 同步与异步

同步、异步与阻塞、非阻塞是相互独立的。**同步与异步的区别在于是否主动确定结果的状态。**当同步请求发出后，应用层会主动向内核层发起问询。倘若结果还没出来，同时处于阻塞模式下，应用层就只会不断发起问询；若当前处于非阻塞模式，应用层暂时去干别的，过会儿再来问询。异步请求不会主动问询，结果出来后由“周到的”内核层通知给应用层，这种模式当然也分阻塞和非阻塞。举个例子：发工资那天，不停地刷新银行 APP，查看工资是否到账等同于同步操作；安心等待工资到账的短信息等同于异步操作。

## 异步非阻塞

综上所述，一个高效、理想的模型应该是异步非阻塞模型。应用层发完请求后就去干别的事情，避免了无效的的等待；结果出来后由内核层去通知应用层，避免了盲目的问询。异步非阻塞模型离不开多线程，但上文提到 JS 是单线程的，因此只能由宿主创建诸如下载、渲染等线程。JS 主线程只负责盯着自己的任务队列，不停地将队列中的任务拿去执行。耗时线程操作执行完毕后发出消息，宿主收到消息后将对应的回调函数放到主线程的任务队列中，这样就实现了异步。

## Promise的由来

上节说到，通过回调函数可以实现异步，但该方案存在**回调地狱**的麻烦。比如有这样的需求：下载 ZIP包，破解 ZIP包，解压 ZIP包，发送解压后的文件，一环扣一环。最初代码如下，看了叫人头皮发麻，比套娃还能套。

```js
downloadZIP((file) => {
  CrackZIP(file, (file, password) => {
    UnZIP(file, password, (neFile) => {
      SendFile(neFile);
    });
  });
});
```

于是乎`Promise`诞生了 ，使用`Promise`将上述代码改成链式调用，顺眼了很多。有一点要注意：在可能的异常发生和调用`Reslove`、`Reject`函数之前，`Promise`会一直处于`Pending`状态， 因此`then`、`catch`方法也不会执行。

```javascript
downloadZIP()
.then( file => CrackZIP(file))
.then( (file, password ) => UnZIP(file, password) )
.then( newFile => SendFile(newFile));
```

## async 与 await

`async`与 `await` 是 ES6 中引入的语法糖，目的是简化`Promise`的使用。引用阮一峰老师说法：

> async 函数完全可以看作多个异步操作包装成的一个 Promise 对象，而 await 命令就是内部 then 方法的语法糖。

示例用法如下：

```javascript
import fetch from "node-fetch";

const response = await fetch("https://api.github.com/users/github");
const json = await response.json();
console.log(json);
```

任何方法用`async`修饰后方法都返回`Promise`对象，这与该方法是否用`await`调用无关。它两经常一起出现，但却是相互独立的。

```javascript
// 原始代码
async function foo() {
  return 1;
}

console.log(foo()); // 打印Promise { 1 }
```

`await`相当于执行了`Promise`对象的`then`方法

```js
async function foo() {
  return 1;
}

console.log(await foo());  // 打印1
```

甚至可以将混用 Promise、async、await，验证代码如下：

```javascript
// 混用示例1
const p1 = new Promise((resolve, reject) => {
  resolve(1);
});
console.log(await p1);   // 打印1

// 混用示例2
async function foo() {
  return 1;
}
foo().then((value) => console.log(value));    // 打印1
```

### 注意事项

1. await语句之间是阻塞的。本节开头的示例代码中，在`fetch`方法返回后才会就继续执行`json()`方法。这也符合逻辑，数据还没拿回来怎么加工？如果想让多个`Promise`并行运行，应使用`Promise.all`方法。

   ```javascript
   const promise1 = fetch("https://api.github.com/users/github");
   const promise2 = fetch("https://api.google.com/users/github");
   Promise.all([promise1, promise2]);
   ```

2. `foreach`、`map`中使用`await`会立即返回，`await`只适用于传统`for`循环或者直接使用`for await`语句。