#  基础面试题：Collections 工具类详解

##### 对集合本体的操作

###### 线程同步集合的包装

- 特征
  - 方法名为synchronizedXxx
- 适用范围
  - List
  - Collection
  - Set
  - Map
- 缺点
  - 每次读写都要加锁，锁的层级为对象，不利于多线程的同时操作
  - 在使用Iterator的遍历时修改元素ConcurrentModificationException
  - 建议使用java.util.concurrent的集合线程同步类

###### 返回不可变集合

- 特征
  - 方法名为emptyXxx(空集合)/singletonXxx(包含传入元素的集合)/unmodifiableXxx(包含传入集合元素的集合)
- 适用范围
  - emptyXxx
    - Set
    - List
    - Map
    - Iterator
    - Enumeration
  - singletonXxx
    - Set
    - List
    - Map
  - unmodifiableXxx
    - Map
    - List
    - Set

###### 返回指定集的动态类型安全视图

- 特征
  - 方法名为checkedXxx
- 适用范围
  - List
  - Map
  - Queue
  - Set
  - Collection

###### 集合间的转换

- 特征
  - asLifoQueue(将传入的Deque转换成Queue)
  - list(将传入的Enumeration转换成ArrayList)
  - newSetFromMap(根据传入的空Map返回Set)
  - nCopies(根据传入的n返回含n个副本的List)

##### 集合内元素的操作

###### 添加元素到集合中

- 特征
  - addAll
  - copy(将源集合元素复制到目标集合中)
- 适用范围
  - addAll
    - Collection
  - copy
    - List

###### 查找元素

- 特征
  - binarySearch(二分查找特定元素)
  - frequency(查找元素出现次数)
  - indexOfSubList(返回目标list在源list的开始位置)
  - subIndexOfSubList(返回目标list在源list的结束位置)
  - shuffle(返回随机索引元素)
- 适用范围
  - binarySearch
    - List
  - frequency
    - Collection
  - shuffle
    - List

###### 替换

- 特征
  - fill(替换集合所有元素)
  - replaceAll(替换特定的值)
- 适用范围
  - fill
    - List
  - replaceAll
    - List

###### 改变元素位置

- 特征
  - sort(排序)
  - swap
  - rotate(反转)
  - reverse
- 适用范围
  - List

###### 对比元素

- 特征
  - min/max(寻找最大/小元素)
  - disJoint(判断两个集合元素是否全不同)
- 适用范围
  - Collection

#### 总结

Collections工具类能对各接口以及实现类实现多种操作

- 集合类级操作
  - 返回线程安全集合
  - 返回不可变集合
  - 返回安全视图
  - 集合间的转换
- 涉及到内部元素的操作
  - 添加元素到集合中
  - 查找特定元素
  - 替换元素
  - 改变元素位置
  - 元素间的比较



Collections 工具类几种常用方法示列:

### 1.排序操作

```
void reverse(List list)//反转
void shuffle(List list)//随机排序
void sort(List list)//按自然排序的升序排序
void sort(List list, Comparator c)//定制排序，由Comparator控制排序逻辑
void swap(List list, int i , int j)//交换两个索引位置的元素
void rotate(List list, int distance)//旋转。当distance为正数时，将list后distance个元素整体移到前面。当distance为负数时，将 list的前distance个元素整体移到后面。
```

**示例代码:**

```
     ArrayList<Integer> arrayList = new ArrayList<Integer>();
		arrayList.add(-1);
		arrayList.add(3);
		arrayList.add(3);
		arrayList.add(-5);
		arrayList.add(7);
		arrayList.add(4);
		arrayList.add(-9);
		arrayList.add(-7);
		System.out.println("原始数组:");
		System.out.println(arrayList);
		
		// void reverse(List list),反转
		Collections.reverse(arrayList);
		System.out.println("Collections.reverse(arrayList):");
		System.out.println(arrayList);

		//void rotate(arrayList, 4),旋转
		Collections.rotate(arrayList, 4);
		System.out.println("Collections.rotate(arrayList, 4):" + arrayList);

		// void sort(List list),按自然排序的升序排序
		Collections.sort(arrayList);
		System.out.println("Collections.sort(arrayList):" + arrayList);
		
		//void sort(List list, Comparator c)//定制排序，由Comparator控制排序逻辑
		Collections.sort(arrayList, new Comparator<Integer>() {
			@Override
			public int compare(Integer o1, Integer o2) {
				return o2.compareTo(o1);
			}
		});
		System.out.println("Collections.sort(List list, Comparator c)：" + arrayList);
		
		// void shuffle(List list),随机排序
		Collections.shuffle(arrayList);
		System.out.println("Collections.shuffle(arrayList):" + arrayList);

		// void swap(List list, int i , int j),交换两个索引位置的元素
		Collections.swap(arrayList, 2, 5);
		System.out.println("Collections.swap(arrayList, 2, 5):" + arrayList);

		
```

### 2.查找,替换操作

```
int binarySearch(List list, Object key)//对List进行二分查找，返回索引，注意List必须是有序的
int max(Collection coll)//根据元素的自然顺序，返回最大的元素。 
int min(Collection coll)//根据元素的自然顺序，返回最小的元素。
int max(Collection coll, Comparator c)//根据定制排序，返回最大元素，排序规则由Comparatator类控制。int min(Collection coll, Comparator c)//根据定制排序，返回最小元素，排序规则由Comparatator类控制。
void fill(List list, Object obj)//用指定的元素代替指定list中的所有元素。
int frequency(Collection c, Object o)//统计元素出现次数
int indexOfSubList(List list, List target)//获取指定源列表中指定的目标列表中最后一次出现的起始位置。找不到则返回-1，类比int lastIndexOfSubList(List source, list target)//获取指定源列表中指定的目标列表中最后一次出现的起始位置。找不到返回-1
boolean replaceAll(List list, Object oldVal, Object newVal), 用新元素替换旧元素
```

