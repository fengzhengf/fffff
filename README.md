#### 1. ArrayList底层是数组（查找快），linkList底层是链表（增删快）

#### 1.2 java的四种引用

- **强引用**：我们常常 new 出来的对象就是强引用类型，只要强引用存在，垃圾回收器将永远不会回收被引用的对象，哪怕内存不足的时候
- **软引用**：使用 SoftReference 修饰的对象被称为软引用，软引用指向的对象在内存要溢出的时候被回收
- **弱引用**：使用 WeakReference 修饰的对象被称为弱引用，只要发生垃圾回收，若这个对象只被弱引用指向，那么就会被回收
- **虚引用**：虚引用是最弱的引用，在 Java 中使用 PhantomReference 进行定义。虚引用中唯一的作用就是用队列接收对象即将死亡的通知

## 2.spring

#### 2.1bean的加载

![image-20220824104840132](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220824104840132.png)

beandifinition：bean的定义信息

#### 2.2简单的反射

1、获取class对象

```java
Class clazz = Class.forName()
Class clazz = 类名.class
Class clazz = 对象名。getClass()
```

2、Constructor ctor = clazz.getDeclaredConstructor();

3、Object obj = ctor.newInstance()

#### 2.3 bean的生命周期

第一个阶段：通过反射调用（**createBeanInstance方法**）bean的实例化，此时bean里面的属性都是初始值

第二个阶段：初始化，调用（**population方法**）给属性赋值，调用（**invokeAwareMethods方法**）给容器赋值，调用（**BeanPostProcessor方法**）执行前置处理方法、后置处理方法，其中aop就是这一步实现，前后置处理方法中间还有一个初始化方法，调用（**invokeInitMethods方法**），但是不常用

三：使用

四：销毁

#### 2.4 三级缓存

在DefaultSingletonBeanRegistry类中存在三个map：

![image-20220825155337787](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220825155337787.png)

一级缓存：singletonObjects：放成品对象

三级缓存：singletonFactories：放lambda表达式

二级缓存：earlySingletonObjects：放半成品对象

一级缓存无法解决循环依赖问题，一级缓存和二级缓存存储的是不同类型的对象，如果只有一级缓存，那么成品对象和半成品对象会放在一起，而半成品状态的对象是不能够直接暴露给其他对象做引用的，所以此时无法进行判断；

二级缓存可以解决某些情况下的循环依赖问题，但是有限制条件，不能出现代理对象

三级缓存中的map对象是ObjectFactory类型，可以传递一个lambda表达式进去，对外暴露的时机是没有办法确定的，所以只有对象被引用的时候才会判断是返回代理对象还是原始对象，所以lambda相当于是一种回调机制，在刚开始的是没有执行，在需要的时候才会调用lambda表达式来判断对外暴露的是原始对象还是代理对象。

********为什么业务中仍然可以遇见循环依赖问题：****spring提供的循环依赖解决方案就类似于我们在编写代码时候的异常处理过程，只能防止某些循环依赖问题，但是并不是所有的循环依赖问题都能解决掉

#### 2.5 IOC理解

控制反转，以前是需要什么对象自己创建什么对象，现在都交给IOC容器来控制对象，控制在实现过程中所需要的对象及需要依赖的对象；**反转**:在没有IOC容器前我们都是在对象中主动去创建依赖的对象，这是正转；而有了IOC之后，依赖的对象直接由IOC容器创建后注入到对象中，由主动创建变成了被动接受，这是反转

#### 2.6 spring的设计模式

![image-20220829171452092](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220829171452092.png)

#### 2.7 spring的事务管理

spring有编程式事务和声明式事务，声明式通过**@Transactional**来实现，当加了这个注解后，事务的自动功能就会关闭，由spring框架来进行控制。事务操作就是aop的一个核心实现，加了注解之后，spring会基于这个类生成一个代理对象，会将这个代理对象作为bean，当使用这个代理对象的方法时，如果有事务处理，会先把事务的自动提交给关闭，然后执行具体的业务逻辑，如果没有出现异常，那么代理逻辑就会提交，否则会直接回滚，当然用户可以控制对哪些异常进行回滚操作。

#### 2.8 spring事务什么时候会失效

![image-20220830143806307](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220830143806307.png)

#### 2.9 spring的bean作用域

