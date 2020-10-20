# HashMap和TreeMap的区别



Map：在数组中是通过数组下标来对 其内容进行索引的，而Map是通过对象来对 对象进行索引的，用来 索引的对象叫键key，其对应的对象叫值value；

1、HashMap是通过hashcode()对其内容进行快速查找的；HashMap中的元素是没有顺序的；

  TreeMap中所有的元素都是有某一固定顺序的，如果需要得到一个有序的结果，就应该使用TreeMap；

2、HashMap和TreeMap都不是线程安全的；

3、HashMap继承AbstractMap类；覆盖了hashcode() 和equals() 方法，以确保两个相等的映射返回相同的哈希值；

   TreeMap继承SortedMap类；他保持键的有序顺序；

4、HashMap：基于hash表实现的；使用HashMap要求添加的键类明确定义了hashcode() 和equals() （可以重写该方法）；为了优化HashMap的空间使用，可以调优初始容量和负载因子；

   TreeMap：基于红黑树实现的；TreeMap就没有调优选项，因为红黑树总是处于平衡的状态；

5、HashMap：适用于Map插入，删除，定位元素；

​     TreeMap：适用于按自然顺序或自定义顺序遍历键（key）；

6、HashMap:默认数组长度: 16，数组最大长度: 2^30，扩容因子: 0.75f，列表转红黑树阈值: 8，红黑树转列表阈值: 6，列表转红黑树最小数组长度: 64

​	  TreeMap:没发现有范围控制

