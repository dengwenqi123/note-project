## Promise

Promise 是一个构造函数，自己身上有all、reject、resolve这几个眼熟的方法，原型上有then、catch 等同样很眼熟的方法。

```javascript
var p = new Promise(function(resolve,reject){
  	setTimeout(function(){
      console.log('执行完成');
      console.log('随便什么数据')
    },2000);
});
```

Promise的构造函数接收一个参数，是函数，并且传入两个参数：resolve,reject,分别表示异步操作执行成功后的回调函数和异步操作执行失败后的回调函数。其实这里用“成功”和“失败”来描述并不准确，按照标准来讲，resolve 是将Promise的状态置为fullfiled,reject 是将Promise 的状态置为rejected。

```javascript
function runAsync() {
  var p = new Promise(function(resolve,reject){
    //做一些异步操作
    setTimeout(function(){
	      console.log('执行完成');
        resolve('随便什么数据');
    },2000)
  });
  return p;
}
runAsync();
```

在runAsync() 的返回上直接调用then方法，then接收一个参数，是函数，并且会拿我们在runAsync中调用resolve时传的参数。

### 链式操作的用法

在表面上看，Promise 只是能够简化层层回调的写法，而实质上，Promise的精髓是"状态",用维护状态、传递状态的方式来使得回调函数能够及时调用，它比传递callback函数要简单、灵活的多。所以使用Promise的正确场景是这样的：

```javascript
runAsync1().
then(function(data){
  console.log(data);
  return runAsync2();
})
.then(function(data){
  console.log(data);
    return runAsync3();
})
.then(function(data){
    console.log(data);
});
```

这样能够按顺序，每隔两秒输出每个异步回调中的内容，在runAsync2中传给resolve的数据，能在接下来的then方法中拿到。

#### catch的用法

用来指定reject的回调，用法是这样：

```javascript
getNumber()
.then(function(data){
  console.log(data);
})
.catch(function(reason){
  console.log(reason);
});
```



































