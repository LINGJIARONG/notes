

# 设计模式

1. ### 适配器模式（Adapter模式）

2. ### jdk动态代理

   ```java
   
   	
   	  public static void main(String[] args) throws IllegalAccessException, InstantiationException {
   	        // 1. 创建被代理的对象，UserService接口的实现类
   	        UserServiceImpl userServiceImpl = new UserServiceImpl();
   	        // 2. 获取对应的 ClassLoader
   	        ClassLoader classLoader = userServiceImpl.getClass().getClassLoader();
   	        // 3. 获取所有接口的Class，这里的UserServiceImpl只实现了一个接口UserService，
   	        Class[] interfaces = userServiceImpl.getClass().getInterfaces();
   	        // 4. 创建一个将传给代理类的调用请求处理器，处理所有的代理对象上的方法调用
   	        //     这里创建的是一个自定义的日志处理器，须传入实际的执行对象 userServiceImpl
   	        InvocationHandler logHandler = new LogHandler(userServiceImpl);
   	        /*
   			   5.根据上面提供的信息，创建代理对象 在这个过程中，
   	               a.JDK会通过根据传入的参数信息动态地在内存中创建和.class 文件等同的字节码
   	               b.然后根据相应的字节码转换成对应的class，
   	               c.然后调用newInstance()创建代理实例
   			 */
   	        UserService proxy = (UserService) Proxy.newProxyInstance(classLoader, interfaces, logHandler);
   	        // 调用代理的方法
   	        proxy.select();
   	        proxy.update();
   	    }
   ```

   

   ```java
   public class LogHandler implements InvocationHandler{
       Object target;  // 被代理的对象，实际的方法执行者
       public LogHandler(Object target) {
           this.target = target;
       }
   	@Override
   	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
   			before();
   	        Object result = method.invoke(target, args);  // 调用 target 的 method 方法
   	        after();
   	        return result;  // 返回方法的执行结果
   	}
   	 // 调用invoke方法之前执行
       private void before() {
           System.out.println(String.format("log start time [%s] ", new Date()));
       }
       // 调用invoke方法之后执行
       private void after() {
           System.out.println(String.format("log end time [%s] ", new Date()) );
       }
   }
   
   ```

   

   生成的xxxxProxy 继承了 Proxy 类，并且实现了被代理的所有接口，以及equals、hashCode、toString等方法，由于 xxxProxy 继承了 Proxy 类，所以每个代理类都会关联一个 InvocationHandler 方法调用处理器

   类和所有方法都被 `public final` 修饰，所以代理类只可被使用，不可以再被继承，每个方法都有一个 Method 对象来描述，Method 对象在static静态代码块中创建，以 `m + 数字` 的格式命名

   ```java
   import java.lang.reflect.InvocationHandler;
   import java.lang.reflect.Method;
   import java.lang.reflect.Proxy;
   import java.lang.reflect.UndeclaredThrowableException;
   import proxy.UserService;
   
   public final class UserServiceProxy extends Proxy implements UserService {
       private static Method m1;
       private static Method m2;
       private static Method m4;
       private static Method m0;
       private static Method m3;
   
       public UserServiceProxy(InvocationHandler var1) throws  {
           super(var1);
       }
   
       public final boolean equals(Object var1) throws  {
           // 省略...
       }
   
       public final String toString() throws  {
           // 省略...
       }
   
       public final void select() throws  {
           try {
               super.h.invoke(this, m4, (Object[])null);
           } catch (RuntimeException | Error var2) {
               throw var2;
           } catch (Throwable var3) {
               throw new UndeclaredThrowableException(var3);
           }
       }
   
       public final int hashCode() throws  {
           // 省略...
       }
   
       public final void update() throws  {
           try {
               super.h.invoke(this, m3, (Object[])null);
           } catch (RuntimeException | Error var2) {
               throw var2;
           } catch (Throwable var3) {
               throw new UndeclaredThrowableException(var3);
           }
       }
   
       static {
           try {
               m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
               m2 = Class.forName("java.lang.Object").getMethod("toString");
               m4 = Class.forName("proxy.UserService").getMethod("select");
               m0 = Class.forName("java.lang.Object").getMethod("hashCode");
               m3 = Class.forName("proxy.UserService").getMethod("update");
           } catch (NoSuchMethodException var2) {
               throw new NoSuchMethodError(var2.getMessage());
           } catch (ClassNotFoundException var3) {
               throw new NoClassDefFoundError(var3.getMessage());
           }
       }
   }
   
   作者：小旋锋
   链接：https://juejin.im/post/6844903744954433544
   来源：掘金
   著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处
   ```

   

   调用方法的时候通过 `super.h.invoke(this, m1, (Object[])null);` 调用，其中的 `super.h.invoke` 实际上是在创建代理的时候传递给 `Proxy.newProxyInstance` 的 xxxHandler 对象，它继承 InvocationHandler 类，负责实际的调用处理逻辑。

   ​      ![image-20200705231529903](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200705231529903.png)

   

3. ### CGLIB动态代理

   ```java
   public class Intercepter implements MethodInterceptor{
   	@Override
   	public Object intercept(Object object, Method arg1, Object[] objects, MethodProxy methodProxy) throws Throwable {
   		before();
           Object result = methodProxy.invokeSuper(object, objects);   // 注意这里是调用 invokeSuper 而不是 invoke，否则死循环，methodProxy.invokesuper执行的是原始类的方法，method.invoke执行的是子类的方法
           after();
           return result;
   	}
   	private void before() {
   		System.out.println(String.format("log start time [%s] ", new Date()));
   	}
   	private void after() {
   		System.out.println(String.format("log end time [%s] ", new Date()));
   	}
   }
   ```

   ```java
   
   public class Client3 {
   	public static void main(String[] args) {
   		Intercepter proxy = new Intercepter();
   		Enhancer enhancer = new Enhancer();
   		enhancer.setSuperclass(UserServiceImpl.class); // 设置超类，cglib是通过继承来实现的
   		enhancer.setCallback(proxy); //设置拦截
   		UserService dao = (UserService) enhancer.create(); // 创建代理类
   		dao.update();
   		dao.select();
   	}
   }
   ```

   CGLIB 创建动态代理类的模式是：

   1. 查找目标类上的所有非final 的public类型的方法定义；
   2. 将这些方法的定义转换成字节码；
   3. 将组成的字节码转换成相应的代理的class对象；
   4. 实现 MethodInterceptor接口，用来处理对代理类上所有方法的请求

   Diff : 

   * cglib动态代理：基于ASM机制实现，通过生成业务类的子类作为代理类。

   * 基于类似 cglib 框架的优势：

     - 无需实现接口，达到代理类无侵入

     - 只操作我们关心的类，而不必为其他相关类增加工作量。

     - 高性能

       

   * JDK动态代理：基于Java反射机制实现，必须要实现了接口的业务类才能用这种办法生成代理对象。

   * JDK Proxy 的优势：

     - 最小化依赖关系，减少依赖意味着简化开发和维护，JDK 本身的支持，可能比 cglib 更加可靠。
     - 平滑进行 JDK 版本升级，而字节码类库通常需要进行更新以保证在新版 Java 上能够使用。
     - 代码实现简单。

     

     

4. ### 描述动态代理的几种实现方式？分别说出相应的优缺点

   代理可以分为 "静态代理" 和 "动态代理"，动态代理又分为 "JDK动态代理" 和 "CGLIB动态代理" 实现。

   **静态代理**：代理对象和实际对象都继承了同一个接口，在代理对象中指向的是实际对象的实例，这样对外暴露的是代理对象而真正调用的是 Real Object

   - **优点**：可以很好的保护实际对象的业务逻辑对外暴露，从而提高安全性。
   - **缺点**：不同的接口要有不同的代理类实现，会很冗余

   **JDK 动态代理**：

   - 为了解决静态代理中，生成大量的代理类造成的冗余；
   - JDK 动态代理只需要实现 InvocationHandler 接口，重写 invoke 方法便可以完成代理的实现，
   - jdk的代理是利用反射生成代理类 Proxyxx.class 代理类字节码，并生成对象
   - jdk动态代理之所以**只能代理接口**是因为**代理类本身已经extends了Proxy，而java是不允许多重继承的**，但是允许实现多个接口
   - **优点**：解决了静态代理中冗余的代理实现类问题。
   - **缺点**：JDK 动态代理是基于接口设计实现的，如果没有接口，会抛异常。

   **CGLIB 代理**：

   - 由于 JDK 动态代理限制了只能基于接口设计，而对于没有接口的情况，JDK方式解决不了；
   - CGLib 采用了非常底层的字节码技术，其原理是通过字节码技术为一个类创建子类，并在子类中采用方法拦截的技术拦截所有父类方法的调用，顺势织入横切逻辑，来完成动态代理的实现。
   - 实现方式实现 MethodInterceptor 接口，重写 intercept 方法，通过 Enhancer 类的回调方法来实现。
   - 但是CGLib在创建代理对象时所花费的时间却比JDK多得多，所以对于单例的对象，因为无需频繁创建对象，用CGLib合适，反之，使用JDK方式要更为合适一些。
   - 同时，由于CGLib由于是采用动态创建子类的方法，对于final方法，无法进行代理。
   - **优点**：没有接口也能实现动态代理，而且采用字节码增强技术，性能也不错。
   - **缺点**：技术实现相对难理解些。

5. 

6. 

   

   

   

   

# Mysql 常见问题

## 第一二三范式 ：

**1NF的定义为：符合1NF的关系中的每个属性都不可再分**

**2NF在1NF的基础之上，消除了非主属性对于码的部分函数依赖**。

>  成绩表 （学号，系，课，分数）， 码 = （学号+课） 
>
> 系只依赖学号

 **3NF在2NF的基础之上，消除了非主属性对于码的传递函数依赖**

> （学号，系，系主任）， 码 = 学号
>
> 学号-》系-》系主任

**BCNF再3NF 的基础上消除主属性对于码的部分与传递函数依赖。**

> （仓库，管理员，物品，数额）
>
> 主属性：仓库名、管理员、物品名
> 非主属性：数量
>
> 

https://www.zhihu.com/question/24696366



## 索引

### 索引是啥？优缺点

```
索引是帮助MySQL高效获取数据的数据结构（有序）。



提高数据检索的效率，降低数据库的IO成本，同时却也降低更新表的速度，如对表进行INSERT、UPDATE、DELETE。因为更新表时，MySQL 不仅要保存数据，还要保存一下索引文件每次更新添加了索引列的字段，都会调整因为更新所带来的键值变化后的索引信息。
```



### 索引类型，结构

```
https://www.jianshu.com/p/f74feb7dbdc1

- 1. BTree 索引 ： 最常见的索引类型，大部分索引都支持 BTree索引。

  2. Hash 索引：Memory引擎支持 ， 使用场景简单 。

  3. R-tree 索引（空间索引）：空间索引是MyISAM引擎的一个特殊索引类型，主要用于地理空间数据类型，通常使用较少。

  4. Full-text （全文索引） ：全文索引也是MyISAM的一个特殊索引类型，主要用于全文索引，InnoDB从Mysql5.6版本开始支持全文索引。

  

- 1） 单值索引 ：即一个索引只包含单个列，一个表可以有多个单列索引。
  2） 唯一索引 ：索引列的值必须唯一，但允许有空值。
  3） 复合索引 ：即一个索引包含多个列，例如用身份证号和考生号两列作为复合索引
  4） 主键索引,也就是在唯一索引的基础上相应的列必须为主键
```



### 什么时候建议/不建议+索引

#### 索引建立原则

```
（1）. 尽量减少like，但不是绝对不可用，”xxxx%” 是可以用到索引的
（2）. 表的主键、外键必须有索引
（3）. 谁的区分度更高（同值的最少），谁建索引，区分度的公式是count(distinct（字段）)/count(*)
（4）. 单表数据太少，不适合建索引
（5）. where，order by ,group by 等过滤时，后面的字段最好加上索引
（6）. 如果既有单字段索引，又有这几个字段上的联合索引，一般可以删除联合索引；
（7）. 联合索引的建立需要进行仔细分析；尽量考虑用单字段索引代替：
（8）. 联合索引: mysql 从左到右的使用索引中的字段，一个查询可以只使用索引中的一部份，但只能是最左侧部分。例如索引是key index(a,b,c). 可以支持 a|a,b|a,b,c 3种组合进行查找，但不支持 b,c 进行查找.当最左侧字段是常量引用时，索引就十分有效。
（9）. 前缀索引: 有时候需要索引很长的字符列，这会让索引变得大且慢。通常可以索引开始的部分字符，这样可以大大节约索引空间，从而提高索引效率。其缺点是不能用于ORDER BY和GROUP BY操作，也不能用于覆盖索引 Covering index（即当索引本身包含查询所需全部数据时，不再访问数据文件本身）
10）. NULL会导致索引形同虚设
```
#### 索引什么时候失效

1. ```
   1. where条件中有or，即使其中有条件带索引也不会使用；如果要想使用or又想让索引生效，只能将or条件中的每个列都加上索引；
   2. 对于多列索引，不是使用的第一部分，则不会使用索引（即不符合最左前缀原则）；
   3. like查询是以%开头；
   4. 如果列类型是字符串，那一定要在条件中将数据使用引号引用起来，否则不使用索引；
   5. 如果mysql估计使用全表扫描要比使用索引快，则不使用索引
   ```



### 索引下推是什么  (Index Condition Pushdown) ICP

```
在Mysql5.6以后引入了索引下推优化，可以在索引遍历过程中对索引所包含的字段先做判断，直接过滤不满足的条件，旨在减少回表次数和减少MySQL server层和引擎层的交互次数；
```



## 什么是游标/光标

```
游标是用来存储查询结果集的数据类型 , 在存储过程和函数中可以使用光标对结果集进行循环的处理
```

## 触发器是什么？

```
触发器是与表有关的数据库对象，指在 insert/update/delete 之前或之后，触发并执行触发器中定义的SQL语句集合。触发器的这种特性可以协助应用在数据库端确保数据的完整性 , 日志记录 , 数据校验等操作 。
```



## 存储引擎有哪些，特点是啥？InnoDB主要特性？

https://blog.csdn.net/zgrgfr/article/details/74455547

## innode 和MyISAM 区别

* InnoDB 中不保存表的具体行数，也就是说，执行select count(*) from table时，InnoDB要扫描一遍整个表来计算有多少行，但是MyISAM只要简单的*
* *读出保存好的行数即可。注意的是，当count(*)语句包含 where条件时，两种表的操作是一样的。
* ***\*两种类型最主要的差别就是Innodb 支持事务处理与外键和行级锁。\****而MyISAM不支持.所以MyISAM往往就容易被人认为只适合在小项目中使用。
* InnoDB 把数据和索引存放在表空间里面， MyISAM 表被存放在三个文件 。frm 文件存放表格定义。 数据文件是MYD (MYData) 。 索引文件是MYI (MYIndex)引伸
* **InnoDB支持事务，MyISAM不支持，对于InnoDB每一条SQL语言都默认封装成事务，自动提交，这样会影响速度，所以最好把多条SQL语言放在begin和commit之间，组成一个事务；**
* **InnoDB表必须有主键（用户没有指定的话会自己找或生产一个主键），而Myisam可以没有**

## mysql 日志

   ```
   https://www.jianshu.com/p/db19a1d384bc
   
   Mysql 有4种类型的日志：Error Log、Genaral Query Log、 Binary Log 和 Slow Query Log
   
   Error Log 记录Mysql运行过程中的Error、Warning、Note等信息，系统出错或者某条记录出问题可以查看Error日志。
   
   General Query Log 记录mysql的日常日志，包括查询、修改、更新等的每条sql。
   
   Binary Log 二进制日志，包含一些事件，这些事件描述了数据库的改动，如建表、数据改动等，主要用于备份恢复、回滚操作等，需要配置一个server-id
   ```

   

## InnoDB 重做日志(redo log) 回滚日志(undo log）是什么

- ```
  InnoDB 有两块非常重要的日志，一个是undo log，另外一个是redo log，前者用来保证事务的原子性以及InnoDB的MVCC，后者用来保证事务的持久性。
  
  - RedoLog
  
  事务将数据的变更写到内存页中同时记录到redolog里面，然后提交就算这个事务已经结束，期间只有一次磁盘IO，效率远比比事务将数据直接写到磁盘里面要高的多，并且redolog写到磁盘可以看作是顺序IO，所以redolog能提升性能的一个方面就在于将对磁盘的随机IO写转换成顺序IO；且事务因为宕机可能会丢失，redolog还有一个作用就是保证事务的持久性；
  
  
  
  
  
  为什么需要Redo log？
  Innodb buffer pool内部储存的是数据页，当修改innodb表上某行数据时，如果该行不在内存中则需要将该行所在的数据页从磁盘读入到内存去，然后在内存中更新该行，现在内存中的数据页与磁盘中就不一致了，把这种内存数据页与磁盘数据页不一致的数据页称为脏页（dirty page），DB需要把脏页数据写入磁盘，但是如果每一次更新数据就会带来一次磁盘操作的话那么机器肯定撑不住；所以说mysql会把标记为脏页的数据页储存在一个flush list里面，用一个专门的后台线程定时刷脏；那么如果在mysql还没有刷脏的时候数据库挂了怎么办呢，所以就需要redo log；
  
  - UndoLog
  
  Undo log的存在保证了事务的原子性，MVCC就是依赖它来实现，当对任何行做了修改的时候都会在undo log里面记录，大量的undo log构成行的历史版本记录，在需要的时候可以回退(rollback)到任何版本；

  
   
  二、不同的DML操作，UNDO BLOCK中保存的前映像内容
  INSERT操作，UNDO中只需要保存插入记录的rowid，如果需要回退，通过保存的rowid进行删除即可（后面有案例）
  UPDATE操作，UNDO中只需要记录被更新字段的旧值，如果需要回退，只需要通过旧值覆盖更新后的值即可。
  DELETE操作，UNDO中必须记录整行的数据，如果需要回退，只需要将这整行的数据重新插入至表中即可。
  
  
     
  https://www.jianshu.com/p/c5a3c3a0508f   
  ```
  
  ## redo log
  
  https://juejin.im/entry/6844903681091977229
  
  redo log是innodb层产生的，只记录该存储引擎中表的修改，redo log包括两部分：一是内存中的日志缓冲(redo log buffer)，该部分日志是易失性的；二是磁盘上的重做日志文件(redo log file)，该部分日志是持久的。
  
  
  
  
  
  在概念上，innodb通过***force log at commit\***机制实现事务的持久性，即在事务提交的时候，必须先将该事务的所有事务日志写入到磁盘上的redo log file和undo log file中进行持久化。
  
  
  
  ![image-20200822001337259](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200822001337259.png)
  
  
  
  ### 写入机制
  
  `redo log`写入前是先写入到`redo log buffer`，buffer中的内容是不会持久化到磁盘中，如果写buffer的过程中异常退出了会丢失这部分日志，但是这部分的事务是没有提交的所有丢失的影响也不大；
  
  
  
  `innodb_flush_log_at_trx_commit`参数控制着redo log的写入策略：
  
  - 0： 表示每次事务提交时都只是把 redo log 留在 redo log buffer中，然后每秒后台线程使用fsync写入磁盘；
  - 1：表示每次事务提交时都使用fsync将 redo log 直接持久化到磁盘；
  - 2：表示每次事务提交时都只是把 redo log 写到 page cache中，然后每秒后台线程使用fsync写入磁盘；
  
  Innodb中有一个后台线程每隔1s会把redo log buffer中的日志write到文件系统的page cache中，然后使用fsync持久化到磁盘里面；
  
  
  
  ![image-20200822001604290](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200822001604290.png)
  
  在主从复制结构中，要保证事务的持久性和一致性，需要对日志相关变量设置为如下：
  
  - **如果启用了二进制日志，则设置sync_binlog=1，即每提交一次事务同步写到磁盘中。**
  - **总是设置innodb_flush_log_at_trx_commit=1，即每提交一次事务都写到磁盘中。**
  
  ==**如果MySQL宕机，而Buffer Pool的数据没有完全刷新到磁盘，就会导致数据丢失，无法保证持久性。** 因此引入Redo Log解决这个问题。==
  
  - 当数据修改时，首先写入Redo Log，再更新到Buffer Pool，保证数据不会因为宕机而丢失，保证持久性。
  - 当事务提交时会调用fsync将redo log刷至磁盘持久化。MySQL宕机时，通过读取Redo Log中的数据，对数据库进行恢复。
  
  ==Redo Log也是记录在磁盘中，为什么会比直接将Buffer Pool写入磁盘更快？==
  
  - Buffer Pool刷入脏页至磁盘是随机IO，每次修改的数据位置随机，而Redo Log永远在页中追加，属于顺序IO。
  - Buffer Pool刷入磁盘是以数据页为单位，每次都需要整页写入。而Redo Log只需要写入真正**物理修改**的部分，IO数据量大大减少。

## undo log

undo log有两个作用：提供回滚和多个行版本控制(MVCC)。

Undo Log中基于回滚指针(DB_ROLL_PT)维护数据行的所有历史版本。

 

### Insert/update

InnoDB中，undo log分为：

- Insert Undo Log
- Update Undo Log

1. Insert Undo Log

Insert Undo Log是INSERT操作产生的undo log。

INSERT操作的记录由于是该数据的第一个记录，对其他事务不可见，该Undo Log可以在事务提交后直接删除。

![image-20200822004014716](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200822004014716.png)



- type_cmpl：undo的类型
- undo_no：事务的ID
- table_id：undo log对应的表对象 接着的部分记录了所有主键的列和值。

Update Undo Log

Update Undo Log记录对DELETE和UPDATE操作产生的Undo Log。

Update Undo Log会提供MVCC机制，因此不能在事务提交时就删除，而是放入undo log链表，等待purge线程进行最后的删除。

![image-20200822004101341](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200822004101341.png)

## binlog记录格式 ,主从复制三种方式

```
MySQL 主从复制有三种方式：基于SQL语句的复制（statement-based  replication，SBR），基于行的复制（row-based replication，RBR)，混合模式复制（mixed-based  replication,MBR)。对应的binlog文件的格式也有三种：STATEMENT,ROW,MIXED。
```



## 两阶段提交 https://zhuanlan.zhihu.com/p/98778890

最后，那么当我执行一条 update 语句时，redo log 和 binlog 是在什么时候被写入的呢？这就有了我们常说的「两阶段提交」：

- 写入：redo log（prepare）
- 写入：binlog
- 写入：redo log（commit）

为什么 redo log 要分两个阶段： prepare 和 commit ？redo log 就不能一次写入吗？

我们分两种情况讨论：

- 先写 redo log，再写 binlog
- 先写 binlog，再写 redo log

1、先写 redo log，再写 binlog

这样会出现 redo log 写入到磁盘了，但是 binlog 还没写入磁盘，于是当发生 crash recovery 时，恢复后，主库会应用 redo log，恢复数据，但是由于没有 binlog，从库就不会同步这些数据，主库比从库“新”，造成主从不一致

2、先写 binlog，再写 redo log

跟上一种情况类似，很容易知道，这样会反过来，造成从库比主库“新”，也会造成主从不一致

而两阶段提交，就解决这个问题，crash recovery 时：

- 如果 redo log 已经 commit，那毫不犹豫的，把事务提交

- 如果 redo log 处于 prepare，则去判断事务对应的 binlog 是不是完整的

- - 是，则把事务提交
  - 否，则事务回滚

两阶段提交，其实是为了保证 redo log 和 binlog 的逻辑一致性。



## DML 

DML语句的处理过程描述
update undotest set object_type='VIEW' where object_type='PROCEDURE';

* 检查shared pool中是否存在相同的语句，如果存在，重用执行计划，执行扫描运算，如果不存在，执行硬解析生成执行计划
    * 根据执行计划中的扫描运算，检查undotest表中的相关数据块是否存在buffer cache中，如果不存在则读取到内存中
    * 检查数据块中符合object_type='PROCEDURE'条件的记录，如果没有符合条件的行记录，则结束语句，如果存在则进入下一步
* 以当前模式（current）获取符合object_type='PROCEDURE'条件的数据块，准备进行更新
    * 在回滚表空间的相应回滚段头的事务表上分配事务槽，这个动作需要记录redo日志
* 从回滚段数据块上创建object_type='PROCEDURE'的前映像数据，这个动作也要记录redo日志
    * 修改object_type='VIEW' ，这是DML操作的数据变更，而需要记录redo日志
* 用户提交时，在redo日志中记录提交信息，将回滚段头上的事务表和回滚段数据块标记为非活动，清除修改数据块上的事务信息（也可能延迟清除）。同时必须确保整个事务的redo日志写到磁盘上的日志文件
    * 注意：如果最后用户回滚了事务，oracle从回滚段中将前映像数据提取出来，覆盖被更新的数据块。这个回滚动作本身也需要产生redo日志，因此，我们要知道回滚的代价非常昂贵。

## MVCC

### 隐藏的列

MVCC 在每行记录后面都保存着两个隐藏的列，用来存储两个版本号：

- 创建版本号：创建一行数据时，将当前系统版本号作为创建版本号赋值。
- 删除版本号：删除一行数据时，将当前系统版本号作为删除版本号赋值。如果该快照的删除版本号大于当前事务版本号表示该快照有效，否则表示该快照已经被删除了。

#### REPEATABLE READ（可重复读）隔离级别下MVCC如何工作：

当开始新一个事务时，该事务的版本号肯定会大于当前所有数据行快照的创建版本号，理解这一点很关键。

### **1. SELECT**

InnoDB会根据以下条件检查每一行记录：

\1. InnoDB只查找版本早于当前事务版本的数据行，这样可以**确保事务读取的行要么是在开始事务之前已经存在要么是事务自身插入或者修改过的**，在事务开始之后才插入的行，事务不会看到。

\2. 行的删除版本号要么未定义，要么大于当前事务版本号，这样可以**确保事务读取到的行在事务开始之前未被删除**，在事务开始之前就已经过期的数据行，该事务也不会看到。

只有符合上述两个条件的才会被查询出来

### **2. INSERT**

将当前系统版本号作为数据行快照的创建版本号。

### **3. DELETE**

将当前系统版本号作为数据行快照的删除版本号。

### 4. UPDATE

将当前系统版本号作为更新前的数据行快照的删除版本号，并将当前系统版本号作为更新后的数据行快照的创建版本号。**可以理解为先执行 DELETE 后执行 INSERT**。

保存这两个版本号，使大多数操作都不用加锁。使数据操作简单，性能很好，并且能保证只会读取到复合要求的行。不足之处是每行记录都需要额外的存储空间，需要做更多的行检查工作和一些额外的维护工作。



**MVCC只在COMMITTED READ（读提交）和REPEATABLE READ（可重复读）两种隔离级别下工作。**

可以认为MVCC是行级锁一个变种，但是他很多情况下避免了加锁操作，开销更低。虽然不同数据库的实现机制有所不同，但大都实现了非阻塞的读操作（读不用加锁，且能避免出现不可重复读和幻读），写操作也只锁定必要的行（写必须加锁，否则不同事务并发写会导致数据不一致）。

- **“快照”不是全量拷贝，而是利用了数据多版本的特性，也就是MVCC**
- **MVCC的核心在于每个事务自己维护的一个事务ID数组**
- **可以用“等价原则”来判断数据版本的可见性**
- **可重复读的关键，来自于“快照”**
- **“快照”并不是在begin后就生成，而是在第一条“快照读”语句后才生成**

https://www.jianshu.com/p/0d1d14ea84e6

### MVCC  Mysql到底是怎么实现MVCC的？

https://blog.csdn.net/chen77716/article/details/6742128

Read Committed隔离级别：每次select都生成一个快照读。

Read Repeatable隔离级别：开启事务后第一个select语句才是快照读的地方，而不是一开启事务就快照读。



## 间隙锁(Gap Lock) 是啥

```
间隙锁是为了解决在RR(可重复读)隔离级别下出现的幻读问题，虽然RR可以保证当前事务的不受其他事务的`update、delete`的影响，但是会因为`insert`而出现幻读；只有RR隔离级别下才会有Gap锁；

间隙锁和行锁合称Next-key lock(临键锁)

Gap Lock和Next-key lock只是在RR隔离级别下存在的，并遵守下面几个规则
    加锁的基本单位是Next-key lock；
    查找过程中访问到的对象才会加锁；
    索引上的等值查询，给唯一索引加锁的时候，Next-key lock退化成行锁；
    索引上的等值查询，向右遍历时最后一个值不满足等值条件的时候，Next-key lock退化成间隙锁；

```







## 事务的隔离级别

```
**读未提交(Read uncommitted)**
 即一个事务做出的改变还未提交就能被其他事务所看到

**读提交(Read committed)**
 即一个事务做出的改变需要提交后才能被其他事务看到

**可重复读(Repeatable read)**
 即一个事务执行的过程中看到的数据总是和这个事务启动时看到数据是一致的；

**串行化(Serializable)**
 对同一行数据，写会加写锁，读会加读锁，当读写锁出现冲突时后访问的事务会等待占用锁事务执行完成才会继续完成；
```

MVCC只会在`读提交`和`可重复读`这两个隔离级别下才会存在，读未提交是直接返回该行最新的数据，而串行化是直接采用加锁的方式避免并行访问。



## MySQL的语句执行顺序

开始->FROM子句->WHERE子句->GROUP BY子句->HAVING子句->ORDER BY子句->SELECT子句->LIMIT子句->最终结果 



(1)from 

(2) on 

(3) join 

(4) where 

(5)group by(开始使用select中的别名，后面的语句中都可以使用)
(6) avg,sum.... 
(7)having 
(8) select 
(9) distinct 

(10) order by
(11) limit 
————————————————

1. **FORM**: 首先对from子句中的前两个表执行一个笛卡尔乘积，此时生成虚拟表 vt1（选择相对小的表做基础表） 

2. **ON**: 对虚表VT1进行ON筛选，只有那些符合<join-condition>的行才会被记录在虚表VT2中。

3. **JOIN**： 如果指定了OUTER JOIN（比如left join、 right  join），那么保留表中未匹配的行就会作为外部行添加到虚拟表VT2中，产生虚拟表VT3, rug  from子句中包含两个以上的表的话，那么就会对上一个join连接产生的结果VT3和下一个表重复执行步骤1~3这三个步骤，一直到处理完所有的表为止。

4. **WHERE**： 对虚拟表VT3进行WHERE条件过滤。只有符合<where-condition>的记录才会被插入到虚拟表VT4中。

5. **GROUP BY**: 根据group by子句中的列，对VT4中的记录进行分组操作，产生VT5.

6. **CUBE | ROLLUP**: 对表VT5进行cube或者rollup操作，产生表VT6.

7. **HAVING**： 对虚拟表VT6应用having过滤，只有符合<having-condition>的记录才会被 插入到虚拟表VT7中。

8. **SELECT**： 执行select操作，选择指定的列，插入到虚拟表VT8中。

9. **DISTINCT**： 对VT8中的记录进行去重。产生虚拟表VT9.

10. **ORDER BY**: 将虚拟表VT9中的记录按照<order_by_list>进行排序操作，产生虚拟表VT10.

11. **LIMIT**：取出指定行的记录，产生虚拟表VT11, 并将结果返回。

    



## 主从复制模式，同步异步半同步

