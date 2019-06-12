# JAVA整理

访问修饰符：

* public : 对所有类可见
* private : 仅在本类中可见
* protected  :  同一个包下的所有类可见，子类可见
* 默认 :  即不添加任何修饰符 ，同一包下的所有类可见



`instanceof`和`Class<?> isInstance()`的区别

public boolean isInstance(Object obj)

`isInstance`  is the dynamic equivalent of the Java `instanceof ` operator.It returns `true` if the specified *Object argument* is non-null and can be cast to the reference type represented by this *Class object* without raising a *ClassCastException* . It returns `false`  otherwise.



## 断言(assertion)

断言assert是在jdk1.4引入的

jvm断言默认是关闭的（在jvm启动参数中添加  -ea: com.yogurt.AssertTest） 来启用指定类的断言

断言是一种调试方式，一般只在开发和测试阶段启用断言

assert 预期其后的语句为true，否则，将抛出AssertionError错误





## java注解

### 自定义注解

* 使用@interface定义注解

* 可定义多个参数和默认值，核心参数推荐使用value

* 必须通过一些元注解来修饰自定义注解，@Target指定注解的应用范围

* 设置@Retention设置注解的作用范围（一般设置为RUNTIME）

  ```java
  @Target({
      ElementType.TYPE,
      ElementType.FIELD,
      ElementType.METHOD
  })
  //定义该注解可以作用域class/interface上，类的成员变量上，类的成员方法上
  @Retention(RetentionPolicy.RUNTIME)
  //定义该注解的作用范围
  public @interface Report{
      int type() default 0;
      String level() default "debug";
      String value() default "";
  }
  ```

  元注解:

  * `@Target`  : 定义Annotation可以被应用于源码的哪些位置

    * 类或接口：ElementType.TYPE

    * 类的字段：ElementType.FIELD

    * 类的方法：ElementType.METHOD

    * 构造方法：ElementType.CONSTRUCTOR

    * 方法参数：ElementType.PARAMETER

      例子：

      ```java
      @Target(ElementType.TYPE)
      //可以传入单一的变量，也可以传入数组
      @Target({
          ElementType.TYPE,
          ElementType.FIELD
      })
      ```

      

  * `@Retention` ：定义Annotation的生命周期

    * 仅编译期：RetentionPolicy.SOURCE  （注解是给编译器看的，编译器编译过后直接丢弃该种注解，如@Override）

    * 仅class文件：RetentionPolicy.CLASS（注解默认的生命周期是CLASS）  (注解会存在于class文件中，但执行时会被VM丢弃)

    * 运行期：RetentionPolicy.RUNTIME （VM在运行期可以保留该注解，可通过反射来读取该注解的信息）

      通常自定义的注解一般的生命周期都是RUNTIME

  * `@Repeatable`：定义该注解是否允许重复（JDK>=1.8）

  * `@Inherited`：定义子类是否可以继承父类定义的Annotationn

    仅针对@Target为ElementType.TYPE，且仅支持class的继承，不支持interface的继承

  * `@Document`：将注解包含在Javadoc中

### 如何处理自定义注解

（只针对RUNTIME类型的注解）

Annotation也是一种类，可以通过反射API来读取Annotation

* Class.**isAnnotationPresent**(annotation.class)  判断某个annotation是否存在
* Field.isAnnotationPresent()
* Method.isAnnotationPresent()
* Constructor.isAnnotationPresent()



获取Annotation

Class | Field | Method .getAnnotation(anno.class)

```java
public class Test{
    @Report(value = "this is String",type = 0）
    private String a;
    
    @Report(value = "this is method",type = 1）
            public void say(){
                Syste.out.println("hello");
            }
            
            public static void main(String[] args){
                Class clazz = Test.class;
                Field[] fs = clazz.getDeclaredFields();
                Method[] ms = clazz.getDeclaredMethods();
                for(Field f:fs){
                    if(f.isAnnotationPresent(Report.class)){
                        Report r = f.getAnnotation(Report.class);
                        System.out.println(r.type()+","+r.value());
                    }
                }
            }
}
```





获取方法参数的annotation比较麻烦，如

```java
public void say(@Report(type = 2) String name, String age){
    //...
}
```

想要获取函数`say()`的参数的注解，如下

```java
Method m = ....;
//这里的m假设是上方的say函数
//因为一个函数的参数是多个，而每个参数又可以有多个注解修饰，所以获取到的是个二维数组
Annotation[][] annos = m.getParameterAnnotations();
Annotation[] annos_1 = annos[0];  //获取第一个参数的注解，上面相当于获取了参数name的注解
for(Annotation anno:annos_1){
    if(anno instanceof Report){
        Report r = (Report) anno;
        System.out.println(".....");
    }
}
```



注意，注解的生命周期一定要为Annotation设置`@Retention(RetentionPolicy.RUNTIME)`，否则，在运行时，无法通过反射对注解进行处理



## 泛型

如ArrayList\<T>

总结：

* 泛型就是编写模板代码来适应任意类型

* 我们不必对类型进行强制类型转换
* 编译器将对类型进行检查
* 泛型的继承关系需要注意
  * `ArrayList<Integer>`可以向上转型为`List<Integer>`(泛型的类型不能变)
  * 然而`ArrayList<Integer>`不能向上转型为`ArrayList<Number>`



给泛型类编写静态方法时，需要将方法声明为泛型方法

```java
public class Pair<T>{
    T first;
    T second;
    //..一些构造方法和set/get方法
    
    //下面这样做是无法通过编译的
    public static Pair<T> create(T t1,T t2){
        return new Pair<>(t1,t2);
    }
    //需要改为
    //其实下面的T和定义中的Pair<T>是不一样的
    public static<T> Pair<T> create(T t1,T t2){
        return new Pair<>(t1,t2);
    }
    //更好的做法是，以示区分
    //表示这个方法接受一个类型参数K，这个K与这个Pair类中的T是不一样的
    public static<K> Pair<K> create(K t1,K t2){
        return new Pair<>(t1,t2);
    }
}
```



类型擦除

java的泛型是通过**类型擦除**（Type Erasure）实现的

在泛型代码编译时，编译器将泛型`<T>`统一视为Object， JVM对泛型一无所知，所有的工作皆是编译器做的

```java
//例子
//下面是编写的代码
public static void main(String[] args){
    Pair<String> pair = new Pair<>("yogurt","bee");
    String first = pair.getFirst();
}

//而下面是编译器实际处理的代码
public static void main(String[] args){
    Pair pair = new Pair("yogurt","bee");
    String first = (String) pair.getFirst();
}

//编译器将<T>视为Object，在需要转型的时候，根据<T>的类型，会执行安全的类型转换
```



