# Junit

| 版本   | JUnit 3.x                       | JUnit 4              | JUnit 5              |
| ------ | ------------------------------- | -------------------- | -------------------- |
| JDK    | JDK < 1.5                       | JDK >= 1.5           | JDK >= 1.8           |
| class  | class MyTest extends TestCase{} | class MyTest{}       | class MyTest{}       |
| method | public testAbc(){}              | @Test public abc(){} | @Test public abc(){} |

## 基本测试

代码示例

```java
public myUtil{
    public int add(int a,int b){
        return a + b;
    }
}

//===========
import org.junit.Test;
import static org.junit.Assert.*;
public myTest{
    @Test
    public void test1(){
        assertEquals(3,new myUtil().add(1,2));
    }
    
    @Test
    public void test2(){
        assertEquals(5,new myUtil().add(3,4));
    }
}

//运行这个单元测试，显示 Test failed:1, passed: 1 of 2 tests

//junit中可使用junit的断言来进行测试
//断言相等: assertEquals(100,x)
//断言数组相等：assertArrayEquals({1,2,3},x)
//断言浮点数相等：assertEquals(3.1416,x,0.0001)
//断言为null：assertNull(x)
//断言为true/false ：assertTrue(x>0)
//其他：assertNotEquals  / assertNotNull
```



## Before & After

同一个单元测试内的多个测试方法，可能需要在测试前初始化对象，或者测试后释放某些资源

```java
public class MyTest{
    MyUtil util;
    @Before
    public void init(){
        util = new MyUtil();
    }
    @After
    public void destroy(){
        util = null;
    }
    @Test
    public void test(){
        assertEquals(4,util.add(3,1));
    }
    @Test
    public void test2(){
        assertTrue(util.add(2,-1) > 0);
    }
    
    //对于每个@Test方法，Junit按下面步骤执行
    //1. 实例化MyTest类
    //2. 执行@Before方法
    //3. 执行@Test方法
    //4. 执行@After方法
    //实际Junit执行代码如下
    
    for(Method testM : testMs){
        MyTest mytest = new MyTest();
        mytest.init();
        testM.invoke(mytest);
        mytest.destroy();
    }
    //使用@Before，@After可以保证
    //单个@Test方法执行前，会创建新的MyTest实例，实例变量不会传递给下一个@Test方法
    //单个@Test方法执行前后会执行@Before和@After方法
    
    //在使用@Before方法初始化一个对象时，将这个对象存放在MyTest的实例字段中
    //每一个@Test方法使用的都是不同的MyTest实例
    
    //另外：
    //Junit提供@BeforeClass和@AfterClass静态方法
    //在执行所有的@Test方法前，会执行@BeforeClass方法
    //在执行完所有的@Test方法后，会执行@AfterClass方法
    //注意：使用@BeforeClass初始化的对象，只能存放在静态字段中
    //该静态字段状态，会影响到所有@Test
    
    //Junit执行逻辑,生命周期
    invokeBeforeClass(MyTest.class);  //Before
    for(Method testM : allTestMethods(MyTest.class)){
        MyTest mytest = new MyTest();
        mytest.init(); //@Before方法
        testM.invoke(mytest);
        mytest.destroy(); //After方法
    }
    invokeAfterClass(MyTest.class);  //After
}
```



小结：

- 需要梳理JUnit执行测试的生命周期
- @Before用于初始化测试对象，测试对象存放为实例字段
- @After用于释放@Before创建的对象资源
- @BeforeClass用于初始化耗时的资源，存放为静态变量
- @AfterClass用于释放@BeforeClass创建的资源



## 异常测试

对可能抛出的异常进行测试：测试错误的输入是否会导致异常抛出，注意对每种可能发生的异常都要进行测试

使用`expected`来测试异常

```java
@Test(expected = NumberFormatException.class)
public void testEx(){
    Integer.parse(null);
}
//Junit成功捕捉到NumberFormatException，测试通过
```



## 参数化测试

待测试的输入和输出是一组数据，那么

- 可以将测试数据组织起来
- 用不同的测试数据来调用相同的测试方法

代码示例

```java
import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.junit.runners.Parameterized;
import java.util.Arrays;
import java.util.Collection;
import static org.junit.Assert.assertEquals;

@RunWith(Parameterized.class)
public class MyTest {
    MyUtil util;
    
    @Parameterized.Parameters
    public static Collection<?> data(){
        //这个方法的签名固定，必须是静态方法，返回值是Collection<?>，方法名为data
        return Arrays.asList(new Object[][]{{2,1,1},{3,2,1},{4,1,3}});
    }
    @Parameterized.Parameter(0)
    public int result; //这里设置为public,否则Junit取不到值
    @Parameterized.Parameter(1)
    public int num1;
    @Parameterized.Parameter(2)
    public int num2;
    @Before
    public void init(){
        util = new MyUtil();
    }
    @After
    public void destroy(){
        util = null;
    }
    @Test
    public void test(){
        assertEquals(result,util.add(num1,num2));
    }
    
    //测试结果：3组数据全部通过
}
```



## 超时测试

`@Test(timeout=1000)`   注意这里的单位是毫秒，故设置了超时为1秒

