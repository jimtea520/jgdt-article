### List扩容实现步骤

总的来说就是分两步：

1、扩容

​		把原来的数组复制到另一个内存空间更大的数组中

2、添加元素

​		把新元素添加到扩容以后的数组中

### 性能分析

ArrayList作为动态数组，其内部元素以数组形式顺序存储的，所以非常适合随机访问的场合。除了尾部插入和删除元素，往往性能会相对较差，比如我们在中间位置插入一个元素，需要移动后续所有元素。

### 源码分析

  先把ArrayList中定义的一些属性贴出来方便下面源码分析



### ArrayList的两个构造方法

1.ArrayList()
2.ArrayList(int initialCapacity)

```
无参构造：
public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
带参构造：
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
```



在无参构造中，创建了一个空数组，长度为0

在有参构造中，传入的参数是正整数就按照传入的参数来确定创建数组的大小，否则异常



### 扩容的方法 

### 插入元素函数 (boolean add(E e))

ArrayList在执行插入元素是超过当前数组预定义的最大值时，数组需要扩容，扩容过程需要调用底层System.arraycopy()方法进行大量的数组复制操作。

   贴上源码

```
public boolean add(E e) { 

	ensureCapacityInternal(size + 1); // Increments modCount!! 

	elementData[size++] = e; 

	return true;

}
```

看，其实add方法就两步，

第一步：增加长度

第二步：添加元素到数组

[![image](E:\技术帖子\笔记\基础\图\arraylist扩容\2.png)](https://img2018.cnblogs.com/blog/1470032/201810/1470032-20181024160424468-1385941258.png)

ensureCapacityInternal(int minCapacity)这个增加长度的方法

 ![img](E:\技术帖子\笔记\基础\图\arraylist扩容\31.png)

这个地方我们看到了，如果在添加的时候远数组是空的，就直接给一个10的长度，否则的话就加一

ensureExplicitCapacity(int minCapacity)

```
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
```

通过这个地方是真正的增加长度，当需要的长度大于原来数组长度的时候就需要扩容了，相反的则不需要扩容

[![image](E:\技术帖子\笔记\基础\图\arraylist扩容\4.png)](https://img2018.cnblogs.com/blog/1470032/201810/1470032-20181024160425322-2009676905.png)

这个地方注意

   int newCapacity = oldCapacity + (oldCapacity >> 1);

  oldCapacity >> 1  右移运算符  原来长度的一半 再加上原长度也就是每次扩容是原来的1.5倍

之前的所有都是确定新数组的长度，确定之后就是把老数组copy到新数组中，这样数组的扩容就结束了

以上的一切都是ArrayList扩容的第一步，第二步就没啥说的了，就是把需要添加的元素添加到数组的最后一位

**ArrayList安全性**

非线程安全

![img](E:\技术帖子\笔记\基础\图\arraylist扩容\5.png)

1.在 add 的扩容的时候会有线程安全问题， ensureCapacityInternal(int minCapacity)这个步骤是有线程安全问题

2.在add  的elementData[size++] = e 这段代码在多线程的时候同样会有线程安全问题，

这里可以分成两个步骤：

elementData[size] = e;

size = size + 1;



