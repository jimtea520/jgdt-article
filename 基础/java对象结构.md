

# 概念

在HotSpot虚拟机中，对象在内存中存储的布局可以分为3块区域：对象头（Header）、实例数据（Instance Data）和对齐填充（Padding）。

具体如下图

java 普通对象结构



java 数组对象结构



# 对象结构组成

### 对象头

HotSpot虚拟机的对象头包括两部分信息：

1. Mark Word 
   第一部分Mark Word,用于存储对象自身的运行时数据，如哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等，这部分数据的长度在32位和64位的虚拟机（未开启压缩指针）中分别为32bit和64bit
2. 类型指针 
   对象头的另外一部分是类型指针，即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例.
3. 数组长度（只有数组对象有） 
   如果对象是一个数组, 那在对象头中还必须有一块数据用于记录数组长度.

### 实例数据

​    实例数据部分是对象真正存储的有效信息，也是在程序代码中所定义的各种类型的字段内容。无论是从父类继承下来的，还是在子类中定义的，都需要记录起来。

对象引用（reference）类型在64位机器上，关闭指针压缩时占用8bytes， 开启时占用4bytes。

原生类型（primitive type）的内存占用如下：

| Primitive Type | Memory Required(bytes) |
| :------------- | :--------------------- |
| byte, boolean  | 1 byte                 |
| short, char    | 2 bytes                |
| int, float     | 4 bytes                |
| long, double   | 8 bytes                |

### **包装类型**

包装类（Boolean/Byte/Short/Character/Integer/Long/Double/Float）占用内存的大小等于对象头大小加上底层基础数据类型的大小。

包装类型的对象内存占用情况如下：

| Numberic Wrappers | +useCompressedOops | -useCompressedOops |
| :---------------- | :----------------- | :----------------- |
| Byte, Boolean     | 16 bytes           | 24 bytes           |
| Short, Character  | 16 bytes           | 24 bytes           |
| Integer, Float    | 16 bytes           | 24 bytes           |
| Long, Double      | 24 bytes           | 24 bytes           |

### 对齐填充

​    第三部分对齐填充并不是必然存在的，也没有特别的含义，它仅仅起着占位符的作用。由于HotSpot VM的自动内存管理系统要求对象起始地址必须是8字节的整数倍，换句话说，就是对象的大小必须是8字节的整数倍。而对象头部分正好是8字节的倍数（1倍或者2倍），因此，当对象实例数据部分没有对齐时，就需要通过对齐填充来补全。

HotSpot的对齐方式为8字节对齐：

（对象头 + 实例数据 + padding） % 8=0且0 <= padding < 8

# jvm相关参数

上面用到的useCompressedOops这个参数，我们可以看看在命令行输入：java -XX:+PrintCommandLineFlags -version 查看jvm默认参数如图：

![jvm参数](E:\技术帖子\笔记\基础\图\java对象结构\jvm参数.png)

分别是 -XX:+UseCompressedOops 和 -XX:+UseCompressedClassPointers
这2个参数都是默认开启（+代表开启，-代表关闭）

UseCompressedOops：普通对象指针压缩（oop是ordinary object pointer的缩写),
UseCompressedClassPointers：类型指针压缩。

例如:

Object o = new Object();
o指向new Object()的引用就是“普通对象指针”，
new Object()自身还需要指向Object类型的引用，也就是"类型指针"。

这2个压缩参数可以有4种组合(++, --, + -, -+)，但有1种组合是会抛出警告的：

