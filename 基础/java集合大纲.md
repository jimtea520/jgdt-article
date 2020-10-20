集合大纲

 ![常用集合大纲](https://img-blog.csdn.net/20180803184611883?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZlaXlhbmFmZmVjdGlvbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![这里写图片描述](https://img-blog.csdn.net/20180803195348216?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZlaXlhbmFmZmVjdGlvbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![Collection集合大纲](https://img-blog.csdn.net/20180803184706534?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZlaXlhbmFmZmVjdGlvbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

1.集合和数组的区别：

![这里写图片描述](https://img-blog.csdn.net/20180803193134355?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZlaXlhbmFmZmVjdGlvbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

2.Collection集合的方法：

![这里写图片描述](https://img-blog.csdn.net/20180803193423722?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZlaXlhbmFmZmVjdGlvbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

3.常用集合的分类：

Collection 接口的接口 对象的集合（单列集合） 
├——-List 接口：元素按进入先后有序保存，可重复 
│—————-├ LinkedList 接口实现类， 链表， 插入删除， 没有同步， 线程不安全 
│—————-├ ArrayList 接口实现类， 数组， 随机访问， 没有同步， 线程不安全 
│—————-└ Vector 接口实现类 数组， 同步， 线程安全 
│ ———————-└ Stack 是Vector类的实现类 
└——-Set 接口： 仅接收一次，不可重复，并做内部排序 
├—————-└HashSet 使用hash表（数组）存储元素 
│————————└ LinkedHashSet 链表维护元素的插入次序 
└ —————-TreeSet 底层实现为二叉树，元素排好序

Map 接口 键值对的集合 （双列集合） 
├———Hashtable 接口实现类， 同步， 线程安全 
├———HashMap 接口实现类 ，没有同步， 线程不安全- 
│—————–├ LinkedHashMap 双向链表和哈希表实现 
│—————–└ WeakHashMap 
├ ——–TreeMap 红黑树对所有的key进行排序 
└———IdentifyHashMap

二、List和Set集合详解：

1.list和set的区别：

![这里写图片描述](https://img-blog.csdn.net/20180803201610610?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZlaXlhbmFmZmVjdGlvbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

2.List：

（1）ArrayList：底层数据结构是数组，查询快，增删慢，线程不安全，效率高，可以存储重复元素 
（2）LinkedList 底层数据结构是链表，查询慢，增删快，线程不安全，效率高，可以存储重复元素 
（3）Vector:底层数据结构是数组，查询快，增删慢，线程安全，效率低，可以存储重复元素 

（4）小结：

![这里写图片描述](https://img-blog.csdn.net/20180803201736883?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZlaXlhbmFmZmVjdGlvbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

3.Set：

(1)HashSet底层数据结构采用哈希表实现，元素无序且唯一，线程不安全，效率高，可以存储null元素，元素的唯一性是靠所存储元素类型是否重写hashCode()和equals()方法来保证的，如果没有重写这两个方法，则无法保证元素的唯一性。 
具体实现唯一性的比较过程：存储元素首先会使用hash()算法函数生成一个int类型hashCode散列值，然后已经的所存储的元素的hashCode值比较，如果hashCode不相等，则所存储的两个对象一定不相等，此时存储当前的新的hashCode值处的元素对象；如果hashCode相等，存储元素的对象还是不一定相等，此时会调用equals()方法判断两个对象的内容是否相等，如果内容相等，那么就是同一个对象，无需存储；如果比较的内容不相等，那么就是不同的对象，就该存储了，此时就要采用哈希的解决地址冲突算法，在当前hashCode值处类似一个新的链表， 在同一个hashCode值的后面存储存储不同的对象，这样就保证了元素的唯一性。 
Set的实现类的集合对象中不能够有重复元素，HashSet也一样他是使用了一种标识来确定元素的不重复，HashSet用一种算法来保证HashSet中的元素是不重复的， HashSet采用哈希算法，底层用数组存储数据。默认初始化容量16，加载因子0.75。 
Object类中的hashCode()的方法是所有子类都会继承这个方法，这个方法会用Hash算法算出一个Hash（哈希）码值返回，HashSet会用Hash码值去和数组长度取模， 模（这个模就是对象要存放在数组中的位置）相同时才会判断数组中的元素和要加入的对象的内容是否相同，如果不同才会添加进去。 
Hash算法是一种散列算法。 
Set hs=new HashSet();

hs.add(o); 
| 
o.hashCode(); 
| 
o%当前总容量 (0–15) 
| 
| 不发生冲突 
是否发生冲突—————–直接存放 
| 
| 发生冲突 
| 假（不相等） 
o1.equals(o2)——————-找一个空位添加 
| 
| 是（相等） 
不添加 
覆盖hashCode()方法的原则： 
1、一定要让那些我们认为相同的对象返回相同的hashCode值 
2、尽量让那些我们认为不同的对象返回不同的hashCode值，否则，就会增加冲突的概率。 
3、尽量的让hashCode值散列开（两值用异或运算可使结果的范围更广） 
HashSet 的实现比较简单，相关HashSet的操作，基本上都是直接调用底层HashMap的相关方法来完成，我们应该为保存到HashSet中的对象覆盖hashCode()和equals()，因为再将对象加入到HashSet中时，会首先调用hashCode方法计算出对象的hash值，接着根据此hash值调用HashMap中的hash方法，得到的值& (length-1)得到该对象在hashMap的transient Entry[] table中的保存位置的索引，接着找到数组中该索引位置保存的对象，并调用equals方法比较这两个对象是否相等，如果相等则不添加，注意：所以要存入HashSet的集合对象中的自定义类必须覆盖hashCode(),equals()两个方法，才能保证集合中元素不重复。在覆盖equals()和hashCode()方法时， 要使相同对象的hashCode()方法返回相同值，覆盖equals()方法再判断其内容。为了保证效率，所以在覆盖hashCode()方法时， 也要尽量使不同对象尽量返回不同的Hash码值。

如果数组中的元素和要加入的对象的hashCode()返回了相同的Hash值（相同对象）,才会用equals()方法来判断两个对象的内容是否相同。

(2)LinkedHashSet底层数据结构采用链表和哈希表共同实现，链表保证了元素的顺序与存储顺序一致，哈希表保证了元素的唯一性。线程不安全，效率高。 
(3)TreeSet底层数据结构采用二叉树来实现，元素唯一且已经排好序；唯一性同样需要重写hashCode和equals()方法，二叉树结构保证了元素的有序性。根据构造方法不同，分为自然排序（无参构造）和比较器排序（有参构造），自然排序要求元素必须实现Compareable接口，并重写里面的compareTo()方法，元素通过比较返回的int值来判断排序序列，返回0说明两个对象相同，不需要存储；比较器排需要在TreeSet初始化是时候传入一个实现Comparator接口的比较器对象，或者采用匿名内部类的方式new一个Comparator对象，重写里面的compare()方法； 
（4）小结：Set具有与Collection完全一样的接口，因此没有任何额外的功能，不像前面有两个不同的List。实际上Set就是Collection,只 是行为不同。(这是继承与多态思想的典型应用：表现不同的行为。)Set不保存重复的元素。 
Set 存入Set的每个元素都必须是唯一的，因为Set不保存重复元素。加入Set的元素必须定义equals()方法以确保对象的唯一性。Set与Collection有完全一样的接口。Set接口不保证维护元素的次序。

4.List和Set总结：

（1）、List,Set都是继承自Collection接口，Map则不是 
（2）、List特点：元素有放入顺序，元素可重复 ，Set特点：元素无放入顺序，元素不可重复，重复元素会覆盖掉，（注意：元素虽然无放入顺序，但是元素在set中的位置是有该元素的HashCode决定的，其位置其实是固定的，加入Set 的Object必须定义equals()方法 ，另外list支持for循环，也就是通过下标来遍历，也可以用迭代器，但是set只能用迭代，因为他无序，无法用下标来取得想要的值。） 
（3）.Set和List对比： 
Set：检索元素效率低下，删除和插入效率高，插入和删除不会引起元素位置改变。 
List：和数组类似，List可以动态增长，查找元素效率高，插入删除元素效率低，因为会引起其他元素位置改变。 
（4）、ArrayList与LinkedList的区别和适用场景 
Arraylist： 
优点：ArrayList是实现了基于动态数组的数据结构,因为地址连续，一旦数据存储好了，查询操作效率会比较高（在内存里是连着放的）。 
缺点：因为地址连续， ArrayList要移动数据,所以插入和删除操作效率比较低。

LinkedList： 
优点：LinkedList基于链表的数据结构,地址是任意的，所以在开辟内存空间的时候不需要等一个连续的地址，对于新增和删除操作add和remove，LinedList比较占优势。LinkedList 适用于要头尾操作或插入指定位置的场景 
缺点：因为LinkedList要移动指针,所以查询操作性能比较低。 
适用场景分析： 
当需要对数据进行对此访问的情况下选用ArrayList，当需要对数据进行多次增加删除修改时采用LinkedList。

ArrayList与Vector的区别和适用场景 
ArrayList有三个构造方法：

public ArrayList(int initialCapacity)//构造一个具有指定初始容量的空列表。    
public ArrayList()      //默认构造一个初始容量为10的空列表。    
public ArrayList(Collection<? extends E> c)//构造一个包含指定 collection 的元素的列表

Vector有四个构造方法：

public Vector()//使用指定的初始容量和等于0的容量增量构造一个空向量。    
public Vector(int initialCapacity)//构造一个空向量，使其内部数据数组的大小，其标准容量增量为零。    
public Vector(Collection<? extends E> c)//构造一个包含指定 collection 中的元素的向量    
public Vector(int initialCapacity,int capacityIncrement)//使用指定的初始容量和容量增量构造一个空的向量    

ArrayList和Vector都是用数组实现的，主要有这么三个区别： 
（1）.Vector是多线程安全的，线程安全就是说多线程访问同一代码，不会产生不确定的结果。而ArrayList不是，这个可以从源码中看出，Vector类中的方法很多有synchronized进行修饰，这样就导致了Vector在效率上无法与ArrayList相比； 
（2）两个都是采用的线性连续空间存储元素，但是当空间不足的时候，两个类的增加方式是不同。 
*（3）*Vector可以设置增长因子，而ArrayList不可以。 
*（4）*Vector是一种老的动态数组，是线程同步的，效率很低，一般不赞成使用。 
适用场景分析： 
1.Vector是线程同步的，所以它也是线程安全的，而ArrayList是线程异步的，是不安全的。如果不考虑到线程的安全因素，一般用ArrayList效率比较高。 
2.如果集合中的元素的数目大于目前集合数组的长度时，在集合中使用数据量比较大的数据，用Vector有一定的优势。

.TreeSet 是二差树（红黑树的树据结构）实现的,Treeset中的数据是自动排好序的，不允许放入null值 
2.HashSet 是哈希表实现的,HashSet中的数据是无序的，可以放入null，但只能放入一个null，两者中的值都不能重复，就如数据库中唯一约束 
3.HashSet要求放入的对象必须实现HashCode()方法，放入的对象，是以hashcode码作为标识的，而具有相同内容的String对象，hashcode是一样，所以放入的内容不能重复。但是同一个类的对象可以放入不同的实例

适用场景分析:HashSet是基于Hash算法实现的，其性能通常都优于TreeSet。为快速查找而设计的Set，我们通常都应该使用HashSet，在我们需要排序的功能时，我们才使用TreeSet。 
（5）何时使用： 

![这里写图片描述](https://img-blog.csdn.net/20180803204923763?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZlaXlhbmFmZmVjdGlvbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


三、Map详解：

Map用于保存具有映射关系的数据，Map里保存着两组数据：key和value，它们都可以使任何引用类型的数据，但key不能重复。所以通过指定的key就可以取出对应的value。

（1）、请注意！！！， Map 没有继承 Collection 接口， Map 提供 key 到 value 的映射，你可以通过“键”查找“值”。一个 Map 中不能包含相同的 key ，每个 key 只能映射一个 value 。 Map 接口提供 3 种集合的视图， Map 的内容可以被当作一组 key 集合，一组 value 集合，或者一组 key-value 映射。 
（2）Map： 

![这里写图片描述](https://img-blog.csdn.net/20180803205119738?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZlaXlhbmFmZmVjdGlvbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

（3）HashMap和HashTable的比较：

![这里写图片描述](https://img-blog.csdn.net/20180803205546704?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZlaXlhbmFmZmVjdGlvbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70) 

（4）TreeMap： 

![这里写图片描述](https://img-blog.csdn.net/20180803205736499?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZlaXlhbmFmZmVjdGlvbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

（5）Map的其它类： 
IdentityHashMap和HashMap的具体区别，IdentityHashMap使用 == 判断两个key是否相等，而HashMap使用的是equals方法比较key值。有什么区别呢？ 
对于==，如果作用于基本数据类型的变量，则直接比较其存储的 “值”是否相等； 如果作用于引用类型的变量，则比较的是所指向的对象的地址。 
对于equals方法，注意：equals方法不能作用于基本数据类型的变量 
如果没有对equals方法进行重写，则比较的是引用类型的变量所指向的对象的地址； 
诸如String、Date等类对equals方法进行了重写的话，比较的是所指向的对象的内容。 

![这里写图片描述](https://img-blog.csdn.net/20180803210616278?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZlaXlhbmFmZmVjdGlvbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

（6）小结： 
HashMap 非线程安全 
HashMap：基于哈希表实现。使用HashMap要求添加的键类明确定义了hashCode()和equals()[可以重写hashCode()和equals()]，为了优化HashMap空间的使用，您可以调优初始容量和负载因子。

TreeMap：非线程安全基于红黑树实现。TreeMap没有调优选项，因为该树总处于平衡状态。

适用场景分析： 
HashMap和HashTable:HashMap去掉了HashTable的contains方法，但是加上了containsValue()和containsKey()方法。HashTable同步的，而HashMap是非同步的，效率上比HashTable要高。HashMap允许空键值，而HashTable不允许。

HashMap：适用于Map中插入、删除和定位元素。 
Treemap：适用于按自然顺序或自定义顺序遍历键(key)。

5.线程安全集合类与非线程安全集合类 
LinkedList、ArrayList、HashSet是非线程安全的，Vector是线程安全的; 
HashMap是非线程安全的，HashTable是线程安全的; 
StringBuilder是非线程安全的，StringBuffer是线程安全的。

数据结构 
ArrayXxx:底层数据结构是数组，查询快，增删慢 
LinkedXxx:底层数据结构是链表，查询慢，增删快 
HashXxx:底层数据结构是哈希表。依赖两个方法：hashCode()和equals() 
TreeXxx:底层数据结构是二叉树。两种方式排序：自然排序和比较器排序
