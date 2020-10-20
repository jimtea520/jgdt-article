spring事务和数据库事务 的区别



# 概念

本质上其实是同一个概念,spring的事务是对数据库的事务的封装,最后本质的实现还是在数据库,假如数据库不支持事务的话,spring的事务是没有作用的.数据库的事务说简单就只有开启,回滚和关闭,spring对数据库事务的包装,原理就是拿一个数据连接,根据spring的事务配置,操作这个数据连接对数据库进行事务开启,回滚或关闭操作.但是spring除了实现这些,还配合spring的传播行为对事务进行了更广泛的管理.那就是事务中涉及的隔离级别,以及spring如何对数据库的隔离级别进行封装.事务与隔离级别放在一起理解会更好些.



# 四个特性：(ACID)

**原子性（Atomicity）**：事务是一个原子操作，由一系列动作组成。事务的原子性确保动作要么全部完成，要么完全不起作用。
**一致性（Consistency）：**一旦事务完成（不管成功还是失败），系统必须确保它所建模的业务处于一致的状态，而不会是部分完成部分失败。在现实中的数据不应该被破坏。
**隔离性（Isolation）：**可能有许多事务会同时处理相同的数据，因此每个事务都应该与其他事务隔离开来，防止数据损坏。
**持久性（Durability）：**一旦事务完成，无论发生什么系统错误，它的结果都不应该受到影响，这样就能从任何系统崩溃中恢复过来。通常情况下，事务的结果被写到持久化存储器中。



# 核心接口

Spring事务管理的实现有许多细节，如果对整个接口框架有个大体了解会非常有利于我们理解事务，下面通过讲解Spring的事务接口来了解Spring实现事务的具体策略。 
Spring事务管理涉及的接口的联系如下：