### 泛型的局限

由于JAVA的泛型是通过**类型擦除**实现的，所以具有一些局限性：

* `<T>`的类型不能是**基本数据类型**，如`int`

* 由于存在**类型擦除**，所以无法获取带有泛型的Class，也无法判断带泛型的Class

  ```java
  Pair<String> sp = new Pair<>("hello","Java");
  Pair<Integer> ip = new Pair<>(1,2);
  boolean is = sp.getClass() == ip.getClass(); //true
  //sp和ip的Class实际都是  Pair.class
  sp.getClass() == Pair.class  //true
  ```

* 无法实例化泛型`<T>`

  ```java
  public class Pair<T>{
      private T first;
      private T second;
      //...
      public Pair(){
          //下面2句代码，编译器会报错
          first = new T();
          second = new T();
      }
  }
  ```

* 实例化`<T>`必须借助`Class<T>`

  ```java
  public class Pair<T>{
      private T first;
      private T second;
      //...
      public Pair(Class<T> clazz) throws InstantiationException, IllegalAccessException{
          //这里的参数一定要是Class<T> clazz  ,而不能是Class clazz
          //否则，编译器会报错：Incompatible types,required T ,found Object
          first = clazz.newInstance();
          second = clazz.newInstance();
      }
  }
  
  //下面是示例代码
  Pair<String> sPair = new Pair<>(String.class);  //这句能够正确执行
  Pair<String> sPair = new Pair<>(Integer.class);  //这句会导致编译器报错
  ```

由于类型擦除

```java
public class Pair<T>{
    private T first;
    private T second;
    //...
    
    //下面的方法会报错，因为进行类型擦除后，会和Object中的equals方法发生冲突
    public boolean equals(T o){
        return true;
    }
    
    //..另一个例子
    public void test(List<String>){}
    public void test(List<Integer>){}
    //上面两行代码，编译器也会报错，因为进行泛型擦除后，两个方法的签名是一模一样的
}

```



### 泛型的继承

ArrayList是继承自List，我们可以将ArrayList向上转型为List，如

```java
List<String> list = new ArrayList<String>();

```

但是，不能将`ArrayList<Integer>`向上转型为`ArrayList<Number>`，这两者是没有继承关系的，若假设它们可以互相转型

```java
ArrayList<Number> list = new ArrayList<Integer>();
//由于list的泛型是Number，则可以
list.add((Double) 1.35);
//然而这个list对象的实际类型仍然是ArrayList<Integer>
//而它是不允许添加除Integer以外的对象的
```



```java
public class IntPair extends Pair<Integer>{
    
}

public class Test{
    public static void main(String[] args){
        Class<IntPair> clazz = IntPair.class;
    	Type t = clazz.getGenericSuperClass();
        if (t instanceof ParameterizedType){
            ParameterizedType pt = (ParameterizedType) t;
            Type[] types = pt.getActualTypeArguments(); //由于Pair<Integer>只有一个泛型参数Integer，所以这个Type数组只有一个元素
            Type firstType = types[0];
            Class<?> typeClass = (Class<?>) firstType;
            System.out.println(typeClass);  //java.lang.Integer
        }
    }
}
```

下图是Type类的继承关系

```mermaid
graph TD
A(Type)
B(Class)
C(ParameterizedType)
D(GenericArrayType)
E(WildcardType)
A-->B
A-->C
A-->D
A-->E
```



### 通配符

#### extends通配符

1. 在传参的时候用

（不确定泛型类的类型参数的时候，可以用`?`，若不确定这个类型参数，但是直到它一定是某一个类的子类时，可以用`<? extends SuperClass>` ）

```java
public static void print(Pair<? extends Number> pair){
    //这样，传入的参数Pair的类型参数可以是Integer，也可以是Double，只要都是Number或Number的子类就可以了
    System.out.println(pair.getFirst());
}
//注意   <? extends Number> 通配符只在使用泛型类时使用，定义泛型类时，一律用<T>或者<K>
//如
public class GenericPair<T,K>{
    //这里的T和K指的是某一具体类型
    private T first;
    private K second;
}
```

注意，如果使用了`<? extends Number> obj` 作为方法的参数时，则这个方法内部

* 允许调用`obj.get`方法获得Number的引用

* 不允许调用`obj.set`方法传入Number的引用，唯一例外`setFirst(null)`

  原因：

  ```java
  public class Test{
      public static void testPair(Pair<? extends Number> pair){
          Number num = pair.getFirst(); //这样是安全的，因为获取到的first无论是什么类型，它一定是Number或Number的子类，这样赋值是安全的（向上转型）
          Float f = new Float(3.5f);
          pair.setFirst(f); //这样是不安全的,编译器会报错，因为不知道Pair的具体类型参数是什么，如这里的例子，main函数中传入的pair的类型是Pair<Double>，而这里试图将一个Float变量设置进去
      }
      public static void main(String[] args){
          Pair<Double> doublePair = new Pair<Double>(4.5);
          testPair(doublePair);  
          
      }
  }
  ```

  

* 其实，若使用`<? extend Number>`作为方法参数时,则这个类型的所有涉及到操作`<T>`的方法都不允许使用

  ```java
  public class Pair<T>{
      private T[];
      public T get(int index){
          //..
      }
      public void add(T t){
          //..
      }
      public void remove(T t){
          ..//
      }
      public void set(int index,T newT){
          T[index] = newT;
      }
  }
  
  public class Test{
      public static void test(Pair<? extends Number> pairNum){
          pairNum.get(1);  //允许执行
          Number num = new Float(1.5f);
          Double d = new Double(2.5f);
          pairNum.add(num);  //编译会报错，不允许执行   <? extends Number> cannot be applied to java.lang.Number
          pairNum.add(d);   //编译会报错，不允许执行    <? extends Number> cannot be applied to java.lang.Number
      }
      public static void main(String[] args){
          Pair<Double> dp = new Pair<>();
          test(dp);
      }
  }
  
  //上述的一些方法，若使用了<? extends Number>，则
  //只允许使用 get方法
  //不允许使用add,remove,set方法，因为它们要不就是参数中包含了具体类型T，要不就是需要对T变量进行操作
  ```

2. 在定义泛型类的时候用

   ```java
   public class Pair<T extends Number>{
       //则这个类型参数被限定是Number或Number的子类
   }
   ```

   

#### super通配符

