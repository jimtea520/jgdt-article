



# ThreadLocal源码解析

概念

ThreadLocal 是线程的局部变量， 是每一个线程所单独持有的，其他线程不能对其进行访问。

当使用ThreadLocal维护变量的时候 为每一个使用该变量的线程提供一个独立的变量副本，即每个线程内部都会有一个该变量，这样同时多个线程访问该变量并不会彼此相互影响，因此他们使用的都是自己从内存中拷贝过来的变量的副本， 这样就不存在线程安全问题，也不会影响程序的执行性能。



threadlocal,ThreadLocalMap,thread 三者关系图



**ThreadLocal的数据结构**

Thread类中有个变量threadLocals，这个类型为ThreadLocal中的一个内部类ThreadLocalMap，这个类没有实现map接口，就是一个普通的Java类，但是实现的类似map的功能。

![img](http://pics1.baidu.com/feed/bd315c6034a85edf2121027dcd5cad25dc5475ca.jpeg?token=91e25098f7ec0ffe3ffb2ac6a9e82dc8)ThreadLocal数据结构

每个线程都要自己的一个map，map是一个数组的数据结构存储数据，每个元素是一个Entry，entry的key是threadlocal的引用，也就是当前变量的副本，value就是set的值。

### `ThreadLocal` 源码分析

1.set方法

```csharp
//set 方法
public void set(T value) {
    //获取当前线程
    Thread t = Thread.currentThread();
    //获取到当前线程的 ThreadLocalMap 类型的变量 threadLocals
    ThreadLocalMap map = getMap(t);
    //如果存在则直接赋值
    if (map != null)
        map.set(this, value);
    else //如果不存在则给该线程创建 ThreadLocalMap 变量并赋值。
        createMap(t, value);
}

//获取线程中的ThreadLocalMap 字段！！
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}

//创建线程的变量
void createMap(Thread t, T firstValue) {
     t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

总结：

对象实例与 `ThreadLocal` 变量的映射关系是存放的一个 `Map` 里面（这个 `Map` 是个抽象的 `Map` 并不是 `java.util` 中的 `Map` ），而这个 `Map` 是 `Thread` 类的一个字段！而真正存放映射关系的 `Map` 就是 `ThreadLocalMap`。

2.get方法

```kotlin
public T get() {
    //获取当前Thread
    Thread t = Thread.currentThread();
    //获取到当前线程的 ThreadLocalMap 类型的变量 threadLocals
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        //存在则返回值
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    //不存在返回初始值
    return setInitialValue();
}

//设置初始化值
private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}
```

总结：

获取当前线程的 `ThreadLocalMap` 变量，如果存在则返回值，不存在则创建并返回初始值。



### `ThreadLocalMap` 源码分析

`ThreadLocal` 的底层实现都是通过 `ThreadLocalMap` 来实现的

```dart
static class ThreadLocalMap {

    /**
     * The entries in this hash map extend WeakReference, using
     * its main ref field as the key (which is always a
     * ThreadLocal object).  Note that null keys (i.e. entry.get()
     * == null) mean that the key is no longer referenced, so the
     * entry can be expunged from table.  Such entries are referred to
     * as "stale entries" in the code that follows.
     */
    static class Entry extends WeakReference<ThreadLocal<?>> {
        /** The value associated with this ThreadLocal. */
        Object value;

        Entry(ThreadLocal<?> k, Object v) {
            super(k);
            value = v;
        }
    }
    
    /**
     * The initial capacity -- MUST be a power of two.
     */
    private static final int INITIAL_CAPACITY = 16;
    
    /**
     * The table, resized as necessary.
     * table.length MUST always be a power of two.
     */
    private Entry[] table;
}
```

table 

存放对象实例与变量的关系，并且实例对象作为 key，变量作为 value 实现对应关系。这里的 key 采用的是对实例对象的弱引用，（因为我们这里的 key 是对象实例，每个对象实例有自己的生命周期，这里采用弱引用就可以在不影响对象实例生命周期的情况下对其引用）。

Entry

```
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

