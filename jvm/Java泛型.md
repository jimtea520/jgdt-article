Java泛型

# 一、简介

 

  泛型是Java 1.5的新特性，泛型的本质是参数化类型，也就是说所操作的数据类型被指定为一个参数。这种参数类型可以用在类、接口和方法的创建中，分别称为泛型类、泛型接口、泛型方法。

 Java泛型被引入的好处是安全简单。

  在Java SE 1.5之前，没有泛型的情况的下，通过对类型Object的引用来实现参数的“任意化”，“任意化”带来的缺点是要做显式的强制类型转换，而这种转换是要求开发者对实际参数类型可以预知的情况下进行的。对于强制类型转换错误的情况，编译器可能不提示错误，在运行的时候才出现异常，这是一个安全隐患 。



# 二、好处

编译的时候检查类型安全，并且所有的强制转换都是自动和隐式的，提高代码的重用率。



# 三、规则和限制

1、泛型的类型参数只能是类类型（包括自定义类），不能是简单类型。

2、同一种泛型可以对应多个版本（因为参数类型是不确定的），不同版本的泛型类实例是不兼容的。

3、泛型的类型参数可以有多个。

4、泛型的参数类型可以使用extends语句，例如。习惯上成为“有界类型”。

5、泛型的参数类型还可以是通配符类型。



# 四、泛型的使用

泛型有三种常用的使用方式：**泛型类**，**泛型接口**和**泛型方法**。

##### 泛型类

一个泛型类（`generic class`）就是具有一个或多个类型变量的类。

举例：

```java
/*
 * 泛型类
 * Java库中 E表示集合的元素类型，K 和 V分别表示表的关键字与值的类型
 * T（需要时还可以用临近的字母 U 和 S）表示“任意类型”
 */
public class Pair<T> {
    
    private T name;
    private T price;

    public Pair() {
    }

    public Pair(T name, T price) {
        this.name = name;
        this.price = price;
    }

    public T getName() {
        return name;
    }

    public void setName(T name) {
        this.name = name;
    }

    public T getPrice() {
        return price;
    }

    public void setPrice(T price) {
        this.price = price;
    }
}
```



##### 泛型接口

```java
public interface Generator<T> {

    public T next();

}
```

继承接口：

```java
public class FruitGenerator implements Generator<String> {

    @Override
    public String next() {
        return "Fruit";
    }
}

public class FruitGenerator<T> implements Inter<T> {

    @Override
    public void show(T t) {
        System.out.println(t);
    }
}
```



##### 泛型方法

```java
public class ArrayAlg {

    public static <T> T getMiddle(T... a) {
        return a[a.length / 2];
    }
    
    public static void main(String[] args){
        System.out.println(ArrayAlg.getMiddle(1,2,3,4,5));
    }
}
```

这个方法是**在普通类中定义的，而不是在泛型类中定义的**。然而，这是一个泛型方法，可以从尖括号和类型变量看出这一点。注意，**类型变量放在修饰符（这里是 `public static`）的后面，返回类型的前面。**



## **五、类型通配符**

（1）无界
类型通配符我感觉上和泛型方法差不多，只是不用在使用前进行定义，例子如下：

```
public void processElements(List<?> elements){
   for(Object o : elements){
      System.out.println(o);
   }
}
```

"?"可以接收任何类型。
（2）上界

```
public void processElements(List<? extends A> elements){
   for(A a : elements){
      System.out.println(a.getValue());
   }
}
```

这种情况下能够接收A类或者A类的子类。
（3）下界

```
public static void insertElements(List<? super A> list){
    list.add(new A());
    list.add(new B());
    list.add(new C());
}
```

这种情况下能够接收A类或者A类的父类



# 六、类型通配符和泛型方法

这两种方式是差不多的，不过在使用时，如果参数之间是有依赖关系的，那么可以使用泛型方法，否则就使用类型通配符。*（如果一个方法的返回值、某些参数的类型依赖另一个参数的类型就应该使用泛型方法，因为被依赖的类型如果是不确定的?，那么其他元素就无法依赖它）*。



# 七、开发相关

泛型通配符< ? extends T >来接收返回的数据，此写法的泛型集合不能使用 add 方 法， 而< ? super T >不能使用 get 方法，做为接口调用赋值时易出错。

这个怎么来理解呢，当我们使用extends时，我们可以读元素，因为元素都是A类或子类，可以放心的用A类拿出。
当使用super时，可以添加元素，因为都是A类或父类，那么就可以安全的插入A类。