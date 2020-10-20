

## 注意

（1）该方法适用于对象型数据的数组（String、Integer...）

（2）该方法不建议使用于基本数据类型的数组（byte,short,int,long,float,double,boolean）

（3）该方法将数组与List列表链接起来：当更新其一个时，另一个自动更新

（4）不支持add()、remove()、clear()等方法

（5）此方法得到的List长度是不可变的

（6）asList返回的是java.util.Arrays.ArrayList，而不是java.util



### 传递的数组必须是对象数组，而不是基本类型

```
public class Demo {
    public static void main(String[] args) {
        int[] myArray = { 1, 2, 3 };
        List myList = Arrays.asList(myArray);
        System.out.println("myList.size():" + myList.size());
        System.out.println("myList.get(0):"+ myList.get(0));

        int [] array=(int[]) myList.get(0);
        System.out.print("array:");
        for (int i = 0; i < array.length; i++) {
            System.out.print(array[i]+",");
        }
        System.out.println();
        System.out.println(myList.get(1));

    }
}
```

输出结果

```
myList.size():1
myList.get(0):[I@1b6d3586
array:1,2,3,
Exception in thread "main" java.lang.ArrayIndexOutOfBoundsException: 1
	at java.util.Arrays$ArrayList.get(Arrays.java:3841)
	at fking.Demo.main(Demo.java:26)
```

### 不支持add()、remove()、clear()等方法

```
public class Demo1 {
    public static void main(String[] args) {
        List myList = Arrays.asList(1, 2, 3);
//        myList.add(4);//运行时报错：UnsupportedOperationException
//        myList.remove(1);//运行时报错：UnsupportedOperationException
//        myList.clear();//运行时报错：UnsupportedOperationException

    }
}
```

这三个分别解除注释输出结果

```
Exception in thread "main" java.lang.UnsupportedOperationException
```

### asList返回的是java.util.Arrays.ArrayList，而不是java.util

```
public class Demo2 {
    public static void main(String[] args) {
        List myList = Arrays.asList(1, 2, 3);
        System.out.println(myList.getClass());//class java.util.Arrays$ArrayList
    }
}
```

输出结果

```
class java.util.Arrays$ArrayList
```



### 几种将数组转换为ArrayList

```
public class Demo3 {
    public static void main(String[] args) {
        //方法一
        List list = new ArrayList<String>(Arrays.asList("a", "b", "c"));

        //方法二使用collect
        Integer [] myArray = { 1, 2, 3 };
        List list2 = Arrays.stream(myArray).collect(Collectors.toList());
        //基本类型也可以实现转换（依赖boxed的装箱操作）
        int [] myArray2 = { 1, 2, 3 };
        List list3 = Arrays.stream(myArray2).boxed().collect(Collectors.toList());

        //方法三
        // 需要导入google guava工具包
        //<dependency>
        //    <groupId>com.google.guava</groupId>
        //    <artifactId>guava</artifactId>
        //    <version>18.0</version>
        //</dependency>

        String[] aStringArray =  new String[]{"a", "b", "c"};
        //不可变数组
        List<String> list4 = ImmutableList.of("string", "elements");  // from varargs
        List<String> list5 = ImmutableList.copyOf(aStringArray);      // from array

        List<String> aStringCollection  = new ArrayList<>();
        aStringCollection.add("a");
        aStringCollection.add("b");
        aStringCollection.add("c");
        //可变数组
        List<String> list6 = Lists.newArrayList(aStringCollection);    // from collection
        List<String> list7 = Lists.newArrayList(aStringArray);               // from array
        List<String> list8 = Lists.newArrayList("or", "string", "elements"); // from varargs

        //方法四
        // 需要导入Apache Commons Collections工具包
        //<dependency>
        //    <groupId>org.apache.commons</groupId>
        //    <artifactId>commons-collections4</artifactId>
        //    <version>4.4</version>
        //</dependency>
        List<String> list9 = new ArrayList<>();
        CollectionUtils.addAll(list9, aStringArray);
    }
}
```



## 总结

传递的数组必须是对象数组，而不是基本类型

如果只是用来遍历，就用Arrays.asList()

如果要添加或删除元素，则选择new一个java.util.ArrayList