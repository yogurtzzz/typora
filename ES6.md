# ES6

## let 和 const

### let不存在变量提升

一个最经典的问题

```javascript
var a = [];
for(var i = 0; i < 10; i++){
    a[i] = function(){
        console.log(i);
    };
}

a[2]();   //10 
console.log(i);   //10

//因为var的变量提升
```



使用let来写for循环

```javascript
var a = [];
for(let i = 0; i < 10; i++){
    a[i] = function(){
        console.log(i);
    };
}

a[2]();   //2
console.log(i);  //ReferenceError  i is not defined
//let 不存在变量提升
```

for循环有一个特别之处，设置循环变量的那部分是一个**父级作用域**，而循环体内部是一个单独的**子作用域**，具体看如下代码：

```javascript
for(let i = 0; i < 10; i++){
    let i = "haha";
    console.log(i); 
}
//输出10次   haha
//在循环体执行结束后，循环体内的i被销毁，回到循环变量那里时，用的是父级作用域的i
```

但是如果

```javascript
for(let i = 0; i < 10; i++){
    i = "haha";
    console.log(i);
}
//只输出1次 haha
//因为子作用域里改变了父作用域里的i，第二次判断时，判断 "haha" < 10 的结果是false

//另外，记得JS中函数参数如果是基本类型，则是按值传递的
```



### 暂时性死区

如果在块级作用域内存在`let`，它所声明的变量就绑定该作用域，不再受外部影响，如

```javascript
var a = 123;
if(true){
    a = "haha"; //会报错，ReferenError a is not defined
    console.log(a);
    let a;
}
```

```javascript
//然而如果
var a = 123;
if(true){
    a = "haha";
    console.log(a);  //haha
}
```

ES6中规定，如果区块中存在`let`或`const` 命令，这个区块对这些命令所声明的变量，从一开始就形成了封闭作用域，凡是在声明之前就使用这些变量，就会报错。

在代码块中，如果使用了`let`，那么在`let`声明变量之前，该变量都是不可用的，这被称为**暂时性死区**(Temporal dead zone) TDZ

```javascript
if(true){
    //TDZ开始
    a = "abc"; //ReferenceError
    console.log(a);
    
    let a;  //TDZ结束
    console.log(a); //undefined
    a = "abc";
    console.log(a);  //abc
}
```



**暂时性死区**意味着`typeof`不再是一个100%安全的操作

不用`let`的话

```javascript
console.log(typeof a);  //undefined
```

使用`let`的话

```javascript
console.log(typeof a); //ReferenceError
let a;
```



在用`let`声明变量`a`之前，都属于`a`的"死区"，只要用到该变量，都会报错。



有一些**死区**比较隐蔽，不太容易发现

```javascript
function foo(x = y,y = 2){
    return [x.y];
}
foo(); //ReferenceError y is not defined
//x的默认值是y，而此时y还没被声明，属于y的死区
foo(1); //若给x一个值，就不会报错

var x = x; //不报错   x为undefined
let x = x; //报错    ReferenceError
```



### let不允许重复声明

在**相同作用域内**，`let`不允许声明同一个变量

```javascript
function fo(){
    //报错
    let a = 10;
    var a = 1;
}

function fo1(){
    //报错
    let a = 2;
    let a = 1;
}
function fo2(arg){
    let arg;
}
fo2(2); //报错

function fo3(arg){
    {
        let arg;
    }
}
fo3(3); //不报错
```



### 块级作用域

ES5只有全局作用域和函数作用域，没有块级作用域，这会导致一些问题。在ES5中，常用**立即执行函数（IIFE）**来模拟块级作用域

1. 内层变量可能会覆盖外层变量

   ```javascript
   var now = new Date();
   function f(){
       console.log(now);
       if(false){
           var now = "hello";
       }
       console.log(now);
   }
   f();
   //结果是
   //undefined
   //undefined
   
   //经过提升后的代码实际如下：（注意会发生变量提升和函数提升）
   var now;
   function f(){
       var now;
       console.log(now);
       if(false){
           now = "hello";
       }
       console.log(now);
   }
   now = new Date();
   f();
   ```