**示例代码：**

```
		ArrayList<Integer> arrayList = new ArrayList<Integer>();
		arrayList.add(-1);
		arrayList.add(3);
		arrayList.add(3);
		arrayList.add(-5);
		arrayList.add(7);
		arrayList.add(4);
		arrayList.add(-9);
		arrayList.add(-7);
		ArrayList<Integer> arrayList2 = new ArrayList<Integer>();
		arrayList2.add(-3);
		arrayList2.add(-5);
		arrayList2.add(7);
		
		System.out.println("原始数组:");
		System.out.println(arrayList);
		
		//int max(Collection coll),根据元素的自然顺序，返回最大的元素。
		System.out.println("Collections.max(arrayList):" + Collections.max(arrayList));
		
		//int min(Collection coll)//根据元素的自然顺序，返回最小的元素。
		System.out.println("Collections.min(arrayList):" + Collections.min(arrayList));
		
		//boolean replaceAll(List list, Object oldVal, Object newVal), 用新元素替换旧元素
		Collections.replaceAll(arrayList, 3, -3);
		System.out.println("Collections.replaceAll(arrayList, 3, -3):" + arrayList);
		
		//int frequency(Collection c, Object o)//统计元素出现次数
		System.out.println("Collections.frequency(arrayList, -3):" + Collections.frequency(arrayList, -3));
	
		////获取指定源列表中指定的目标列表中最后一次出现的起始位置。
		System.out.println("Collections.indexOfSubList(arrayList, arrayList2):" + Collections.indexOfSubList(arrayList, arrayList2));
		
		System.out.println("Collections.binarySearch(arrayList, 7):");
		// 对List进行二分查找，返回索引，List必须是有序的
		Collections.sort(arrayList);
		System.out.println(Collections.binarySearch(arrayList, 7));
```

### 同步控制

Collections提供了多个`synchronizedXxx()`方法·，该方法可以将指定集合包装成线程同步的集合，从而解决多线程并发访问集合时的线程安全问题。

我们知道 HashSet，TreeSet，ArrayList,LinkedList,HashMap,TreeMap 都是线程不安全的。Collections提供了多个静态方法可以把他们包装成线程同步的集合。

**最好不要用下面这些方法，效率非常低，需要线程安全的集合类型时请考虑使用 JUC 包下的并发集合。**

方法如下：

```
synchronizedCollection(Collection<T>  c) //返回指定 collection 支持的同步（线程安全的）collection。
synchronizedList(List<T> list)//返回指定列表支持的同步（线程安全的）List。
synchronizedMap(Map<K,V> m) //返回由指定映射支持的同步（线程安全的）Map。
synchronizedSet(Set<T> s) //返回指定 set 支持的同步（线程安全的）set。
```

### Collections还可以设置不可变集合，提供了如下三类方法：

```
emptyXxx(): 返回一个空的、不可变的集合对象，此处的集合既可以是List，也可以是Set，还可以是Map。
singletonXxx(): 返回一个只包含指定对象（只有一个或一个元素）的不可变的集合对象，此处的集合可以是：List，Set，Map。
unmodifiableXxx(): 返回指定集合对象的不可变视图，此处的集合可以是：List，Set，Map。
上面三类方法的参数是原有的集合对象，返回值是该集合的”只读“版本。
```

**示例代码：**

```
        ArrayList<Integer> arrayList = new ArrayList<Integer>();
        arrayList.add(-1);
        arrayList.add(3);
        arrayList.add(3);
        arrayList.add(-5);
        arrayList.add(7);
        arrayList.add(4);
        arrayList.add(-9);
        arrayList.add(-7);
        HashSet<Integer> integers1 = new HashSet<>();
        integers1.add(1);
        integers1.add(3);
        integers1.add(2);
        Map scores = new HashMap();
        scores.put("语文" , 80);
        scores.put("Java" , 82);

        //Collections.emptyXXX();创建一个空的、不可改变的XXX对象
        List<Object> list = Collections.emptyList();
        System.out.println(list);//[]
        Set<Object> objects = Collections.emptySet();
        System.out.println(objects);//[]
        Map<Object, Object> objectObjectMap = Collections.emptyMap();
        System.out.println(objectObjectMap);//{}

        //Collections.singletonXXX();
        List<ArrayList<Integer>> arrayLists = Collections.singletonList(arrayList);
        System.out.println(arrayLists);//[[-1, 3, 3, -5, 7, 4, -9, -7]]
        //创建一个只有一个元素，且不可改变的Set对象
        Set<ArrayList<Integer>> singleton = Collections.singleton(arrayList);
        System.out.println(singleton);//[[-1, 3, 3, -5, 7, 4, -9, -7]]
        Map<String, String> nihao = Collections.singletonMap("1", "nihao");
        System.out.println(nihao);//{1=nihao}

        //unmodifiableXXX();创建普通XXX对象对应的不可变版本
        List<Integer> integers = Collections.unmodifiableList(arrayList);
        System.out.println(integers);//[-1, 3, 3, -5, 7, 4, -9, -7]
        Set<Integer> integers2 = Collections.unmodifiableSet(integers1);
        System.out.println(integers2);//[1, 2, 3]
        Map<Object, Object> objectObjectMap2 = Collections.unmodifiableMap(scores);
        System.out.println(objectObjectMap2);//{Java=82, 语文=80}

        //添加出现异常：java.lang.UnsupportedOperationException
//        list.add(1);
//        arrayLists.add(arrayList);
//        integers.add(1);
```

