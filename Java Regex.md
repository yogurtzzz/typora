# Java 正则

JDK内置的正则表达式引擎：`java.util.regex`

用字符串来描述规则，用于匹配字符串的一种机制



最简单的代码示例

```java
String input = "1926";
String regex = "19\\d{2}";
System.out.println(input.matches(regex)); //true

//注意String 的matches方法，参数中的reg字符串，必须和input完全匹配
//不能用matches()来判断输入字符串中，是否包含符合正则规则的子串
String input2 = "aa1926bb";
System.out.println(input2.matches(regex)); //false

//String里的matches()方法，实际也是利用了Pattern和Matcher来实现的
//如果要多次使用一个正则表达式来匹配字符串
//直接调用String里的matches()方法效率较低，因为每次都会生成一个Pattern和Matcher
//所以，此时可以先将正则字符串编译成Pattern，在用Matcher的matches方法去匹配
```



若需要在一个很长的字符串中寻找所有满足匹配规则的子串，代码实例如下

```java
String regx = "[1][3,4,5,7,8]\d{9}";
Pattern p = Pattern.compile(regx);
String input = "alonglonglongstring15626481661###13512341564..32";
Matcher m = p.matcher(input);
while(m.find()){
    System.out.println(m.group()); 
    //group()默认调用group(0)
}

//利用Matcher的group(n)可以快速提取子串，分组匹配
String regzz = "([a-zA-Z]{3})(\\d{5})";
String inputzz = "Leo456789###Ann123456##1";
Pattern p2 = Pattern.compile(regzz);
Matcher m2 = p2.matcher(intputzz);
while(m2.find()){
    String whole = m2.group(0); //0表示匹配整个字符串     Leo456789
    String g1 = m2.group(1);    //1表示匹配的第1个子串    Leo
    String g2 = m2.group(2);    //2表示皮皮诶的第2个子串  456789
}

//利用正则表达式来分割字符串
String input = "hello,apple;banana	world yogurt";
String reg = "[\\s,;]+";
String[] strs = input.split(reg);
for(String item : strs){
    System.out.println(item);
    //hello
    //apple
    //banana
    //world
    //yogurt
}

//利用正则表达式来进行查找
String str = "The shark is one of the most dangerous animals in the world.";
Pattern pattern = Pattern.compile("the",Pattern.CASE_INSENSITIVE);
Matcher matcher = pattern.matcher(str);
while (matcher.find()){
	String substr = str.substring(matcher.start(),matcher.end());
	System.out.println(substr);
    //The
    //the
    //the
}

//利用正则来做替换
//如，去除多余的空格
String str = "Hello world    this         is    yogurt.";
String simple = str.replaceAll("\\s+"," ");
System.out.println(simple);  //Hello world this is yogurt.

//正则后向引用
String str = "Hello world";
String res = str.replaceAll("(\\w+)","#$1#");  //$1 表示向前匹配第一个捕获组
System.out.println(res);  //#Hello# #world#
```



# 正则基础

## 元字符

- `\b` ：代表单词的开头或结尾，即单词的分界处，单词通常由`空格`，`标点符号`，`换行符` 来分隔，但`\b`不匹配这些分隔符，它**只匹配一个位置**

- `\B` ： 匹配非单词的边界位置

- `^` ：匹配字符串开始的位置

- `$` ：匹配字符串结束的位置

- `\d` ：匹配一个数字，相当于`[0-9]`

- `\D` ：匹配一个非数字，相当于`[^0-9]`

- `\w` ：匹配字母、数字、下划线，相当于`[A-Za-z0-9_]`

- `\W` ：与`\w`相反，相当于`[^A-Za-z0-9_]`

- `\s` ：匹配任一空白符，如空格符，制表符，换行符，相当于`[\f\n\r\t\v ]` (*注意* ：末尾还有一个空格)

- `\S` ：匹配任一非空白符，相当于`[^\f\n\r\t\v ]`

- `\n` ：表示"新行"

- `\r` ：表示"回车"

- `.` ：匹配除换行符(`\n`，`\r`)以外的任一单个字符

  举例：

- `[012]`：匹配一个数字，数字可以是0，1，或2

