# Java创建多线程的四种方式



### java有以下四种创建多线程的方式：

- **1:继承Thread类创建线程**

- **2:实现Runnable接口创建线程**

- **3:使用Callable和FutureTask创建线程**

- **4:使用线程池，例如用Executor框架创建线程**

  

### DEMO代码：

```java
package thread;
 
import java.util.concurrent.*;
 

public class ThreadTest {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
//      创建线程的第一种方法
        Thread1 thread1 = new Thread1();
        thread1.start();
 
//      创建线程的第二种方法
        Thread2 thread2 = new Thread2();
        Thread thread = new Thread(thread2);
        thread.start();
 
//      创建线程的第三种方法
        Callable<String> callable = new Thread3();
        FutureTask<String> futureTask = new FutureTask<>(callable);
        Thread thread3 = new Thread(futureTask);
        thread3.start();
        String s = futureTask.get();
        System.out.println(s);
 
//      创建线程的第四种方法
        Executor executor = Executors.newFixedThreadPool(5);
        executor.execute(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread()+"创建线程的第四种方法");
            }
        });
        ((ExecutorService) executor).shutdown();
 
    }
}
class Thread1 extends Thread{
    @Override
    public void run() {
        System.out.println(Thread.currentThread()+"创建线程的第一种方法");
    }
}
 
class Thread2 implements Runnable {
 
    @Override
    public void run() {
        System.out.println(Thread.currentThread()+"创建线程的第二种方法");
    }
}
 
class Thread3 implements Callable<String> {
 
    @Override
    public String call() throws Exception {
        return Thread.currentThread()+"创建线程的第三种方法";
    }
}

```



### 创建线程的三种方式的对比

1、采用实现Runnable、Callable接口的方式创建多线程

      优势：
    
       线程类只是实现了Runnable接口或Callable接口，还可以继承其他类。
    
       在这种方式下，多个线程可以共享同一个target对象，所以非常适合多个相同线程来处理同一份资源的情况，从而可以将CPU、代码和数据分开，形成清晰的模型，较好地体现了面向对象的思想。
    
       劣势：
    
     编程稍微复杂，如果要访问当前线程，则必须使用Thread.currentThread()方法。

2、使用继承Thread类的方式创建多线程

      优势：
    
      编写简单，如果需要访问当前线程，则无需使用Thread.currentThread()方法，直接使用this即可获得当前线程。
    
      劣势：
    
      线程类已经继承了Thread类，所以不能再继承其他父类。

3、Runnable和Callable的区别

	Runnable接口定义的run方法，Callable定义的是call方法。
	run方法没有返回值，call方法必须有返回值。
	run方法无法抛出异常，call方法可以抛出checked exception。
	Callable和Runnable都可以应用于executors。而Thread类只支持Runnable.


### 总结

  鉴于上面分析，因此一般推荐采用实现Runnable接口、Callable接口的方式来创建多线程。
