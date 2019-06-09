### Java

- JVM

**内存分配** ：运行时数据区，对象可达性分析，垃圾回收算法，垃圾收集器

**类加载** ：类加载过程，双亲委派

**Javac 编译器** ：语法糖

JIT编译器：运行时优化

**内存模型**： volatile 语义，final语义，Synchronized实现与优化（对象头-Mark Word（hashCode，份代年龄，锁标记位）-锁标记位），happens-before，as-if-serial，有序性。

- 源码

**集合** :  ArrayList（Vector，Stack），LinkedList，HashSet，LinkedHashSet，TreeSet，HashMap（重点），LinkedHashMap，TreeMap；

ConcurrentHashMap（重点），ConcurrentSkipListMap（跳表），ConcurrentSkipListSet，CopyOnWriteArrayList；

ArrayBlockingQueue，LinkedBlockingQueue，LinkedTranferQueue，PriorityBlockingQueue，SynchronousQueue，DelayQUeue（重点）

**并发框架** ：线程池（Executors，ThreadPoolExecutor，ScheduledThreadPoolExecutor，Future，CompletionService），并发工具类（**CountDownLatch**，**CyclicBarrier**，**Semaphore**，**Exchanger**，**Phaser**，**ThreadLocalRandom** ），

Fork/Join 框架

**包装类**（原子包装类）：Integer，Short，Long等，（语法糖，缓存）

**ThreadLocal** ： ThreadLocalMap，内存泄漏

**String**：（StringBuilder，StringBuffer）

### 框架

Spring : IOC（BeanFactory--有个map--map的key是BeanName，value是BeanDefinition，BeanDefinition就是Bean的信息），AOP，声明式事务

SpringMVC

mybatis

Spring boot

Spring Cloud

- [guava-cache](https://github.com/seaswalker/Spring/blob/master/note/guava-cache.md)

### 分布式

CAP定理，相关的服务治理，远程调用等基础知识，分布式ID，分布式锁，

分布式事务：两阶段提交，三阶段提交，补偿机制

，一致性hash

redis

MQ： kafka，rabbitMQ，rocketMQ

Dubbo

Zookeeper

### 网络

http/https

TCP/IP

IO模型，NIO，AIO

### 数据库

mysql

### 项目





Zookeeper 脑裂问题

redis 单线程，redis锁，多实例是否需要同步问题

canal binlog日志更新缓存

