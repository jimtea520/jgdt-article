# HashTable原理以及源码解析(通俗易懂)

## UML图



## 概念

HashTable也是一个散列表，它存储的内容是键值对映射。HashTable继承于Dictionary，实现了Map、Cloneable、java.io.Serializable接口。HashTable的函数都是同步的，这意味着它是线程安全的。它的Key、Value都不可以为null。此外，HashTable中的映射不是有序的。

HashTable的实例有两个参数影响其性能：初始容量和加载因子。容量是哈希表中桶的数量，初始容量就是哈希表创建时的容量。注意，哈希表的状态为open：在发生“哈希冲突”的情况下，单个桶会存储多个条目，这些条目必须按顺序搜索。加载因子是对哈希表在其容量自动增加之前可以达到多满的一个尺度。初始容量和加载因子这两个参数只是对该实现的提示。关于何时以及是否调用rehash方法的具体细节则依赖于该实现。通常，默认加载因子是0.75。

## 源码分析

### 比较重要的参数

```
private transient Entry<?,?>[] table;

private transient int count;

private int threshold;

private float loadFactor;

private transient int modCount = 0;
```

#### table

   为一个Entry[]数组类型，Entry代表了“拉链”的节点，每一个Entry代表了一个键值对，哈希表的"key-value键值对"都是存储在Entry数组中的。

#### count

   HashTable的大小，注意这个大小并不是HashTable的容器大小，而是他所包含Entry键值对的数量。

#### threshold

   Hashtable的阈值，用于判断是否需要调整Hashtable的容量。threshold的值="容量*加载因子"。

#### loadFactor

   加载因子。

#### modCount

   用来实现“fail-fast”机制的（也就是快速失败）。所谓快速失败就是在并发集合中，其进行迭代操作时，若有其他线程对其进行结构性的修改，这时迭代器会立马感知到，并且立即抛出ConcurrentModificationException异常，而不是等到迭代完成之后才告诉你（你已经出错了）

### 构造方法

#### 1.默认构造函数，容量为11，加载因子为0.75

```
public Hashtable() {
    this(11, 0.75f);
}
```

#### 2.用指定初始容量和默认的加载因子 (0.75) 构造一个新的空哈希表

```
public Hashtable(int initialCapacity) {
    this(initialCapacity, 0.75f);
}
```

#### 3.用指定初始容量和指定加载因子构造一个新的空哈希表。

```
public Hashtable(int initialCapacity, float loadFactor) {
	//验证初始容量
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    //验证加载因子                                       
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal Load: "+loadFactor);

    if (initialCapacity==0)
        initialCapacity = 1;
    this.loadFactor = loadFactor;
    //初始化table，获得大小为initialCapacity的table数组
    table = new Entry<?,?>[initialCapacity];
    //计算阀值
    threshold = (int)Math.min(initialCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
}
```

```
private int hash(Object k) {
        return hashSeed ^ k.hashCode();
    }
```



#### 4.构造一个与给定的 Map 具有相同映射关系的新哈希表

```
public Hashtable(Map<? extends K, ? extends V> t) {
    this(Math.max(2*t.size(), 11), 0.75f);
    putAll(t);
}
```

### 几个常用的方法

#### 1.**put方法**

```
//获取synchronized锁
public synchronized V put(K key, V value) {
    // Make sure the value is not null
    //如果value是空抛出异常
    if (value == null) {
        throw new NullPointerException();
    }

    // Makes sure the key is not already in the hashtable.
    Entry<?,?> tab[] = table;
    //计算key的哈希值和index
    int hash = key.hashCode();
    int index = (hash & 0x7FFFFFFF) % tab.length;
    @SuppressWarnings("unchecked")
    Entry<K,V> entry = (Entry<K,V>)tab[index];
    //遍历对应位置列表
    for(; entry != null ; entry = entry.next) {
        if ((entry.hash == hash) && entry.key.equals(key)) {
            V old = entry.value;
            entry.value = value;
            return old;
        }
    }

    addEntry(hash, key, value, index);
    return null;
}
```

##### 步骤

1.获取synchronized锁。
2.put方法不允许null值，如果发现是null，则直接抛出异常。
3.计算key的哈希值和index
4.遍历对应位置的链表，如果发现已经存在相同的hash和key，则更新value，并返回旧值。
5.如果不存在相同的key的Entry节点，则调用addEntry方法增加节点。

#### 2.addEntry(hash, key, value, index)

