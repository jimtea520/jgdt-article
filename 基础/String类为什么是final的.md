# String类为什么是final的



# 声明为final类的目的

主要目的就是保证String是不可变（immutable）。不可变就是第二次给一个String 变量赋值的时候，不是在原内存地址上修改数据，而是重新指向一个新对象，新地址。下面看String类源码如何保证是不可变的：

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence
{
    /** The value is used for character storage. */
    private final char value[];
    
    ...
```

String类的主力成员字段value是个char[ ]数组，而且是用final修饰的。编译器不允许把value指向另一个地址。但可以直接对数组元素修改。为了保证这个数组元素不能修改，做了如下措施：
 （1）所有String的方法里很小心的没有去动数组里的元素，没有暴露内部成员字段。
 （2）避免被其他人继承后破坏，整个String设成final禁止继承。如果有一个String的引用，它引用的一定是一个String对象，而不可能是其他类的对象。

# 为什么要不String设计为不可变

- 从内存角度来看
   字符串常量池的要求：创建字符串时，如果该字符串已经存在于池中，则将返回现有字符串的引用，而不是创建新对象。字符串池的实现可以在运行时节约很多heap空间，多个String变量引用指向同一个内地地址。如果字符串是可变的，用一个引用更改字符串将导致其他引用的值错误。这是很危险的。

- 缓存Hashcode
   字符串的Hashcode在java中经常配合基于散列的集合一起正常运行，这样的散列集合包括HashSet、HashMap以及HashTable。不可变的特性保证了hashcode永远是相同的。不用每次使用hashcode就需要计算hashcode。这样更有效率。因为当向集合中插入对象时，是通过hashcode判别在集合中是否已经存在该对象了（不是通过equals方法逐个比较，效率低）。

- 方便其它类使用
   其他类的设计基于string不可变，如set存储string，改变该string后set包含了重复值。

- 安全性
   String被广泛用作许多java类的参数，例如网络连接、打开文件等。如果对string的某一处改变一不小心就影响了该变量所有引用的表现，则连接或文件将被更改，这可能导致严重的安全威胁。
   不可变对象不能被写，所以不可变对象自然是线程安全的，因为不可变对象不能更改，它们可以在多个线程之间自由共享。

  

# 总结 

由于效率和安全性的原因，字符串被设计为不可变





