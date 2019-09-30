Spring Expression Language

SpEL表达式是在Spring3引入的，可用于将**值装配**到bean属性和构造器参数中，所使用的表达式会在运行时计算并得到值

SpEL表达式要放在 `#{}` 中

注意属性占位符是放在 `${}`中

SpEL的特性：

1. 引用bean，属性，和方法

   ```xml
   <bean id="sand" class="niceFood.Sandwitch"/>
       <bean id="cd" class="niceFood.Hamburger" c:_0-ref="sand" c:_1="#{sand.getName()}"/>
   ```

   

   ```xml
   <bean id="cd" class="niceFood.Hamburger" c:_0-ref="sand" c:_1="#{sand.name?.toUpperCase()}">
      
       <!-- ?保证了null安全，即在sand.name为null时，则不会调用后面的方法 -->
   ```

2. 在表达式中使用类型（通常用来获取常量或调用静态方法）

   使用 `T()`来获取某一个类型

   ```xml
   <bean id="sand" class="niceFood.Sandwitch"/>
       <bean id="cd" class="niceFood.Hamburger" c:_0-ref="sand" c:_1="#{sand.name}" c:_2="#{T(System).currentTimeMillis()}" />
   
   <bean class="com.math.Util" c:_0="#{T(Math).PI}"/>
   ```

   也可以获取一些环境变量

   ```xml
   <bean id="cd" class="niceFood.Hamburger" c:_0-ref="sand" c:_1="#{systemProperties['user.name']}" />
   ```

   

3. 支持对值的运算

   1. SpEL运算符

      * 算数运算: `+ - * / % ^`

      * 比较运算: `< > == <= >= lt gt eq le ge`

      * 逻辑运算:`and or not`

      * 条件运算: `?:`

        * `#{student.score > 60 ? "Not bad" : "bad"}`
        * `#{student.name ?: "default"}`  （Elvis运算符，用来检测null值）

      * 正则: `matches`

        ```xml
        <bean id="sand" class="niceFood.Sandwitch"/>
        <bean id="cd" class="niceFood.Hamburger" c:_0-ref="sand" c:_1="#{sand.phone matches '\d{11}'}">
        ```

        

   2. 集合运算

      * 通过下标获取集合中某一元素

        `#{students[0].score}`  第一名学生的分数

        ```xml
        <?xml version="1.0" encoding="UTF-8" ?>
        <beans  xmlns="http://www.springframework.org/schema/beans"
                xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                xmlns:util="http://www.springframework.org/schema/util"
                xmlns:c="http://www.springframework.org/schema/c"
                xmlns:p="http://www.springframework.org/schema/p"        xmlns:context="http://www.springframework.org/schema/context"
                xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
            http://www.springframework.org/schema/context
            http://www.springframework.org/schema/context/spring-context-4.0.xsd
            http://www.springframework.org/schema/util
            http://www.springframework.org/schema/util/spring-util-4.0.xsd
            ">
            
        <util:list id="songList">
                <value>不能说的秘密</value>
                <value>简单爱</value>
                <value>晴天</value>
                <value>发如雪</value>
                <value>彩虹</value>
                <value>告白气球</value>
        </util:list>
        <bean id="songs" class="niceFood.Songs" p:list-ref="songList"/>
        <bean id="cd" class="niceFood.Hamburger" c:_0="#{songs.list[T(Math).random() * songs.list.size()]}"/>
            <!-- 可以实现随机选择一首歌曲 -->
        ```

      * 查询运算符 `.?[]`

        可以用来对集合进行过滤，比如，希望得到分数大于60分的学生集合

        `#{students.?[score >= 60]}`

        SpEL迭代学生列表时，会对每一个元素，计算`[]`内的表达式，计算为true，则会把该元素放进新的集合

        ```xml
        <util:list id="students">
                <bean class="com.Student" p:name="yogurt1" p:score="60"/>
                <bean class="com.Student" p:name="yogurt2" p:score="59"/>
                <bean class="com.Student" p:name="yogurt3" p:score="78"/>
                <bean class="com.Student" p:name="yogurt4" p:score="83"/>
                <bean class="com.Student" p:name="yogurt5" p:score="22"/>
            </util:list>
            <bean id="studentHolder" class="com.StudentHolder" p:students="#{students.?[score > 60]}"/>
        ```

        

        ```java
        public class Student {
            private String name;
            private Integer score;
            //省略了setter/getter
        }
        public class StudentHolder {
            private List<Student> students;
            //省略了setter/getter
        }
        ```

        ![1569674837524](C:\Users\Yogur\AppData\Roaming\Typora\typora-user-images\1569674837524.png)

      * `.^[]`  匹配第一个满足`[]`内表达式的元素

      * `.$[]` 匹配最后一个满足`[]`内表达式的元素

      * `.![]` 投影运算符，从集合中选取指定属性，放入另一集合

        `#{songs.![author]}` 取出 songs这个集合中每个元素的author字段，放入另一集合

