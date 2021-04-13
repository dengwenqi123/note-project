## NodeJS Promise

## Promise是什么？

1.抽象表达

​	Promise 是一门新的技术(ES规范)

​    Promise 是JS中进行异步编程的新解决方案。备注：旧方案是单纯使用回调函数

2. 具体表达：

   1） 从语法上来说：Promise是一个构造函数

   2） 从功能上来说：Promise对象用来封装一个异步操作并可以获取其成功/失败的结果值。

   ```javascript
   btn.addEventListener('click',function(){
     //Promise 形式实现
     // resolve 解决 函数类型的数据
     //reject 拒绝 函数类型
     const p = new Promise((resolve,reject) =>{
       if (n <=30){
         //将promise对象的状态设置为成功
         resolve(n);
       }else {
         //将promise 对象的状态设置为 失败
         reject(n);
       }
     })
   })
   // 调用 then 方法
   p.then(()=>{
     alert('')
   },()=>{
     
   })
   ```

   fs 模块的作用，对磁盘操作

   ```javascript
   const fs = require('fs');
   fs.readFile('./resource/conent.txt',(err,data) => {
     if(err) throw err;
     console.log(data.toString());
   })
   //promise形式
   let p = new Promise((resolve,reject) =>{
     fs.readFile('./resource/content.txt',(err,data) => {
       if(err) reject(err);
       //如果成功
       resolve(data);
     });
   });
   p.then(value => {
     console.log(value.toString());
   },reason =>{
     console.log(reason);
   })
   ```

   ```java
   const util = require('util')
   const fs = require('fs')
     let mineReadFile = util.promisify(fs.readFile)
   function mineReadFile(path){
     return new Promise((resolve,reject) => {
       //读取文件
       require('fs').readFile(path,(err,data) => {
         if(err) reject(err);
         resolve(data);
       })
     })
   }
   Promise.resolve(521)
   Promise.reject(123 )
   ```

   ### promise的状态改变

   1. pending 变为 resolved
   2. pending 变为 rejected

   说明：只有这2种，且一个promise对象只能改变一次 无论变为成功还是失败，都会有一个结果数据 成功的结果数据一般称为value，失败的结果数据一般称为reason 

   ### 如何使用Promise?

   一个promise指定多个成功/失败回调函数，都会调用吗？

   当promise改变为对应状态时都会调用

   

   ```javascript
   let p = new Promise((resolve,reject) => {
     resolve('ok');
   });
   let result = p.then(value =>{
     console.log(value)
   },reason => {
     console.warn(reason);
   });
   console.log(result)
   ```

   

   

   Promise 构建函数： Promise(executor){}

   1) executor 函数： 执行器(resolve,reject) =>{}

- promise是一个对象，对象和函数的区别就是对象可以保存状态，函数不可以(闭包除外)
- 并未剥夺函数return的能力，因此无需层层传递callback,进行回调获取数据
- 代码风格，容易理解，便于维护
- 多个异步等待合并便于解决

```nodejs
new Promise((resolve,reject) =>{
	resolve('成功')
}).then((res) =>{console.log(res)},(err) =>{console.log(err)})
```

- resolve作用是，将Promise对象的状态从“未完成”变成“成功”(即从pending变为resolved),在异步操作成功时调用，并将一异步操作的结果，作为参数传递出去；reject作用是，将Promise对象的状态从"未完成"变为“失败”(即从pending变为rejected),在异步操作失败时调用，并将异步操作报错信息，作为参数传递出去。

- promise有三种状态：

  - Pending[待定]初始状态
  - Fulfilled[实现]操作成功
  - rejected[被否决]操作失败

  当promise状态发生改变，就会触发then()里的响应函数处理后续步骤，p romise状态一经改变，不会再变。

- Promise对象的状态改变，只有两种可能：

  - 从pending变为fulfilled

  - 从pending 变为rejected

    这两种情况只要发生，状态就凝固了，不会再变了。

#### 错误处理

Promise会自动捕获内部异常，并交给rejected响应函数处理。

### Promise.all() 批量执行

Promise.all([p1,p2,p3]) 用于将多个promise实例，包装成一个新的Promise实例，返回的实例就是普通的promise它接收一个数组作为参数，数组里可以是Promise对象，也可以是别的值，只有Promise会等待状态改变，当所有的子Promise都完成，该Promise完成，返回值是全部值的数值，有任何一个失败，该Promise失败，返回值是一个失败的子Promise结果。

## async/await 基础

首先，我们使用async 关键字，把它放在函数声明之前，使其成为async function。异步函数是一个知道怎样使用await关键字调用异步代码的函数。

将async 关键字加到函数声明中，可以告诉它们返回的是promise,而不是直接返回值。此外，它避免了同步函数为支持使用await 带来的任何潜在的开销。在函数声明为async时，JavaScript引擎会添加必要的处理，以优化你的程序。

#### await关键字

