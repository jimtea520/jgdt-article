# BigDecimal详细解析

## 简介

​		BigDecimal 由任意精度的整数非标度值 和32 位的整数标度 (scale) 组成。如果为零或正数，则标度是小数点后的位数。如果为负数，则将该数的非标度值乘以 10 的负scale 次幂。因此，BigDecimal表示的数值是(unscaledValue × 10-scale)。

先运行一套代码

```
public static void main(String[] args)
    {
        System.out.println(0.2 + 0.1);
        System.out.println(0.3 - 0.1);
        System.out.println(0.2 * 0.1);
        System.out.println(0.3 / 0.1);
    }
```

输出效果

```
0.30000000000000004
0.19999999999999998
0.020000000000000004
2.9999999999999996
```

原因

我们的计算机是二进制的。浮点数没有办法是用二进制进行精确表示。我们的CPU表示浮点数由两个部分组成：指数和尾数，这样的表示方法一般都会失去一定的精确度，有些浮点数运算也会产生一定的误差。如：2.4的二进制表示并非就是精确的2.4。反而最为接近的二进制表示是 2.3999999999999999。浮点数的值实际上是由一个特定的数学公式计算得到的。

## 构造方法

　　1.public BigDecimal(double val)   将double表示形式转换为BigDecimal *不建议使用

　　2.public BigDecimal(int val)　　将int表示形式转换成BigDecimal

　　3.public BigDecimal(String val)　　将String表示形式转换成BigDecimal

示例

```
BigDecimal bigDecimal = new BigDecimal(2);
BigDecimal bDouble = new BigDecimal(2.4);
BigDecimal bString = new BigDecimal("2.4");
System.out.println("bigDecimal=" + bigDecimal);
System.out.println("bDouble=" + bDouble);
System.out.println("bString=" + bString);
```

输出结果

```
bigDecimal=2
bDouble=2.399999999999999911182158029987476766109466552734375
bString=2.4
```

JDK的描述(原因)

1、参数类型为double的构造方法的结果有一定的不可预知性。有人可能认为在Java中写入newBigDecimal(0.1)所创建的BigDecimal正好等于 0.1（非标度值 1，其标度为 1），但是它实际上等于0.1000000000000000055511151231257827021181583404541015625。这是因为0.1无法准确地表示为 double（或者说对于该情况，不能表示为任何有限长度的二进制小数）。这样，传入到构造方法的值不会正好等于 0.1（虽然表面上等于该值）。

​    2、另一方面，String 构造方法是完全可预知的：写入 newBigDecimal("0.1") 将创建一个 BigDecimal，它正好等于预期的 0.1。因此，比较而言，**通常建议优先使用String构造方法**。

**如果是确定用double，则可以用Double.toString(2.4),如下代码**

```
BigDecimal bDouble1 = BigDecimal.valueOf(2.4);
BigDecimal bDouble2 = new BigDecimal(Double.toString(2.4));

System.out.println("bDouble1=" + bDouble1);
System.out.println("bDouble2=" + bDouble2);
```

输出结果

```
bDouble1=2.4
bDouble2=2.4
```

## 几种常用函数

1. 加：BigDecimal add(BigDecimal augend) 

2. 减：BigDecimal subtract(BigDecimal subtrahend)

3. 乘：BigDecimal multiply(BigDecimal multiplicand)

4. 除法:BigDecimal divide(BigDecimal divisor, int scale, int roundingMode) 

   （BigDecimal  divisor 除数， int scale 精确小数位，  int roundingMode 舍入模式）

5. 绝对值:BigDecimal abs()

例子代码