WeakReference 两个构造函数

```java
public class WeakReference<T> extends Reference<T> {
    public WeakReference(T referent) {
        super(referent);
    }

    public WeakReference(T referent, ReferenceQueue<? super T> q) {
        super(referent, q);
    }

}
```

1.WeakReference(T referent)：referent就是被弱引用的对象（注意区分弱引用对象和被弱引用的对应，弱引用对象是指WeakReference的实例或者其子类的实例）

2.WeakReference(T referent, ReferenceQueue<? super T> q)：与上面的构造方法比较，多了个ReferenceQueue，在对象被回收后，会把弱引用对象，也就是WeakReference对象或者其子类的对象，放入队列ReferenceQueue中，注意不是被弱引用的对象，被弱引用的对象已经被回收了。

从上可知，这是个弱引用，意味这可能会被垃圾回收器回收掉，threadLocal.get()==null，也就意味着被回收掉了

1.set方法

```csharp
private void set(ThreadLocal<?> key, Object value) {

    Entry[] tab = table;
    int len = tab.length;
    //获取 hash 值，用于数组中的下标
    int i = key.threadLocalHashCode & (len-1);
		
    //如果数组该位置有对象则进入
    //通过hash寻找下标，寻找相等的ThreadLocal对象
    //1.找到相同的对象
    //2.一直往数组下一个下标查询，指导下一个下标对应的为null,退出循环
    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();

        //k 相等则覆盖旧值
        if (k == key) {
            e.value = value;
            return;
        }

        //此时说明此处 Entry 的 k 中的对象实例已经被回收了，需要替换掉这个位置的 key 和 value
        if (k == null) {
            //key过期了，需要替换
            replaceStaleEntry(key, value, i);
            return;
        }
    }
	//没找到
    //创建 Entry 对象
    tab[i] = new Entry(key, value);
    int sz = ++size;
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        //扩容
        rehash();
}


//获取 Entry
private Entry getEntry(ThreadLocal<?> key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.get() == key)
        return e;
    else
        return getEntryAfterMiss(key, i, e);
}

//获取从i到len的下一个增量
private static int nextIndex(int i, int len) {
            return ((i + 1 < len) ? i + 1 : 0);
 }

//扩容
private void rehash() {
            expungeStaleEntries();

            // Use lower threshold for doubling to avoid hysteresis
            if (size >= threshold - threshold / 4)
                resize();
        }

//清除所有过时的对象 referent
private void expungeStaleEntries() {
            Entry[] tab = table;
            int len = tab.length;
            for (int j = 0; j < len; j++) {
                Entry e = tab[j];
                if (e != null && e.get() == null)
                    expungeStaleEntry(j);
            }
        }

        /**
         * 2倍扩容
         */
private void resize() {
            Entry[] oldTab = table;
            int oldLen = oldTab.length;
            int newLen = oldLen * 2;
            Entry[] newTab = new Entry[newLen];
            int count = 0;

            for (int j = 0; j < oldLen; ++j) {
                Entry e = oldTab[j];
                if (e != null) {
                    ThreadLocal<?> k = e.get();
                    if (k == null) {
                        e.value = null; // Help the GC
                    } else {
                        int h = k.threadLocalHashCode & (newLen - 1);
                        while (newTab[h] != null)
                            h = nextIndex(h, newLen);
                        newTab[h] = e;
                        count++;
                    }
                }
            }

            setThreshold(newLen);
            size = count;
            table = newTab;
        }

        
```

replaceStaleEntry

