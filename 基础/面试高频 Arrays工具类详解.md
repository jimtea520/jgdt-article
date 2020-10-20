

## Aarry的几个常用函数：

1. 排序 : `sort()`
2. 查找 : `binarySearch()`
3. 比较: `equals()`
4. 填充 : `fill()`
5. 转列表: `asList()`
6. 转字符串 : `toString()`
7. 复制: `copyOf()`

### 排序 : `sort()`

   1 void sort(Object[] o)：对数组从小到大的排序(String类型的数组不宜使用)

   2.void sort(int[] a, int fromIndex, int toIndex)：对数组(from,to)从小到大的排序(String类型的数组不宜使用)

​     只排(fromIndex,toIndex)之间的数值，且不包括fromIndex和toIndex。

   3.void sort(T[] a, Comparator<? super T> c)：类似Collections工具类的sort

   4.void parallelSort(int[] a) 按照数字顺序排列指定的数组(并行的)。同sort方法一样也有按范围的排序

```
		int a[] = { 1, 3, 2, 7, 6, 5, 4, 9 };
        // void sort(Object[] o)：对数组从小到大的排序
        Arrays.sort(a);
        System.out.println("Arrays.sort(a):");
        for (int i : a) {
            System.out.print(i);
        }
        // 换行
        System.out.println();

        String[] strs = { "abc", "abd", "abf" };
        Arrays.sort(strs);
        System.out.println("Arrays.toString(strs)");
        for (String str : strs) {
            System.out.print(str);
        }
        // 换行
        System.out.println();

        // void sort(int[] a, int fromIndex, int toIndex)：对数组(from,to)从小到大的排序
        //只排(fromIndex,toIndex)之间的数值，且不包括fromIndex和toIndex。
        int b[] = a.clone();
        Arrays.sort(b, 2, 6);
        System.out.println("Arrays.sort(b, 2, 6):");
        for (int i : b) {
            System.out.print(i);
        }
        // 换行
        System.out.println();

       // void sort(T[] a, Comparator<? super T> c)：类似Collections工具类的sort
        Integer[] c=new Integer[]{1, 3, 2, 7, 6, 5, 4, 9};
        Arrays.sort(c,new Comparator<Integer>()
                {

                    public int compare(Integer o1, Integer o2)
                    {
                        return o2-o1;
                    }


                    public boolean equals(Object obj)
                    {
                        return false;
                    }
                });
        System.out.println("Arrays.sort(b, new Comparator<Integer>):");
        for (Integer integer:c)
        {
            System.out.print(integer);
        }

        // 换行
        System.out.println();

        // parallelSort(int[] a) 按照数字顺序排列指定的数组(并行的)。同sort方法一样也有按范围的排序
        int d[] = a.clone();
        Arrays.parallelSort(d);
        System.out.println("Arrays.parallelSort(d)：");
        for (int i : d) {
            System.out.print(i);
        }
        // 换行
        System.out.println();

        //parallelSort给字符数组排序，sort也可以
        char e[] = { 'a', 'f', 'b', 'c', 'e', 'A', 'C', 'B' };
        Arrays.parallelSort(d);
        System.out.println("Arrays.parallelSort(e)：");
        for (char ch : e) {
            System.out.print(ch);
        }
        // 换行
        System.out.println();
```



输出结果：

```
Arrays.sort(a):
12345679
Arrays.toString(strs)
abcabdabf
Arrays.sort(b, 2, 6):
12345679
Arrays.sort(b, new Comparator<Integer>):
97654321
Arrays.parallelSort(d)：
12345679
Arrays.parallelSort(e)：
afbceACB
```



### 查找 : `binarySearch()`

​		1.int binarySearch(Object[] a, Object key)：在a数组中查找key,返回key在数组中的位置

​		二分查找法找指定元素的索引值，数组一定是排好序的，否则会出错。找到元素，只会返回最后一个位置 。

​		2.int binarySearch(int[] a, int fromIndex, int toIndex, int key)：在a数组中查找key,返回key在数组中的位置[fromIndex,toIndex)(不包括toIndex)

```
int arr[] = { 1, 3, 2, 7, 6, 5, 4, 9 };
        //int binarySearch(Object[] a, Object key)：在a数组中查找key,返回key在数组中的位置 (无序)
        int a1=Arrays.binarySearch(arr,10);
        int a2=Arrays.binarySearch(arr,6);
        System.out.println("not sort arr:");
        for (int i = 0; i < arr.length; i++) {
            System.out.print(arr[i]+",");
        }
        System.out.println();
        System.out.println("search a1, a2:");
        System.out.print("a1:"+a1+",a2:"+a2);

        // 换行
        System.out.println();
        Arrays.sort(arr);
        //int binarySearch(Object[] a, Object key)：在a数组中查找key,返回key在数组中的位置 (有序)
        a1=Arrays.binarySearch(arr,10);
        a2=Arrays.binarySearch(arr,6);
        System.out.println("sort arr:");
        for (int i = 0; i < arr.length; i++) {
            System.out.print(arr[i]+",");
        }
        System.out.println();
        System.out.println("search a1, a2:");
        System.out.print("a1:"+a1+",a2:"+a2);
        System.out.println();

        //int binarySearch(int[] a, int fromIndex, int toIndex, int key)：在a数组中查找key,返回key在数组中的位置[fromIndex,toIndex)
        a1=Arrays.binarySearch(arr,0,4,6);
        a2=Arrays.binarySearch(arr,0,6,6);
        System.out.println("search index a1, a2:");
        System.out.print("a1:"+a1+",a2:"+a2);
        System.out.println();
```



