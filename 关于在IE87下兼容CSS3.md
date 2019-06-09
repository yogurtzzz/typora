# 关于在IE8/7下兼容CSS3

## 使用PIE.htc

* 圆角border-radius
* 盒阴影box-shadow
* 渐变gradient

引入`PIE.htc`文件

然后在css样式中添加 `behavior:url(PIE.htc);`即可

需要注意的问题:

1. `PIE.htc`文件存放的位置要相对于HTML文件，而不是CSS文件

   如：在`test.css`中设置某元素为圆角，写上`behavior:url(PIE.htc)`，然后在`test.html`或`test.jsp`中调用`test.css`，那么需要将`PIE.htc`文件放在与`test.html`/`test.jsp`文件同级的目录下

   不起作用时，尝试把`PIE.htc`放在html文件所在的上一级目录，然后在css中写`behavior:url(../PIE.htc);`

2. 使用`PIE.htc`实现css3渲染效果，属性值只能写缩写，如

   `border-radius:5px;`而不支持如

   `border-top-left-radius:5px;`的写法

3. 若目前的元素`position`属性为`static`，即默认属性，则

   `z-index`属性无效，渲染不成功。解决方法:设置目标元素`position:relative;`,或者，设置祖先元素`position:relative`并赋予一个`z-index`值（不能为-1）



## 使用IE滤镜

使用滤镜可实现**旋转**,**透明**等效果

1. *旋转*

   ```css
   .demo{
       transform:rotate(45deg);  /*CSS3写法*/
     filter:progid:DXImageTransform.Microsoft.Matrix(M11=0.707,M12=-0.707,M21=0.707,M22=0.707,SizingMethod='auto expand'); /*兼容IE8*/
   }
   ```

   滤镜公式中，`M11=`$cos\theta$ ,`M12=`$-sin\theta$,`M21=`$sin\theta$,`M22=`$cos\theta$

2. 透明

   ```css
   .demo{
       background:none;
       /*下面代码兼容IE6下背景不透明问题*/
       _background:none;     _filter:progid:DXImageTransform.Microsoft.AlphaImageLoader(src="../images1024/tg.png");
   }
   ```

   

