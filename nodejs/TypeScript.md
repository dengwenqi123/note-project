## TypeScript

TypeScript 是微软2012年推出的一种编程语言，属于JavaScript 的超级，可以编译为JavaScript 执行。它的最大特点就是支持`强类型`和`ES6 Class`。

首先，安装TypeScript:

```bash
npm install -g typescript
```

然后，为变量指定类型。

```javascript
//greet.ts
function greet(person: string) {
  console.log("Hello, "+ person);
}
greet([0,1,2])
```

如上文件为greet.ts 的代码，后缀名 ts 表明这是 TypeScript 的代码。函数greet 的参数，声明类型为字符串，但在调用时，传入一个数组。

### 基本概念

如下是一段 TypeScript 示例代码 helo.ts:

```bash
function sayHello(person: string) {
	return "Hello, " + person;
}
let user = 'Tom';
console.log(sayHello(user));
```

执行

```bash
tsc hello.ts
```

这样就会生成一个编译好的文件 hello.js

```bash
function sayHello(person) {
	return 'Hello, '+ person;
}
var user = 'Tom';
console.log(sayHello(user));
```

#### 原始数据类型

JavaScript 的类型分为两种： 原始数据类型 和 对象类型。

原始数据类型包括: 布尔值、数值、字符串、null、undefined 以及 ES6 中的新类型Symbol 和BigInt。

```typescript
let isDone: boolean = false;
// 使用构造函数Boolean 创造的对象不是布尔值。
let createdByNewBoolean : boolean = new Boolean(1);
// 字符串：使用string 定义字符串类型
let myName: string = 'Tom';
let myAge : number = 25;
//模版字符串
let sentence : string = `Hello, my name is ${myName}`
```

##### 空值

JavaScript 没有空值(Void)的概念，在TypeScript中，可以用void 表示没有任何返回值的函数：

```typescript
function alertName(): void {
  alert('My name is Tom');
}
```

声明一个 void 类型的变量没有什么用，因为你只能将它赋值为 undefined 和null:

```typescript
let unusable : void = undefined;
```











































