# String、StringBuffer和StringBuilder的区别和使用场景



### java中用于处理字符串常用的有三个类:

### 1、java.lang.String

### 2、java.lang.StringBuffer

### 3、java.lang.StrungBuilder



## 相同点

都是final类,不允许被继承



## 不同点

### **可变性不同**

​	1. String类中使用 final 关键字修饰字符数组来保存字符串

​			private final char value[]

​		所以String对象是不可变的。在jdk9之后，改用byte数组来存取

​			private final byte[] value

​    2.StringBuilder和 StringBuffer都继承自AbstractStringBuilder类，在AbstractStringBuilder中也是使用字符数组保存字符串，但是没有用 final关键字修饰，所以这两种对象都是可变的。

### 实现接口不同

String实现了三个接口:Serializable、Comparable<String>、CarSequence。

StringBuilder只实现了两个接口Serializable、CharSequence，相比之下String的实例可以通过compareTo方法进行比较，其他两个不可以。

### **线程安全性不同**

String中的对象是不可变的，也就可以理解为常量，线程安全。

StringBuffer对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。

StringBuilder并没有对方法进行加同步锁，所以是非线程安全的。

## **性能不同**

由于String是不可变的，每次对 String类型进行改变的时候，都会生成一个新的String对象，然后将指针指向新的String对象，性能是最低下的。

StringBuffer和StringBuilder每次都会对对象本身进行操作，而不是生成新的对象并改变对象引用。

相同情况下使用StringBuilder相比使用StringBuffer仅能获得 10%~15% 左右的性能提升，但却要冒多线程不安全的风险。

所以运行速度是 StringBuilder > StringBuffer > String 。

## 使用场景

**String：适用于少量的字符串操作的情况。**

**StringBuilder：适用于单线程下在字符缓冲区进行大量操作的情况。**

**StringBuffer：适用多线程下在字符缓冲区进行大量操作的情况。**