当await关键字与异步函数一起使用时，它的真正优势就变得明显了,事实上，await只在异步函数里面才起作用。它可以放在任何异步的，基于promise的函数之前。它会暂停代码在该行上，直到promise完成，然后返回结果值。在暂停的同时，其他正在等待执行的代码就有机会执行了。

您可以在调用任何返回Promise的函数时使用await,包括Web API函数。

### 概述

ES6 出来可以通过Promise 来进行异步处理，

- async 是异步简写，而await 可以认为是async wait 的简写，所以 async用于声明一个异步的function,而await用于等待一个异步方法执行完成。
- 简单理解：async 是让方法变成异步，await是等待异步方法执行完毕。

```bash
async function test() {
	return "hello world!"
}
async function main() {
	var data = await test();
	console.log(data);
}
```

async/await 和 Promise 的关系，用一句话总结，就是async function 就是返回 Promise的function。

Async 函数返回的是一个Promise对象，可以使用then方法添加回调函数。当函数执行的时候，一旦遇到await就会线返回，等到触发的异步操作完成，再接着执行函数体内后面的语句。







```javascript
function Promise(executor){
  this.PromiseState = 'pending';
  this.PromiseResult = null;
  const self = this;// self _this that
  //resolve 函数
  function resolve(data){
    this.PromiseState='fullfilled'
    this.PromiseResult = data;
    //修改对象的状态(promiseState)
    // 设置对象的结果值(promiseResult)
    
  }
  function reject(data){
    self.PromiseState = 'rejected';
    self.PromiseResult = data;
    
  }
  executor(resolve,reject);
}
Promise.prototype.then = function(onResolved,onRejected)
```

​	

```javascript
//返回一个Promise 对象
async function hello(){
  //1. 如果返回值时一个非Promise类型的数据
  return 521;
  //2.如果返回的是一个Promise对象
  return new Promise((resolve,reject) => {
    resolve('ok');
  })
}
let result = main();
```

### await 表达式

1. await 右侧的表达式一般为promise 对象，但也可以是其它的值
2. 如果表达式是promise对象，await返回的是promise 成功的值
3. 如果表达式是其它值，直接将此值作为await的返回值

**注意**

1. await必须写在async 函数中，但async 函数中可以没有await
2. 如果await 的promise失败了，就会抛出异常，需要通过try...catch 捕获处理

```javascript
let p = new Promise((resolve,reject) => {
  reject('Error');
})
try{
  let res3 = await p;
}catch(e) {
  console.log(e)
}
```



### Nodejs模块化

我们可以把公共的功能抽取成一个单独的 js 文件作为一个模块，默认情况下 这个模块里面的方法或者属性，外面是没法访问的。

如果要让外部可以访问模块里面的方法或者属性，就必须在模块里面通过exports 或者 module,exports 暴露属性或者方法。

在要使用这些模块的文件中，通过require 的方式引入这个模块，这个时候就可以使用这个模块里面暴露的属性和方法

```javascript
var obj = {
  get:function(){
    
  },
  post:function(){
    
  }
}
exports.xxxx= obj;
module.exports.xxx = obj;
var request = require('./module/request');
npm init --yes
```

```bash
npm list
# 指定包的版本
npm install node-media-server@2.1.0 --save
npm install jquery@1.8.0

```

### Package.json

package.json 定义了这个项目所需要的各种模块，以及的配置信息(比如名称、版本、许可证等元数据)

1 创建 package.json

```bash
npm uninstall ModuleName
# npm list 查看当前目录下已安装的node 包
npm list
npm info jquery 查看jquery 的版本
npm init
npm init --yes
```

2. Package.json 文件

```bash
{
	"name":"test",
	"version":"1.0.0",
	"description":"test",
	"main":"main.js",
	"keywords":[
		"test"
	],
	"author":"wade",
	
}
```

^ 表示第一位版本号不变，后面两位取最新的

~ 表示前两位不变，最后一个取最新

* 表示全部取最新

### 文件操作

```bash
1. fs.stat 检查是文件还是目录
2. fs.mkdir 创建目录
3. fs.writeFile 创建写入文件
4. fs.appendFile 追加文件
5. fs.readFile 读取文件
6. fs.readdir 读取目录
7. fs.rename 重命名
8. fs.rmdir 删除目录
9. fs.unlink 删除文件
```

```javascript
fs.mkdir('./css',(err) =>{
  if(err){
    console.log(err);
    return;
  }
  console.log('创建成功'); 
})
```

```javascript
const fs = require('fs');
var path = './upload';

fs.stat(path,(err,data) => {
  if(err){
    mkdir(path);
    return;
  }
  if(data.isDirectory()){
    console.log('upload目录存在')
  }else {
    mkdir(path)
  }
})
```

1. 方法的简写

```javascript
var app = {
  name,
  method(){
    
  }
}
```

2. 使用Promise 来处理异步 resolve 成功的回调函数 reject 失败的回调函数

```javascript
var http = require('http')
var server = http.createServer()
server.on('request',function(request,response){
  console.log('start')
})
server.listen(3000,function(){
  console.log('launch')
})
```

在Node中没有全局作用域，只有模块作用域



















