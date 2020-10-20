# HashMap和Hashtable的区别

1.线程安全

Hashtable是线程安全，HashMap则非线程安全

Hashtable的实现方法里面都添加了synchronized关键字来确保线程同步，因此相对而言HashMap性能会高一些，我们平时使用时若无特殊需求建议使用HashMap，在多线程环境下若使用HashMap需要使用Collections.synchronizedMap()方法来获取一个线程安全的集合。

2.针对null的不同

HashMap可以使用null作为key，而Hashtable则不允许null作为key

3.继承结构

HashMap是对Map接口的实现，HashTable实现了Map接口和Dictionary抽象类。

4.初始容量与扩容

HashMap的初始容量为16，Hashtable初始容量为11，两者的填充因子默认都是0.75。

HashMap扩容时是当前容量翻倍即:capacity×2，Hashtable扩容时是容量翻倍+1即:capacity×2+1。

5.两者计算hash的方法不同

Hashtable计算hash是直接使用key的hashcode对table数组的长度直接进行取模

```
int hash = key.hashCode();
int index = (hash & 0x7FFFFFFF) % tab.length;
```

HashMap计算hash对key的hashcode进行了二次hash，以获得更好的散列值，然后对table数组长度取摸。

```
int hash = hash(key.hashCode());
int i = indexFor(hash, table.length);

static int hash(int h) {
        // This function ensures that hashCodes that differ only by
        // constant multiples at each bit position have a bounded
        // number of collisions (approximately 8 at default load factor).
        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }
 
 static int indexFor(int h, int length) {
        return h & (length-1);
```

