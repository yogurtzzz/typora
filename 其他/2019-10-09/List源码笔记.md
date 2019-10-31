List的遍历

* 针对实现了RandomAccess接口的List（支持随机访问的），如ArrayList，优先使用普通的for循环
* 针对未实现RandomAccess接口的List，如LinkedList，优先使用Iterator进行遍历（也可使用foreach，其底层也是通过Iterator），若对于链表，使用普通for循环，每次调用`get(i)`都会从头结点开始向后寻早，效率非常低

**Iterator和Enumeration**

二者都是用来实现遍历的，Enumeration已经被Iterator所取代了。Iterator与Enumeration在以下2点有所不同

- Iterator允许在迭代期间移除元素（主要区别就是Iterator可以删除元素，而Enumeration不行）
- Iterator的方法名称得到了改进

**Iterator和ListIterator**

`ListIterator`继承自`Iterator`

`Iterator`只能单向的遍历，它里面包含的方法有

`hasNext()`  , `next()`  , `remove()`

而`ListIterator`可以**双向遍历，并且可以获取元素的下标**，还可以修改或添加元素，`ListIterator`中包含的方法有

`hasNext()`  , `next()`  , `nextIndex()` ，`hasPrevious`，`previous()`，`previousIndex()`，`remove()`，`set()`，`add()`

其中`set()`和`add()`是针对了最近调用过的`next()`或`previous()`进行设置的，会设置当前`cursor`指向的地方

注意`ListIterator`没有当前元素，它的`cursor position` 总是位于2个元素之间，一个长度为`n`的list，其`ListIterator`拥有`n+1`个可能的`cursor position`

![1570777211490](C:\Users\Yogur\AppData\Roaming\Typora\typora-user-images\1570777211490.png)

Comparable 和 Comparator

* Comparable

  排序接口，某个类实现了这个接口，就意味着该类支持排序。实现了Comparable接口的类，可用作有序map(TreeMap) 和有序集合(TreeSet)

  ![1570853131802](C:\Users\Yogur\AppData\Roaming\Typora\typora-user-images\1570853131802.png)

  如此，调用Arrays.sort或Collections.sort即可实现对Person数组的排序，按照年龄从小到大

* Comparator

  比较器接口。若某个类本身没有实现Comparable接口，又需要对它进行排序，那么可以通过Comparator接口。我们创建一个Comparator接口的实现类。

  ![1570853461776](C:\Users\Yogur\AppData\Roaming\Typora\typora-user-images\1570853461776.png)

  当然，在jdk1.8后，可以替换为lambda表达式 

  `Collections.sort(list, (o1, o2) -> o1.getScore() - o2.getScore());`

  或者使用



**为什么hash算法要使用质数来做模？**

因为质数的因子只有1和他本身。当需要对一个数列使用模运算来执行hash时，将这个数列排序。观察其元素之间的间隔，若间隔为1，则模数为任意数，都是均匀分布；若间隔为模数本身，则全部冲突；总之，模数的因子越多，产生冲突的概率就越大

[参考链接](https://blog.csdn.net/w_y_x_y/article/details/82288178)

**关于String类的hashCode函数中，使用了31作为乘子的解释**

[参考链接](https://www.cnblogs.com/nullllun/p/8350178.html)

* 31是质数，是优质的hash乘子
* 31可以被JVM优化，`31 * i = (i << 5) - i`，移位运算效率非常高



**ArrayList源码**，尤其关注扩容机制

* 创建ArrayList对象
  * 使用` new ArrayList<>()` 创建，则`elementData`为DEFAULTCAPACITY_EMPTY_ELEMENTDATA
  * 使用` new ArrayList<>(0)` 创建，则`elementData`为EMPTY_ELEMENTDATA

* 不是线程安全的
  * 可以通过`Collections.synchronizedList()`方法来转变为线程安全的List，底层是通过一个`this`的对象锁，来完成的
  * 若对一个ArrayList调用`Collections.synchronizedList()`，将返回一个`SynchronizedRandomAccessList`，其继承自`SynchronizedList`

* 底层是用一个Object数组 elementData，被transient修饰，我们知道被transient修饰的变量是不会被序列化的。可以看ArrayList中的writeObject方法，它会依次从elementData中取值，并调用ObjectOutputStream的writeObject方法，这是因为elementData其实是一个缓存数组，它通常会预留一些容量，等容量不足时再扩充，如果直接用elementData来进行序列化，则数组中有些位置是空的。采用ArrayList中的writeObject来序列化，则只会序列化数组中实际存在的元素，而不是整个数组，从而节省时间和空间

* `Arrays.copyOf()`和 `System.arraycopy()`

  * `Arrays.copyOf(T[] src, int newLength,Class type)`

    会根据type创建一个数组，并调用`System.arraycopy`

  * `System.arraycopy(Object src,int srcPos,Object dest,int desPos,int length)`

    将src数组从srcPos位置开始，拷贝到dest数组，从destPos开始，拷贝length的长度

* **扩容机制**

  从`add(E e)`找起

  * `ensureCapacityInternal(size + 1)`
    * 创建ArrayList时，若使用无参构造函数，则第一次插入时，会扩容到`DEFAULT_CAPACITY` (10)；若指定了初始容量为0 （`new ArrayList<>(0)`） ，则第一次扩容时只会扩容到1，随后每次尝试扩容至当前容量的1.5倍
    * 调用`ensureExplicitCapacity(minCapacity)`
      * `modCount++`
      * 若`minCapacity > elementData.length`，即若需要的大小，超出了现有的大小，则调用`grow()`进行扩容
        * `grow(minCapacity)`
          * 先获取elementData的原有长度oldCapacity
          * 计算原有长度的1.5倍，作为newCapacity
          * 若newCapacity < minCapacity，即若1.5倍后还不满足需求，则令newCapacity = minCapacity
          * 判断 newCapacity是否超过MAX_ARRAY_SIZE，若超过，调用hugeCapacity(minCapacity)，结果赋值给newCapacity
          * 调用Arrays.copyOf(elementData,newCapacity) 进行扩容
  * `elementData[size++] = e`

Arrays.asList(T... a) 返回的是一个Arrays的内部类ArrayList，这个ArrayList是final的，不能调用修改的方法。若要返回一个可以修改的方法

将`int[]`转换为 `List<Integer>`，可以使用java8的流

```java
int[] intarr = {1,2,3,4,5,6};
List<Integer> list = Arrays.stream(intarr).boxed().collect(Collectors.toList());
```

Arrays.copyOf是浅拷贝 !  ArrayList的clone方法也一样是浅拷贝 !

![1570674172119](C:\Users\Yogur\AppData\Roaming\Typora\typora-user-images\1570674172119.png)

看的出，修改拷贝后数组的元素属性，源数组中的元素也发生相应的改变，拷贝的仅仅是引用，而两个数组里的元素，则仍然指向了相同的对象。若想实现深拷贝，则需要使用Object的clone方法

**LinkedList**源码	

是一个双向链表，包含了一个头结点（first）和一个尾结点（last）

* `node(int index)` 获取下标为index的Node，会采取优化，若index <   size >> 1 ，则会从头结点开始查找，否则从尾结点开始查找

* `listIterator(int index)` 返回一个从指定位置开始的`ListIterator`，下面是`LinkedList`内部对`ListIterator`的实现

  ![1570775170319](C:\Users\Yogur\AppData\Roaming\Typora\typora-user-images\1570775170319.png)