![image-20200911194633871](https://gitee.com/fking86/images4typora/raw/master/imgs/20200911194634.png)

## 事务管理器

Spring并不直接管理事务，而是提供了多种事务管理器，他们将事务管理的职责委托给[hibernate](http://lib.csdn.net/base/javaee)或者JTA等持久化机制所提供的相关平台框架的事务来实现。 
Spring事务管理器的接口是org.springframework.transaction.PlatformTransactionManager，通过这个接口，Spring为各个平台如JDBC、Hibernate等都提供了对应的事务管理器，但是具体的实现就是各个平台自己的事情了。此接口的内容如下：

```
Public interface PlatformTransactionManager()...{  



    // 由TransactionDefinition得到TransactionStatus对象



    TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException; 



    // 提交



    Void commit(TransactionStatus status) throws TransactionException;  



    // 回滚



    Void rollback(TransactionStatus status) throws TransactionException;  
```

从这里可知具体的具体的事务管理机制对Spring来说是透明的，它并不关心那些，那些是对应各个平台需要关心的，所以Spring事务管理的一个优点就是为不同的事务API提供一致的编程模型，如JTA、JDBC、Hibernate、JPA。下面分别介绍各个平台框架实现事务管理的机制。



###  JDBC事务

如果应用程序中直接使用JDBC来进行持久化，DataSourceTransactionManager会为你处理事务边界。为了使用DataSourceTransactionManager，你需要使用如下的XML将其装配到应用程序的上下文定义中：

```
 <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource" />
 /bean>
```

实际上，DataSourceTransactionManager是通过调用[Java](http://lib.csdn.net/base/java).sql.Connection来管理事务，而后者是通过DataSource获取到的。通过调用连接的commit()方法来提交事务，同样，事务失败则通过调用rollback()方法进行回滚。



### Hibernate事务

如果应用程序的持久化是通过Hibernate实习的，那么你需要使用HibernateTransactionManager。对于Hibernate3，需要在Spring上下文定义中添加如下的`<bean>`声明：

```
    <bean id="transactionManager" class="org.springframework.orm.hibernate3.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>
```

sessionFactory属性需要装配一个Hibernate的session工厂，HibernateTransactionManager的实现细节是它将事务管理的职责委托给org.hibernate.Transaction对象，而后者是从Hibernate Session中获取到的。当事务成功完成时，HibernateTransactionManager将会调用Transaction对象的commit()方法，反之，将会调用rollback()方法。



### Java持久化API事务（JPA）

Hibernate多年来一直是事实上的Java持久化标准，但是现在Java持久化API作为真正的Java持久化标准进入大家的视野。如果你计划使用JPA的话，那你需要使用Spring的JpaTransactionManager来处理事务。你需要在Spring中这样配置JpaTransactionManager：

```
    <bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
        <property name="sessionFactory" ref="sessionFactory" />
    </bean>
```

JpaTransactionManager只需要装配一个JPA实体管理工厂（javax.persistence.EntityManagerFactory接口的任意实现）。JpaTransactionManager将与由工厂所产生的JPA EntityManager合作来构建事务。



### Java原生API事务

如果你没有使用以上所述的事务管理，或者是跨越了多个事务管理源（比如两个或者是多个不同的数据源），你就需要使用JtaTransactionManager：

```
    <bean id="transactionManager" class="org.springframework.transaction.jta.JtaTransactionManager">
        <property name="transactionManagerName" value="java:/TransactionManager" />
    </bean>
```

JtaTransactionManager将事务管理的责任委托给javax.transaction.UserTransaction和javax.transaction.TransactionManager对象，其中事务成功完成通过UserTransaction.commit()方法提交，事务失败通过UserTransaction.rollback()方法回滚。



## 基本事务属性的定义

上面讲到的事务管理器接口PlatformTransactionManager通过getTransaction(TransactionDefinition definition)方法来得到事务，这个方法里面的参数是TransactionDefinition类，这个类就定义了一些基本的事务属性。 
那么什么是事务属性呢？事务属性可以理解成事务的一些基本配置，描述了事务策略如何应用到方法上。事务属性包含了5个方面，如图所示：

![image-20200911200146727](https://gitee.com/fking86/images4typora/raw/master/imgs/20200911200146.png)

TransactionDefinition接口内容如下：

```
public interface TransactionDefinition {
    int getPropagationBehavior(); // 返回事务的传播行为
    int getIsolationLevel(); // 返回事务的隔离级别，事务管理器根据它来控制另外一个事务可以看到本事务内的哪些数据
    int getTimeout();  // 返回事务必须在多少秒内完成
    boolean isReadOnly(); // 事务是否只读，事务管理器能够根据这个返回值进行优化，确保事务是只读的
} 
```



### 传播行为

事务的第一个方面是传播行为（propagation behavior）。当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。例如：方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。Spring定义了七种传播行为：

Spring是用枚举来表示事务传播行为的

```
package org.springframework.transaction.annotation;

import org.springframework.transaction.TransactionDefinition;


public enum Propagation {

	REQUIRED(TransactionDefinition.PROPAGATION_REQUIRED),

	SUPPORTS(TransactionDefinition.PROPAGATION_SUPPORTS),

	MANDATORY(TransactionDefinition.PROPAGATION_MANDATORY),

	REQUIRES_NEW(TransactionDefinition.PROPAGATION_REQUIRES_NEW),

	PPORTED(TransactionDefinition.PROPAGATION_NOT_SUPPORTED),

	NEVER(TransactionDefinition.PROPAGATION_NEVER),

	NESTED(TransactionDefinition.PROPAGATION_NESTED);

	private final int value;

	Propagation(int value) { this.value = value; }

	public int value() { return this.value; }

}
```



| 传播行为                  | 含义                                                         |
| :------------------------ | :----------------------------------------------------------- |
| PROPAGATION_REQUIRED      | 表示当前方法必须运行在事务中。如果当前事务存在，方法将会在该事务中运行。否则，会启动一个新的事务（如果被调用端发生异常，那么调用端和被调用端事务都将回滚） |
| PROPAGATION_SUPPORTS      | 表示当前方法不需要事务上下文，但是如果存在当前事务的话，那么该方法会在这个事务中运行 |
| PROPAGATION_MANDATORY     | 表示该方法必须在事务中运行，如果当前事务不存在，则会抛出一个异常 |
| PROPAGATION_REQUIRED_NEW  | 表示当前方法必须运行在它自己的事务中。一个新的事务将被启动。如果存在当前事务，在该方法执行期间，当前事务会被挂起。如果使用JTATransactionManager的话，则需要访问TransactionManager |
| PROPAGATION_NOT_SUPPORTED | 表示该方法不应该运行在事务中。如果存在当前事务，在该方法运行期间，当前事务将被挂起。如果使用JTATransactionManager的话，则需要访问TransactionManager |
| PROPAGATION_NEVER         | 表示当前方法不应该运行在事务上下文中。如果当前正有一个事务在运行，则会抛出异常 |
| PROPAGATION_NESTED        | 表示如果当前已经存在一个事务，那么该方法将会在嵌套事务中运行。嵌套的事务可以独立于当前事务进行单独地提交或回滚。如果当前事务不存在，那么其行为与PROPAGATION_REQUIRED一样。注意各厂商对这种传播行为的支持是有所差异的。可以参考资源管理器的文档来确认它们是否支持嵌套事务 |



@Transactional(propagation=Propagation.REQUIRED)
如果有事务, 那么加入事务, 没有的话新建一个(默认情况下)
@Transactional(propagation=Propagation.NOT_SUPPORTED)
容器不为这个方法开启事务
@Transactional(propagation=Propagation.REQUIRES_NEW)
不管是否存在事务,都创建一个新的事务,原来的挂起,新的执行完毕,继续执行老的事务
@Transactional(propagation=Propagation.MANDATORY)
必须在一个已有的事务中执行,否则抛出异常
@Transactional(propagation=Propagation.NEVER)
必须在一个没有的事务中执行,否则抛出异常(与Propagation.MANDATORY相反)
@Transactional(propagation=Propagation.SUPPORTS)
如果其他bean调用这个方法,在其他bean中声明事务,那就用事务.如果其他bean没有声明事务,那就不用事务.



### 事务隔离级别

| **隔离级别**               | **含义**                                                     |
| -------------------------- | ------------------------------------------------------------ |
| isolation_default          | 使用数据库默认的事务隔离级别                                 |
| isolation_read_uncommitted | 允许读取尚未提交的修改，可能导致脏读、幻读和不可重复读       |
| isolation_read_committed   | 允许从已经提交的事务读取，可防止脏读、但幻读，不可重复读仍然有可能发生 |
| isolation_repeatable_read  | 对相同字段的多次读取的结果是一致的，除非数据被当前事务自生修改。可防止脏读和不可重复读，但幻读仍有可能发生 |
| isolation_serializable     | 完全服从acid隔离原则，确保不发生脏读、不可重复读、和幻读，但执行效率最低。 |

#### 脏读、不可重复读、幻象读概念：

**a.脏读**：指当一个事务正字访问数据，并且对数据进行了修改，而这种数据还没有提交到数据库中，这时，另外一个事务也访问这个数据，然后使用了这个数据。因为这个数据还没有提交那么另外一个事务读取到的这个数据我们称之为脏数据。依据脏数据所做的操作肯能是不正确的。
**b.不可重复读：**指在一个事务内，多次读同一数据。在这个事务还没有执行结束，另外一个事务也访问该同一数据，那么在第一个事务中的两次读取数据之间，由于第二个事务的修改第一个事务两次读到的数据可能是不一样的，这样就发生了在一个事物内两次连续读到的数据是不一样的，这种情况被称为是不可重复读。
**c.幻象读：**一个事务先后读取一个范围的记录，但两次读取的纪录数不同，我们称之为幻象读（两次执行同一条 select 语句会出现不同的结果，第二次读会增加一数据行，并没有说这两次执行是在同一个事务中）



### 事务只读属性

pring事务只读的含义是指，如果后端数据库发现当前事务为只读事务，那么就会进行一系列的优化措施。它是在后端数据库进行实施的，因此，只有对于那些有可能启动一个新事务的传播行为（REQUIRED,REQUIRES_NEW,NESTED）的方法来说，才有意义。（测试表明，当使用JDBC事务管理器并设置当前事务为只读时，并不能发生预期的效果，即能执行删除，更新，插入操作）



### 事务超时

  有的时候为了系统中关键部分的性能问题，它的事务执行时间应该尽可能的短。因此可以给这些事务设置超时时间，以秒为单位。我们知道事务的开始往往都会发生数据库的表锁或者被数据库优化为行锁，如果允许时间过长，那么这些数据会一直被锁定，影响系统的并发性。

  因为超时时钟是在事务开始的时候启动，因此只有对于那些有可能启动新事物的传播行为（REQUIRED,REQUIRES_NEW,NESTED）的方法来说，事务超时才有意义。



### 事务回滚规则

  spring中可以指定当方法执行并抛出异常的时候，哪些异常回滚事务，哪些异常不回滚事务。默认情况下，只在方法抛出运行时异常的时候才回滚(runtime exception)。而在出现受阻异常(checked exception)时不回滚事务。当然可以采用申明的方式指定哪些受阻异常像运行时异常那样指定事务回滚。





