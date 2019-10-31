spring使用构造器注入时，可能会造成循环依赖

比如，A通过构造器注入，注入它的依赖B，B通过构造器注入，注入他的依赖A。

spring在运行时会检测到这个循环依赖，并抛出BeanCurrentlyInCreationException





解决方案是：修改为setter注入，而不使用构造器注入。此时如果发生循环依赖，那么A和B必然有其一会先注入到另一个bean里去（尽管此时它并没有完成初始化）