# 设计模式(三):传统模式VS简单原型模式及其优缺点

# 概念

原型模式是一个创建型的模式。用原型实例指定创建对象的种类，并且通过拷贝这些原型，创建新的对象。原型模式多用于创建复杂的或者构造耗时的实例，因为这种情况下，复制一个已经存在的实例可使程序运行更高效。原型模式是用于创建重复的对象，同时又能保证性能。，原型模式提供了一种创建对象的最佳方式。这种模式是实现了一个原型接口，该接口用于创建当前对象的克隆。当直接创建对象的代价比较大时，则采用这种模式。

# 工作原理

通过将一个原型对象传给那个要发动创建的对象，这个要发动创建的对象通过请求原型对象拷贝它们自己来实现创建，即 对象.clone()

传统方式克隆和原型模式比较

传统方式:

```
public class Person {
    //名字
    private String name;
    //年龄
    private int age;
    //肤色
    private String color;
    //性别
    private String sex;

    public Person(String name, int age, String color, String sex) {
        super();
        this.name = name;
        this.age = age;
        this.color = color;
        this.sex = sex;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", color='" + color + '\'' +
                ", sex=" + sex +
                '}';
    }
}

public class PrototypeDemo {
    public static void main(String[] args) {
        Person person = new Person("张三", 20, "黄色","男");
        Person person2 = new Person(person.getName(), person.getAge(), person.getColor(), person.getSex());
        Person person3 = new Person(person.getName(), person.getAge(), person.getColor(), person.getSex());
        System.out.println(person.toString());
        System.out.println(person2.toString());
        System.out.println(person3.toString());
    }
}

```

输出

```
Person{name='张三', age=12, color='黄色', sex=男}
Person{name='张三', age=12, color='黄色', sex=男}
Person{name='张三', age=12, color='黄色', sex=男}
123
```

优点：

好理解，简单易操作

缺点：

1.在创建新的对象时，总是需要重新获取原始对象的属性，如果创建的对象比较复杂时，效率较低。

2.总是需要重新初始化对象，而不是动态地获得对象运行时的状态，不够灵活。

简单原型模式方式:

```
public class Person2  implements Cloneable {
    //名字
    private String name;
    //年龄
    private int age;
    //肤色
    private String color;
    //性别
    private String sex;

    public Person2(String name, int age, String color, String sex) {
        super();
        this.name = name;
        this.age = age;
        this.color = color;
        this.sex = sex;
    }

    /**
     * 克隆该实例，使用默认的clone方法来完成
     */
    @Override
    protected Person2 clone() {
        Person2 person2 = null;
        try {
            person2 = (Person2)super.clone();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return person2;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }

    @Override
    public String toString() {
        return "Person2{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", color='" + color + '\'' +
                ", sex=" + sex +
                '}';
    }
}

public class PrototypeDemo2 {
    public static void main(String[] args) {
        Person2 person = new Person2("张三", 20, "黄色","男");
        Person2 person2 = person.clone();
        Person2 person3 = person.clone();
        System.out.println(person.toString());
        System.out.println(person2.toString());
        System.out.println(person3.toString());
    }
}
12345678910
```

输出：

```
Person2{name='张三', age=20, color='黄色', sex=男}
Person2{name='张三', age=20, color='黄色', sex=男}
Person2{name='张三', age=20, color='黄色', sex=男}
123
```

除了简单原型模式，对象的拷贝还有深拷贝，浅拷贝：具体看文章：

# [深拷贝和浅拷贝详解以及实例](https://blog.csdn.net/jinxinxin1314/article/details/105859326)

优点：
1.当创建新的对象实例较为复杂时，使用原型模式可以简化对象的创建过程，通过复制一个已有实例可以提高新实例的创建效率。
2.扩展性较好，由于在原型模式中提供了抽象原型类，在客户端可以针对抽象原型类进行编程，而将具体原型类写在配置文件中，增加或减少产品类对原有系统都没有任何影响。
3.原型模式提供了简化的创建结构，工厂方法模式常常需要有一个与产品类等级结构相同的工厂等级结构，而原型模式就不需要这样，原型模式中产品的复制是通过封装在原型类中的克隆方法实现的，无须专门的工厂类来创建产品。
4.可以使用深克隆的方式保存对象的状态，使用原型模式将对象复制一份并将其状态保存起来，以便在需要的时候使用（如恢复到某一历史状态），可辅助实现撤销操作。

5.性能提高。

6.逃避构造函数的约束。

缺点：
1.配备克隆方法需要对类的功能进行通盘考虑，这对于全新的类不是很难，但对于已有的类不一定很容易，特别当一个类引用不支持串行化的间接对象，或者引用含有循环结构的时候。
2.必须实现 Cloneable 接口。