2. 用于计数的变量泄露为全局变量

   ```javascript
   var num = 0;
   for(var i = 0; i < 50; i++){
       num += i;
   }
   console.log(i);  //50
   
   
   ```

   

### ES6的块级作用域

`let`实际为JavaScript增加了块级作用域

```javascript
function f(){
    let n = 1;
    if(true){
        let n = 5;
        console.log(n); //5
    }
    console.log(n); //1
}
f();

//然而如果
function f(){
    let n = 1;
    if(true){
        n = 5;
        console.log(n); //5
    }
    console.log(n); // 5
}
f();
```



### 块级作用域与函数声明

ES5规定，函数只能在**顶层作用域**和**函数作用域**中声明，不能在块级作用域中声明。然而浏览器并没有遵守这个规定。

ES6中，引入了块级作用域，明确允许在块级作用域之中声明函数，在块级作用域中声明函数类似与`let`，在作用域之外不可引用。然而为了兼容一些老代码，ES6在*附录*中规定，浏览器的实现可以不遵守上述规定，可以有自己的行为方式。

* 允许在块级作用域内声明函数
* 函数声明类似于`var`，会提升到**全局作用域**或**函数作用域**的头部
* 函数声明也会提升到所在**块级作用域**的头部

注意，上述3条规则，支队ES6的浏览器实现有效，其他环境，还是将块级作用域的函数声明当作`let`处理



考虑到环境的差异，应尽量避免在**块级作用域**内声明函数，若确实需要，则应写成函数表达式，而不是函数声明。

```javascript
{
    let f = function(){
        console.log("hello");
    };
}
```



另，ES6的块级作用域允许神功函数的规则，只在使用了**大括号**{} 的时候成立，若无大括号，则报错

```javascript
'use strict';
if(true){
    function f(){};
}
//不报错


'use strict'
if(true)
    function f(){};

//报错
```





### const命令

`const`声明一个只读变量，一旦声明，变量值不可改变。

`const`一旦声明变量，就必须立即初始化，不能留到以后再赋值

```javascript
const PI = 3.1415;
```

`const`与`let`一样，**不存在变量提升**，**存在暂时性死区**，**不允许重复声明**



`const`实际上保证的，是**变量指向的那个内存地址所保存的数据不得改动**，而不是变量的值不可改动。



对简单类型，如boolean，number，string，值就保存在变量指向的那块内存区域。

对复合类型数据，如**对象**和**数组**，变量指向的内存地址内，保存的是一个指针（指向实际数据）。而`const`只能保证这个指针的值是不变的（始终指向同一块内存区域），至于那块内存区域里的值，就无法控制了。

```javascript
const obj = {};
obj.haha = 123;  //为obj添加属性haha，可以成功
```

因此，将一个对象声明为`const`需要格外小心



e.g.2

```javascript
const a = [];
a.push('Hello');  //可以成功执行
a.length = 0;    //可以成功执行
a = ['Tomcat'];   //报错
//这个数组本身是可写的，但是给他赋值，就会报错
```



可以对对象进行冻结，冻结后，对对象添加新属性将不起作用，也无法修改已有属性

```javascript
const foo = Object.freeze({});

//正常模式，下一行将不起作用
//严格模式，下一行将报错
foo.haha = "abc";
```



### ES6声明变量的6种方法

ES5只有`var`和`function`两种声明变量的方法

ES6中添加了`let`，`const`，`import`，`class`



### 顶层对象

在浏览器中，指的是`window`对象，Node中指的是`global`对象，ES5中。**顶层对象的属性**和**全局变量**是等价的

```javascript
window.a = 1;
console.log(a); //1
```

**顶层对象的属性和全局变量挂钩，被认为是JavaScript语言最大的设计败笔之一**



ES6为了改变这一点，规定：一方面，为了保持兼容性，`var`和`function`命令声明的全局变量，仍然是顶层对象的属性；另一方面，`let`，`const`，`class`命令声明的全局变量，不属于顶层对象的属性。

```javascript
function f(){
  console.log("this is function f");  
};

window.f();   //this is function f 


let a = function(){
    console.log("this is function a");
};
window.a();  //报错  window.a不是一个function
```



