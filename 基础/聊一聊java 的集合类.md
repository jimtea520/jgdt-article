聊一聊java 的集合类

# 概述

Java中集合分为两种类型
第一种：以单个元素存储。其超级父接口是：java.util.Collection;
第二种：以键值对存储。（类似于python的集合）其超级父接口是：java.util.Map;

前者每个位置只能保存一个元素，后者可以保存两个元素。

## 分类

Collection又可分为List、Set、Queue
List下常用的有ArrayList、LinkedList、Vector、Stack
Set下常用的有HashSet、TreeSet
Queue又有Deque、Stack、LinkedList



collection 如下图

![collection](E:\技术帖子\笔记\基础\图\collection.png)

# SET

​    无序不可重复,没有下标。规定Set的实例不包含重复的元素。在一个规则集内，一定不存在两个相等的元素 。

​	实现类都不是线程安全的类，解决方案：Set set = Collections.sysnchronizedSet(Set对象);

## HashSet

​		底层是HashMap,放到HashSet集合中的元素等同于当道HashMap中的key部分。

​		是一个用于实现Set接口的具体类，可以使用它的无参构造方法来创建空的散列集，也可以由一个现有的集合创建散列集。在散列集中，有两个名词需要关注，初始容量和客座率。客座率是确定在增加规则集之前，该规则集的饱满程度，当元素个数超过了容量与客座率的乘积时，容量就会自动翻倍。

## SortedSet

无序不可重复的。但是SortedSet集合中的元素是可排序的。

无序：存进去的顺序和去除的顺序不一定相同，另外集合元素没有下标，不可重复。
可排序：可以按照大小顺序排序。

## TreeSet

​    实现了 SortedSet接口, 底层为红黑二叉树，实际上就是TreeMap,放到TreeSet集合中的元素等同于当道TreeMap中的key部分。不允许为null，不能重复，有序存储（顺序可以自定义）//存储空会报错

TreeSet的有序存储，存储元素时会判断他是否重复，并且自动排序，判断重复元素由compareTo()方法来实现。因此自定义类要使用TreeSet必须覆写Comparable接口, 如下示例

```
public class Demo1 {
    public static void main(String[] args) {

        Set<Person> list = new HashSet<>();
        Person per1 = new Person("p0", "s1", 21);
        list.add(per1);
        list.add(per1);
        list.add(new Person("p1", "s1", 22));
        list.add(new Person("p2", "s1", 23));
        for (Person person : list) {
            System.out.println(person);
        }
    }
}

class Person implements Comparable<Person> {
    public String name;
    public String school;
    private int age;

    public Person(String name, String school, int age) {
        this.name = name;
        this.school = school;
        this.age = age;
    }

    @Override
    public String toString() {
        return "[name: " + this.name + "    school: " + this.school + "    age: " + age + "]";
    }

    /*
     * 复写 .compareTo() 的规定： 当前对象大于传入对象，返回一个正数 当前对象等于传入对象，返回一个0 当前对象小于传入对象，返回一个负数
     */
    @Override
    public int compareTo(Person o) {
        if (this.age > o.age) {
            return 1;
        } else if (this.age < o.age) {
            return -1;
        } else {
            int i = this.name.compareTo(o.name);
            if (i != 0) {
                return i;
            }
            return this.school.compareTo(school);
        }
    }
}
```



## LinkedHashSet

​		底层是HashMap,LinkedHashSet是用一个链表实现来扩展HashSet类，它支持对规则集内的元素排序。HashSet中的元素是没有被排序的，而LinkedHashSet中的元素可以按照它们插入规则集的顺序提取。



## Set示例

```
public class Demo {
    public static void main(String[] args) {

        //HashSet
        Set<String> hs = new HashSet<>();
        hs.add("a");
        hs.add("g");
        hs.add("c");
        hs.add("d");
        System.out.println("HashSet:");
        for(String cc : hs){
            System.out.print(cc+",");
        }

        System.out.println();

        //TreeSet
        Set<String> ts = new TreeSet<>();
        ts.add("a");
        ts.add("g");
        ts.add("c");
        ts.add("d");
        System.out.println("TreeSet:");
        for(String cc : ts){
            System.out.print(cc+",");
        }

        System.out.println();

        //LinkedHashSet
        Set<String> linkedHashSets = new LinkedHashSet<>();
        linkedHashSets.add("a");
        linkedHashSets.add("g");
        linkedHashSets.add("c");
        linkedHashSets.add("d");
        System.out.println("LinkedHashSet:");
        for(String cc : linkedHashSets){
            System.out.print(cc+",");
        }

    }
}
```

# List 

有下标。

可以重复，通过索引取出加入数据，顺序与插入顺序一致，可以含有null元素,

长度可变，元素存放有一定的顺序，下标从0开始。

## ArrayList

