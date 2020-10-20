java内部类的区别(成员内部类、静态嵌套类、方法内部类、匿名内部类)



# 一、什么是内部类

　　内部类是指在一个外部类的内部再定义一个类。内部类作为外部类的一个成员，并且依附于外部类而存在的。内部类可为静态，可用protected和private修饰（而外部类只能使用public和缺省的包访问权限）。

 　内部类主要有一下几种：**成员内部类、局部内部类、静态内部类、匿名内部类**。

# 二、内部类的共性

(1)、内部类仍然是一个独立的类，在编译之后内部类会被编译成独立的.class文件，但是前面冠以外部类的类名和$符号 。

(2)、内部类不能用普通的方式访问。

(3)、内部类声明成静态的，就不能随便的访问外部类的成员变量了，此时内部类只能访问外部类的静态成员变量 。

(4)、外部类不能直接访问内部类的的成员，但可以通过内部类对象来访问

 

# 三、为什么需要内部类

其主要原因有以下几点：

　　1、内部类方法可以访问该类定义所在的作用域的数据，包括私有的数据

　　2、内部类可以对同一个包中的其他类隐藏起来,一般的非内部类，是不允许有 private 与protected权限的，但内部类可以

　　3、可以实现多重继承

　　4、当想要定义一个回调函数且不想编写大量代码时，使用匿名内部类比较便捷

使用内部类最吸引人的原因是：

　　每个内部类都能独立地继承自一个（接口的）实现，所以无论外围类是否已经继承了某个（接口的）实现，对于内部类都没有影响。大家都知道Java只能继承一个类，它的多重继承在我们没有学习内部类之前是用接口来实现的。但使用接口有时候有很多不方便的地方。比如我们实现一个接口就必须实现它里面的所有方法。而有了内部类就不一样了。它可以使我们的类继承多个具体类或抽象类。



# 四、内部类分类



## **1、成员内部类**

即在一个类中直接定义的内部类，成员内部类与普通类的成员没什么区别，可以与普通成员一样进行修饰和限制。成员内部类不能含有static的变量和方法。

```
package com.test;

public class Outer {
    private static int number = 100;
    private int j = 20;
    private String name = "Java";

    public static void outer_funOne(){
        System.out.println("外部类Outer的静态方法:outer_funOne");
    }

    public void outer_funTwo(){
        System.out.println("外部类的普通方法：outer_funTwo");
    }

    //成员内部类，可以访问外部类的所有成员
    class Demo{
        //内部类不允许定义静态变量
        //static int demo_i = 100;
        int j =50; //内部类和外部类的实例变量可以共存

        //成员内部类中的方法定义
        public void demo_funOne(){
            //内部类中访问内部类自己的变量直接用变量名
            //也可以用  this.j
            System.out.println(j);

            //内部类中访问外部类的成员变量语法：外部类类名.this.变量名
            System.out.println("内部类访问外部类变量："+Outer.this.j);

            //如果内部类中没有与外部类中有相同的变量，则可以直接用变量名使用
            System.out.println(name);

            //内部类调用外部类方法
            outer_funOne();  //静态方法
            outer_funTwo();  //非静态方法

        }

    }

    public static void outer_funThree(){
        //外部类静态方法访问成员内部类
        // 1、建立外部类对象
        Outer out = new Outer();
        //  2、根据外部类建立内部类对象
        Demo demo = out.new Demo();
        // 访问内部类方法
        demo.demo_funOne();
        //访问内部类字段
        System.out.println("内部类成员字段："+demo.j);
    }

    public static void main(String[] args) {
        //调用内部类的方法
        // 1、创建外部类对象
        Outer out = new Outer();
        // 2、通过外部类对象创建内部类对象
        Outer.Demo demo = out.new Demo();

        // 1、2步简写
        // Outer.Demo demo1 = new Outer().new Demo();

        //方法调用
        demo.demo_funOne();

    }

}
```



## **2、局部内部类**

　　　　在方法中定义的内部类称为局部内部类。与局部变量类似，局部内部类不能有访问说明符，因为它不是外围类的一部分，但是它可以访问当前代码块内的常量，和此外围类所有的成员。

需要注意的是：

　　(1)、局部内部类只能在定义该内部类的方法内实例化，不可以在此方法外对其实例化。

　　(2)、局部内部类对象不能使用该内部类所在方法的非final局部变量。