- `[abdf]` ：匹配一个字母，字母可以是a，b，d，或f

- `[.?!*]` ：匹配 . ? ! * 四个字符中的任一一个标点（在[ ]内不需要转义）

## 限定符

- `*` ：匹配前面的子表达式0次或多次，相当于`{0,}`
- `+` ：匹配前面的子表达式1次货多次，相当于`{1,}`
- `?` ：匹配前面的子表达式0次或1次，相当于`{0,1}`
- `{n}` ：n是一个非负整数，匹配前面的子表达式n次
- `{n,m} ` ：m≥n≥0，最少匹配n次，最多匹配m次
- `{n,}` ：n≥0，最少匹配n次（*注意* ：没有`{,m}`的写法，请写成`{0,m}`）



Examples:

如下的正则表达式：

`\(?0\d{2}[)-]?\d{8}`

> *分析*：首先一个转义字符 `\(`表示匹配`(` ，后面`?`表示0次或1次，然后是2个数字，接着是`[)-]?` 表示匹配`)`或`-` 字符 0次或1次，最后是8个数字

它能够匹配诸如 `(010)88886666` 或 `022-22334455` 或 `01212345678`等字符串，然而，它也能匹配到诸如 `(0108888666`  `(010-8888666`  `010)8888666` 等**不符合预期规则**的字符串，要解决此问题，需要用到**分支条件**



## 分支条件

指的是有几种规则，满足其中任意一种规则，都能完成匹配，使用`|`把不同的规则分开，例子如下：

**例1**

`0\d{2}-\d{8}|0\d{3}-\d{7}`

能匹配`010-12345678` 或 `0376-1234567` 这两种类型的电话号码

注意`|`表示完全分开两种规则，而不是就近分隔某一种，就近分隔需要用到`()

**例2**

`\(0\d{2}\)[- ]?\d{8}|0\d{2}[- ]?\d{8}`

匹配3位区号，其中区号可以用小括号括起来，也可以不用，区号和本地号之间，可以用连字符或空格间隔，也可以没间隔



**注意**：分支条件会从左往右的测试每个条件，一旦满足某个条件，就不会继续管其他条件了（短路运算），如：

`\d{5}-\d{4}|\d{5}` 这个表达式用于匹配美国邮编，美国邮编要么是5位数字，要么是连字符间隔的9位数字，如果将正则写成如下

`\d{5}|\d{5}-\d{4}` 则只能匹配5位的邮编，或者9位邮编的前5位

**所以要特别注意分支条件的放置顺序**



## 分组

我们已经能实现重复单个字符的匹配，如`\d{8}`匹配8个数字，如果想要重复多个字符，可以使用分组，例子如下：

`(\d{1,3}.){3}\d{1-3}` 简单的IP地址匹配表达式，重复匹配子表达式 `\d{1,3}.` 3次，但是它也能匹配到诸如`255.888.999.777`这样的无效IP地址，正确的IP地址匹配表达式如下（不唯一）：

`^((25[0-5]|2[0-4]\d|[1]\d{2}|\d{1,2})\.){3}(25[0-5]|2[0-4]\d|[1]\d{2}|\d{1,2})$`

注意分支条件的放置顺序



## 反义

查找除某个能简单定义的字符之外的字符

常用的反义字符(前面几个是大写字母)：

`\W` : 匹配任意不是字母，数字，下划线的字符  

`\S` ：匹配任意不是空白符的字符

`\D` ：匹配任意非数字的字符

`\B` ： 匹配不是单词开头和结尾的位置

`[^x]` ： 匹配除x以外的任意字符

`[^aeiou]` ：匹配除a,e,i,o,u以外的任意字符



## 后向引用

使用小括号指定一个子表达式后，**匹配这个子表达式的内容**，此分组捕获到的内容，**默认有一个组号（从左到右，捕获到的分组组号依次为1,2,3...）**

**后向引用**可以重复搜索前面某个分组已捕获到的文本，如`\1`表示分组1捕获到的文本，如`\b(\w+)\b\s+\1\b` ，可以匹配到如`hello  hello`

分组的名字也可以自己指定，使用`(?<myWord>\w+)`这样的形式来指定分组名称，其中的**尖括号**也可以用**单引号**来代替，如`(?'myWord'\w+)`，在之后若要引用该分组，使用`\k<myWord>`，或`\k'myWord'`所以要捕获并使用这个分组，如下

`\b(?<myWord>\w+)\b\s+\k<myWord>\b`

**注意：**

JAVA中后向引用分为：

- 在正则表达式中，使用`\\1`，`\\2`来进行后向引用
- 在正则表达式外，使用`$1`，`$2`来进行后向引用

```java
//代码示例
//正则表达式中使用后向引用
String regx1 = "(hello)[\\s\\w,]+\\1";
String input1 = "hello this is yogurt ,hello";
boolean f = input1.matches(regx1); //true