[https://zhuanlan.zhihu.com/p/50597960

## 方案

### mysql 分页

https://www.jianshu.com/p/59a28a0a88aa

###### 一、基于偏移的分页

> 例如： `http://XXXXXXXlist?page=1&count=20`
>
> **缺点：** 1、数据重复 2、数据缺失 3、效率低
>
> 使用limit在数据量小的时候并不会有效率问题，但是当数据偏移量很大时性能会开始急剧下降，查询性能比对会在接下来提到。
>
> 综上所述，流式分页不需要也不适合使用传统分页，而是使用一种更合适的方法：游标分页。
>
> 所以这种分页方法 适用于 数据部经常变化的情况。或者拿到数据后做去重处理。

###### 二、基于游标分页方案

>基于游标的分页是最有效的分页方法，应尽量使用这种方法。游标是指标记数据列表中特定项目的一个随机字符串。该项目未被删除时，游标将始终指向列表的同一部分，项目被删除时游标即失效。因此，您的应用不应存储任何旧的游标，也不能假定它们仍然有效。
>
>游标分页的连线支持以下参数：
>
>- **before**：这是指向已返回的数据页面开头的游标。
>- **after**：这是指向已返回的数据页面末尾的游标。
>- **limit**：这是每个页面返回的单独对象的数量。请注意：这是上限。如果数据列表中没有足够的剩余对象，那么返回的数量将小于这个数。为提升性能，将为某些连线的 limit 值设置上限。在调用案例中，API 将返回正确的分页链接。
>- **next**：将返回下一页数据的图谱 API 端点。如果未包含，则显示的是最后一页数据。根据分页的可见性和隐私，页面可能为空，但包含“next”分页链接。“next”链接不再出现时，应停止分页。
>- **previous**：将返回上一页数据的图谱 API 端点。如果未包含，则显示的是第一页数据。
>
>

##### 三、基于时间的分页

>  时间分页使用指向数据列表中的特定时间的 Unix 时间戳在结果数据中导航。
>
>  基于时间的连线支持以下参数：

- until：指向基于时间的数据范围末尾的 Unix 时间戳或 `strtotime`
  数据值。
- since：指向基于时间的数据范围开头的 Unix 时间戳或 `strtotime`
  数据值。
- limit：这是每个页面返回的单独对象的数量。限值 0 将不返回结果。为提升性能，将为某些连线的 `limit`值设置上限。在调用案例中，API 将返回正确的分页链接。
- next：将返回下一页数据的图谱 API 端点。
- previous：将返回上一页数据的图谱 API 端点。

![image-20200706190650629](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200706190650629.png)



https://juejin.im/entry/5b5eb7f2e51d4519700f7d3c

## 分页

```sql
SELECT 查询列表 FROM 表名 WHERE 筛选条件
GROUP BY 分组条件 HAVING 分组后筛选
ORDER BY 排序列表
LIMIT 起始索引，显示的条目数
```

要显示的是第p页，每页显示s条数据， 则limit后参数为(p-1)*s



## 优化分页查询

**优化思路一**
在索引上完成排序分页操作，最后根据主键关联回原表查询所需要的其他列内容。

**优化思路二**
该方案适用于主键自增的表，可以把Limit 查询转换成某个位置的查询 。

```sql
SELECT * FROM tb_user_1 WHERE id> 900000 LIMIT 10;
相当于
SELECT * FROM tb_user_1 WHERE id BETWEEN 900001 AND 900010;
```



## 视图作用

```
是一种虚拟存在的表，行和列的数据来自定义视图的查询中使用的表，并且是在使用视图时动态生成的，只保存了sql逻辑，不保存查询结果。
实现了sql语句的重用
简化了复杂的sql操作，不必知道其细节
保护数据，提高安全性
```

不可更新的视图

- 包含以下关键字: 分组函数,DISTINCT,GROUP BY,HAVING,UNION,UNION ALL

- 常量视图，例: 

  ```sql
  CREATE VIEW v AS SELECT 'abc' NAME;
  ```

- SELECT 中包含子查询，例: 	

  ```sql
  CREATE VIEW v AS SELECT(SELECT 1) AS number;
  ```

- JOIN，例: 

  ```sql
  CREATE VIEW v AS 
  SELECT * FROM employees e JOIN departments d
  ON e.department_id=d.department_id;
  ```

- FROM 一个不能更新的视图，例：

  ```sql
  CREATE VIEW v2 AS 
  SELECT * FROM v;
  ```

- WHERE 子句的子查询引用了 FROM 子句中的表，例：

  ```sql
  CREATE VIEW v AS 
  SELECT * FROM employees
  WHERE employees_id IN (SELECT DISTINCT manager_id FROM employees);
  ```



## MYSQL优化

写日志的操作是磁盘上一小块区域内的顺序I/O，而不像随机I/O需要在磁盘的多个地方移动磁头，所以采用事务日志的方式相对来说要快得多

### undo log ： https://blog.csdn.net/harryptter/article/details/87388164

**插入**: 但通过事务**插入**新的数据时候，数据行版本号（DB_TRX_ID）为当前事务版本号，删除版本号设置为NULL。

**删除**：在删除版本号增加当前系统全局事务ID号

**修改** : 先做命中的数据行的复制,然后间原行数据的删除版本号设置为当前全局事务ID，新的行数据的数据行版本号也设置为当前全局事务ID，删除版本号为NULL

**查找** ：1.寻找数据行版本号小于或者等于当前全局事务ID（这样可以确认事务读取的行在开始前已经存在，或者是事务自身插入或者修改的），2.查找删除版本号为NULL，或者大于当前事务版本号的ID（即确保读取出来的行在此事务开启之前没有被删除）

## JDBC 流程

（1）导入相应的jar包
（2）加载、注册sql驱动
（3）获取Connection连接对象
（4）创建Statement对象并执行SQL语句
（5）使用ResultSet对象获取查询结果集
（6）依次关闭ResultSet、Statement、Connection对象

```java
public class JDBCUtils {
    //获取数据库连接
    public static Connection getConnection() throws Exception {
        //读取配置文件信息
        InputStream in = ClassLoader.getSystemClassLoader().getResourceAsStream("jdbc.properties");
        Properties properties=new Properties();
        properties.load(in);
        String driver = properties.getProperty("driver");
        String url = properties.getProperty("url");
        String user = properties.getProperty("user");
        String password = properties.getProperty("password");
        //注册驱动
        Class.forName(driver);
        //获取数据库连接
        return DriverManager.getConnection(url, user, password);
    }
```

```java
//通用的增删改操作
    public void commonUpdate(String sql,Object...args){
        Connection connect=null;
        PreparedStatement ps=null;
        try{
            connect = JDBCUtils.getConnection();
            ps = connect.prepareStatement(sql);
            for(int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);//数据库中索引从1开始
            }
            ps.execute();//返回值true代表查询 false代表增删改
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            JDBCUtils.closeConnection(connect,ps);
        }
    }
```

```java
//通用查询操作
    @Test
    public <T>List<T> query(Class<T> clazz,String sql,Object...args){
        Connection connect=null;
        PreparedStatement ps=null;
        ResultSet rs=null;
        List<T> result=new ArrayList<>();
        try{
            connect = JDBCUtils.getConnection();
            ps = connect.prepareStatement(sql);
            for(int i=0;i<args.length;i++){
                ps.setObject(i+1,args[i]);
            }
            ResultSet resultSet = ps.executeQuery();
            ResultSetMetaData metaData = resultSet.getMetaData();
            int columnCount = metaData.getColumnCount();
            while (resultSet.next()){
                T t=clazz.newInstance();
               for(int i=0;i<columnCount;i++){
                   Object value = resultSet.getObject(i + 1);
                   String columnName = metaData.getColumnLabel(i + 1);
                   Field field = clazz.getDeclaredField(columnName);
                   field.setAccessible(true);
                   field.set(t,value);
               }
               result.add(t);
            }
            return result;
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            JDBCUtils.closeConnection(connect,ps,rs);
        }
        return null;
    }
```



-  PreparedStatement 和Statement的区别

> - PreparedStatement代码的可读性和可维护性更强
>
> - PreparedStatement 能实现更高效的批量操作
>
>   - DBServer会对预编译语句提供性能优化。因为预编译语句有可能被重复调用，所以语句在被DBServer的编译器编译后的执行代码被缓存下来，那么下次调用时只要是相同的预编译语句就不需要编译，只要将参数直接传入编译过的语句执行代码中就会得到执行。
>   - 在statement语句中,每执行一次都要对传入的语句编译一次。
>
> - PreparedStatement 可以防止 SQL 注入
>
> - PreparedStatement  可操作Blob类数据
>
>   

- Java与SQL对应数据类型转换表

| Java类型           | SQL类型                  |
| ------------------ | ------------------------ |
| boolean            | BIT                      |
| byte               | TINYINT                  |
| short              | SMALLINT                 |
| int                | INTEGER                  |
| long               | BIGINT                   |
| String             | CHAR,VARCHAR,LONGVARCHAR |
| byte   array       | BINARY  ,    VAR BINARY  |
| java.sql.Date      | DATE                     |
| java.sql.Time      | TIME                     |
| java.sql.Timestamp | TIMESTAMP                |

###  ResultSet

> - PreparedStatement 的 `executeQuery() `方法，查询结果是一个ResultSet 对象
> - ResultSet 对象以逻辑表格的形式封装了执行数据库操作的结果集，ResultSet 接口由数据库厂商提供实现
> - ResultSet 返回的实际上就是一张数据表，有一个指针指向数据表的第一条记录的前面。
> - ResultSet 对象维护了一个指向当前数据行的**游标**，初始的时候，游标在第一行之前，可以通过 ResultSet 对象的` next() `方法移动到下一行。调用 next()方法检测下一行是否存在。若存在，该方法返回
>   true，且指针下移，相当于Iterator对象的 `hasNext()` 和 `next() `方法的结合体。
> - 可以通过调用 getXxx()获取每一列的值。

   **注：Java与数据库交互涉及到的相关Java API中的索引都从1开始。**

### ResultSetMetaData

> - 可用于获取关于 ResultSet 对象中列的类型和属性信息的对象
> - 通过调用ResultSet对象的`getMetaData()`方法获得ResultSetMetaData对象
>   - **getColumnName**(int column)：获取指定列的名称
>   - **getColumnLabel**(int column)：获取指定列的别名
>   - **getColumnCount**()：返回当前 ResultSet 对象中的列数。

### 资源的释放

> - 释放ResultSet, Statement,Connection。
> - 数据库连接（Connection）是非常稀有的资源，用完后必须马上释放，如果Connection不能及时正确的关闭将导致系统问题。Connection的使用原则是**尽量晚创建，尽量早的释放。**
> - 可以在finally中关闭，保证及时其他代码出现异常，资源也一定能被关闭。

### JDBC API小结

- 两种思想

>   - 面向接口编程的思想
>   - ORM思想(object relational mapping)
>     - 一个数据表对应一个java类
>     - 表中的一条记录对应java类的一个对象
>     - 表中的一个字段对应java类的一个属性

- 两种技术

>   - JDBC结果集的元数据：ResultSetMetaData
>
>     - 获取列数：getColumnCount()
>     - 获取列的别名：getColumnLabel()
>
>   - 通过反射，创建指定类的对象，获取指定的属性并赋值
>
>     

在jdbc中有3种执行sql的语句分别是execute，executeQuery和executeUpdate

> execute执行增删改查操作
>
> execute返回的结果是个boolean型，当返回的是true的时候，表明有ResultSet结果集，通常是执行了select操作，当返回的是false时，通常是执行了insert、update、delete等操作。execute通常用于执行不明确的sql语句。
>
> 
>
> executeQuery执行查询操作
>
> executeQuery返回的是ResultSet结果集，通常是执行了select操作。
>
> 
>
> executeUpdate执行增删改操作
>
> executeUpdate返回的是int型，表明受影响的行数，通常是执行了insert、update、delete等操作。




## mysql 死锁，**什么情况下会造成死锁**？



## 如果在创建表时没有显式地定义主键,则InnoDB存储引擎会按如下方式选择或创建主键？

> 如果表没有主键，甚至都没有唯一键索引的话，InnoDB内部会基于一个包含了ROW_ID值的列生成一个隐式的聚簇索引，行都会根据这个ROW_ID排序。ROW_ID是一个6个字节，即48位的单调递增字段。有新数据插入时，就会生成一个新的递增的ROW_ID。所以，根据ROW_ID排序的行，本质上是按照插入顺序排序。

需要说明的是，这个单调递增的ROW_ID，和我们平时申明主键时指定AUTO_INREMENT的实现逻辑是**完全不一样**的。

- 隐式ROW_ID实现

这个在既没有主键，也没有一个非空唯一键的InnoDB表中自动添加的被称为ROW_ID的列，既不能被任何查询访问，也不能被内部（例如基于行的复制）使用。

、、、

##  MySQL主从复制原理的是啥？



https://blog.csdn.net/jaryle/article/details/52002727

MySQL里有一个概念，叫binlog日志，就是每个增删改类的操作，会改变数据的操作，除了更新数据以外，对这个增删改操作还会写入一个日志文件，记录这个操作的日志。

主库将变更写binlog日志，然后从库连接到主库之后，从库有一个IO线程，将主库的binlog日志拷贝到自己本地，写入一个中继日志中。接着从库中有一个SQL线程会从中继日志读取binlog，然后执行binlog日志中的内容，也就是在自己本地再次执行一遍SQL，这样就可以保证自己跟主库的数据是一样的。

这里有一个非常重要的一点，就是从库同步主库数据的过程是串行化的，也就是说主库上并行的操作，在从库上会串行执行。所以这就是一个非常重要的点了，由于从库从主库拷贝日志以及串行执行SQL的特点，在高并发场景下，从库的数据一定会比主库慢一些，是有延时的。所以经常出现，刚写入主库的数据可能是读不到的，要过几十毫秒，甚至几百毫秒才能读取到。而且这里还有另外一个问题，就是如果主库突然宕机，然后恰好数据还没同步到从库，那么有些数据可能在从库上是没有的，有些数据可能就丢失了。

所以mysql实际上在这一块有两个机制，一个是**半同步复制**，用来解决主库数据丢失问题；一个是**并行复制**，用来解决主从同步延时问题。

这个所谓半同步复制，semi-sync复制，指的就是主库写入binlog日志之后，就会将强制此时立即将数据同步到从库，从库将日志写入自己本地的relay log之后，接着会返回一个ack给主库，主库接收到至少一个从库的ack之后才会认为写操作完成了。

所谓并行复制，指的是从库开启多个线程，并行读取relay log中不同库的日志，然后并行重放不同库的日志，这是库级别的并行。

- 主从复制的原理
- 主从延迟问题产生的原因
- 主从复制的数据丢失问题，以及半同步复制的原理
- 并行复制的原理，多库并发重放relay日志，缓解主从延迟问题

![image-20200730005610522](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200730005610522.png)



**半同步模式(mysql semi-sync)**
这种模式下主节点只需要接收到**其中一台**从节点的返回信息，就会commit；否则需要等待直到超时时间然后切换成异步模式再提交；这样做的目的可以使主从数据库的数据延迟缩小，可以提高数据安全性，确保了事务提交后，binlog至少传输到了一个从节点上，不能保证从节点将此事务更新到db中。性能上会有一定的降低，响应时间会变长。如下图所示：

![img](https://pic3.zhimg.com/80/v2-d9ac9c5493d1d772f5bf57ede089f0d5_1440w.jpg)



**l 全同步模式**
全同步模式是指主节点和从节点全部执行了commit并确认才会向客户端返回成功。

l Statement-base Replication (SBR)就是记录sql语句在bin log中，Mysql 5.1.4 及之前的版本都是使用的这种复制格式。优点是只需要记录会修改数据的sql语句到binlog中，减少了binlog日质量，节约I/O，提高性能。缺点是在某些情况下，会导致主从节点中数据不一致（比如sleep(),now()等）。


l Row-based Relication(RBR)是mysql master将SQL语句分解为基于Row更改的语句并记录在bin log中，也就是只记录哪条数据被修改了，修改成什么样。优点是不会出现某些特定情况下的**存储过程、或者函数、或者trigger的调用或者触发无法被正确复制的问题**。缺点是会产生大量的日志，尤其是修改table的时候会让日志暴增,同时增加bin log同步时间。也不能通过bin log解析获取执行过的sql语句，只能看到发生的data变更。

l Mixed-format Replication(MBR)，MySQL NDB cluster 7.3 和7.4 使用的MBR。是以上两种模式的混合，对于一般的复制使用STATEMENT模式保存到binlog，对于STATEMENT模式无法复制的操作则使用ROW模式来保存，MySQL会根据执行的SQL语句选择日志保存方式。

## InnoDB结构

InnoDB存储引擎由内存池和一些后台线程组成，其各自主要的工作是：

**内存池主要工作**

- 维护所有进程/线程需要访问的多个内部数据结构

- 缓存磁盘上的数据，方便快速读取，同时在对磁盘文件修改之前进行缓存

- 缓存重做日志（redo log）

  ![image-20200731135311965](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200731135311965.png)

 **缓冲池**
InnoDB缓冲池是为了通过内存的速度来弥补磁盘速度慢对数据库性能造成的影响。其工作方式总是将数据库文件按页（每页16K）读取到缓冲池，然后按最近最少使用（LRU）的算法来保留在缓冲池中的缓存数据。在数据库中进行读操作时，首先将从磁盘读到的页存放在缓冲池中，下一次读取相同的页时，首先判定是否存在缓冲池中，如果有就是被命中直接读取，没有的话就从磁盘中读取。在数据库进行改操作时，首先修改缓冲池中的页（修改后，该页即为脏页），然后在以一定的频率刷新到磁盘上。这里的刷新机制不是每页在发生变更时触发。而是通过一种checkpoint机制刷新到磁盘的。
所以缓冲池的大小直接影响着数据库的整体性能，可以通过配置参数innodb_buffer_pool_size来设置。
从架构图中可以看出缓冲池中缓存的数据页类型有:索引页、数据页、 undo 页、插入缓冲、自适应哈希索引、 InnoDB 的锁信息、数据字典信息等。索引页和数据页占缓冲池的很大一部分。

- 数据页和索引页: Page是Innodb存储的最基本结构，也是Innodb磁盘管理的最小单位，与数据库相关的所有内容都存储在Page结构里。Page分为几种类型，数据页和索引页就是其中最为重要的两种类型。
- 插入缓存: 在InnoDB引擎上进行插入操作时，一般需要按照主键顺序进行插入，这样才能获得较高的插入性能。当一张表中存在非聚簇的且不唯一的索引时，在插入时，数据页的存放还是按照主键进行顺序存放，但是对于非聚簇索引叶节点的插入不再是顺序的了，这时就需要离散的访问非聚簇索引页，由于随机读取的存在导致插入操作性能下降。
  InnoDB为此设计了Insert Buffer来进行插入优化。对于非聚簇索引的插入或者更新操作，不是每一次都直接插入到索引页中，而是先判断插入的非聚集索引是否在缓冲池中，若在，则直接插入；若不在，则先放入到一个Insert Buffer中。看似数据库这个非聚集的索引已经查到叶节点，而实际没有，这时存放在另外一个位置。然后再以一定的频率和情况进行Insert Buffer和非聚簇索引页子节点的合并操作。这时通常能够将多个插入合并到一个操作中，这样就大大提高了对于非聚簇索引的插入性能。
- 自适应哈希索引: InnoDB会根据访问的频率和模式，为热点页建立哈希索引，来提高查询效率。InnoDB存储引擎会监控对表上各个索引页的查询，如果观察到建立哈希索引可以带来速度上的提升，则建立哈希索引，所以叫做自适应哈希索引。
  自适应哈希索引是通过缓冲池的B+树页构建而来，因此建立速度很快，而且不需要对整张数据表建立哈希索引。其 有一个要求，即对这个页的连续访问模式必须是一样的，也就是说其查询的条件(WHERE)必须完全一样，而且必须是连续的。
- 锁信息 : nnoDB存储引擎会在行级别上对表数据进行上锁。不过InnoDB也会在数据库内部其他很多地方使用锁，从而允许对多种不同资源提供并发访问。数据库系统使用锁是为了支持对共享资源进行并发访问，提供数据的完整性和一致性。关于锁的具体知识我们之后再进行详细学习。
- 数据字典信息 : InnoDB有自己的表缓存，可以称为表定义缓存或者数据字典。当InnoDB打开一张表，就增加一个对应的对象到数据字典。
  数据字典是对数据库中的数据、库对象、表对象等的元信息的集合。在MySQL中，数据字典信息内容就包括表结构、数据库名或表名、字段的数据类型、视图、索引、表字段信息、存储过程、触发器等内容。MySQL INFORMATION_SCHEMA库提供了对数据局元数据、统计信息、以及有关MySQL server的访问信息（例如：数据库名或表名，字段的数据类型和访问权限等）。该库中保存的信息也可以称为MySQL的数据字典。

**重做日志冲池**

InnoDB有buffer pool（简称bp）。bp是数据库页面的缓存，对InnoDB的任何修改操作都会首先在bp的page上进行，然后这样的页面将被标记为dirty并被放到专门的flush list上，后续将由master thread或专门的刷脏线程阶段性的将这些页面写入磁盘（disk or ssd）。这样的好处是避免每次写操作都操作磁盘导致大量的随机IO，阶段性的刷脏可以将多次对页面的修改merge成一次IO操作，同时异步写入也降低了访问的时延。然而，如果在dirty page还未刷入磁盘时，server非正常关闭，这些修改操作将会丢失，如果写入操作正在进行，甚至会由于损坏数据文件导致数据库不可用。为了避免上述问题的发生，Innodb将所有对页面的修改操作写入一个专门的文件，并在数据库启动时从此文件进行恢复操作，这个文件就是redo log file。这样的技术推迟了bp页面的刷新，从而提升了数据库的吞吐，有效的降低了访问时延。带来的问题是额外的写redo log操作的开销（顺序IO，当然很快），以及数据库启动时恢复操作所需的时间。

redo日志由两部分构成：redo log buffer、redo log file。innodb是支持事务的存储引擎，在事务提交时，必须先将该事务的所有日志写入到redo日志文件中，待事务的commit操作完成才算整个事务操作完成。在每次将redo log buffer写入redo log file后，都需要调用一次fsync操作，因为重做日志缓冲只是把内容先写入操作系统的缓冲系统中，并没有确保直接写入到磁盘上，所以必须进行一次fsync操作。因此，磁盘的性能在一定程度上也决定了事务提交的性能。

InnoDB 存储引擎先将重做日志信息放入这个缓冲区,然后以一定频率将其刷新到重做日志文件。重做日志文件一般不需要设置得很大,因为在下列三种情况下重做日志缓冲中的内容会刷新到磁盘的重做日志文件中。

- Master Thread 每一秒将重做日志缓冲刷新到重做日志文件
- 每个事物提交时会将重做日志缓冲刷新到重做日志文件
- 当重做日志缓冲剩余空间小于1/2时,重做日志缓冲刷新到重做日志文件

**额外的缓冲池**
在 InnoDB 存储引擎中，对一些数据结构本身的内存进行分配时，需要从额外的内存池中进行申请。例如: 分配了缓冲池,但是每个缓冲池中的帧缓冲还有对应的缓冲控制对象，这些对象记录以一些诸如 LRU, 锁,等待等信息,而这个对象的内存需要从额外的内存池中申请。

**后台线程主要工作**

- 刷新内存池中的数据，保证缓冲池中缓存的数据最新
- 将已修改数据文件刷新到磁盘文件
- 保证数据库异常时InnoDB能恢复到正常运行状态
- ![image-20200731140027071](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200731140027071.png)



核心的后台线程，主要负责将缓冲池中的数据异步刷新到磁盘，保证数据的一致性，包括脏页的刷新、合并插入缓冲、undo页的回收等。
Master thread在主循环中，分两大部分操作，每秒钟的操作和每10秒钟的操作：

- **每秒一次的操作**

- - 日志缓冲刷新到磁盘: 即使这个事务还没有提交（总是），这点解释了为什么再大的事务commit时都很快；
  - 合并插入缓冲（可能）: 合并插入并不是每秒都发生，InnoDB会判断当前一秒内发生的IO次数是否小于5，如果是，则系统认为当前的IO压力很小，可以执行合并插入缓冲的操作。
  - 至多刷新100个InnoDB的缓冲池的脏页到磁盘（可能) : 这个刷新100个脏页也不是每秒都在做，InnoDB引擎通过判断当前缓冲池中脏页的比例(buf_get_modified_ratio_pct)是否超过了配置文件中innodb_max_drity_pages_pct参数(默认是90，即90%)，如果超过了这个阈值，InnoDB引擎认为需要做磁盘同步操作，将100个脏页写入磁盘。

- **每10秒一次的操作**

- - 刷新100个脏页到磁盘（可能）: InnoDB引擎先判断过去10秒内磁盘的IO操作是否小于200次，如果是，认为当前磁盘有足够的IO操作能力，即将100个脏页刷新到磁盘。
  - 合并至多5个插入缓冲（总是）: 此次的合并插入缓冲操作总会执行，不同于每秒操作时可能发生的合并操作。
  - 将日志缓冲刷新到磁盘（总是）: InnoDB引擎会再次执行日志缓冲刷新到磁盘的操作，与每秒发生的操作一样。
  - 删除无用的undo页（总是）: 当对表执行update，delete操作时，原先的行会被标记为删除，但是为了一致性读的关系，需保留这些行版本的信息，在进行10S一次的删除操作时，InnoDB引擎会判断当前事务系统中已被删除的行是否可以删除，如果可以，InnoDB会立即将其删除。InnoDB每次最多删除20个Undo页。
  - 产生一个检查点（checkpoing）；



**IO threads**
在 InnoDB 存储引擎中大量使用了异步 IO 来处理写 IO 请求，IO Thread 的工作主要是负责这些 IO 请求的回调.。分别为write、read、insert buffer和log IO thread。线程数量可以通过参数进行调整。5.6以后的版本可以通过innodb_write_io_threads和innodb_read_io_threads来限制读写线程，而在5.6版本以前，只有一个参数innodb_file_io_threads来控制读写总线程数。

**purge threads**
负责回收已经使用并分配的undo页，purge操作默认是由master thread中完成的，为了减轻master thread的工作，提高cpu使用率以及提升存储引擎的性能。用户可以在参数文件中添加如下命令来启动独立的purge thread。
innodb_purge_threads=1
从innodb1.2版本开始，可以指定多个innodb_purge_threads来进一步加快和提高undo回收速度。

**page cleaner threads**
Page Cleaner Thread是在InnoDB1.2.X版本中引入的。其作用是将之前版本中脏页的刷新操作都放入到单独的线程中来完成。 其目的是减轻master thread的工作以及对于用户查询线程的阻塞，进一步提高InnoDB存储引擎的性能。

## 三、InnoDB重要特性

MySQL InnoDB通过如下重要特性实现了更好的新能和更高的特性

- 插入缓冲（insert buffer）
- 两次写（Double write）
- 自适应哈希索引（adaptive hash index）
- 异步io（Async IO）
- 刷新领接页（Flush Neighbor Page）

# MYBATIS

https://juejin.im/entry/59127ad4da2f6000536f64d8



# JVM

### 内存区域划分 8

#### Q1：运行时数据区是什么？

虚拟机在执行 Java 程序的过程中会把它所管理的内存划分为若干不同的数据区，这些区域有各自的用途、创建和销毁时间。

线程私有：程序计数器、Java 虚拟机栈、本地方法栈。

线程共享：Java 堆、方法区。

------

#### Q2：程序计数器是什么？

**程序计数器**是一块较小的内存空间，可以看作当前线程所执行字节码的行号指示器。字节码解释器工作时通过改变计数器的值选取下一条执行指令。分支、循环、跳转、线程恢复等功能都需要依赖计数器完成。是唯一在虚拟机规范中没有规定内存溢出情况的区域。

如果线程正在执行 Java 方法，计数器记录正在执行的虚拟机字节码指令地址。如果是本地方法，计数器值为 Undefined。

```java
public class Test {
    private void run() {
        List<String> list = new ArrayList<>(); // invokespecial 构造器调用
        list.add("a"); // invokeinterface 接口调用 
        ArrayList<String> arrayList = new ArrayList<>(); // invokespecial 构造器调用
        arrayList.add("b"); // invokevirtual 虚函数调用
    }
    public static void main(String[] args) {
        Test test = new Test(); // invokespecial 构造器调用
        test.run(); // invokespecial 私有函数调用
    }
}
————————————————
版权声明：本文为CSDN博主「TellH」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/TellH/java/article/details/77370223
```

垃圾回收

4种算法： 

好多垃圾回收器

方法区

![image-20200809220615632](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200809220615632.png)



![image-20200809220822476](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200809220822476.png)



![image-20200809220839717](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200809220839717.png)

JVM 深入了解java 虚拟机 Page 39 -- 49 jvm结构

![image-20200809170725658](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200809170725658.png)

![image-20200809170812538](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200809170812538.png)

# JAVA 

1. ### Synchronized同步静态方法和非静态方法？

   **Synchronized修饰非静态方法，实际上是对调用该方法的对象加锁，俗称“对象锁”。**

   **Synchronized修饰静态方法，实际上是对该类对象加锁，俗称“类锁”。**

   

       1.对象锁钥匙只能有一把才能互斥，才能保证共享变量的唯一性
       
       2.在静态方法上的锁，和 实例方法上的锁，默认不是同样的，如果同步需要制定两把锁一样。
       
       3.关于同一个类的方法上的锁，来自于调用该方法的对象，如果调用该方法的对象是相同的，那么锁必然相同，否则就不相同。比如 new A().x() 和 new A().x(),对象不同，锁不同，如果A的单利的，就能互斥。
       
       4.静态方法加锁，能和所有其他静态方法加锁的 进行互斥
       
       5.静态方法加锁，和xx.class 锁效果一样，直接属于类的
   
2. ## JAVA 中有以下几个『元注解』：

   - @Target：注解的作用目标
   
     ElementType.TYPE：允许被修饰的注解作用在类、接口和枚举上
   
     ElementType.FIELD：允许作用在属性字段上
   
     ElementType.METHOD：允许作用在方法上
   
     ElementType.PARAMETER：允许作用在方法参数上
   
     ElementType.CONSTRUCTOR：允许作用在构造器上
   
     ElementType.LOCAL_VARIABLE：允许作用在本地局部变量上
   
     ElementType.ANNOTATION_TYPE：允许作用在注解上
   
     ElementType.PACKAGE：允许作用在包上
   
     
   
   - @Retention：注解的生命周期
   
     RetentionPolicy.SOURCE：当前注解编译期可见，不会写入 class 文件
   
     RetentionPolicy.CLASS：类加载阶段丢弃，会写入 class 文件
   
     RetentionPolicy.RUNTIME：永久保存，可以反射获取
   
     ​	
   
   - @Documented：注解是否应当被包含在 JavaDoc 文档中
   
   - @Inherited：是否允许子类继承该注解
   
3. ## ConcurrentHashMap put 方法

   1. 检查key ， value是否为null ， null 直接抛出异常
   2. 计算hashcode，
   3. 检查数组是否初始化，没有就调用initTable
   4. 检查table【0】 是否为null， null 则直接用 cas 设置为new Node（k，v）
   5. 如果当前位置的 `hashcode == MOVED == -1`,则需要进行扩容。
   6. 如果都不满足，则利用 synchronized 锁写入数据
   7. 如果数量大于 `TREEIFY_THRESHOLD` 则要转换为红黑树。
   
   
   
4. ## get 方法

   - 根据计算出来的 hashcode 寻址，如果就在桶上那么直接返回值。
   - 如果是红黑树那就按照树的方式获取值。
   - 就不满足那就按照链表的方式遍历获取值。
   -  *//hash值为负值表示正在扩容，这个时候查的是ForwardingNode的find方法来定位到nextTable来*        *//eh=-1，说明该节点是一个ForwardingNode，正在迁移，此时调用ForwardingNode的find方法去nextTable里找。*        *//eh=-2，说明该节点是一个TreeBin，此时调用TreeBin的find方法遍历红黑树，由于红黑树有可能正在旋转变色，所以find里会有读写锁。*        *//eh>=0，说明该节点下挂的是一个链表，直接遍历该链表即可。*

5. ## 多态的原理

   多态允许具体访问时实现方法的动态绑定。Java对于动态绑定的实现主要依赖于方法表，通过继承和接口的多态实现有所不同。

   

# JUC

# AQS

https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html

何时出队列？”和“如何出队列？

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

final boolean acquireQueued(final Node node, int arg) {
	// 标记是否成功拿到资源
	boolean failed = true;
	try {
		// 标记等待过程中是否中断过
		boolean interrupted = false;
		// 开始自旋，要么获取锁，要么中断
		for (;;) {
			// 获取当前节点的前驱节点
			final Node p = node.predecessor();
			// 如果p是头结点，说明当前节点在真实数据队列的首部，就尝试获取锁（别忘了头结点是虚节点）
			if (p == head && tryAcquire(arg)) {
				// 获取锁成功，头指针移动到当前node
				setHead(node);
				p.next = null; // help GC
				failed = false;
				return interrupted;
			}
			// 说明p为头节点且当前没有获取到锁（可能是非公平锁被抢占了）或者是p不为头结点，这个时候就要判断当前node是否要被阻塞（被阻塞条件：前驱节点的waitStatus为-1），防止无限循环浪费资源。具体两个方法下面细细分析
			if (shouldParkAfterFailedAcquire(p, node) && parkAndCheckInterrupt())
				interrupted = true;
		}
	} finally {
		if (failed)
			cancelAcquire(node);
	}
}
```

```java
private void setHead(Node node) {
	head = node;
	node.thread = null;
	node.prev = null;
}

// java.util.concurrent.locks.AbstractQueuedSynchronizer

// 靠前驱节点判断当前线程是否应该被阻塞
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
	// 获取头结点的节点状态
	int ws = pred.waitStatus;
	// 说明头结点处于唤醒状态
	if (ws == Node.SIGNAL)
		return true; 
	// 通过枚举值我们知道waitStatus>0是取消状态
	if (ws > 0) {
		do {
			// 循环向前查找取消节点，把取消节点从队列中剔除
			node.prev = pred = pred.prev;
		} while (pred.waitStatus > 0);
		pred.next = node;
	} else {
		// 设置前任节点等待状态为SIGNAL
		compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
	}
	return false;
}
```



```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

public final boolean release(int arg) {
	// 上边自定义的tryRelease如果返回true，说明该锁没有被任何线程持有
	if (tryRelease(arg)) {
		// 获取头结点
		Node h = head;
		// 头结点不为空并且头结点的waitStatus不是初始化节点情况，解除线程挂起状态
		if (h != null && h.waitStatus != 0)
			unparkSuccessor(h);
		return true;
	}
	return false;
}
```

```java
// java.util.concurrent.locks.AbstractQueuedSynchronizer