![image-20220830150407916](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220830150407916.png)

## 3. Redis

#### 3.1 redis的架构形式

主从，哨兵，集群（多主多从）

#### 3.2 redis执行命令的死循环bug

在redis5.0之前存在这个问题，RANDOMKEY命令会随机获取一个key，如果key过期，会删除然后重新获取，若是存在大量过期key，就会陷入死循环，而且如果是在slave里查找因为不会删除key，会更严重。5.0后加了一个查询次数，超过次数就停止查询

slave不会自己清理过期的key，当一个key要过期时，master会先清理，然后发送一个DEL命令给slave，slave才会删除这个key。

#### 3.3 redis的持久化

**RDB：**根据设置的策略来生成RDB快照文件（如一秒内操作1000次）。

生成方式有**save**和**bgsave**。save命令是同步生成dump.rdb文件，不会消耗额外的内存，但是会阻塞后续的操作命令；bgsave是异步创建一个副本进程去生成dump.rdb文件，生成过程中有写操作时会同步修改副本数据，会消耗额外的内存。

**AOF：**将修改的每一条指令写进文件appendonly.aof里面（先写进os cache，每隔一段时间fsync到磁盘）

aof有三种选项：

![image-20220831141430162](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220831141430162.png)

**AOF重写**就是清除aof文件里面无用的指令，重新写入，减少内存占用量。

AOF和RDB混合持久化就是不再单纯的把内存数据写入aof文件，而是把重写之前的内存做快照处理，将此时的RDB文件和重写后的增量AOF文件存在一起；这样redis重启的时候就可以先加载RDB里面的文件再加载AOF文件

#### 3.4 redis线上数据如何备份

![image-20220831145812586](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220831145812586.png)

#### 3.5 redis的主从复制风暴

假如redis主节点有很多从节点，在某一时刻所有从节点同时连接主节点，那么主节点会同时把内存快照RDB发给多个从节点，导致主节点压力大。

#### 3.6 redis集群网络抖动导致频繁主从切换怎么处理

![image-20220831150847584](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220831150847584.png)

#### 3.7 redis集群为什么至少需要三个master节点

因为新master的选举需要大于半数以上的master节点同意才能选举成功

#### 3.8 redis主从复制的核心原理

![image-20220831154427747](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220831154427747.png)

#### 3.9 redis有哪些数据结构，分别有哪些典型的应用场景

![image-20220831154536694](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220831154536694.png)

#### 3.10 redis分布式锁的底层实现原理

https://zhuanlan.zhihu.com/p/150602212

![image-20220831154723538](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220831154723538.png)

![image-20220831154747129](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220831154747129.png)

#### 3.11 redis缓存穿透、缓存击穿、缓存雪崩

**缓存穿透：**redis和db里面都没有。***解决方案：***1.对参数做合法性校验；2. 将数据库中没有查到的数据写入到缓存中，有效期要设置的短一点；3.引入布隆过滤器，在访问redis之前判断数据是否存在，布隆过滤器只能加数据，不能减数据。

**缓存击穿：**缓存中没有，db中有，一般是出现在数据初始化以及key过期了的情况。这种问题在于，重新写入缓存需要时间，如果是在高并发场景下，过多的请求会瞬间到db上，给db造成很大的压力。***解决方案：***1.设置这个热点缓存永不过期。这时要注意在value当中包含一个逻辑上的过期时间，然后另起一个线程，定期扫描这些缓存，失效了就重新加入缓存中，可以保证数据的实时性。  2.加载db的时候，要防止并发 

**缓存雪崩：**缓存大面积失效，访问压力都到了db上。***解决方案：***1.把缓存的失效时间分散开。2.对热的数据设置永不过期

![image-20220831163157883](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220831163157883.png)



## 4. 多线程

#### 4.1 并发、并行、串行的区别

![image-20220901091702277](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220901091702277.png)

#### 4.2 并发的三大特性

1.原子性：在一个操作中cpu不可以中途暂停然后再调度，就是不被中断操作，要不全部执行完成，要不都不执行，就像转账。java中的i++不是线程安全的。

![image-20220901092805091](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220901092805091.png)

2.可见性：当多个线程访问同一个变量的时候，一个线程修改了变量的值，其他线程能够立即看到修改的值

