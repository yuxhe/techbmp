
Eureka控制台参数说明

[https://www.cnblogs.com/linjiqin/p/10087462.html](https://www.cnblogs.com/linjiqin/p/10087462.html)


服务端集成安全角色

<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security</artifactId>
</dependency>

security:
  basic:
    enabled: true
  user:
    name: admin
    password: admin123



如果 eureka.client.fetchRegistry 设置成true（默认值true），Eureka client在启动时会从Eureka server获取注册信息并缓存到本地，之后只会增量获取信息（可以把 eureka.client.shouldDisableDelta 设置成false来强制每次都全量获取）。获取注册信息的操作也是一个异步任务，每隔30秒执行一次（通过 eureka.client.registryFetchIntervalSeconds 配置），如果操作失败，也是以2的指数形式延长重试时间，直到达到
eureka.client.registryFetchIntervalSeconds * eureka.client.cacheRefreshExecutorExponentialBackOffBound 这个上限，之后一直以这个上限值作为重试间隔，直至重新获取到注册信息，并且重新尝试获取注册信息的次数是不受限制的。
这些任务都是在com.netflix.discovery.DiscoveryClient中启动


@EnableEurekaServer   注册的服务端 在org.springframework.cloud.netflix.eureka.server内的注解


----------------
//客户端只需配置连接那个地方 用户名、密码


@EnableDiscoveryClient  公共包里面的  注册客户端 ，比服务端多
#客户端可配置用户名、密码访问 eurekaserver服务端
eureka:
  client:
    serviceUrl:
      defaultZone: http://admin:admin123@localhost:8761/eureka/


zuul就是 网关，反向代理，代理的实例id是什么，对应的前缀；注意均要同注册服务器达交到


zuul:                       #代理谁 及前缀的配置哈
  prefix: /api
  routes:
    client: 
      path: /client/**
      serviceId: client-service  

访问时同区优先策略在里面哈
eureka:                 #注意实例有个区的概念在里面  eureka.instance.metadataMap.zone
  instance:
    hostname: peer2
    metadataMap:
      zone: zone2


-------------------------
@RefreshScope          //允许实时刷新配置信息  客户端获取

--spring.profiles.active=zone1  区管理启动时配置的环境变量


spring cloud config 原理:一开始各客户端就从git或maven拉取配置信息，注册服务器也一开始去拉取，然后定时自动去刷新配置信息

可看出  其结构为  spring.cloud.config.uri  uri内可指定访问用户及密码哈
spring:  
  application:
    name: discovery-service
  cloud:
    config:
      uri: http://localhost:8889

config服务无端需要开启 @EnableConfigServer
spring.cloud.config 服务端 主要是 spring.cloud.config.server.git.uri或者maven方面的配置
spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: https://github.com/piomin/sample-spring-cloud-config-repo.git
          username: #${github.username}
          password: #${github.password}
          cloneOnStart: true

服务端需要包含 

<parent>
		<groupId>pl.piomin.services</groupId>
		<artifactId>sample-spring-cloud-config-bus</artifactId>
		<version>1.0-SNAPSHOT</version>
	</parent>
	<artifactId>sample-config-service</artifactId>

	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-config-server</artifactId>
		</dependency>
	</dependencies>

而客户端

<parent>
		<groupId>pl.piomin.services</groupId>
		<artifactId>sample-spring-cloud-config-bus</artifactId>
		<version>1.0-SNAPSHOT</version>
	</parent>
	<artifactId>sample-client-service</artifactId>

	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-eureka</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
	</dependencies>


-------------这种管理版本的思路可采用下
<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Dalston.SR4</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>


-- 手工方式 spring cloud  config 注册作为某实例存在
而客户端使用方式不同，向这样访问某实例方式 spring.cloud.config.discovery.serviceId

spring:  
  application:
    name: client-service
  cloud:
    config:
      discovery:
        enabled: true
        serviceId: config-server

--------------------------
获取某个所有的实例
private static final Logger LOGGER = LoggerFactory.getLogger(ClientController.class);

@Autowired
	private DiscoveryClient discoveryClient;
	
	@GetMapping("/ping")
	public List<ServiceInstance> ping() {
		List<ServiceInstance> instances = discoveryClient.getInstances("CLIENT-SERVICE");


--------------

程序内 配置要访问那些服务注解，同时要配置注解 负载算法实现 RestTemplate 

@SpringBootApplication
@RibbonClients({
	@RibbonClient(name = "account-service"),
	@RibbonClient(name = "customer-service"),
	@RibbonClient(name = "product-service")
})
public class OrderApplication {

	@LoadBalanced
	@Bean
	RestTemplate restTemplate() {
		return new RestTemplate();
	}


而具体访问代码：
@GetMapping("/withAccounts/{id}")
	public Customer findByIdWithAccounts(@PathVariable("id") Long id) {
		**Account[] accounts = template.getForObject("http://account-service/customer/{customerId}", Account[].class, id);**
		Customer c = repository.findById(id);
		c.setAccounts(Arrays.stream(accounts).collect(Collectors.toList()));
		return c;
	}

客户端：

<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-ribbon</artifactId>
		</dependency>

-----
网关内要做的东西就是  配置映射关系，同时开启就可以了；其他配置比如关于头的传递配置等；另外疑问 一个网关对应多个服务如何配置头的不同传递 ？

zuul:
  ignoredServices: '*'
  routes:
    account:
      path: /account/**
      url: http://localhost:8091
    customer:
      path: /customer/**
      url: http://localhost:8092
    order:
      path: /order/**
      url: http://localhost:8090
    product:
      path: /product/**
      url: http://localhost:8093


@SpringBootApplication
@EnableZuulProxy
public class GatewayApplication {

<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-zuul</artifactId>
		</dependency>
	</dependencies>

------------------------------

另一种采取注册服务模式，也可负载平衡方式哈代码一样，只是每个服务注册自己吧了

-----------------------------------------------------
客户端：
<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-ribbon</artifactId>
		</dependency>

-----------------------------


http://admin:896155yxhyhy@localhost:8000/crumbIssuer/api/json


docker run \
  --rm \
  -u root \
  -p 8000:8080 \
  -v jenkins-data:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "e:\jenkis":/home \
  jenkinsci/blueocean






docker run ^
  --rm ^
  -u root ^
  -p 8000:8080 ^
  -v jenkins-data:/var/jenkins_home ^
  -v /var/run/docker.sock:/var/run/docker.sock ^
  -v "e:/jenkis":/home ^
  jenkinsci/blueocean



docker run ^
  --rm ^
  -u root ^
  -p 8000:8080 ^
  -v jenkins-data:/var/jenkins_home ^
  -v /var/run/docker.sock:/var/run/docker.sock ^
  -v "e:/jenkis":/home ^
  -v "C:\Program Files\Java\jdk1.8.0_231":/jdk18 ^
  -v "D:\apache-maven-3.6.2":/maven ^
  -v "e:\Git\cmd":/git ^
  -v "C:\Program Files\Docker\Docker\resources":/docker24 ^
  jenkinsci/blueocean



docker run ^
  --name  jenkinsci ^
  -u root ^
  -p 8000:8080 ^
  -v jenkins-data:/var/jenkins_home ^
  -v /var/run/docker.sock:/var/run/docker.sock ^
  -v "e:/jenkis":/home ^
  -v "C:\Program Files\Java\jdk1.8.0_231":/jdk18 ^
  -v "D:\apache-maven-3.6.2":/maven ^
  -v "e:\Git\cmd":/git ^
  -v "C:\Program Files\Docker\Docker\resources":/docker24 ^
  jenkinsci/blueocean


替换内容
sed -i  's/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' /var/jenkins_home/updates/default.json

sed -i  's/http:\/\/www.google.com/http:\/\/www.baidu.com/g' /var/jenkins_home/updates/default.json



89ce6bb0eb3f41e5919eaed5ec48ac38

docker run ^
  --name  jenkinsyxh ^
  -u root ^
  -p 8000:8080 ^
  -v jenkins-data:/var/jenkins_home ^
  -v /var/run/docker.sock:/var/run/docker.sock ^
  -v "e:/jenkis":/home ^
  -v "C:\Program Files\Java\jdk1.8.0_231":/jdk18 ^
  -v "D:\apache-maven-3.6.2":/maven ^
  -v "e:\Git\cmd":/git ^
  -v "C:\Program Files\Docker\Docker\resources":/docker24 ^
  jenkins

docker cp jenkins.war 8c6e2d3b8a41:/usr/share/jenkins/jenkins.war

/usr/share/jenkins/jenkins.war

http://localhost:8000/restart

-rw-r--r-- 1 root root 70620203 Jul 17  2018 /usr/share/jenkins/jenkins.war



sed -i 's#https://updates.jenkins.io/download#https://mirrors.tuna.tsinghua.edu.cn/jenkins#g' default.json && sed -i 's#http://www.google.com#https://www.baidu.com#g' default.json

伪终端操作方式： docker exec -it jenkinsyxh /bin/bash

docker run --help

docker -H tcp://localhost:2375  ps

https://docs.docker.com/docker-for-windows/wsl/

https://docs.microsoft.com/en-us/windows/wsl/install-win10





docker run -d -p 8000:8080 -p 50000:50000 -v jenkins:/var/jenkins_home -v /etc/localtime:/etc/localtime --name jenkinsyxh  jenkins

cat /var/lib/jenkins/secrets/initialAdminPassword

cat /var/jenkins_home/secrets/initialAdminPassword
db44d575b9a740258ab56d627a71a5c1

http://192.168.220.153/

chmod u+w /etc/sudoers


docker cp jenkins.war jenkinsyxh:/usr/share/jenkins/jenkins.war


docker exec -u 0 -it jenkinsyxh /bin/bash

centos8
nmcli c reload

docker inspect jenkinsyxh | grep -i "ipaddr"

systemctl stop firewalld.service  防火墙问题是个坑


/usr/bin/mvn

/usr/share/maven


docker run -d -p 8000:8080 -p 50000:50000 -v jenkins:/var/jenkins_home -v  /usr/share/maven:/usr/share/maven  -v /etc/localtime:/etc/localtime --name jenkinsyxh  jenkins


方式一：修改配置文件（需停止docker服务）
1、停止docker服务
systemctl stop docker.service（关键，修改之前必须停止docker服务）
2、vim /var/lib/docker/containers/container-ID/config.v2.json
修改配置文件中的目录位置，然后保存退出

 "MountPoints":{"/home":{"Source":"/docker","Destination":"/home","RW":true,"Name":"","Driver":"","Type":"bind","Propagation":"rprivate","Spec":{"Type":"bind","Source":"//docker/","Target":"/home"}}}


yum list installed  maven

yum remove 


docker -H tcp://192.168.220.153:2375  ps


e820d5491f374030ace890ba5a750159

docker rmi sentry_web:latest  可强制删除某文件



1.卸载可能存在的CentOS自带的java环境

#rpm -qa|grep java

#rpm -qa|grep jdk

#rpm -qa|grep gcj 

2.如果有，卸载

#rpm -e --nodeps java--1.8.xxxxxxxxxxxxxxxxxxxxxxxxxxx

tart zxvf 解压文件 -C  目录

可以看出来jenkis 放到 docker 中运行会有很多的问题，并且windows的docker也有难料的问题哈
设置jenkins主目录，并启动

设置启动脚本的一种方式 可获取到当前运行的目录哈
#! /bin/sh
CurrentDir=`dirname $0`
export JENKINS_HOME=$CurrentDir/jenkins_home

echo ---------------------------------------------------------------------------
echo "                " start up jenkins
echo JENKINS_HOME=$JENKINS_HOME
echo ---------------------------------------------------------------------------

java -jar /Applications/Jenkins/jenkins.war


-----------------------------------
客户端 均衡模式：

@RibbonClients({
	@RibbonClient(name = "account-service"),
	@RibbonClient(name = "customer-service"),
	@RibbonClient(name = "product-service")
})

yml内配置ribbon无提示，顶级是服务名称 ....

turbine只能监控hystrix服务，不是hystrix服务，不能监控

hoverfly 是轻量级模拟访问微服务的框架


hystrix 熔断器；

@EnableHystrix 

@CachePut("accounts")
	@HystrixCommand(commandKey = "account-service.findByCustomer", fallbackMethod = "findCustomerAccountsFallback", 
		commandProperties = {//熔断策略从本地缓存获取数据  有这个就会被监控
			@HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "500"),
			@HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),
			@HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "30"),
			@HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "10000"),
			@HystrixProperty(name = "metrics.rollingStats.timeInMilliseconds", value = "10000")
		}
	)


account-service.findByCustomer 会登记访问信息 能画出一条曲线
--------------------------------
有个观察器

<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-turbine</artifactId>
			<version>1.4.7.RELEASE</version>
		</dependency>
	</dependencies>


@EnableTurbine     实现的思路就是主动轮询去抓取数据

http://turbine-hostname:port/turbine.stream

----------------------------------------------
<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-gateway</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>

gateway 需要 拉取注册的服务哈  网关不用注册到服务器端，网关发现主要目的就是解决服务的ip问题哈，不用写死就可负载了哈

网关重点就是 配置文件哈

server:
  port: ${PORT:8080}
  
spring:
  application:
    name: gateway-service
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true
      routes:
      - id: account-service
        uri: lb://account-service
        predicates:
        - Path=/account/**
        filters:
        - RewritePath=/account/(?<path>.*), /$\{path}
      - id: customer-service
        uri: lb://customer-service
        predicates:
        - Path=/customer/**
        filters:
        - RewritePath=/customer/(?<path>.*), /$\{path}
      - id: order-service
        uri: lb://order-service
        predicates:
        - Path=/order/**
        filters:
        - RewritePath=/order/(?<path>.*), /$\{path}
      - id: product-service
        uri: lb://product-service
        predicates:
        - Path=/product/**
        filters:
        - RewritePath=/product/(?<path>.*), /$\{path}
        
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
    registerWithEureka: false

----------------------------------------------------------

@EnableZipkinServer

-----------logstash 收集日志
<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-zipkin</artifactId>
		</dependency>

<dependency>
			<groupId>net.logstash.logback</groupId>
			<artifactId>logstash-logback-encoder</artifactId>
			<version>4.11</version>
		</dependency>

net.logstash.logback.appender.LogstashTcpSocketAppender


<appender name="STASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
		<destination>192.168.99.100:5000</destination>
		<encoder
			class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
			<providers>
				<mdc />
				<context />
				<logLevel />
				<loggerName />
				<pattern>
					<pattern>
						{
						"appName": "order-service"
						}
					</pattern>
				</pattern>
				<threadName />
				<message />
				<logstashMarkers />
				<stackTrace />
			</providers>
		</encoder>
	</appender>


java -DKAFKA_ZOOKEEPER=localhost:2181 -DSTORAGE_TYPE=elasticsearch -jar zipkin-server-*-exec.jar

zipkin 就是一个jar包，开源的支持几种存储方式 ，可传递 -D参数支持哈 默认端口9441

前端存储代码基本不用修改，只需加入相关的配置即可，假如是发现模式 基本不用改代码了哈


git reflog   q退出

git reset --hard  d22d366  回退到某个具体的版本
git branch -a 查看远程分支 ，git branch -r 列远程分支；

删除本地分支 git branch -d 分支名字 ;git branch -D webyang ；git  switch 分支名字

删除远程分支 git push origin --delete [branch_name]

直接克隆某分支
git clone -b feign_with_discovery  git@github.com:piomin/sample-spring-cloud-comm.git

获取远程某个分支哈
git checkout origin/feign_with_discovery -b feign_with_discovery

--spring.profiles.active=zone1  启动的程序参数哈

spring cloud sleuth穿透了很多组件  > ---------- zipkin ;对应的坐标 --对应的配置--- 对应的注解

Trace ID ，Span ID