private void unparkSuccessor(Node node) {
	// 获取头结点waitStatus
	int ws = node.waitStatus;
	if (ws < 0)
		compareAndSetWaitStatus(node, ws, 0);
	// 获取当前节点的下一个节点
	Node s = node.next;
	// 如果下个节点是null或者下个节点被cancelled，就找到队列最开始的非cancelled的节点
	if (s == null || s.waitStatus > 0) {
		s = null;
		// 就从尾部节点开始找，到队首，找到队列第一个waitStatus<0的节点。
		for (Node t = tail; t != null && t != node; t = t.prev)
			if (t.waitStatus <= 0)
				s = t;
	}
	// 如果当前节点的下个节点不为空，而且状态<=0，就把当前节点unpark
	if (s != null)
		LockSupport.unpark(s.thread);
}
```



1. ReentrantReadWriteLock 

   **同步状态的低16位用来表示写锁的获取次数**

   **同步状态的高16位用来表示读锁被获取的次数**

   ​      Write Lock  加锁： 

   1. 有共享锁/有独占锁： 直接返还错误

   2. 独占是自己： 重入
   3. CAS 设置同步状态

   ```java
   protected final boolean tryAcquire(int acquires) {
       /*
        * Walkthrough:
        * 1. If read count nonzero or write count nonzero
        *    and owner is a different thread, fail.
        * 2. If count would saturate, fail. (This can only
        *    happen if count is already nonzero.)
        * 3. Otherwise, this thread is eligible for lock if
        *    it is either a reentrant acquire or
        *    queue policy allows it. If so, update state
        *    and set owner.
        */
       Thread current = Thread.currentThread();
   	// 1. 获取写锁当前的同步状态
       int c = getState();
   	// 2. 获取写锁获取的次数
       int w = exclusiveCount(c);
       if (c != 0) {
           // (Note: if c != 0 and w == 0 then shared count != 0)
   		// 3.1 当读锁已被读线程获取或者当前线程不是已经获取写锁的线程的话
   		// 当前线程获取写锁失败
           if (w == 0 || current != getExclusiveOwnerThread())
               return false;
           if (w + exclusiveCount(acquires) > MAX_COUNT)
               throw new Error("Maximum lock count exceeded");
           // Reentrant acquire
   		// 3.2 当前线程获取写锁，支持可重复加锁
           setState(c + acquires);
           return true;
       }
   	// 3.3 写锁未被任何线程获取，当前线程可获取写锁
       if (writerShouldBlock() ||
           !compareAndSetState(c, c + acquires))
           return false;
       setExclusiveOwnerThread(current);
       return true;
   }
   ```

   释放

   ```java
   protected final boolean tryRelease(int releases) {
       if (!isHeldExclusively())
           throw new IllegalMonitorStateException();
   	//1. 同步状态减去写状态
       int nextc = getState() - releases;
   	//2. 当前写状态是否为0，为0则释放写锁
       boolean free = exclusiveCount(nextc) == 0;
       if (free)
           setExclusiveOwnerThread(null);
   	//3. 不为0则更新同步状态
       setState(nextc);
       return free;
   }
   
   ```

   只需要用**当前同步状态直接减去写状态的原因正是我们刚才所说的写状态是由同步状态的低16位表示的**。

   ## 读锁的获取

   \1. If write lock held by another thread, fail. *

   \ 2. Otherwise, this thread is eligible for * lock wrt state, so ask if it should block * because of queue policy. If not, try * to grant by CASing state and updating count. Note that step does not check for reentrant * acquires, which is postponed to full version * to avoid having to check hold count in * the more typical non-reentrant case. * 

   \3. If step 2 fails either because thread * apparently not eligible or CAS fails or count * saturated, chain to version with full retry loop.

   
   
2. 

3. 

https://zhuanlan.zhihu.com/p/137624149

https://juejin.im/post/5ae755606fb9a07ab97942a4

# 算法

1. 洗牌算法，**设计一个公平的洗牌算法**

   
   
   ```pseudocode
   -- To shuffle an array a of n elements (indices 0..n-1):
   for i from n − 1 downto 1 do
        j ← random integer such that 0 ≤ j ≤ i
        exchange a[j] and a[i]
   ```
   
2. 哈夫曼编码

   https://blog.csdn.net/FX677588/article/details/70767446





# 操作系统

上下文切换： https://juejin.im/post/5c872a255188257e3f1afaef



## 以下主要说明Swap机制：

在Linux下，SWAP的作用类似Windows系统下的“虚拟内存”。当物理内存不足时，拿出部分硬盘空间当SWAP分区（虚拟成内存）使用，从而解决内存容量不足的情况。

SWAP意思是交换，顾名思义，**当某进程向OS请求内存发现不足时，OS会把内存中暂时不用的数据交换出去，放在SWAP分区中，这个过程称为SWAP OUT**。**当某进程又需要这些数据且OS发现还有空闲物理内存时，又会把SWAP分区中的数据交换回物理内存中，这个过程称为SWAP IN**。

当然，swap大小是有上限的，一旦swap使用完，操作系统会触发OOM-Killer机制，把消耗内存最多的进程kill掉以释放内存。





## kill pid与kill -9 pid的区别

​        kill pid的作用是向进程号为pid的进程发送SIGTERM（这是kill默认发送的信号），该信号是一个结束进程的信号且可以被应用程序捕获。若应用程序没有捕获并响应该信号的逻辑代码，则该信号的默认动作是kill掉进程。这是终止指定进程的推荐做法。
​        kill -9 pid则是向进程号为pid的进程发送SIGKILL（该信号的编号为9），从本文上面的说明可知，SIGKILL既不能被应用程序捕获，也不能被阻塞或忽略，其动作是立即结束指定进程。通俗地说，应用程序根本无法“感知”SIGKILL信号，它在完全无准备的情况下，就被收到SIGKILL信号的操作系统给干掉了，显然，在这种“暴力”情况下，应用程序完全没有释放当前占用资源的机会。事实上，SIGKILL信号是直接发给init进程的，它收到该信号后，负责终止pid指定的进程。

---



## **进程上下文**

​    所谓的进程上下文，就是一个进程在执行的时候，CPU的所有寄存器中的值、进程的状态以及堆栈上的内容，当内核需要切换到另一个进程时，它需要保存当前进程的所有状态，即保存当前进程的进程上下文，以便再次执行该进程时，能够恢复切换时的状态，继续执行。

​    一个进程的上下文可以分为三个部分:**用户级上下文**、**寄存器上下文**以及**系统级上下文**。

* 用户级上下文: 正文、数据、用户堆栈以及共享存储区；

* 寄存器上下文: 通用寄存器、程序寄存器(IP)、处理器状态寄存器(EFLAGS)、栈指针(ESP)；

* 系统级上下文: 进程控制块task_struct、内存管理信息(mm_struct、vm_area_struct、pgd、pte)、内核栈。

​    当发生进程调度时，进行进程切换就是上下文切换(context switch)。

​    操作系统必须对上面提到的全部信息进行切换，新调度的进程才能运行。

而**系统调用进行的是模式切换(mode switch)**。模式切换与进程切换比较起来，容易很多，而且节省时间，因为**模式切换最主要的任务只是切换进程寄存器上下文的切换**。进程上下文主要是异常处理程序和内核线程。内核之所以进入进程上下文是因为进程自身的一些工作需要在内核中做。例如，系统调用是为当前进程服务的，异常通常是处理进程导致的错误状态等。所以在进程上下文中引用current是有意义的。

## 中断上下文具体包括：

（1）硬件传递过来的参数

因此上下文切换可以分为以下几类：

（1）进程之间的上下文切换：A进程切换到B进程

（2）进程和中断之间的上下文切换：进程A被中断打断

（3）中断之间的上下文切换：低级别中断被高级别中断打断

### **模式切换**

这是要说一种特殊的上下文切换：模式切换，即进程A从用户态因为系统调用进入内核态，这种切换之所以特殊，是因为它并没有经过完整的上下文切换，只是寄存器上下文进行了切换，所以模式切换的耗时相对完整进程上下文更低。

https://blog.csdn.net/zqixiao_09/article/details/50877756

## 进程切换分两步

1.切换**页目录以使用新的地址空间**

2.切换**内核栈和硬件上下文**。

对于linux来说，线程和进程的最大区别就在于地址空间。
对于线程切换，第1步是不需要做的，第2是进程和线程切换都要做的。所以明显是进程切换代价大

线程上下文切换和进程上下文切换一个最主要的区别是线程的切换虚拟内存空间依然是相同的，但是进程切换是不同的。这两种上下文切换的处理都是通过操作系统内核来完成的。内核的这种切换过程伴随的最显著的性能损耗是将寄存器中的内容切换出.

另外一个隐藏的损耗是上下文的切换会扰乱处理器的缓存机制。简单的说，一旦去切换上下文，处理器中所有已经缓存的内存地址一瞬间都作废了。还有一个显著的区别是当你改变虚拟内存空间的时候，处理的页表缓（processor’s Translation Lookaside Buffer (TLB)）或者相当的神马东西会被全部刷新，这将导致内存的访问在一段时间内相当的低效。但是在线程的切换中，不会出现这个问题。

## 进程上下文切换跟系统调用又有什么区别呢

首先，**进程是由内核来管理和调度的，进程的切换只能发生在内核态**。所以，进程的上下文不仅包括了虚拟内存、栈、全局变量等用户空间的资源，还包括了内核堆栈、寄存器等内核空间的状态。

因此，**进程的上下文切换就比系统调用时多了一步：在保存内核态资源（当前进程的内核状态和 CPU 寄存器）之前，需要先把该进程的用户态资源（虚拟内存、栈等）保存下来；而加载了下一进程的内核态后，还需要刷新进程的虚拟内存和用户栈**。

如下图所示，保存上下文和恢复上下文的过程并不是“免费”的，需要内核在 CPU 上运行才能完成。

 Linux 通过 TLB（Translation Lookaside Buffer）来管理虚拟内存到物理内存的映射关系。当虚拟内存更新后，TLB 也需要刷新，内存的访问也会随之变慢。特别是在多处理器系统上，缓存是被多个处理器共享的，刷新缓存不仅会影响当前处理器的进程，还会影响共享缓存的其他处理器的进程。



![img](https://pic2.zhimg.com/80/v2-440bb1699b2fa0f0340b38eabcbd7452_1440w.jpg)

## 进程上下文切换的场景

1. 为了保证所有进程可以得到公平调度，CPU 时间被划分为一段段的时间片，这些时间片再被轮流分配给各个进程。这样，当某个进程的时间片耗尽了，就会被系统挂起，切换到其它正在等待 CPU 的进程运行。
2. 进程在系统资源不足（比如内存不足）时，要等到资源满足后才可以运行，这个时候进程也会被挂起，并由系统调度其他进程运行。
3. 当进程通过睡眠函数 sleep 这样的方法将自己主动挂起时，自然也会重新调度。
4. 当有优先级更高的进程运行时，为了保证高优先级进程的运行，当前进程会被挂起，由高优先级进程来运行
5. 发生硬件中断时，CPU 上的进程会被中断挂起，转而执行内核中的中断服务程序。

## 线程上下文切换

线程与进程最大的区别在于：线程是调度的基本单位，而进程则是资源拥有的基本单位。说白了，所谓内核中的任务调度，实际上的调度对象是线程；而进程只是给线程提供了虚拟内存、全局变量等资源。

所以，对于线程和进程，我们可以这么理解：

- 当进程只有一个线程时，可以认为进程就等于线程。
- 当进程拥有多个线程时，这些线程会共享相同的虚拟内存和全局变量等资源。这些资源在上下文切换时是不需要修改的。
-  另外，线程也有自己的私有数据，比如栈和寄存器等，这些在上下文切换时也是需要保存的。。







## INODE

stat -x file 

![image-20200724230508284](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200724230508284.png)

## fork的实现分为以下两步

\1. 复制进程资源
\2. 执行该进程

复制进程的资源包括以下几步
\1. 进程pcb
\2. 程序体，即代码段数据段等
\3. 用户栈
\4. 内核栈
\5. 虚拟内存池
\6. 页表

- 子进程用于独立且唯一的进程 ID；
- 子进程的父进程 ID 与父进程 ID 完全相同；
- 子进程不会继承父进程的内存锁；
- 子进程会重新设置进程资源利用率和 CPU 计时器；



fork后子进程和父进程共享的资源还包括：

* 打开的文件

* 实际用户ID、实际组ID、有效用户ID、有效组ID

* 添加组ID

* 进程组ID

* 会话期ID

* 控制终端

* 设置-用户-ID标志和设置-组-ID标志

* 当前工作目录

* 根目录

* 文件方式创建屏蔽字

* 信号屏蔽和排列

* 对任一打开文件描述符的在执行时关闭标志

* 环境

* 连接的共享存储段（共享内存）

* 资源限制

  父子进程之间的区别是：

* fork的返回值

* 进程ID

* 不同的父进程ID

* 子进程的tms_utime，tms_stime，tms-cutime以及tms_ustime设置为0

* 父进程设置的锁，子进程不继承

* 子进程的未决告警被清除

* 子进程的未决信号集设置为空集

  

## 进程的调度

1 先来先服务调度算法
先来先服务(FCFS)调度算法是一种最简单的调度算法，该算法既可用于作业调度，也可用于进程调度。当在作业调度中采用该算法时，每次调度都是从后备作业队列中选择一个或多个最先进入该队列的作业，将它们调入内存，为它们分配资源、创建进程，然后放入就绪队列。在进程调度中采用FCFS算法时，则每次调度是从就绪队列中选择一个最先进入该队列的进程，为之分配处理机，使之投入运行。该进程一直运行到完成或发生某事件而阻塞后才放弃处理机。

 

2、短作业(进程)优先调度算法
短作业(进程)优先调度算法，是指对短作业或短进程优先调度的算法。它们可以分别用于作业调度和进程调度。短作业优先(SJF)的调度算法是从后备队列中选择一个或若干个估计运行时间最短的作业，将它们调入内存运行。而短进程优先(SPF)调度算法则是从就绪队列中选出一个估计运行时间最短的进程，将处理机分配给它，使它立即执行并一直执行到完成，或发生某事件而被阻塞放弃处理机时再重新调度。

 

3、时间片轮转法
在早期的时间片轮转法中，系统将所有的就绪进程按先来先服务的原则排成一个队列，每次调度时，把CPU分配给队首进程，并令其执行一个时间片。时间片的大小从几ms到几百ms。当执行的时间片用完时，由一个计时器发出时钟中断请求，调度程序便据此信号来停止该进程的执行，并将它送往就绪队列的末尾；然后，再把处理机分配给就绪队列中新的队首进程，同时也让它执行一个时间片。这样就可以保证就绪队列中的所有进程在一给定的时间内均能获得一时间片的处理机执行时间。换言之，系统能在给定的时间内响应所有用户的请求。

 

4、多级反馈队列调度算法
前面介绍的各种用作进程调度的算法都有一定的局限性。如短进程优先的调度算法，仅照顾了短进程而忽略了长进程，而且如果并未指明进程的长度，则短进程优先和基于进程长度的抢占式调度算法都将无法使用。而多级反馈队列调度算法则不必事先知道各种进程所需的执行时间，而且还可以满足各种类型进程的需要，因而它是目前被公认的一种较好的进程调度算法。在采用多级反馈队列调度算法的系统中，调度算法的实施过程如下所述：

1）应设置多个就绪队列，并为各个队列赋予不同的优先级。第一个队列的优先级最高，第二个队列次之，其余各队列的优先权逐个降低。该算法赋予各个队列中进程执行时间片的大小也各不相同，在优先权愈高的队列中，为每个进程所规定的执行时间片就愈小。例如，第二个队列的时间片要比第一个队列的时间片长一倍，第i+1个队列的时间片要比第i个队列的时间片长一倍。

2）当一个新进程进入内存后，首先将它放入第一队列的末尾，按FCFS原则排队等待调度。当轮到该进程执行时，如它能在该时间片内完成，便可准备撤离系统；如果它在一个时间片结束时尚未完成，调度程序便将该进程转入第二队列的末尾，再同样地按FCFS原则等待调度执行；如果它在第二队列中运行一个时间片后仍未完成，再依次将它放入第三队列，……，如此下去，当一个长作业(进程)从第一队列依次降到第n队列后，在第n队列便采取按时间片轮转的方式运行。

3）仅当第一队列空闲时，调度程序才调度第二队列中的进程运行；仅当第1～(i-1)队列均空时，才会调度第i队列中的进程运行。如果处理机正在第i队列中为某进程服务时，又有新进程进入优先权较高的队列(第1～(i-1)中的任何一个队列)，则此时新进程将抢占正在运行进程的处理机，即第i队列中某个正在运行的进程的时间片用完后，由调度程序选择优先权较高的队列中的那一个进程，把处理机分配给它。

优先权调度算法
为了照顾紧迫型作业，使之在进入系统后便获得优先处理，引入了最高优先权优先(FPF)调度算法。此算法常被用于批处理系统中，作为作业调度算法，也作为多种操作系统中的进程调度算法，还可用于实时系统中。当把该算法用于作业调度时，系统将从后备队列中选择若干个优先权最高的作业装入内存。当用于进程调度时，该算法是把处理机分配给就绪队列中优先权最高的进程，这时，又可进一步把该算法分成如下两种。

1) 非抢占式优先权算法

在这种方式下，系统一旦把处理机分配给就绪队列中优先权最高的进程后，该进程便一直执行下去，直至完成；或因发生某事件使该进程放弃处理机时，系统方可再将处理机重新分配给另一优先权最高的进程。这种调度算法主要用于批处理系统中；也可用于某些对实时性要求不严的实时系统中。

2) 抢占式优先权调度算法

在这种方式下，系统同样是把处理机分配给优先权最高的进程，使之执行。但在其执行期间，只要又出现了另一个其优先权更高的进程，进程调度程序就立即停止当前进程(原优先权最高的进程)的执行，重新将处理机分配给新到的优先权最高的进程。因此，在采用这种调度算法时，是每当系统中出现一个新的就绪进程i时，就将其优先权Pi与正在执行的进程j的优先权Pj进行比较。如果Pi≤Pj，原进程Pj便继续执行；但如果是Pi>Pj，则立即停止Pj的执行，做进程切换，使i进程投入执行。显然，这种抢占式的优先权调度算法能更好地满足紧迫作业的要求，故而常用于要求比较严格的实时系统中，以及对性能要求较高的批处理和分时系统中。

Linux 从整体上区分实时进程和普通进程，因为实时进程和普通进程度调度是不同的，它们两者之间，实时进程应该先于普通进程而运行，然后，对于同一类型的不同进程，采用不同的标准来选择进程。对普通进程的调度策略是动态优先调度，对于实时进程采用了两种调度策略，FIFO(先来先服务调度)和RR（时间片轮转调度）。

UNIX系统是单纯的分时系统，所以没有设置作业调度。UNIX系统的进程调度采用的算法是，多级反馈队列调度法。其核心思想是先从最高休先级就绪队列中取出排在队列最前面的进程，当进程执行完一个时间片仍未完成则剥夺它的执行，将它放入到相应的队列中，取出下一个就绪进程投入运行，对于同一个队列中的各个进程，按照时间片轮转法调度。多级反馈队列调度算法即能使高优先级的作业得到响应又能使短作业（进程）迅速完成。但是它还是存在某些方面的不足，当不断有新进程到来时，则长进程可能饥饿。

# 完全公平调度算法CFS


原文链接：https://blog.csdn.net/fuzhongmin05/java/article/details/55802925

https://blog.csdn.net/XD_hebuters/article/details/79623130

## 进程的组成

 **进程是由程序控制块（PCB）、程序段、数据段组成。**
  操作系统是通过PCB来管理进程，因此PCB中应该包含操作系统对其进行管理所需的各种信息，如进程描述信息、进程控制和管理信息、资源分配清单和处理机相关信息。
  程序段：程序代码存放的位置。
  数据段：程序运行时使用、产生的运算数据。如全局变量、局部变量、宏定义的常量就存放在数据段内。

PCB 里面 task—Struct

标识相关：pid，ppid等等
文件相关：进程需要记录打开的文件信息，于是需要文件描述符表
内存相关：内存指针，指向进程的虚拟地址空间（用户空间）信息
优先级相关：进程相对于其他进程的调度优先级
上下文信息相关：CPU的所有寄存器中的值、进程的状态以及堆栈上的内容，当内核需要切换到另一个进程时，需要保存当前进程的所有状态，即保存当前进程的进程上下文，以便再次执行该进程时，能够恢复切换时的状态，继续执行。
状态相关：进程当前的状态，说明该进程处于什么状态
信号相关：进程的信号处理函数，以及记录当前进程是否还有待处理的信号
I/O相关：记录进程与各种I/O设备之间的交互



float

![image-20200629130913247](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200629130913247.png)

![image-20200629130931296](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200629130931296.png)



![image-20200629130947046](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200629130947046.png)

1. double 编码

   ![image-20200629131016685](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200629131016685.png)

   ![image-20200629131050228](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200629131050228.png)

2. ## 进程与线程

   **进程是程序在计算机上的一次执行活动。**当你运行一个程序，你就启动了一个进程。显然，程序只是一组指令的有序集合，它本身没有任何运行的含义，只是一个静态实体。而进程则不同，它是程序在某个数据集上的执行，是一个动态实体。它因创建而产生，因调度而运行，因等待资源或事件而被处于等待状态，因完成任务而被撤消，反映了一个程序在一定的数据集上运行的全部动态过程。

   **进程是操作系统分配资源的单位。**在Windows下，进程又被细化为线程，也就是一个进程下有多个能独立运行的更小的单位。

   **线程(Thread)是进程的一个实体，是CPU调度和分派的基本单位。**线程不能够独立执行，必须依存在应用程序中，由应用程序提供多个线程执行控制

   

   

3. ## PCB一般包括：

   1.程序ID（PID、进程句柄）：它是唯一的，一个进程都必须对应一个PID。PID一般是整形数字

   2.特征信息：一般分系统进程、用户进程、或者内核进程等

   3.进程状态：运行、就绪、阻塞，表示进程现的运行情况

   4.优先级：表示获得CPU控制权的优先级大小

   5.通信信息：进程之间的通信关系的反映，由于操作系统会提供通信信道

   6.现场保护区：保护阻塞的进程用

   7.资源需求、分配控制信息

   8.进程实体信息，指明程序路径和名称，进程数据在物理内存还是在交换分区（分页）中

   9.其他信息：工作单位，工作区，文件信息等 [1]

4. ## 逻辑地址、相对地址与物理地址

   

 逻辑地址：与当前数据在内存中的物理分配地址无关的访问地址，在执行对内存的访问之前必须转化为物理地址。

相对地址：特殊的逻辑地址，相对于某些已知点的存储单元。

物理地址：数据在主存中的实际位置

5. ## 分段分页

   **分页**逻辑地址到物理地址的映射过程，分页系统中，允许将进程的每一页离散地存储在内存的任一物理块中，为了能在内存中找到每个页面对应的物理块，系统为每个进程建立一张页表，用于记录进程逻辑页面与内存物理页面之间的对应关系。页表的作用是实现从页号到物理块号的地址映射，地址空间有多少页，该页表里就登记多少行，且按逻辑页的顺序排列，形如：

   

   分段存储管理

   1.基本思想

   页面是主存物理空间中划分出来的等长的固定区域。分页方式的优点是页长固定，因而便于构造页表、易于管理，且不存在外碎片。但分页方式的缺点是页长与程序的逻辑大小不相关。例如，某个时刻一个子程序可能有一部分在主存中，另一部分则在辅存中。这不利于编程时的独立性，并给换入换出处理、存储保护和存储共享等操作造成麻烦。

   另一种划分可寻址的存储空间的方法称为分段。段是按照程序的自然分界划分的长度可以动态改变的区域。通常，程序员把子程序、操作数和常数等不同类型的数据划分到不同的段中，并且每个程序可以有多个相同类型的段。

    段表本身也是一个段，可以存在辅存中，但一般是驻留在主存中。将用户程序地址空间分成若干个大小不等的段，每段可以定义一组相对完整的逻辑信息。存储分配时，以段为单位，段与段在内存中可以不相邻接，也实现了离散分配。

   2. 分段地址结构

   作业的地址空间被划分为若干个段，每个段定义了一组逻辑信息。例程序段、数据段等。每个段都从0开始编址，并采用一段连续的地址空间。段的长度由相应的逻辑信息组的长度决定，因而各段长度不等。整个作业的地址空间是二维的。

   在段式虚拟存储系统中，虚拟地址由段号和段内地址组成，虚拟地址到实存地址的变换通过段表来实现。每个程序设置一个段表，段表的每一个表项对应一个段，每个表项至少包括三个字段：有效位（指明该段是否已经调入主存）、段起址(该段在实存中的首地址)和段长（记录该段的实际长度）。

   ***分段存储方式的优缺点***

   分页对程序员而言是不可见的，而分段通常对程序员而言是可见的，因而分段为组织程序和数据提供了方便。与页式虚拟存储器相比，段式虚拟存储器有许多优点：

   (1)    段的逻辑独立性使其易于编译、管理、修改和保护，也便于多道程序共享。

   (2)    段长可以根据需要动态改变，允许自由调度，以便有效利用主存空间。

   (3)    方便编程，分段共享，分段保护，动态链接，动态增长

    因为段的长度不固定，段式虚拟存储器也有一些缺点：

   (1)    主存空间分配比较麻烦。

   (2)    容易在段间留下许多碎片，造成存储空间利用率降低。

   (3)    由于段长不一定是2的整数次幂，因而不能简单地像分页方式那样用虚拟地址和实存地址的最低若干二进制位作为段内地址，并与段号进行直接拼接，必须用加法操作通过段起址与段内地址的求和运算得到物理地址。因此，段式存储管理比页式存储管理方式需要更多的硬件支持。






   段页式结合 对于每一个虚拟地址，处理器使用段号检索进程段表来寻找该段的页表；页号用来检索页表并查找到帧号。

   段页式存储管理的基本思想

   段页式存储组织是分段式和分页式结合的存储组织方法，这样可充分利用分段管理和分页管理的优点。

   　  (1) 用分段方法来分配和管理虚拟存储器。程序的地址空间按逻辑单位分成基本独立的段，而每一段有自己的段名，再把每段分成固定大小的若干页。

   (2) 用分页方法来分配和管理实存。即把整个主存分成与上述页大小相等的存储块，可装入作业的任何一页。程序对内存的调入或调出是按页进行的。但它又可按段实现共享和保护。

   

   

   段页式存储管理的优缺点

    优点

   (1) 它提供了大量的虚拟存储空间。

   (2) 能有效地利用主存，为组织多道程序运行提供了方便。

   缺点：

   (1) 增加了硬件成本、系统的复杂性和管理上的开消。

   (2) 存在着系统发生抖动的危险。

   (3) 存在着内碎片。

   (4) 还有各种表格要占用主存空间。

   　段页式存储管理技术对当前的大、中型计算机系统来说，算是最通用、最灵活的一种方案。
   ————————————————

   原文链接：https://blog.csdn.net/zephyr_be_brave/java/article/details/8944967

   

5. ### 进程间通信IPC (InterProcess Communication)

   **1.** **管道/** 匿名管道(pipe) 只能用于父子进程或者兄弟进程之间(具有亲缘关系的进程

6. ### 虚拟内存

   虚拟内存的目的是为了让物理内存扩充成更大的逻辑内存，从而让程序获得更多的可用内存，为了更好的管理内存，操作系统将内存抽象成地址空间。每个程序拥有自己的地址空间，这个地址空间被分割成多个块，每一块称为一页。这些页被映射到物理内存，但不需要映射到连续的物理内存，也不需要所有页都必须在物理内存中。当程序引用到不在物理内存中的页时，由硬件执行必要的映射，将缺失的部分装入物理内存并重新执行失败的指令。

   内存管理单元（MMU）管理着地址空间和物理内存的转换，其中的页表（Page table）存储着页（程序地址空间）和页框（物理内存空间）的映射表。

   一个虚拟地址分成两个部分，一部分存储页面号，一部分存储偏移量。

7. ### 死锁必要条件和处理方法

   - 互斥：每个资源要么已经分配给了一个进程，要么就是可用的。
   - 占有和等待：已经得到了某个资源的进程可以再请求新的资源。
   - 不可抢占：已经分配给一个进程的资源不能强制性地被抢占，它只能被占有它的进程显式地释放。
   - 环路等待：有两个或者两个以上的进程组成一条环路，该环路中的每个进程都在等待下一个进程所占有的资源。

8. ### 硬盘有什么组成

   （磁头、磁道、扇区、柱面）

   

9. ## linux启动方式

   
   原文链接：https://blog.csdn.net/a624731186/java/article/details/22691049
   
   1. BIOS自检,BIOS的第一个步骤是加电自检,BIOS的第二个步骤是检测本地设备。,侦测电脑周边配套设备是否工作正  常,如cpu的类型,速度,缓存等;主板类型,内存的速度,容量,硬盘的大小,类型和工作模式,风扇速度等,主要是为了检查这些设备在开机的时候是否能正常的工作.
   2. 载入启动程序
         主板的BIOS会读取硬盘的主引导记录(MBR),MBR中存放的是一段很小的程序,他的功能是从硬盘读取操作系统核心文件并运行,因为这个小程序太小了,因此通常这个小程序不具备直接引导系统内核的能力,他先去引导另一个稍微大一点的小程序,再由这个大一点的小程序去引导系统内核.
   3. 加载内核
        LINUX内核一般是压缩保存的，因此，它首先要进行自身的解压缩。内核映象前面的一些代码完成解压缩。解压后将其放入高端内存中，如果有初始RAM磁盘映像，就会将它移动到内存中，并标明以后使用，然后内核映象前面的代码调用内核，并开始启动内核引导的过程 
   4. 启动init服务
          这里的Init程序，一般放在/sbin下,(到这里会出现很多不同的启动方式，主要有：SystemV,BSD,upstart和systemd).
           这里主要说SystemV,init进程是所有进程的起点，也是Linux内核启动后的第一个动作，所以这个程序的PID是永远是1，init进程是所有进程的发起者和控制者
   5. 执行run level对应目录中的脚本，例如：等级为5，则执行/etc/rc.d/rc5.d下面的脚本
        执行时按脚本的文件名 串行执行，这样就造成开机比较慢。目前systemd是以并行执行（号称最快2秒开机）
   
10. ## 用户态到内核态切换可以通过三种方式：

    1. 系统调用，这个上面已经讲解过了。其实系统调用本身就是中断，但是软件中断，跟硬中断不同。
    2. 异常：如果当前进程运行在用户态，如果这个时候发生了异常事件，就会触发切换。例如：缺页异常。
    3. 外设中断：当外设完成用户的请求时，会向CPU发送中断信号。

11. ## c/c++ struct的大小

    3条原则:(在没有#pragma pack宏的情况下)

    １:数据成员对齐规则：结构(struct)(或联合(union))的数据成员，第一个数据成员放在offset为0的地方，以后每个数据成员存储的起始位置要从该成员大小的整数倍开始(比如int在３２位机为４字节,则要从４的整数倍地址开始存储。

    ２:结构体作为成员:如果一个结构里有某些结构体成员,则结构体成员要从其内部最大元素大小的整数倍地址开始存储.(struct a里存有struct b,b里有char,int ,double等元素,那b应该从8的整数倍开始存储.)

    ３:收尾工作:结构体的总大小,也就是sizeof的结果,.必须是其内部最大成员的整数倍.不足的要补齐

12. ## Linux下 文件描述符（fd）与 文件指针（FILE*）

    ![image-20200719165144942](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200719165144942.png)

13. ### 磁盘调度算法  （SCAN：电梯调度算法）

   https://blog.csdn.net/Jaster_wisdom/article/details/52345674



![image-20200719192434420](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200719192434420.png)

# 网络

## 概述

**物理层** OSI 模型的最低层或第一层，该层包括物理连网媒介，如电缆连线连接器。物理层的协议产生并检测电压以便发送和接收携带数据的信号。在你的[桌面](http://baike.baidu.com/view/79807.htm)PC 上插入[网络接口卡](http://baike.baidu.com/view/547393.htm)，你就建立了计算机连网的基础。换言之，你提供了一个物理层。尽管物理层不提供纠错服务，但它能够设定[数据传输速率](http://baike.baidu.com/view/434019.htm)并监测数据出错率。网络物理问题，如电线断开，将影响物理层。用户要传递信息就要利用一些物理媒体，如双绞线、同轴电缆等，但具体的物理媒体并不在OSI的7层之内，有人把物理媒体当做第0层，物理层的任务就是为它的上一层提供一个物理连接，以及它们的机械、电气、功能和过程特性。如规定使用电缆和接头的类型、传送信号的电压等。在这一层，数据还没有被组织，仅作为原始的位流或电气电压处理，单位是bit比特。 

**数据链路层** OSI模型的第二层，它控制[网络层](http://baike.baidu.com/view/239600.htm)与物理层之间的通信。它的主要功能是如何在不可靠的物理线路上进行数据的可靠传递。为了保证传输，从网络层接收到的数据被分割成特定的可被物理层传输的帧。帧是用来移动数据的结构包，它不仅包括原始数据，还包括发送方和接收方的物理地址以及检错和控制信息。其中的地址确定了帧将发送到何处，而纠错和控制信息则确保帧无差错到达。 如果在传送数据时，接收点检测到所传数据中有差错，就要通知发送方重发这一帧。数据链路层的功能独立于网络和它的节点和所采用的物理层类型，它也不关心是否正在运行 Word 、Excel 或使用Internet。有一些连接设备，如[交换机](http://baike.baidu.com/view/1077.htm)，由于它们要对帧解码并使用帧信息将数据发送到正确的接收方，所以它们是工作在数据链路层的。在物理层提供比特流服务的基础上，建立相邻结点之间的数据链路，通过差错控制提供数据帧（Frame）在信道上无差错的传输，并进行各电路上的动作系列。数据链路层在不可靠的物理介质上提供可靠的传输。该层的作用包括：物理地址寻址、数据的[成帧](http://baike.baidu.com/view/3871125.htm)、流量控制、数据的检错、重发等。数据链路层协议的代表包括：[SDLC](http://baike.baidu.com/view/206610.htm)、[HDLC](http://baike.baidu.com/view/89174.htm)、[PPP](http://baike.baidu.com/view/30514.htm)、[STP](http://baike.baidu.com/view/28816.htm)、[帧中继](http://baike.baidu.com/view/21773.htm)等。

 **网络层** OSI 模型的第三层，其主要功能是决定如何将数据从发送方路由到接收方。网络层通过综合考虑发送优先权、[网络拥塞](http://baike.baidu.com/view/1452069.htm)程度、服务质量以及可选路由的花费来决定从一个网络中节点A 到另一个网络中节点B 的最佳路径。由于网络层处理，并智能指导数据传送，[路由器](http://baike.baidu.com/view/1360.htm)连接网络各段，所以路由器属于网络层。在网络中，“路由”是基于编址方案、使用模式以及可达性来指引数据的发送。网络层负责在源机器和目标机器之间建立它们所使用的路由。这一层本身没有任何错误检测和修正机制，因此，网络层必须依赖于端端之间的由DLL提供的可靠传输服务。网络层用于本地LAN网段之上的[计算机系统](http://baike.baidu.com/view/1130583.htm)建立通信，它之所以可以这样做，是因为它有自己的路由地址结构，这种结构与第二层机器地址是分开的、独立的。这种协议称为路由或可路由协议。路由协议包括IP、Novell公司的IPX以及Apple Talk协议。网络层是可选的，它只用于当两个计算机系统处于不同的由路由器分割开的网段这种情况，或者当通信应用要求某种网络层或传输层提供的服务、特性或者能力时。例如，当两台主机处于同一个LAN网段的直接相连这种情况，它们之间的通信只使用LAN的通信机制就可以了(即OSI 参考模型的一二层)。

**传输层** 传输协议同时进行流量控制或是基于接收方可接收数据的快慢程度规定适当的发送速率。除此之外，传输层按照网络能处理的最大尺寸将较长的数据包进行强制分割。例如，以太网无法接收大于1500字节的数据包。发送方节点的传输层将[数据分割](http://baike.baidu.com/view/4466818.htm)成较小的数据片，同时对每一数据片安排一序列号，以便数据到达接收方节点的传输层时，能以正确的顺序重组。该过程即被称为排序。工作在传输层的一种服务是 TCP/IP 协议套中的TCP （Transport Control Protocol，[传输控制协议](http://baike.baidu.com/view/544903.htm)），UDP (User Packet Data，用户数据报协议)，另一项传输层服务是IPX/SPX协议集的SPX（序列包交换）。

1. ### 物理层和链路层

   物理层的线路有传输介质与通信设备组成，比特流在传输介质上传输时肯定会存在误差的。这样就引入了数据链路层在物理层之上，采用差错检测、差错控制和流量控制等方法，向网络层提供高质量的数据传输服务。对于网络层，由于链路层的存在，而不需要关心物理层具体采用了那种传输介质和通信设备。

   ## **那链路层的功能有哪些呢？需要完成哪些事情呢？**

   1. 链路管理，帧同步
   2. 流量控制，差错控制
   3. 数据和控制信息分开
   4. 透明传输和**寻址**

   ## 网络层和传输层的作用

   网络层只把分组发送到目的主机，但是真正通信的并不是主机而是主机中的进程。传输层提供了进程间的逻辑通信，传输层向高层用户屏蔽了下面网络层的核心细节，使应用程序看起来像是在两个传输层实体之间有一条端到端的逻辑通信信道。

   

   ## PPP

   ![image-20200704172812352](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200704172812352.png)

   ### 透明传输的基本概念：

   数据透明传输就是用户不受协议中的任何限制，可随机的传输任意比特编码的信息
   用户可以完全不必知道协议中所规定的结束段的比特编码或者其他的控制字符，因而不受限制的进行传输。
   数据透明传输技术：

   转义字符填充法
   零比特填充法
   采用特殊的信号与编码法：IEEE802.3(由于使用CSMA/CD协议，没有结束字符段；IEEE802.4（令牌总线，在起始定界符SD/结束定界符ED这两个字段被使用模拟编码，而不是0和1）；IEEE802.5（令牌环，违例的曼切斯特码）
   确定长度法，固定数据段长度法：各控制字段的长度固定，数据段长度也是固定的，那么在帧格式中就不必设结束符，也不必设数据长度字段。
   ————————————————
   原文链接：https://blog.csdn.net/xie294777315/java/article/details/24176611

   ## MAC 地址

   MAC 地址是链路层地址，长度为 6 字节（48 位），用于唯一标识网络适配器（网卡）。一台主机拥有多少个网络适配器就有多少个 MAC 地址。例如笔记本电脑普遍存在无线网络适配器和有线网络适配器，因此就有两个 MAC 地址。

   

   ### 链路层三个基本问题

   1. 将网络层传下来的分组添加首部和尾部，用于标记帧的开始和结束。

   2. 透明表示一个实际存在的事物看起来好像不存在一样。帧使用首部和尾部进行定界，如果帧的数据部分含有和首部尾部相同的内容，那么帧的开始和结束位置就会被错误的判定。需要在数据部分出现首部尾部相同的内容前面插入转义字符。如果数据部分出现转义字符，那么就在转义字符前面再加个转义字符。在接收端进行处理之后可以还原出原始数据。这个过程透明传输的内容是转义字符，用户察觉不到转义字符的存在。

   ![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/e738a3d2-f42e-4755-ae13-ca23497e7a97.png)

   

   3. 差错校验目前数据链路层广泛使用了循环冗余检验（CRC）来检查比特差错。

   ## ip数据报

   ![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/85c05fb1-5546-4c50-9221-21f231cdc8c5.jpg)

   

   - **版本** : 有 4（IPv4）和 6（IPv6）两个值；
   - **首部长度** : 占 4 位，因此最大值为 15。值为 1 表示的是 1 个 32 位字的长度，也就是 4 字节。因为固定部分长度为 20 字节，因此该值最小为 5。如果可选字段的长度不是 4 字节的整数倍，就用尾部的填充部分来填充。
   - **区分服务** : 用来获得更好的服务，一般情况下不使用。
   - **总长度** : 包括首部长度和数据部分长度。
   - **生存时间** ：TTL，它的存在是为了防止无法交付的数据报在互联网中不断兜圈子。以路由器跳数为单位，当 TTL 为 0 时就丢弃数据报。
   - **协议** ：指出携带的数据应该上交给哪个协议进行处理，例如 ICMP、TCP、UDP 等。
   - **首部检验和** ：因为数据报每经过一个路由器，都要重新计算检验和，因此检验和不包含数据部分可以减少计算的工作量。
   - **标识** : 在数据报长度过长从而发生分片的情况下，相同数据报的不同分片具有相同的标识符。
   - **片偏移** : 和标识符一起，用于发生分片的情况。片偏移的单位为 8 字节。
   - ![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/23ba890e-e11c-45e2-a20c-64d217f83430.png)

   ### ip 地址分类

   IP 地址 ::= {< 网络号 >, < 主机号 >}

   ![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/cbf50eb8-22b4-4528-a2e7-d187143d57f7.png)

10. ### 子网

   通过在主机号字段中拿一部分作为子网号，把两级 IP 地址划分为三级 IP 地址。

   IP 地址 ::= {< 网络号 >, < 子网号 >, < 主机号 >}

   要使用子网，必须配置子网掩码。一个 B 类地址的默认子网掩码为 255.255.0.0，如果 B 类地址的子网占两个比特，那么子网掩码为 11111111 11111111 11000000 00000000，也就是 255.255.192.0。

   注意，外部网络看不到子网的存在。

11. ### ICMP

   ICMP 是为了更有效地转发 IP 数据报和提高交付成功的机会。它封装在 IP 数据报中，但是不属于高层协议。

   ![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/e3124763-f75e-46c3-ba82-341e6c98d862.jpg)

   

   ICMP 报文分为差错报告报文和询问报文。

   ![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/aa29cc88-7256-4399-8c7f-3cf4a6489559.png)

## ARP(Address Resolution Protocol，地址解析协议) 

在局域网中获取目的 IP 地址对应的 MAC 地址：

1. 源主机会向当前局域网中发送 ARP 请求，目标的 MAC 地址是 `FF-FF-FF-FF-FF-FF`，这表示当前请求是一个广播请求，局域网内的所有设备都会收到该请求；
2. 接收到 ARP 请求的主机都会检查目的 IP 和自己的 IP 地址是否一致；
   1. 如果 IP 地址不一致，主机会忽略当前的 ARP 请求；
   2. 如果 IP 地址一致，主机会直接向源主机发送 ARP 响应；
3. 源主机在接收到 ARP 的响应之后，会更新本地的缓存表并继续向目的主机发送数据；

   ## [1. Ping]

   Ping 是 ICMP 的一个重要应用，主要用来测试两台主机之间的连通性。

   Ping 的原理是通过向目的主机发送 ICMP Echo 请求报文，目的主机收到之后会发送 Echo 回答报文。Ping 会根据时间和成功响应的次数估算出数据包往返时间以及丢包率。

ping+一个公网地址，网络层及以下会发生什么

1. 判断是不是外网；

2. arp 找路由器的mac地址，ping -> icmp request, mac destination  

3. 当路由器收到主机A发过来的ICMP报文，发现自己的目的地址是其本身MAC地址，根据目的的IP2.1.1.1，查路由表，发现2.1.1.1/24的路由表项，得到一个出口指针，去掉原来的MAC头部，加上自己的MAC地址向主机C转发。(如果网关也没有主机C的MAC地址，还是要向前面一个步骤一样，ARP广播一下即可相互学到。路由器2端口能学到主机D的MAC地址，主机D也能学到路由器2端口的MAC地址。)报文格式如下：

   ![img](https://img-blog.csdn.net/20160311232426001)

4. 最后，在主机C已学到路由器2端口MAC地址，路由器2端口转发给路由器1端口，路由1端口学到主机A的MAC地址的情况下，他们就不需要再做ARP解析，就将ICMP的回显请求回复过来。报文格式大致如下: 

5. ![image-20200722174438310](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200722174438310.png)



三层转发
 三层转发是什么？简单来说就是通过路由器的在不同网段间的转发。工作在TCP/IP网络模型的第三层。

 三层转发可以很复杂，也可以很简单。

其实打开网页，进入百度的过程就是一个三层转发，因为这个过程肯定使用了路由器，并且肯定是进入了不同的网段。但是这个过程十分复杂，需要涉及多个路由器（少则十几，多则几十。），并且涉及到了路由器的另一大功能nat，即网络地址转换。

 当然也有简单的三层转发，可以使用能够配置VLAN的路由器，连接两台设备。这样数据报通过路由器，并且还设计到不同的网段（子网）。VALN和子网的关系另外再介绍

PC1（192.168.0.2）----------------(eth0（192.168.0.1） 路由器 eth1（192.168.1.1）)------------------PC2（192.168.1.2）

则过程

1、  PC1想要ping PC2。

2、  PC1上层已知PC2的IP地址，并将数据报配置完毕。但是需要查询路由表，确认路径

3、  通过查询PC1的路由表，通过路由表选路原则，从众多的路由条目中确认，必须从接口（192.168.0.2）发出，发送到网关（192.168.0.1）。

4、  然后必须知道网关的MAC地址（路由器的IP地址）。从ARP缓存中去查询，没有就发送ARP广播

5、  接口eht0获得广播报文，并将报文上传给路由器的CPU。路由器判断是在询问它的MAC地址，然后发送ARP应答报文，并将PC1的IP地址和MAC地址学习到路由器的ARP缓存中。（这里涉及到了另一个问题，路由器有ARP缓存么？嘿嘿，看下图。）

路由器会根据cef和arp的cache维护一个所谓的adjacency table 来提高转发时候的速度。

6、  有了路由器的MAC地址，然后PC1就可以向路由器发送ICMP回显请求报文了（目的IP地址还是PC2的IP地址。三层转发不会动IP数据报里的东西，只是改变以太网首部）

7、  路由器拿到这个数据报，解析后发现，通过查询FIB表（可以看做路由表的升级），发现原来在eth1口，所连的1网段内。然后确认了路径192.168.1.1 -- 192.168.1.2。

8、  有了路径，就去路由器的ARP表（可能是FIB表）中查询路径终端192.168.1.2对应的MAC地址，没有就发送ARP请求广播。这个过程与PC1的广播一样。

9、  获得了PC1的MAC地址，并学习到ARP缓存中。然后将目的MAC地址改为PC2的MAC地址，将源MAC地址改为路由器的MAC地址。

10、然后就发送给了PC2。PC2再发送ICMP应答报文。去查询路由表。。。。。。所有过程与ICMP请求到PC2的过程一样

路由器的几张表一直没搞太清楚。所以可能会有一些问题。附带两张图

 ![image-20200722175458938](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200722175458938.png)



上图为路由器中各表的关系图

![image-20200722175509907](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200722175509907.png)

上图为路由器内部转发图




   ## [2. Traceroute]

   Traceroute 是 ICMP 的另一个应用，用来跟踪一个分组从源点到终点的路径。

   Traceroute 发送的 IP 数据报封装的是无法交付的 UDP 用户数据报，并由目的主机发送终点不可达差错报告报文。

   - 源主机向目的主机发送一连串的 IP 数据报。第一个数据报 P1 的生存时间 TTL 设置为 1，当 P1 到达路径上的第一个路由器 R1 时，R1 收下它并把 TTL 减 1，此时 TTL 等于 0，R1 就把 P1 丢弃，并向源主机发送一个 ICMP 时间超过差错报告报文；

   - 源主机接着发送第二个数据报 P2，并把 TTL 设置为 2。P2 先到达 R1，R1 收下后把 TTL 减 1 再转发给 R2，R2 收下后也把 TTL 减 1，由于此时 TTL 等于 0，R2 就丢弃 P2，并向源主机发送一个 ICMP 时间超过差错报文。

   - 不断执行这样的步骤，直到最后一个数据报刚刚到达目的主机，主机不转发数据报，也不把 TTL 值减 1。但是因为数据报封装的是无法交付的 UDP，因此目的主机要向源主机发送 ICMP 终点不可达差错报告报文。

   - 之后源主机知道了到达目的主机所经过的路由器 IP 地址以及到达每个路由器的往返时间。

     

   

## Tcp UDP 区别

   * 用户数据报协议 UDP（User Datagram Protocol）是无连接的，尽最大可能交付，没有拥塞控制，面向报文（对于应用程序传下来的报文不合并也不拆分，只是添加 UDP 首部），支持一对一、一对多、多对一和多对多的交互通信。

   * 传输控制协议 TCP（Transmission Control Protocol）是面向连接的，提供可靠交付，有流量控制，拥塞控制，提供全双工通信，面向字节流（把应用层传下来的报文看成字节流，把字节流组织成大小不等的数据块），每一条 TCP 连接只能是点对点的（一对一）。

## UDP 首部格式

![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/d4c3a4a1-0846-46ec-9cc3-eaddfca71254.jpg)



首部字段只有 8 个字节，包括源端口、目的端口、长度、检验和。12 字节的伪首部是为了计算检验和临时添加的。

## TCP 首部格式

![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/55dc4e84-573d-4c13-a765-52ed1dd251f9.png)



- **序号** ：用于对字节流进行编号，例如序号为 301，表示第一个字节的编号为 301，如果携带的数据长度为 100 字节，那么下一个报文段的序号应为 401。

- **确认号** ：期望收到的下一个报文段的序号。例如 B 正确收到 A 发送来的一个报文段，序号为 501，携带的数据长度为 200 字节，因此 B 期望下一个报文段的序号为 701，B 发送给 A 的确认报文段中确认号就为 701。

- **数据偏移** ：指的是数据部分距离报文段起始处的偏移量，实际上指的是首部的长度。

- **确认 ACK** ：当 ACK=1 时确认号字段有效，否则无效。TCP 规定，在连接建立后所有传送的报文段都必须把 ACK 置 1。

- **同步 SYN** ：在连接建立时用来同步序号。当 SYN=1，ACK=0 时表示这是一个连接请求报文段。若对方同意建立连接，则响应报文中 SYN=1，ACK=1。

- **终止 FIN** ：用来释放一个连接，当 FIN=1 时，表示此报文段的发送方的数据已发送完毕，并要求释放连接。

- **窗口** ：窗口值作为接收方让发送方设置其发送窗口的依据。之所以要有这个限制，是因为接收方的数据缓存空间是有限的。

  

## TCP层的分段和IP层的分片之间的关系 & MTU和MSS之间的关系

分组可以发生在运输层和网络层，运输层中的TCP会分段，网络层中的IP会分片。IP层的分片更多的是为运输层的UDP服务的，由于TCP自己会避免IP的分片，所以使用TCP传输在IP层都不会发生分片的现象。

MTU（Maximum Transmission Unit，MTU），最大传输单元

（1）以太网和802.3对数据帧的长度都有一个限制，其最大 值分别是1500和1492个字节。链路层的这个特性称作MTU。不同类型的网络大多数都有一个上限。如果IP层有一个数据要传，且数据的长度比链路层的 MTU还大，那么IP层就要进行分片（fragmentation），把数据报分成若干片，这样每一个分片都小于MTU。

（2）把一份IP数据报进行分片以后，由到达目的端的IP层来进行重新组装，其目的是使分片和重新组装过程对运输层（TCP/UDP）是透明的。由于每一分片都是一个独立的包，当这些数据报的片到达目的端时有可能会失序，但是在IP首部中有足够的信息让接收端能正确组装这些数据报片。

（3）尽管IP分片过程看起来透明的，但有一点让人不想使用它：即使只丢失一片数据也要重新传整个数据报。why？因为IP层本身没有超时重传机制------由更高层（比如TCP）来负责超时和重传。当来自TCP报文段的某一片丢失后，TCP在超时后会重发整个TCP报文段，该报文段对应于一份IP数据报（而不是一个分片），没有办法只重传数据报中的一个数据分片。 

（4）使用UDP很容易导致IP分片，TCP试图避免IP分片。 那么TCP是如何试图避免IP分片的呢？其实说白了，采用TCP协议进行数据传输是不会造成IP分片的，因为一旦TCP数据过大，超过了MSS，则在传输层会对TCP包进行分段（如何分，见下文！），自然到了IP层的数据报肯定不会超过MTU，当然也就不用分片了。而对于UDP数据报，如果UDP组成的 IP数据报长度超过了1500，那么IP数据报显然就要进行分片，因为UDP不能像TCP一样自己进行分段。总结：UDP不会分段，就由我IP来分。TCP会分段，当然也就不用我IP来分了！

MSS（Maxitum Segment Size）最大分段大小的缩写，是TCP协议里面的一个概念

MSS就是TCP数据包每次能够传输的最大数据分段。为了达到最佳的传输效能TCP协议在建立连接的时候通常要协商双方的MSS值，这个值TCP协议在实现的时候往往用MTU值代替（需要减去IP数据包包头的大小20Bytes和TCP数据段的包头20Bytes）所以往往MSS为1460。通讯双方会根据双方提供的MSS值得最小值确定为这次连接的最大MSS值。





### 三次握手的原因

第三次握手是为了防止失效的连接请求到达服务器，让服务器错误打开连接。

客户端发送的连接请求如果在网络中滞留，那么就会隔很长一段时间才能收到服务器端发回的连接确认。客户端等待一个超时重传时间之后，就会重新请求连接。但是这个滞留的连接请求最后还是会到达服务器，如果不进行三次握手，那么服务器就会打开两个连接。如果有第三次握手，客户端会忽略服务器之后发送的对滞留连接请求的连接确认，不进行第三次握手，因此就不会再次打开连接。

### 为什么A在TIME-WAIT状态必须等待2MSL的时间？

MSL最长报文段寿命Maximum Segment Lifetime，MSL=2

两个理由：

**1**）保证**A**发送的最后一个ACK报文段能够到达B。

**2**）防止“已失效的连接请求报文段”出现在本连接中。

- 1）这个ACK报文段有可能丢失，使得处于LAST-ACK状态的B收不到对已发送的FIN+ACK报文段的确认，B超时重传FIN+ACK报文段，而A能在2MSL时间内收到这个重传的FIN+ACK报文段，接着A重传一次确认，重新启动2MSL计时器，最后A和B都进入到CLOSED状态，**若A在TIME-WAIT状态不等待一段时间，而是发送完ACK报文段后立即释放连接，则无法收到B重传的FIN+ACK报文段，所以不会再发送一次确认报文段，则B无法正常进入到CLOSED状态。**
- 2）A在发送完最后一个ACK报文段后，再经过2MSL，就可以使本连接持续的时间内所产生的所有报文段都从网络中消失，使下一个新的连接中不会出现这种旧的连接请求报文段。

15. ## **为什么A还要发送一次确认呢？可以二次握手吗？**

    为了**实现可靠数据传输**， TCP 协议的通信双方， 都**必须维护一个序列号， 以标识发送出去的数据包中， 哪些是已经被对方收到**的。 三次握手的过程即是通信双方相互告知序列号起始值， 并确认对方已经收到了序列号起始值的必经步骤 **如果只是两次握手， 至多只有连接发起方的起始序列号能被确认， 另一方选择的序列号则得不到确认**

16. ## **Server端易受到SYN攻击？**

    服务器端的资源分配是在二次握手时分配的，而客户端的资源是在完成三次握手时分配的，所以服务器容易受到SYN洪泛攻击，SYN攻击就是Client在短时间内伪造大量不存在的IP地址，并向Server不断地发送SYN包，Server则回复确认包，并等待Client确认，由于源地址不存在，因此Server需要不断重发直至超时，这些伪造的SYN包将长时间占用未连接队列，导致正常的SYN请求因为队列满而被丢弃，从而引起网络拥塞甚至系统瘫痪。

    防范SYN攻击措施：降低主机的等待时间使主机尽快的释放半连接的占用，短时间受到某IP的重复SYN则丢弃后续请求。

17. ## **TIME_WAIT**

   客户端接收到服务器端的 FIN 报文后进入此状态，此时并不是直接进入 CLOSED 状态，还需要等待一个时间计时器设置的时间 2MSL。这么做有两个理由：

   - 确保最后一个确认报文能够到达。如果 B 没收到 A 发送来的确认报文，那么就会重新发送连接释放请求报文，A 等待一段时间就是为了处理这种情况的发生。
   - 等待一段时间是为了让本连接持续时间内所产生的所有报文都从网络中消失，使得下一个新的连接不会出现旧的连接请求报文。

     **Maximum Segment Lifetime** 报文最大生存时间，它是任何报文在网络上存在的最长时间，超过这个时间报文将被丢弃

## 大量Time—wait 怎么解决

解决思路： 让服务器能够快速回收和重用那些TIME_WAIT的资源。

## **大量CLOSE_WAIT\** 怎么解决

unix 网络编程书里面写“TCP FIN sent by kernel when client is killed or crashed”当client被kill的时候，内核会发送fin包给server。这样服务器这边进入close wait的状态，若epoll注册了HUP的事件，把连接关闭close wait变为close；若没有处理，服务器这里就有一个close wait的状态，占用了fd。

过多的close_wait可能是什么原因：

1. 程序问题：说的具体一点，服务器端的代码，没有写 close 函数关闭 socket 连接，也就不会发出 FIN 报文段；或者出现死循环，服务器端的代码永远执行不到 close。
2. 客户机响应太慢或者 timeout 设置过小

解决思路： 检测出对方已经关闭的socket，然后关闭它。
1.代码需要判断socket，一旦read返回0，断开连接，read返回负，检查一下errno，如果不是AGAIN（表示现在没有数据稍后重新读取），也断开连接。
2.给每一个socket设置一个时间戳last_update，每接收或者是发送成功数据，就用当前时间更新这个时间戳。定期检查所有的时间戳，如果时间戳与当前时间差值超过一定的阈值，就关闭这个socket。
3.使用一个Heart-Beat线程，定期向socket发送指定格式的心跳数据包，如果接收到对方的RST报文，说明对方已经关闭了socket，那么我们也关闭这个socket。
4.设置SO_KEEPALIVE选项，并修改内核参数
————————————————
close_wait过多的解决方案
代码层面做到第一：使用完socket调用close方法；第二：socket读控制，当读取的长度为0时（读到结尾），立即close；第三：如果read返回-1，出现错误，检查error返回码，有三种情况：INTR（被中断，可以继续读取），WOULDBLOCK（表示当前socket_fd文件描述符是非阻塞的，但是现在被阻塞了），AGAIN（表示现在没有数据稍后重新读取）。如果不是AGAIN，立即close
可以设置TCP的连接时长keep_alive_time还有tcp监控连接的频率以及连接没有活动多长时间被迫断开连接

## 为什么 TCP 协议有粘包问题

- TCP 协议是面向字节流的协议，它可能会组合或者拆分应用层协议的数据；

- 应用层协议的没有定义消息的边界导致数据的接收方无法拼接数据；

  既然 TCP 协议是基于字节流的，这其实就意味着应用层协议要自己划分消息的边界。如果我们能在应用层协议中定义消息的边界，那么无论 TCP 协议如何对应用层协议的数据包进程拆分和重组，接收方都能根据协议的规则恢复对应的消息。在应用层协议中，最常见的两种解决方案就是基于长度或者基于终结符（Delimiter）。
  
  

## 三次握手

用于保证可靠性和流控制机制的信息，包括 Socket、序列号以及窗口大小叫做连接。

![what-is-tcp-connection](https://img.draveness.me/what-is-tcp-connection.png)

所以，建立 TCP 连接就是通信的双方需要对上述的三种信息达成共识，连接中的一对 Socket 是由互联网地址标志符和端口组成的，窗口大小主要用来做流控制，最后的序列号是用来追踪通信发起方发送的数据包序号，接收方可以通过序列号向发送方确认某个数据包的成功接收。

初始序列号并建立 TCP 连接：

- 通过三次握手才能阻止重复历史连接的初始化；
- 通过三次握手才能对通信双方的初始序列号进行初始化；

## 拥塞,TCP 四个算法来进行拥塞控制：慢开始、拥塞避免、快重传、快恢复。

##  TCP/IP 协议在传输数据时都需要对数据进行拆分

，但是它们做出拆分数据的设计基于不同的上下文，也有着不同的目的，我们在这里总结一下两个网络协议做出类似决定的原因：

- IP 协议拆分数据是因为物理设备的限制，一次能够传输的数据由路径上 MTU 最小的设备决定，一旦 IP 协议传输的数据包超过 MTU 的限制就会发生丢包，所以我们需要通过路径 MTU 发现获取传输路径上的 MTU 限制；
- TCP 协议拆分数据是为了保证传输的可靠性和顺序，作为可靠的传输协议，为了保证数据的传输顺序，它需要为每一个数据段增加包含序列号的 TCP 协议头，如果数据段大小超过了 IP 协议的 MTU 限制， 就会带来更多额外的重传和重组开销，影响性能。

## TCP 滑动窗口

窗口是缓存的一部分，用来暂时存放字节流。发送方和接收方各有一个窗口，接收方通过 TCP 报文段中的窗口字段告诉发送方自己的窗口大小，发送方根据这个值和其它信息设置自己的窗口大小。

## **TCP拥塞控制算法**

**1、基于丢包的拥塞控制：Tahoe、Reno、New Reno**

因为Reno等算法是后续算法的基础，这里详细的描述下Reno算法的过程。

（1）慢热启动算法 – Slow Start

-  连接建好的开始先初始化cwnd = 1，表明可以传一个MSS大小的数据。
- 每当收到一个ACK，cwnd++; 呈线性上升。
- 每当过了一个RTT，cwnd = cwnd*2; 呈指数让升。
- 还有一个ssthresh（slow start threshold），是一个上限，当cwnd >= ssthresh时，就会进入“拥塞避免算法”。

（2）拥塞避免算法 – Congestion Avoidance

当cwnd >= ssthresh时，就会进入“拥塞避免算法”。算法如下：

- 收到一个ACK时，cwnd = cwnd + 1/cwnd
- 当每过一个RTT时，cwnd = cwnd + 1

（3）拥塞状态算法 – Fast Retransmit

Tahoe是等RTO超时，FR是在收到3个duplicate ACK时就开启重传，而不用等到RTO超时。拥塞发生时：

- cwnd = cwnd /2
- sshthresh = cwnd

（4）快速恢复 – Fast Recovery

- cwnd = sshthresh  + 3 * MSS （3的意思是确认有3个数据包被收到了）
- 重传Duplicated ACKs指定的数据包
- 如果再收到 duplicated Acks，那么cwnd = cwnd +1
- 如果收到了新的Ack，那么，cwnd = sshthresh ，然后进入拥塞避免算法。
- ![image-20200727200827065](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200727200827065.png)



**二分搜索最佳cwnd：BIC-TCP** BIC-TCP是Linux 2.6.18默认拥塞控制算法，依赖丢包条件触发。BIC-TCP认为TCP拥塞窗口调整的本质就是找到最适合当前网络的一个发送窗口，为了找到这个窗口值，TCP采取的方式是(拥塞避免阶段)每RTT加1，缓慢上升，丢包时下降一半，接着再来慢慢上升。BIC-TCP的提出者们看穿了事情的本质，其实这就是一个搜索的过程，而TCP的搜索方式类似于逐个遍历搜索方法，可以认为这个值是在1和一个比较大的数(large_window)之间，既然在这个区间内需要搜索一个最佳值，那么显然最好的方式就是二分搜索思想。

BIC-TCP就是基于这样一个二分思想的：当出现丢包的时候，说明最佳窗口值应该比这个值小，那么BIC就把此时的cwnd设置为max_win，把乘法减小后的值设置为min_win，然后BIC就开始在这两者之间执行二分思想--每次跳到max_win和min_win的中点。

![img](https://ask.qcloudimg.com/draft/4141261/oc7oqxn0qt.jpg?imageView2/2/w/1620)

图2 BIC-TCP 算法仿真曲线（来源BIC-TCP RFC）

BIC也具备RTT的不公平性。RTT小的连接，窗口调整发生的速度越快，因此可能更快的抢占带宽。



## CUBIC

CUBIC是BIC-TCP的下一代版本。 它通过用三次函数（包含凹和凸部分）代替BIC-TCP的凹凸窗口生长部分，大大简化了BIC-TCP的窗口调整算法。实际上，任何奇数阶多项式函数都具有这种形状。三次函数的选择是偶然的，并且不方便。CUBIC的关键特征是其窗口增长仅取决于两个连续拥塞事件之间的时间。一个拥塞事件是指出现TCP快速恢复的时间。因此，窗口增长与RTT无关。 这个特性允许CUBIC流在同一个瓶颈中竞争，有相同的窗口大小，而不依赖于它们的RTT，从而获得良好的RTT公平性。而且，当RTT较短时，由于窗口增长率是固定的，其增长速度可能比TCP标准慢。 由于TCP标准（例如，TCP-SACK）在短RTT下工作良好，因此该特征增强了协议的TCP友好性。



## **基于精准带宽计算：BBR**



https://blog.csdn.net/yue2388253/article/details/88925203

 BBR通过实时计算带宽和最小RTT来决定发送速率pacing rate和窗口大小cwnd。完全摒弃丢包作为拥塞控制的直接反馈因素。

### 原理

传统的拥塞控制算法是计算cwnd值来规定当前可以发送多少数据，但是并不关注以什么样的速度发送数据。如果简单而粗暴地将窗口大小（send.cwnd、recv.cwnd的最小值）数据全部突发出去，这往往会造成路由器的排队，在深队列的情况下，会测量出rtt剧烈地抖动。bbr在计算cwnd的同时，还计算了一个与之适配的pacing rate，该pacing rate规定cwnd指示的一窗数据的数据包之间，以多大的时间间隔发送出去。

- BtlBW：最大带宽
- RtProp：物理链路延迟
- BDP：管道容量，BDP=BtlBW * RtProp

我们知道，网络工作的最优点是在物理链路延迟状态下，以最大速率传输数据。传统的拥塞控制算法思想是根据数据传输及ACK来确定RTT，但是这个RTT并不是物理链路延时，可能包含了路由器缓存耗时，也可能是拥塞状态下的耗时。传统的带宽计算也是在不断的试探逼近最优发送窗口，并在RTT或者统计周期内计算带宽。这种情况下，RTT并不是真正的物理链路延迟，带宽也有可能是在有路由缓存或丢包状况下计算得到，那么必然得到的不是精准的值。

BBR摒弃了丢包和实时RTT作为拥塞控制因素。引入BDP管道容量来衡量链路传输水平。BBR追求的是在链路最小RTT（物理链路延迟）的状态下，找到最大带宽。

而BBR对于这个问题的解决方式就是取一定时间范围内的RTprop极小值与BtlBw极大值作为估计值。

在连接建立的时候，BBR也采用类似慢启动的方式逐步增加发送速率，然后根据收到的ack计算BDP，当发现BDP不再增长时，就进入拥塞避免阶段（这个过程完全不管有没有丢包）。在慢启动的过程中，由于几乎不会填充中间设备的缓冲区，这过程中的延迟的最小值就是最初估计的最小延迟；而慢启动结束时的最大带宽就是最初的估计的最大延迟。

慢启动结束之后，为了把慢启动过程中可能填充到缓冲区中的数据排空，BBR会进入排空阶段，这期间会降低发送速率，如果缓冲区中有数据，降低发送速率就会使延时下降（缓冲区逐渐被清空），直到延时不再下降。

排空阶段结束后，进入稳定状态，这个阶段会交替探测带宽和延迟。带宽探测阶段是一个正反馈系统：定期尝试增加发包速率，如果收到确认的速率也增加了，就进一步增加发包速率。具体来说，以每8个RTT为周期，在第一个RTT中，尝试以估计带宽的5/4的速度发送数据，第二个RTT中，为了把前一个RTT多发出来的包排空，以估计带宽的3/4的速度发送数据。剩下6个RTT里，使用估计的带宽发包（估计带宽可能在前面的过程中更新）。 这个机制使得BBR在带宽增加时能够迅速提高发送速率，而在带宽下降时则需要一定的时间才能降低到稳定的水平。

除了带宽检测，BBR还会进行最小延时的检测。每过10s，如果最小RTT没有改变（也就是没有发现一个更低的延迟），就进入延迟探测阶段。延迟探测阶段持续的时间仅为 200 毫秒（或一个往返延迟，如果后者更大），这段时间里发送窗口固定为4个包，也就是几乎不发包。这段时间内测得的最小延迟作为新的延迟估计。也就是说，大约有2%的时间BBR会用极低的发包速率来测量延迟。


链接：https://juejin.im/post/5e0894a3f265da33d83e85fe

## DHCP

DHCP (Dynamic Host Configuration Protocol) 提供了即插即用的连网方式，用户不再需要手动配置 IP 地址等信息。DHCP 配置的内容不仅是 IP 地址，还包括子网掩码、网关 IP 地址。

DHCP 工作过程如下：

1. 客户端发送 Discover 报文，该报文的目的地址为 255.255.255.255:67，源地址为 0.0.0.0:68，被放入 UDP 中，该报文被广播到同一个子网的所有主机上。如果客户端和 DHCP 服务器不在同一个子网，就需要使用中继代理。
2. DHCP 服务器收到 Discover 报文之后，发送 Offer 报文给客户端，该报文包含了客户端所需要的信息。因为客户端可能收到多个 DHCP 服务器提供的信息，因此客户端需要进行选择。
3. 如果客户端选择了某个 DHCP 服务器提供的信息，那么就发送 Request 报文给该 DHCP 服务器。
4. DHCP 服务器发送 Ack 报文，表示客户端此时可以使用提供给它的信息。

![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/23219e4c-9fc0-4051-b33a-2bd95bf054ab.jpg)



## 为什么出现了NAT?

IP地址只有32位，最多只有42.9亿个地址，还要去掉保留地址、组播地址，能用的地址只有36亿左右，但是当下有数以万亿的主机，没有这么多IP地址怎么办，后面有了IPv6，但是当下IPv4还是主流，利用IPv4怎么满足这么多主机的IP地址呢？答案就是NAT，NAT技术使公司、机构以及个人产生以及局域网，然后在各个局域网的边界WAN端口使用一个或多个公网的IPv4进行一对多转换

NAT(Network Address Translation，网络地址转换), 用来将内网地址和端口号转换成合法的公网地址和端口号，建立一个会话，与公网主机进行通信。NAT的使用是为了解决公网IP有限及局域网安全性的问题。

实际上NAT分为**基础型NAT**（静态NAT即Static NAT，动态NAT即Dynamic NAT/Pooled NAT）和**NAPT**(Network Address Port Translation)两种，但由于基础型NAT已不常用，我们通常提到的NAT就代指NAPT。**NAPT**是指网络地址转换过程中使用了端口复用技术，即PAT(Port address Translation)。

### NAT工作原理：

- 网络被分为私网和公网两个部分，NAT网关设置在私网到公网的路由出口位置，双向流量必须都要经过NAT网关；
- 网络访问只能先由私网侧发起，公网无法主动访问私网主机；
- NAT网关在两个访问方向上完成两次地址的转换或翻译，出方向做源信息替换，入方向做目的信息替换；
- NAT网关的存在对通信双方是保持透明的；
- NAT网关为了实现双向翻译的功能，需要维护一张关联表，把会话的信息保存下来。


​    

    **NAT使用基于session的转换规则**
    
    TCP/UDP ：私有Host的Ipv4 + port <======> NAT公网的Ipv4 + port
    ICMP ：私有Host的Ipv4 + sessionID <======> NAT公网的Ipv4 + sessionID
    
    链接：https://www.jianshu.com/p/62028875d53e


​    

    动态NAT ：动态NAT是在路由器上配置一个外网IP地址池，当内部有计算机需要和外部通信时，就从地址池里动态的取出一个外网IP，并将他们的对应关系绑定到NAT表中，通信结束后，这个外网IP才被释放，可供其他内部IP地址转换使用，这个DHCP租约IP有相似之处。


​    

    PAT(port address Translation，端口地址转换，也叫端口地址复用)
    这是最常用的NAT技术，也是IPv4能够维持到今天的最重要的原因之一，它提供了一种多对一的方式，对多个内网IP地址，边界路由可以给他们分配一个外网IP，利用这个外网IP的不同端口和外部进行通信。

  







## 防止syn攻击

* syn cookie：syn  cookie技术是服务器在收到syn包时并不马上分配储存连接的数据区，而是根据这个syn包计算出一个cookie，把这个cookie填入tcp的Sequence Number字段发送syn+ack包，等对方回应ack包时检查回复的Acknowledgment  Number字段的合法性，如果合法再分配专门的数据区。

* 缩短SYN Timeout时间，由于SYN  Flood攻击的效果取决于服务器上保持的SYN半连接数，这个值=SYN攻击的频度 x SYN  Timeout，所以通过缩短从接收到SYN报文到确定这个报文无效并丢弃改连接的时间，例如设置为20秒以下，可以成倍的降低服务器的负荷。但过低的SYN Timeout设置可能会影响客户的正常访问。

* 网关超时设置

  ​      防火墙计数器到时，还没收到第3次握手包，则往服务器发送RST包，以使服务器从对列中删除该半连接。

  ​      网关超时设置，不宜过小也不宜过大。过小影响正常通讯，过大，影响防范SYN攻击的效果。

## **HTTPS 协议降级**

https://zhuanlan.zhihu.com/p/22917510

## http 请求头

```html
GET /home.html HTTP/1.1
Host: developer.mozilla.org
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:50.0) Gecko/20100101 Firefox/50.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://developer.mozilla.org/testpage.html
Connection: keep-alive
Upgrade-Insecure-Requests: 1
If-Modified-Since: Mon, 18 Jul 2016 02:36:04 GMT
If-None-Match: "c561c68d0ba92bbeb8b0fff2a9199f722e3a621a"
Cache-Control: max-age=0
```

## http 状态码

服务器返回的 **响应报文** 中第一行为状态行，包含了状态码以及原因短语，用来告知客户端请求的结果。

| 状态码 | 类别                             | 含义                       |
| ------ | -------------------------------- | -------------------------- |
| 1XX    | Informational（信息性状态码）    | 接收的请求正在处理         |
| 2XX    | Success（成功状态码）            | 请求正常处理完毕           |
| 3XX    | Redirection（重定向状态码）      | 需要进行附加操作以完成请求 |
| 4XX    | Client Error（客户端错误状态码） | 服务器无法处理请求         |
| 5XX    | Server Error（服务器错误状态码） | 服务器处理请求出错         |

###### [1XX 信息]

- **100 Continue** ：表明到目前为止都很正常，客户端可以继续发送请求或者忽略这个响应。

###### [2XX 成功】

- **200 OK**
- **204 No Content** ：请求已经成功处理，但是返回的响应报文不包含实体的主体部分。一般在只需要从客户端往服务器发送信息，而不需要返回数据时使用。
- **206 Partial Content** ：表示客户端进行了范围请求，响应报文包含由 Content-Range 指定范围的实体内容。

###### [3XX 重定向]

- **301 Moved Permanently** ：永久性重定向
- **302 Found** ：临时性重定向
- **303 See Other** ：和 302 有着相同的功能，但是 303 明确要求客户端应该采用 GET 方法获取资源。
- 注：虽然 HTTP 协议规定 301、302 状态下重定向时不允许把 POST 方法改成 GET 方法，但是大多数浏览器都会在 301、302 和 303 状态下的重定向把 POST 方法改成 GET 方法。
- **304 Not Modified** ：如果请求报文首部包含一些条件，例如：If-Match，If-Modified-Since，If-None-Match，If-Range，If-Unmodified-Since，如果不满足条件，则服务器会返回 304 状态码。
- **307 Temporary Redirect** ：临时重定向，与 302 的含义类似，但是 307 要求浏览器不会把重定向请求的 POST 方法改成 GET 方法。

###### [4XX 客户端错误]

- **400 Bad Request** ：请求报文中存在语法错误。
- **401 Unauthorized** ：该状态码表示发送的请求需要有认证信息（BASIC 认证、DIGEST 认证）。如果之前已进行过一次请求，则表示用户认证失败。
- **403 Forbidden** ：请求被拒绝。
- **404 Not Found**

###### [5XX 服务器错误]

- **500 Internal Server Error** ：服务器正在执行请求时发生错误。

- **503 Service Unavailable** ：服务器暂时处于超负载或正在进行停机维护，现在无法处理请求。

  

### HTTP/1.x 的缺陷

- **连接无法复用**：连接无法复用会导致每次请求都经历三次握手和慢启动。三次握手在高延迟的场景下影响较明显，慢启动则对大量小文件请求影响较大（没有达到最大窗口请求就被终止）。
  - HTTP/1.0 传输数据时，每次都需要重新建立连接，增加延迟。
  - HTTP/1.1 虽然加入 keep-alive 可以复用一部分连接，但域名分片等情况下仍然需要建立多个 connection，耗费资源，给服务器带来性能压力。
- **Head-Of-Line Blocking（HOLB）**：导致带宽无法被充分利用，以及后续健康请求被阻塞。[HOLB](http://stackoverflow.com/questions/25221954/spdy-head-of-line-blocking)是指一系列包（package）因为第一个包被阻塞；当页面中需要请求很多资源的时候，HOLB（队头阻塞）会导致在达到最大请求数量时，剩余的资源需要等待其他资源请求完成后才能发起请求。
  - HTTP 1.0：下个请求必须在前一个请求返回后才能发出，`request-response`对按序发生。显然，如果某个请求长时间没有返回，那么接下来的请求就全部阻塞了。
  - HTTP 1.1：尝试使用 pipeling 来解决，即浏览器可以一次性发出多个请求（同个域名，同一条 TCP 链接）。但 pipeling 要求返回是按序的，那么前一个请求如果很耗时（比如处理大图片），那么后面的请求即使服务器已经处理完，仍会等待前面的请求处理完才开始按序返回。所以，pipeling 只部分解决了 HOLB。
- **协议开销大**： HTTP/1.x 在使用时，header 里携带的内容过大，在一定程度上增加了传输的成本，并且每次请求 header 基本不怎么变化，尤其在移动端增加用户流量。
- **安全因素**：HTTP/1.x 在传输数据时，所有传输的内容都是明文，客户端和服务器端都无法验证对方的身份，这在一定程度上无法保证数据的安全性

## http1.0 和http1.1 区别



1. **缓存处理**，在HTTP1.0中主要使用header里的If-Modified-Since,Expires来做为缓存判断的标准，HTTP1.1则引入了更多的缓存控制策略例如Entity tag，If-Unmodified-Since, If-Match, If-None-Match等更多可供选择的缓存头来控制缓存策略。

2. **带宽优化及网络连接的使用**，HTTP1.0中，存在一些浪费带宽的现象，例如客户端只是需要某个对象的一部分，而服务器却将整个对象送过来了，并且不支持断点续传功能，HTTP1.1则在请求头引入了range头域，它允许只请求资源的某个部分，即返回码是206（Partial Content），这样就方便了开发者自由的选择以便于充分利用带宽和连接。

3. **错误通知的管理**，在HTTP1.1中新增了24个错误状态响应码，如409（Conflict）表示请求的资源与资源的当前状态发生冲突；410（Gone）表示服务器上的某个资源被永久性的删除。

4. **Host头处理**，在HTTP1.0中认为每台服务器都绑定一个唯一的IP地址，因此，请求消息中的URL并没有传递主机名（hostname）。但随着虚拟主机技术的发展，在一台物理服务器上可以存在多个虚拟主机（Multi-homed Web Servers），并且它们共享一个IP地址。HTTP1.1的请求消息和响应消息都应支持Host头域，且请求消息中如果没有Host头域会报告一个错误（400 Bad Request）。

5. **长连接**，HTTP 1.1支持长连接（PersistentConnection）和请求的流水线（Pipelining）处理，在一个TCP连接上可以传送多个HTTP请求和响应，减少了建立和关闭连接的消耗和延迟，在HTTP1.1中默认开启Connection： keep-alive，一定程度上弥补了HTTP1.0每次请求都要创建连接的缺点。默认情况下，HTTP 请求是按顺序发出的，下一个请求只有在当前请求收到响应之后才会被发出。由于受到网络延迟和带宽的限制，在下一个请求被发送到服务器之前，可能需要等待很长时间。

   流水线是在同一条长连接上连续发出请求，而不用等待响应返回，这样可以减少延迟。

### HTTP/2 新特性

#### 1. 二进制传输

HTTP/2 采用二进制格式传输数据，而非 HTTP 1.x 的文本格式，二进制协议解析起来更高效。 HTTP / 1 的请求和响应报文，都是由起始行，首部和实体正文（可选）组成，各部分之间以文本换行符分隔。**HTTP/2 将请求和响应数据分割为更小的帧，并且它们采用二进制编码**。

#### 2. 多路复用

在 HTTP/2 中引入了多路复用的技术。多路复用很好的解决了浏览器限制同一个域名下的请求数量的问题，同时也接更容易实现全速传输，毕竟新开一个 TCP 连接都需要慢慢提升传输速度。

#### 3. Header 压缩

在 HTTP/1 中，我们使用文本的形式传输 header，在 header 携带 cookie 的情况下，可能每次都需要重复传输几百到几千的字节。

为了减少这块的资源消耗并提升性能， HTTP/2 对这些首部采取了压缩策略：

- HTTP/2 在客户端和服务器端使用“首部表”来跟踪和存储之前发送的键－值对，对于相同的数据，不再通过每次请求和响应发送；
- 首部表在 HTTP/2 的连接存续期内始终存在，由客户端和服务器共同渐进地更新;
- 每个新的首部键－值对要么被追加到当前表的末尾，要么替换表中之前的值

### 4. Server Push

Server Push 即服务端能通过 push 的方式将客户端需要的内容预先推送过去，也叫“cache push”。

可以想象以下情况，某些资源客户端是一定会请求的，这时就可以采取服务端 push 的技术，提前给客户端推送必要的资源，这样就可以相对减少一点延迟时间。当然在浏览器兼容的情况下你也可以使用 prefetch。
例如服务端可以主动把 JS 和 CSS 文件推送给客户端，而不需要客户端解析 HTML 时再发送这些请求。

## QUIC 

http://www.52im.net/thread-1309-1-1.html

## GET 和 POST 比较

最直观的区别就是GET把参数包含在URL中，POST通过request body传递参数。

- GET 请求可被缓存
- GET 请求保留在浏览器历史记录中
- GET 请求可被收藏为书签
- GET 请求不应在处理敏感数据时使用
- GET 请求有长度限制
- GET 请求只应当用于取回数据
- POST 请求不会被缓存
- POST 请求不会保留在浏览器历史记录中
- POST 不能被收藏为书签
- POST 请求对数据长度没有要求

1. GET参数通过URL传递，POST放在Request body中。
2. GET在浏览器回退时是无害的，而POST会再次提交请求。（按“后退”或“刷新”按钮时，post会重新提交数据，get不会）
3. GET产生的URL地址可以被Bookmark（书签），而POST不可以。（get请求可被收藏为书签，post不行）
4. GET请求会被浏览器主动cache（缓存），而POST不会，除非手动设置。（get请求会被缓存，post不会）
5. GET请求只能进行url编码，而POST支持以下4种编码方式（）。
         1.application/x-www-form-urlencoded     2.multipart/formdata  3.application/json  4.text/xml
6. GET请求参数会被完整保留在浏览器历史记录里，而POST中的参数不会被保留。
7. GET请求在URL中传送的参数是有长度限制的，而POST没有。
8. 对参数的数据类型，GET只接受ASCII字符，而POST没有限制。

## 

## JWT,OAuth2,Session对比

### 传统的session认证

http协议本身是一种无状态的协议，而这就意味着如果用户向我们的应用提供了用户名和密码来进行用户认证，那么下一次请求时，用户还要再一次进行用户认证才行，因为根据http协议，我们并不能知道是哪个用户发出的请求，所以为了让我们的应用能识别是哪个用户发出的请求，我们只能在服务器存储一份用户登录的信息，这份登录信息会在响应时传递给浏览器，告诉其保存为cookie,以便下次请求时发送给我们的应用，这样我们的应用就能识别请求来自哪个用户了,这就是传统的基于session认证。但是这种基于session的认证使应用本身很难得到扩展，随着不同客户端用户的增加，独立的服务器已无法承载更多的用户，而这时候基于session认证应用的问题就会暴露出来：

- Session: 每个用户经过我们的应用认证之后，我们的应用都要在服务端做一次记录，以方便用户下次请求的鉴别，通常而言session都是保存在内存中，而随着认证用户的增多，服务端的开销会明显增大
- 扩展性: 用户认证之后，服务端做认证记录，如果认证的记录被保存在内存中的话，这意味着用户下次请求还必须要请求在这台服务器上,这样才能拿到授权的资源，这样在分布式的应用上，相应的限制了负载均衡器的能力。这也意味着限制了应用的扩展能力
- CSRF: 因为是基于cookie来进行用户识别的, cookie如果被截获，用户就会很容易受到跨站请求伪造的攻击

###  基于**token**的鉴权机制

**JWT和OAuth2都是基于token的鉴权机制**。基于token的鉴权机制类似于http协议也是无状态的，它不需要在服务端去保留用户的认证信息或者会话信息。这就意味着基于token认证机制的应用不需要去考虑用户在哪一台服务器登录了，这就为应用的扩展提供了便利。

其基本的流程如下：

1. 用户使用用户名密码来请求服务器
2. 服务器进行验证用户的信息
3. 服务器通过验证发送给用户一个token
4. 客户端存储token，并在每次请求时附送上这个token值
5. 服务端验证token值，并返回数据

这个token必须要在每次请求时传递给服务端，它应该保存在请求头里， 另外，服务端要支持CORS(跨来源资源共享)策略，一般我们在服务端这么做就可以了`Access-Control-Allow-Origin: *`。

### JWT 认证协议与 OAuth2.0 授权框架不恰当比较

之所以说是不恰当，是因为JWT和OAuth2是完全不通过的概念。既然 JWT 和 OAuth2 没有可比性，为什么还要把这两个放在一起说呢？很多情况下，在讨论OAuth2的实现时，会把JSON Web Token作为一种认证机制使用。这也是为什么他们会经常一起出现。

1. JWT 是一种认证协议
    JWT提供了一种用于发布接入令牌（Access Token),并对发布的签名接入令牌进行验证的方法。 令牌（Token）本身包含了一系列声明，应用程序可以根据这些声明限制用户对资源的访问。
2. OAuth2 是一种授权框架
    OAuth2是一种授权框架，提供了一套详细的授权机制。用户或应用可以通过公开的或私有的设置，授权第三方应用访问特定资源。
3. JWT 使用场景
    JWT 的主要优势在于使用无状态、可扩展的方式处理应用中的用户会话。服务端可以通过内嵌的声明信息，很容易地获取用户的会话信息，而不需要去访问用户或会话的数据库。在一个分布式的面向服务的框架中，这一点非常有用。但是，如果系统中需要使用黑名单实现长期有效的 Token 刷新机制，这种无状态的优势就不明显了。

- 优势
  - 快速开发
  - 不需要 Cookie
  - JSON 在移动端的广泛应用
  - 不依赖于社交登录
  - 相对简单的概念理解
- 限制
  - Token有长度限制
  - Token不能撤销
  - 需要 Token 有失效时间限制（exp）

1. OAuth2 使用场景
    如果不介意API的使用依赖于外部的第三方认证提供者，你可以简单地把认证工作留给认证服务商去做。也就是常见的，去认证服务商（比如 Facebook）那里注册你的应用，然后设置需要访问的用户信息，比如电子邮箱、姓名等。当用户访问站点的注册页面时，会看到连接到第三方提供商的入口。用户点击以后被重定向到对应的认证服务商网站，获得用户的授权后就可以访问到需要的信息，然后重定向回来。

- 优势
  - 快速开发
  - 实施代码量小
  - 维护工作减少
  - 可以和 JWT 同时使用
  - 可针对不同应用扩展
- 限制
  - 框架沉重



## 同源 -》 协议，域名，端口 

## 跨域解决方案

#### jsonp 

利用script 无跨域，把本地函数作为回调函数传给服务器

###### JSONP的实现流程

- 声明一个回调函数，其函数名(如show)当做参数值，要传递给跨域请求数据的服务器，函数形参为要获取目标数据(服务器返回的data)。
- 创建一个\<script\>标签，把那个跨域的API数据接口地址，赋值给script的src,还要在这个地址中向服务器传递该函数名（可以通过问号传参:?callback=show）。
- 服务器接收到请求后，需要进行特殊的处理：把传递进来的函数名和它需要给你的数据拼接成一个字符串,例如：传递进去的函数名是show，它准备好的数据是`show('我不爱你')`。
- 最后服务器把准备的数据通过HTTP协议返回给客户端，客户端再调用执行之前声明的回调函数（show），对返回的数据进行操作。
- ![image-20200719144237595](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200719144237595.png)

JSONP优点是简单兼容性好，可用于解决主流浏览器的跨域数据访问的问题。**缺点是仅支持get方法具有局限性**

**不安全可能会遭受XSS攻击。** url劫持。

#### **CORS 需要浏览器和后端同时支持。IE 8 和 9 需要通过 XDomainRequest 来实现**。

浏览器会自动进行 CORS 通信，实现 CORS 通信的关键是后端。只要后端实现了 CORS，就实现了跨域。

```java
// 允许哪个方法访问我
res.setHeader('Access-Control-Allow-Methods', 'PUT')
// 预检的存活时间
res.setHeader('Access-Control-Max-Age', 6)
// OPTIONS请求不做任何处理
if (req.method === 'OPTIONS') {
  res.end() 
}
// 定义后台返回的内容
app.put('/getData', function(req, res) {
  console.log(req.headers)
  res.end('sss')
})