3.有序性：虚拟机在进行代码编译时，对于那些改变顺序后不会对最终结果造成影响的代码，虚拟机不一定会按照我们写的代码顺序来执行，有可能会将他们重排序，这样就有可能会出现线程安全。

#### 4.3 线程的生命周期

![image-20220901094850624](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220901094850624.png)

#### 4.4 sleep()、wait()、join()、yield()的区别

**锁池：**所有需要竞争同步锁的线程都会放在锁池当中，比如当前对象的锁已经被其中一个线程得到，则其他线程都在锁池里等待释放锁后去竞争同步锁，当某个线程得到后会进入就绪队列进行等待cpu资源分配

**等待池：**当我们调用wait()方法后，线程会放到等待池中，等待池中的线程不会竞争锁，只有当调用notify()或notifyAll() 后等待池中的线程才会竞争锁，notify()是随机选出一个线程进入锁池，notifyAll()是将等待池中的所有线程都放入锁池。

![image-20220901100810979](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220901100810979.png)

![image-20220901100904687](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220901100904687.png)

#### 4.5 守护线程

为所有非守护线程也就是用户线程提供服务。例如GC垃圾回收线程就是，当程序中不再有任何运行的Thread，程序不会产生垃圾，垃圾回收线程会自动离开。因此由于守护线程的终止是自身无法控制的，所有不要把IO、File等重要操作逻辑分配给它。

#### 4.6 ThreadLocal的原理和使用场景

![image-20220901113852651](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220901113852651.png)

例如通过aop切面把user信息存入ThreadLocal中；执行数据库操作，一个方法下有俩个方法，为了保证事务，需要俩个方法从数据库连接池拿到的连接是同一个

ThreadLocal调用set方法赋值，方法里面建了一个**ThreadLocalMap**类，以当前的ThreadLocal对象为key，传的值为value，如果传多个值就建多个ThreadLocal对象。map的set方法里面建了一个**Entry[]**对象（键值对），把Entry放进map里面，Entry对象的父类是**WeakReference**（弱引用），好处是，当ThreadLocal对象置为null的时候这个对象就会被回收。如果不是弱引用，那么线程不停止ThreadLocalMap里面设置的kv就不会被回收，导致内存越占越多，造成**内存泄漏**，例如线程池。但是如果只是将ThreadLocal对象置为null，那么key会被回收，但是value如果是一个大的数组对象，还是不会被回收，造成内存泄露，所有最好还是使用**ThreadLocal.remove**去清楚map里的对象。

#### 4.7 锁的升级过程

![image-20220905104420687](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220905104420687.png)

![image-20220902155642840](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220902155642840.png)

#### 4.8 CAS的概念（轻量级锁）

![image-20220902155907251](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220902155907251.png)

线程读取到当前的值E，然后进行操作得到值V，之后拿E与当前的新值N进行比较，如果相等，说明没有别的线程进行操作，就赋值，如果不等就拿当前的新值N重新操作。

当然这样也会有有俩个问题：1.**aba问题**，就是第一次拿到手的时候值是a，但是回去比较之前被别的线程变成b再变成a，这样如果是引用就会出现问题，想要解决可以加个版本号来解决或者用ture来标记。jdk已经自带这些功能。2.**原子性问题**，比较发现未被修改然后进行修改的时候被设置成别的值了之后又被设置回当前线程的值。解决方法是底层的一个lock cmpxchg指令。最底层保证只有一个线程可以修改值，所有不管是乐观还是悲观最底层都是悲观实现。

#### 4.9 重量级锁

jvm处理不了的线程交给操作系统处理的都是重量级锁。重量级锁存在一个等待队列

在并发量比较高、临界区比较长的情况下使用重量级锁比较合适，因为等待线程自旋是需要消耗cpu的，而放进等待队列里面是不用的，所以这个时候使用轻量级锁不合适

## 4.2 JUC和AQS

#### 4.2.1 java的线程安全的主要原因就是java的内存模型

![image-20220909092907829](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220909092907829.png)

#### 4.2.2 volatile可以确保可见性，禁止指令重排序，可解决一写多读的并发安全

#### 4.2.3 Atomic类可以保证并发安全，底层是cas+自旋













## 5.kafka

#### 5.1 kafka的rebalance机制（针对年限高）

