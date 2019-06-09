关于scanner.nextInt之后再scanner.nextLine的问题

```java
Scanner sc = new Scanner(System.in);
int a = sc.nextInt();
String s = sc.nextLine(); //这一句会被跳过

//实际时在nextInt的时候输入了一个数字，然后回车，nextInt只会读取数字，所以scanner缓存流中留下了一个回车符，下一个nextLine就直接读取了
//故nextLine看上去好像没执行（没有等待用户输入），实际是执行了

// -->解决
int a = sc.nextInt();
sc.nextLine();
String s = sc.nextLine();
```