```

```js
  if (whitList.includes(origin)) {
    // 设置哪个源可以访问我
    res.setHeader('Access-Control-Allow-Origin', origin)
    // 允许携带哪个头访问我
    res.setHeader('Access-Control-Allow-Headers', 'name')
    // 允许哪个方法访问我
    res.setHeader('Access-Control-Allow-Methods', 'PUT')
    // 允许携带cookie
    res.setHeader('Access-Control-Allow-Credentials', true)
    // 预检的存活时间
    res.setHeader('Access-Control-Max-Age', 6)
    // 允许返回的头
    res.setHeader('Access-Control-Expose-Headers', 'name')
    if (req.method === 'OPTIONS') {
      res.end() // OPTIONS请求不做任何处理
    }
  }
 
```







## HTTP 方法

get, post, option, head,put,delete,connect, trace

## HTTP 首部

## ![img](http://dl.iteye.com/upload/attachment/0069/3451/412b4451-2738-3ebc-b1f6-a0cc13b9697b.jpg)

## **HTTPS与HTTP的区别**

- HTTPS协议需要到CA申请证书，一般免费证书很少，需要交费。

- HTTP协议运行在TCP之上，所有传输的内容都是明文，HTTPS运行在SSL/TLS之上，SSL/TLS运行在TCP之上，所有传输的内容都经过加密的。

- HTTP和HTTPS使用的是完全不同的连接方式，用的端口也不一样，前者是80，后者是443。

- HTTPS可以有效的防止运营商劫持，解决了防劫持的一个大问题。

  

## 五层协议

- **应用层** ：为特定应用程序提供数据传输服务，例如 HTTP、DNS 等协议。数据单位为报文。
- **传输层** ：为进程提供通用数据传输服务。由于应用层协议很多，定义通用的传输层协议就可以支持不断增多的应用层协议。运输层包括两种协议：传输控制协议 TCP，提供面向连接、可靠的数据传输服务，数据单位为报文段；用户数据报协议 UDP，提供无连接、尽最大努力的数据传输服务，数据单位为用户数据报。TCP 主要提供完整性服务，UDP 主要提供及时性服务。
- **网络层** ：为主机提供数据传输服务。而传输层协议是为主机中的进程提供数据传输服务。网络层把传输层传递下来的报文段或者用户数据报封装成分组。
- **数据链路层** ：网络层针对的还是主机之间的数据传输服务，而主机之间可以有很多链路，链路层协议就是为同一链路的主机提供数据传输服务。数据链路层把网络层传下来的分组封装成帧。
- **物理层** ：考虑的是怎样在传输媒体上传输数据比特流，而不是指具体的传输媒体。物理层的作用是尽可能屏蔽传输媒体和通信手段的差异，使数据链路层感觉不到这些差异。

## SSL/TLS 握手过程 

https://www.jianshu.com/p/7158568e4867

![image-20200628223628782](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200628223628782.png)

###### Client Hello

握手第一步是客户端向服务端发送 Client Hello 消息，这个消息里包含了一个客户端生成的随机数 **Random1**、客户端支持的加密套件（Support Ciphers）和 SSL Version 等信息

###### Server Hello

第二步是服务端向客户端发送 Server Hello 消息，这个消息会从 Client Hello 传过来的 Support Ciphers 里确定一份加密套件，这个套件决定了后续加密和生成摘要时具体使用哪些算法，另外还会生成一份随机数 **Random2**。注意，至此客户端和服务端都拥有了两个随机数（Random1+ Random2），这两个随机数会在后续生成对称秘钥时用到。

###### Certificate

这一步是服务端将自己的证书下发给客户端，让客户端验证自己的身份，客户端验证通过后取出证书中的公钥。



![image-20200628230637454](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200628230637454.png)



![image-20200628231144309](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200628231144309.png)







## dns

1、在浏览器中输入www  . qq  .com 域名，操作系统会先检查自己本地的hosts文件是否有这个网址映射关系，如果有，就先调用这个IP地址映射，完成域名解析。 

2、如果hosts里没有这个域名的映射，则查找本地DNS解析器缓存，是否有这个网址映射关系，如果有，直接返回，完成域名解析。 

3、如果hosts与本地DNS解析器缓存都没有相应的网址映射关系，首先会找TCP/ip参数中设置的首选DNS服务器，在此我们叫它本地DNS服务器，此服务器收到查询时，如果要查询的域名，包含在本地配置区域资源中，则返回解析结果给客户机，完成域名解析，此解析具有权威性。 

4、如果要查询的域名，不由本地DNS服务器区域解析，但该服务器已缓存了此网址映射关系，则调用这个IP地址映射，完成域名解析，此解析不具有权威性。 

5、如果本地DNS服务器本地区域文件与缓存解析都失效，则根据本地DNS服务器的设置（是否设置转发器）进行查询，如果未用转发模式，本地DNS就把请求发至13台根DNS，根DNS服务器收到请求后会判断这个域名(.com)是谁来授权管理，并会返回一个负责该顶级域名服务器的一个IP。本地DNS服务器收到IP信息后，将会联系负责.com域的这台服务器。这台负责.com域的服务器收到请求后，如果自己无法解析，它就会找一个管理.com域的下一级DNS服务器地址([http://qq.com](https://link.zhihu.com/?target=http%3A//qq.com))给本地DNS服务器。当本地DNS服务器收到这个地址后，就会找[http://qq.com](https://link.zhihu.com/?target=http%3A//qq.com)域服务器，重复上面的动作，进行查询，直至找到www  . qq  .com主机。 

6、如果用的是转发模式，此DNS服务器就会把请求转发至上一级DNS服务器，由上一级服务器进行解析，上一级服务器如果不能解析，或找根DNS或把转请求转至上上级，以此循环。不管是本地DNS服务器用是是转发，还是根提示，最后都是把结果返回给本地DNS服务器，由此DNS服务器再返回给客户机。

## 一个完整的HTTP请求过程

输入URL到页面展现的过程:

1. 输入URL后，会先进行域名解析。优先查找本地host文件有无对应的IP地址，没有的话去本地DNS服务器查找，还不行的话，本地DNS服务器会去找根DNS服务器要一个域服务器的地址进行查询，域服务器将要查询的域名的解析服务器地址返回给本地DNS，本地DNS去这里查询就OK了。
2. 浏览器拿到服务器的IP地址后，会向它发送HTTP请求。HTTP请求经由一层层的处理、封装、发出之后，最终经由网络到达服务器，建立TCP/IP连接，服务器接收到请求并开始处理。
3. 服务器构建响应，再经由一层层的处理、封装、发出后，到达客户端，浏览器处理请求。
4. 浏览器开始渲染页面，解析HTML，构建render树，根据render树的节点和CSS的对应关系，进行布局，绘制页面。





# WEB 攻防

## csrf

![image-20200719125351860](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200719125351860.png)



![image-20200719125410144](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200719125410144.png)



防止： 

* 加入referer /origin判断

* 加入csrf token

* 新增http请求头

### referer /origin判断

CSRF大多数情况下来自第三方域名，但并不能排除本域发起。如果攻击者有权限在本域发布评论（含链接、图片等，统称UGC），那么它可以直接在本域发起攻击，这种情况下**同源策略无法达到防护的作用。**综上所述：同源验证是一个相对简单的防范方法，能够防范绝大多数的 CSRF 攻击。但这并不是万无一失的，对于安全性要求较高，或者有较多用户输入内容的网站，我们就要对关键的接口做额外的防护措施。

### CSRF Token

我们可以要求所有的用户请求都携带一个 CSRF 攻击者无法获取到的Token。服务器通过校验请求是否携带正确的 Token，来把正常的请求和攻击的请求区分开，也可以防范 CSRF 的攻击。

1. 将CSRF Token输出到页面中
   首先，用户打开页面的时候，服务器需要给这个用户生成一个 Token，该 Token 通过加密算法对数据进行加密，一般 Token 都包括随机字符串和时间戳的组合，显然在提交时 Token 不能再放在 Cookie 中了，否则又会被攻击者冒用。因此，为了安全起见 Token 最好还是存在服务器的 Session 中，之后在每次页面加载时，使用 JS 遍历整个 DOM 树，对于 DOM 中所有的 a 和 form 标签后加入 Token。这样可以解决大部分的请求，但是对于在页面加载之后动态生成的 HTML 代码，这种方法就没有作用，还需要程序员在编码时手动添加 Token。
2. 页面提交的请求携带这个 Token
   对于 GET 请求，Token 将附在请求地址之后，这样 URL 就变成 `http://url?csrftoken=tokenvalue`。而对于 POST 请求来说，要在 form 的最后加上 ``
