- 1. Math.random() 静态方法
- 2. java.util.Random 工具类
- 3. java.util.concurrent.ThreadLocalRandom 工具类
- 4. java.Security.SecureRandom
- 5. 随机字符串

------

# 1. Math.random() 静态方法

产生的随机数是 0 - 1 之间的一个 `double`，即 `0 <= random <= 1`。

```
for (int i = 0; i < 10; i++) {
  System.out.println(Math.random());
}
```

结果：

> 0.3598613895606426
>
> 0.2666778145365811
>
> 0.25090731064243355
>
> 0.011064998061666276
>
> 0.600686228175639
>
> 0.9084006027629496
>
> 0.12700524654847833
>
> 0.6084605849069343
>
> 0.7290804782514261
>
> 0.9923831908303121

实现原理：

> When this method is first called, it creates a single new pseudorandom-number generator, exactly as if by the expression new java.util.Random() This new pseudorandom-number generator is used thereafter for all calls to this method and is used nowhere else.

当第一次调用 `Math.random()` 方法时，自动创建了一个**伪随机数生成器**，实际上用的是 `new java.util.Random()`。当接下来继续调用 `Math.random()` 方法时，就会使用这个新的**伪随机数生成器**。

源码如下：

```
public static double random() {
    Random rnd = randomNumberGenerator;
    if (rnd == null) rnd = initRNG(); // 第一次调用，创建一个伪随机数生成器
    return rnd.nextDouble();
}

private static synchronized Random initRNG() {
    Random rnd = randomNumberGenerator;
    return (rnd == null) ? (randomNumberGenerator = new Random()) : rnd; // 实际上用的是new java.util.Random()
}
```

> This method is properly synchronized to allow correct use by more than one thread. However, if many threads need to generate pseudorandom numbers at a great rate, it may reduce contention for each thread to have its own pseudorandom-number generator.

`initRNG()` 方法是 `synchronized` 的，因此在多线程情况下，只有一个线程会负责创建伪随机数生成器（使用当前时间作为种子），其他线程则利用该伪随机数生成器产生随机数。

因此 `Math.random()` 方法是线程安全的。

什么情况下随机数的生成线程不安全：

- 线程1在第一次调用 `random()` 时产生一个生成器 `generator1`，使用当前时间作为种子。
- 线程2在第一次调用 `random()` 时产生一个生成器 `generator2`，使用当前时间作为种子。
- 碰巧 `generator1` 和 `generator2` 使用相同的种子，导致 `generator1` 以后产生的随机数每次都和 `generator2` 以后产生的随机数相同。

什么情况下随机数的生成线程安全：`Math.random()` 静态方法使用

- 线程1在第一次调用 `random()` 时产生一个生成器 `generator1`，使用当前时间作为种子。
- 线程2在第一次调用 `random()` 时发现已经有一个生成器 `generator1`，则直接使用生成器 `generator1`。

```
public class JavaRandom {
    public static void main(String args[]) {
        new MyThread().start();
        new MyThread().start();
    }
}
class MyThread extends Thread {
    public void run() {
        for (int i = 0; i < 2; i++) {
            System.out.println(Thread.currentThread().getName() + ": " + Math.random());
        }
    }
}
```

结果：

> Thread-1: 0.8043581595645333
>
> Thread-0: 0.9338269554390357
>
> Thread-1: 0.5571569413128877
>
> Thread-0: 0.37484586843392464

# 2. java.util.Random 工具类

基本算法：linear congruential pseudorandom number generator (LGC) 线性同余法伪随机数生成器

缺点：可预测

> An attacker will simply compute the seed from the output values observed. This takes significantly less time than 2^48 in the case of java.util.Random. 从输出中可以很容易计算出种子值。
>
> It is shown that you can predict future Random outputs observing only two(!) output values in time roughly 2^16. 因此可以预测出下一个输出的随机数。
>
> You should never use an LCG for security-critical purposes. 在注重信息安全的应用中，不要使用 LCG 算法生成随机数，请使用 SecureRandom。

使用：

```
Random random = new Random();

for (int i = 0; i < 5; i++) {
    System.out.println(random.nextInt());
}
```

结果：

> -24520987
>
> -96094681
>
> -952622427
>
> 300260419
>
> 1489256498

Random类默认使用当前系统时钟作为种子:

```
public Random() {
    this(seedUniquifier() ^ System.nanoTime());
}

public Random(long seed) {
    if (getClass() == Random.class)
        this.seed = new AtomicLong(initialScramble(seed));
    else {
        // subclass might have overriden setSeed
        this.seed = new AtomicLong();
        setSeed(seed);
    }
}
```

Random类提供的方法：API

- nextBoolean() - 返回均匀分布的 true 或者 false
- nextBytes(byte[] bytes)
- nextDouble() - 返回 0.0 到 1.0 之间的均匀分布的 double
- nextFloat() - 返回 0.0 到 1.0 之间的均匀分布的 float
- nextGaussian()- 返回 0.0 到 1.0 之间的高斯分布（即正态分布）的 double
- nextInt() - 返回均匀分布的 int
- nextInt(int n) - 返回 0 到 n 之间的均匀分布的 int （包括 0，不包括 n）
- nextLong() - 返回均匀分布的 long
- setSeed(long seed) - 设置种子

只要种子一样，产生的随机数也一样：因为种子确定，随机数算法也确定，因此输出是确定的！

