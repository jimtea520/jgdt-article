# java枚举的底层实现原理

先定义一个手机操作系统类型枚举PhoneOsEnum：

```java
Copypackage club.throwable.enumeration;

public enum PhoneOsEnum {

	/**
	 * 安卓
	 */
	ANDROID(1, "android"),

	/**
	 * ios
	 */
	IOS(2, "ios");


	private final Integer type;
	private final String typeName;

	PhoneOsEnum(Integer type, String typeName) {
		this.type = type;
		this.typeName = typeName;
	}

	public Integer getType() {
		return type;
	}

	public String getTypeName() {
		return typeName;
	}
}
```

这是一个很简单的枚举，接着使用JDK的反编译工具反编译出其字节码，执行下面的命令：

```
Copyjavap -c -v D:\Projects\rxjava-seed\target\classes\club\throwable\enumeration\PhoneOsEnum.class
```

然后就得到了关于PhoneOsEnum.class的很长的字节码，这里全部贴出来：

```java
CopyClassfile /D:/Projects/rxjava-seed/target/classes/club/throwable/enumeration/PhoneOsEnum.class
  Last modified 2018-10-6; size 1561 bytes
  MD5 checksum 6d3186042f54233219000927a2f196aa
  Compiled from "PhoneOsEnum.java"
public final class club.throwable.enumeration.PhoneOsEnum extends java.lang.Enum<club.throwable.enumeration.PhoneOsEnum>
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_FINAL, ACC_SUPER, ACC_ENUM
Constant pool:
   #1 = Fieldref           #4.#49         // club/throwable/enumeration/PhoneOsEnum.$VALUES:[Lclub/throwable/enumeration/PhoneOsEnum;
   #2 = Methodref          #50.#51        // "[Lclub/throwable/enumeration/PhoneOsEnum;".clone:()Ljava/lang/Object;
   #3 = Class              #26            // "[Lclub/throwable/enumeration/PhoneOsEnum;"
   #4 = Class              #52            // club/throwable/enumeration/PhoneOsEnum
   #5 = Methodref          #17.#53        // java/lang/Enum.valueOf:(Ljava/lang/Class;Ljava/lang/String;)Ljava/lang/Enum;
   #6 = Methodref          #17.#54        // java/lang/Enum."<init>":(Ljava/lang/String;I)V
   #7 = Fieldref           #4.#55         // club/throwable/enumeration/PhoneOsEnum.type:Ljava/lang/Integer;
   #8 = Fieldref           #4.#56         // club/throwable/enumeration/PhoneOsEnum.typeName:Ljava/lang/String;
   #9 = String             #18            // ANDROID
  #10 = Methodref          #57.#58        // java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
  #11 = String             #59            // android
  #12 = Methodref          #4.#60         // club/throwable/enumeration/PhoneOsEnum."<init>":(Ljava/lang/String;ILjava/lang/Integer;Ljava/lang/String;)V
  #13 = Fieldref           #4.#61         // club/throwable/enumeration/PhoneOsEnum.ANDROID:Lclub/throwable/enumeration/PhoneOsEnum;
  #14 = String             #20            // IOS
  #15 = String             #62            // ios
  #16 = Fieldref           #4.#63         // club/throwable/enumeration/PhoneOsEnum.IOS:Lclub/throwable/enumeration/PhoneOsEnum;
  #17 = Class              #64            // java/lang/Enum
  #18 = Utf8               ANDROID
  #19 = Utf8               Lclub/throwable/enumeration/PhoneOsEnum;
  #20 = Utf8               IOS
  #21 = Utf8               type
  #22 = Utf8               Ljava/lang/Integer;
  #23 = Utf8               typeName
  #24 = Utf8               Ljava/lang/String;
  #25 = Utf8               $VALUES
  #26 = Utf8               [Lclub/throwable/enumeration/PhoneOsEnum;
  #27 = Utf8               values
  #28 = Utf8               ()[Lclub/throwable/enumeration/PhoneOsEnum;
  #29 = Utf8               Code
  #30 = Utf8               LineNumberTable
  #31 = Utf8               valueOf
  #32 = Utf8               (Ljava/lang/String;)Lclub/throwable/enumeration/PhoneOsEnum;
  #33 = Utf8               LocalVariableTable
  #34 = Utf8               name
  #35 = Utf8               <init>
  #36 = Utf8               (Ljava/lang/String;ILjava/lang/Integer;Ljava/lang/String;)V
  #37 = Utf8               this
  #38 = Utf8               Signature
  #39 = Utf8               (Ljava/lang/Integer;Ljava/lang/String;)V
  #40 = Utf8               getType
  #41 = Utf8               ()Ljava/lang/Integer;
  #42 = Utf8               getTypeName
  #43 = Utf8               ()Ljava/lang/String;
  #44 = Utf8               <clinit>
  #45 = Utf8               ()V
  #46 = Utf8               Ljava/lang/Enum<Lclub/throwable/enumeration/PhoneOsEnum;>;
  #47 = Utf8               SourceFile
  #48 = Utf8               PhoneOsEnum.java
  #49 = NameAndType        #25:#26        // $VALUES:[Lclub/throwable/enumeration/PhoneOsEnum;
  #50 = Class              #26            // "[Lclub/throwable/enumeration/PhoneOsEnum;"
  #51 = NameAndType        #65:#66        // clone:()Ljava/lang/Object;
  #52 = Utf8               club/throwable/enumeration/PhoneOsEnum
  #53 = NameAndType        #31:#67        // valueOf:(Ljava/lang/Class;Ljava/lang/String;)Ljava/lang/Enum;
  #54 = NameAndType        #35:#68        // "<init>":(Ljava/lang/String;I)V
  #55 = NameAndType        #21:#22        // type:Ljava/lang/Integer;
  #56 = NameAndType        #23:#24        // typeName:Ljava/lang/String;
  #57 = Class              #69            // java/lang/Integer
  #58 = NameAndType        #31:#70        // valueOf:(I)Ljava/lang/Integer;
  #59 = Utf8               android
  #60 = NameAndType        #35:#36        // "<init>":(Ljava/lang/String;ILjava/lang/Integer;Ljava/lang/String;)V
  #61 = NameAndType        #18:#19        // ANDROID:Lclub/throwable/enumeration/PhoneOsEnum;
  #62 = Utf8               ios
  #63 = NameAndType        #20:#19        // IOS:Lclub/throwable/enumeration/PhoneOsEnum;
  #64 = Utf8               java/lang/Enum
  #65 = Utf8               clone
  #66 = Utf8               ()Ljava/lang/Object;
  #67 = Utf8               (Ljava/lang/Class;Ljava/lang/String;)Ljava/lang/Enum;
  #68 = Utf8               (Ljava/lang/String;I)V
  #69 = Utf8               java/lang/Integer
  #70 = Utf8               (I)Ljava/lang/Integer;
{
  public static final club.throwable.enumeration.PhoneOsEnum ANDROID;
    descriptor: Lclub/throwable/enumeration/PhoneOsEnum;
    flags: ACC_PUBLIC, ACC_STATIC, ACC_FINAL, ACC_ENUM

  public static final club.throwable.enumeration.PhoneOsEnum IOS;
    descriptor: Lclub/throwable/enumeration/PhoneOsEnum;
    flags: ACC_PUBLIC, ACC_STATIC, ACC_FINAL, ACC_ENUM

  public static club.throwable.enumeration.PhoneOsEnum[] values();
    descriptor: ()[Lclub/throwable/enumeration/PhoneOsEnum;
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=1, locals=0, args_size=0
         0: getstatic     #1                  // Field $VALUES:[Lclub/throwable/enumeration/PhoneOsEnum;
         3: invokevirtual #2                  // Method "[Lclub/throwable/enumeration/PhoneOsEnum;".clone:()Ljava/lang/Object;
         6: checkcast     #3                  // class "[Lclub/throwable/enumeration/PhoneOsEnum;"
         9: areturn
      LineNumberTable:
        line 9: 0

  public static club.throwable.enumeration.PhoneOsEnum valueOf(java.lang.String);
    descriptor: (Ljava/lang/String;)Lclub/throwable/enumeration/PhoneOsEnum;
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=1, args_size=1
         0: ldc           #4                  // class club/throwable/enumeration/PhoneOsEnum
         2: aload_0
         3: invokestatic  #5                  // Method java/lang/Enum.valueOf:(Ljava/lang/Class;Ljava/lang/String;)Ljava/lang/Enum;
         6: checkcast     #4                  // class club/throwable/enumeration/PhoneOsEnum
         9: areturn
      LineNumberTable:
        line 9: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      10     0  name   Ljava/lang/String;

  public java.lang.Integer getType();
    descriptor: ()Ljava/lang/Integer;
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: getfield      #7                  // Field type:Ljava/lang/Integer;
         4: areturn
      LineNumberTable:
        line 31: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lclub/throwable/enumeration/PhoneOsEnum;

  public java.lang.String getTypeName();
    descriptor: ()Ljava/lang/String;
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: getfield      #8                  // Field typeName:Ljava/lang/String;
         4: areturn
      LineNumberTable:
        line 35: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lclub/throwable/enumeration/PhoneOsEnum;

  static {};
    descriptor: ()V
    flags: ACC_STATIC
    Code:
      stack=6, locals=0, args_size=0
         0: new           #4                  // class club/throwable/enumeration/PhoneOsEnum
         3: dup
         4: ldc           #9                  // String ANDROID
         6: iconst_0
         7: iconst_1
         8: invokestatic  #10                 // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
        11: ldc           #11                 // String android
        13: invokespecial #12                 // Method "<init>":(Ljava/lang/String;ILjava/lang/Integer;Ljava/lang/String;)V
        16: putstatic     #13                 // Field ANDROID:Lclub/throwable/enumeration/PhoneOsEnum;
        19: new           #4                  // class club/throwable/enumeration/PhoneOsEnum
        22: dup
        23: ldc           #14                 // String IOS
        25: iconst_1
        26: iconst_2
        27: invokestatic  #10                 // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
        30: ldc           #15                 // String ios
        32: invokespecial #12                 // Method "<init>":(Ljava/lang/String;ILjava/lang/Integer;Ljava/lang/String;)V
        35: putstatic     #16                 // Field IOS:Lclub/throwable/enumeration/PhoneOsEnum;
        38: iconst_2
        39: anewarray     #4                  // class club/throwable/enumeration/PhoneOsEnum
        42: dup
        43: iconst_0
        44: getstatic     #13                 // Field ANDROID:Lclub/throwable/enumeration/PhoneOsEnum;
        47: aastore
        48: dup
        49: iconst_1
        50: getstatic     #16                 // Field IOS:Lclub/throwable/enumeration/PhoneOsEnum;
        53: aastore
        54: putstatic     #1                  // Field $VALUES:[Lclub/throwable/enumeration/PhoneOsEnum;
        57: return
      LineNumberTable:
        line 14: 0
        line 19: 19
        line 9: 38
}
Signature: #46                          // Ljava/lang/Enum<Lclub/throwable/enumeration/PhoneOsEnum;>;
SourceFile: "PhoneOsEnum.java"
```