```
BigDecimal num1 = new BigDecimal(2.4);
BigDecimal num2 = new BigDecimal(2);
BigDecimal num3 = new BigDecimal(-2);
//尽量用字符串的形式初始化
BigDecimal num12 = new BigDecimal("2.4");
BigDecimal num22 = new BigDecimal("2");
BigDecimal num32 = new BigDecimal("-2");

//加法
BigDecimal result1 = num1.add(num2);
BigDecimal result12 = num12.add(num22);
//减法
BigDecimal result2 = num1.subtract(num2);
BigDecimal result22 = num12.subtract(num22);
//乘法
BigDecimal result3 = num1.multiply(num2);
BigDecimal result32 = num12.multiply(num22);
//绝对值
BigDecimal result4 = num3.abs();
BigDecimal result42 = num32.abs();
//除法
BigDecimal result5 = num2.divide(num1,20,BigDecimal.ROUND_HALF_UP);
BigDecimal result52 = num22.divide(num12,20,BigDecimal.ROUND_HALF_UP);

System.out.println("加法用value结果 result1："+result1);
System.out.println("加法用string结果 result12："+result12);

System.out.println("减法value结果："+result2);
System.out.println("减法用string结果："+result22);

System.out.println("乘法用value结果 result3："+result3);
System.out.println("乘法用string结果 result32："+result32);

System.out.println("绝对值用value结果 result4："+result4);
System.out.println("绝对值用string结果 result42："+result42);

System.out.println("除法用value结果 result5："+result5);
System.out.println("除法用string结果 result52："+result52);
```

输出结果

```
加法用value结果 result1：4.399999999999999911182158029987476766109466552734375
加法用string结果 result12：4.4
减法value结果：0.399999999999999911182158029987476766109466552734375
减法用string结果：0.4
乘法用value结果 result3：4.799999999999999822364316059974953532218933105468750
乘法用string结果 result32：4.8
绝对值用value结果 result4：2
绝对值用string结果 result42：2
除法用value结果 result5：0.83333333333333336417
除法用string结果 result52：0.83333333333333333333
```

## 除法的八种舍入模式

#### 1、ROUND_UP

舍入远离零的舍入模式。

在丢弃非零部分之前始终增加数字(始终对非零舍弃部分前面的数字加1)。

注意，此舍入模式始终不会减少计算值的大小。

#### 2、ROUND_DOWN

接近零的舍入模式。

在丢弃某部分之前始终不增加数字(从不对舍弃部分前面的数字加1，即截短)。

注意，此舍入模式始终不会增加计算值的大小。

#### 3、ROUND_CEILING

接近正无穷大的舍入模式。

如果 BigDecimal 为正，则舍入行为与 ROUND_UP 相同;

如果为负，则舍入行为与 ROUND_DOWN 相同。

注意，此舍入模式始终不会减少计算值。

#### 4、ROUND_FLOOR

接近负无穷大的舍入模式。

如果 BigDecimal 为正，则舍入行为与 ROUND_DOWN 相同;

如果为负，则舍入行为与 ROUND_UP 相同。

注意，此舍入模式始终不会增加计算值。

#### 5、ROUND_HALF_UP

向“最接近的”数字舍入，如果与两个相邻数字的距离相等，则为向上舍入的舍入模式。

如果舍弃部分 >= 0.5，则舍入行为与 ROUND_UP 相同;否则舍入行为与 ROUND_DOWN 相同。

注意，这是我们大多数人在小学时就学过的舍入模式(四舍五入)。

#### 6、ROUND_HALF_DOWN

向“最接近的”数字舍入，如果与两个相邻数字的距离相等，则为上舍入的舍入模式。

如果舍弃部分 > 0.5，则舍入行为与 ROUND_UP 相同;否则舍入行为与 ROUND_DOWN 相同(五舍六入)。

#### 7、ROUND_HALF_EVEN

向“最接近的”数字舍入，如果与两个相邻数字的距离相等，则向相邻的偶数舍入。

如果舍弃部分左边的数字为奇数，则舍入行为与 ROUND_HALF_UP 相同;

如果为偶数，则舍入行为与 ROUND_HALF_DOWN 相同。

注意，在重复进行一系列计算时，此舍入模式可以将累加错误减到最小。