//正则表达式外使用后向引用
String regx2 = "([a-zA-Z]+)";
String input2 = "Wireshark Tomcat Zookeeper are funny names.";
String after = input2.replaceAll(regx2,"#$1#");
System.out.println(after);
//#Wireshark# #Tomcat# #Zookeeper# #are# #funny# #names#.
```

**分组语法**

| 代码           | 描述                                  |
| -------------- | ------------------------------------- |
| `(exp)`        | 匹配exp，并捕获文本到自动命名的组里   |
| `(?<name>exp)` | 匹配exp，并捕获文本到名称为name的组里 |
| `(?:exp)`      | 匹配exp，但不捕获匹配到的文本         |



## 零宽断言

用于查找在某些内容（但并不包括这些内容）之前或之后的东西，也就是说像`\b`，`^`，`$` 那样用于指定一个位置，这个位置应该满足一定的条件。

- **零宽正向断言**（肯定断言）

  - **零宽正向先行断言**   `(?=exp)`

    断言此位置的后面**能匹配**表达式exp，比如`\b\w+(?=ing\b)`能匹配以ing结尾的单词前面的部分

  - **零宽正向回顾断言**  `(?<=exp)`

    断言此位置的前面**能匹配**表达式exp，比如`(?<=\bpre)\w+\b`能匹配以pre开头的单词后面的部分

- **零宽负向断言**（否定断言）

  - **零宽负向先行断言**  `(?!exp)`

    断言此位置的后面**不能匹配**表达式exp，比如`\d{3}(?!\d)`匹配3个数字，并且这3个数字后面不能是数字

  - **零宽负向回顾断言**  `(?<!exp)`

    断言此位置的前面**不能匹配**表达式exp，比如`(?<![a-z])\d`匹配1个数字，该数字前不能是小写字母



综合例子：

`(?<=<(\w{3})>).*(?=<\/\1>)` 能够匹配字母数为3的html标签，如`<div></div>`,`<img></img>`等



***注意*** ：零宽断言**不允许**含有**不定长**的表达式，如`(?<=<(\w+)>).*(?=<\/\1>)`



## 贪婪与懒惰

当正则表达式包含能接受重复的限定符（如`*`，`+`，`{n,}` ），通常的行为时（在使整个表达式得到匹配的前提下）匹配**尽可能多**的字符，例如若用表达式`a.*b`来匹配`aababb`，它将匹配整个字符串`aababb`，这被称为**贪婪**匹配。



然而有时我们需要匹配**尽可能少**的字符串，即需要**懒惰**匹配。此时在限定符后面加上一个问号`?` 即可转换为**懒惰**匹配。如用`a.*?b`去匹配`aababb`，则会匹配`aab`（1-3位的字符）和`ab`（4-5位）

为什么第一个匹配到的是`aab`（1-3位），而不是`ab`（2-3位）呢？因为正则表达式有另一条规则，**最先开始的匹配拥有最高的优先权**



| 代码   | 说明                            |
| ------ | ------------------------------- |
| *?     | 重复任意次，但尽可能少重复      |
| +?     | 重复1次或更多次，但尽可能少重复 |
| ??     | 重复0次或1次，但尽可能少重复    |
| {n,m}? | 重复n到m次，但尽可能少重复      |
| {n,}?  | 重复n此以上，但尽可能少重复     |

