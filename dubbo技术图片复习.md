
**dubbo**

![](https://raw.githubusercontent.com/yuxhe/techbmp/master/Dubbo/Dubbo%E5%8E%9F%E7%90%86%E5%9B%BE.png)
1）dubbo干的事本质就是个中介，服务提供方向dubbo注册中心注册服务地址，消费方从注册中心拉取地址，地址变更了dubbo注册中心会通知服务消费方拉取新的地址

2）服务消费方同服务提供方是直接连接的

3）注册中心挂掉，不影响服务消费方，除非服务提供方挂掉

4）服务消费方、服务提供方 两者中间还有个监控中心，是通过filter机制实现的，是http方式间隔段时间汇报的，比如汇报调用次数等


dubbo的工作流程是：

        privider到服务注册中心暴露服务接口；

        服务注册中心将服务地址列表推送给订阅了这某个服务的consumer；

        Consumer通过服务目录，选择一个privider进行调用；
		
        privider和consumer都接入monitor进行监控。

---

![](https://raw.githubusercontent.com/yuxhe/techbmp/master/Dubbo/Dubbo%E6%9C%8D%E5%8A%A1%E6%8F%90%E4%BE%9B%E5%8F%8A%E6%B3%A8%E5%86%8C%E6%96%B9.png)



服务注册及提供方：

服务方xml配置，消费方xml配置；两个服务名称(提供服务的及消费方)、两个地址（注册中心及服务提供方）
服务提供方暴漏的实现类 、 消费方依赖的实现类

<!-- dubbo提供方 -->

<!-- 1.名称-->
<dubbo:application name="babasport-service-product" />

<!--2.设置注册中心地址-->

<dubbo:registry address="192.168.200.128:2181,..." protocol="zookeeper" />


<!-- 3.dubbo://192.168.39.80:20800 设置提供服务的端口号-->

<dubbo:protocol port="20880"  name="dubbo" />

<!--4、暴漏实现类 -->

<dubbo:service interface="带包名全称"    ref="类名"/>

-------
<!-- dubbo消费方 -->
<!-- 1.名称-->
<dubbo:application name="babasport-service-console" />

<!--2.设置注册中心地址-->

<dubbo:registry address="192.168.200.128:2181,..." protocol="zookeeper" />

<!--4.调用此接口的实现类 -->

<dubbo:reference interface="带包名全称"  id="类名"/>

服务提供方4个配置、消费方三个配置

注册中心地址及端口2181，服务提供方

-- ----

Registry负载均衡  负载算法大多差不多哈

        Dubbo支持多种负载均衡策略，可用于多个provider的选择。

        在Reference注解的参数里，配置loadBalanace：

        1）Random：表示完全随机

        2）RoundRobin：表示一个一个换着来，轮循

        3）ConsistentHash：相同参数的请求，发到同一个provider。

Cluster高可用重试
        Dubbo支持多种高可用策略，可用于服务挂了之后的选择：

        在Reference注解的参数里，配置cluster：

        1）Failover：挂了以后，读另一个

        2）Failback：挂了以后，重发

        3）Forking：并发调多个Provider，效率高，只要有一个返回就行

        4）Broadcast：广播，一个一个调所有的Provider.

        另外，还可以配置retries，来明确失败了以后，重试几次。

第一种在实现类上加@Activate来加载，这种定义的Filter会变成Dubbo的原生filter调用链的一部分

第二种直接在consumer的@Reference注解里，或者provider的@Service注解里面，配置参数filter

链路跟踪 ->filter 实现filter traceid追踪

dubbo支持jdk\javassist两种动态代理，默认使用javassist；dubbo协议：主要用于高并发且数据量小的场景

 Dubbo支持多种序列化协议，包括hessian2\fastJson\JDK自带的等，一般用hessian2的序列化、反序列化效率最高



@Activate(group = {"provider","consumer"},order = -9999)

public class TraceFilter implements Filter {    

        @Override    

        public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {        

                String traceId= invocation.getAttachment("traceId");        

                if(StringUtils.isBlank(traceId)){            

                        traceId = UUID.randomUUID().toString();        

                }       

                RpcContext.getContext().setAttachment("traceId", traceId);        

                System.out.println(invoker.getInterface()+" Method: "+invocation.getMethodName()+" TraceId："+traceId);        

                return invoker.invoke(invocation);    

        }

}


服务降级
        dubbo自带了一套服务降级的方法，只要在consumer的@Reference注解里配置mock，然后实现一个Mock类就可以自动降级了

考虑幂等性
1）数据库方式
2）引入Redis，同样也为每一次执行请求增加一个唯一ID，在我们操作时，查看Redis里有没有这个ID的纪录，如果没有，往Redis里面插一条这个ID的记录，执行服务调用；如果有，则不执行


保证微服务执行的顺序性

   provider内部是用多线程来执行的，那我们为了保证多线程不会影响执行顺序，还可以将请求先放到一个队列里，强行排队执行

如何设计一个微服务框架
        一个完整的微服务框架至少需要包含如下几个部分：

        1）服务注册中心，保存服务目录，让consumer能够获取provider的情况；

        2）服务代理，通过动态代理代理consumer和provider，实现服务之间的调用；

        3）因为是分布式的，所以需要支持负载均衡策略，如loadBalance和ribbon；

        4）为保证高可用，还需要支持故障重试，和服务降级；

        5）服务之间通信，是通过RPC走，还是走restful，如果走RPC，那可以用netty通信；如果走restful，那就直接走httpClient调。

        6）如果走RPC，还需要序列化对象；而restful走的是json；

        7）看需不需要服务网关，统一服务的入口，如果要的话，还可以参与gateway或者zuul。



![](https://raw.githubusercontent.com/yuxhe/techbmp/master/Dubbo/Dubbo%E7%9B%B8%E4%BA%92%E9%97%B4%E5%AD%98%E5%9C%A8%E5%BF%83%E8%B7%B3%E4%BF%A1%E6%81%AF.png)


![](https://raw.githubusercontent.com/yuxhe/techbmp/master/Dubbo/Dubbo%E8%AE%BE%E7%BD%AE%E5%8F%82%E6%95%B0%E4%BC%98%E5%8C%96.png)

开发时的一个技巧

**服务方不注册**

<dubbo:registry address="N/A"  check="false" />

**消费方直接指定地址**

<dubbo:reference interface="带包名全称"  id="类名"

url="dubbo://127.0.0.1:20880"/>

<!-- 全局设置 超时设置 开发时设置大点 -->

<dubbo:consumer timeout="600000" check="false" /> 


-------------------
![](https://raw.githubusercontent.com/yuxhe/techbmp/master/Dubbo/spring%20boot%20Dubbo%E6%95%B4%E5%90%88%E5%93%88.png)

奇怪 服务提供方上配置有 负载平衡 轮询算法  采用的线程池

![](https://raw.githubusercontent.com/yuxhe/techbmp/master/Dubbo/spring_dubbo%E7%9A%84%E9%85%8D%E7%BD%AE%E5%BB%B6%E8%BF%9F%E9%98%9F%E5%88%97%E5%A4%9A%E7%BA%BF%E7%A8%8B%E6%8B%86%E5%8D%95.png)


![](https://raw.githubusercontent.com/yuxhe/techbmp/master/Dubbo/springboot_dubbo%E4%BD%BF%E7%94%A8%E5%8F%8A%E9%85%8D%E7%BD%AE.png)

spring boot dubbo配置


![](https://raw.githubusercontent.com/yuxhe/techbmp/master/Dubbo/%E9%9D%A2%E8%AF%95%E7%9A%84%E7%9F%A5%E8%AF%86%E7%82%B9%E5%8F%8A%E9%A2%98%E7%9B%AE.png)


http的结构  大的由\r\n分割的  小块内由空格分割

请求行 （method 空格 url 空格 version \r\n）
请求头  (键 空格 值 \r\n)
       ...
空白行  \r\n
请求体  body


websocket 全双工 二进制协议

sse是服务端推送客户端

**jwt服务端不保存任何会话数据，服务器变为无状态的**

tps上不去，考虑使用分布式，通常tps就200左右吧，单数据库上不去

线程start方式使得线程处于就绪状态

线程5个状态 新建、就绪、运行、终止、阻塞

开启线程运行的4种方式

execute 只接收runable接口run方法
submit 接收Callable接口的call方法


![](https://img2018.cnblogs.com/blog/715283/201908/715283-20190826110232418-222204028.jpg)

//重点代码！触发 java.util.concurrent.FutureTask#run 执行
task.run();

Callable任务被submit时，会生成一个FutureTask对象，封装Callable，在FutureTask的run方法里面执行Callable#call方法，并且调用

synchronized
synchronized 代码块执行完或异常时释放锁，主动释放性差

lock 是java5 后juc -- java.util.concurrent.locks下的 比较丰富

binglog三种类型 statement sql/row基于行模式 /mixed混合模式

解析日志数据

mysqlbinlog --base64-output=decode-rows -v mysql-bin.000001