此舍入模式也称为“银行家舍入法”，主要在美国使用。四舍六入，五分两种情况。

如果前一位为奇数，则入位，否则舍去。

以下例子为保留小数点1位，那么这种舍入方式下的结果。

1.15>1.2 1.25>1.2

#### 8、ROUND_UNNECESSARY

断言请求的操作具有精确的结果，因此不需要舍入。

示例

```
BigDecimal num1 = new BigDecimal("2.4");
BigDecimal num2 = new BigDecimal("2");

BigDecimal result1 = num2.divide(num1,3,BigDecimal.ROUND_UP);
BigDecimal result2 = num2.divide(num1,3,BigDecimal.ROUND_DOWN);
BigDecimal result3 = num2.divide(num1,3,BigDecimal.ROUND_CEILING);
BigDecimal result4 = num2.divide(num1,3,BigDecimal.ROUND_FLOOR);
BigDecimal result5 = num2.divide(num1,3,BigDecimal.ROUND_HALF_UP);
BigDecimal result6 = num2.divide(num1,3,BigDecimal.ROUND_HALF_DOWN);
BigDecimal result7 = num2.divide(num1,3,BigDecimal.ROUND_HALF_EVEN);

System.out.println("BigDecimal.ROUND_UP result1："+result1);
System.out.println("BigDecimal.ROUND_DOWN result2："+result2);
System.out.println("BigDecimal.ROUND_CEILING result3："+result3);
System.out.println("BigDecimal.ROUND_FLOOR result4："+result4);
System.out.println("BigDecimal.ROUND_HALF_UP result5："+result5);
System.out.println("BigDecimal.ROUND_HALF_DOWN result6："+result6);
System.out.println("BigDecimal.ROUND_HALF_EVEN result7："+result7);
```

输出结果

```
BigDecimal.ROUND_UP result1：0.834
BigDecimal.ROUND_DOWN result2：0.833
BigDecimal.ROUND_CEILING result3：0.834
BigDecimal.ROUND_FLOOR result4：0.833
BigDecimal.ROUND_HALF_UP result5：0.833
BigDecimal.ROUND_HALF_DOWN result6：0.833
BigDecimal.ROUND_HALF_EVEN result7：0.833
```

这里要注意ROUND_UNNECESSARY如果相除是无限小数会报异常

```
BigDecimal num1 = new BigDecimal("2.4");
BigDecimal num2 = new BigDecimal("2");
BigDecimal result8 = num2.divide(num1,2,BigDecimal.ROUND_UNNECESSARY);
System.out.println("BigDecimal.ROUND_UNNECESSARY result8："+result8);
```

输出结果

```
Exception in thread "main" java.lang.ArithmeticException: Rounding necessary
	at java.math.BigDecimal.commonNeedIncrement(BigDecimal.java:4179)
	at java.math.BigDecimal.needIncrement(BigDecimal.java:4235)
	at java.math.BigDecimal.divideAndRound(BigDecimal.java:4143)
	at java.math.BigDecimal.divide(BigDecimal.java:5214)
	at java.math.BigDecimal.divide(BigDecimal.java:1564)
	at fking.Demo5.main(Demo5.java:35)
```

如果改成

```
BigDecimal num1 = new BigDecimal("2");
BigDecimal num2 = new BigDecimal("5");
BigDecimal result8 = num2.divide(num1,2,BigDecimal.ROUND_UNNECESSARY);
System.out.println("BigDecimal.ROUND_UNNECESSARY result8："+result8);
```

输出结果

```
BigDecimal.ROUND_UNNECESSARY result8：2.50
```



## 总结结论

   (1)商业计算使用BigDecimal。

​    (2)尽量使用参数类型为String的构造函数。

​    (3) BigDecimal都是不可变的（immutable）的，在进行每一步运算时，都会产生一个新的对象，所以在做加减乘除运算时千万要保存操作后的值。

