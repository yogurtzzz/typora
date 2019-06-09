# Java基础类库学习

## 1. Collection  

注意区分Collection和Collections  [参考链接](https://www.cnblogs.com/cathyqq/p/5279859.html)

[Collection参考链接](https://www.cnblogs.com/taiwan/p/6954135.html)

[参考链接2](http://www.importnew.com/23715.html)

来源于java.util包

类图：

![1555985652206](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1555985652206.png)



### HashMap和HashTable



几乎完全一样，它们的区别在于

1.HashTable是synchronized的，即是线程安全的，而HashMap不是

2.HashTable不接受null的键和null的值，而HashMap可以

3.由于HashTable是线程安全，所以HashTable的性能不如HashMap

4.HashTable的iterator迭代器不是fail-fast的，而HashMap的iterator是fail-fast的

5.HashMap不能保证随着时间推移，Map中的元素次序保持不变。若要保持元素顺序不变，应该用LinkedHashMap



**JAVA 5+** 实现了线程安全的HashMap - >   `ConcurrentHashMap`

若要自己实现HashMap的同步，可以调用Collections类的synchronizedMap方法

```java
HashMap<String,String> hashMap = new HashMap<String,String>();
Map map = Collections.synchronizedMap(hashMap);
```

（fail-fast:我们知道HashMap不是线程安全的，因此如果在使用iterator的过程中，有其他线程修改了HashMap的结构(新增或删除一个元素)，那么将抛出`ConcurrentModificationException`，这就是所谓的fail-fast策略，除非是iterator本身的remove方法，其他任何方式的修改，iterator都将抛出`ConcurrentModificationException`）

[fail-fast参考链接](https://www.cnblogs.com/think-in-java/p/5170914.html)





## 并发编程与多线程

[参考链接](https://www.cnblogs.com/dolphin0520/p/3920373.html)

### 并发基本问题

要想并发程序正确地执行，必须要保证**原子性**，**可见性**，以及**有序性**



* 原子性

  即一个操作或多个操作，要么全部执行成功，且执行的过程不被打断，要么就不执行。

  经典的例子是转账问题：从A账户中扣100元，再在B账户中加100元。两个操作要么全部执行成功，要么不执行，这就是操作的原子性

* 可见性

  即当多个线程访问同一变量时，若一个线程修改了该变量的值，其他的线程能够立即看到修改后的值，如

  ```java
  //下面是线程1执行的代码
  int i = 0;
  i = 10;
  //下面是线程2执行的代码
  j = i;
  ```

  线程一执行时，会先将0写入到CPU缓存中，接着赋值为10，最后再写入主存。

  若在线程一将10赋值给i后，写入到主存之前，线程二执行 `j = i;`，它会先去主存读取i的值，并加载到它的工作缓存中，此时主存中的i是0，那么就会使j为0。这就产生了**可见性问题**：线程一对i做了修改后，线程二并没有立即看到修改后的i值

* 有序性

  即程序执行的顺序按照代码的先后顺序，如下

  ```java
  int i = 0;    //语句1
  int j = 1;    //语句2
  i++;
  j++;
  ```

  语句1和语句2，从代码顺序上看，1在2的前面，但在JVM真正执行这段代码时，并不能保证1就在2前面，因为可能会发生**指令重排**（Instruction Reorder）

  **指令重排**：处理器为了提高程序运行效率，可能会对输入代码进行优化，它不保证程序中的各个语句的执行顺序和代码中的顺序一致，但是会保证程序最终执行的结果和代码顺序执行的结果是一致的。上述代码中，语句1和2谁先执行对最终结果并没有影响，那么这种语句是可能发生**重排**的。虽然会进行**指令重排**，但是相互之间有**数据依赖**的指令会被考虑，若指令a会用到指令b的结果，那么处理器会保证b在a之前执行。**指令重排**对单线程的执行结果没有影响，但是在涉及到有**共享变量**的多线程中，则需要考虑**指令重排**的问题。

  指令重排可以分为**编译器重排**和**处理器重排**，编译器的重排才是真正对指令顺序做了调整，处理器的重排并不是刻意为之，而是由于CPU是流水线执行的，可能由于后面的指令命中cache而先于前面的指令完成，也可能前面的指令执行时间较长，而后面的指令执行较短而先完成。

  为什么会发生指令重排呢？-> [参考链接](https://blog.csdn.net/lindanpeng/article/details/72459493)



java语言本身对于**原子性**，**可见性**，**有序性**，都提供了哪些保证呢？

* 原子性：java中，对基本数据类型的变量的**读取**和**赋值**的操作都是原子性的，如下

  ```java
  x = 10;   //语句1
  y = x;	  //语句2
  x++;      //语句3
  x = x +1; //语句4
  ```

  上述代码中，只有语句1时原子性操作。

  语句2会先读取x的值，然后再赋值给y，包含了2个原子性操作。

  语句3和4先读取x的值，然后执行+1操作，最后写入x

* 可见性：java中，提供了volatile关键字来保证可见性

  当一个共享变量被volatile修饰时，它会保证修改的新值立即更新到主存中，当其他线程需要读取该变量时，会直接去主存中读取新值，另，通过`synchronized`和`Lock`也能保证可见性，因为它们能保证同一时刻只有一个线程获取锁，并且执行同步代码，释放锁之前会将变量的修改更新到主存中。

* 有序性：java内存模型中，允许**编译器**和**处理器**对指令进行重排，重排不会影响到单线程，却会影响到多线程并发的正确性。在Java中，可以通过volatile来保证可见性，通过synchronized和Lock来保证有序性

### volatile解析

java内存模型规定所有的变量都存在**主存**（内存）中，每个线程都有自己的**工作缓存**（类似于CPU高速缓存），线程对变量的所有操作都是在**工作缓存**中进行，而不能直接对**主存**进行操作，每个线程不能访问其他线程的工作缓存。如`i = 10;`这条语句，执行线程必须先在自己的**工作缓存**中对变量i进行赋值操作，然后再写入到主存里，而不是直接将数值10写入主存。

**volatile的两层语义：**

一旦一个共享变量（类的成员变量，类的静态成员变量）被volatile修饰后，它就具备了两层语义

* 保证了不同线程对这个变量进行操作时的**可见性**，即一个线程修改了该变量的值，这个新的值对其他线程来说是立即可见的
* 禁止进行指令重排

注意：volatile并不能保证**原子性**，它只能保证每次读取到的是最新的值，所以仅仅用volatile来实现并发程序的正确执行，是不够的。例子如下：

```java
public class TestVolatile {
    public volatile int index = 0;
    public void incre(){
        this.index++;
    }
}
public class Test{
	private static final int MAX_THREADS = 10;
    public static void main(String[] args) throws InterruptedException{
        final TestVolatile test = new TestVolatile();
        final CountDownLatch coutDown = new CountDownLatch(MAX_THREADS);
        for (int i = 0; i < MAX_THREADS; i++) {
            //启动10个线程
            new Thread(){
                @Override
                public void run(){
                    //每个线程对test执行1000遍incre()调用
                    try{
                        for (int j = 0; j < 1000; j++) {
                            test.incre();
                        }
                    }finally{
                        System.out.println("线程"+Thread.currentThread().getName()+"执行结束");
                        coutDown.countDown();
                    }
                }
            }.start();
        }
        //等待全部线程执行完毕
        coutDown.await();
        System.out.println("全部线程执行完毕，test.index="+test.index);
    }
}
```

每次的执行结果都会小于10000

```
.....
线程Thread-8执行结束
线程Thread-7执行结束
线程Thread-9执行结束
全部线程执行完毕，test.index=9853
```

这是因为自增操作并不是**原子性**操作



使用以下任意一种方式可实现上述程序的正确执行

* 使用`synchronized`修饰自增方法`incre`

  ```java
  public class TestVolatile {
      public volatile int index = 0;
      public synchronized void incre(){
          this.index++;
      }
  }
  ```

  执行结果如下

  ```
  ...
  线程Thread-5执行结束
  线程Thread-8执行结束
  线程Thread-9执行结束
  全部线程执行完毕，test.index=10000
  ```

* 使用Lock（在`incre`方法内部使用Lock）

  ```java
  public class TestVolatile {
      public volatile int index = 0;
      Lock lock = new ReentrantLock();
      public void incre(){
          lock.lock();
          try{
              this.index++;
          }finally{
              lock.unlock();
          }
      }
  }
  ```

* 使用`AtomicInteger`

  ```java
  public class TestVolatile {
      public AtomicInteger index = new AtomicInteger();
      public void incre(){
              this.index.getAndIncrement();
      }
  }
  ```

  jdk1.5的java.util.concurrent.atomic包下提供了一些原子操作类，对基本数据类型的自增，自减，加法，减法操作进行了封装，保证这些操作是原子性操作。atmoic包下的类是通过CAS（Compare and Swap）来实现原子性操作的



volatile可以禁止指令重排，一定程度上保证**有序性**

* 当程序执行到volatile变量的读写操作时，在其前面的操作肯定已经全部完成，且结果对后面的操作可见，在其后面的操作肯定都还没执行
* 在进行指令重排时，不能将在对volatile变量访问的语句之前的语句放在其后执行，也不能将volatile变量后的语句放到其漆面执行

```java
public int x;
public int y;
public volatile boolean flag;

x = 2;           //语句1
y = 3;           //语句2
flag = true;     //语句3
x = 4;           //语句4
y = -1;          //语句5
```

通俗来说，语句3的位置一定是不变的：在执行到语句3时，1和2一定是已经执行完成了，且1和2的执行结果对3，4，5是可见的，并且此时4，5一定还没有被执行。但是语句1和2的顺序，是不被保证的，语句4和5的顺序也一样不会被保证。



### volatile的原理和实现机制

下面的内容讨论一下volatile到底如何保证**可见性**和**禁止指令重排**

以下内容摘自《深入理解Java虚拟机》

> 观察加入volatile关键字和没有i骄傲如volatile关键字时所生成的汇编代码可以发现，加入了volatile关键字时，会多出一个lock前缀指令

lock前缀指令实际相当于一个**内存屏障**（-> [参考链接]()），内存屏障会提供3个功能

* 确保指令重排序时，不会把**屏障**后面的指令排到**屏障**前面，也不会把前面的指令排到后面
* 会强制将对缓存的修改立即写入主存
* 若是写操作，会导致其他CPU中对应的缓存行无效（其他线程若要访问对应的变量，需要重新去主存中取）





**内存屏障**<span id="MemoryBarrier"></span>

Memory Barrier，有时也叫做内存栅栏（Memory Fence），是一种CPU指令，用于控制特定条件下的**重排序**和**内存可见性问题**，Java编译器也uhi根据内存屏障的规则禁止重排序

内存屏障可以被分为以下几个类型：

* LoadLoad屏障

  对于语句`Load1; LoadLoad; Load2;` 在Load2以及后续读取操作之前，保证Load1要读取的数据被读取完毕

* StoreStore屏障

  对于语句`Store1; StoreStore; Store2; `在Store2以及后续的写入操作之前，保证Store1的写入操作对其他处理器可见

* LoadStore屏障

  对于语句`Load1; LoadStore; Store2; `在Store2以及后续的写入操作之前，保证Load1要读取的数据被读取完毕

* StoreLoad屏障

  对于语句`Store1; StoreLoad; Load2;` 在Load2以及后续的读取操作之前，保证Store1的写入操作对其他处理器可见。该屏障是4种屏障中开销最大的，在大多数处理器中，这个屏障是万能屏障，兼具其他3种屏障的功能



### volatile的使用场景

synchronized是为了防止多个线程同时执行一段代码，这样很影响程序的并发效率，而volatile在有些情况下的性能要优于synchronized。但是需要注意volatile无法替代synchronized，因为volatile无法保证操作的**原子性**，一般来讲，使用volatile需要具备以下2个条件（其实就是保证了原子性）

* 对变量的写操作不依赖于当前值
* 该变量没有包含在具有其他变量的不变式中  ???



下面列举几个volatile的使用场景

* 状态标记

  ```java
  //以下是2个共享变量
  volatile boolean initilized = false;
  Context context；
  
  //下面是线程1
  context = loadContext(); //加载上下文环境
  initilized = true;
  
  //下面是线程2
  while(!initilized){
      //上下文环境未就绪时，sleep
      sleep();
  }
  doSomethingWithContext(context);
  
  
  //volatile保证了，在改变状态标记时，之前的代码已经全部执行完毕
  ```

* double check(  [参考链接](https://www.iteye.com/topic/652440))

  ```java
  //以下是一个双重校验单例模型
  class SingleTon{
      private volatile static SingleTon instance = null;
      private SingleTon(){}
      public static SingleTon getInstance(){
          if(instance == null){
              synchronized(SingleTon.class){//1
                  if(instance == null)//2
                      instance = new SingleTon();//3
              }
          }
          return instance;
      }
  }
  ```

  



### 线程池相关

[参考链接](https://www.cnblogs.com/dolphin0520/p/3932921.html)



## IO框架

## 一些面试题



#### `int`和`Integer`的区别

> 首先int是java八大原始数据类型之一，Integer是Object类型，是其包装类型。int的初始值是0，Integer初始值为null。那么为什么会有Integer呢？
>
> 因为原始数据类型与java泛型不能配合使用，所以就引入了包装类，并在java5中引入了**自动装箱**和**自动拆箱**，实际是原始数据类型和包装类型之间的隐式转换。
>
> int和Integer有各自的优势
>
> 原始数据类型在内存中存的是值，找到其内存地址，就能取到值，而Object在内存中存的是引用（reference），找到其内存地址后，还要根据reference找到下一个内存空间，会产生更多的IO操作，故原始数据类型比Object的性能要好。
>
> 但是Object具备泛型的能力，更加抽象，更适合解决业务问题，业务代码开发的效率会比较高。为了弥补Object在运算性能上的不足，又涉及了缓存机制，在调用Integer.valueOf方法时，会利用缓存机制，对于在-128~127之间的int，会去常量池中取。
>
> JVM会自动维护八种基本类型的常量池，int常量池的范围是-128~127，所以在`Integer a = 127;`时，会自动装箱，变成`Integer a = Integer.valueOf(127);`，会去取到常量池中的值，若`Integer b = 128;`则变成了`Integer b = new Integer(128);`

一段测试代码如下：

```java
		Integer a = 127;  //会变成 a = Integer.valueOf(127); 会从常量池中取
        Integer b = 127;  
        
        Integer aa = new Integer(127);
        Integer bb = new Integer(127);
        
        Integer c = 128;
        Integer d = 128;
        System.out.println(a == b);   //true
        System.out.println(a == aa);  //false
        System.out.println(b == bb);  //false
        System.out.println(aa == bb); //false
          
        System.out.println(c == d);    //false
        
        System.out.println(127 == a);  //true  
        System.out.println(127 == b);  //true
        System.out.println(127 == aa); //true
        System.out.println(127 == bb);  //true
```



#### 抽象类和接口的区别   

[参考链接](https://www.cnblogs.com/dolphin0520/p/3811437.html)

> * 语法层面区别
>   * 抽象类可以提供成员方法的实现，而接口中的方法只能是public abstract
>   * 抽象类中的成员变量可以是各种类型，而接口中的成员变量只能是public static final
>   * 抽象类中可以含有静态代码块和静态方法，而接口中不可以
>   * 一个类只能继承一个抽象类，但可以实现多个接口
>
> * 设计层面的区别
>
>   * 抽象类是**对事物的抽象**，即对类的抽象；而接口，是**对行为的抽象**。抽象类是对整个类的整体进行抽象，包括属性，行为；但是接口只是对类的局部（行为）进行抽象。如，飞机和鸟是两种不同类型的事物，它们有一个共性，就是都会飞。设计时，可以将飞机设计成一个类Plane，鸟设计成一个类Bird，但是不能将 “飞” 这个特性设计为类，因为它仅仅只是一个行为特性，而不是对一类事物的整体描述。此时可以将 “ 飞”这个行为设计成一个接口Fly，包含方法fly() ，然后Plane和Bird根据自己的需求实现Fly接口。然后至于不同类型的飞机：如民用飞机，战斗机等可以直接继承Plane，不同的鸟如海鸥，麻雀，乌鸦，直接继承Bird即可。抽象类的继承是一种 “ 是不是” 的关系，如 麻雀 是一种 鸟 ；而接口的实现是一种 “ 有没有/会不会 ”的关系，如鸟会飞。
>
>   * 抽象类作为很多子类的父类，它是一种**模板式设计**；而接口是一种**行为规范**，是**辐射式设计**。举个例子，若使用模板A设计了PPT1和PPT2，那么PPT1和2的公共部分就是模板A，若要对公共部分进行改动，则只需要改模板A就行了，不用改PPT1和2，这是**模板式设计**； 而如在电梯里装了某种报警器，一旦要更换报警器，就必须对所有电梯进行更新，这是**辐射式设计**。
>
>     即，对于抽象类，如果要添加新的方法，可以直接在抽象类中添加具体的实现，子类可以不进行改动；而对于接口，如果接口进行了变更，所有实现这个接口的类都必须做相应的改动。
>
> 
>
> 再举个例子：门和报警器的例子，对于门，都有open和close两个动作，我们可以通过**抽象类**或者**接口**来定义这个抽象概念
>
> ```java
> //抽象类
> public abstract class Door{
>     public abstract void open();
>     public abstract void close();
> }
> 
> //接口
> public interface Door{
>     public abstract void open();
>     public abstract void close();
> }
> ```
>
> 若现在需要门具备报警功能（alarm），那么如何实现呢？有以下两种思路
>
> 1. 将open,close,alarm都放在抽象类里。但是这样一来，所有继承Door的子类都具备了报警功能，但有的门并不一定具备报警功能
> 2. 将open,close,alarm都放在接口里。但是只需要报警功能的类，也要实现open和close方法。
>
> 
>
> 总结：open和close属于**门**本身的固有属性，而alarm属于附加的行为。故，最佳的解决方法是，将“报警”设计成一个接口，Door设计成一个抽象类，这样，带有报警功能的门只需要继承Door并实现Alarm接口即可



==\=================\=\=\=\=\=\=\=\==\=\===\==\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=\=

为什么泛型不能使用基础类型，如

```java
HashMap<Integer,Integer> map;
//而不能是
HashMap<int,int> map;
```





可通过getClassloader().getResource("xxxxx.properties") 来读取classpath下的一些文件用Properties类来读取其中的信息







RESTful相关   [参考链接](http://www.ruanyifeng.com/blog/2018/10/restful-api-best-practices.html)



HTTP



RMI，RPC，远程调用。。。。