先看类的签名是`public final class club.throwable.enumeration.PhoneOsEnum extends java.lang.Enum<club.throwable.enumeration.PhoneOsEnum>`，它的父类是java.lang.Enum，父类的泛型就是自身club.throwable.enumeration.PhoneOsEnum。上面的字节码的可读性相对比较低，直接翻译为Java代码(当然我们不能声明一个类直接继承java.lang.Enum，这里仅仅为了说明反编译后的枚举类的原型)如下：

```java
Copypublic final class PhoneOsEnumeration extends Enum<PhoneOsEnumeration> {

	public PhoneOsEnumeration(String name, int ordinal, Integer type, String typeName) {
		super(name, ordinal);
		this.type = type;
		this.typeName = typeName;
	}

	public Integer getType() {
		return type;
	}

	public String getTypeName() {
		return typeName;
	}

	public static PhoneOsEnumeration[] values() {
		return $VALUES.clone();
	}

	public static PhoneOsEnumeration valueOf(String name) {
		return Enum.valueOf(PhoneOsEnumeration.class, name);
	}
	
	private final Integer type;
	private final String typeName;
	public static final PhoneOsEnumeration ANDROID;
	public static final PhoneOsEnumeration IOS;
	private static final PhoneOsEnumeration[] $VALUES;

	static {
		ANDROID = new PhoneOsEnumeration("ANDROID", 0, 1, "android");
		IOS = new PhoneOsEnumeration("IOS", 1, 2, "ios");
		$VALUES = new PhoneOsEnumeration[]{ANDROID, IOS};
	}
}
```