```
Random random1 = new Random(10000);
Random random2 = new Random(10000);

for (int i = 0; i < 5; i++) {
    System.out.println(random1.nextInt() + " = " + random2.nextInt());
}
```

结果：

> -498702880 = -498702880
>
> -858606152 = -858606152
>
> 1942818232 = 1942818232
>
> -1044940345 = -1044940345
>
> 1588429001 = 1588429001

# 3. java.util.concurrent.ThreadLocalRandom 工具类

`ThreadLocalRandom` 是 JDK 7 之后提供，也是继承至 java.util.Random。

```
private static final ThreadLocal<ThreadLocalRandom> localRandom =
    new ThreadLocal<ThreadLocalRandom>() {
        protected ThreadLocalRandom initialValue() {
            return new ThreadLocalRandom();
        }
};
```

每一个线程有一个独立的随机数生成器，用于并发产生随机数，能够解决多个线程发生的竞争争夺。效率更高！`ThreadLocalRandom` 不是直接用 `new` 实例化，而是第一次使用其静态方法 `current()`得到 `ThreadLocal<ThreadLocalRandom>` 实例，然后调用 `java.util.Random` 类提供的方法获得各种随机数。

使用：

```
public class JavaRandom {
    public static void main(String args[]) {
        new MyThread().start();
        new MyThread().start();
    }
}
class MyThread extends Thread {
    public void run() {
        for (int i = 0; i < 2; i++) {
            System.out.println(Thread.currentThread().getName() + ": " + ThreadLocalRandom.current().nextDouble());
        }
    }
}
```

结果：

> Thread-0: 0.13267085355389086
>
> Thread-1: 0.1138484950410098
>
> Thread-0: 0.17187774671469858
>
> Thread-1: 0.9305225910262372

# 4. java.Security.SecureRandom

也是继承至 java.util.Random。

> Instances of java.util.Random are not cryptographically secure. Consider instead using SecureRandom to get a cryptographically secure pseudo-random number generator for use by security-sensitive applications.
>
> SecureRandom takes Random Data from your os (they can be interval between keystrokes etc - most os collect these data store them in files - /dev/random and /dev/urandom in case of linux/solaris) and uses that as the seed.
>
> 操作系统收集了一些随机事件，比如鼠标点击，键盘点击等等，SecureRandom 使用这些随机事件作为种子。

`SecureRandom` 提供加密的强随机数生成器 (RNG)，要求种子必须是不可预知的，产生非确定性输出。

`SecureRandom` 也提供了与实现无关的算法，因此，调用方（应用程序代码）会请求特定的 RNG 算法并将它传回到该算法的 `SecureRandom` 对象中。

如果仅指定算法名称，如下所示：

```
SecureRandom random = SecureRandom.getInstance("SHA1PRNG");
```

如果既指定了算法名称又指定了包提供程序，如下所示：

```
SecureRandom random = SecureRandom.getInstance("SHA1PRNG", "SUN");
```

使用：

```
SecureRandom random1 = SecureRandom.getInstance("SHA1PRNG");
SecureRandom random2 = SecureRandom.getInstance("SHA1PRNG");

for (int i = 0; i < 5; i++) {
    System.out.println(random1.nextInt() + " != " + random2.nextInt());
}
```

结果：

> 704046703 != 2117229935
>
> 60819811 != 107252259
>
> 425075610 != -295395347
>
> 682299589 != -1637998900
>
> -1147654329 != 1418666937

# 5. 随机字符串

可以使用 Apache Commons-Lang 包中的 RandomStringUtils 类。Maven 依赖如下：

```
<dependency>
    <groupId>commons-lang</groupId>
    <artifactId>commons-lang</artifactId>
    <version>2.6</version>
</dependency>
```

API 参考：https://commons.apache.org/proper/commons-lang/javadocs/api-2.6/org/apache/commons/lang/RandomStringUtils.html

示例：

```
public class RandomStringDemo {
    public static void main(String[] args) {
        // Creates a 64 chars length random string of number.
        String result = RandomStringUtils.random(64, false, true);
        System.out.println("random = " + result);

        // Creates a 64 chars length of random alphabetic string.
        result = RandomStringUtils.randomAlphabetic(64);
        System.out.println("random = " + result);

        // Creates a 32 chars length of random ascii string.
        result = RandomStringUtils.randomAscii(32);
        System.out.println("random = " + result);

        // Creates a 32 chars length of string from the defined array of
        // characters including numeric and alphabetic characters.
        result = RandomStringUtils.random(32, 0, 20, true, true, "qw32rfHIJk9iQ8Ud7h0X".toCharArray());
        System.out.println("random = " + result);

    }
}
```

`RandomStringUtils` 类的实现上也是依赖了 `java.util.Random` 工具类：



```
public class RandomStringUtils {

    /**
     * <p>Random object used by random method. This has to be not local
     * to the random method so as to not return the same value in the 
     * same millisecond.</p>
     */
    private static final Random RANDOM = new Random();

    /**
     * <p><code>RandomStringUtils</code> instances should NOT be constructed in
     * standard programming. Instead, the class should be used as
     * <code>RandomStringUtils.random(5);</code>.</p>
     *
     * <p>This constructor is public to permit tools that require a JavaBean instance
     * to operate.</p>
     */
    public RandomStringUtils() {
      super();
    }
    
    。。。
```



来源:http://i7q.cn/4RS8he