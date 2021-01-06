高并发原则：

系统拆分，水平、垂直，衡量原则QPS及并发用户数

容错原则
    zookeeper,幂等性、心跳包、多级缓存
CAP
    一致性、可用性、分区容错

幂等性原则
    去重、不管执行多次操作，影响结果一样的
可扩展原则
     增加系统或设备
可维护、可监控原则
    有监控手段、及时发现问题所在

session共享
    1、session复制
    2、session sticky 定向到固定机器
    3、独立的session机器

优先考虑无状态服务
    内存、数据库、非结构化数据、文件、cdn;故障转移原则，比如跨机房

技术选型与数据库
    mysql单机承受有限，数据库读写分离以及集群，配合高可用redis集群（哨兵模式），心跳检测

缓存穿透、雪崩问题  即资源保护
    缓存穿透，拦截核查请求，验证码、IP限制等，布隆过滤器

还有可空现象 植入缓存中，防止打到数据库

缓存雪崩
    过期时间问题，应均匀分布，过期时间乘上随机数，队列、锁机制、控制并发

选择机器机型、集群、动静分离、分布式系统、限流、扩容、转发分流、服务降级

分布式id
   雪花算法、uuid、数据库自增id

-----------------------
wait,notify,notifyAll

Condition 接口提供 await使当前线程处于等待状态 ,signal、signalAll 为唤醒当前对象的等待线程

private  Lock lock=new ReentrantLock();//锁
private  Condition condition=lock.newCondition();

lock.lock()  lock.Unlock()   condition.await()   condition.signal()   condition.signalAll()

--------------------------------------------------

    Synchrionized加锁可能出现死锁，lock锁则不会

lock方式加锁会一直独占，除非被中断打断

trylock 尝试获取锁，获取不到返回false
trylock(long time,TimeUnit unit) 在某时间内不间断尝试获取锁，


#cas无锁算法，比较交换属于无锁，乐观锁，也不会出现死锁情况

#信号量semaphore 控制资源数、可保护资源

new Semaphore(3)

重写了Runnable对象

excutor.execute(()->{
  try {
        semp.acquire();//获取资源 同一时间只能3个线程允许放行
})

  semp.release();//记得归还资源


四大函数式接口

/**
 * Function函数式接口，有一个输入参数，有一个输出
 * 只要是函数型接口 可以用lambda表达式简化
 */

 //lambda表达式写法
  Function f = (str) ->{return str;};
  System.out.println(f.apply("a"));


断定型接口
/**
 * 断定型接口：有一个输入参数，返回值只能是 布尔值！
 */
 Predicate<String> predicate = s ->{
          return s.isEmpty();
        };
                System.out.println(predicate.test("a"));


消费型接口

/**
 * Consumer 消费型接口：只有输入，没有返回值
 */

Consumer<String> consumer = s -> {
            System.out.println(s);
        };
        consumer.accept("a");



供给型接口

/**
 * Supplier: 供给型接口，没有参数，只有返回值
 */

 Supplier supplier = ()->{
          return "null";
        };
        System.out.println(supplier.get());

BlockingQueue是一个接口，它的实现类有ArrayBlockingQueue、DelayQueue、 LinkedBlockingDeque、LinkedBlockingQueue、PriorityBlockingQueue、SynchronousQueue等，它们的区别主要体现在存储结构上或对元素操作上的不同，但是对于take与put操作的原理，却是类似的

阻塞与非阻塞
入队
offer(E e)：如果队列没满，立即返回true； 如果队列满了，立即返回false-->不阻塞

put(E e)：如果队列满了，一直阻塞，直到队列不满了或者线程被中断-->阻塞

offer(E e, long timeout, TimeUnit unit)：在队尾插入一个元素,，如果队列已满，则进入等待，直到出现以下三种情况：-->阻塞

被唤醒

等待时间超时

当前线程被中断

出队
poll()：如果没有元素，直接返回null；如果有元素，出队

take()：如果队列空了，一直阻塞，直到队列不为空或者线程被中断-->阻塞

poll(long timeout, TimeUnit unit)：如果队列不空，出队；如果队列已空且已经超时，返回null；如果队列已空且时间未超时，则进入等待，直到出现以下三种情况：

被唤醒

等待时间超时

当前线程被中断

remove()方法
remove()方法用于删除队列中一个元素，如果队列中不含有该元素，那么返回false；有的话则删除并返回true。入队和出队都是只获取一个锁，而remove()方法需要同时获得两把锁


#基于计数的线程

countDownLatch 实现的闭锁，主线程内await，子线程内countDown,如果>0 则一直等待

CyclicBarrier 多线程设置屏障，各线程到达后才放开，各子线程内最后一句await


callable 是重写call方法，runable重写run方法

funtureTask的get接收返回，get方法是个闭锁式的阻塞方法，该方法一直等待，直到有返回值；

#futureTask ： 
1. get: 判断是否执行完毕 -> 把线程信息存到Task waiters -> park挂起当前调用get方法的线程
2. run：调用callable.call方法 -> 改状态 -> 通知消费者(遍历waiters，unpark机制唤醒消费者)


AQs是juc的基石，直接或间接依赖AQS


#thrift相同的玩法也是，通过thrift工具，不同语言生成相关语法规范,拷入源代码目录下面，加入thrift的相关依赖包

#不同点thrift有相关的网络传输支持包

google的grpc没有传输支持,需要借助netty等

#disruptor 多线程并发框架，观察者模式，可配置线程间依赖关系，即组合执行先后顺序，核心生产者、消费者模式，内部概念SequenceBarrier、RingBuffer

自选、lock不是一个东西，自选次数，让出cpu