概括来说就是成员变量都是通过静态代码块声明，这里注意一点父类Enum实例化的时候需要覆盖父类构造器`protected Enum(String name, int ordinal)`，其他方法的实现都是十分简单。

# JDK的枚举描述

总结一下重要内容有以下几点：

- 枚举的声明格式是：`{ClassModifier} enum Identifier [Superinterfaces] EnumBody`，ClassModifier是修饰符，Identifier是枚举的名称可以类比为类名，枚举类型可以实现接口。
- 枚举类型不能使用abstract或者final修饰，否则会产生编译错误。
- 枚举类型的直接超类是java.lang.Enum。
- 枚举类型除了枚举常量定义之外没有其他实例，也就是枚举类型不能实例化。
- 枚举类型禁用反射操作进行实例化(这个特性就是Effetive Java中推荐使用枚举实现单例的原因)。

枚举的公共父类java.lang.Enum的源码如下(已经去掉全部注释)：

```java
Copypublic abstract class Enum<E extends Enum<E>>
        implements Comparable<E>, Serializable {

    private final String name; 

    public final String name() {
        return name;
    } 

    private final int ordinal;

    public final int ordinal() {
        return ordinal;
    }

    protected Enum(String name, int ordinal) {
        this.name = name;
        this.ordinal = ordinal;
    }

    public String toString() {
        return name;
    }

    public final boolean equals(Object other) {
        return this==other;
    }

    public final int hashCode() {
        return super.hashCode();
    } 

    protected final Object clone() throws CloneNotSupportedException {
        throw new CloneNotSupportedException();
    }  

    public final int compareTo(E o) {
        Enum<?> other = (Enum<?>)o;
        Enum<E> self = this;
        if (self.getClass() != other.getClass() && // optimization
            self.getDeclaringClass() != other.getDeclaringClass())
            throw new ClassCastException();
        return self.ordinal - other.ordinal;
    }

    public final Class<E> getDeclaringClass() {
        Class<?> clazz = getClass();
        Class<?> zuper = clazz.getSuperclass();
        return (zuper == Enum.class) ? (Class<E>)clazz : (Class<E>)zuper;
    } 

    public static <T extends Enum<T>> T valueOf(Class<T> enumType,
                                                String name) {
        T result = enumType.enumConstantDirectory().get(name);
        if (result != null)
            return result;
        if (name == null)
            throw new NullPointerException("Name is null");
        throw new IllegalArgumentException(
            "No enum constant " + enumType.getCanonicalName() + "." + name);
    }  

    protected final void finalize() { }

    private void readObject(ObjectInputStream in) throws IOException,
        ClassNotFoundException {
        throw new InvalidObjectException("can't deserialize enum");
    }

    private void readObjectNoData() throws ObjectStreamException {
        throw new InvalidObjectException("can't deserialize enum");
    }                              
}            
```

大部分方法都比较简单，值得注意的几点是：

- 1、`valueOf`方法依赖到的`Class<?>#enumConstantDirectory()`，这个方法首次调用完成之后，结果会缓存在`Class<?>#enumConstantDirectory`变量中。
- 2、Enum实现了Serializable接口，但是`readObject`和`readObjectNoData`直接抛出了InvalidObjectException异常，注释说到是"防止默认的反序列化"，这一点有点不明不白，既然禁用反序列化为何要实现Serializable接口，这里可能考虑到是否实现Serializable接口应该交给开发者决定。
- 3、Enum禁用克隆。

# 小结

枚举本质上是通过普通的类来实现的，只是编译器为我们进行了处理。每个枚举类型都继承自java.lang.Enum，并自动添加了values和valueOf方法。而每个枚举常量是一个静态常量字段，使用内部类实现，该内部类继承了枚举类。所有枚举常量都通过静态代码块来进行初始化，即在类加载期间就初始化。另外通过把clone、readObject、writeObject这三个方法定义为final的，同时实现是抛出相应的异常。这样保证了每个枚举类型及枚举常量都是不可变的。可以利用枚举的这两个特性来实现线程安全的单例。