```
package com.test;

public class Outer {
    private static int number = 100;
    private int j = 20;
    private String name = "Java";

    //定义外部类方法
    public void outer_funOne(int k){
        final int number = 100;
        int j = 50;

        //方法内部的类（局部内部类）
        class Demo{
            public Demo(int k){
                demo_funOne(k);
            }

            int number = 300; //可以定义与外部类同名的变量
            // static int j = 10;  //不可以定义静态变量

            //内部类的方法
            public void demo_funOne(int k){
                System.out.println("内部类方法：demo_funOne");
                //访问外部类的变量，如果没有与内部类同名的变量，则可直接用变量名
                System.out.println(name);
                //访问外部类与内部类同名的变量
                System.out.println(Outer.this.number);
                System.out.println("内部类方法传入的参数是："+k);
            }
        }

        new Demo(k);
    }

    public static void main(String[] args) {
        //访问内部类必须要先有外部类对象
        Outer out = new Outer();
        out.outer_funOne(11);
    }

}
```



## **3、静态内部类（嵌套类）**

　　如果你不需要内部类对象与其外围类对象之间有联系，那你可以将内部类声明为static。这通常称为嵌套类（nested class）。想要理解static应用于内部类时的含义，你就必须记住，普通的内部类对象隐含地保存了一个引用，指向创建它的外围类对象。然而，当内部类是static的时，就不是这样了。嵌套类意味着：

　　1、 要创建嵌套类的对象，并不需要其外围类的对象。

​		2、不能从嵌套类的对象中访问非静态的外围类对象。



```
package com.test;

public class Outer {
    private static int number = 100;
    private int j = 20;
    private String name = "Java";

    public static void outer_funOne(){
        System.out.println("外部类静态方法:outer_funOne");
    }

    public void outer_funTwo(){

    }

    //静态内部类可以用public、protected、private修饰
    //静态内部类可以定义静态类或非静态内部类
    private static class Demo{
        static int j = 100;
        String name = "C#";

        //静态内部类里的静态方法
        static void demo_funOne(){
            //静态内部类只能访问外部类的静态成员（静态变量、静态方法）
            System.out.println("静态内部类访问外部类静态变量："+number);
            outer_funOne();//访问外部类静态方法

        }

        //静态内部类非静态方法
        void demo_funTwo(){

        }

    }

    public void outer_funThree(){
        //外部类访问内部类静态成员：内部类.静态成员
        System.out.println(Demo.j);
        //访问静态方法
        Demo.demo_funOne();
        //访问静态内部类的非静态成员，实例化内部类即可
        Demo demo = new Demo();
        //访问非静态变量
        System.out.println(demo.name);
        //访问非静态方法
        demo.demo_funTwo();

    }


    public static void main(String[] args) {
        new Outer().outer_funThree();
    }

}
```





## 4、匿名内部类

匿名内部类就是没有名字的内部类。什么情况下需要使用匿名内部类？如果满足下面的一些条件，使用匿名内部类是比较合适的：

　　1、只用到类的一个实例。

　　2、类在定义后马上用到。

　　3、类非常小（SUN推荐是在4行代码以下）

　　4、给类命名并不会导致你的代码更容易被理解。

在使用匿名内部类时，要记住以下几个原则：

　　1、　匿名内部类不能有构造方法。

　　2、　匿名内部类不能定义任何静态成员、方法和类。

　　3、　匿名内部类不能是public,protected,private,static。

　　4、　只能创建匿名内部类的一个实例。

　　5、  一个匿名内部类一定是在new的后面，用其隐含实现一个接口或实现一个类。

　　6、　因匿名内部类为局部内部类，所以局部内部类的所有限制都对其生效。



```JAVA
public class Demo {
    public static void main (String[] args){
        //我们要求是调用PersonDemo中的method方法
        PersonDemo pd = new PersonDemo();
        //这里要求我们传一个抽象类对象，但是抽象类对象不能够初始化。
        //所以要我们传一个匿名内部类进去,这里不能直接传抽象类对象，我们可以
        //传两种，要么传有名的子类对象，要么传匿名内部对象
//        Person p = new Student();
//        pd.method(p);

        //我们用匿名内部类来做一下--匿名内部类当做参数传递，把匿名内部类当做一个对象
        pd.method(new Person(){
              //重写show()方法
            public void show(){
                System.out.println("匿名内部类实现");
            }
        });

    }
}

//写个抽象类
abstract  class  Person{
    public  abstract void show();
}

//普通类
class PersonDemo {
    public  void method(Person p){
        p.show();
    }
}
//再创建个子类对象去继承抽象类
class Student extends Person {
    public void show(){
        System.out.println("我是一个有名的子类");
    }
}
```