3. 服务器验证 Token 是否正确
   当用户从客户端得到了 Token，再次提交给服务器的时候，服务器需要判断 Token 的有效性，验证过程是先解密 Token，对比加密字符串以及时间戳，如果加密字符串一致且时间未过期，那么这个 Token 就是有效的。

这种方法要比之前检查 Referer 或者 Origin 要安全一些，Token 可以在产生并放于 Session 之中，然后在每次请求时把 Token 从 Session 中拿出，与请求中的 Token 进行比对，但这种方法的比较麻烦的在于如何把 Token 以参数的形式加入请求。如果在请求中找不到 Token，或者提供的值与会话中的值不匹配，则应中止请求，应重置 Token 并将事件记录为正在进行的潜在 CSRF 攻击。

Token 是一个比较有效的 CSRF 防护方法，只要页面没有 XSS 漏洞泄露 Token，那么接口的 CSRF 攻击就无法成功。
但是此方法的实现比较复杂，需要给每一个页面都写入 Token（前端无法使用纯静态页面），每一个 Form 及 Ajax 请求都携带这个 Token，后端对每一个接口都进行校验，并保证页面 Token 及请求 Token 一致。这就使得这个防护策略不能在通用的拦截上统一拦截处理，而需要每一个页面和接口都添加对应的输出和校验。这种方法工作量巨大，且有可能遗漏。（验证码和密码其实也可以起到 CSRF Token 的作用，而且更安全）

### 双重 Cookie 验证

在会话中存储 CSRF Token 比较繁琐，而且不能在通用的拦截上统一处理所有的接口。
那么另一种防御措施是使用双重提交 Cookie。利用 CSRF 攻击不能获取到用户 Cookie 的特点，我们可以要求 Ajax 和表单请求携带一个 Cookie 中的值。

双重Cookie采用以下流程：

1. 在用户访问网站页面时，向请求域名注入一个 Cookie，内容为随机字符串（例如 `csrfcookie=v8g9e4ksfhw`）。
2. 在前端向后端发起请求时，取出 Cookie，并添加到 URL 的参数中（接上例 `POST https://www.a.com/comment?csrfcookie=v8g9e4ksfhw`）。
3. 后端接口验证 Cookie 中的字段与 URL 参数中的字段是否一致，不一致则拒绝。

用双重 Cookie 防御 CSRF 的优点：

- 无需使用 Session，适用面更广，易于实施。
- Token 储存于客户端中，不会给服务器带来压力。
- 相对于 Token，实施成本更低，可以在前后端统一拦截校验，而不需要一个个接口和页面添加。

缺点：

- Cookie 中增加了额外的字段。
- 如果有其他漏洞（例如 XSS），攻击者可以注入 Cookie，那么该防御方式失效。
- 难以做到子域名的隔离。
- 为了确保 Cookie 传输安全，采用这种防御方式的最好确保用整站 HTTPS 的方式，如果还没切 HTTPS 的使用这种方式也会有风险。

### 

## SYN Flood攻击原理与防范

SYN Flood攻击正是利用了TCP连接的三次握手，假设一个用户向服务器发送了SYN报文(第一次握手)后突然死机或掉线，那么服务器在发出SYN+ACK应答报文(第二次握手)后是无法收到客户端的ACK报文的(第三次握手无法完成)，这种情况下服务器端一般会重试(再次发送SYN+ACK给客户端)并等待一段时间后丢弃这个未完成的连接，这段时间的长度我们称为SYN Timeout，一般来说这个时间是分钟的数量级(大约为30秒-2分钟)；一个用户出现异常导致服务器的一个线程等待1分钟并不会对服务器端造成什么大的影响，但如果有大量的等待丢失的情况发生，服务器端将为了维护一个非常大的半连接请求而消耗非常多的资源。我们可以想象大量的保存并遍历也会消耗非常多的CPU时间和内存，再加上服务器端不断对列表中的IP进行SYN+ACK的重试，服务器的负载将会变得非常巨大。如果服务器的TCP/IP栈不够强大，最后的结果往往是堆栈溢出崩溃。相对于攻击数据流，正常的用户请求就显得十分渺小，服务器疲于处理攻击者伪造的TCP连接请求而无暇理睬客户的正常请求，此时从正常客户会表现为打开页面缓慢或服务器无响应，这种情况就是我们常说的服务器端SYN Flood攻击(SYN洪水攻击)。



 第一种是缩短SYN Timeout时间，由于SYN Flood攻击的效果取决于服务器上保持的SYN半连接数，这个值=SYN攻击的频度 x SYN Timeout，所以通过缩短从接收到SYN报文到确定这个报文无效并丢弃改连接的时间，例如设置为20秒以下，可以成倍的降低服务器的负荷。但过低的SYN Timeout设置可能会影响客户的正常访问。

　　第二种方法是设置SYN Cookie，就是给每一个请求连接的IP地址分配一个Cookie，如果短时间内连续受到某个IP的重复SYN报文，就认定是受到了攻击，并记录地址信息，以后从这个IP地址来的包会被一概丢弃。这样做的结果也可能会影响到正常用户的访问。



　通过设置注册表防御SYN Flood攻击，采用的是被动的策略，无论系统如何强大，始终不能靠被动的防护支撑下去，下面我们来看看另外一种比较有效的方法。

　　我称这种策略为“牺牲”策略，基于SYN Flood攻击代码的一个缺陷，我们重新来分析一下SYN Flood攻击者的流程：SYN Flood程序有两种攻击方式，基于IP的和基于域名的，前者是攻击者自己进行域名解析并将IP地址传递给攻击程序，后者是攻击程序自动进行域名解析，但是它们有一点是相同的，就是一旦攻击开始，将不会再进行域名解析，我们就是要利用这一点，假设一台服务器在受到SYN Flood攻击后迅速更换自己的IP地址，那么攻击者仍在不断攻击的只是一个空的IP地址，并没有任何主机，而管理员只需将DNS解析更改到新的IP地址就能在很短的时间内恢复用户通过域名进行的正常访问，这种做法取决于DNS的刷新时间。为了迷惑攻击者，我们甚至可以放置一台“牺牲”服务器，对攻击数据流进行牵引。

　　同样的原因，在众多的负载均衡架构中，基于DNS解析的负载均衡本身就拥有对SYN Flood的免疫力，基于DNS解析的负载均衡能将用户的请求分配到不同IP的服务器主机上，攻击者的一次攻击永远只能是其中一台服务器，虽然说攻击者也能不断去进行DNS请求来持续对用户的攻击，但是这样增加了攻击者的攻击强度，同时由于过多的DNS请求，可以帮助管理员查找到攻击者的地址，这主要是由于DNS请求需要返回数据，而这个数据是很难被伪装的。

# NGINX

https://blog.csdn.net/yusiguyuan/article/details/39249953



## MASTER& WORKER 

https://aimuke.github.io/nginx/2019/06/20/nginx-concurrent/

## master进程

主要用来管理worker进程，包含：

- 接收来自外界的信号
- 向各worker进程发送信号
- 监控worker进程的运行状态，当worker进程退出后(异常情况下)，会自动重新启动新的worker进程

master进程充当整个进程组与用户的交互接口，同时对进程进行监护。它不需要处理网络事件，不负责业务的执行，只会通过管理worker进程来实现重启服务、平滑升级、更换日志文件、配置文件实时生效等功能。

我们要控制nginx，只需要通过 `kill` 向master进程发送信号就行了。比如`kill -HUP pid` 是告诉nginx从容地重启nginx。我们一般用这个信号来重启nginx，或重新加载配置，因为是从容地重启，因此服务是不中断的。master进程在接收到`HUP`信号后是怎么做的呢？

- 首先master进程在接到信号后，会先重新加载配置文件
- 然后再启动新的worker进程
- 并向所有老的worker进程发送信号，告诉他们可以光荣退休了
- 新的worker在启动后，就开始接收新的请求，
- 老的worker在收到来自master的信号后，就不再接收新的请求，并且在当前进程中的所有未处理完的请求处理完成后，再退出。

当然，直接给master进程发送信号，这是比较老的操作方式，nginx在0.8版本之后，引入了一系列命令行参数，来方便我们管理。比如 `./nginx -s reload` 就是来重启nginx，`./nginx -s stop` 就是来停止nginx的运行。如何做到的呢？我们还是拿`reload` 来说，我们看到，执行命令时，我们是启动一个新的nginx进程，而新的nginx进程在解析到reload参数后，就知道我们的目的是控制nginx来重新加载配置文件了，它会向master进程发送信号，然后接下来的动作，就和我们直接向master进程发送信号一样了。

## worker进程

而基本的网络事件，则是放在worker进程中来处理了。多个worker进程之间是对等的，他们同等竞争来自客户端的请求，各进程互相之间是独立的。一个请求只可能在一个worker进程中处理，一个worker进程不可能处理其它进程的请求。worker进程的个数是可以设置的，一般我们会设置与机器cpu核数一致，这里面的原因与nginx的进程模型以及事件处理模型是分不开的。

worker进程之间是平等的，每个进程处理请求的机会也是一样的。当我们提供80端口的http服务时，一个连接请求过来，每个进程都有可能处理这个连接，怎么做到的呢？首先，每个worker进程都是从master进程fork过来，在master进程里面，先建立好需要listen的socket（listenfd）之后，然后再fork出多个worker进程。所有worker进程的listenfd会在新连接到来时变得可读，为保证只有一个进程处理该连接，所有worker进程在注册listenfd读事件前抢`accept_mutex`，抢到互斥锁的那个进程注册`listenfd`读事件，在读事件里调用`accept`接受该连接。当一个worker进程在`accept`这个连接之后，就开始读取请求，解析请求，处理请求，产生数据后，再返回给客户端，最后才断开连接，这样一个完整的请求就是这样的了。我们可以看到，一个请求，完全由worker进程来处理，而且只在一个worker进程中处理。



# worker进程工作流程



当一个 worker 进程在` accept()` 这个连接之后，就开始读取请求，解析请求，处理请求，产生数据后，再返回给客户端，最后才断开连接，一个完整的请求。一个请求完全由 worker 进程来处理，而且只能在一个 worker 进程中处理。

这样做带来的好处：

- 节省锁带来的开销。每个 worker 进程都是独立的进程，不共享资源，不需要加锁。同时在编程以及问题查上时，也会方便很多。
- 独立进程，减少风险。采用独立的进程，可以让互相之间不会影响，一个进程退出后，其它进程还在工作，服务不会中断，master 进程则很快重新启动新的 worker 进程。当然，worker 进程的也能发生意外退出。

多进程模型每个进程/线程只能处理一路IO，那么 Nginx是如何处理多路IO呢？

如果不使用 IO 多路复用，那么在一个进程中，同时只能处理一个请求，比如执行 `accept()`，如果没有连接过来，那么程序会阻塞在这里，直到有一个连接过来，才能继续向下执行。

而多路复用，允许我们只在事件发生时才将控制返回给程序，而其他时候内核都挂起进程，随时待命。



### 核心：Nginx采用的 IO多路复用模型epoll

`epoll`通过在Linux内核中申请一个简易的文件系统（文件系统一般用什么数据结构实现？B+树），其工作流程分为三部分：

1. 调用 `int epoll_create(int size)` 建立一个`epoll`对象，内核会创建一个`eventpoll`结构体，用于存放通过`epoll_ctl()`向 `epoll` 对象中添加进来的事件，这些事件都会挂载在红黑树中。
2. 调用 `int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)` 在 epoll 对象中为 `fd` 注册事件，所有添加到epoll中的事件都会与设备驱动程序建立回调关系，也就是说，当相应的事件发生时会调用这个sockfd的回调方法，将`sockfd`添加到 `eventpoll` 中的双链表
3. 调用 `int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout)` 来等待事件的发生，`timeout` 为 `-1` 时，该调用会阻塞直到有事件发生

这样，注册好事件之后，只要有 `fd` 上事件发生，`epoll_wait()` 就能检测到并返回给用户，用户就能”非阻塞“地进行 I/O 了。

`epoll()` 中内核则维护一个链表， `epoll_wait` 直接检查链表是不是空就知道是否有文件描述符准备好了。（`epoll` 与 `select` 相比最大的优点是不会随着 `sockfd` 数目增长而降低效率，使用 `select()` 时，内核采用轮训的方法来查看是否有 `fd` 准备好，其中的保存 `sockfd` 的是类似数组的数据结构 `fd_set`，key 为 `fd` ，value 为 `0` 或者 `1`。）

能达到这种效果，是因为在内核实现中 `epoll` 是根据每个 `sockfd` 上面的与设备驱动程序建立起来的回调函数实现的。那么，某个 `sockfd` 上的事件发生时，与它对应的回调函数就会被调用，来把这个 `sockfd` 加入链表，其他处于“空闲的”状态的则不会。在这点上， `epoll` 实现了一个”伪”AIO。但是如果绝大部分的 I/O 都是“活跃的”，每个 socket 使用率很高的话，epoll效率不一定比 select 高（可能是要维护队列复杂）。

可以看出，因为一个进程里只有一个线程，所以一个进程同时只能做一件事，但是可以通过不断地切换来“同时”处理多个请求。

**例子：** Nginx 会注册一个事件：“如果来自一个新客户端的连接请求到来了，再通知我”，此后只有连接请求到来，服务器才会执行 `accept()` 来接收请求。又比如向上游服务器（比如 PHP-FPM）转发请求，并等待请求返回时，这个处理的 worker 不会在这阻塞，它会在发送完请求后，注册一个事件：“如果缓冲区接收到数据了，告诉我一声，我再将它读进来”，于是进程就空闲下来等待事件发生。

这样，基于 多进程 + `epoll` ， Nginx 便能实现高并发。

使用 epoll 处理事件的一个框架，代码转自：http://www.cnblogs.com/fnlingnzb-learner/p/5835573.html

```c
for( ; ; )  //  无限循环
{
	nfds = epoll_wait(epfd,events,20,500);  //  最长阻塞 500s
	for(i=0;i<nfds;++i)
	{
		if(events[i].data.fd==listenfd) //有新的连接
		{
			connfd = accept(listenfd,(sockaddr *)&clientaddr, &clilen); //accept这个连接
			ev.data.fd=connfd;
			ev.events=EPOLLIN|EPOLLET;
			epoll_ctl(epfd,EPOLL_CTL_ADD,connfd,&ev); //将新的fd添加到epoll的监听队列中
		}
		else if( events[i].events&EPOLLIN ) //接收到数据，读socket
		{
			n = read(sockfd, line, MAXLINE)) < 0    //读
			ev.data.ptr = md;     //md为自定义类型，添加数据
			ev.events=EPOLLOUT|EPOLLET;
			epoll_ctl(epfd,EPOLL_CTL_MOD,sockfd,&ev);//修改标识符，等待下一个循环时发送数据，异步处理的精髓
		}
		else if(events[i].events&EPOLLOUT) //有数据待发送，写socket
		{
			struct myepoll_data* md = (myepoll_data*)events[i].data.ptr;    //取数据
			sockfd = md->fd;
			send( sockfd, md->ptr, strlen((char*)md->ptr), 0 );        //发送数据
			ev.data.fd=sockfd;
			ev.events=EPOLLIN|EPOLLET;
			epoll_ctl(epfd,EPOLL_CTL_MOD,sockfd,&ev); //修改标识符，等待下一个循环时接收数据
		}
		else
		{
			//其他的处理
		}
	}
}
```

### 正向代理和反向代理的区别

正向代理是一个位于客户端和目标服务器之间的代理服务器(中间服务器)。为了从原始服务器取得内容，客户端向代理服务器发送一个请求，并且指定目标服务器，之后代理向目标服务器转交并且将获得的内容返回给客户端。正向代理的情况下客户端必须要进行一些特别的设置才能使用。

- 正向代理：正向代理用途是为了在防火墙内的局域网提供访问internet的途径。另外还可以使用缓冲特性减少网络使用率
- 正向代理：正向代理允许客户端通过它访问任意网站并且隐蔽客户端自身，因此你必须采取安全措施来确保仅为经过授权的客户端提供服务

反向代理正好相反。对于客户端来说，反向代理就好像目标服务器。并且客户端不需要进行任何设置。客户端向反向代理发送请求，接着反向代理判断请求走向何处，并将请求转交给客户端，使得这些内容就好似他自己一样，一次客户端并不会感知到反向代理后面的服务，也因此不需要客户端做任何设置，只需要把反向代理服务器当成真正的服务器就好了。

- 反向代理：反向代理的用途是将防火墙后面的服务器提供给internet用户访问。同时还可以完成诸如负载均衡等功能
- 反向代理：对外是透明的，访问者并不知道自己访问的是代理。对访问者而言，他以为访问的就是原始服务器



### Nginx 处理一个 HTTP 请求的全过程

1. Read Request Headers：解析请求头。
2. Identify Configuration Block：识别由哪一个 location 进行处理，匹配 URL。
3. Apply Rate Limits：判断是否限速。例如可能这个请求并发的连接数太多超过了限制，或者 QPS 太高。
4. Perform Authentication：连接控制，验证请求。例如可能根据 Referrer 头部做一些防盗链的设置，或者验证用户的权限。
5. Generate Content：生成返回给用户的响应。为了生成这个响应，做反向代理的时候可能会和上游服务（Upstream Services）进行通信，然后这个过程中还可能会有些子请求或者重定向，那么还会走一下这个过程（Internal redirects and subrequests）。
6. Response Filters：过滤返回给用户的响应。比如压缩响应，或者对图片进行处理。
7. Log：记录日志。



## Nginx的事件处理机制：

对于一个基本的web服务器来说，事件通常有三种类型，网络事件、信号、定时器。
首先看一个请求的基本过程：建立连接---接收数据---发送数据 。
再次看系统底层的操作 ：上述过程（建立连接---接收数据---发送数据）在系统底层就是读写事件。

分析：

​	1）如果采用阻塞调用的方式，当读写事件没有准备好时，必然不能够进行读写事件，那么久只好等待，等事件准备好了，才能进行读写事件。那么请求就会被耽搁 。阻塞调用会进入内核等待，cpu就会让出去给别人用了，对单线程的worker来说，显然不合适，当网络事 件越多时，大家都在等待呢，cpu空闲下来没人用，cpu利用率自然上不去了，更别谈高并发了 。      

​	2）既然没有准备好阻塞调用不行，那么采用非阻塞方式。非阻塞就是，事件，马上返回EAGAIN，告诉你，事件还没准备好呢，你慌什么，过会再来吧。好吧，你过一会，再来检查一下事件，直到事件准备好了为止，在这期间，你就可以先去做其它事情，然后再来看看事件好了没。虽然不阻塞了，但你得不时地过来检查一下事件的状态，你可以做更多的事情了，但带来的开销也是不小的 

小结：非阻塞通过不断检查事件的状态来判断是否进行读写操作，这样带来的开销很大。 

！！！因此才有了**异步非阻塞**的事件处理机制。具体到系统调用就是像select/poll/epoll/kqueue这样的系统调用。他们提供了一种机制，让你可以同时监控多个事件，调用他们是阻塞的，但可以设置超时时间，在超时时间之内，如果有事件准备好了，就返回。这种机制解决了我们上面两个问题。 



以epoll为例：当事件没有准备好时，就放入epoll(队列)里面。如果有事件准备好了，那么就去处理；如果事件返回的是EAGAIN，那么继续将其放入epoll里面。从而，只要有事件准备好了，我们就去处理她，只有当所有时间都没有准备好时，才在epoll里面等着。这样 ，我们就可以并发处理大量的并发了，当然，这里的并发请求，是指未处理完的请求，线程只有一个，所以同时能处理的请求当然只有一个了，只是在请求间进行不断地切换而已，切换也是因为异步事件未准备好，而主动让出的。这里的切换是没有任何代价，你可以理 解为循环处理多个准备好的事件，事实上就是这样的。 

4）与多线程的比较：
与多线程相比，这种事件处理方式是有很大的优势的，不需要创建线程，每个请求占用的内存也很少，没有上下文切换，事件处理非常的轻量级。并发数再多也不会导致无谓的资源浪费（上下文切换）。

小结：通过异步非阻塞的事件处理机制，Nginx实现由进程循环处理多个准备好的事件，从而实现高并发和轻量级。



## nginx做负载均衡时其中一台服务器挂掉宕机时响应速度慢的问题解决

nginx会根据预先设置的权重转发请求，若给某一台服务器转发请求时，达到默认超时时间未响应，则再向另一台服务器转发请求。默认超时时间1分钟。



# I/O 模型

Unix 有五种 I/O 模型：

- 阻塞式 I/O
- 非阻塞式 I/O
- I/O 复用（select 和 poll）
- 信号驱动式 I/O（SIGIO）
- 异步 I/O（AIO）

## [阻塞式 I/O](https://cyc2018.github.io/CS-Notes/#/notes/Socket?id=阻塞式-io)

应用进程被阻塞，直到数据从内核缓冲区复制到应用进程缓冲区中才返回。

应该注意到，在阻塞的过程中，其它应用进程还可以执行，因此阻塞不意味着整个操作系统都被阻塞。因为其它应用进程还可以执行，所以不消耗 CPU 时间，这种模型的 CPU 利用率会比较高。

下图中，recvfrom() 用于接收 Socket 传来的数据，并复制到应用进程的缓冲区 buf 中。这里把 recvfrom() 当成系统调用。

```c
ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags, struct sockaddr *src_addr, socklen_t *addrlen);Copy to clipboardErrorCopied
```

![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/1492928416812_4.png)



## [非阻塞式 I/O](https://cyc2018.github.io/CS-Notes/#/notes/Socket?id=非阻塞式-io)

应用进程执行系统调用之后，内核返回一个错误码。应用进程可以继续执行，但是需要不断的执行系统调用来获知 I/O 是否完成，这种方式称为轮询（polling）。

由于 CPU 要处理更多的系统调用，因此这种模型的 CPU 利用率比较低。

![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/1492929000361_5.png)



## [I/O 复用](https://cyc2018.github.io/CS-Notes/#/notes/Socket?id=io-复用)

使用 select 或者 poll 等待数据，并且可以等待多个套接字中的任何一个变为可读。这一过程会被阻塞，当某一个套接字可读时返回，之后再使用 recvfrom 把数据从内核复制到进程中。

它可以让单个进程具有处理多个 I/O 事件的能力。又被称为 Event Driven I/O，即事件驱动 I/O。

如果一个 Web 服务器没有 I/O 复用，那么每一个 Socket 连接都需要创建一个线程去处理。如果同时有几万个连接，那么就需要创建相同数量的线程。相比于多进程和多线程技术，I/O 复用不需要进程线程创建和切换的开销，系统开销更小。

![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/1492929444818_6.png)



## [信号驱动 I/O](https://cyc2018.github.io/CS-Notes/#/notes/Socket?id=信号驱动-io)

应用进程使用 sigaction 系统调用，内核立即返回，应用进程可以继续执行，也就是说等待数据阶段应用进程是非阻塞的。内核在数据到达时向应用进程发送 SIGIO 信号，应用进程收到之后在信号处理程序中调用 recvfrom 将数据从内核复制到应用进程中。

相比于非阻塞式 I/O 的轮询方式，信号驱动 I/O 的 CPU 利用率更高。

![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/1492929553651_7.png)



## [异步 I/O](https://cyc2018.github.io/CS-Notes/#/notes/Socket?id=异步-io)

应用进程执行 aio_read 系统调用会立即返回，应用进程可以继续执行，不会被阻塞，内核会在所有操作完成之后向应用进程发送信号。

异步 I/O 与信号驱动 I/O 的区别在于，异步 I/O 的信号是通知应用进程 I/O 完成，而信号驱动 I/O 的信号是通知应用进程可以开始 I/O。

![img](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/1492930243286_8.png)



## IO复用 https://www.cnblogs.com/Anker/p/3265058.html

select 和 poll 的功能基本相同，不过在一些实现细节上有所不同。

- select 会修改描述符，而 poll 不会；
- select 的描述符类型使用数组实现，FD_SETSIZE 大小默认为 1024，因此默认只能监听少于 1024 个描述符。如果要监听更多描述符的话，需要修改 FD_SETSIZE 之后重新编译；而 poll 没有描述符数量的限制；
- poll 提供了更多的事件类型，并且对描述符的重复利用上比 select 高。
- 如果一个线程对某个描述符调用了 select 或者 poll，另一个线程关闭了该描述符，会导致调用结果不确定。

select 和 poll 速度都比较慢，每次调用都需要将全部描述符从应用进程缓冲区复制到内核缓冲区。

## epoll

```c
int epoll_create(int size);
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)；
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);Copy to clipboardErrorCopied
```

epoll_ctl() 用于向内核注册新的描述符或者是改变某个文件描述符的状态。已注册的描述符在内核中会被维护在一棵红黑树上，通过回调函数内核会将 I/O 准备好的描述符加入到一个链表中管理，进程调用 epoll_wait() 便可以得到事件完成的描述符。

从上面的描述可以看出，epoll 只需要将描述符从进程缓冲区向内核缓冲区拷贝一次，并且进程不需要通过轮询来获得事件完成的描述符。

epoll 仅适用于 Linux OS。

epoll 比 select 和 poll 更加灵活而且没有描述符数量限制。

epoll 对多线程编程更有友好，一个线程调用了 epoll_wait() 另一个线程关闭了同一个描述符也不会产生像 select 和 poll 的不确定情况。

```c
// Create the epoll descriptor. Only one is needed per app, and is used to monitor all sockets.
// The function argument is ignored (it was not before, but now it is), so put your favorite number here
int pollingfd = epoll_create( 0xCAFE );

if ( pollingfd < 0 )
 // report error

// Initialize the epoll structure in case more members are added in future
struct epoll_event ev = { 0 };

// Associate the connection class instance with the event. You can associate anything
// you want, epoll does not use this information. We store a connection class pointer, pConnection1
ev.data.ptr = pConnection1;

// Monitor for input, and do not automatically rearm the descriptor after the event
ev.events = EPOLLIN | EPOLLONESHOT;
// Add the descriptor into the monitoring list. We can do it even if another thread is
// waiting in epoll_wait - the descriptor will be properly added
if ( epoll_ctl( epollfd, EPOLL_CTL_ADD, pConnection1->getSocket(), &ev ) != 0 )
    // report error

// Wait for up to 20 events (assuming we have added maybe 200 sockets before that it may happen)
struct epoll_event pevents[ 20 ];

// Wait for 10 seconds, and retrieve less than 20 epoll_event and store them into epoll_event array
int ready = epoll_wait( pollingfd, pevents, 20, 10000 );
// Check if epoll actually succeed
if ( ret == -1 )
    // report error and abort
else if ( ret == 0 )
    // timeout; no event detected
else
{
    // Check if any events detected
    for ( int i = 0; i < ret; i++ )
    {
        if ( pevents[i].events & EPOLLIN )
        {
            // Get back our connection pointer
            Connection * c = (Connection*) pevents[i].data.ptr;
            c->handleReadEvent();
         }
    }
}Copy to clipboardErrorCopied
```

## [工作模式](https://cyc2018.github.io/CS-Notes/#/notes/Socket?id=工作模式)

epoll 的描述符事件有两种触发模式：LT（level trigger）和 ET（edge trigger）。

### [1. LT 模式](https://cyc2018.github.io/CS-Notes/#/notes/Socket?id=_1-lt-模式)

当 epoll_wait() 检测到描述符事件到达时，将此事件通知进程，进程可以不立即处理该事件，下次调用 epoll_wait() 会再次通知进程。是默认的一种模式，并且同时支持 Blocking 和 No-Blocking。

### [2. ET 模式](https://cyc2018.github.io/CS-Notes/#/notes/Socket?id=_2-et-模式)

和 LT 模式不同的是，通知之后进程必须立即处理事件，下次再调用 epoll_wait() 时不会再得到事件到达的通知。

很大程度上减少了 epoll 事件被重复触发的次数，因此效率要比 LT 模式高。只支持 No-Blocking，以避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死。

## [应用场景](https://cyc2018.github.io/CS-Notes/#/notes/Socket?id=应用场景)

很容易产生一种错觉认为只要用 epoll 就可以了，select 和 poll 都已经过时了，其实它们都有各自的使用场景。

### [1. select 应用场景](https://cyc2018.github.io/CS-Notes/#/notes/Socket?id=_1-select-应用场景)

select 的 timeout 参数精度为微秒，而 poll 和 epoll 为毫秒，因此 select 更加适用于实时性要求比较高的场景，比如核反应堆的控制。

select 可移植性更好，几乎被所有主流平台所支持。

### [2. poll 应用场景](https://cyc2018.github.io/CS-Notes/#/notes/Socket?id=_2-poll-应用场景)

poll 没有最大描述符数量的限制，如果平台支持并且对实时性要求不高，应该使用 poll 而不是 select。

### [3. epoll 应用场景](https://cyc2018.github.io/CS-Notes/#/notes/Socket?id=_3-epoll-应用场景)

只需要运行在 Linux 平台上，有大量的描述符需要同时轮询，并且这些连接最好是长连接。

需要同时监控小于 1000 个描述符，就没有必要使用 epoll，因为这个应用场景下并不能体现 epoll 的优势。

需要监控的描述符状态变化多，而且都是非常短暂的，也没有必要使用 epoll。因为 epoll 中的所有描述符都存储在内核中，造成每次需要对描述符的状态改变都需要通过 epoll_ctl() 进行系统调用，频繁系统调用降低效率。并且 epoll 的描述符存储在内核，不容易调试。

### 

# 框架

##  Spring 

### 什么是 Spring Framework？

Spring 是一个开源应用框架，旨在降低应用程序开发的复杂度。它是轻量级、松散耦合的。它具有分层体系结构，允许用户选择组件，同时还为 J2EE 应用程序开发提供了一个有凝聚力的框架。它可以集成其他框架，如 Structs、Hibernate、EJB 等，所以又称为框架的框架。

### **什么是依赖注入？** （DI）

**依赖注入**就是将实例变量传入到一个对象中去(Dependency injection means giving an object its instance variables)。是一种设计模式



###  **什么是 Spring IOC 容器？**

Spring 框架的核心是 Spring 容器。容器创建对象，将它们装配在一起，配置它们并管理它们的完整生命周期。Spring 容器使用依赖注入来管理组成应用程序的组件。容器通过读取提供的配置元数据来接收对象进行实例化，配置和组装的指令。该元数据可以通过 XML，Java 注解或 Java 代码提供。

IoC 的好处是：

- 它将最小化应用程序中的代码量。

- 易于测试，因为它不需要单元测试用例中的任何单例或 JNDI 查找机制。

- 它以最小的影响和最少的侵入机制促进松耦合。

- 它支持即时的实例化和延迟加载服务

  

### **spring 支持集中 bean scope？**

Spring bean 支持 5 种 scope：

- Singleton - 每个 Spring IoC 容器仅有一个单实例。
- Prototype - 每次请求都会产生一个新的实例。
- Request - 每一次 HTTP 请求都会产生一个新的实例，并且该 bean 仅在当前 HTTP 请求内有效。
- Session - 每一次 HTTP 请求都会产生一个新的 bean，同时该 bean 仅在当前 HTTP session 内有效。
- Global-session - 类似于标准的 HTTP Session 作用域，不过它仅仅在基于 portlet 的 web 应用中才有意义。Portlet 规范定义了全局 Session 的概念，它被所有构成某个 portlet web 应用的各种不同的 portlet 所共享。在 global session 作用域中定义的 bean 被限定于全局 portlet Session 的生命周期范围内。如果你在 web 中使用 global session 作用域来标识 bean，那么 web 会自动当成 session 类型来使用。

### **spring bean 容器的生命周期是什么样的**

![image-20200808113153863](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200808113153863.png)

spring bean 容器的生命周期流程如下：

1. Spring 容器根据配置中的 bean 定义中实例化 bean

2. Spring 使用依赖注入填充所有属性，如 bean 中所定义的配置。

3. 如果 bean 实现 BeanNameAware 接口，则工厂通过传递 bean 的 ID 来调用 setBeanName()。

4. 如果 bean 实现 BeanFactoryAware 接口，工厂通过传递自身的实例来调用 setBeanFactory()。

5. 如果存在与 bean 关联的任何 BeanPostProcessors，则调用 preProcessBeforeInitialization() 方法。

6. 如果为 bean 指定了 init 方法（ <bean>的 init-method 属性），那么将调用它。

7. 最后，如果存在与 bean 关联的任何 BeanPostProcessors，则将调用 postProcessAfterInitialization() 方法。

8. 如果 bean 实现 DisposableBean 接口，当 spring 容器关闭时，会调用 destory()。

9. 如果为 bean 指定了 destroy 方法（ <bean>的 destroy-method 属性），那么将调用它。

   ![image-20200720182058068](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200720182058068.png)

###  **什么是 AOP？**

AOP(Aspect-Oriented Programming), 即 面向切面编程, 它与 OOP( Object-Oriented Programming, 面向对象编程) 相辅相成, 提供了与 OOP 不同的抽象软件结构的视角.在 OOP 中, 我们以类(class)作为我们的基本单元, 而 AOP 中的基本单元是 Aspect(切面)