当一个方法的参数是`<? super Integer> obj` 的时候，在这个方法的内部可以调用`obj.set`方法，只要参数是Integer的超类，即可，然而，无法调用`obj.get`，因为此时获取到的只能肯定是Integer的超类

* 允许使用set方法，传入Integer的引用
* 不允许使用get方法，获取Integer的返回值，除非用Object类型来接收返回值

```java
public static void print(Pair<? super Integer> pair){
    Integer i = 15;
    pair.setFirst(i); //这样是合法的,因为将Integer赋值给其超类，是可以的
    Integer num = pair.getFirst();  
    //上面的语句，编译器会报错，required Number,found <? super Integer>
    //无法确认get方法的返回值是什么类型
    
    //唯一例外，使用Object来接受get函数的返回类型
    Object obj = pair.getFirst();
    
}
```



在定义泛型类时，可以用`<T super Integer>` 来限定某一泛型类的类型参数必须时Integer的超类



比较方法参数为`<? extends Number>` 和`<? super Number>`的区别

* `<? extends Number>` 允许在方法内部调用泛型类的get方法，获取T的引用，不允许调用set等方法，传入Number的引用
* `<? super Number>` 允许在方法内部调用泛型类的set等方法，传入T的引用，不允许调用get方法，获得T的引用，除非用Object对象类接收get方法的返回值

#### 无限定通配符

若一个方法的参数中包含`<?>`通配符，则

* 不允许调用泛型类的set等方法，除非是set(null)
* 调用泛型类的get方法，只能用Object来接收
* 即，既包含了extends通配符的限制，又包含了super通配符的限制

```java
public static void test(Pair<?> pair){
    Object obj = pair.get();
}
```

注意`Pair<?>`和`Pair`是不同的



通常我们引入泛型参数`<T>`来消除`<?>`通配符，如下

```java
public static<T> void test(Pair<T> p){
    Object obj = p.get();
}
```



### 总结

* JAVA的泛型，是采用了**类型擦除**的方法实现的
* **类型擦除**决定了泛型`<T>`
  * 不能是基本数据类型，如int,double
  * 不能获取带有泛型类型的Class，如`Pair<String>.class`
  * 不能判断带有泛型类型的类，如`x instanceof Pair<String>`
  * 不能实例化T，如`new T()`
  * 泛型方法要注意防止重复定义方法，如`public boolean equals(T o)`
* 继承了带泛型类型，子类可以获取到父类的泛型类型`<T>`



### 泛型与反射

部分反射API是泛型

* `Class<T>`是泛型，如

```java
Class<Integer> clazz = Integer.class;
Integer a = clazz.newInstance();
Class<? super Integer> father = clazz.getSuperclass();
```

* `Constructor<T>`也是泛型，如

```java
Class clazz = Person.class;
Constructor<Person> cons = clazz.getConstructor(String.class,int.class);
Person p = cons.newInstance("haha",123);
System.out.println(p);       //haha,123

//以下是Person
public class Person{
    private String name;
    private int age;
    public Person(String name,int age){
        this.name = name;
        this.age = age;
    }
    public String toString(){
        return this.name+","+this.age;
    }
}
```



* 泛型数组

可以声明泛型数组，但不能用new来创建带泛型的数组

```java
Pair<String>[] ps = null;   //ok
Pair<String>[] ps = new Pair<String>[2];   //Complie error
//必须通过强制转型来实现泛型数组
Pair<String>[] ps = (Pair<String>[]) new Pair[2];  //ok
```



使用泛型数组的 时候，要特别小心，因为数组在运行期是没有泛型的

```java
//不安全地使用泛型数组
Pair[] arr = new Pair[2];
Pair<String>[] ps = (Pair<String>[]) arr;
ps[0] = new Pair<String>("a","b");
arr[1] = new Pair<Integer>(1,2);

Pair<String> p1 = ps[1]; 
String s = p1.getFirst();   //编译时没错，运行时抛出异常,ClassCastException

//安全地使用泛型数组
Pair<String>[] ps = (Pair<String>[]) new Pair[2];    //只让Pair<String>[] ps来引用这个数组，不让Pair[] 暴露出来
...
    
    //因为带泛型的数组实际上时编译器进行的类型擦除
```

不能直接创建T[ ]数组。可以通过借助`Class<T>`，或可变参数，来创建T[ ]

* 借助`Class<T>`和`Array.newInstance()`

```java
public class Pair<T>{
    private T first;
    public T[] getTs(){
        return new T[5];  //Type parameter 'T' cannot be instantiated directly
    }
}
//需要借助Class<T>和Array.newInstance

public class Pair<T>{
    private T first;
    public T[] getTs(Class<T> clzz){
        return (T[]) Array.newInstance(clzz,5);  //ok
    }
}
```

* 传入可变参数

  ```java
  public class Pair<T>{
      private T first;
      public static<K> K[] getTs(K ...args){
          return (K[]) args;  
      }
  }
  ```

  

## java集合

### List

`List<E>`是一种有序链表，List的元素可重复，**元素可为null**

```java
List<Integer> list = new ArrayList<Integer>();
list.add(1);
list.add(2);
list.add(null);
list.add(3);
System.out.println(list.size()); //4
```

遍历一个List

* `get(int index)`

* `Iterator<E>`

  ```java
  List<String> list = .....;
  for(Iterator<String> it = list.iterator();it.hasNext();){
      String s = it.next(); 
      //it.next会自动获取下一个元素，并将iterator向后移一位
  }
  ```

* `foreach`

  ```java
  List<String> list = ....;
  for(String s:list){
      System.out.println(s);
  }
  ```

List和Array转换

* List -> Array
  * `Object[] toArray()`

  ```java
  List<String> list = new ArrayList<>();
  list.add("1");
  list.add("2");
  Object[] arr = list.toArray();
  ```
  * `<T> T[] toArray(T[] a)`

  ```java
  List<Integer> list = new ArrayList<>();
  list.add(1);
  list.add(2);
  list.add(3);
  Integer[] arr = list.toArray(new Integer[list.size()]);
  
  //如果传入的T[] a 的长度小于list的长度，则会丢掉这个传入的数组，创造一个新的数组
  Integer[] arrs = list.toArray(new Integer[2]);
  //arrs = [1,2,3]
  //如果传入的T[] a的长度大于list的长度，后面的会为null
  Integer[] arrb = list.toArray(new Integer[5]);
  //arrb = [1,2,3,null,null]
  ```