```
private void addEntry(int hash, K key, V value, int index) {
    modCount++;

    Entry<?,?> tab[] = table;
    //当前容量超过阈值,需要扩容
    if (count >= threshold) {
        // Rehash the table if the threshold is exceeded
        //重新构建桶数组，并对数组中所有键值对重哈希，耗时
        rehash();

        tab = table;
        hash = key.hashCode();
        index = (hash & 0x7FFFFFFF) % tab.length;//取摸运算
    }

    // Creates the new entry.
    @SuppressWarnings("unchecked")
    Entry<K,V> e = (Entry<K,V>) tab[index];
    //生成一个新结点, 将新结点插到链表首部
    tab[index] = new Entry<>(hash, key, value, e);
    count++;
}
```

##### 步骤

1. 当前容量超过阈值,需要扩容
2. 生成一个新结点, 将新结点插到链表首部

#### 3.rehash()

(扩容方法)相当于hashmap中的resize()方法

```
protected void rehash() {
    int oldCapacity = table.length;
    Entry<?,?>[] oldMap = table;

    //扩容扩为原来的两倍+1
    int newCapacity = (oldCapacity << 1) + 1;
    //判断是否超过最大容量
    if (newCapacity - MAX_ARRAY_SIZE > 0) {
        if (oldCapacity == MAX_ARRAY_SIZE)
            // Keep running with MAX_ARRAY_SIZE buckets
            return;
        newCapacity = MAX_ARRAY_SIZE;
    }
    Entry<?,?>[] newMap = new Entry<?,?>[newCapacity];

    modCount++;
    //计算下一次rehash的阈值
    threshold = (int)Math.min(newCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
    table = newMap;

    //把旧哈希表的键值对重新哈希到新哈希表中去
    for (int i = oldCapacity ; i-- > 0 ;) {
        for (Entry<K,V> old = (Entry<K,V>)oldMap[i] ; old != null ; ) {
            Entry<K,V> e = old;
            old = old.next;

            int index = (e.hash & 0x7FFFFFFF) % newCapacity;
            e.next = (Entry<K,V>)newMap[index];
            newMap[index] = e;
        }
    }
}
```

##### 步骤

1.数组长度增加一倍（如果超过上限，则设置成上限值）。
2.更新哈希表的扩容门限值。
3.遍历旧表中的节点，计算在新表中的index，插入到对应位置链表的头部。

#### 4.**get方法**

```
public synchronized V get(Object key) {//根据键取出对应索引  
      Entry tab[] = table;  
      int hash = hash(key);//先根据key计算hash值  
      int index = (hash & 0x7FFFFFFF) % tab.length;//再根据hash值找到索引  
      for (Entry<K,V> e = tab[index] ; e != null ; e = e.next) {//遍历entry链  
          if ((e.hash == hash) && e.key.equals(key)) {//若找到该键  
              return e.value;//返回对应的值  
          }  
      }  
      return null;//否则返回null  
  }  
```

##### 步骤

1.先获取synchronized锁。
2.计算key的哈希值和index。
3.在对应位置的链表中寻找具有相同hash和key的节点，返回节点的value。
4.如果遍历结束都没有找到节点，则返回null。

#### 5.**remove方法**

```java
public synchronized V remove(Object key) {
    Entry<?,?> tab[] = table;
    //计算key的哈希值和index
    int hash = key.hashCode();
    int index = (hash & 0x7FFFFFFF) % tab.length;
    @SuppressWarnings("unchecked")
    Entry<K,V> e = (Entry<K,V>)tab[index];
    for(Entry<K,V> prev = null ; e != null ; prev = e, e = e.next) {
        //遍历对应位置的链表，寻找待删除节点
        if ((e.hash == hash) && e.key.equals(key)) {
            modCount++;
			//更新前驱节点的next，指向e的next
            if (prev != null) {
                prev.next = e.next;
            } else {
                tab[index] = e.next;
            }
            count--;
            V oldValue = e.value;
            e.value = null;
            //返回待删除节点的value值
            return oldValue;
        }
    }
    //如果不存在，返回null
    return null;
}
```

##### 步骤

1.先获取synchronized锁。
2.计算key的哈希值和index。
3.遍历对应位置的链表，寻找待删除节点，如果存在，用e表示待删除节点，pre表示前驱节点。如果不存在，返回null。
4.更新前驱节点的next，指向e的next。返回待删除节点的value值。