![img](https://img-blog.csdnimg.cn/20200408161813162.png)

-XX:+UseCompressedClassPointers -XX:-UseCompressedOops，不要使用这种参数组合，用这种参数启动jvm时会抛出警告。

原因是jvm层面的hotspot源码对jvm的参数组合做了限制，一看就懂：


![在这里插入图片描述](https://img-blog.csdnimg.cn/2020040816190118.png)

### HotSpot对象模型

HotSpot中采用了OOP-Klass模型，它是描述Java对象实例的模型，它分为两部分：

- 类被加载到内存时，就被封装成了klass，klass包含类的元数据信息，像类的方法、常量池这些信息都是存在klass里的，你可以认为它是java里面的java.lang.Class对象，记录了类的全部信息；

- OOP（Ordinary Object Pointer）指的是普通对象指针，它包含MarkWord 和元数据指针，MarkWord用来存储当前指针指向的对象运行时的一些状态数据；元数据指针则指向klass,用来告诉你当前指针指向的对象是什么类型，也就是使用哪个类来创建出来的；


  那么为何要设计这样一个一分为二的对象模型呢？这是因为HotSopt JVM的设计者不想让每个对象中都含有一个vtable（虚函数表），所以就把对象模型拆成klass和oop，其中oop中不含有任何虚函数，而klass就含有虚函数表，可以进行method dispatch。

HotSpot中，OOP-Klass实现的代码都在/hotspot/src/share/vm/oops/路径下，oop的实现为instanceOop 和 arrayOop，他们来描述对象头，其中arrayOop对象用于描述数组类型。

以下就是oop.hhp文件中oopDesc的源码，可以看到两个变量_mark就是MarkWord，_metadata就是元数据指针，指向klass对象，这个指针压缩的是32位，未压缩的是64位；

```
volatile markOop _mark;  //标识运行时数据
  union _metadata {
    Klass*      _klass;
    narrowKlass _compressed_klass;
  } _metadata;  //klass指针
```

一个Java对象在内存中的布局可以连续分成两部分：instanceOop（继承自oop.hpp）和实例数据；

如图:

![java对象布局](E:\技术帖子\笔记\基础\图\java对象结构\java对象布局.png)

通过栈帧中的对象引用reference找到Java堆中的对象，再通过对象的instanceOop中的元数据指针klass来找到方法区中的instanceKlass，从而确定该对象的类型。

# 对象大小的计算

有以下几点:

1.在32位系统下，存放Class指针的空间大小是4字节，MarkWord是4字节，对象头为8字节。

2.在64位系统下，存放Class指针的空间大小是8字节，MarkWord是8字节，对象头为16字节。

3.64位开启指针压缩的情况下，存放Class指针的空间大小是4字节，MarkWord是8字节，对象头为12字节。

4.数组长度4字节+数组对象头8字节(对象引用4字节（未开启指针压缩的64位为8字节）+数组markword为4字节（64位未开启指针压缩的为8字节）)+对齐4=16字节。

5.静态属性不算在对象大小内。

贴网上的一个比较实用的工具类:

```
import java.lang.instrument.Instrumentation;  
import java.lang.reflect.Array;  
import java.lang.reflect.Field;  
import java.lang.reflect.Modifier;  
import java.util.ArrayDeque;  
import java.util.Deque;  
import java.util.HashSet;  
import java.util.Set;  

/** 

​	*对象占用字节大小工具类 

​    **/  
public class SizeOfObject {  
static Instrumentation inst;  

public static void premain(String args, Instrumentation instP) {  
    inst = instP;  
}  

/** 

 * 直接计算当前对象占用空间大小，包括当前类及超类的基本类型实例字段大小、<br></br> 

*引用类型实例字段引用大小、实例基本类型数组总占用空间、实例引用类型数组引用本身占用空间大小;<br></br> 

*但是不包括超类继承下来的和当前类声明的实例引用字段的对象本身的大小、实例引用数组引用的对象本身的大小 <br></br> 

*

*@param obj 

*@return 
*/  
public static long sizeOf(Object obj) {  
return inst.getObjectSize(obj);  
}  

/** 

*递归计算当前对象占用空间总大小，包括当前类和超类的实例字段大小以及实例字段引用对象大小 

*

*@param objP 

*@return 

*@throws IllegalAccessException 
*/  
public static long fullSizeOf(Object objP) throws IllegalAccessException {  
Set<Object> visited = new HashSet<Object>();  
Deque<Object> toBeQueue = new ArrayDeque<Object>();  
toBeQueue.add(objP);  
long size = 0L;  
while (toBeQueue.size() > 0) {  
    Object obj = toBeQueue.poll();  
    //sizeOf的时候已经计基本类型和引用的长度，包括数组  
    size += skipObject(visited, obj) ? 0L : sizeOf(obj);  
    Class<?> tmpObjClass = obj.getClass();  
    if (tmpObjClass.isArray()) {  
        //[I , [F 基本类型名字长度是2  
        if (tmpObjClass.getName().length() > 2) {  
            for (int i = 0, len = Array.getLength(obj); i < len; i++) {  
                Object tmp = Array.get(obj, i);  
                if (tmp != null) {  
                    //非基本类型需要深度遍历其对象  
                    toBeQueue.add(Array.get(obj, i));  
                }  
            }  
        }  
    } else {  
        while (tmpObjClass != null) {  
            Field[] fields = tmpObjClass.getDeclaredFields();  
            for (Field field : fields) {  
                if (Modifier.isStatic(field.getModifiers())   //静态不计  
                        || field.getType().isPrimitive()) {    //基本类型不重复计  
                    continue;  
                }  

​            field.setAccessible(true);  
​            Object fieldValue = field.get(obj);  
​            if (fieldValue == null) {  
​                continue;  
​            }  
​            toBeQueue.add(fieldValue);  
​        }  
​        tmpObjClass = tmpObjClass.getSuperclass();  
​    }  
}  

}  
return size;  
}  

/** 

   * String.intern的对象不计；计算过的不计，也避免死循环 

*

*@param visited 

*@param obj 

*@return 
*/  
static boolean skipObject(Set<Object> visited, Object obj) {  
if (obj instanceof String && obj == ((String) obj).intern()) {  
    return true;  
}  
return visited.contains(obj);  
}  
}
```

下面举三个例子：

首先需要创建一个mavean项目，引入包

```
<dependency>
  <groupId>org.openjdk.jol</groupId>
  <artifactId>jol-core</artifactId>
  <version>0.9</version>
</dependency>
```

1.需要补齐的对象

代码

```
public class User {
    long sex;
    Long mobile;
    String name;

    public static void main(String[] args) {
        System.out.println(ClassLayout.parseInstance(new User()).toPrintable());
    }
}
```

输出如图

2.不需要padding补齐的对象

代码：

```
public class User {

    String name;
    Long mobile;
    int sex;

    public static void main(String[] args) {
        System.out.println(ClassLayout.parseInstance(new User()).toPrintable());
    }
}
```

输出如图

3.空对象，所占字节数

代码：

```
public class User {

    public static void main(String[] args) {
        System.out.println(ClassLayout.parseInstance(new User()).toPrintable());
    }
}
```

输出如图

4.数组对象结构

代码：

```
public class ArrayTest {

    public static void main(String[] args) {
        System.out.println(ClassLayout.parseInstance(new Integer[7]).toPrintable());
        System.out.println(ClassLayout.parseInstance(new Integer[8]).toPrintable());
        System.out.println(ClassLayout.parseInstance(new int[7]).toPrintable());
    }
}
```

输出如图

![数组对象](E:\技术帖子\笔记\基础\图\java对象结构\数组对象.jpg)