底层数据结构使数组结构array，查询速度快，增删改慢，因为是一种类似数组的形式进行存储，因此它的随机访问速度极快；线程不安全，每次扩容是原来长度的1.5倍，默认容量为10, 	jdk源码如下

```
public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable{

private static final int DEFAULT_CAPACITY = 10;

}
```

初始化示例

```
//如何使用ArrayList创建一个List
//不指定存放的元素类型

//默认容量为10
List list = new ArrayList();

//容量为6
List list = new ArrayList(6);
```

扩容源代码：

![arraylist扩容](E:\技术帖子\笔记\基础\图\arraylist扩容.png)



## Vector

- Vector集合底层是一个数组。
- 初始化容量也为10，扩容之后是原来容量的两倍，而ArrayList集合是原来的1.5倍。
- Vector中所有的方法都是线程同步的，都带有synchronized关键字，是线程安全的。（效率较低，使用较少）
  那如何将非线程安全的转换成线程安全的呢？
- 使用集合工具类。
  1.java.util.Collections;
  2.注意java.util.Collection 是集合接口。
  但是：java.util.Collection是集合工具类。

示例

```
import java.util.*;

public class VectorDemo {

   public static void main(String args[]) {
      // initial size is 3, increment is 2
      Vector v = new Vector(3, 2);
      System.out.println("Initial size: " + v.size());
      System.out.println("Initial capacity: " +
      v.capacity());
      v.addElement(new Integer(1));
      v.addElement(new Integer(2));
      v.addElement(new Integer(3));
      v.addElement(new Integer(4));
      System.out.println("Capacity after four additions: " +
          v.capacity());

      v.addElement(new Double(5.45));
      System.out.println("Current capacity: " +
      v.capacity());
      v.addElement(new Double(6.08));
      v.addElement(new Integer(7));
      System.out.println("Current capacity: " +
      v.capacity());
      v.addElement(new Float(9.4));
      v.addElement(new Integer(10));
      System.out.println("Current capacity: " +
      v.capacity());
      v.addElement(new Integer(11));
      v.addElement(new Integer(12));
      System.out.println("First element: " +
         (Integer)v.firstElement());
      System.out.println("Last element: " +
         (Integer)v.lastElement());
      if(v.contains(new Integer(3)))
         System.out.println("Vector contains 3.");
      // enumerate the elements in the vector.
      Enumeration vEnum = v.elements();
      System.out.println("\nElements in vector:");
      while(vEnum.hasMoreElements())
         System.out.print(vEnum.nextElement() + " ");
      System.out.println();
   }
}
```

输出

```
Initial size: 0
Initial capacity: 3
Capacity after four additions: 5
Current capacity: 5
Current capacity: 7
Current capacity: 9
First element: 1
Last element: 12
Vector contains 3.

Elements in vector:
1 2 3 4 5.45 6.08 7 9.4 10 11 12
```

线程安全

```java
        List myList = new ArrayList();
//        变成线程安全的
        Collections.synchronizedList(myList);
        myList.add("444");
        System.out.println(myList.get(0));
```

## Stack 

继承Vector

执行push时(即，将元素推入栈中)，是通过将元素追加的数组的末尾中。
执行peek时(即，取出栈顶元素，不执行删除)，是返回数组末尾的元素。
执行pop时(即，取出栈顶元素，并将该元素从栈中删除)，是取出数组末尾的元素，然后将该元素从数组中删除

```
public class StackDemo {
    static void showpush(Stack<Integer> st, int a) {
        st.push(new Integer(a));
        System.out.println("push(" + a + ")");
        System.out.println("stack: " + st);
    }

    static void showpop(Stack<Integer> st) {
        System.out.print("pop -> ");
        Integer a = (Integer) st.pop();
        System.out.println(a);
        System.out.println("stack: " + st);
    }

    public static void main(String args[]) {
        Stack<Integer> st = new Stack<Integer>();
        System.out.println("stack: " + st);
        showpush(st, 42);
        showpush(st, 66);
        showpush(st, 99);
        showpop(st);
        showpop(st);
        showpop(st);
        try {
            showpop(st);
        } catch (EmptyStackException e) {
            System.out.println("empty stack");
        }
    }
}
```

输出结果

```
stack: []
push(42)
stack: [42]
push(66)
stack: [42, 66]
push(99)
stack: [42, 66, 99]
pop -> 99
stack: [42, 66]
pop -> 66
stack: [42]
pop -> 42
stack: []
pop -> empty stack

Process finished with exit code 0
```

## LinkedList

底层用双链表实现，与前两个相比，优点是便于增删元素，缺点是单链表的存储地址不连续，插中间一个节点，需要从头节点开始查找，知道其上一个节点存储的地址才可，访问元素性能效率相对低。

（数组连续内存空间，查找速度快，增删慢；链表充分利用了内存，存储空间是不连续的，首尾存储上下一个节点的信息，所以寻址麻烦，查找速度慢，但是增删快。）