```
private void replaceStaleEntry(ThreadLocal<?> key, Object value,
                               int staleSlot) {
    Entry[] tab = table;
    int len = tab.length;
    Entry e;

    //从当前的staleSlot向前遍历 i--;
    //为了把前面所有的已经被垃圾回收的也一起是放空间出来
    //这里只key被回收，value还没被回收，entry更加没回收，所依需要让他们回收
    //同时也避免这样存在很多过期的对象占用，导致这个时候刚好来了一个新元素达到阈值而触发一次心的rehash
    int slotToExpunge = staleSlot;
    for (int i = prevIndex(staleSlot, len);
         (e = tab[i]) != null;
         i = prevIndex(i, len))
        if (e.get() == null)
            slotToExpunge = i;

    // 这个时候是从数据下标小的往下标大的方向遍历，i++刚好跟上面相反
    //这两个遍历为了在左边遇到第一个空的entry到右边遇到第一个空的entry之间查询所有过期的对象
    //在右边如果找到需要设置值的key 相同的时候开始清理
    	然后返回，不再继续遍历
    for (int i = nextIndex(staleSlot, len);
         (e = tab[i]) != null;
         i = nextIndex(i, len)) {
        ThreadLocal<?> k = e.get();

       //说明之前已经存在相同的key，所依需要替换旧的值并且和前面那个过期的对象交换位置
        if (k == key) {
            e.value = value;

            tab[i] = tab[staleSlot];
            tab[staleSlot] = e;

            //前面的第一个for循环(i--)往前查找的时候没有找到的过期的，只有taleSlot
            //这个过期由于前面过期的对象已经通过交换位置的方式到index=i上了，所以需要清理i，而不是staleslot
            if (slotToExpunge == staleSlot)
                slotToExpunge = i;
            //清理过期数据
            cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
            return;
        }

        //如果在第一个for循环(i--)向前遍历无任何过期对象
        //那么我们需要把slotToExpunge设置为后遍历(i++)的第一个过期对象的位置
        //如果整个数组都没找到要设置的key的时候，该key会设置在该staleslot的位置上
        //如果数组中存在要设置的key,那么上面也会通过交换位置的时候把有效值设置到staleSlot位置上
        //综上: staleSlot存放的是有效值，不需要被清理
        if (k == null && slotToExpunge == staleSlot)
            slotToExpunge = i;
    }

    // 如果key 在数组中不存在，则新建一个放进去
    tab[staleSlot].value = null;
    tab[staleSlot] = new Entry(key, value);

    // 如果有其他已经过期的对象，则清理此过期对象
    if (slotToExpunge != staleSlot)
        cleanSomeSlots(expungeStaleEntry(slotToExpunge), len);
}


//获取前一个序号
private static int prevIndex(int i, int len) {
    return ((i - 1 >= 0) ? i - 1 : len - 1);
}
```

expungeStaleEntry(int staleSlot)

```
private int expungeStaleEntry(int staleSlot) {
    Entry[] tab = table;
    int len = tab.length;

    // expunge entry at staleSlot
    tab[staleSlot].value = null;
    tab[staleSlot] = null;
    size--;

    // Rehash until we encounter null
    Entry e;
    int i;
    for (i = nextIndex(staleSlot, len);
         (e = tab[i]) != null;
         i = nextIndex(i, len)) {
        ThreadLocal<?> k = e.get();
        if (k == null) {
            e.value = null;
            tab[i] = null;
            size--;
        } else {
        //采用开放地址法，删除的元素是多个冲突元素重的一个，需要对后面的元素做处理(让后面元素往前移动)，这么做，住要是开放地址法寻找元素的时候，遇到null就停止熏着了，你前面key=null的时候已经设置entery为null，不移动后面的元素永远访问不了
            int h = k.threadLocalHashCode & (len - 1);
           //不相等说明hash是有冲突的
           if (h != i) {
                tab[i] = null;

                while (tab[h] != null)
                    h = nextIndex(h, len);
                tab[h] = e;
            }
        }
    }
    return i;
}
```

ThreadLocal 适用于每个线程需要自己独立的实例且该实例需要在多个方法中被使用（相同线程数据共享），也就是变量在线程间隔离（不同的线程数据隔离）而在方法或类间共享的场景。