# JSP相关技术



## 如何在JSP页面调用java函数获取值

=> 使用自定义标签taglib，例如

先在/WebRoot/WEB-INF目录下新建一个叫做yogurt.tld的文件，其中的内容是

```xml
<?xml version="1.0" encoding="UTF-8"?>
<taglib version="2.0" xmlns="http://java.sun.com/xml/ns/j2ee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee web-jsptaglibrary_2_0.xsd">
	<tlib-version>1.0</tlib-version>
	<short-name>by</short-name>
	<uri>/yogurt/tag</uri> 
	
	<function>
		<name>sayHello</name>
		<function-class>com.yogurt.taglib.Test</function-class>
		<function-signature>
		java.lang.String sayHello()
		</function-signature>
	</function>
</taglib>
```



新建一个java文件

```java
package com.yogurt.taglib;
public class Test{
    public String sayHello(){
        return "Hello,World";
    }
}
```



在JSP里引入该自定义标签

```jsp
<%@ taglib prefix="by" uri="/yogurt/tag" %>
<c:set var="helloStr">${by:sayHello()}</c:set>

<input type="hidden" id="hello" name="hello" value="${helloStr}"/>

<!-- 
这样即可调用java代码，获取到返回值，并在页面中使用
-->
```