当消费者数量发生变化、消费超时或者topic个数发生变化的时候，消息需要重新分配，就是rebalance。kafka在rebalance过程中是阻塞的，不能进行读写

![image-20220906101105390](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220906101105390.png)

#### 5.2 kafka的副本同步机制

![image-20220906110842074](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220906110842074.png)

![image-20220906110943119](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220906110943119.png)

#### 5.3 kafka的架构设计

![image-20220906111129258](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220906111129258.png)

#### 5.4 kafka的pull和push的区别

pull是消费者主动去拉去，可批量或单条，可以设置不同的提交方式实现不同的传输语义，但可能拉取到空消息，通过参数设置，consumer拉取数据为空或者达到一定数量时进行阻塞；push是broker主动给消费者推送消息，推多少消费者接受多少，可能会造成网络堵塞、消费者压力大等问题。

#### 5.5 kafka高性能高吞吐的原因

1、磁盘顺序读：保证了消息的堆积

![image-20220906141408825](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220906141408825.png)

2、**零拷贝（kafka和RocketMQ都是通过这个来保证消息的高效读写）**：避免cpu将数据从一块存储拷贝到另外一块存储的技术。因为kafka只是传递消息，不是干预存储，所以可以零拷贝。

java中对零拷贝进行了封装，Mmap方式（适合小文件，1.5G~2G）通过MappedByteBuffer对象、transfile通过FileChannel进行操作。

RocketMQ使用的就是Mmap方式进行读写，如commitlog文件，一次1G，而Kafka的index日志文件也是mmap方式，但是其他日志文件没有使用，使用的是transfile方式将硬盘数据加载到网卡

![image-20220906141520524](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220906141520524.png)

3、分区分段+索引

![image-20220906141759889](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220906141759889.png)

4、批量压缩：多条消息一起压缩，降低了带宽

5、批量读写

6、直接操作page cache，而不是JVM、避免GC耗时及对象创建耗时，且读写速度更高，进程重启、缓存也不会丢失。

#### 5.6 kafka为什么比RocketMQ的吞吐量要高

![image-20220906142223072](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220906142223072.png)

#### 5.7 kafka消息丢失及解决方案

![image-20220906142923837](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220906142923837.png)

![image-20220906143004974](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220906143004974.png)

#### 5.8 kafka中zk的作用（老版本）

## 6.RocketMQ

#### 6.1 怎么保证消息顺序

生产者把一组有序的消息放到同一个队列中，而消费者一次消费整个队列中的消息

#### 6.2 如何选择消息中间件

![image-20220906164839856](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220906164839856.png)

#### 6.3 底层实现原理

![image-20220906165030177](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220906165030177.png)

#### 6.4 事务消息原理

![image-20220907094601764](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220907094601764.png)

![image-20220907094914448](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220907094914448.png)

![image-20220907094931971](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220907094931971.png)

#### 6.5 如何保证消息不丢失

![image-20220907095227488](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220907095227488.png)

#### 6.6 持久化机制

![image-20220907095902804](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220907095902804.png)

## 7. RabbitMQ

#### 7.1 架构设计

![image-20220907102247770](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220907102247770.png)

#### 7.2 持久化机制

![image-20220907105433311](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220907105433311.png)

#### 7.3 事务消息

![image-20220908095916572](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220908095916572.png)

#### 7.4 镜像队列原理

可以实现RabbitMQ的高可用

![image-20220908100405213](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220908100405213.png)

#### 7.5 死信队列、延迟队列原理

![image-20220908100755978](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220908100755978.png)

#### 7.6 消息的可靠性传输

![image-20220908101321397](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220908101321397.png)

## 8. Docker和k8s

#### 8.1 docker启动容器的生命周期

![image-20220908142517669](C:\Users\fwx1103156\AppData\Roaming\Typora\typora-user-images\image-20220908142517669.png)

#### 8.2 使用Dockerfile构建镜像

1、什么是Dockerfile：用来构建docker镜像的源码。dockerfile是一个文本文档（包含一些命令和参数），通过读取文档执行命令行自动构建镜像













































epic:重要战略举措

feature：业务系统下具体的功能

story：feature的一个细分，通常是一个迭代内可以交付的模块

task是store的一个拆分，是开发人员可以直接进行工作的力度
