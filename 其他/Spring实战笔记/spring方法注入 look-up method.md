当遇到这样一种情况，一个单例的bean  A，依赖于另一个非单例的bean  B

我们希望A中每次使用B，都是使用的一个全新的B

然而通过简单的DI是不能完成这样的效果的

因为单例A只会实例化一次，也就只有一次DI的机会，所以容器就无法每次给A提供一个新的B实例。

要解决这个问题，有2种方法，

+ 可以对A做一些修改，让它实现ApplicationContextAware

  每次使用B时，通过ApplicationContext去IOC容器中取 （这样取到的就是最新的）

  但是这样就和spring强耦合了，所以推荐第二种方法

+ 使用方法注入

  + 使用注解

    ```java
    @Component
    public abstract class FruitPlate {
        //在单例bean中注入非单例的bean
        //若希望每次使用都是新的bean
        //使用lookup  方法注入
        private Apple apple;
    
        private Pear pear;
    
        public void serveFruits(){
            apple = getApple();
            pear = getPear();
            System.out.println("请你吃苹果" + apple);
            System.out.println("请你吃梨" + pear);
        }
    
        @Lookup
        public abstract Apple getApple();
    
        @Lookup
        public abstract Pear getPear();
    }
    ```

    