输出结果：

```
not sort arr:
1,3,2,7,6,5,4,9,
search a1, a2:
a1:-9,a2:-4
sort arr:
1,2,3,4,5,6,7,9,
search a1, a2:
a1:-9,a2:5
search index a1, a2:
a1:-5,a2:5
```



### 比较: `equals()`

​		boolean equals(Object[] a, Object[] a2)：比较两个数据是否相等，如果两个数组引用都是null，则它们被认为是相等的

```
		//boolean equals(Object[] a, Object[] a2)：比较两个数据是否相等
        String string1[]={"1","2","3","4","5"};
        String string2[]={"1","3","5","7","9"};
        String string3[]={"1","3","5","7","9"};
        String string4[]=null;
        String string5[]=null;

        boolean b1= Arrays.equals(string1,string2);
        boolean b2=Arrays.equals(string2,string3);
        boolean b3=string2.equals(string3);
        boolean b4=Arrays.equals(string1,string3);
        boolean b5=Arrays.equals(string4,string5);

        System.out.println("b1:"+b1+",b2:"+b2+",b3:"+b3+",b4:"+b4+",b5:"+b5);
```

输出结果：

```
b1:false,b2:true,b3:false,b4:false,b5:true
```



### 填充 : `fill()`

 		1. void fill(Object[] a, Object val)：填充数组
    		2. void fill(Object[] a, int fromIndex, int toIndex, Object val)：填充数组[fromIndex,toIndex)

```
		int[] a = { 1, 2, 3, 4, 5, 6, 7, 8, 9 };
        // 数组中所有元素重新分配值
        Arrays.fill(a, 3);
        System.out.println("Arrays.fill(a, 3)：");
        for (int i : a) {
            System.out.print(i);
        }
        // 换行
        System.out.println();

        int[] b = a.clone();
        // 数组中指定范围元素重新分配值
        Arrays.fill(b, 0, 2, 9);
        System.out.println("Arrays.fill(b, 0, 2, 9):");
        for (int i : b) {
            System.out.print(i);
        }
```

输出结果：

```
Arrays.fill(a, 3)：
333333333
Arrays.fill(b, 0, 2, 9):
993333333
```



### 转列表 `asList()`

​	List<T> asList(T... a)：将数组转换成集合

```
		int[] a = { 1, 2, 3, 4, 5, 6, 7, 8, 9 };
        // 数组中所有元素重新分配值
        System.out.println("int[] a：");
        for(int i:a){
            System.out.print(i);
        }
        System.out.println();

        List<int[]> list=Arrays.asList(a);
        Iterator<int[]> iterator=list.iterator();

        System.out.println("List<int[]> list：");
        while (iterator.hasNext()){
            int []result=iterator.next();
            for(int i:result){
                System.out.print(i);
            }
        }

```

输出结果：

```
int[] a：
123456789
List<int[]> list：
123456789
```



### 转字符串 `toString()`

​		返回指定数组的内容的字符串表示形式

```
		int[] a = { 1, 2, 3, 4, 5, 6, 7, 8, 9 };

        System.out.println("Arrays.toString(a)：");
        System.out.println(Arrays.toString(a));
```

输出结果：

```
Arrays.toString(a)：
[1, 2, 3, 4, 5, 6, 7, 8, 9]
```



### 复制 `copyOf()`

​		1.int[] copyOf(int[] original, int newLength)：从original数组中截取长度为newLength的新数组

​		2.int[] copyOfRange(int[] original, int from, int to)：从original数组中截取[from,to)新数组

```
		int[] a = { 1, 2, 3, 4, 5, 6, 7, 8, 9 };
        //copyOf 方法实现数组复制,a为数组，6为复制的长度
        int a1[] = Arrays.copyOf(a, 6);
        System.out.println("Arrays.copyOf(a, 6)：");
        for (int i : a1) {
            System.out.print(i);
        }
        // 换行
        System.out.println();
        // copyOfRange将指定数组的指定范围复制到新数组中
        int a2[] = Arrays.copyOfRange(a, 4, 8);
        System.out.println("Arrays.copyOfRange(a, 4, 8)：");
        for (int j : a2) {
            System.out.print(j);
        }
        // 换行
        System.out.println();
```

输出结果:

```
Arrays.copyOf(a, 6)：
123456
Arrays.copyOfRange(a, 4, 8)：
5678
```

