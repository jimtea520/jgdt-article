## 深拷贝 vs 浅拷贝

# 浅拷贝

## 概念

复制基本类型的属性；引用类型的属性复制，复制栈中的变量 和 变量指向堆内存中的对象的指针，不复制堆内存中的对象。

如图：

​	

## 	特点

​		1.对于基本数据类型的成员对象，因为基础数据类型是值传递的，所以是直接将属性值赋值给新的对象。基础类型的拷贝，其中一个对象修改该值，不会影响另外一个。
 		2.对于引用类型，比如数组或者类对象，因为引用类型是引用传递，所以浅拷贝只是把内存地址赋值给了成员变量，它们指向了同一内存空间。改变其中一个，会对另外一个也产生影响。

​		

## 	实现

​		实现 Cloneable 接口，重写 clone() 方法



```
/**
 * @ClassName ShallowCopyDemo
 * @Description: 实现 Cloneable 接口，重写 clone() 方法。
 * @Author fking
 * @Date 2020/4/30 0030
 * @Version V1.0
 **/
public class ShallowCopyDemo {
    public static void main(String[] args) throws CloneNotSupportedException {
        Person p1 = new Person(1, "fking01");//创建对象 Person p1
        Person p2 = (Person)p1.clone();//克隆对象 p1
        p2.setName("fking02");//修改 p2的name属性，p1的name未变
        System.out.println(p1);
        System.out.println(p2);

        p2 = p1;
        p2.setName("fking02");//修改 p2的name属性，p1的也跟着变
        System.out.println(p1);
        System.out.println(p2);
    }
}

class Person implements Cloneable {

    private int pid;

    private String name;

    public Person(int pid, String name) {
        this.pid = pid;
        this.name = name;
        System.out.println("Person constructor call");
    }

    public int getPid() {
        return pid;
    }

    public void setPid(int pid) {
        this.pid = pid;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }

    @Override
    public String toString() {
        return "Person [pid:" + pid + ", name:" + name + "]";
    }
}
```



# 深拷贝

## 	概念

​		复制基本类型的属性；引用类型的属性复制，复制栈中的变量 和 变量指向堆内存中的对象的指针和堆内存中的对象。

![img](https://img-blog.csdnimg.cn/20190618154423857.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L21laXNtNQ==,size_16,color_FFFFFF,t_70)

## 	特点

​		1.对于基本数据类型的成员对象，因为基础数据类型是值传递的，所以是直接将属性值赋值给新的对象。基础类型的拷贝，其中一个对象修改该值，不会影响另外一个（和浅拷贝一样）。

​		2.对于引用类型，比如数组或者类对象，深拷贝会新建一个对象空间，然后拷贝里面的内容，所以它们指向了不同的内存空间。改变其中一个，不会对另外一个也产生影响。

​		3.对于有多层对象的，每个对象都需要实现 `Cloneable` 并重写 `clone()` 方法，进而实现了对象的串行层层拷贝。

​		4.深拷贝相比于浅拷贝速度较慢并且花销较大。



## 		实现方式

​			1.对象的属性的Class 也实现 Cloneable 接口，在克隆对象时也手动克隆属性

```
/**
 * @ClassName DeepCopyDemo01
 * @Description: 对象的属性的Class 也实现 Cloneable 接口，在克隆对象时也手动克隆属性。
 * @Author fking
 * @Date 2020/4/30 0030
 * @Version V1.0
 **/
public class DeepCopyDemo01 {
    public static void main(String[] args) throws CloneNotSupportedException {
        DPerson01 p1 = new DPerson01(1, "fking01", new DFood01("food01"));//创建Person 对象 p1
        DPerson01 p2 = (DPerson01)p1.clone();//克隆p1
        p2.setName("fking02");//修改p2的name属性
        p2.getFood().setName("food02");//修改p2的自定义引用类型 food 属性
        System.out.println(p1);//修改p2的自定义引用类型 food 属性被改变，p1的自定义引用类型 food 属性也随之改变，说明p2的food属性，只拷贝了引用，没有拷贝food对象
        System.out.println(p2);
    }
}

class DPerson01 implements Cloneable {

    private int pid;

    private String name;

    private DFood01 food;

    public DPerson01(int pid, String name, DFood01 food) {
        this.pid = pid;
        this.name = name;
        this.food = food;
        System.out.println("DPerson01 constructor call");
    }

    public int getPid() {
        return pid;
    }

    public void setPid(int pid) {
        this.pid = pid;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    /**
     * 对象的属性的Class 也实现 Cloneable 接口，在克隆对象时也手动克隆属性。
     * @return
     * @throws CloneNotSupportedException
     */
    @Override
    public Object clone() throws CloneNotSupportedException {
        DPerson01 p = (DPerson01)super.clone();
        p.setFood((DFood01)p.getFood().clone());
        return p;
    }

    @Override
    public String toString() {
        return "DPerson01 [pid:"+pid+", name:"+name+", food:"+food.getName()+"]";
    }

    public DFood01 getFood() {
        return food;
    }

    public void setFood(DFood01 food) {
        this.food = food;
    }

}

class DFood01 implements Cloneable {

    private String name;

    public DFood01(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}
```

​		

​		2.结合序列化(JDK java.io.Serializable 接口、JSON格式、XML格式等)，完成深拷贝

```
/**
 * @ClassName DeepCopyDemo02
 * @Description: 结合序列化(JDK java.io.Serializable 接口、JSON格式、XML格式等)，完成深拷贝
 * @Author fking
 * @Date 2020/4/30 0030
 * @Version V1.0
 **/
public class DeepCopyDemo02 {
    public static void main(String[] args) throws CloneNotSupportedException {
        DPerson02 p1 = new DPerson02(1, "fking01", new SFood02("food01"));//创建 DPerson02 对象 p1
        DPerson02 p2 = (DPerson02)p1.cloneBySerializable();//克隆 p1
        p2.setName("fking02");//修改 p2 的 name 属性
        p2.getFood().setName("food02");//修改 p2 的自定义引用类型 food 属性
        System.out.println(p1);//修改 p2 的自定义引用类型 food 属性被改变，p1的自定义引用类型 food 属性未随之改变，说明p2的food属性，只拷贝了引用和 food 对象
        System.out.println(p2);
    }
}

class DPerson02 implements Cloneable, Serializable {

    private static final long serialVersionUID = -7710144514831611031L;

    private int pid;

    private String name;

    private SFood02 food;

    public DPerson02(int pid, String name, SFood02 food) {
        this.pid = pid;
        this.name = name;
        this.food = food;
        System.out.println("DPerson02 constructor call");
    }

    public int getPid() {
        return pid;
    }

    public void setPid(int pid) {
        this.pid = pid;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    /**
     * 通过序列化完成克隆
     * @return
     */
    public Object cloneBySerializable() {
        Object obj = null;
        try {
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            ObjectOutputStream oos = new ObjectOutputStream(baos);
            oos.writeObject(this);
            ByteArrayInputStream bais = new ByteArrayInputStream(baos.toByteArray());
            ObjectInputStream ois = new ObjectInputStream(bais);
            obj = ois.readObject();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return obj;
    }

    @Override
    public String toString() {
        return "DPerson02 [pid:"+pid+", name:"+name+", food:"+food.getName()+"]";
    }

    public SFood02 getFood() {
        return food;
    }

    public void setFood(SFood02 food) {
        this.food = food;
    }

}

class SFood02 implements Serializable {

    private static final long serialVersionUID = -3443815804346831432L;

    private String name;

    public SFood02(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