![preview](https://pic4.zhimg.com/v2-da46bf7e930122cfe63bbe21deb3c5a7_r.jpg)



### **AOP 中的 Aspect、Advice、Pointcut、JointPoint 和 Advice 参数分别是什么？**

1. Aspect - Aspect 是一个实现交叉问题的类，例如事务管理。方面可以是配置的普通类，然后在 Spring Bean 配置文件中配置，或者我们可以使用 Spring AspectJ 支持使用 @Aspect 注解将类声明为 Aspect。

2. Advice - Advice 是针对特定 JoinPoint 采取的操作。在编程方面，它们是在应用程序中达到具有匹配切入点的特定 JoinPoint 时执行的方法。您可以将 Advice 视为 Spring 拦截器（Interceptor）或 Servlet 过滤器（filter）。

3. Advice Arguments - 我们可以在 advice 方法中传递参数。我们可以在切入点中使用 args() 表达式来应用于与参数模式匹配的任何方法。如果我们使用它，那么我们需要在确定参数类型的 advice 方法中使用相同的名称。

4. Pointcut - Pointcut 是与 JoinPoint 匹配的正则表达式，用于确定是否需要执行 Advice。Pointcut 使用与 JoinPoint 匹配的不同类型的表达式。Spring 框架使用 AspectJ Pointcut 表达式语言来确定将应用通知方法的 JoinPoint。

5. JoinPoint - JoinPoint 是应用程序中的特定点，例如方法执行，异常处理，更改对象变量值等。在 Spring AOP 中，JoinPoint 始终是方法的执行器。

   
   
   ### **有哪些类型的通知（Advice）？**

- Before - 这些类型的 Advice 在 joinpoint 方法之前执行，并使用 @Before 注解标记进行配置。
- After Returning - 这些类型的 Advice 在连接点方法正常执行后执行，并使用@AfterReturning 注解标记进行配置。
- After Throwing - 这些类型的 Advice 仅在 joinpoint 方法通过抛出异常退出并使用 @AfterThrowing 注解标记配置时执行。
- After (finally) - 这些类型的 Advice 在连接点方法之后执行，无论方法退出是正常还是异常返回，并使用 @After 注解标记进行配置。
- Around - 这些类型的 Advice 在连接点之前和之后执行，并使用 @Around 注解标记进行配置。

### **AOP 有哪些实现方式？**

实现 AOP 的技术，主要分为两大类：

- 静态代理 - 指使用 AOP 框架提供的命令进行编译，从而在编译阶段就可生成 AOP 代理类，因此也称为编译时增强；
- 编译时编织（特殊编译器实现）
- 类加载时编织（特殊的类加载器实现）。
- 动态代理 - 在运行时在内存中“临时”生成 AOP 动态代理类，因此也被称为运行时增强。
- JDK 动态代理
- CGLIB

### **Spring AOP and AspectJ AOP 有什么区别？**

Spring AOP 基于动态代理方式实现；AspectJ 基于静态代理方式实现。

Spring AOP 仅支持方法级别的 PointCut；提供了完全的 AOP 支持，它还支持属性级别的 PointCut。

### Spring 如何解决循环依赖？

spring对循环依赖的处理有三种情况： ①构造器的循环依赖：这种依赖spring是处理不了的，直 接抛出BeanCurrentlylnCreationException异常。 ②单例模式下的setter循环依赖：通过“三级缓存”处理循环依赖。 ③非单例循环依赖：无法处理。



https://juejin.im/post/5c98a7b4f265da60ee12e9b2

Spring为了解决单例的循环依赖问题，使用了三级缓存。这三级缓存的作用分别是：

singletonFactories ： 进入实例化阶段的单例对象工厂的cache （三级缓存）

earlySingletonObjects ：完成实例化但是尚未初始化的，提前暴光的单例对象的Cache （二级缓存）

singletonObjects：完成初始化的单例对象的cache（一级缓存）

我们在创建bean的时候，会首先从cache中获取这个bean，这个缓存就是sigletonObjects。主要的调用方法是：



```java
/** Cache of singleton objects: bean name –> bean instance */
private final Map singletonObjects = new ConcurrentHashMap(256);
/** Cache of singleton factories: bean name –> ObjectFactory */
private final Map> singletonFactories = new HashMap>(16);
/** Cache of early singleton objects: bean name –> bean instance */
private final Map earlySingletonObjects = new HashMap(16);

protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    Object singletonObject = this.singletonObjects.get(beanName);
    //isSingletonCurrentlyInCreation()判断当前单例bean是否正在创建中
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        synchronized (this.singletonObjects) {
            singletonObject = this.earlySingletonObjects.get(beanName);
            //allowEarlyReference 是否允许从singletonFactories中通过getObject拿到对象
            if (singletonObject == null && allowEarlyReference) {
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    singletonObject = singletonFactory.getObject();
                    //从singletonFactories中移除，并放入earlySingletonObjects中。
                    //其实也就是从三级缓存移动到了二级缓存
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }
    return (singletonObject != NULL_OBJECT ? singletonObject : null);
}



```

### 解决读问题: 设置事务隔离级别（5种）

DEFAULT 这是一个PlatfromTransactionManager默认的隔离级别，使用数据库默认的事务隔离级别.
未提交读（read uncommited） :脏读，不可重复读，虚读都有可能发生
已提交读 （read commited）:避免脏读。但是不可重复读和虚读有可能发生
可重复读 （repeatable read） :避免脏读和不可重复读.但是虚读有可能发生.
串行化的 （serializable） :避免以上所有读问题.
Mysql 默认:可重复读
Oracle 默认:读已提交

read uncommited：是最低的事务隔离级别，它允许另外一个事务可以看到这个事务未提交的数据。
read commited：保证一个事物提交后才能被另外一个事务读取。另外一个事务不能读取该事物未提交的数据。
repeatable read：这种事务隔离级别可以防止脏读，不可重复读。但是可能会出现幻象读。它除了保证一个事务不能被另外一个事务读取未提交的数据之外还避免了以下情况产生（不可重复读）。
serializable：这是花费最高代价但最可靠的事务隔离级别。事务被处理为顺序执行。除了防止脏读，不可重复读之外，还避免了幻象读（避免三种）。

### 事务的传播行为（7 种）

PROPAGION_XXX :事务的传播行为

* 保证同一个事务中
  PROPAGATION_REQUIRED 支持当前事务，如果不存在 就新建一个(默认)
  PROPAGATION_SUPPORTS 支持当前事务，如果不存在，就不使用事务
  PROPAGATION_MANDATORY 支持当前事务，如果不存在，抛出异常
* 保证没有在同一个事务中
  PROPAGATION_REQUIRES_NEW 如果有事务存在，挂起当前事务，创建一个新的事务
  PROPAGATION_NOT_SUPPORTED 以非事务方式运行，如果有事务存在，挂起当前事务
  PROPAGATION_NEVER 以非事务方式运行，如果有事务存在，抛出异常
  PROPAGATION_NESTED 如果当前事务存在，则嵌套事务执行

1. ### spring事务：

   什么是事务:
   事务逻辑上的一组操作,组成这组操作的各个逻辑单元,要么一起成功,要么一起失败.

   事务特性（4种）:
   原子性 （atomicity）:强调事务的不可分割.
   一致性 （consistency）:事务的执行的前后数据的完整性保持一致.
   隔离性 （isolation）:一个事务执行的过程中,不应该受到其他事务的干扰
   持久性（durability） :事务一旦结束,数据就持久到数据库

   如果不考虑隔离性引发安全性问题:
   脏读 :一个事务读到了另一个事务的未提交的数据
   不可重复读 :一个事务读到了另一个事务已经提交的 update 的数据导致多次查询结果不一致.
   虚幻读 :一个事务读到了另一个事务已经提交的 insert 的数据导致多次查询结果不一致.

   

   ### @Autowired 原理


1.变量名用userService1,userService2，而不是userService。通常情况下@Autowired是通过byType的方法注入的，可是在多个实现类的时候，byType的方式不再是唯一，而需要通过byName的方式来注入，而这个name默认就是根据变量名来的。

2.通过@Qualifier注解来指明使用哪一个实现类，实际上也是通过byName的方式实现。由此看来，@Autowired注解到底使用byType还是byName，其实是存在一定策略的，也就是有优先级。优先用byType，而后是byName。

## Spring MVC

###   Spring MVC 框架有什么用？

Spring Web MVC 框架提供 模型-视图-控制器 架构和随时可用的组件，用于开发灵活且松散耦合的 Web 应用程序。MVC 模式有助于分离应用程序的不同方面，如输入逻辑，业务逻辑和 UI 逻辑，同时在所有这些元素之间提供松散耦合。

### DispatcherServlet 的工作流程

![image-20200712210842032](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200712210842032.png)

①客户端的所有请求都交给前端控制器DispatcherServlet来处理，它会负责调用系统的其他模块来真正处理用户的请求。

② DispatcherServlet收到请求后，将根据请求的信息（包括URL、HTTP协议方法、请求头、请求参数、Cookie等）以及HandlerMapping的配置找到处理该请求的Handler（任何一个对象都可以作为请求的Handler）。

③在这个地方Spring会通过HandlerAdapter对该处理器进行封装。

④ HandlerAdapter是一个适配器，它用统一的接口对各种Handler中的方法进行调用。

⑤ Handler完成对用户请求的处理后，会返回一个ModelAndView对象给DispatcherServlet，ModelAndView顾名思义，包含了数据模型以及相应的视图的信息。

⑥ ModelAndView的视图是逻辑视图，DispatcherServlet还要借助ViewResolver完成从逻辑视图到真实视图对象的解析工作。 ⑦ 当得到真正的视图对象后，DispatcherServlet会利用视图对象对模型数据进行渲染。

⑧ 客户端得到响应，可能是一个普通的HTML页面，也可以是XML或JSON字符串，还可以是一张图片或者一个PDF文件。


   链接：https://juejin.im/post/5d05cfa56fb9a07ee9586eb4


### SpringMVC的运行机制

1. 用户发送请求时会先从DispathcherServler的doService方法开始，在该方法中会将ApplicationContext、localeResolver、themeResolver等对象添加到request中，紧接着就是调用doDispatch方法。

2. 进入该方法后首先会检查该请求是否是文件上传的请求(校验的规则是是否是post并且contenttType是否为multipart/为前缀)即调用的是checkMultipart方法；如果是的将request包装成MultipartHttpServletRequest。

3. 然后调用getHandler方法来匹配每个HandlerMapping对象，如果匹配成功会返回这个Handle的处理链HandlerExecutionChain对象，在获取该对象的内部其实也获取我们自定定义的拦截器，并执行了其中的方法。

4. 执行拦截器的preHandle方法，如果返回false执行afterCompletion方法并理解返回

5. 通过上述获取到了HandlerExecutionChain对象，通过该对象的getHandler()方法获得一个object通过HandlerAdapter进行封装得到HandlerAdapter对象。

6. 该对象调用handle方法来执行Controller中的方法，该对象如果返回一个ModelAndView给DispatcherServlet。

7. DispatcherServlet借助ViewResolver完成逻辑试图名到真实视图对象的解析，得到View后DispatcherServlet使用这个View对ModelAndView中的模型数据进行视图渲染。

 

## SpringBoot 应用程序启动过程

https://www.jianshu.com/p/dc12081b3598

https://zhuanlan.zhihu.com/p/53022678

* SpringApplication的实例化 

  * 推断应用类型是否是Web环境
  * 设置初始化器（Initializer）：META-INF/spring.factories 读取相应配置文件，然后进行遍历，读取配置文件中Key为：org.springframework.context.ApplicationContextInitializer 的 value。
  * 设置监听器（Listener）
  * 推断应用入口类（Main）
  * 

* 

  ​	

  
  
  SpringBoot的启动主要是通过实例化SpringApplication来启动的，启动过程主要做了以下几件事情：配置属性、获取监听器，发布应用开始启动事件初、始化输入参数、配置环境，输出banner、**创建上下文**、预处理上下文、**刷新上下文(加载tomcat容器)**、再刷新上下文、发布应用已经启动事件、发布应用启动完成事件。

![image-20200808120457256](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200808120457256.png)





![image-20200808121233209](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200808121233209.png)

![image-20200808121705655](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200808121705655.png)



![image-20200808123648811](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200808123648811.png)



## @Transactional 注解管理事务的实现步骤

```xml
<tx:annotation-driven />
<bean id="transactionManager"
class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
<property name="dataSource" ref="dataSource" />
</bean>
```

将@Transactional 注解添加到合适的**方法**上，并设置合适的属性信息。@Transactional 注解的属性信息如下表  展示。

#####  @Transactional 注解的属性信息

| 属性名           | 说明                                                         |
| :--------------- | :----------------------------------------------------------- |
| name             | 当在配置文件中有多个 TransactionManager , 可以用该属性指定选择哪个事务管理器。 |
| propagation      | 事务的传播行为，默认值为 REQUIRED。                          |
| isolation        | 事务的隔离度，默认值采用 DEFAULT。                           |
| timeout          | 事务的超时时间，默认值为-1。如果超过该时间限制但事务还没有完成，则自动回滚事务。 |
| read-only        | 指定事务是否为只读事务，默认值为 false；为了忽略那些不需要事务的方法，比如读取数据，可以设置 read-only 为 true。 |
| rollback-for     | 用于指定能够触发事务回滚的异常类型，如果有多个异常类型需要指定，各类型之间可以通过逗号分隔。 |
| no-rollback- for | 抛出 no-rollback-for 指定的异常类型，不回滚事务。            |

@Transactional 注解也可以添加到**类级别**上。当把@Transactional 注解放在类级别时，表示所有该类的**公共方法**都配置相同的事务属性信息。见下，EmployeeService 的所有方法都支持事务并且是只读。当类级别配置了@Transactional，方法级别也配置了@Transactional，应用程序会以方法级别的事务属性信息来管理事务，换言之，方法级别的事务属性信息会覆盖类级别的相关配置信息。

```java
@Transactional(propagation= Propagation.SUPPORTS,readOnly=true)
@Service(value ="employeeService")
public class EmployeeService


```



## Spring 的注解方式的事务实现机制

在应用系统调用声明@Transactional 的目标方法时，Spring Framework 默认使用 AOP 代理，在代码运行时生成一个代理对象，根据@Transactional 的属性配置信息，这个代理对象决定该声明@Transactional 的目标方法是否由拦截器 TransactionInterceptor 来使用拦截，在 TransactionInterceptor 拦截时，会在在目标方法开始执行之前创建并加入事务，并执行目标方法的逻辑, 最后根据执行情况是否出现异常，利用抽象事务管理器(图 2 有相关介绍)AbstractPlatformTransactionManager 操作数据源 DataSource 提交或回滚事务, 如图 。

![image-20200711211019358](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200711211019358.png)











## 事务将不会发生回滚情况

需要注意下面三种 propagation 可以不启动事务。本来期望目标方法进行事务管理，但若是错误的配置这三种 propagation，事务将不会发生回滚。

1. TransactionDefinition.PROPAGATION_SUPPORTS：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。

2. TransactionDefinition.PROPAGATION_NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起。

3. TransactionDefinition.PROPAGATION_NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。

   

   默认情况下，如果在事务中抛出了未检查异常（继承自 RuntimeException 的异常）或者 Error，则 Spring 将回滚事务；除此之外，Spring 不会回滚事务。如果在事务中抛出其他类型的异常，并期望 Spring 能够回滚事务，可以指定 rollbackFor。例：

   >@Transactional(propagation= Propagation.REQUIRED,rollbackFor= MyException.class)

   通过分析 Spring 源码可以知道，若在目标方法中抛出的异常是 rollbackFor 指定的异常的子类，事务同样会回滚。

> ```java
> private int getDepth(Class<?> exceptionClass, int depth) {
>         if (exceptionClass.getName().contains(this.exceptionName)) {
>             // Found it!
>             return depth;
> }
>         // If we've gone as far as we can go and haven't found it...
>         if (exceptionClass == Throwable.class) {
>             return -1;
> }
> return getDepth(exceptionClass.getSuperclass(), depth + 1);
> }
> ```



4. @Transactional 只能应用到 public 方法才有效

5. 在 Spring 的 AOP 代理下，只有目标方法由外部调用，目标方法才由 Spring 生成的代理对象来管理，这会造成自调用问题。若同一类中的其他没有@Transactional 注解的方法内部调用有@Transactional 注解的方法，有@Transactional 注解的方法的事务被忽略，不会发生回滚。

   `AOP`使用的是动态代理的机制，它会给类生成一个代理类，事务的相关操作都在代理类上完成。内部方式使用`this`调用方式时，使用的是实例调用，并没有通过代理类调用方法，所以会导致事务失效。

   为解决这两个问题，使用 AspectJ 取代 Spring AOP 代理。

   

## OthERs

##  1. [MyBatis缓存机制]

### 一级缓存介绍

在应用运行过程中，我们有可能在一次数据库会话中，执行多次查询条件完全相同的SQL，MyBatis提供了一级缓存的方案优化这部分场景，如果是相同的SQL语句，会优先命中一级缓存，避免直接对数据库进行查询，提高性能。**一级缓存只在数据库会话内部共享**

2. ## 分布式锁

REDIS基于SetNX实现：
 setNX是Redis提供的一个原子操作，如果指定key存在，那么setNX失败，如果不存在会进行Set操作并返回成功。我们可以利用这个来实现一个分布式的锁，主要思路就是，set成功表示获取锁，set失败表示获取失败，失败后需要重试。

### set key value [EX seconds][PX milliseconds][NX|XX]

释放锁时需要验证value值，也就是说我们在获取锁的时候需要设置一个value，不能直接用del key这种粗暴的方式，因为直接del key任何客户端都可以进行解锁了，所以**解锁时，我们需要判断锁是否是自己的，基于value值来判断**

String luaScript = "if redis.call('get',KEYS[1]) == ARGV[1] then " + "return redis.call('del',KEYS[1]) else return 0 end";return jedis.eval(luaScript, Collections.singletonList(key), Collections.singletonList(value)).equals(1L);。

# 常见





## 字节序号

字节序分为两类：Big-Endian 和 Little-Endian，引用标准的 Big-Endian 和 Little-Endian 的定义如下：

- Little-Endian：就是低位字节排放在内存的低地址端，高位字节排放在内存的高地址端。
- Big-Endian：就是高位字节排放在内存的低地址端，低位字节排放在内存的高地址端。
- 网络字节序：TCP/IP各层协议将字节序定义为 Big-Endian（这与主机序相反），因此TCP/IP协议中使用的字节序通常称之为网络字节序。





## 乐观锁的缺点

> ABA 问题是乐观锁一个常见的问题

#### 1 ABA 问题

如果一个变量V初次读取的时候是A值，并且在准备赋值的时候检查到它仍然是A值，那我们就能说明它的值没有被其他线程修改过了吗？很明显是不能的，因为在这段时间它的值可能被改为其他值，然后又改回A，那CAS操作就会误认为它从来没有被修改过。这个问题被称为CAS操作的 **"ABA"问题。**

JDK 1.5 以后的 `AtomicStampedReference 类`就提供了此种能力，其中的 `compareAndSet 方法`就是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。

#### 2 循环时间长开销大

**自旋CAS（也就是不成功就一直循环执行直到成功）如果长时间不成功，会给CPU带来非常大的执行开销。** 如果JVM能支持处理器提供的pause指令那么效率会有一定的提升，pause指令有两个作用，第一它可以延迟流水线执行指令（de-pipeline）,使CPU不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零。第二它可以避免在退出循环的时候因内存顺序冲突（memory order violation）而引起CPU流水线被清空（CPU pipeline flush），从而提高CPU的执行效率。

#### 3 只能保证一个共享变量的原子操作

CAS 只对单个共享变量有效，当操作涉及跨多个共享变量时 CAS 无效。但是从 JDK 1.5开始，提供了`AtomicReference类`来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行 CAS 操作.所以我们可以使用锁或者利用`AtomicReference类`把多个共享变量合并成一个共享变量来操作。



链接：https://juejin.im/post/5b4977ae5188251b146b2fc8
处。



## 压缩算法：

> 1. RLE 算法的机制：
>
>   就是把相同的字符`去重化`，也就是 `字符 * 重复次数` 的方式进行压缩，例如**AAAAAABBCDDEEEEEF** 的17个字符成功被压缩成了 **A6B2C1D2E5F1** 的12个字符，也就是 12 / 17 = 70%，压缩比为 70%。RLE 算法的缺点：RLE 的压缩机制比较简单，所以 RLE 算法的程序也比较容易编写，所以使用 RLE 的这种方式更能让你体会到压缩思想，但是 RLE 只针对特定序列的数据管用
>
> 2.  哈夫曼算法和莫尔斯编码
>
>    哈夫曼算法的关键就在于 **多次出现的数据用小于 8 位的字节数表示，不常用的数据则可以使用超过 8 位的字节数表示**。
>
>    对 AAAAAABBCDDEEEEEF 中的 A - F 这些字符，按照`出现频率高的字符用尽量少的位数编码来表示`这一原则进行整理。按照出现频率从高到低的顺序整理后，结果如下，同时也列出了编码方案。
>
>    | 字符 | 出现频率 | 编码（方案） | 位数 |
>    | :--: | :------: | :----------: | :--: |
>    |  A   |    6     |      0       |  1   |
>    |  E   |    5     |      1       |  1   |
>    |  B   |    2     |      10      |  2   |
>    |  D   |    2     |      11      |  2   |
>    |  C   |    1     |     100      |  3   |
>    |  F   |    1     |     101      |  3   |
>
>    
>    
>
>    ![image-20200707132735634](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200707132735634.png)
>
> ![image-20200707133446327](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200707133446327.png)
>
> 2. 

# Filter/ Listener/Intecepter



1. 过滤器（Filter）：所谓过滤器顾名思义是用来过滤的，Java的过滤器能够为我们提供系统级别的过滤，也就是说，能过滤所有的web请求，这一点，是拦截器无法做到的。在Java Web中，你传入的request,response提前过滤掉一些信息，或者提前设置一些参数，然后再传入servlet或者struts的action进行业务逻辑，比如过滤掉非法url（不是login.do的地址请求，如果用户没有登陆都过滤掉）,或者在传入servlet或者struts的action前统一设置字符集，或者去除掉一些非法字符（聊天室经常用到的，一些骂人的话）。filter 流程是线性的，url传来之后，检查之后，可保持原来的流程继续向下执行，被下一个filter, servlet接收。

2. 监听器（Listener）：Java的监听器，也是系统级别的监听。监听器随web应用的启动而启动。Java的监听器在c/s模式里面经常用到，它会对特定的事件产生产生一个处理。监听在很多模式下用到，比如说观察者模式，就是一个使用监听器来实现的，在比如统计网站的在线人数。又比如struts2可以用监听来启动。Servlet监听器用于监听一些重要事件的发生，监听器对象可以在事情发生前、发生后可以做一些必要的处理。

3. 拦截器（Interceptor）：java里的拦截器提供的是非系统级别的拦截，也就是说，就覆盖面来说，拦截器不如过滤器强大，但是更有针对性。Java中的拦截器是基于Java反射机制实现的，更准确的划分，应该是基于JDK实现的动态代理。它依赖于具体的接口，在运行期间动态生成字节码。拦截器是动态拦截Action调用的对象，它提供了一种机制可以使开发者在一个Action执行的前后执行一段代码，也可以在一个Action执行前阻止其执行，同时也提供了一种可以提取Action中可重用部分代码的方式。在AOP中，拦截器用于在某个方法或者字段被访问之前，进行拦截然后再之前或者之后加入某些操作。java的拦截器主要是用在插件上，扩展件上比如 Hibernate Spring Struts2等，有点类似面向切片的技术，在用之前先要在配置文件即xml，文件里声明一段的那个东西。

   ![image-20200711223941102](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200711223941102.png)



# JWT

https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html



跨域的服务导向架构，就要求 session 数据共享，每台服务器都能够读取 session。

Example，A 网站和 B 网站是同一家公司的关联服务。现在要求，用户只要在其中一个网站登录，再访问另一个网站就会自动登录，请问怎么实现？



一种解决方案是 session 数据持久化，写入数据库或别的持久层。各种服务收到请求后，都向持久层请求数据。这种方案的优点是架构清晰，缺点是工程量比较大。另外，持久层万一挂了，就会单点失败。

另一种方案是服务器索性不保存 session 数据了，所有数据都保存在客户端，每次请求都发回服务器。JWT 就是这种方案的一个代表。

![image-20200714184240157](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200714184240157.png)

## JWT 的几个特点

（1）JWT 默认是不加密，但也是可以加密的。生成原始 Token 以后，可以用密钥再加密一次。

（2）JWT 不加密的情况下，不能将秘密数据写入 JWT。

（3）JWT 不仅可以用于认证，也可以用于交换信息。有效使用 JWT，可以降低服务器查询数据库的次数。

（4）JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。

（5）JWT 本身包含了认证信息，一旦泄露，任何人都可以获得该令牌的所有权限。为了减少盗用，JWT 的有效期应该设置得比较短。对于一些比较重要的权限，使用时应该再次对用户进行认证。

（6）为了减少盗用，JWT 不应该使用 HTTP 协议明码传输，要使用 HTTPS 协议传输。



# REDIS



## Redis 为何单线程

Redis 是单进程单线程的模型，因为 Redis 完全是基于内存的操作，CPU 不是 Redis 的瓶颈，Redis 的瓶颈最有可能是机器内存的大小或者网络带宽。单线程容易实现，而且 CPU 不会成为瓶颈，那就顺理成章的采用单线程的方案了（毕竟采用多线程会有很多麻烦）。

## Redis 和 Memcached 的区别

- **存储方式上：**Memcache 会把数据全部存在内存之中，断电后会挂掉，数据不能超过内存大小。Redis 有部分数据存在硬盘上，这样能保证数据的持久性。
- **数据支持类型上：**Memcache 对数据类型的支持简单，只支持简单的 key-value，而 Redis 支持五种数据类型。
- **使用底层模型不同：**它们之间底层实现方式以及与客户端之间通信的应用协议不一样。Redis 直接自己构建了 VM 机制，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求。
- **Value 的大小：**Redis 可以达到 1GB，而 Memcache 只有 1MB。

## Redis 为何这么快

- Redis 完全基于内存，绝大部分请求是纯粹的内存操作，非常迅速，数据存在内存中。

- 数据结构简单，对数据操作也简单。

- 采用单线程，避免了不必要的上下文切换和竞争条件，不存在多线程导致的 CPU 切换，不用去考虑各种锁的问题，不存在加锁释放锁操作，没有死锁问题导致的性能消耗。

- 使用多路复用 IO 模型，非阻塞 IO。

  

## Redis 的淘汰策略

![image-20200623114805276](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200623114805276.png)



## Redis 的持久化机制了解吗？

Redis 为了保证效率，数据缓存在了内存中，但是会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件中，以保证数据的持久化。

Redis 的持久化策略有两种：

- **RDB：**快照形式是直接把内存中的数据保存到一个 dump 的文件中，定时保存，保存策略。

- **AOF：**把所有的对 Redis 的服务器进行修改的命令都存到一个文件里，命令的集合。

  Redis 默认是快照 RDB 的持久化方式。

当 Redis 重启的时候，它会优先使用 AOF 文件来还原数据集，因为 AOF 文件保存的数据集通常比 RDB 文件所保存的数据集更完整。你甚至可以关闭持久化功能，让数据只在服务器运行时存。

## RDB 是怎么工作的？

Redis 是会以快照"RDB"的形式将数据持久化到磁盘的一个二进制文件 dump.rdb。

工作原理：当 Redis 需要做持久化时，Redis 会 fork 一个子进程，子进程将数据写到磁盘上一个临时 RDB 文件中。当子进程完成写临时文件后，将原来的 RDB 替换掉，这样的好处是可以 copy-on-write。

优点：这种文件非常适合用于备份：比如，可以在最近的 24 小时内，每小时备份一次，并且在每个月的每一天也备份一个 RDB 文件。

这样的话，即使遇上问题，也可以随时将数据集还原到不同的版本。RDB 非常适合灾难恢复。

缺点：可能丢失数据。

### save&BGSave

https://draveness.me/whys-the-design-redis-bgsave-fork/

其中 `SAVE` 命令在执行时会直接阻塞当前的线程，由于 Redis 是 [单线程](https://draveness.me/whys-the-design-redis-single-thread) 的，所以 `SAVE` 命令会直接阻塞来自客户端的所有其他请求，这在很多时候对于需要提供较强可用性保证的 Redis 服务都是无法接受的。我们往往需要 `BGSAVE` 命令在后台生成 Redis 全部数据对应的 RDB 文件，当我们使用 `BGSAVE` 命令时，Redis 会立刻 `fork` 出一个子进程，子进程会执行『将内存中的数据以 RDB 格式保存到磁盘中』这一过程，而 Redis 服务在 `BGSAVE` 工作期间仍然可以处理来自客户端的请求。

## Redis 为什么在使用 RDB 进行快照时会通过子进程的方式进行实现：

1. 通过 `fork` 创建的子进程能够获得和父进程完全相同的内存空间，父进程对内存的修改对于子进程是不可见的，两者不会相互影响；
2. 通过 `fork` 创建子进程时不会立刻触发大量内存的拷贝，内存在被修改时会以页为单位进行拷贝，这也就避免了大量拷贝内存而带来的性能问题；



## AOF 怎么工作

```java
appendfsync yes   
appendfsync always     #每次有数据修改发生时都会写入AOF文件。
appendfsync everysec   #每秒钟同步一次，该策略为AOF的缺省策略。
```

AOF 可以做到全程持久化，只需要在配置中开启 appendonly yes。这样 Redis 每执行一个修改数据的命令，都会把它添加到 AOF 文件中，当 Redis 重启时，将会读取 AOF 文件进行重放，恢复到 Redis 关闭前的最后时刻。

优点： 让 Redis 变得非常耐久。可以设置不同的 Fsync 策略，AOF的默认策略是每秒钟 Fsync 一次，在这种配置下，就算发生故障停机，也最多丢失一秒钟的数据。

缺点：相同的数据集来说，AOF 的文件体积通常要大于 RDB 文件的体积。根据所使用的 Fsync 策略，AOF 的速度可能会慢于 RDB。

https://www.cnblogs.com/cc11001100/p/6487158.html

so 

如果你非常关心你的数据，但仍然可以承受数分钟内的数据丢失，那么可以只使用 RDB 持久化。

AOF 将 Redis 执行的每一条命令追加到磁盘中，处理巨大的写入会降低Redis的性能。

数据库备份和灾难恢复：定时生成 RDB 快照非常便于进行数据库备份，并且 RDB 恢复数据集的速度也要比 AOF 恢复的速度快。当然了，Redis 支持同时开启 RDB 和 AOF，系统重启后，Redis 会优先使用 AOF 来恢复数据，这样丢失的数据会最少。

AOF 后台重写

当子进程在执行 AOF 重写时， 主进程需要执行以下三个工作：

1. 处理命令请求。
2. 将写命令追加到现有的 AOF 文件中。
3. 将写命令追加到 AOF 重写缓存中。

这样一来可以保证：

1. 现有的 AOF 功能会继续执行，即使在 AOF 重写期间发生停机，也不会有任何数据丢失。
2. 所有对数据库进行修改的命令都会被记录到 AOF 重写缓存中。

## Redis 雪崩了解吗?

同一时间大面积失效，瞬间 Redis 跟没有一样.

Solution: 在批量往 Redis 存数据的时候，把每个 Key 的失效时间都加个随机值就好了，这样可以保证数据不会再同一时间大面积失效。

缓存雪崩的事前事中事后的解决方案如下。

- 事前：redis 高可用，主从+哨兵，redis cluster，避免全盘崩溃。
- 事中：本地 ehcache 缓存 + hystrix 限流&降级，避免 MySQL 被打死。
- 事后：redis 持久化，一旦重启，自动从磁盘上加载数据，快速恢复缓存数据。



## 缓存穿透和击穿?

缓存穿透是指缓存和数据库中都没有的数据，而用户（黑客）不断发起请求。

example : id 都是从 1 自增的，如果发起 id=-1 的数据或者 id 特别大不存在的数据，这样的不断攻击导致数据库压力很大，严重会击垮数据库。

solution : 在接口层增加校验，比如用户鉴权，参数做校验，不合法的校验直接 return，比如 id 做基础校验，id<=0 直接拦截。布隆过滤器（Bloom Filter）也能很好的预防缓存穿透的发生。

缓存击穿指一个 Key 非常热点，在不停地扛着大量的请求，大并发集中对这一个点进行访问，当这个 Key 在失效的瞬间，持续的大并发直接落到了数据库上，就在这个 Key 的点上击穿了缓存。

solution ： 设置热点数据永不过期，或者加上互斥锁就搞定了

![image-20200710000746609](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200710000746609.png)



## REDIS   HA 

持久化、复制、哨兵和集群，其主要作用和解决的问题是：

- 持久化：持久化是最简单的高可用方法(有时甚至不被归为高可用的手段)，主要作用是数据备份，即将数据存储在硬盘，保证数据不会因进程退出而丢失。
- 复制：复制是高可用Redis的基础，哨兵和集群都是在复制基础上实现高可用的。复制主要实现了数据的多机备份，以及对于读操作的负载均衡和简单的故障恢复。缺陷：故障恢复无法自动化；写操作无法负载均衡；存储能力受到单机的限制。
- 哨兵：在复制的基础上，哨兵实现了自动化的故障恢复。缺陷：写操作无法负载均衡；存储能力受到单机的限制。
- 集群：通过集群，Redis解决了写操作无法负载均衡，以及存储能力受到单机限制的问题，实现了较为完善的高可用方案。

### 主从

#### 主从复制过程/作用？ https://zhuanlan.zhihu.com/p/60239657

复制过程：                                      

- 从节点执行 slaveof [masterIP] [masterPort]，保存主节点信息。
- 从节点中的定时任务发现主节点信息，建立和主节点的 Socket 连接。
- 从节点发送 Ping 信号，主节点返回 Pong，两边能互相通信。
- 连接建立后，主节点将所有数据发送给从节点（数据同步）。
- 主节点把当前的数据同步给从节点后，便完成了复制的建立过程。接下来，主节点就会持续的把写命令发送给从节点，保证主从数据一致性。
- ![image-20200623151253153](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200623151253153.png)

#### **psync 命令需要 3 个组件支持：**

1.主从节点各自复制偏移量
2.主节点复制积压缓冲区
3.主节点运行 ID

#### 主从复制会存在以下问题：

- 一旦主节点宕机，从节点晋升为主节点，同时需要修改应用方的主节点地址，还需要命令所有从节点去复制新的主节点，整个过程需要人工干预。

- 主节点的写能力受到单机的限制。

- 主节点的存储能力受到单机的限制。

- 原生复制的弊端在早期的版本中也会比较突出，比如：Redis 复制中断后，从节点会发起 psync。
  此时如果同步不成功，则会进行全量同步，主库执行全量备份的同时，可能会造成毫秒或秒级的卡顿。

  

  #### **主从复制的作用**

  主从复制的作用主要包括：

  1. 数据冗余：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。
  2. 故障恢复：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复；实际上是一种服务的冗余。
  3. 负载均衡：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务（即写Redis数据时应用连接主节点，读Redis数据时应用连接从节点），分担服务器负载；尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高Redis服务器的并发量。
  4. 高可用基石：除了上述作用以外，主从复制还是哨兵和集群能够实施的基础，因此说主从复制是Redis高可用的基础。

  

## Redis 的过期策略是如何实现的？

redisDb 结构的 expire 字典（过期字典）保存了所有键的过期时间

过期字典的键是一个指向键空间中的某个键对象的指针

过期字典的值保存了键所指向的数据库键的过期时间

![image-20200728015102720](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200728015102720.png)

### 哨兵

#### 哨兵有哪些功能

Redis Sentinel（哨兵）主要功能包括**主节点存活检测、主从运行情况检测、自动故障转移、主从切换**。Redis Sentinel 最小配置是一主一从。Redis 的 Sentinel 系统可以用来管理多个 Redis 服务器。

该系统可以执行以下四个任务：

- **监控：**不断检查主服务器和从服务器是否正常运行。

- **通知：**当被监控的某个 Redis 服务器出现问题，Sentinel 通过 API 脚本向管理员或者其他应用程序发出通知。

- **自动故障转移：**当主节点不能正常工作时，Sentinel 会开始一次自动的故障转移操作，它会将与失效主节点是主从关系的其中一个从节点升级为新的主节点，并且将其他的从节点指向新的主节点，这样人工干预就可以免了。

- **配置提供者：**在 Redis Sentinel 模式下，客户端应用在初始化时连接的是 Sentinel 节点集合，从中获取主节点的信息。

  

#### 哨兵工作原理 https://juejin.im/post/5b7d226a6fb9a01a1e01ff64

![image-20200623200842696](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200623200842696.png)

哨兵系统的搭建过程，有几点需要注意：

（1）哨兵系统中的主从节点，与普通的主从节点并没有什么区别，故障发现和转移是由哨兵来控制和完成的。

（2）哨兵节点本质上是redis节点。

（3）每个哨兵节点，只需要配置监控主节点，便可以自动发现其他的哨兵节点和从节点。

（4）在哨兵节点启动和故障转移阶段，各个节点的配置文件会被重写(config rewrite)。



##### 哨兵节点支持的命令

哨兵节点作为运行在特殊模式下的redis节点，其支持的命令与普通的redis节点不同。在运维中，我们可以通过这些命令查询或修改哨兵系统；不过更重要的是，哨兵系统要实现故障发现、故障转移等各种功能，离不开哨兵节点之间的通信，而通信的很大一部分是通过哨兵节点支持的命令来实现的。下面介绍哨兵节点支持的主要命令。

（1）基础查询：通过这些命令，可以查询哨兵系统的拓扑结构、节点信息、配置信息等。

- info sentinel：获取监控的所有主节点的基本信息
- sentinel masters：获取监控的所有主节点的详细信息
- sentinel master mymaster：获取监控的主节点mymaster的详细信息
- sentinel slaves mymaster：获取监控的主节点mymaster的从节点的详细信息
- sentinel sentinels mymaster：获取监控的主节点mymaster的哨兵节点的详细信息
- sentinel get-master-addr-by-name mymaster：获取监控的主节点mymaster的地址信息，前文已有介绍
- sentinel is-master-down-by-addr：哨兵节点之间可以通过该命令询问主节点是否下线，从而对是否客观下线做出判断

（2）增加/移除对主节点的监控

sentinel monitor mymaster2 192.168.92.128 16379 2：与部署哨兵节点时配置文件中的sentinel monitor功能完全一样，不再详述

sentinel remove mymaster2：取消当前哨兵节点对主节点mymaster2的监控

（3）强制故障转移

sentinel failover mymaster：该命令可以**强制对****mymaster****执行故障转移，**即便当前的主节点运行完好；例如，如果当前主节点所在机器即将报废，便可以提前通过failover命令进行故障转移。

## Redis 集群

![image-20200718220229933](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200718220229933.png)

这里要先将节点握手讲清楚。我们让两个redis节点之间进行通信的时候，需要在客户端执行下面一个命令

```
127.0.0.1:7000>cluster meet 127.0.0.1:7001
```

如下图所示
![img](https://img2018.cnblogs.com/blog/725429/201908/725429-20190829164714480-362698612.png)

意思很简单，让7000节点和7001节点知道彼此存在！
在握手成功后，连个节点之间会**定期**发送ping/pong消息，交换**数据信息**，如下图所示。
![img](https://img2018.cnblogs.com/blog/725429/201908/725429-20190829164722650-2064735559.png)

在这里，我们需要关注三个重点。

- (1)交换什么数据信息
- (2)数据信息究竟多大
- (3)定期的频率什么样

*到底在交换什么数据信息？*
交换的数据信息，由消息体和消息头组成。
消息体无外乎是一些节点标识啊，IP啊，端口号啊，发送时间啊。
我们来看消息头，结构如下
![img](https://img2018.cnblogs.com/blog/725429/201908/725429-20190829164739879-973731722.jpg)

注意看红框的内容，type表示消息类型。
另外，消息头里面有个myslots的char数组，长度为16383/8，这其实是一个bitmap,每一个位代表一个槽，如果该位为1，表示这个槽是属于这个节点的。

**Failover的流程**

一、主观下线

集群中每个节点都会定期向其他节点发送ping消息，接收节点回复pong消息作为响应。如果在cluster-node-timeout时间内通信一直失败，则发送节点会认为接收节点存在故障，把接收节点标记为主观下线（pfail）状态。


二、客观下线

当某个节点判断另一个节点主观下线后，相应的节点状态会跟随消息在集群内传播。通过Gossip消息传播，集群内节点不断收集到故障节点的下线报告。当半数以上持有槽的主节点都标记某个节点是主观下线时，触发客观下线流程。

集群中的节点每次接收到其他节点的pfail状态，都会尝试触发客观下线，流程说明：

\1. 首先统计有效的下线报告数量，如果小于集群内持有槽的主节点总数的一半则退出。

\2. 当下线报告大于槽主节点数量一半时，标记对应故障节点为客观下线状态。

\3. 向集群广播一条fail消息，通知所有的节点将故障节点标记为客观下线，fail消息的消息体只包含故障节点的ID。广播fail消息是客观下线的最后一步，它承担着非常重要的职责：

​	\1. 通知集群内所有的节点标记故障节点为客观下线状态并立刻生效。

​	\2. 通知故障节点的从节点触发故障转移流程。 

故障切换

故障节点变为客观下线后，如果下线节点是持有槽的主节点则需要在它的从节点中选出一个替换它，从而保证集群的高可用。下线主节点的所有从节点承担故障恢复的义务，当从节点通过内部定时任务发现自身复制的主节点进入客观下线时，将会触发故障切换流程。

1.资格检查

每个从节点都要检查最后与主节点断线时间，判断是否有资格替换故障的主节点。如果从节点与主节点断线时间超过cluster-node-time*cluster-slave-validity-factor，则当前从节点不具备故障转移资格。参数cluster-slavevalidity-factor用于从节点的有效因子，默认为10。

2.准备选举时间

当从节点符合故障切换资格后，更新触发切换选举的时间，只有到达该时间后才能执行后续流程。

这里之所以采用延迟触发机制，主要是通过对多个从节点使用不同的延迟选举时间来支持优先级问题。复制偏移量越大说明从节点延迟越低，那么它应该具有更高的优先级来替换故障主节点。

3.发起选举

当从节点定时任务检测到达故障选举时间（failover_auth_time）到达后，发起选举流程如下：

1> 更新配置纪元

2> 广播选举消息

在集群内广播选举消息（FAILOVER_AUTH_REQUEST），并记录已发送过消息的状态，保证该从节点在一个配置纪元内只能发起一次选举。

4.选举投票

只有持有槽的主节点才会处理故障选举消息（FAILOVER_AUTH_REQUEST），因为每个持有槽的节点在一个配置纪元内都有唯一的一张选票，当接到第一个请求投票的从节点消息时回复FAILOVER_AUTH_ACK消息作为投票，之后相同配置纪元内其他从节点的选举消息将忽略。

Redis集群没有直接使用从节点进行领导者选举，主要因为从节点数必须大于等于3个才能保证凑够N/2+1个节点，将导致从节点资源浪费。使用集群内所有持有槽的主节点进行领导者选举，即使只有一个从节点也可以完成选举过程。

5.替换主节点

当从节点收集到足够的选票之后，触发替换主节点操作：

1> 当前从节点取消复制变为主节点。

2> 执行clusterDelSlot操作撤销故障主节点负责的槽，并执行clusterAddSlot把这些槽委派给自己。

3> 向集群广播自己的pong消息，通知集群内所有的节点当前从节点变为主节点并接管了故障主节点的槽信息。

## 布隆过滤器原理？

利用高效的数据结构和算法快速判断出你这个 Key 是否在数据库中存在，不存在你 return 就好了，存在你就去查 DB 刷新 KV 再 return。

### 场景

- 数据库防止穿库。 Google Bigtable，HBase 和 Cassandra 以及 Postgresql 使用BloomFilter来减少不存在的行或列的磁盘查找。避免代价高昂的磁盘查找会大大提高数据库查询操作的性能。
- 业务场景中判断用户是否阅读过某视频或文章，比如抖音或头条，当然会导致一定的误判，但不会让用户看到重复的内容。还有之前自己遇到的一个比赛类的社交场景中，需要判断用户是否在比赛中，如果在则需要更新比赛内容，也可以使用布隆过滤器，可以减少不在的用户查询db或缓存的次数。
- 缓存宕机、缓存击穿场景，一般判断用户是否在缓存中，如果在则直接返回结果，不在则查询db，如果来一波冷数据，会导致缓存大量击穿，造成雪崩效应，这时候可以用布隆过滤器当缓存的索引，只有在布隆过滤器中，才去查询缓存，如果没查询到，则穿透到db。如果不在布隆器中，则直接返回。
- WEB拦截器，如果相同请求则拦截，防止重复被攻击。用户第一次请求，将请求参数放入布隆过滤器中，当第二次请求时，先判断请求参数是否被布隆过滤器命中。可以提高缓存命中率。



redis对事务是部分支持的，如果是在入队时报错，那么都不会执行；在非入队时报错，那么成功的就会成功执行。



## redis事务三大特性：

单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。
没有隔离级别的概念：队列中的命令没有提交之前都不会实际的被执行，因为事务提交前任何指令都不会被实际执行，也就不存在”事务内的查询要看到事务里的更新，在事务外查询不能看到”这个让人万分头痛的问题
不保证原子性：redis同一个事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚

## 数据类型

### sds

- Redis 的字符串表示为 `sds` ，而不是 C 字符串（以 `\0` 结尾的 `char*`）。

- 对比 C 字符串，sds有以下特性：

  - 可以高效地执行长度计算（`strlen`）；
  - 可以高效地执行追加操作（`append`）；
  - 二进制安全；

- `sds` 会为追加操作进行优化：加快追加操作的速度，并降低内存分配的次数，代价是多占用了一些内存，而且这些内存不会被主动释放

  

### 双端链表

Redis 实现了自己的双端链表结构。双端链表还是 Redis 列表类型的底层实现之一， 当对列表类型的键进行操作 —— 比如执行 [RPUSH](http://redis.readthedocs.org/en/latest/list/rpush.html#rpush) 、 [LPOP](http://redis.readthedocs.org/en/latest/list/lpop.html#lpop) 或 [LLEN](http://redis.readthedocs.org/en/latest/list/llen.html#llen) 等命令时， 程序在底层操作的可能就是双端链表。

双端链表主要有两个作用：

- 作为 Redis 列表类型的底层实现之一；
- 作为通用数据结构，被其他功能模块所使用；

双端链表及其节点的性能特性如下：

- 节点带有前驱和后继指针，访问前驱节点和后继节点的复杂度为 O(1) ，并且对链表的迭代可以在从表头到表尾和从表尾到表头两个方向进行；

- 链表带有指向表头和表尾的指针，因此对表头和表尾进行处理的复杂度为 O(1) ；

- 链表带有记录节点数量的属性，所以可以在 O(1) 复杂度内返回链表的节点数量（长度）；

  

### dict

 大部分针对数据库的命令， 比如 [DBSIZE](http://redis.readthedocs.org/en/latest/server/dbsize.html#dbsize) 、 [FLUSHDB](http://redis.readthedocs.org/en/latest/server/flushdb.html#flushdb) 、 [RANDOMKEY](http://redis.readthedocs.org/en/latest/key/randomkey.html#randomkey) ， 等等， 都是构建于对字典的操作之上的； 而那些创建、更新、删除和查找键值对的命令， 也无一例外地需要在键空间上进行操作。dict也用作 Hash 类型键的底层实现之一；

- 字典是由键值对构成的抽象数据结构。

- Redis 中的数据库和哈希键都基于字典来实现。

- Redis 字典的底层实现为哈希表，每个字典使用两个哈希表，一般情况下只使用 0 号哈希表，只有在 rehash 进行时，才会同时使用 0 号和 1 号哈希表。

- 哈希表使用链地址法来解决键冲突的问题。

- Rehash 可以用于扩展或收缩哈希表。

- 对哈希表的 rehash 是分多次、渐进式地进行的。

- 字典的 rehash 操作实际上就是执行以下任务：

  1. 创建一个比 `ht[0]->table` 更大的 `ht[1]->table` ；
  2. 将 `ht[0]->table` 中的所有键值对迁移到 `ht[1]->table` ；
  3. 将原有 `ht[0]` 的数据清空，并将 `ht[1]` 替换为新的 `ht[0]` ；

  

### 跳跃表

- 跳跃表是一种随机化数据结构，查找、添加、删除操作都可以在对数期望时间下完成。
- 跳跃表目前在 Redis 的唯一作用，就是作为有序集类型的底层数据结构（之一，另一个构成有序集的结构是字典）。
- 为了满足自身的需求，Redis 基于 William Pugh 论文中描述的跳跃表进行了修改，包括：
  1. `score` 值可重复。
  2. 对比一个元素需要同时检查它的 `score` 和 `memeber` 。
  3. 每个节点带有高度为 1 层的后退指针，用于从表尾方向向表头方向迭代。

### 整数集合（intset）

* Intset 用于有序、无重复地保存多个整数值，会根据元素的值，自动选择该用什么长度的整数类型来保存元素。

* 当一个位长度更长的整数值添加到 intset 时，需要对 intset 进行升级，新 intset 中每个元素的位长度，会等于新添加值的位长度，但原有元素的值不变。

* 升级会引起整个 intset 进行内存重分配，并移动集合中的所有元素，这个操作的复杂度为 O(N)O(N) 。

* Intset 只支持升级，不支持降级。

* Intset 是有序的，程序使用二分查找算法来实现查找操作，复杂度为 O(lgN)O(lg⁡N) 。



### Ziplist

因为 ziplist 节约内存的性质， 哈希键、列表键和有序集合键初始化的底层实现皆采用 ziplist.

ziplist 是由一系列特殊编码的内存块构成的列表，可以保存字符数组或整数值，同时是哈希键、列表键和有序集合键的底层实现之一。



![image-20200708201423179](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200708201423179.png)

为什么Hash与List会使用ziplist来存储数据呢？

因为

1. ziplist会比hashtable与ziplist节省跟多的内存
2. 内存中以连续块方式保存的数据比起hashtable与linkedlist使用的链表可以更快的载入缓存中
3. 当ziplist的长度比较小的时候，从ziplist读写数据的效率比hashtable或者linkedlist的差异并不大。

本质上，使用ziplist就是以时间换空间的一种优化，但是他的时间损坏小到几乎可以忽略不计，但却能带来可观的内存减少，所以满足条件时，Redis会使用ziplist作为Hash与List的存储结构。
链接：https://www.jianshu.com/p/fc9d65b5bb9a

### RedisObject

![image-20200708200804376](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200708200804376.png)

### watch

![image-20200709205220653](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200709205220653.png)

通过 `watched_keys` 字典， 如果程序想检查某个键是否被监视， 那么它只要检查字典中是否存在这个键即可； 如果程序要获取监视某个键的所有客户端， 那么只要取出键的值（一个链表）， 然后对链表进行遍历即可。

在任何对数据库键空间（key space）进行修改的命令成功执行之后 （比如 [FLUSHDB](http://redis.readthedocs.org/en/latest/server/flushdb.html#flushdb) 、 [SET](http://redis.readthedocs.org/en/latest/string/set.html#set) 、 [DEL](http://redis.readthedocs.org/en/latest/key/del.html#del) 、 [LPUSH](http://redis.readthedocs.org/en/latest/list/lpush.html#lpush) 、 [SADD](http://redis.readthedocs.org/en/latest/set/sadd.html#sadd) 、 [ZREM](http://redis.readthedocs.org/en/latest/sorted_set/zrem.html#zrem) ，诸如此类）， `multi.c/touchWatchedKey` 函数都会被调用 —— 它检查数据库的 `watched_keys` 字典， 看是否有客户端在监视已经被命令修改的键， 如果有的话， 程序将所有监视这个/这些被修改键的客户端的 `REDIS_DIRTY_CAS` 选项打开

当客户端发送 [EXEC](http://redis.readthedocs.org/en/latest/transaction/exec.html#exec) 命令、触发事务执行时， 服务器会对客户端的状态进行检查：

- 如果客户端的 `REDIS_DIRTY_CAS` 选项已经被打开，那么说明被客户端监视的键至少有一个已经被修改了，事务的安全性已经被破坏。服务器会放弃执行这个事务，直接向客户端返回空回复，表示事务执行失败。
- 如果 `REDIS_DIRTY_CAS` 选项没有被打开，那么说明所有监视键都安全，服务器正式执行事务。

最后，当一个客户端结束它的事务时，无论事务是成功执行，还是失败， `watched_keys` 字典中和这个客户端相关的资料都会被清除。



## 事务

Redis 事务保证了其中的一致性（C）和隔离性（I），但并不保证原子性（A）和持久性（D）。

A：

单个 Redis 命令的执行是原子性的，但 Redis 没有在事务上增加任何维持原子性的机制，所以 Redis 事务的执行并不是原子性的。如果一个事务队列中的所有命令都被成功地执行，那么称这个事务执行成功。另一方面，如果 Redis 服务器进程在执行事务的过程中被停止 —— 比如接到 KILL 信号、宿主机器停机，等等，那么事务执行失败。当事务失败时，Redis 也不会进行任何的重试或者回滚动作。

C:

* 入队错误:带有不正确入队命令的事务不会被执行，也不会影响数据库的一致性。

* 执行的过程中发生错误 : 比如说，对一个不同类型的 key 执行了错误的操作， 那么 Redis 只会将错误包含在事务的结果中， 这不会引起事务中断或整个失败，不会影响已执行事务命令的结果，也不会影响后面要执行的事务命令， 所以它对事务的一致性也没有影响。

  

* Redis 进程被终结： 如果 Redis 服务器进程在执行事务的过程中被其他进程终结，或者被管理员强制杀死，那么根据 Redis 所使用的持久化模式，可能有以下情况出现：

  - 内存模式：如果 Redis 没有采取任何持久化机制，那么重启之后的数据库总是空白的，所以数据总是一致的。

  - RDB 模式：在执行事务时，Redis 不会中断事务去执行保存 RDB 的工作，只有在事务执行之后，保存 RDB 的工作才有可能开始。所以当 RDB 模式下的 Redis 服务器进程在事务中途被杀死时，事务内执行的命令，不管成功了多少，都不会被保存到 RDB 文件里。恢复数据库需要使用现有的 RDB 文件，而这个 RDB 文件的数据保存的是最近一次的数据库快照（snapshot），所以它的数据可能不是最新的，但只要 RDB 文件本身没有因为其他问题而出错，那么还原后的数据库就是一致的。

  - AOF 模式：因为保存 AOF 文件的工作在后台线程进行，所以即使是在事务执行的中途，保存 AOF 文件的工作也可以继续进行，因此，根据事务语句是否被写入并保存到 AOF 文件，有以下两种情况发生：

    1）如果事务语句未写入到 AOF 文件，或 AOF 未被 SYNC 调用保存到磁盘，那么当进程被杀死之后，Redis 可以根据最近一次成功保存到磁盘的 AOF 文件来还原数据库，只要 AOF 文件本身没有因为其他问题而出错，那么还原后的数据库总是一致的，但其中的数据不一定是最新的。

    2）如果事务的部分语句被写入到 AOF 文件，并且 AOF 文件被成功保存，那么不完整的事务执行信息就会遗留在 AOF 文件里，当重启 Redis 时，程序会检测到 AOF 文件并不完整，Redis 会退出，并报告错误。需要使用 redis-check-aof 工具将部分成功的事务命令移除之后，才能再次启动服务器。还原之后的数据总是一致的，而且数据也是最新的（直到事务执行之前为止）。

    

## 订阅与发布模式

![image-20200709211009546](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200709211009546.png)

- 订阅信息由服务器进程维持的 `redisServer.pubsub_channels` 字典保存，字典的键为被订阅的频道，字典的值为订阅频道的所有客户端。
- 当有新消息发送到频道时，程序遍历频道（键）所对应的（值）所有客户端，然后将消息发送到所有订阅频道的客户端上。
- 订阅模式的信息由服务器进程维持的 `redisServer.pubsub_patterns` 链表保存，链表的每个节点都保存着一个 `pubsubPattern` 结构，结构中保存着被订阅的模式，以及订阅该模式的客户端。程序通过遍历链表来查找某个频道是否和某个模式匹配。
- 当有新消息发送到频道时，除了订阅频道的客户端会收到消息之外，所有订阅了匹配频道的模式的客户端，也同样会收到消息。
- 退订频道和退订模式分别是订阅频道和订阅模式的反操作。



## LUA 脚本

- 初始化 Lua 脚本环境需要一系列步骤，其中最重要的包括：
  - 创建 Lua 环境。
  - 载入 Lua 库，比如字符串库、数学库、表格库，等等。
  - 创建 `redis` 全局表格，包含各种对 Redis 进行操作的函数，比如 `redis.call` 和 `redis.log` ，等等。
  - 创建一个无网络连接的伪客户端，专门用于执行 Lua 脚本中的 Redis 命令。
- Reids 通过一系列措施保证被执行的 Lua 脚本无副作用，也没有有害的写随机性：对于同样的输入参数和数据集，总是产生相同的写入命令。
- [EVAL](http://redis.readthedocs.org/en/latest/script/eval.html#eval) 命令为输入脚本定义一个 Lua 函数，然后通过执行这个函数来执行脚本。
- [EVALSHA](http://redis.readthedocs.org/en/latest/script/evalsha.html#evalsha) 通过构建函数名，直接调用 Lua 中已定义的函数，从而执行相应的脚本。

## Redis 慢查询

- 针对慢查询日志有三种操作，分别是查看、清空和获取日志数量：
  - 查看日志：在日志链表中遍历指定数量的日志节点，复杂度为 O(N)O(N) 。
  - 清空日志：释放日志链表中的所有日志节点，复杂度为 O(N)O(N) 。
  - 获取日志数量：获取日志的数量等同于获取 `server.slowlog` 链表的数量，复杂度为 O(1)O(1) 。
- Redis 用一个链表以 FIFO 的顺序保存着所有慢查询日志。
- 每条慢查询日志以一个慢查询节点表示，节点中记录着执行超时的命令、命令的参数、命令执行时的时间，以及执行命令所消耗的时间等信息。

## REDIS 数据库

因为数据库本身是一个字典， 所以对数据库的操作基本上都是对字典的操作， 加上以下一些维护操作：

- 更新键的命中率和不命中率，这个值可以用 [INFO](http://redis.readthedocs.org/en/latest/server/info.html#info) 命令查看；

- 更新键的 LRU 时间，这个值可以用 [OBJECT](http://redis.readthedocs.org/en/latest/key/object.html#object) 命令来查看；

- 删除过期键（稍后会详细说明）；

- 如果键被修改了的话，那么将键设为脏（用于事务监视），并将服务器设为脏（等待 RDB 保存）；

- 将对键的修改发送到 AOF 文件和附属节点，保持数据库状态的一致；

  

  在数据库中， 所有键的过期时间都被保存在 `redisDb` 结构的 `expires` 字典里：

  通过 `expires` 字典， 可以用以下步骤检查某个键是否过期：

  1. 检查键是否存在于 `expires` 字典：如果存在，那么取出键的过期时间；
  2. 检查当前 UNIX 时间戳是否大于键的过期时间：如果是的话，那么键已经过期；否则，键未过期。

  

  我们知道了过期时间保存在 `expires` 字典里， 又知道了该如何判定一个键是否过期， 现在剩下的问题是， 如果一个键是过期的， 那它什么时候会被删除？

  1. 定时删除：在设置键的过期时间时，创建一个定时事件，当过期时间到达时，由事件处理器自动执行键的删除操作。对内存是最友好的,对 CPU 时间是最不友好的,  目前 Redis 事件处理器对时间事件的实现方式 —— 无序链表， 查找一个时间复杂度为 O(N)O(N) —— 并不适合用来处理大量时间事件。
  2. 惰性删除：放任键过期不管，但是在每次从 `dict` 字典中取出键值时，要检查键是否过期，如果过期的话，就删除它，并返回空；如果没过期，就返回键值。对 CPU 时间来说是最友好的.
  3. 定期删除：每隔一段时间，对 `expires` 字典进行检查，删除里面的过期键.定期删除是这两种策略的一种折中：
     - 它每隔一段时间执行一次删除操作，并通过限制删除操作执行的时长和频率，籍此来减少删除操作对 CPU 时间的影响。
     - 另一方面，通过定期删除过期键，它有效地减少了因惰性删除而带来的内存浪费。

Redis 使用的过期键删除策略是**惰性删除加上定期删除**， 这两个策略相互配合，可以很好地在合理利用 CPU 时间和节约内存空间之间取得平衡。



## redis TTL实现原理

### TTL存储的数据结构

 redis针对TTL时间有专门的dict进行存储，就是redisDb当中的dict *expires字段，dict顾名思义就是一个hashtable，key为对应的rediskey，value为对应的TTL时间。
链接：https://www.jianshu.com/p/53083f5f2ddc
https://juejin.im/post/6844903965927145479

![image-20200804213323189](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200804213323189.png)

>  图中过期字段和键空间中键对象有重复，实际中不会出现重复对象，键空间的键和过期字典的键都指向同一个键对象

## 过期键会被保存在更新后的 RDB 文件、 AOF 文件或者重写后的 AOF 文件里面吗？

### 更新后的 RDB 文件

在创建新的 RDB 文件时，程序会对键进行检查，过期的键不会被写入到更新后的 RDB 文件中。因此，过期键对更新后的 RDB 文件没有影响。

### AOF 文件

在键已经过期，但是还没有被惰性删除或者定期删除之前，这个键不会产生任何影响，AOF 文件也不会因为这个键而被修改。当过期键被惰性删除、或者定期删除之后，程序会向 AOF 文件追加一条 `DEL` 命令，来显式地记录该键已被删除。

### AOF 重写

和 RDB 文件类似， 当进行 AOF 重写时， 程序会对键进行检查， 过期的键不会被保存到重写后的 AOF 文件。因此，过期键对重写后的 AOF 文件没有影响。

### 复制

当服务器带有附属节点时， 过期键的删除由主节点统一控制：

- 如果服务器是主节点，那么它在删除一个过期键之后，会显式地向所有附属节点发送一个 `DEL` 命令。
- 如果服务器是附属节点，那么当它碰到一个过期键的时候，它会向程序返回键已过期的回复，但并不真正的删除过期键。因为程序只根据键是否已经过期、而不是键是否已经被删除来决定执行流程，所以这种处理并不影响命令的正确执行结果。当接到从主节点发来的 `DEL` 命令之后，附属节点才会真正的将过期键删除掉。

附属节点不自主对键进行删除是为了和主节点的数据保持绝对一致， 因为这个原因， 当一个过期键还存在于主节点时，这个键在所有附属节点的副本也不会被删除。这种处理机制对那些使用大量附属节点，并且带有大量过期键的应用来说，可能会造成一部分内存不能立即被释放，但是，因为过期键通常很快会被主节点发现并删除，所以这实际上也算不上什么大问题。

- 数据库主要由 `dict` 和 `expires` 两个字典构成，其中 `dict` 保存键值对，而 `expires` 则保存键的过期时间。
- 数据库的键总是一个字符串对象，而值可以是任意一种 Redis 数据类型，包括字符串、哈希、集合、列表和有序集。
- `expires` 的某个键和 `dict` 的某个键共同指向同一个字符串对象，而 `expires` 键的值则是该键以毫秒计算的 UNIX 过期时间戳。
- Redis 使用惰性删除和定期删除两种策略来删除过期的键。
- 更新后的 RDB 文件和重写后的 AOF 文件都不会保留已经过期的键。
- 当一个过期键被删除之后，程序会追加一条新的 `DEL` 命令到现有 AOF 文件末尾。
- 当主节点删除一个过期键之后，它会显式地发送一条 `DEL` 命令到所有附属节点。
- 附属节点即使发现过期键，也不会自作主张地删除它，而是等待主节点发来 `DEL` 命令，这样可以保证主节点和附属节点的数据总是一致的。
- 数据库的 `dict` 字典和 `expires` 字典的扩展策略和普通字典一样。它们的收缩策略是：当节点的填充百分比不足 10% 时，将可用节点数量减少至大于等于当前已用节点数量。



## Redis 分别提供了哪些持久化机制

- RDB 将数据库的快照（snapshot）以二进制的方式保存到磁盘中。
- AOF 则以协议文本的方式，将所有对数据库进行过写入的命令（及其参数）记录到 AOF 文件，以此达到记录数据库状态的目的。

## RDB

- `rdbSave` 会将数据库数据保存到 RDB 文件，并在保存完成之前阻塞调用者。

- [SAVE](http://redis.readthedocs.org/en/latest/server/save.html#save) 命令直接调用 `rdbSave` ，阻塞 Redis 主进程； [BGSAVE](http://redis.readthedocs.org/en/latest/server/bgsave.html#bgsave) 用子进程调用 `rdbSave` ，主进程仍可继续处理命令请求。

- [SAVE](http://redis.readthedocs.org/en/latest/server/save.html#save) 执行期间， AOF 写入可以在后台线程进行， [BGREWRITEAOF](http://redis.readthedocs.org/en/latest/server/bgrewriteaof.html#bgrewriteaof) 可以在子进程进行，所以这三种操作可以同时进行。

- 为了避免产生竞争条件， [BGSAVE](http://redis.readthedocs.org/en/latest/server/bgsave.html#bgsave) 执行时， [SAVE](http://redis.readthedocs.org/en/latest/server/save.html#save) 命令不能执行。

- 为了避免性能问题， [BGSAVE](http://redis.readthedocs.org/en/latest/server/bgsave.html#bgsave) 和 [BGREWRITEAOF](http://redis.readthedocs.org/en/latest/server/bgrewriteaof.html#bgrewriteaof) 不能同时执行。

- 调用 `rdbLoad` 函数载入 RDB 文件时，不能进行任何和数据库相关的操作，不过订阅与发布方面的命令可以正常执行，因为它们和数据库不相关联。

- RDB 文件的组织方式如下：

  ```
  +-------+-------------+-----------+-----------------+-----+-----------+
  | REDIS | RDB-VERSION | SELECT-DB | KEY-VALUE-PAIRS | EOF | CHECK-SUM |
  +-------+-------------+-----------+-----------------+-----+-----------+
  
                        |<-------- DB-DATA ---------->|
  ```

- 键值对在 RDB 文件中的组织方式如下：

  ```
  +----------------------+---------------+-----+-------+
  | OPTIONAL-EXPIRE-TIME | TYPE-OF-VALUE | KEY | VALUE |
  +----------------------+---------------+-----+-------+
  ```

  RDB 文件使用不同的格式来保存不同类型的值。
  
  

## AOF



 AOF 文件的整个过程可以分为三个阶段：

1. 命令传播：Redis 将执行完的命令、命令的参数、命令的参数个数等信息发送到 AOF 程序中。
2. 缓存追加：AOF 程序根据接收到的命令数据，将命令转换为网络通讯协议的格式，然后将协议内容追加到服务器的 AOF 缓存中。
3. 文件写入和保存：AOF 缓存中的内容被写入到 AOF 文件末尾，如果设定的 AOF 保存条件被满足的话， `fsync` 函数或者 `fdatasync`函数会被调用，将写入的内容真正地保存到磁盘中。

Redis 目前支持三种 AOF 保存模式，它们分别是：

1. `AOF_FSYNC_NO` ：不保存，

   在这种模式下， SAVE 只会在以下任意一种情况中被执行：

   - Redis 被关闭
   - AOF 功能被关闭
   - 系统的写缓存被刷新（可能是缓存已经被写满，或者定期保存操作被执行）

2. `AOF_FSYNC_EVERYSEC` ：每一秒钟保存一次。SAVE 原则上每隔一秒钟就会执行一次， 因为 SAVE 操作是由后台子线程调用的， 所以它不会引起服务器主进程阻塞

3. `AOF_FSYNC_ALWAYS` ：每执行一个命令保存一次。

对于三种 AOF 保存模式， 它们对服务器主进程的阻塞情况如下：

1. 不保存（`AOF_FSYNC_NO`）：写入和保存都由主进程执行，两个操作都会阻塞主进程。
2. 每一秒钟保存一次（`AOF_FSYNC_EVERYSEC`）：写入操作由主进程执行，阻塞主进程。保存操作由子线程执行，不直接阻塞主进程，但保存操作完成的快慢会影响写入操作的阻塞时长。
3. 每执行一个命令保存一次（`AOF_FSYNC_ALWAYS`）：和模式 1 一样。安全性是最高的， 但性能也是最差的

- AOF 文件通过保存所有修改数据库的命令来记录数据库的状态。
- AOF 文件中的所有命令都以 Redis 通讯协议的格式保存。
- 不同的 AOF 保存模式对数据的安全性、以及 Redis 的性能有很大的影响。
- AOF 重写的目的是用更小的体积来保存数据库状态，整个重写过程基本上不影响 Redis 主进程处理命令请求。
- AOF 重写是一个有歧义的名字，实际的重写工作是针对数据库的当前值来进行的，程序既不读写、也不使用原有的 AOF 文件。
- AOF 可以由用户手动触发，也可以由服务器自动触发。

REDIS Server

- 服务器经过初始化之后，才能开始接受命令。
- 服务器初始化可以分为六个步骤：
  1. 初始化服务器全局状态。
  2. 载入配置文件。
  3. 创建 daemon 进程。
  4. 初始化服务器功能模块。
  5. 载入数据。
  6. 开始事件循环。
- 服务器为每个已连接的客户端维持一个客户端结构，这个结构保存了这个客户端的所有状态信息。
- 客户端向服务器发送命令，服务器接受命令然后将命令传给命令执行器，执行器执行给定命令的实现函数，执行完成之后，将结果保存在缓存，最后回传给客户端。

## REDIS Hyperlog log



链接：https://juejin.im/post/5d066d11e51d4510a50335bc

场景： 统计一个集合中不重复的元素个数，比如日常需求场景有，统计页面的UV或者统计在线的用户数、注册IP数。。

简单的做法就是记录集合中的所有不重复的 集合S，新来一个元素x，首先判断x在不在S中，如果不在，则将x加入到S，否则不记录。常用的SET数据结构就可以实现。

但是这样实现，如果数据量越来越大，会造成什么问题？

- 当统计的数据量变大时，相应的存储内存会线性增长。
- 当集合S越大时，判断x元素是否在集合S中的所花的成本会越大。



常用的基数计数有三种： B+树、bitmap、概率算法。

- B+ 树。 B+ 树插入和查找效率比较高。可以快速查找元素是否存在，以及进行插入。如果要计算基数值（不重复的元素值），则只需要树的节点个数即可。但是依然存在没有节省内存空间的问题。
- bitmap。 bitmap 是通过一个bit数组来存在特定数据的一种数据结构。基数计数则将每一个元素对应到bit数组的其中一位，比如Bit数组010010101，代表[1,4,6,8]。新加入的元素只需要已有的Bit数组和新加入的元素进行按位或计算。这种方式可以大大减少内存，如果存储1亿数据的话，大概只需要 100000000/8/1024/1024 ≈ 12M 的内存。 相比B+树确实节省不少，但是在某些非常大数据的场景下，如果有10000个对象有1亿数据，则需要120G内存，可以说在特定场景下内存的消耗还是蛮大的。
- 概率算法，概率算法是通过牺牲准确率来换取空间，对于不要求绝对准确率的场景下，概率算法是一种不错的选择，因为概率算法不直接存储数据集合本身，通过一定的概率统计方法预估基数值，同时保证误差在一定范围内，这种方式可以大大减少内存。HyperLogLog就是概率算法的一种实现，下面重点介绍一下此算法。

HyperLogLog 原理思路是通过给定 n 个的元素集合，记录集合中数字的比特串第一个1出现位置的最大值k，也可以理解为统计二进制低位连续为零的最大个数。通过k值可以估算集合中不重复元素的数量m，m近似等于2^k。

下图来源于网络，通过给定一定数量的用户User，通过Hash得到一串Bitstring，记录其中最大连续零位的计数为4，User的不重复个数为 2 ^ 4 = 16

![image-20200710215403750](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200710215403750.png)



Redis采用了16384个桶来存储计算HyperLogLog，那所占的内存会是多少？ Redis最大可以统计2^64个数据，也就是说每个桶的最大maxbits需要 6 个bit来存储(2^6=64)。那么所占内存就是 16384 * 6 / 8 = 12kb。

* 目的是做基数统计，故不是集合，不会保存元数据，只记录数量而不是数值。
* 耗空间极小，支持输入非常体积的数据量
* 核心是基数估算算法，主要表现为计算时内存的使用和数据合并的处理。最终数值存在一定误差
* redis中每个hyperloglog key占用了12K的内存用于标记基数（官方文档）
* pfadd命令并不会一次性分配12k内存，而是随着基数的增加而逐渐增加内存分配；而pfmerge操作则会将sourcekey
* 合并后存储在12k大小的key中，这由hyperloglog合并操作的原理（两个hyperloglog合并时需要单独比较每个桶的值）可以很容易理解。
* 误差说明：基数估计的结果是一个带有 0.81% 标准错误（standard error）的近似值。是可接受的范围
* Redis 对 HyperLogLog 的存储进行了优化，在计数比较小时，它的存储空间采用稀疏矩阵存储，空间占用很小，仅仅在计数慢慢变大，稀疏矩阵占用空间渐渐超过了阈值时才会一次性转变成稠密矩阵，才会占用 12k 的空间
  



## Sorted set(Zset)

![image-20200727195050035](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200727195050035.png)

# Rabbitmq 

## 作用

AMQP，即Advanced Message Queuing Protocol,一个提供统一消息服务的应用层标准高级消息队列协议，是应用层协议的一个开放标准,为面向消息的中间件设计。

RabbitMQ，是一个消息代理和队列服务器，它实现了AMQP标准协议。

分布式消息队列有很多应用场景，比如异步处理、应用解耦、流量削峰等。

#### 1、异步处理

用户注册后需要发送短信和邮件，传统做法是先将用户信息写入数据库，然后发送短信、发送邮件，都完成后返回。

如果用到消息队列，可以先将用户信息写入数据库，然后将注册信息写入消息队列，发送短信、发送邮件或者还有其他的业务逻辑都订阅此消息，完成发送。

#### 2、应用解耦

还是上面的例子，如果在一个大型分布式网站中，用户系统、短信系统、邮件系统可能都是独立的系统服务。

这时候，在用户注册成功后，你可以通过RPC远程调用不同的服务接口，但更好的做法还是通过消息队列，订阅自己感兴趣的数据，日后就算增加或者删减功能，主业务都不用变动。

#### 3、流量削峰

一般在秒杀或者团购活动中，流量激增，应用面临压力过大。可以在应用前端加入消息队列，


链接：https://juejin.im/post/5dc15cdde51d4529e730696d

![image-20200722221415011](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200722221415011.png)

## Exchange

**交换机的属性**

- Name : 交换机名称
- Type : 交换机类型, direct, topic, fanout, headers
- Durability : 是否需要持久化, true为持久化
- Auto Delete : 当最后一个绑定到Exchange上的队列删除后, 自动删除该Exchange
- Internal : 当前Exchange是否用于RabbitMQ内部使用, 默认为False, 这个属性很少会用到
- Arguments : 扩展参数, 用于扩展AMQP协议制定化使用

交换器，消息到达服务的第一站就是交换器，然后根据分发规则，匹配路由键，将消息放到对应队列中。值得注意的是，交换器的类型不止一种。

- Direct 直连交换器，只有在消息中的路由键和绑定关系中的键一致时，交换器才把消息发到相应队列，Direct exchange（直连交换机）是根据消息携带的路由键（routing key）将消息投递给对应队列的，注意 : Direct模式可以使用RabbitMQ自带的Exchange(default Exchange), 所以不需要将Exchange进行任何绑定(binding)操作, 消息传递时, RoutingKey必须完全匹配才会被队列接收, 否则该消息会被抛弃  

- Fanout 广播交换器，只要消息被发送到广播交换器，它会将消息发到所有的队列，Fanout exchange（扇型交换机）将消息路由给绑定到它身上的所有队列

  - 不处理路由键, 只需要简单的将队列绑定到交换机上
  - 发送到交换机的消息都会被转发到与该交换机绑定的所有队列上
  - Fanout交换机转发消息是最快的

- Topic 主题交换器，根据路由键，通配规则(*和#)，将消息发到相应队列，Topic exchange（主题交换机）队列通过路由键绑定到交换机上，然后，交换机根据消息里的路由值，将消息路由给一个或多个绑定队列（模糊匹配）

  - “#” : 匹配一个或多个词
  - “*” : 匹配一个词

  

  Headers exchange（头交换机）类似主题交换机，但是头交换机使用多个消息属性来代替路由键建立路由规则。通过判断消息头的值能否与指定的绑定相匹配来确立路由规则。

  

![image-20200714181915190](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200714181915190.png)

交换器负责接收来自生产者的消息，并将将消息路由到一个或者多个队列中，如果路由不到，则返回给生产者或者直接丢弃，这取决于交换器的 mandatory 属性：

- 当 mandatory 为 true 时：如果交换器无法根据自身类型和路由键找到一个符合条件的队列，则会将该消息返回给生产者；
- 当 mandatory 为 false 时：如果交换器无法根据自身类型和路由键找到一个符合条件的队列，则会直接丢弃该消息。


作者：heibaiying
链接：https://juejin.im/post/5e1302315188253a5d560145
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



## RabbitMQ常用的5种工作模式

https://juejin.im/post/5e625a216fb9a07c8334e9db

## 持久化

事实上，上图所示只是一个最基本的消息流转过程，交换器和队列这些组件还有一个比较重要的属性：持久化。

默认情况下，重启RabbitMQ服务器之后，我们创建的交换器和队列都会消失不见，当然了，如果里面还有未来得及消费的数据，也将难于幸免。 持久化交换器和队列，为的是在AMQP服务器重启之后，重新创建它们并绑定关系，在RabbitMQ中，设置durable属性为true即可。

不过，除了这些还不够。虽然保证了交换器和队列是安全的，但那些还未来得及消费的数据就变得岌岌可危。所以，我们还要设置消息的投递模式为持久的。

这样，如果RabbitMQ服务器重启的话，我们的策略和相关数据才会确保无忧。所以，我们说能从AMQP服务器崩溃中恢复的消息，称之为持久化消息。那么，它必须保证以下三点：

- 设置投递模式为持久的
- 交换器为持久的
- 队列为持久的

## 消息可靠性传递或回退（生产者端）

生产者发送消息出去之后，不知道到底有没有发送到RabbitMQ服务器， 默认是不知道的。而且有的时候我们在发送消息之后，后面的逻辑出问题了，我们不想要发送之前的消息了，需要撤回该怎么做。

**AMQP 事务机制**

- txSelect  将当前channel设置为transaction模式
- txCommit  提交当前事务
- txRollback  事务回滚

**Confirm 模式**

消息的确认, 是指生产者投递消息后, 如果Broker收到消息, 则会给我们产生一个应答

生产者进行接收应答, 用来确定这条消息是否正常发送到Broker, 这种方式也是消息的可靠性投递的核心保障

- 在channel上开启确认模式 : channel.confirmSelect()
- 在channel上添加监听 : addConfirmListener, 监听成功和失败的返回结果, 根据具体的结果对消息进行重新发送, 或记录日志等后续处理

**Return消息机制**

Return Listener用于处理一些不可路由的消息

正常情况下消息生产者通过指定一个Exchange和RoutingKey, 把消息送到某一个队列中去, 然后消费者监听队列, 进行消费，但在某些情况下, 如果在发送消息的时候, 当前的exchange不存在或者指定的路由key路由不到,这个时候如果我们需要监听这种不可达的消息, 就要使用Return Listener。

在基础API中有一个关键的配置项Mandatory : 如果为true, 则监听器会接收到路由不可达的消息, 然后进行后续处理（补偿或人工处理）, 如果为false, 那么broker端自动删除该消息。

**如何保障消息可靠传递**

- 保障消息的成功发出
- 保障MQ节点的成功接收
- 发送端收到MQ节点(Broker)的确认应答
- 完善的消息补偿机制

方案：

1、消息落库, 对消息状态进行标记



![image-20200714183231037](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200714183231037.png)



- step1:消息入库
- step2:消息发送
- step3:消费端消息确认
- step4:更新库中消息状态为已确认
- step5:定时任务读取数据库中未确认的消息
- step6:未收到确认结果的消息重新发送
- step7:如果重试几次之后仍然失败, 则将消息状态更改为投递失败的终态, 后面需要人工介入

2、消息的延迟投递, 做二次确认, 回调检查

![image-20200714183622204](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200714183622204.png)





- step1 : 第一次消息发送, 必须业务数据落库之后才能进行消息发送
- step2 : 第二次消息延迟发送, 设定延迟一段时间发送第二次check消息
- step3 : 消费端监听Broker, 进行消息消费
- step4 : 消费成功之后, 发送确认消息到确认消息队列
- step5 : Callback Service监听step4中的确认消息队列, 维护消息状态, 是否消费成功等状态
- step6 : Callback Service监听step2发送的Delay Check的消息队列, 检测内部的消息状态, 如果消息是发送成功状态, 则流程结束, 如果消息是失败状态, 或者查不到当前消息状态时, 会通知生产者, 进行消息重发, 重新上述步骤



# MQ的缺点

如果没有MQ消息队列，那么肯定就不会有当MQ服务宕机的时候，导致整体系统的宕机了。除了宕机这种情况，我们还要考虑到消息有没有重复消费，或者消息丢失了怎么办，或者说其他的服务都消费MQ成功了，但是偏偏还有一个服务消费MQ失败了，这等等一系列的问题。（需要保持最终一致性问题，并且要有业务监控系统。）

## 1.重复消费

因为消费者是隔一段时间来提交自己消费的消息的编号offset的，也就是说如果消费者突然宕机了，那么有可能有的消息已经消费了，但是没有提交到MQ那里让MQ知道，因此当消费者重启后，可能会导致消息的重复消费。 通过业务的判断，来保证代码业务的幂等性。

## 2.消息丢失

### 2.1生产者搞丢了数据

可能是因为网络或者其他的一些问题，导致生产者发送的MQ消息在半路就丢了，没有发送到mq那里去。

因此一般确保MQ消息不丢失，我们需要在生产者发送了MQ消息给MQ之后（MQ将消息持久化），需要MQ返回一个Ack消息给生产者，这样的话生产者就能保证自己的消息已经发送给MQ了，如果一段时间内MQ没有回传消息给到生产者，那么生产者可以选择重发消息。

### 2.2 mq弄丢了消息

可能是因为mq宕机了等等一系列的问题，导致mq的消息丢失了，那么这个时候要开启mq的持久化，将mq消息持久化到磁盘上，这样即使是mq宕机了，也不影响。

### 2.3 消费者将消息搞丢了

一般来说，当消费者消费到一条消息后，会通知MQ说我已经消费到这条消息了，但是如果消费这条消息而且正在处理的时候，消费者宕机了，那么就会导致消息丢失。 消费者应该在消费到消息并且已经处理完了，再通知MQ。（rabbitMQ关掉autoAck模式）（如果是kafka，需要关闭自动提交offset，那么也是在处理完消息后，再返回offset。）

## 3.如果保持MQ消息的顺序性

rabbitMQ在mq里面创建多个队列queue，保证一个队列queue只被一个消费者消费，这样的话就能保证消息的顺序性。



![img](https://user-gold-cdn.xitu.io/2020/7/5/1731efa5e62ce686?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Kafka中可以指定一个key，保证这个key对应的mq消息是进入到一个partition中去，而一个partiton在kafka中是规定了只被一个消费者消费，这样就能保证mq消息的顺序性了。



## 4.消息队列里面堆积了几百万消息

### 4.1

堆积了太多的消息，rabbitMQ里面消息会过期。因此需要只能在晚上来补偿这些消息

### 4.2

kafka堆积了太多消息，那么只能紧急扩容消费端，扩大消费端的消费能力。也可以新建一个topic，partition数量是之前的10倍，然后一个partition只能对应一个消费端，然后又将消费端的性能扩容，紧急的赶紧把这些消息消费掉。当这些消息消费完了之后，再使用之前的架构来消费。




# 分布式

https://doocs.github.io/advanced-java/#/./docs/distributed-system/distributed-system-interview

1. 一致性hash算法

   ![image-20200718220126922](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200718220126922.png)
   
   
   
2. XA事务

   https://juejin.im/post/5dcd53d6f265da0beb53cf49

   https://segmentfault.com/a/1190000012534071

   

3. 选举 https://zhuanlan.zhihu.com/p/130332285

4. 分布式事务 https://doocs.github.io/advanced-java/#/./docs/distributed-system/distributed-transaction

5. Redis 分布式锁http://zhangtielei.com/posts/blog-redlock-reasoning.html

6. zk分布式锁 https://zhuanlan.zhihu.com/p/73807097

7. 分布式Session共享解决方案 https://juejin.im/post/5c13bea65188251595128d4b



## 幂等性实现方案



## 乐观锁

如果只是更新已有的数据，没有必要对业务进行加锁，设计表结构时使用乐观锁，一般通过version来做乐观锁，这样既能保证执行效率，又能保证幂等。

## 防重表

使用订单号 orderNo 做为去重表的唯一索引，每次请求都根据订单号向去重表中插入一条数据。第一次请求查询订单支付状态，订单没有支付，进行支付操作，无论成功与否，执行完后更新订单状态为成功或失败，删除去重表中的数据。后续的订单因为表中唯一索引而插入失败，则返回操作失败，直到第一次的请求完成（成功或失败）。



# 分布式锁

对于防重表可以用分布式锁代替，比如 Redis 和 Zookeeper

## **Redis**

1. 订单发起支付请求，支付系统会去 Redis 缓存中查询是否存在该订单号的 Key，如果不存在，则向 Redis 增加 Key 为订单号
2. 查询订单支付状态，如果未支付，则进行支付流程，支付完成后删除该订单号的 key



**Zookeeper**

1. 订单发起支付请求，支付系统会去 Zookeeper 中创建一个 node，如果创建失败，则表示订单已经被支付
2. 如果创建成功，则进行支付流程，支付完成后删除 node

## Token 机制

这种方式分成两个阶段：申请 Token 阶段和支付阶段。 第一阶段，在进入到提交订单页面之前，需要订单系统根据用户信息向支付系统发起一次申请 Token 的请求，支付系统将 Token 保存到 Redis 缓存中，为第二阶段支付使用。 第二阶段，订单系统拿着申请到的 Token 发起支付请求，支付系统会检查 Redis 中是否存在该 Token ，如果存在，表示第一次发起支付请求，删除缓存中 Token 后开始支付逻辑处理；如果缓存中不存在，表示非法请求。

## 消息队列缓冲

将订单的支付请求全部发送到消息队列中，然后使用异步任务处理队列中的数据，过滤掉重复的待支付订单，再进行支付流程。



## Redis分布式锁问题

因为Redis的复制是异步的。

举例来说：

1. 线程A在master节点拿到了锁。
2. master节点在把A创建的key写入slave之前宕机了。
3. slave变成了master节点。
4. 线程B也得到了和A还持有的相同的锁。（因为原来的slave里面还没有A持有锁的信息）



## 服务雪崩的应对策略

针对造成服务雪崩的不同原因, 可以使用不同的应对策略:

1. 流量控制
2. 改进缓存模式
3. 服务自动扩容
4. 服务调用者降级服务

流量控制 的具体措施包括:

- 网关限流
- 用户交互限流
- 关闭重试

因为Nginx的高性能, 目前一线互联网公司大量采用Nginx+Lua的网关进行流量控制, 由此而来的OpenResty也越来越热门.

用户交互限流的具体措施有: 1. 采用加载动画,提高用户的忍耐等待时间. 2. 提交按钮添加强制等待时间机制.

改进缓存模式 的措施包括:

- 缓存预加载
- 同步改为异步刷新

服务自动扩容 的措施主要有:

- AWS的auto scaling

服务调用者降级服务 的措施包括:

- 资源隔离
- 对依赖服务进行分类
- 不可用服务的调用快速失败

资源隔离主要是对调用服务的线程池进行隔离.

我们根据具体业务,将依赖服务分为: 强依赖和若依赖. 强依赖服务不可用会导致当前业务中止,而弱依赖服务的不可用不会导致当前业务的中止.

不可用服务的调用快速失败一般通过 超时机制, 熔断器 和熔断后的 降级方法 来实现.

## 服务熔断Hystrix

## Hystrix的内部处理逻辑

下图为Hystrix服务调用的内部逻辑: 
![img](https://img-blog.csdn.net/20171030152140596?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFveWVxaXU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

1. 构建Hystrix的Command对象, 调用执行方法.
2. Hystrix检查当前服务的熔断器开关是否开启, 若开启, 则执行降级服务getFallback方法.
3. 若熔断器开关关闭, 则Hystrix检查当前服务的线程池是否能接收新的请求, 若超过线程池已满, 则执行降级服务getFallback方法.
4. 若线程池接受请求, 则Hystrix开始执行服务调用具体逻辑run方法.
5. 若服务执行失败, 则执行降级服务getFallback方法, 并将执行结果上报Metrics更新服务健康状况.
6. 若服务执行超时, 则执行降级服务getFallback方法, 并将执行结果上报Metrics更新服务健康状况.
7. 若服务执行成功, 返回正常结果.
8. 若服务降级方法getFallback执行成功, 则返回降级结果.
9. 若服务降级方法getFallback执行失败, 则抛出异常.



# 分布式事务

## 二提交

**阶段1：**

  TM通知各个RM准备提交它们的事务分支。如果RM判断自己进行的工作可以被提交，那就就对工作内容进行持久化，再给TM肯定答复；要是发生了其他情况，那给TM的都是否定答复。在发送了否定答复并回滚了已经的工作后，RM就可以丢弃这个事务分支信息。

  以mysql数据库为例，在第一阶段，事务管理器向所有涉及到的数据库服务器发出prepare"准备提交"请求，数据库收到请求后执行数据修改和日志记录等处理，处理完成后只是把事务的状态改成"可以提交",然后把结果返回给事务管理器。

**阶段2**

  TM根据阶段1各个RM prepare的结果，决定是提交还是回滚事务。如果所有的RM都prepare成功，那么TM通知所有的RM进行提交；如果有RM prepare失败的话，则TM通知所有RM回滚自己的事务分支。

​    以mysql数据库为例，如果第一阶段中所有数据库都prepare成功，那么事务管理器向数据库服务器发出"确认提交"请求，数据库服务器把事务的"可以提交"状态改为"提交完成"状态，然后返回应答。如果在第一阶段内有任何一个数据库的操作发生了错误，或者事务管理器收不到某个数据库的回应，则认为事务失败，回撤所有数据库的事务。数据库服务器收不到第二阶段的确认提交请求，也会把"可以提交"的事务回撤。

### 优化

 **只读断言**

   在Phase 1中，RM可以断言“我这边不涉及数据增删改”来答复TM的prepare请求，从而让这个RM脱离当前的全局事务，从而免去了Phase 2。

这种优化发生在其他RM都完成prepare之前的话，使用了只读断言的RM早于AP其他动作（比如说这个RM返回那些只读数据给AP）前，就释放了相关数据的上下文（比如读锁之类的），这时候其他全局事务或者本地事务就有机会去改变这些数据，结果就是无法保障整个系统的可序列化特性——通俗点说那就会有脏读的风险。

**一阶段提交**

   如果需要增删改的数据都在同一个RM上，TM可以使用一阶段提交——跳过两阶段提交中的Phase 1，直接执行Phase 2。

这种优化的本质是跳过Phase 1，RM自行决定了事务分支的结果，并且在答复TM前就清除掉事务分支信息。对于这种优化的情况，TM实际上也没有必要去可靠的记录全局事务的信息，在一些异常的场景下，此时TM可能不知道事务分支的执行结果。 

###  **两阶段提交协议(2PC)存在的问题**

二阶段提交看起来确实能够提供原子性的操作，但是不幸的是，二阶段提交还是有几个缺点的：

**1、同步阻塞问题。**两阶段提交方案下全局事务的ACID特性，是依赖于RM的。例如mysql5.7官方文档关于对XA分布式事务的支持有以下介绍：

https://dev.mysql.com/doc/refman/5.7/en/xa.html

>  A global transaction involves several actions that are transactional in themselves, but that all must either complete successfully as a group, or all be rolled back as a group. In essence, this extends ACID properties “up a level” so that multiple ACID transactions can be executed in concert as components of a global operation that also has ACID properties. (As with nondistributed transactions, [SERIALIZABLE](https://dev.mysql.com/doc/refman/5.7/en/innodb-transaction-isolation-levels.html#isolevel_serializable) may be preferred if your applications are sensitive to read phenomena. [REPEATABLE READ](https://dev.mysql.com/doc/refman/5.7/en/innodb-transaction-isolation-levels.html#isolevel_repeatable-read) may not be sufficient for distributed transactions.)



  大致含义是说，一个全局事务内部包含了多个独立的事务分支，这一组事务分支要不都成功，要不都失败。各个事务分支的ACID特性共同构成了全局事务的ACID特性。也就是将单个事务分支的支持的ACID特性提升一个层次(up a level)到分布式事务的范畴。括号中的内容的意思是： 即使在非分布事务中(即本地事务)，如果对操作读很敏感，我们也需要将事务隔离级别设置为SERIALIZABLE。而对于分布式事务来说，更是如此，可重复读隔离级别不足以保证分布式事务一致性。

   也就是说，如果我们使用mysql来支持XA分布式事务的话，那么最好将事务隔离级别设置为SERIALIZABLE。 地球人都知道，SERIALIZABLE(串行化)是四个事务隔离级别中最高的一个级别，也是执行效率最低的一个级别。

**2、单点故障。**由于协调者的重要性，一旦协调者TM发生故障。参与者RM会一直阻塞下去。尤其在第二阶段，协调者发生故障，那么所有的参与者还都处于锁定事务资源的状态中，而无法继续完成事务操作。（如果是协调者挂掉，可以重新选举一个协调者，但是无法解决因为协调者宕机导致的参与者处于阻塞状态的问题）

**3、数据不一致。**在二阶段提交的阶段二中，当协调者向参与者发送commit请求之后，发生了局部网络异常或者在发送commit请求过程中协调者发生了故障，这回导致只有一部分参与者接受到了commit请求。而在这部分参与者接到commit请求之后就会执行commit操作。但是其他部分未接到commit请求的机器则无法执行事务提交。于是整个分布式系统便出现了数据不一致性的现象。

由于二阶段提交存在着诸如同步阻塞、单点问题等缺陷，所以，研究者们在二阶段提交的基础上做了改进，提出了三阶段提交。 



## **三阶段提交协议(Three-phase commit)**

  三阶段提交（3PC)，是二阶段提交（2PC）的改进版本。参考维基百科：https://en.wikipedia.org/wiki/Three-phase_commit_protocol

与两阶段提交不同的是，三阶段提交有两个改动点。

  1、引入**超时机制**。同时在协调者和参与者中都引入超时机制。

  2、在第一阶段和第二阶段中插入一个**准备阶段**。保证了在最后提交阶段之前各参与节点的状态是一致的。也就是说，除了引入超时机制之外，3PC把2PC的准备阶段再次一分为二，这样三阶段提交就有CanCommit、PreCommit、DoCommit三个阶段。

![9AFDC04C-016D-4CA6-8C04-AE7702264AFC.png](http://static.tianshouzhi.com/ueditor/upload/image/20180205/1517793175448046667.png)



**CanCommit阶段**

   3PC的CanCommit阶段其实和2PC的准备阶段很像。协调者向参与者发送commit请求，参与者如果可以提交就返回Yes响应，否则返回No响应。

1. **事务询问** 协调者向参与者发送CanCommit请求。询问是否可以执行事务提交操作。然后开始等待参与者的响应。

2. **响应反馈** 参与者接到CanCommit请求之后，正常情况下，如果其自身认为可以顺利执行事务，则返回Yes响应，并进入预备状态。否则反馈No

**PreCommit阶段**

   协调者根据参与者的反应情况来决定是否可以记性事务的PreCommit操作。根据响应情况，有以下两种可能。

  假如协调者从所有的参与者获得的反馈都是Yes响应，那么就会执行事务的预执行。

1. 发送预提交请求 协调者向参与者发送PreCommit请求，并进入Prepared阶段。   

2. 事务预提交 参与者接收到PreCommit请求后，会执行事务操作，并将undo和redo信息记录到事务日志中。

3. 响应反馈 如果参与者成功的执行了事务操作，则返回ACK响应，同时开始等待最终指令。

  假如有任何一个参与者向协调者发送了No响应，或者等待超时之后，协调者都没有接到参与者的响应，那么就执行事务的中断。

1. 发送中断请求 协调者向所有参与者发送abort请求。

2. 中断事务 参与者收到来自协调者的abort请求之后（或超时之后，仍未收到协调者的请求），执行事务的中断。

**doCommit阶段**

  该阶段进行真正的事务提交，也可以分为以下两种情况。

  **Case 1：执行提交**

1. 发送提交请求 协调接收到参与者发送的ACK响应，那么他将从预提交状态进入到提交状态。并向所有参与者发送doCommit请求。

2. 事务提交 参与者接收到doCommit请求之后，执行正式的事务提交。并在完成事务提交之后释放所有事务资源。

3. 响应反馈 事务提交完之后，向协调者发送Ack响应。

4. 完成事务 协调者接收到所有参与者的ack响应之后，完成事务。

  **Case 2：中断事务** 协调者没有接收到参与者发送的ACK响应（可能是接受者发送的不是ACK响应，也可能响应超时），那么就会执行中断事务。

1. 发送中断请求 协调者向所有参与者发送abort请求

2. 事务回滚 参与者接收到abort请求之后，利用其在阶段二记录的undo信息来执行事务的回滚操作，并在完成回滚之后释放所有的事务资源。

3. 反馈结果 参与者完成事务回滚之后，向协调者发送ACK消息

4. 中断事务 协调者接收到参与者反馈的ACK消息之后，执行事务的中断。 



  在doCommit阶段，如果参与者无法及时接收到来自协调者的doCommit或者rebort请求时，会在等待超时之后，会继续进行事务的提交。（其实这个应该是基于概率来决定的，当进入第三阶段时，说明参与者在第二阶段已经收到了PreCommit请求，那么协调者产生PreCommit请求的前提条件是他在第二阶段开始之前，收到所有参与者的CanCommit响应都是Yes。（一旦参与者收到了PreCommit，意味他知道大家其实都同意修改了）所以，一句话概括就是，当进入第三阶段时，由于网络超时等原因，虽然参与者没有收到commit或者abort响应，但是他有理由相信：成功提交的几率很大。 ）



**小结：2PC与3PC的区别 ** https://zhuanlan.zhihu.com/p/35616810

   相对于2PC，3PC主要解决的单点故障问题，并减少阻塞，因为一旦参与者无法及时收到来自协调者的信息之后，他会默认执行commit。而不会一直持有事务资源并处于阻塞状态。但是这种机制也会导致数据一致性问题，因为，由于网络原因，协调者发送的abort响应没有及时被参与者接收到，那么参与者在等待超时之后执行了commit操作。这样就和其他接到abort命令并执行回滚的参与者之间存在数据不一致的情况。

   了解了2PC和3PC之后，我们可以发现，无论是二阶段提交还是三阶段提交都无法彻底解决分布式的一致性问题。Google Chubby的作者Mike Burrows说过， there is only one consensus protocol, and that’s Paxos” – all other approaches are just broken versions of Paxos. 意即世上只有一种一致性算法，那就是Paxos，所有其他一致性算法都是Paxos算法的不完整版。后面的文章会介绍这个公认为难于理解但是行之有效的Paxos算法。  



# 秒杀项目



![image-20200729153323255](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200729153323255.png)





![image-20200729153809397](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200729153809397.png)

### 拦截器

继承 HandlerInterceptorAdapter # 重写   

> public  **boolean** preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) **throws** Exception 

方法，返回true 表示继续， 返回false 表示不继续

	**if** (handler **instanceof** HandlerMethod)   // 指明拦截的是方法

```java

        // 指明拦截的是方法
        if (handler instanceof HandlerMethod) {
            SeckillUser user = this.getUser(request, response);// 获取用户对象
            UserContext.setUser(user);// 保存用户到ThreadLocal，这样，同一个线程访问的是同一个用户
            HandlerMethod hm = (HandlerMethod) handler;
            // 获取标注了@AccessLimit的方法，没有注解，则直接返回
            AccessLimit accessLimit = hm.getMethodAnnotation(AccessLimit.class);
            // 如果没有添加@AccessLimit注解，直接放行（true）
            if (accessLimit == null) {
                return true;
            }
```

解析请求继承 HandlerMethodArgumentResolver 

```java
 /**
     * 当请求参数为MiaoshaUser时，使用这个解析器处理
     * 客户端的请求到达某个Controller的方法时，判断这个方法的参数是否为MiaoshaUser，
     * 如果是，则这个MiaoshaUser参数对象通过下面的resolveArgument()方法获取，
     * 然后，该Controller方法继续往下执行时所看到的MiaoshaUser对象就是在这里的resolveArgument()方法处理过的对象
     *
     * @param methodParameter
     * @return
     */
    @Override
    public boolean supportsParameter(MethodParameter methodParameter) {
        Class<?> parameterType = methodParameter.getParameterType();
        return parameterType == SeckillUser.class;
    }
 /**
     * 获取MiaoshaUser对象
     *
     * @param methodParameter
     * @param modelAndViewContainer
     * @param nativeWebRequest
     * @param webDataBinderFactory
     * @return
     * @throws Exception
     */
    @Override
    public Object resolveArgument(MethodParameter methodParameter,
                                  ModelAndViewContainer modelAndViewContainer,
                                  NativeWebRequest nativeWebRequest,
                                  WebDataBinderFactory webDataBinderFactory) throws Exception {
        // 获取请求和响应对象
        HttpServletRequest request = nativeWebRequest.getNativeRequest(HttpServletRequest.class);
        HttpServletResponse response = nativeWebRequest.getNativeResponse(HttpServletResponse.class);

        // 从请求对象中获取token（token可能有两种方式从客户端返回，1：通过url的参数，2：通过set-Cookie字段）
        String paramToken = request.getParameter(SeckillUserService.COOKIE_NAME_TOKEN);
        String cookieToken = getCookieValue(request, SeckillUserService.COOKIE_NAME_TOKEN);

        if (StringUtils.isEmpty(cookieToken) && StringUtils.isEmpty(paramToken)) {
            return null;
        }

        // 判断是哪种方式返回的token，并由该种方式获取token（cookie）
        String token = StringUtils.isEmpty(paramToken) ? cookieToken : paramToken;
        // 通过token就可以在redis中查出该token对应的用户对象
        return seckillUserService.getMisaoshaUserByToken(response, token);
    }
```





URL 缓存/页面缓存

```java
@RequestMapping(value = "/to_detail/{goodsId}", produces = "text/html")// 注意这种写法
    @ResponseBody
    public String toDetail(
            HttpServletRequest request,
            HttpServletResponse response,
            Model model,
            SeckillUser user,
            @PathVariable("goodsId") long goodsId) {

        // 1. 根据商品id从redis中取详情数据的缓存
        String html = redisService.get(GoodsKeyPrefix.goodsDetailKeyPrefix, "" + goodsId, String.class);
        if (!StringUtils.isEmpty(html)) {// 如果缓存中数据存在着直接返回
            return html;
        }

        // 2. 如果缓存中数据不存在，则需要手动渲染详情界面数据并返回
        model.addAttribute("user", user);
        // 通过商品id查询
        GoodsVo goods = goodsService.getGoodsVoByGoodsId(goodsId);
        model.addAttribute("goods", goods);

        // 获取商品的秒杀开始与结束的时间
        long startDate = goods.getStartDate().getTime();
        long endDate = goods.getEndDate().getTime();
        long now = System.currentTimeMillis();

        // 秒杀状态; 0: 秒杀未开始，1: 秒杀进行中，2: 秒杀已结束
        int miaoshaStatus = 0;
        // 秒杀剩余时间
        int remainSeconds = 0;

        if (now < startDate) { // 秒杀未开始
            miaoshaStatus = 0;
            remainSeconds = (int) ((startDate - now) / 1000);
        } else if (now > endDate) { // 秒杀已结束
            miaoshaStatus = 2;
            remainSeconds = -1;
        } else { // 秒杀进行中
            miaoshaStatus = 1;
            remainSeconds = 0;
        }

        // 将秒杀状态和秒杀剩余时间传递给页面（goods_detail）
        model.addAttribute("miaoshaStatus", miaoshaStatus);
        model.addAttribute("remainSeconds", remainSeconds);

//        return "goods_detail";

        // 3. 渲染html
        WebContext webContext = new WebContext(request, response, request.getServletContext(), request.getLocale(), model.asMap());
        // (第一个参数为渲染的html文件名，第二个为web上下文：里面封装了web应用的上下文)
        html = thymeleafViewResolver.getTemplateEngine().process("goods_detail", webContext);

        if (!StringUtils.isEmpty(html)) // 如果html文件不为空，则将页面缓存在redis中
            redisService.set(GoodsKeyPrefix.goodsDetailKeyPrefix, "" + goodsId, html);

        return html;
    }

```

对

```java
    @RequestMapping(value = "/to_detail_static/{goodsId}")// 注意这种写法
//    @RequestMapping(value = "/to_detail/{goodsId}")// 注意这种写法
    @ResponseBody
    public Result<GoodsDetailVo> toDetailStatic(SeckillUser user, @PathVariable("goodsId") long goodsId) {

        // 通过商品id再数据库查询
        GoodsVo goods = goodsService.getGoodsVoByGoodsId(goodsId);

        // 获取商品的秒杀开始与结束的时间
        long startDate = goods.getStartDate().getTime();
        long endDate = goods.getEndDate().getTime();
        long now = System.currentTimeMillis();

        // 秒杀状态; 0: 秒杀未开始，1: 秒杀进行中，2: 秒杀已结束
        int miaoshaStatus = 0;
        // 秒杀剩余时间
        int remainSeconds = 0;

        if (now < startDate) { // 秒杀未开始
            miaoshaStatus = 0;
            remainSeconds = (int) ((startDate - now) / 1000);
        } else if (now > endDate) { // 秒杀已结束
            miaoshaStatus = 2;
            remainSeconds = -1;
        } else { // 秒杀进行中
            miaoshaStatus = 1;
            remainSeconds = 0;
        }

        // 服务端封装商品数据直接传递给客户端，而不用渲染页面
        GoodsDetailVo goodsDetailVo = new GoodsDetailVo();
        goodsDetailVo.setGoods(goods);
        goodsDetailVo.setUser(user);
        goodsDetailVo.setRemainSeconds(remainSeconds);
        goodsDetailVo.setSeckillStatus(miaoshaStatus);

        return Result.success(goodsDetailVo);
    }

```

**如何做的IP限流**  

   我的实现思路是放在缓存，每次IP进来都去缓存里访问看看有没有存在此IP，如果存在且还没过期那就不放行，如果不存在或者已经到期就放行，这里其实是针对某一特定IP的频率限制。 

### 分布式Session一致性？

**解决方案：**

- 使用cookie来完成（很明显这种不安全的操作并不可靠）

- 使用Nginx中的ip绑定策略，同一个ip只能在指定的同一个机器访问（不支持负载均衡）

- 利用数据库同步session（效率不高）

- 使用tomcat内置的session同步（同步可能会产生延迟）

- 使用token代替session

- **我们使用spring-session以及集成好的解决方案，存放在redis中**

  链接：https://juejin.im/post/5c13bea65188251595128d4b





### spring-session 

### 工作流程

Tomcat web.xml解析步骤：

```java
contextInitialized(ServletContextEvent arg0); // Listener
init(FilterConfig filterConfig); // Filter
init(ServletConfig config); // Servlet

```

![img](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

初始化顺序：Listener > Filter > Servlet。

  **如果要实现更加广义上的，比如一秒钟放行1k请求，怎么做**  

   用桶算法去处理，令牌桶/漏桶

## 页面静态化

HTML静态化的好处:

（1）减轻服务器负担，浏览网页无需调用系统数据库。

（2）有利于搜索引擎优化SEO，Baidu、Google都会优先收录静态页面，不仅被收录的快还收录的全；

（3）加快页面打开速度，静态页面无需连接数据库打开速度较动态页面有明显提高；

（4）网站更安全，HTML页面不会受php程序相关漏洞的影响；观看一下大一点的网站基本全是静态页面，而且可以减少攻击，防sql注入。数据库出错时，不影响网站正常访问。

（5）数据库出错时，不影响网站的正常访问。

（6）最主要是可以增加访问速度,减轻服务器负担,当数据量有几万，几十万或是更多的时候你知道哪个更快了. 而且还容易被搜索引擎找到。生成html文章虽操作上麻烦些，程序上繁杂些，但为了更利于搜索，为了速度更快些，更安全，这些牺牲还是值得的。





[https://machenxing.github.io/2019/07/20/%E5%A4%A7%E5%9E%8B%E7%BD%91%E7%AB%99%E7%9A%84%E9%A1%B5%E9%9D%A2%E9%9D%99%E6%80%81%E5%8C%96/](https://machenxing.github.io/2019/07/20/大型网站的页面静态化/)

### 方案一：网页静态HTML化

![image-20200728153100041](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200728153100041.png)

上图的核心思想：

> 1）管理后台调用新闻服务创建文章成功后，发送消息到消息队列
>
> 2）静态服务监听消息，把文章静态化，也就是生成html文件
>
> 3）在静态服务器上面安装一个文件同步工具，此工具的功能可以做到只同步有变动的文件，即做增量同步
>
> 4）通过同步工具把html文件同步到所有的web服务器上面

这样的话就达到了，用户访问一些变化不大的页面时，是直接访问的html文件，直接在web服务器那边直接返回，不需要在访问数据库了，系统吞吐量比较高。

这个方案的问题：

**1、网页布局样式僵化，无法修改**

> 如果产品经理觉得新闻详情页面的布局要调整一下，现在的不够美观，或者加个其他模块，那就坑爹了，我们需要把所有的已经静态html化的文章全部重新静态化。这个是不现实的，因为像网易这么大的体量，新闻量是很大的，会被搞死。

**2、页面会出现暂时间不一致**

> 会出现用户刚刚再看最新的新闻，刷新一下又不存在了。这个是因为同步工具在同步到web服务器是要有时间的，同步到web服务器A上面了，但web服务器B还没有来得及同步。用户在访问的时候通过**nginx进行负载均衡，随机把请求分配给web服务器**的导致的。当然可以调整nginx负载均衡策略去解决。

**3、Html文件太多，无法维护**

> 这个是很明显的问题，html文件会越来越多，对存储空间要求很大，而且每台web服务器都一样，浪费磁盘空间；将来迁移维护也会带来很大的麻烦。

**4、同步工具的不稳定**

> 因为文件一旦多之后，同步工具稳定性就出现了问题

这个方案应该是比较传统的（不推荐）

### 方案二：伪静态化

**什么是伪静态？**

举个例子：我们一般访问一个文章，一般的链接地址为：http://www.xxx.com/news?id=1代表请求id为1的文章。**不过这种链接方式对SEO不是太友好（SEO对网站来说太重要了）**；所以一般进行改造：http://www.xxx.com/news/1.html 这样看上去就是个静态页面。一般我们可以**采用nginx对url进行rewrite**。小伙伴如何有兴趣可以自行了解，比较简单。

![image-20200728153425035](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200728153425035.png)

此方案的核心思想

> 1）管理后台调用新闻服务创建文章成功后，发送消息到消息队列
>
> 2）缓存服务监听消息，把文章内容缓存到缓存服务器上面
>
> 3）用户发起请求，web服务器根据id，直接查询缓存服务器
>
> 4）获取数据返回给用户

此方案就解决了方案一的一个大问题，就是html文件多的问题，因为不需要生成html，而且用缓存的方式，解决不需要访问数据库，提升系统吞吐量。

不过此方案的问题：

> 1、**网页布局样式维护成本比较高**，因为此方案照样是把所有的内容放到了缓存中，如果需要修改布局，需要重新设置缓存。
>
> 2、分布式缓存压力比较大，一旦缓存故障就导致所有请求会查询数据库，导致系统崩溃

还有个小问题，就是实时数据处理，就是页面中如价格，库存需要到后台读取的。当然小伙伴也许就会说，也可以处理啊，用户把商品内容请求到后，然后在用浏览器发送异步的ajax请求获得商品数量就好了啊。这样就是无形的增加了一次请求。（此问题可以忽略）

此方案类似很多公司都在使用，如：同程旅游等



### 方案三：布局样式模板化

针对方案二的问题，我们可以采用openresty技术方案进行，利用http模板插件lua脚本进行解决，这里老顾不会介绍openresty+lua技术，有兴趣的小伙伴，可以到访问https://www.roncoo.com/view/139 这个视频课程。

![image-20200728153610722](/Users/lingjiarong/Library/Application Support/typora-user-images/image-20200728153610722.png)

# 分布式锁

zk ， redis ， redlock

# DOCKER



https://cloud.tencent.com/developer/article/1334962

| namespace          | 引入的相关内核版本                   | 被隔离的全局系统资源                                         | 在容器语境下的隔离效果                                       |
| :----------------- | :----------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| Mount namespaces   | Linux 2.4.19                         | 文件系统挂接点                                               | 将一个文件系统的顶层目录挂到另一个文件系统的子目录上，使它们成为一个整体，称为挂载。把该子目录称为挂载点。 　　Mount namespace用来隔离文件系统的挂载点, 使得不同的mount namespace拥有自己独立的挂载点信息，不同的namespace之间不会相互影响，这对于构建用户或者容器自己的文件系统目录非常有用。 |
| UTS namespaces     | Linux 2.6.19                         | nodename 和 domainname                                       | UTS，UNIX Time-sharing System namespace提供了主机名和域名的隔离。能够使得子进程有独立的主机名和域名(hostname)，这一特性在Docker容器技术中被用到，使得docker容器在网络上被视作一个独立的节点，而不仅仅是宿主机上的一个进程。 |
| IPC namespaces     | Linux 2.6.19                         | 特定的进程间通信资源，包括System V IPC 和  POSIX message queues | IPC全称 Inter-Process Communication，是Unix/Linux下进程间通信的一种方式，IPC有共享内存、信号量、消息队列等方法。所以，为了隔离，我们也需要把IPC给隔离开来，这样，只有在同一个Namespace下的进程才能相互通信。如果你熟悉IPC的原理的话，你会知道，IPC需要有一个全局的ID，即然是全局的，那么就意味着我们的Namespace需要对这个ID隔离，不能让别的Namespace的进程看到。 |
| PID namespaces     | Linux 2.6.24                         | 进程 ID 数字空间 （process ID number space）                 | PID namespaces用来隔离进程的ID空间，使得不同pid namespace里的进程ID可以重复且相互之间不影响。PID namespace可以嵌套，也就是说有父子关系，在当前namespace里面创建的所有新的namespace都是当前namespace的子namespace。父namespace里面可以看到所有子孙后代namespace里的进程信息，而子namespace里看不到祖先或者兄弟namespace里的进程信息。 |
| Network namespaces | 始于Linux 2.6.24 完成于 Linux 2.6.29 | 网络相关的系统资源                                           | 每个容器用有其独立的网络设备，IP 地址，IP 路由表，/proc/net 目录，端口号等等。这也使得一个 host 上多个容器内的同一个应用都绑定到各自容器的 80 端口上。 |
| User namespaces    | 始于 Linux 2.6.23 完成于 Linux 3.8)  | 用户和组 ID 空间                                             | User namespace用来隔离user权限相关的Linux资源，包括user IDs and group IDs。这是目前实现的namespace中最复杂的一个，因为user和权限息息相关，而权限又事关容器的安全，所以稍有不慎，就会出安全问题。在不同的user namespace中，同样一个用户的user ID 和group ID可以不一样，换句话说，一个用户可以在父user namespace中是普通用户，在子user namespace中是超级用户 |





**有了namespace为什么还要cgroup:**

 Docker 容器使用 linux namespace 来隔离其运行环境，使得容器中的进程看起来就像一个独立环境中运行一样。但是，光有运行环境隔离还不够，因为这些进程还是可以不受限制地使用系统资源，比如网络、磁盘、CPU以及内存 等。关于其目的，一方面，是为了防止它占用了太多的资源而影响到其它进程；另一方面，在系统资源耗尽的时候，linux 内核会触发 OOM，这会让一些被杀掉的进程成了无辜的替死鬼。因此，为了让容器中的进程更加可控，Docker 使用 Linux cgroups 来限制容器中的进程允许使用的系统资源。 

**（2）原理**

  Linux Cgroup 可为系统中所运行任务（进程）的用户定义组群分配资源 — 比如 CPU 时间、系统内存、网络带宽或者这些资源的组合。可以监控管理员配置的 cgroup，拒绝 cgroup 访问某些资源，甚至在运行的系统中动态配置 cgroup。所以，可以将 controll groups 理解为 controller （system resource） （for） （process）groups，也就是是说它以一组进程为目标进行系统资源分配和控制。它主要提供了如下功能： 

- **Resource limitation**: 限制资源使用，比如内存使用上限以及文件系统的缓存限制。
- **Prioritization**: 优先级控制，比如：CPU利用和磁盘IO吞吐。
- **Accounting**: 一些审计或一些统计，主要目的是为了计费。
- **Controll**: 挂起进程，恢复执行进程。

使用 cgroup，系统管理员可更具体地控制对系统资源的分配、优先顺序、拒绝、管理和监控。可更好地根据任务和用户分配硬件资源，提高总体效率。

在实践中，系统管理员一般会利用CGroup做下面这些事：

- 隔离一个进程集合（比如：nginx的所有进程），并限制他们所消费的资源，比如绑定CPU的核。
- 为这组进程分配其足够使用的内存
- 为这组进程分配相应的网络带宽和磁盘存储限制
- 限制访问某些设备（通过设置设备的白名单）

# System design 



## 单点登录

https://zebinh.github.io/2020/03/SSOANDOAuth/





## 百亿级微信红包的高并发资金交易系统设计方案