ES5中的**顶层对象**，本身也是一个问题

* 浏览器中，顶层对象是`window`，`self`
* Web Worker中，顶层对象是`self`
* Node中，顶层对象是`global`





## 变量的解构赋值

### 数组的解构赋值

ES6允许按照一定模式，从数组和对象中提取值，对变量进行赋值，这被称为**解构**（Destructing）

```javascript
let [a,b,c] = [1,2,3];
```

本质上，上面的写法属于"**模式匹配**"，只要等号两边的模式相同，左边的变量就会被赋予对应的值，如

```javascript
let [wire,[shark],[tom,cat]] = [1,[2],[3,4]];

let [, , hei] = ["abc","play","lol"]; 
console.log(hei); //lol

let[hear,...tail] = [1,2,3,4,5];
console.log(tail); //[2,3,4,5]
```



若结构不成功，变量的值就是`undefined`



只要某种数据结构具有`Iterator`接口，都可以采用数组形式进行解构赋值

```javascript
let mySet = new Set(['a','b','c']);
let [x,y,z] = mySet;
```



#### 默认值

解构赋值允许默认值，ES6内部采用严格相等符`===` 来判断一个位置是否有值，只有当一个数组成员严格等于`undefined`，默认值才会生效

```javascript
let[x = 1] = [undefined];
console.log(x); //1

let[y = 1] = [null];
console.log(y); //null
```



```javascript
function f(){
    console.log("Hello");
    return 3;
}
let [x = f()] = [1];
//因为解构成功，所以x=1，并且f函数不会执行

//若
let [x = f()] = [];
//解构失败，则使用默认值，此时才会调用f函数
```



```javascript
let [x = 1, y = x] = [];     // x=1; y=1
let [x = 1, y = x] = [2];    // x=2; y=2
let [x = 1, y = x] = [1, 2]; // x=1; y=2
let [x = y, y = 1] = [];     // ReferenceError: y is not defined
```



### 对象的解构赋值



```javascript
let {foo,boo,too} = {foo:"tom",boo:"cat",too:"hello"};
//变量名必须与属性名相同，才能取到值


let {fuck,ie} = {ie:"xixi",fuck:"haha"}; //并且次序无关紧要
fuck; //haha

let {bar} = {car:"abc",bar:"123"};
bar; //123
```



若变量名和属性名不一致，必须写成这样：

```javascript
let {first:f,second:s} = {first:"tomcat",second:"wireshark"};
f; //tomcat
s; //wireshark
//保持模式匹配即可
```



**注意**，对象结构错误的写法

```javascript
let x;
{x} = {x:"abc"}; //Syntax error
//原因：JavaScript引擎会将 {x}理解为一个代码块，从而发生语法错误，只有不将大括号写在行首，避免JavaScript引擎将其解释为代码块，才能解决这个问题
```

正确写法：

```javascript
//将整个解构赋值语句，放在一个圆括号里，即可
let x;
({x} = {x:"abc"});
```



### 字符串的解构赋值

字符串也能结构赋值，因为此时的字符串被转化成了类似数组的对象

```javascript
const [a,b,c,d,e] = "hello";
a  // h
b  //e
c  //l
d  //l
e  //o
```

类数组的对象都有一个`length`属性，因此还可以用该属性进行解构赋值

```javascript
let {length:len} = "hello";
len   // 5
```

### 数值和布尔值的解构赋值

解构赋值时，若等号右边是**数值**或**布尔值**，则会先转为对象

```javascript
let {toString: s} = 123;
s === Number.prototype.toString ;  //true
```



**总结** ： 解构赋值的规则是，只要等号右边的值不是**对象**或**数组**，就先将其转为对象，而`undefined` 和 `null` 无法转为对象，所以无法对它们进行解构赋值。



### 函数参数的解构赋值

```javascript
function add([x,y]){
    return x + y;
}

add([1,2]);  //3

[[1,2],[3,4]].map(([x,y]) => x + y);
// [3,7]
```

函数参数的解构也能使用默认值

```javascript
function reset({ x = 0, y = 0}){
    return [x,y];
}

move({1,4}); //[1,4]
move({3});  //[3,0]
move({});   //[0,0]
```

