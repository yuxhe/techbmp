
![](https://raw.githubusercontent.com/yuxhe/techbmp/master/GC/GC%E7%AE%97%E6%B3%95%E5%8F%8A%E5%9E%83%E5%9C%BE%E6%94%B6%E9%9B%86%E5%99%A8.png)

![](https://raw.githubusercontent.com/yuxhe/techbmp/master/GC/Gc%20Roots.png)

调优就是减少full gc次数及执行时间


分代年龄存储在对象头中的，gc一次机会加一次，达到某次数后 ，移到老年代；

gc roots根节点 虚拟机本地变量、static成员变量、常量引用、本地方法栈的变量等

打印gc日志  -XX:+PrintGCDetails  -XX:+PrintGCTimeStamps
           -XX:+PrintGCDateStamps -Xloggc:./gc.log

old eden from to  (2:1 ,8:1:1)比例关系


![](https://raw.githubusercontent.com/yuxhe/techbmp/master/GC/Gc%E8%B0%83%E4%BC%98-Gceasy%E5%88%86%E6%9E%90%E5%B7%A5%E5%85%B7-%E5%90%9E%E5%90%90%E9%87%8F.png)


-XX:+UseParNewGc 用于新生代的垃圾回收  默认的

-XX:+UseG1Gc  G1设计目标减少Full GC

-XX:+UseConcMarkSweepGc  CMs收集器  主要用于老年的回收

