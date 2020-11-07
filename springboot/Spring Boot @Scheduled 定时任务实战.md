## Spring Boot @Scheduled 定时任务实战



假设我们已经搭建好了一个基于Spring Boot项目，首先我们要在Application中设置启用定时任务功能@EnableScheduling。



**启动定时任务**

```
package com.scheduling;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableScheduling;
@SpringBootApplication
@EnableScheduling
public class Application {
    public static void main(String[] args) throws Exception {
        SpringApplication.run(Application.class);
    }
}
```

其中 @EnableScheduling 注解的作用是发现注解@Scheduled的任务并后台执行。

### 

***定时任务具体实现类***



接下来我们来创建一个定时任务

```
package com.Scheduler.utils;
import java.text.SimpleDateFormat;
import java.util.Date;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class Scheduler{
    private static final SimpleDateFormat dateFormat = new SimpleDateFormat("HH:mm:ss");

//每隔2秒执行一次
@Scheduled(fixedRate = 2000)
public void testTasks() {
    System.out.println("定时任务执行时间：" + dateFormat.format(new Date()));
}

//每天3：05执行
@Scheduled(cron = "0 05 03 ? * *")
public void testTasks() {
    System.out.println("定时任务执行时间：" + dateFormat.format(new Date()));
}

}
```

运行Spring Boot，输出结果为如下，每2秒钟打印出当前时间。

```
定时任务执行时间：23:28:00

定时任务执行时间：23:28:02

定时任务执行时间：23:28:04

定时任务执行时间：23:28:06

定时任务执行时间：23:28:08
```



**注意：** 需要在定时任务的类上加上注释：@Component，在具体的定时任务方法上加上注释@Scheduled即可启动该定时任务。

关注微信公众号：Java技术栈，在后台回复：boot，可以获取我整理的 N 篇最新Spring Boot 教程，都是干货。

***@Scheduled参数描述***

***

**@Scheduled(fixedRate=3000)**：上一次***开始执行时间***点后**3秒**再次执行；

**@Scheduled(fixedDelay=3000)**：上一次***执行完毕时间***点**3秒**再次执行；

**@Scheduled(initialDelay=1000, fixedDelay=3000)**：第一次延迟**1秒**执行，然后在上一次***执行完毕时间***点**3秒**再次执行；

**@Scheduled(cron="\* \* \* \* \* ?")**：按**cron**规则执行；



***cron规则***



cron表达式中各时间元素使用**空格**进行分割，表达式有至少6个（也可能7个）分别表示如下含义：

```
秒（0~59）
分钟（0~59）
小时（0~23）
天（月）（0~31，但是你需要考虑你月的天数）
月（0~11）
天（星期）（1~7 1=SUN 或 SUN，MON，TUE，WED，THU，FRI，SAT）
年份（1970－2099）
```

其中每个元素可以是一个值(如6),一个连续区间(9-12),一个间隔时间(8-18/4)(/表示每隔4小时),一个列表(1,3,5),通配符。由于"月份中的日期"和"星期中的日期"这两个元素互斥的,必须要对其中一个设置?.

```
0 0 10,14,16 * * ? 每天上午10点，下午2点，4点
0 0/30 9-17 * * ?   朝九晚五工作时间内每半小时
0 0 12 ? * WED 表示每个星期三中午12点 
"0 0 12 * * ?" 每天中午12点触发 
"0 15 10 ? * *" 每天上午10:15触发 
"0 15 10 * * ?" 每天上午10:15触发 
"0 15 10 * * ? *" 每天上午10:15触发 
"0 15 10 * * ? 2005" 2005年的每天上午10:15触发 
"0 * 14 * * ?" 在每天下午2点到下午2:59期间的每1分钟触发 
"0 0/5 14 * * ?" 在每天下午2点到下午2:55期间的每5分钟触发 
"0 0/5 14,18 * * ?" 在每天下午2点到2:55期间和下午6点到6:55期间的每5分钟触发 
"0 0-5 14 * * ?" 在每天下午2点到下午2:05期间的每1分钟触发 
"0 10,44 14 ? 3 WED" 每年三月的星期三的下午2:10和2:44触发 
"0 15 10 ? * MON-FRI" 周一至周五的上午10:15触发 
"0 15 10 15 * ?" 每月15日上午10:15触发 
"0 15 10 L * ?" 每月最后一日的上午10:15触发 
"0 15 10 ? * 6L" 每月的最后一个星期五上午10:15触发 
"0 15 10 ? * 6L 2002-2005" 2002年至2005年的每月的最后一个星期五上午10:15触发 
"0 15 10 ? * 6#3" 每月的第三个星期五上午10:15触发
```

有些子表达式能包含一些范围或列表

**“\*”**字符代表所有可能的值

因此，“*”在子表达式（**月**）里表示每个月的含义，“*”在子表达式（**天（星期）**）表示星期的每一天



**“/”字符用来指定数值的增量**

例如：在子表达式（分钟）里的“0/15”表示从第0分钟开始，每15分钟

在子表达式（分钟）里的“3/20”表示从第3分钟开始，每20分钟（它和“3，23，43”）的含义一样



**“？****”字符仅被用于天（月）和天（星期）两个子表达式，表示不指定值**

当2个子表达式其中之一被指定了值以后，为了避免冲突，需要将另一个子表达式的值设为“？”



**“L” 字符仅被用于天（月）和天（星期）两个子表达式，它是单词“last”的缩写**

但是它在两个子表达式里的含义是不同的。

在天（月）子表达式中，“L”表示一个月的最后一天

在天（星期）自表达式中，“L”表示一个星期的最后一天，也就是SAT

如果在“L”前有具体的内容，它就具有其他的含义了

例如：“6L”表示这个月的倒数第６天，“ＦＲＩＬ”表示这个月的最一个星期五

注意：在使用“L”参数时，不要指定列表或范围，因为这会导致问题



| **字段**   | **允许值**        | **允许的特殊字符** |
| ---------- | ----------------- | ------------------ |
| 秒         | 0-59              | , - * /            |
| 分         | 0-59              | , - * /            |
| 小时       | 0-23              | , - * /            |
| 日期       | 1月31日           | , - * ? / L W C    |
| 月份       | 1-12 或者 JAN-DEC | , - * /            |
| 星期       | 1-7 或者 SUN-SAT  | , - * ? / L C #    |
| 年（可选） | 留空, 1970-2099   |                    |



作者：J'KYO

来源：http://www.cnblogs.com/pejsidney/p/9046818.html