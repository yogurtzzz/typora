# JS知识点

## 变量提升

`var`声明的变量将提升到它所在的作用于顶端去执行

```javascript
function test(){
    console.log(a);
    var a = 100;
}
test();  //undefined
```

```javascript
a = 1;
var a;
console.log(a);  //1
```

```javascript
console.log(a);
var a = 100;
function foo(){
    console.log(a);
    var a = 200;
    console.log(a);
}
foo();
console.log(a);

//undefined
//undefined
//200
//100
```



## 函数提升

函数的声明有两种方式：**函数声明**  ， **函数表达式**

```javascript
//函数声明
function foo(){
    alert("Hello,World");
}
//函数表达式
var foo = function(){
    alert("Hello,World");
};

//实质只有函数声明会发生提升，会将整个函数声明的代码块提升到作用域顶部
//而函数表达式，则只会提升变量foo，给foo赋值是在运行时
```



函数提升优先级高于变量提升，如

```javascript
function f(){
	console.log(a);
	var a = 10;
	function a(){
		console.log("a is function");
	}
	console.log(a);
}
f();

//输出结果是：
//f a(){ console.log("a is function")}
//10

//上面的代码实质会转换为
function f(){
    function a(){
        console.log("a is function");
    }
    var a;
    console.log(a);
    a = 10;
    console.log(a);
}
f();

//实质，如果函数名和变量名相同，那么无论先声明函数还是先声明变量，实质最后都是函数

funciton a(){};
var a;
console.log(typeof a);

var a;
function a(){};
console.log(typeof a);

//结果都是function
//除非对a进行赋值，如a = 10; 此时a才变成number类型
```

尽量不要在块作用域内声明函数



## this指针

this绑定的是函数运行时的上下文环境...切记