* Array -> List

  * `<T> List<T> Arrays.asList(T... a)`

    注意这个方法获取的List是**不可变的**

    ```java
    Integer[] arr = {1,2,3};
    List<Integer> list = Arrays.asList(arr);
    //注意返回的是一个List，是Arrays内部实现的一个List类，不是ArrayList，并且这个List是只读的，是不可变的
    list.add(4);  //UnsupportedOperationException
    
    //若想把一个Array变成一个ArrayList,可变的
    Integer[] arr = {2,3,4};
    List<Integer> readOnlyList = Arrays.asList(arr);
    List<Integer> arrList = new ArrayList<>();
    arrList.addAll(list);
    
    //整合一下
    Integer[] arr = {4,5,6};
    List<Integer> arrList = new ArrayList<>(Arrays.asList(arr));
    ```



#### 关于重写equals方法

List中的contains和indexOf方法，用来查找一个元素是否存在于List中，这时就涉及到元素的比较，在集合框架中，元素的比较是使用了equals方法

若要将某个自定义的POJO正确放置在java集合中，如List,Map，需要正确重写equals方法，且要正确重写hashCode方法

equals方法的规范

1. 自反性：x.equals(x) 一定是true
2. 对称性：x.equals(y)一定与y.equals(x)的结果一致
3. 传递性：若x.equals(y)为true，且y.equals(z)为true，那么x.equals(z)也为true
4. 一致性：若x,y引用的对象没有发生改变，那么反复调用x.equals(y)应返回相同的结果



就**对称性**来讲，当参数不属于同一类时，需要好好考虑一下，如：

```java
public class Employee{
    private String name;
    private int salary;
    @Override
    public boolean equals(Object obj){
        //....
    }
}

public class Manager extends Employee{
    private int bonus;
}
```

现有父类Employee对象e和子类Manager对象m，且这两个对象的`name`和`salary`都相同，如果在Employee类的equals方法中，使用instanceof来检测，则返回true，然而如果反过来调用m.equals(e)，也需要返回true(根据对称性原则)。然而，这样就使得Manager类受到了束缚，这个类的equals方法必须能够让Manager类与任何一个Employee对象进行比较，而不考虑Manager类拥有的那部分特殊信息(bonus)。

若子类可以拥有自己的相等概念，则**对称性**原则要求采用`getClass()`来进行检测

若仅仅由父类来决定相等的概念，则可以用`instanceof`进行检测，这样可以在不同的子类对象之间进行比较



下面是实现一个比较完美的equals方法的建议：

1) 显式参数命名为otherObject，稍后需要将它转换成一个叫other的变量（关于**显式参数**和**隐式参数** ->   调用函数x.equals(y), 那么x就是隐式参数，y就是显式参数）

2)先检测this和otherObject是否引用同一个对象，该语句实际只是一个优化，因为计算这一类等式要比一个一个地比较类中的域所付出的开销小很多

```java
if(this == otherObject) return true;
```

3)检测otherObject是否为null

```java
if(otherObject == null) return false;
```

4) 比较this和otherObject是否属于同一类，如果该类的子类有自己独立的相等概念，则用`getClass()`比较，若该类的所有子类的相等概念都相同(相等概念仅在父类中定义)，则用`instanceof`

5)将otherObject转换为相对应的类的变量

```java
//如该equals方法是定义在Employee中，则
Employee other = (Employee) otherObject;
```

6) 现在开始对所需要比较的域进行依次比较



注：如果要在该类的子类中重新定义equals方法，就要在其中包含调用super.equals()

注意重写Object的equals方法时，要注意参数一定得和Object里的equals一致

```java
public class Employee{
    @Override
    public boolean equals(Object obj){
        //...
    }
}
```



- ArrayList   基于数组，随机访问快，添加/删除节点慢
- LinkedList 基于链表，随机访问慢，添加/删除节点快



### Map

是一种Key-Value键值对的映射表

遍历map

* 通过`keySet()`获取到key的集合，再依次取出对应的value

* 通过`entrySet()`获取到entry的集合，再依次去除对应的key和value

  ```java
  Map<String,Integer> map = new HashMap<>();
  map.put("大黄",1);
  map.put("金霞",2);
  map.put("金毛",3);
  for (Map.Entry<String,Integer> entry:map.entrySet()){
  	System.out.println(entry.getKey()+","+entry.getValue());
  }
  //注意HashMap不保证里面的元素有序，遍历时，既不保证是put放入的顺序，也不保证按照key的排序顺序
  
  //SortedMap可以保证遍历时时以key的顺序来遍历的，其实现类是TreeMap
  ```

下面是map的类图