示例

```
LinkedList<String> lList = new LinkedList<String>();
lList.add("1");
lList.add("2");
lList.add("3");
lList.add("4");
lList.add("5");
System.out.println(lList);
lList.addFirst("0");
System.out.println(lList);
lList.addLast("6");
System.out.println(lList);
```

输出

```
[1, 2, 3, 4, 5]
[0, 1, 2, 3, 4, 5]
[0, 1, 2, 3, 4, 5, 6]
```

# Queue 

先进先出

自从Java 1.5之后，在java.util.concurrent包下提供了若干个阻塞队列，主要有以下几个：

## ArrayBlockingQueue

基于数组实现的一个阻塞队列，在创建ArrayBlockingQueue对象时必须制定容量大小。并且可以指定公平性与非公平性，默认情况下为非公平的，即不保证等待时间最长的队列最优先能够访问队列。

## LinkedBlockingQueue

基于链表实现的一个阻塞队列，在创建LinkedBlockingQueue对象时如果不指定容量大小，则默认大小为Integer.MAX_VALUE。

## PriorityBlockingQueue

以上2种队列都是先进先出队列，而PriorityBlockingQueue却不是，它会按照元素的优先级对元素进行排序，按照优先级顺序出队，每次出队的元素都是优先级最高的元素。注意，此阻塞队列为无界阻塞队列，即容量没有上限（通过源码就可以知道，它没有容器满的信号标志），前面2种都是有界队列。

## DelayQueue

基于PriorityQueue，一种延时阻塞队列，DelayQueue中的元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素。DelayQueue也是一个无界队列，因此往队列中插入数据的操作（生产者）永远不会被阻塞，而只有获取数据的操作（消费者）才会被阻塞。


# Map

含健值对，每个里面又可以再细分一些类可以排序，或支持并发

![map](E:\技术帖子\笔记\基础\图\map.png)

## Hashtable

　　Hashtable继承Map接口，实现一个key-value映射的哈希表。任何非空（non-null）的对象都可作为key或者value。
　　添加数据使用put(key, value)，取出数据使用get(key)，这两个基本操作的时间开销为常数。
Hashtable通过initial capacity和load factor两个参数调整性能。通常缺省的load factor 0.75较好地实现了时间和空间的均衡。增大load factor可以节省空间但相应的查找时间将增大，这会影响像get和put这样的操作。



## HashMap

HashMap是基于哈希表的Map接口的非同步实现，继承自AbstractMap，AbstractMap是部分实现Map接口的抽象类。

在之前版本HashMap采用数组+链表实现，即使用链表处理冲突，同一hash值的链表都存储在一个链表里。但是当链表中的元素较多，即hash值相等的元素较多时，通过key值依次查找的效率较低。而JDK1.8中，HashMap采用数组+链表+红黑树实现，当链表长度超过阈值（8）时，将链表转换为红黑树，这样大大减少了查找时间。

在HashMap中要找到某个元素，需要根据key的hash值来求得对应数组中的位置。对于任意给定的对象，只要它的hashCode()返回值相同，那么程序调用hash(int h)方法所计算得到的hash码值总是相同的。我们首先想到的就是把hash值对数组长度取模运算，这样一来，元素的分布相对来说是比较均匀的。但是，“模”运算的消耗还是比较大的，在HashMap中，(n - 1) & hash用于计算对象应该保存在table数组的哪个索引处。HashMap底层数组的长度总是2的n次方，当数组长度为2的n次幂的时候，(n - 1) & hash 算得的index相同的几率较小，数据在数组上分布就比较均匀，也就是说碰撞的几率小，相对的，查询的时候就不用遍历某个位置上的链表，这样查询效率也就较高了。

## LinkedHashMap

LinkedHashMap继承自HashMap，它主要是用链表实现来扩展HashMap类，HashMap中条目是没有顺序的，但是在LinkedHashMap中元素既可以按照它们插入的顺序排序，也可以按它们最后一次被访问的顺序排序。

## TreeMap

TreeMap基于红黑树数据结构的实现，键值可以使用Comparable或Comparator接口来排序。TreeMap继承自AbstractMap，同时实现了接口NavigableMap，而接口NavigableMap则继承自SortedMap。SortedMap是Map的子接口，使用它可以确保图中的条目是排好序的。

在实际使用中，如果更新图时不需要保持图中元素的顺序，就使用HashMap，如果需要保持图中元素的插入顺序或者访问顺序，就使用LinkedHashMap，如果需要使图按照键值排序，就使用TreeMap。

## ConcurrentHashMap

Concurrent，并发，ConcurrentHashMap是HashMap的线程安全版。同HashMap相比，ConcurrentHashMap不仅保证了访问的线程安全性，而且在效率上与HashTable相比，也有较大的提高。