![map类图](https://i.loli.net/2019/06/09/5cfcc1b0aa2e556775.png)

#### equals和hashCode

在Map内部，对key作比较，使用的是equals方法，所以作为key的对象必须正确重写equals方法

**HashMap**是通过计算key的hashCode()来定位该元素的存储位置的

若要正确使用Map,需要保证：

* 作为key的对象，必须要正确重写equals方法和hashCode方法

重写hashCode方法，可以利用Objects.hash()辅助实现

* 若两个对象equals为true，则hashCode一定也为true
* 但反过来不一定成立，尽可能保证不同的对象的hashCode也不同，这样有利于提高效率



### Properties

用于读取配置文件（String），但是xx.properties文件中**只能使用ASCII编码**

可获取文件系统的properties文件，也可以读取classpath下的properties文件

```java
String file = "D:\\conf\\setting.properties";
Properties props = new Properties();
props.load(new FileInputStream(file));
String user = props.getProperty("user");
String pw = props.getProperty("password","root"); //若获取不到，则取默认值root

props.load(JavaTest.class.getResourceAsStream("/db.properties"));
String user = properties.getProperty("user");
String none = properties.getProperty("none","default");

//properties可以多次load,如果后面的properties文件中存在相同的key，则会覆盖前面的load的文件中的key-value
```

Properties是继承自HashTable的，这是有设计缺陷的。

使用Properties时，记得只使用getProperty和setProperty方法，而不要去调用从HashTable继承而来的get和put



### Set

用于存储不重复的元素集合

* `add(E e)`
* `remove(Object o)`
* `contains(Object o)`

Set相当于只存储Key的Map

Set用于去除重复元素

放入Set的元素要正确重写equals和hashCode方法

* HashSet是无序的
* TreeSet是有序的

**可以使用Set来达到去重的效果**

```java
List<String> list1 = Arrays.asList("apple","banana","apple");
Set<String> set = new HashSet<>(list1);
List<String> monoList = new ArrayList<>(set);
```



Set的类图和Map类似



### Queue

`Queue<E>`是一个接口，是一个FIFO的队列

LinkedList实现了`Queue<E>`接口，常用方法如下

* `add(E e) / offer(E e)`添加元素到队尾
* `remove() / poll()` 获取队首元素并删除
* `element()  / peek()` 获取队首元素，但不删除

要避免将null值添加到队列中

当添加/获取元素失败时

|                    | throw Exception | 返回false或null |
| ------------------ | --------------- | --------------- |
| 添加元素到队尾     | `add(E e)`      | `offer(E e)`    |
| 取队首元素并删除   | `E remove()`    | `E poll()`      |
| 取队首元素但不删除 | `E element()`   | `E peek()`      |

#### PriorityQueue

带有优先级的队列

`PriorityQueue<E>`是一个class，其实现了`Queue<E>`接口

获取队首元素，总是先返回优先级更高的元素，放入PriorityQueue的对象，必须要实现Comparable接口

#### Deque

双端队列(Double Ended Queue)，队首和队尾都能添加和删除元素

Deque是一个接口，其实现类有ArrayDeque，LinkedList

|                  | Queue              | Deque                       |
| ---------------- | ------------------ | --------------------------- |
| 添加元素到队尾   | add() / offer()    | addLast() / offerLast()     |
| 取队首元素并删除 | remove() / poll()  | removeFirst() / pollFirst() |
| 取队首元素不删除 | element() / peek() | getFirst() / peekFirst()    |

```java
Deque<String> deq = new LinkedList<>();
deq.offerFirst("big");

//始终用接口来引用对象，能具有更好的语义，更高层次的抽象
```



### Stack

是一种LIFO的数据结构，常用的方法有

* push(E e) : 把元素压栈
* pop(E e) : 把栈顶元素弹出

**可以用Deque来模拟实现Stack**，注意只调用下面的方法

栈的操作  -> Deque中对应的方法

push - > addFirst

pop - > removeFirst

peek - > peekFirst

java是没有stack接口的，这是因为有一个遗留的class，叫做Stack

JVM中方法的嵌套调用就是用栈来实现的

栈的应用：

* 进制转换：将余数压栈，直到商为0为止
* 后缀表达式的计算（注意中缀表达式转后缀表达式）



### Iterator

Java的集合类可以使用for..each循环，这是因为它们实现了Iterable接口，实际java编译器将for..each循环，通过Iterator转换为for循环，进行了遍历。

若要自己编写的类也能使用for..each循环，则需要

- 实现Iterable接口，并正确返回Iterator对象

使用Iterator进行迭代的好处：

* 对任何集合类型都采用同一种访问模式
* 调用者对集合内部的结构一无所知（很好的封装性）
* Iterator是一种抽象的数据访问模型

在编写代码时，可以通过一个内部类来实现Iterator接口。

内部类可以直接访问外部类的字段和方法

### Collections

是JDK提供的工具类

* `addAll(Collection<? super T> c, T... elemenst)`

创建空的集合（注意创建的集合是不可变的，不能向其中添加元素）

* `List<T> emptyList()` 
* `Map<K,V> emptyMap()`
* `Set<T> emptySet()`

* `sort(List<T> list)` 必须传入可变的List

* `shuffle(List<T> list)` 随机重置List的元素，如传入一个有序的list，则会打乱顺序

* `unmodifiableList<List<? extends T> list>`将可变List转换为不可变List

  ```java
  //这个方法得到的并不是一个真正不可变的List
  List<String> list = new ArrayList<>(Arrays.asList("A"));
  List<String> readOnly = Collections.unmodifiableList(list);
  list.add("B");  //调用这一句，也会导致变量readOnly发生改变
  //故，使用unmodifiableList方法，应该马上将原有的list的引用给丢弃掉
  //如下
  
  List<String> readOnly = Collections.unmodifiableList(new ArrayList<>(Arrays.asList("A")));
  ```

  

* `synchronizedList(List<T> list)` 将线程不安全的List转换为线程安全的List（由于jdk1.5之后添加了concurrent包，所以这些方法已不推荐使用了）

=========

注意区分Collection和Collections  [参考链接](https://www.cnblogs.com/cathyqq/p/5279859.html)

[Collection参考链接](https://www.cnblogs.com/taiwan/p/6954135.html)

[参考链接2](http://www.importnew.com/23715.html)

来源于java.util包

Collection的类图：

![1555985652206](https://i.loli.net/2019/06/09/5cfcc1e4e825349961.png)

Java集合的特点：

* 接口和实现分离

  List接口，ArrayList，LinkedList实现类

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



## java IO

### IO基础

#### 总结

* IO流是一种流式的数据输入/输出模型
* 二进制数据以byte为最小单位在InputStream/OutputStream中单向流动
* 字符数据以char为最小单位在Reader/Writer中单向流动
* JDK的java.io包提供了基础的同步IO功能（java.nio包提供了异步IO功能）
* Java的IO流是**接口**和**实现**分离的
  * 字节流接口：InputStream/OutputStream
  * 字符流接口：Reader/Writer



====================================

IO，即Input/Output

Input:将数据从外部读取到内存，如，读文件，从网络读取等

Output:将数据从内存输出到外部，如，写文件，输出到网络



**IO流**（字节流）：一种顺序读写数据的模式（像水管里的水流）

* 单向流动
* 以byte为最小单位（字节流）



**字符流**（若字符不是单字节表示的ASCII码，则可以使用字符流）

* JAVA提供了Reader/Writer
* 传输的最小单位是char
* 字符流输出的byte取决于编码方式



**Reader/Writer** : 本质上，是能自动编码/解码的InputStream / OutputStream

使用Reader/Writer还是InputStream/OutputStream，取决于具体的应用场景。

* 若数据源不是文本，则只能使用InputStream/OutputStream（按字节传输）
* 若数据源是文本，则使用Reader/Writer 更方便一些（自动对字节解码/编码，按字符来传输）



**同步IO & 异步IO**

同步IO：（ java.io.* ）

* 读写IO时，代码必须等待IO操作结束后，才继续执行后续代码
* 代码编写较简单，CPU效率低下

异步IO：（java.nio.*）

* 读写IO时，仅发出IO请求，然后立刻继续执行后续代码
* 代码编写较复杂，CPU效率高



同步IO的类图关系如下

```mermaid
graph BT
A(InputStream)
B(OutputStream)
C(Reader)
D(Writer)
A1(FileInputStream)
B1(FileOutputStream)
C1(FileReader)
D1(FileWriter)
A1-->A
B1-->B
C1-->C
D1-->D
```

java的IO接口主要包括了上图上方的4个抽象类

其实现类，以读写文件为例，有上图下方的4个实现类。

 

### File对象

#### 总结

* File对象表示一个**文件**或**目录**
* 创建File对象本身不涉及IO操作
* 获取路径/绝对路径/规范路径      `getPath()` / `getAbsolutePath()` / `getCanonicalPath()`
* 可以获取目录下的文件和子目录，可以创建或删除文件或目录



`java.io.File` 表示文件系统的一个**文件**或**目录**

```java
/*创建一个File对象*/
/*@paramete:pathName 注意不同操作系统下路径格式不一样*/

//Windows下，用2个反斜杠来做路劲分隔
File f = new File("C:\\Users\\Administrator\\");

//Linux下，用正斜杠
File f = new File("/usr/local/boot.xml");

//可以传入绝对路径
File f = new File("C:\\Users\\Administrator\\yogurt\\test.txt");
//也可以传入相对路劲
File f = new File("yogurt\\test.txt");
File f = new File(".\\yogurt\\test.txt");    //1个点表示当前目录
File f = new File("..\\yogurt\\test.txt");   //2个点表示上一级目录

//返回File的路径
File f = new File("..");
String path = f.getPath(); // ".."    返回的是构造函数传入的路径
String apath = f.getAbsolutePath();  // "D:\\Workspace\\yogurt\\.."    绝对路径
String cpath = f.getCanonicalPath();  // "D:\\Workspace"     规范路径

//boolean isFile()  判断是否是一个文件
//boolean isDirectory()  判断是否是一个目录
//注意，构造一个File对象时，不会产生实际的IO操作，只要当调用File对象的某些方法时，才真正进行IO操作，如，调用上面的2个方法。故构造一个File对象时，如果该文件不存在，也不会报错

//当一个File对象表示一个文件时，可以调用：

//boolean canRead() ： 是否允许读取该文件
//boolean canWrite() : 是否允许写入该文件
//boolean canExecute() : 是否允许运行该文件
//long length() : 获取文件大小
//boolean createNewFile() : 创建一个新的文件
//static boolean createTempFile(String prefix,String suffix) ： 创建一个临时文件
//boolean delete()
//void deleteOnExit() : 在JVM退出的时候，删除该文件，通常配合临时文件使用


//当一个File对象表示一个目录时，可以调用：
//String[] list() : 列出目录下的文件和子目录名
//File[] listFiles() : 列出目录下的文件和子目录
//File[] listFiles(FileFilter filter)
//File[] listFiles(FilenameFilter filter)
//boolean mkdir() : 创建该目录
//boolean mkdirs() : 创建该目录，并将不存在的父级目录也创建出来
//boolean delete() : 删除该目录

File dir = new File("C:\\Windows");
File[] files = dir.listFiles(new FilenameFilter(){
   @Override
    public boolean accpet(File dir,String name){
        return name.endsWith(".exe");
    }
});
//过滤出exe文件

File dir = new File("D:\\Example\\test");
dir.mkdir(); //要求 D:\Example 目录必须存在
dir.mkdirs(); //若 D:\Example 不存在，就自动创建
```



### InputStream

#### 总结

* InputStream定义了所有输入流的父类
* FileInputStream实现了文件流输入
* ByteArrayInputStream在内存中模拟一个字节流输入
* 使用try-with-resource保证InputStream被正确关闭

`java.io.InputStream`是所有输入流的父类，其包含以下方法

* `abstract int read()` ：读取**一个**字节，并返回字节代表的整数（0-255），若已读到末尾，返回-1
* `int read(byte[] b)` ：读取若干字节并填充到`byte[]`数组中，返回读取的字节数
* `int read(byte[] b,int off,int len)` ：指定`byte[]`数组的偏移量和最大填充数
* `void close()` ：关闭输入流

```java
//完整地读取一个InputStream里的所有字节
InputStream input = new FileInputStream("src/test.txt");
int n;
while((n = input.read()) != -1){
    System.out.println(n);
}
input.close();
//需要用try..finally语块来保证最终都会将InputStream关闭

//jdk1.7 有新增一个try-with-resource方式，这实际只是一个语法糖，编译时编译器会自动加上finally并调用close方法，前提是这资源有实现Closeable接口
try(InputStream input = new FileInputStream("src/test.txt")){
    int n;
    while((n = input.read()) != -1){
        System.out.println(n);
    }
}//在此会自动关闭InputStream
```



```java
//一次读取多个字节
try(InputStream input = new FileInputStream("src/test.txt")){
    byte[] buffer = new byte[1000];
    int n = input.read(buffer);//一次可读取最多1000个字节
}
```



注意InputStream中的read方法是**阻塞**（blocking）的，必须等待read执行完毕，才会执行后面的代码



**ByteArrayInputStream**可在内存中模拟一个InputStream，它实际能把一个`byte[]`变成一个InputStream，这个类可以在测试的时候用来构造InputStream

```java
byte[] data = {72,103,112,126,25};
try(InputStream input = new ByteArrayInputStream(data)){
    int n;
    while((n = input.read()) != -1){
        System.out.println(n);
    }
}
```



### OutputStream

#### 总结

* OutputStream 是所有输出流的父类
* FileOutputStream实现了文件流输出
* **ByteArrayOutputStream在内存中模拟一个字节流输出**（可用这个类来暂时储存读取到的byte数组，最后调用它的toByteArray方法，来将流中的数据全部转换为字节数组，起到一个缓冲的作用）
* 使用try-with-resource保证OutputStream正确关闭



`OutputStream`是所有输出流的父类，其包含如下方法

* `abstract write(int b)` : 输出一个字节
* `void write(byte[] b)`：输出`byte[]`数组中的所有字节
* `void write(byte[] b,int off,int len)` ：输出`byte[]`数组中指定范围的字节
* `void close()` 
* `void flush()` ：将缓冲区的内容输出。通常情况我们不用调用该方法，因为缓冲区满了时，会自动调用。在调用`close`关闭输出流之前，也会自动调用`flush`

在向**电脑磁盘**，**网络**输出数据时，出于效率考虑，并不是输出一个字节，就立即输出，而是将待输出的字节，放入到内存的一个缓冲区里，当这个缓冲区满了以后，再一次性全部输出。对于很多设备来说，一次写入1个字节，和一次写入1000。花费的时间是一样的。



### Filter模式

JDK提供的InputStream包括：

- FileInputStream：从文件读取数据
- ServletInputStream：从HTTP请求读取数据
- Socket.getInputStream( ) ：从TCP连接读取数据



若要给FileInputStream添加额外的功能（如，添加缓冲区，计算签名，添加加解密），可以派生FileInputStream的子类，但是这样以来，对其他的InputStream实现这些功能，也需要进行派生子类，且如果功能有交叉，产生的子类的数量非常之大。故

JDK将InputStream分为两类：

* 直接提供数据的InputStream:

  FileInputStream, ByteArrayInputStream,ServletInputStream

* 提供额外附加功能的InputStream：

  BufferedInputStream,DigestInputStream,CipherInputStream

* 通过组合功能，而非继承，这样的设计模式称为Filter模式，或者Decorator模式
* 可以通过少量的类来实现各种功能的组合

使用InputStream时，可以根据情况进行组合

![inputstream](https://i.loli.net/2019/06/10/5cfe5791a0f5835143.png)

```java
InputStream input = new GZIPInputStream(
    new BufferedInputStream(
        new FileInputStream("test.gz")
    )
);
```



InputStream/OutputStream继承体系

![in](https://i.loli.net/2019/06/10/5cfe62981d55a59054.png)

![out](https://i.loli.net/2019/06/10/5cfe62e162b1681253.png)



**小结**：

* JAVA IO使用Filter模式为InputStream/OutputStream增加功能
* 可以把一个InputStream和任意FilterInputStream组合，OuputStream同理
* Filter模式可以在运行期动态增加功能（又称为Decorator模式）



例子：读取文件时，计算读取的字节数（文件的大小）

```java
import java.io.*;
public class CountInputStream extends FilterInputStream{
        public int count = 0;
        public CountInputStream(InputStream in) {
            super(in);
        }
    
    //重写read(byte[] b,int off,int len)就行了
    //不必重写read(byte[] b)，因为该方法的实现也是调用了read(byte[] b,int off,int len)
        @Override
        public int read(byte[] b, int off, int len) throws IOException {
            int n = super.read(b, off, len);
            count += n;
            return n;
        }
}

public class Test{
    public static void main(String[] args){
        CountInputStream countIn = new CountInputStream(new FileInputStream("E:\\test.txt"));
        ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
       int n;
       byte[] bytes = new byte[1024];
       while((n = countIn.read(bytes)) != -1){
           outputStream.write(bytes); //先将读取到的byte数组写到字节输出流中
       }
        //将流中的字节全部转换成byte数组
       String res = new String(outputStream.toByteArray(),"UTF-8");  
       System.out.println(res);
       System.out.println("size : " +countIn.count+" bytes" );
    }
}
```

### ZIP

ZipInputStream也是一种FilterInputStream

![zip](https://i.loli.net/2019/06/10/5cfe6bed80f7b25208.png)

* 可以直接读取Zip的内容





#### ZipInputStream

基本用法

```java
public static void main(String[] args) throws IOException {
        byte[] bytes = new byte[1024];
        try (ZipInputStream zip = new ZipInputStream(new FileInputStream("E:\\haha.zip"))) {
            //对于每一个ZipEntry，都表示一个压缩文件或者目录
            ZipEntry entry = null;
            while ((entry = zip.getNextEntry()) != null) {
                ByteArrayOutputStream byteArrayOut = new ByteArrayOutputStream();
                String name = entry.getName();
                if (!entry.isDirectory()) {
                    int n;
                    while ((n = zip.read(bytes)) != -1) {
                        byteArrayOut.write(bytes);
                    }
                }
                String res = new String(byteArrayOut.toByteArray(),"UTF-8");
                System.out.println("文件名："+name+"\n文件内容：\n"+res+"\n==================================");
            }
        }
    }
```

输出结果：

![out](https://i.loli.net/2019/06/10/5cfe6f2f7d05323506.png)



#### ZipOutputStream

基本用法

```java
public static void main(String[] args) throws IOException {
        byte[] bytes = new byte[1024];
        try(ZipOutputStream zipOut = new ZipOutputStream(new FileOutputStream("E:zipOut.zip"))){
            File[] files = {new File("E:\\haha.txt"),new File("E:\\lalala.txt")};
            for (File file : files){
                //一个Entry表示一个文件或目录
                zipOut.putNextEntry(new ZipEntry(file.getName()));
                //写入
                zipOut.write(getFileDataAsBytes(file));
                //该Entry写入完毕
                zipOut.closeEntry();
            }
        }
    }

public static byte[] getFileDataAsBytes(File f) throws IOException {
        InputStream input = new FileInputStream(f);
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        int n;
        byte[] bytes = new byte[1024];
        while((n = input.read(bytes)) != -1){
            byteArrayOutputStream.write(bytes);
        }
        return byteArrayOutputStream.toByteArray();
    }
//最终生成了一个zipOut.zip文件，里面包含了2个文件，haha.txt和lalala.txt
```



#### 小结

* ZipInputStream可以读取Zip格式的流
* ZipOutputStream可以把数据写入Zip
* ZipInputStream/ZipOutputStream都是FilterInputStream/FilterOutputStream
* 配合FileInputStream/FileOutputStream就可以对zip文件进行读写



### classpath

从classpath下读取文件，可以避免不同环境下文件路径不一致的问题

在classpath路径下的文件，路径总是以`/`开头

```java
InputStream in = getClass().getResourceAsStream("/db.properties");
```



### 序列化

指的是把一个Java对象变成二进制内容（即 `byte[]`）

* 序列化后可以把`byte[]`通过网络进行传输，或保存在文件中

一个可序列化的Java对象，必须实现`Serializable`接口，`Serializable`接口没有定义任何方法，这样的空接口被称为`标记接口`（Marker Interface）



反序列化，类似。



ObjectOutputStream

可以将一个Java对象，写入到二进制流

```java
try（ObjectOutputStream out = new ObjectOutputStream(...)）{
    out.writeObject(new Person("Big Yellow"));
}
```



ObjectInputStream

负责从二进制流读取一个Java对象

```java
try(ObjectInputStream in = new ObjectInputStream(...)){
    Object p1 = in.readObject();
    Person p2 = (Person) in.readObject();
    //readObject()可能抛出异常： ClassNotFoundException，InvalidClassException
    
    //ClassNotFoundException：机器A的JAVA程序传递给机器B的JAVA程序，A中有定义Person类，而B中没定义Persion类，就会报这个错
    //InvalidClassException：机器A定义的Person对象有一个int age，机器B定义的Person中时long age，则会报这个错，Class不匹配
}
```



**反序列化的特点**：（重要）

* 反序列化由JVM直接构造出Java对象，**不调用构造方法**



```java
//测试
try(ObjectOutputStream output = new ObjectOutputStream(new FileOutputStream("/out.data"))){
            output.writeInt(777);
            output.writeUTF("We will not use hello world");
            output.writeObject(new Person("yogurt",23));
        }
        
        try(ObjectInputStream input = new ObjectInputStream(new FileInputStream("/out.data"))){
            System.out.println(input.readInt());
            System.out.println(input.readUTF());
            System.out.println(input.readObject());
        }
```



**小结**

* 要使Java对象可序列化，必须实现Serializable接口
* Serializable接口是空接口，这样的接口被称为”标记接口“
* 反序列化时，不调用构造方法
* 可设置serialVersionUID作为版本号（非必需）
* Java的序列化机制，仅适用于Java程序之间交互数据，若要与其他语言交换数据，需要使用更通用的序列化方法，如JSON



**注意**：

若不显示指定serialVersionUID，则JAVA序列化机制会根据Class自动生成一个serialVersionUID作为序列化版本用，这种情况下，若Class不发生修改（类名修改，增删字段或方法），则每次编译生成的serialVersionUID皆时一致的，但是若Class发生了修改，编译生成serialVersionUID就会不同，这时，反序列化检测到接收的序列化流中的serialVersionUID与本地Class的serialVersionUID不一致，就会报错（InvalidClassException）



例子如下：

```java
/*try(ObjectOutputStream output = new ObjectOutputStream(new FileOutputStream("/out.data"))){
            output.writeInt(777);
            output.writeUTF("We will not use hello world");
            output.writeObject(new Person("yogurt",23));
        }*/
//首先通过上面被注释掉的代码，将Person写入到out.data
//随后注释掉上面代码，运行下面代码，发现输出了正确结果
//若此时，对Person类进行一定的修改，如增加一个方法域
//再执行下面代码，即会报错

        try(ObjectInputStream input = new ObjectInputStream(new FileInputStream("/out.data"))){
            System.out.println(input.readInt());
            System.out.println(input.readUTF());
            Person person = (Person) input.readObject();
            System.out.println(person);
        }
```

```
777
We will not use hello world
Exception in thread "main" java.io.InvalidClassException: com.yogurt.test.Person; local class incompatible: stream classdesc serialVersionUID = -2744247394581433250, local class serialVersionUID = 1594032614617864095
	at java.io.ObjectStreamClass.initNonProxy(ObjectStreamClass.java:616)
	at java.io.ObjectInputStream.readNonProxyDesc(ObjectInputStream.java:1843)
	at java.io.ObjectInputStream.readClassDesc(ObjectInputStream.java:1713)
	at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2000)
	at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1535)
	at java.io.ObjectInputStream.readObject(ObjectInputStream.java:422)
	at com.yogurt.test.JavaTest.main(JavaTest.java:23)

Process finished with exit code 1

```



### Reader

Reader是所有字符输入流的父类

`java.io.Reader`和`java.io.InputStream`的区别

| InputStream                                    | Reader                                      |
| ---------------------------------------------- | ------------------------------------------- |
| 字节流，单位是`byte`                           | 字符流，单位是`char`                        |
| `int read()` 返回 -1，或0~255  （一个字节8位） | `int read()`返回-1，0~65535（一个字符16位） |
| `int read(byte[] b)`，读取字节数组             | `int read(char[] c)`，读取字符数组          |



例子：完整地读取一个Reader里的所有字符

```java
Reader reader = null;
try{
    reader = FileReader("E:\\reader.txt");
    int n;
    while((n = reader.read()) != -1){
        System.out.println((char)n);
    }
}finally{
    if(reader != null)
        reader.close();
}

//利用缓冲区，一次读取多个字符
try(Reader reader = new FileReader("E:\\reader.txt")){  
    //这种try-with-resource是jdk1.7引入的，这里要注意FileReader采用的是系统默认编码
    char[] charBuff = new char[1024];
    int n = reader.read(charBuff);
    System.out.println(n);
}
```



`CharArrayReader`可以在内存中模拟一个Reader，类似于ByteArrayInputStream之于InputStream



Reader实际是基于InputStream构造的

* FileReader内部是持有一个FileInputStream的

* Reader可以通过InputStream来构造

  ```java
  InputStream inStream = new FileInputStream("E:\\test.txt");
  Reader reader = new InputStreamReader(inStream,"UTF-8");
  reader.close();
  //调用了reader的close，就不要再调用inStream的close了
  ```



**小结**

* Reader是所有字符输入流的父类
* FileReader实现了文件字符流输入
* CharArrayReader在内存中模拟一个字符流输入
* Reader是基于InputStream构造，FileReader使用系统默认编码，无法指定编码，可以通过InputStreamReader来指定编码



### Writer

Write是所有字符输出流的父类

`java.io.Writer`和`java.io.OutputStream`的区别

| OutputStream                              | Writer                                    |
| ----------------------------------------- | ----------------------------------------- |
| 字节流，以`byte`为单位                    | 字符流，以`char`为单位                    |
| `void write(int byte)`，写入字节（0~255） | `void write(int char)`，写入字符，0~65535 |
| `void write(byte[] b)`，写入字节数组      | `void write(char[] c)`，写入字符数组      |
|                                           | `void write(String s)`，写入字符串        |

```java
try(Writer writer = new FileWriter("E:\\out.txt")){
    writer.write(65);
    writer.write("Hello".toCharArray()); //写入char[]
    writer.write("World"); //写入String
}
```



可以用`CharArrayWriter`模拟一个字符输出流，类似于`ByteArrayOutputStream`。可以作为一个字符缓冲区

```java
try(Writer writer = new CharArrayWriter()){
    writer.write(65);
    writer.write("Hello");
    char[] buff = writer.toCharArray();
}
```



注意Reader的read方法和Writer的write方法都是阻塞的（和InputStream/OutputStream类似）

Writer也是基于OutputStream来构造的

* FileWrite内部持有一个FileOutputStream

* Writer可以通过OutputStream来构造（可指定编码方式）

  ```java
  OutputStream out = new FileOutputStream("E:\\out.txt");
  Writer writer = new OutputStreamWriter(out,"UTF-8");
  writer.close();
  //调用了writer的close，就不要再调用out的close了
  ```



**小结**

* Writer是所有字符输出流的父类
* FileWriter实现了文件字符输出流
* CharArrayWriter再内存中模拟一个字符输出流，可用做缓冲区
* Writer是基于OutputStream构造，FileWriter使用系统默认编码，无法指定编码，可以通过OutputStreamWriter来指定编码