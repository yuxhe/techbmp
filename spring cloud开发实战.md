关于jpa的使用哈

import com.cd826dong.clouddemo.user.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;

/**
 * 用户信息repository
 *
 * @author CD826
 * @since 1.0.0
 */
public interface UserRepository extends JpaRepository<User, Long> {

}



/**
 * 获取用户列表
 * @return
 */
@RequestMapping(method = RequestMethod.GET)
@ApiOperation(value = "获取用户分页数据", notes = "获取用户分页数据", httpMethod = "GET", tags = "用户管理相关Api")
@ApiImplicitParams({
        @ApiImplicitParam(name = "page", value = "第几页，从0开始，默认为第0页", dataType = "int", paramType = "query"),
        @ApiImplicitParam(name = "size", value = "每一页记录数的大小，默认为20", dataType = "int", paramType = "query"),
        @ApiImplicitParam(name = "sort", value = "排序，格式为:property,property(,ASC|DESC)的方式组织，如sort=firstname&sort=lastname,desc", dataType = "String", paramType = "query")
})
public List<UserDto> findAll(Pageable pageable){
    Page<User> page = this.userService.getPage(pageable);
    if (null != page) {
        return page.getContent().stream().map((user) -> {
            return new UserDto(user);
        }).collect(Collectors.toList());
    }
    return Collections.EMPTY_LIST;
}

-------------------------
以下这个里面有包含jdbc 以及 hibernate的包哈 ，以及事务等

 <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>


JPA注解简介：

JPA注解	说明
@Entity	标记为实体类
@Table	对应表名，不写默认为类名，首字母小写
@Id	声明一个属性映射到主键的字段
@GeneratedValue	设定主键生成策略，自增可用AUTO或IDENTITY
@Column	表明属性对应到数据库的一个字段。
@ManyToOne	多对一，默认fetch = FetchType.EAGER
@JoinColumn	与@ManyToOne搭配，指明外键字段
@OneToMany	一对多，mappedBy这里以声明Many端的对象(User实体)的属性(department)提供了对应的映射关系


Repository
Repository是Spring Data的核心概念。提供有如下接口：

CrudRepository：提供基本的增删该查，批量操作接口。
PagingAndSortingRepository：集成CrudRepository，提供附加的分页查询功能。
JpaRepository：专用于JPA，是重点。
public interface UserRepository extends JpaRepository<User, Integer>{
    // 不需要我们编写实现
}


如果想做分页查询可通过 findAll(Pageable pageable) ，利用Pageable进行分页查询。构造Pageable可使用其实现类PageRequest，Sort用来指定排序方式。示例代码如下：

// 分页查询第一页用户，限制每页10条记录，按创建时间降序排序
int page = 0, size = 10;
PageRequest pageRequest = PageRequest.of(page, size, Sort.by(Sort.Order.desc("gmtCreate")));
Page<User> userPage = userRepository.findAll(pageRequest);


如果即想做分页查询又需要条件查询，可以再实现接口 JpaSpecificationExecutor<T> ，使用 Specification 设置条件。示例代码如下：


如果想做分页查询可通过 findAll(Pageable pageable) ，利用Pageable进行分页查询。构造Pageable可使用其实现类PageRequest，Sort用来指定排序方式。示例代码如下：

// 分页查询第一页用户，限制每页10条记录，按创建时间降序排序，同时要求用户的id大于2
PageRequest pageRequest = PageRequest.of(0, 10, Sort.by(Sort.Order.desc("gmtCreate")));
Specification<User> specification = (root, query, builder) -> {
    /*
        root：实体对象引用，这里是user
        query：规则查询对象
        builder：规则构建对象
     */
    List<Predicate> predicates = new ArrayList<>();
    predicates.add(builder.greaterThan(root.get("id").as(Integer.class), 2));
    return builder.and(predicates.toArray(new Predicate[predicates.size()]));
};
userPage = userRepository.findAll(specification, pageRequest);

查询关键字：
Keyword	Sample	JPQL snippet
Distinct	findDistinctByLastnameAndFirstname	select distinct …​ where x.lastname = ?1 and x.firstname = ?2
And	findByLastnameAndFirstname	… where x.lastname = ?1 and x.firstname = ?2
Or	findByLastnameOrFirstname	… where x.lastname = ?1 or x.firstname = ?2
Is, Equals	findByFirstname,findByFirstnameIs,findByFirstnameEquals	… where x.firstname = ?1
Between	findByStartDateBetween	… where x.startDate between ?1 and ?2
LessThan	findByAgeLessThan	… where x.age < ?1
LessThanEqual	findByAgeLessThanEqual	… where x.age <= ?1
GreaterThan	findByAgeGreaterThan	… where x.age > ?1
GreaterThanEqual	findByAgeGreaterThanEqual	… where x.age >= ?1
After	findByStartDateAfter	… where x.startDate > ?1
Before	findByStartDateBefore	… where x.startDate < ?1
IsNull, Null	findByAge(Is)Null	… where x.age is null
IsNotNull, NotNull	findByAge(Is)NotNull	… where x.age not null
Like	findByFirstnameLike	… where x.firstname like ?1
NotLike	findByFirstnameNotLike	… where x.firstname not like ?1
StartingWith	findByFirstnameStartingWith	… where x.firstname like ?1 (parameter bound with appended %)
EndingWith	findByFirstnameEndingWith	… where x.firstname like ?1 (parameter bound with prepended %)
Containing	findByFirstnameContaining	… where x.firstname like ?1 (parameter bound wrapped in %)
OrderBy	findByAgeOrderByLastnameDesc	… where x.age = ?1 order by x.lastname desc
Not	findByLastnameNot	… where x.lastname <> ?1
In	findByAgeIn(Collection<Age> ages)	… where x.age in ?1
NotIn	findByAgeNotIn(Collection<Age> ages)	… where x.age not in ?1
TRUE	findByActiveTrue()	… where x.active = true
FALSE	findByActiveFalse()	… where x.active = false
IgnoreCase	findByFirstnameIgnoreCase	… where UPPER(x.firstname) = UPPER(?1)



如果即想做分页查询又需要条件查询，可以再实现接口 JpaSpecificationExecutor<T> ，使用 Specification 设置条件。示例代码如下：

// 分页查询第一页用户，限制每页10条记录，按创建时间降序排序，同时要求用户的id大于2
PageRequest pageRequest = PageRequest.of(0, 10, Sort.by(Sort.Order.desc("gmtCreate")));
Specification<User> specification = (root, query, builder) -> {
    /*
        root：实体对象引用，这里是user
        query：规则查询对象
        builder：规则构建对象
     */
    List<Predicate> predicates = new ArrayList<>();
    predicates.add(builder.greaterThan(root.get("id").as(Integer.class), 2));
    return builder.and(predicates.toArray(new Predicate[predicates.size()]));
};
userPage = userRepository.findAll(specification, pageRequest);


.@Query查询
使用 @Query 注解标记在Repository方法上可以使用JPQL进行查询，示例代码如下：

@Query(value = "select u from User u where u.id = ?1")
User getOneByJpql(Integer id);

也可以使用原生SQL查询，不过要把设置 nativeQuery = true，示例代码如下：

@Query(value = "select * from user where user_id =:id", nativeQuery = true)
User getOneBySql(@Param("id") Integer id);

注意：Idea中使用JPA原生sql查询时需要配置好Database的数据源，指定数据库方言，指定Schema。
否则可能遇到异常报错如：
SQL dialect is not configured
This inspection performs unresolved SQL references check

现在重新来看上面分页查询加条件查询，利用@Query是如何轻松的解决的。示例代码如下：

@Query(value ="select u from User u where u.id > :id")
Page<User> queryUsers(@Param("id") Integer id, Pageable pageable);

// 方法中调用查询
Page<User> userPage = userRepository.queryUsers(2, PageRequest.of(0, 10, Sort.by(Sort.Order.desc("gmtCreate"))));

6
@Query允许SQL更新、删除语句，但必须搭配 @Modifying 使用。示例代码如下：

@Modifying
@Query(value = "update User u set u.name = ?1 where u.id = ?2")
int update(String name, Integer id);


----------------------------------------

**集成Mybatis**
pom文件引入mybatis支持：

<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.2.2</version>
</dependency>

配置文件指定mapper文件所在处，例如我的：

mybatis:
  mapper-locations: classpath:mapper/*Mapper.xml
启动类指定mapper java文件所在包，例如我的：

@MapperScan(basePackages = {"pers.hdh.d_database.mapper"})
@SpringBootApplication
public class DDatabaseApplication {
    //...
}
然后在指定包下创建Mapper Java文件，mapper文件夹下创建xml文件即可

--------------------------------------------------------


解决办法是在parent项目的pom文件中使用<dependencyManagement></dependencyManagement>将springmvc.jar管理起来，如果有哪个子项目要用，那么子项目在自己的pom文件中使用


定义依赖一定是有版本的，没有的化那就是来自父母的版本管理哈
<dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka-server</artifactId>
        </dependency>
    </dependencies>


而下面这种是定义自身的属性哈，没有的必要属性肯定来自父母哈
 <artifactId>service-discovery</artifactId>
    <packaging>jar</packaging>
    <name>SpringCloud Demo Projects(HelloCloud) -- Eureka Server</name>

父母中的一些属性

 <groupId>cd826dong.cloud</groupId>
    <artifactId>springcloud-book-parent</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>pom</packaging>
    <name>SpringCloud Demo Projects(HelloCloud) -- Parent Pom</name>


父母中定义以下dependencyManagement这样的则会影响儿子们哈，也就是说父母帮助着管理哈

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

而下面这样搞，意思是绝对直接来自父母

<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.2.RELEASE</version>
</parent>

而这样搞呢

<parent>
        <groupId>cd826dong.cloud</groupId>
        <artifactId>springcloud-book-parent</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <relativePath>../parent</relativePath>   <!--引用路径或继承啥 注意前三个必要属性哈 -->
    </parent>


<parent></parent>，parent标签中写上parent项目的pom坐标就可以引用到


<packaging>jar</packaging>    为pom或jar哈


feign.hystrix.enabled: true作用
Feign使用Hystrix

----------------------------------------------------------------------------------------------

WaitTimeInMsWhenSyncEmpty(*)

在Eureka服务器获取不到集群里对等服务器上的实例时，需要等待的时间，单位为毫秒，默认为1000 * 60 * 5


显示配置ribbon

@Configuration
public class UserRibbonConfiguration {
//    String listOfServers = "http://localhost:2100," +
//            "http://localhost:2110,http://localhost:2100";

    String listOfServers = "http://localhost:2100";
    @Bean
    public ServerList<Server> ribbonServerList() {
        List<Server> list = Lists.newArrayList();
        if (!Strings.isNullOrEmpty(listOfServers)) {
            for (String s: listOfServers.split(",")) {
                list.add(new Server(s.trim()));
            }
        }
        return new StaticServerList<>(list.toArray(new Server[0]));
    }

    @Bean
    public IPing ribbonPing() {
        return new PingUrl(false, "/cs/hostRunning");
    }

    @Bean
    public IRule ribbonRule() {
        return new ZoneAvoidanceRule();
    }
}


调用服务配置

@Configuration
@RibbonClient(name="userservice", configuration=UserRibbonConfiguration.class)
public class RibbonConfiguration {

}


@EnableDiscoveryClient
@SpringBootApplication
public class Application {
    @Bean(value = "restTemplate")
    @LoadBalanced
    RestTemplate restTemplate() { //这个已负载算法了
        return new RestTemplate();
    }

    @Primary
    @Bean(value = "lbcRestTemplate")
    RestTemplate lbcRestTemplate() {
        return new RestTemplate();
    }

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}



 protected UserDto loadUser(Long userId) {
        UserDto userDto = this.restTemplate.getForEntity("http://USERSERVICE/users/{id}", UserDto.class, userId).getBody();
        if (userDto != null) {
            this.logger.debug("I came from server : {}", userDto.getUserServicePort());
        }
        return userDto;
    }

    public UserDto loadUserEx(Long userId) {
        ServiceInstance instance = this.loadBalancerClient.choose("USERSERVICE");//选择获得实例 采用轮询的算法
        //URI userUri = URI.create(String.format("http://%s:%s/users/{id}", instance.getHost(), instance.getPort()));
        String url=String.format("http://%s:%s/users/{id}", instance.getHost(), instance.getPort()) ;
        logger.info("Target service uri = {}. ", url);
        UserDto userDto = this.lbcRestTemplate.getForEntity(url, UserDto.class, userId).getBody();
        if (userDto != null) {
            this.logger.debug("I came from server : {}", userDto.getUserServicePort());
        }
        return userDto;
    }

----------------------------------------------------
Feign的使用哈  ；首先是ribbon 及 Feign的坐标引入
启动地方
@EnableDiscoveryClient
@EnableFeignClients


使用的service地方
/**
 * 用户服务，使用Feign实现
 *
 * @author CD826(CD826Dong@gmail.com)
 * @since 1.0.0
 */
@FeignClient("USERSERVICE")
public interface UserService {//注意使用的地方采用大写哈  USERSERVICE
    @RequestMapping(value = "/users", method = RequestMethod.GET)
    List<UserDto> findAll();

    @RequestMapping(value = "/users/{id}", method = RequestMethod.GET)
    UserDto load(@PathVariable("id") Long id);
}

使用的地方只需如下引用，调用同以往一样：

@Autowired
    private UserService userService;


public UserDto loadUserByFeign(Long userId) {
        return this.userService.load(userId);
    }

----------------------------------------------------

 $.ajax({
		                url: "dict/list",
						type: 'POST',
		                dataType: 'json',  
		                data: {'_method':'DELETE', 'codeGroupIds':'abcd', 'codeIds':'abcd'},  
		                success: function (reps) {
		                	if(reps.success){
		                		loadDictList();
				           	}
				           	// 弹出提示信息 
							showMessage(reps.success, reps.tipMsg);
		                },
		                error: function (e){
						}
					});
即在 data 参数中添加  '_method':'DELETE' 配置 且Type为POST，由此可以看出，其实DELETE是POST的一种衍生方式；

如上配置后，Controller端就可以正常使用了 

	@RequestMapping(value="list", method=RequestMethod.DELETE)
	public String dictDelete(@RequestParam(value="codeGroupIds", required=true) String codeGroupIds, 
			@RequestParam(value="codeIds", required=false) String codeIds, 
			HttpServletRequest request, HttpServletResponse response) {
		logger.info("RequestMapping : "+REQUEST_URL+" -> DELETE");
		
		System.out.println(codeGroupIds);
		System.out.println(codeIds);
		return null;
	}
-------------------------------------------------
这个功能比较实用   得注意下这个配置  有时搞不懂呀
management.port=58888
management.context-path=/actuator

management.endpoints.web.exposure.include=*    //开启
management.security.enabled=false   //这个属性重要
endpoints.shutdown.enabled=true     //开启shutdown 默认没打开  其他方式类似 ，2.x的方式有点变化

http://localhost:58888/actuator/autoconfig   自动配置信息报告
http://localhost:58888/actuator/configprops  配置属性报告
http://localhost:58888/actuator/env 
http://localhost:58888/actuator/beans 
http://localhost:58888/actuator/health 
http://localhost:58888/actuator/mappings  可用于获取应用接口哈
http://localhost:58888/actuator/metrics  重要指标硬件信息 包含垃圾回收等

http://localhost:58888/actuator/info  自定义的配置信息哈  info.开头的配置信息
http://localhost:58888/actuator/trace  可跟踪信息哈

curl -X POST localhost:58888/actuator/shutdown  可关闭应用

-----------------------------------------------------------------
 <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>

# 安全配置
security.basic.enabled=true
security.user.name=cd826
security.user.password=pwd


eureka.client.service-url.defaultZone=http://cd826:pwd@localhost:8260/eureka

一个短信登录验证过程

![](https://user-gold-cdn.xitu.io/2019/9/2/16cf275bc967fc81?imageView2/0/w/1280/h/960/format/web/ignore-error/1)


Oauth2

https://juejin.cn/post/6844903942543900679

![](https://user-gold-cdn.xitu.io/2019/9/16/16d38a4606bb13d4?imageslim)


基于JWT的单点登陆(SSO)开发及原理解析

https://juejin.cn/post/6844903942820888590


https://github.com/BUG9/spring-security


https://juejin.cn/post/6844903952643784711

Spring Security 解析(七) —— Spring Security Oauth2 源码解析

![](https://user-gold-cdn.xitu.io/2019/9/25/16d6746e016c8ac7?imageView2/0/w/1280/h/960/format/web/ignore-error/1)


1）Turbine通过Hystrix Dashboard来可视化监控数据

是客户端模式，启动 Turbine服务，监控哪个服务；监控采样及负责推送信息

Hystrix Stream: http://localhost:8280/turbine.stream

<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-turbine</artifactId>
        </dependency>

turbine.app-config=PRODUCTSERVICE
turbine.cluster-name-expression="default"
turbine.combine-host-port=true

@EnableTurbine
@EnableDiscoveryClient
@SpringBootApplication

2）熔断监控

<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-hystrix-dashboard</artifactId>
        </dependency>

开启 监控  http://localhost:2200/hystrix

@EnableHystrixDashboard   这个是首页
@EnableCircuitBreaker     这个是开启服务
@EnableDiscoveryClient    发现模式
@SpringBootApplication
public class Application {

为sse单向的服务端推送模式，推向浏览器
Content-Type: text/event-stream;charset=UTF-8

service中处理哈

 @Override
    @HystrixCommand(fallbackMethod = "findAllFallback")
    public List<UserDto> findAll() {
        UserDto[] userDtos = this.restTemplate.getForObject(
                "http://USERSERVICE/users/", UserDto[].class);
        return Arrays.asList(userDtos);
    }

 protected List<UserDto> findAllFallback() {
        List<UserDto> userDtos = new ArrayList<>();
        userDtos.add(new UserDto(1L, "zhangSan_static", "/users/avatar/zhangsan.png"));
        userDtos.add(new UserDto(2L, "lisi_static", "/users/avatar/lisi.png"));
        userDtos.add(new UserDto(3L, "wangwu_static", "/users/avatar/wangwu.png"));
        userDtos.add(new UserDto(4L, "yanxiaoliu_static", "/users/avatar/yanxiaoliu.png"));
        return userDtos;
    }

----------------------------- 这种模式 turbine  <->  hystrix
@EnableDiscoveryClient 这个是基础哈


feign_hystrix ,feign中运用断路器，需要设置如下开启：

feign.hystrix.enabled=true

同时应用上  @EnableCircuitBreaker 可不用配置哈


@FeignClient(name = "USERSERVICE", fallback = UserServiceFallback.class)
public interface UserService {
    @RequestMapping(value = "/users", method = RequestMethod.GET)
    List<UserDto> findAll();

    @RequestMapping(value = "/users/{id}", method = RequestMethod.GET)
    UserDto load(@PathVariable("id") Long id);
}


实现接口就可以了：
@Component
public class UserServiceFallback implements UserService {

    @Override
    public List<UserDto> findAll() {
        return this.findAllFallback();
    }

    @Override
    public UserDto load(Long id) {
        return this.loadFallback(id);
    }

    protected List<UserDto> findAllFallback() {
        List<UserDto> userDtos = new ArrayList<>();
        userDtos.add(new UserDto(1L, "zhangSan_static", "/users/avatar/zhangsan.png"));
        userDtos.add(new UserDto(2L, "lisi_static", "/users/avatar/lisi.png"));
        userDtos.add(new UserDto(3L, "wangwu_static", "/users/avatar/wangwu.png"));
        userDtos.add(new UserDto(4L, "yanxiaoliu_static", "/users/avatar/yanxiaoliu.png"));
        return userDtos;
    }

    protected UserDto loadFallback(Long id) {
        return new UserDto(id, "Anonymous1", "default1.png");
    }

}
-----------------------------------------------------------------

@EnableZuulProxy   @EnableZuulServer

zuul有路由及负载均衡的作用

Zuul中定义了四种标准的过滤器类型，这些过滤器类型对应于典型的生命周期。

PRE: 这种过滤器在请求被路由之前调用。可利用其实现身份验证等

ROUTING: 这种过滤器将请求路由到微服务，用于构建发送给微服务的请求，并使用Apache Http Client或者Netflix Ribbon请求微服务

POST: 这种过滤器在路由到微服务以后执行，比如为响应添加标准的HTTP Header，收集统计信息和指标，将响应从微服务发送到客户端等

ERROR: 在其他阶段发生错误时执行该过滤器

除了默认的过滤器类型，Zuul还允许创建自定义的过滤器类型

服务降级
@Component
public class UserServiceFallbackProvider implements ZuulFallbackProvider {//服务降级

路由映射配置

#zuul.prefix=/api
#zuul.routes.userservice = /user/**
#zuul.ignored-services=userservice


复杂的就是过滤器的一些处理哈 ，过滤器、处理头等，以及一些权限控制等哈

@EnableZuulProxy    // 前者是后者的增强 @EnableZuulServer ，加入了hystrix及服务发现功能

EnableZuulProxy 注解包括了hystrix 以及 服务发现哈

 <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zuul</artifactId>
        </dependency>

以上这个内包含了 hystrix 以及ribbon哈
因此可以进行健康检查，如下的监控，因为有hystrix的逻辑处理：

http://localhost:8280/hystrix.stream
------------------------------------------

#config 的配置中心  注意事项， --- 分析是如何读取的

spring-cloud-config  内部集成了 spring-boot-starter-actuator  读 AKt以儿拖

bootstrap.properties 配置文件上下文，优先级高、同时通常不会被覆盖；优先于application.properties

数据优先级高

以下表明加入注册中心，同时以为着 http://configserver 可以被访问到

spring.application.name=configserver
eureka.client.service-url.defaultZone=http://localhost:8260/eureka


----对应客户端读取的对应配置
spring.cloud.config.uri= http://configserver

spring.application.name=productservice
#spring.cloud.config.profile=dev    这个是启用这个配置哈，会查找带了这个 -dev结尾的配置文件,注意可逗号分割包含多个

----------------
management.security.enabled=false
http://localhost:2200/env  能查看配置文件

#http://localhost:2200/refresh  可手动刷新git的配置文件哈

http://localhost:8888/productservice/dev

spring.cloud.config.server.git.basedir=tmp/  表示本地缓存目录 ，如果远程git不可访问则利用本地缓存目录
启用 某个文件或段
spring:
    profiles:
        active: dev
        #active: prod

encrypt.key=XXXX   这个是非对称加解密用到的key 哈

spring-cloud-config-server
spring-cloud-starter-config
------------------------------------------------

Sleuth  读音 [sluːθ] 分布式系统跟踪

#服务端  集成包启动出一个单独的服务哈
第三方 zipkin 的使用 ，开启注解  @EnableZipkinServer ，同时 zipkin 未注册成 eureka服务，zipkin还是必须要配对哈

 <dependency>
            <groupId>io.zipkin.java</groupId>
            <artifactId>zipkin-server</artifactId>
        </dependency>
        <dependency>
            <groupId>io.zipkin.java</groupId>
            <artifactId>zipkin-autoconfigure-ui</artifactId>
            <scope>runtime</scope>
        </dependency>


#客户端
客户端使用则引入坐标

<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zipkin</artifactId>
        </dependency>


# zipkin配置
spring.zipkin.base-url=http://localhost:8240
spring.sleuth.sampler.percentage=1.0    #采样率设置哈


http://localhost:8240/   可查看调用消耗的时间关系
---------------------------------------------------
#Logstash 是一个应用程序日志、事件的传输、处理、管理和搜索的平台，Logstash 现在是 ElasticSearch 家族成员之一

需要指定参数启动 -f 配置文件
logstash.bat  -f H:\logstash-7.10.0\bin\logstash.conf

配置文件尽然忘了：

input {
    tcp {
	   port=>5000
	   codec=>json
	}
    #stdin{
    #}
} 
 
output {
    stdout{
    }
}

---


日志收集配置   客户端  tcp--->- logstash服务端  -- 后面可接入es等

java端只需引入客户端坐标哈 logstash-logback-encoder

 <!-- Logstash -->
        <dependency>
            <groupId>net.logstash.logback</groupId>
            <artifactId>logstash-logback-encoder</artifactId>
            <version>4.11</version>
        </dependency>


logback-spring.xml 内的内容配置哈

<!-- Logstash日志Appender -->  
    <appender name="logstash" 
        class="net.logstash.logback.appender.LogstashTcpSocketAppender">  
        <param name="Encoding" value="UTF-8"/>
        <!-- 配置Logstash服务器的地址 -->    
        <remoteHost>localhost</remoteHost>
        <port>5000</port>
        <!-- 通过日志编码器，将日志转换成JSON字符串 -->  
        <encoder class="net.logstash.logback.encoder.LogstashEncoder" />  
    </appender>  


更复杂的日志过滤,可自定义输出的内容

<appender name="STASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
		<destination>localhost:5000</destination>
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
						"appName": "account-service"
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


-------------------Linux 、window下两者路径有差别
windows下注意启动有坑
zookeeper-server-start.bat  ../../config/zookeeper.properties
kafka-server-start.bat  ../../config/server.properties 


 @Bean
    public RestTemplate restTemplate(RestTemplateBuilder builder) {
        return builder.build();
    }


 @Primary
    @Bean(value = "lbcRestTemplate")
    RestTemplate lbcRestTemplate() {
        return new RestTemplate();
    }

------------------
@Autowired
    private LoadBalancerClient loadBalancerClient;

    @Autowired
    private  RestTemplate restTemplate;


 @Autowired
    @Qualifier(value = "lbcRestTemplate")
    private RestTemplate lbcRestTemplate;


--------------
@Autowired
    private LoadBalancerClient loadBalancerClient;

    @Autowired
    private  RestTemplate restTemplate;


 // Redis中不存在，那么从远程获取  手动调用
        ServiceInstance instance = this.loadBalancerClient.choose("USERSERVICE");//选择获得实例 采用轮询的算法
        //URI userUri = URI.create(String.format("http://%s:%s/users/{id}", instance.getHost(), instance.getPort()));
        String url=String.format("http://%s:%s/users/{id}", instance.getHost(), instance.getPort()) ;
        logger.info("Target service uri = {}. ", url);
        //UserDto userDto = this.lbcRestTemplate.getForEntity(url, UserDto.class, userId).getBody();

        ResponseEntity<UserDto> restExchange =
                this.restTemplate.exchange(
                        url,//"http://USERSERVICE//users/{userId}"
                        HttpMethod.GET,
                        null,
                        UserDto.class,
                        userId);
        UserDto user = restExchange.getBody();
        if (null != user) {
            // 获取后将用户信息进行缓存
            this.userRedisRepository.saveUser(user);
        }
        return user;

------------------------------
Feign方式的调用 ，feign内包含ribbon的坐标，只有客户端一说 spring-cloud-starter-feign

@FeignClient("USERSERVICE")
public interface UserRemoteClient {
    @RequestMapping(value = "/users/{id}", method = RequestMethod.GET)
    UserDto load(@PathVariable("id") Long id);
}

@Override
    public UserDto load(Long id) {
        UserDto userDto = this.userRedisRepository.findOne(id);
        if (null != userDto) {
            this.logger.debug("已从Redis缓存中获取到用户:{} 的信息", id);
            return userDto;
        }

        this.logger.debug("Redis缓存中不存在用户:{} 的信息，尝试从远程进行加载", id);
        userDto = this.userRemoteClient.load(id);//Feign 调用方式
        if (null != userDto) {//获取数据同时放入缓存
            this.userRedisRepository.saveUser(userDto);
        }
        return userDto;
    }
-------------------------------------

#有一种配置 典型的叫 末尾节点是 map方式的配置
spring.cloud.stream.bindings.inboundUserMsg.destination=cd826-cloud-usertopic
spring.cloud.stream.bindings.inboundUserMsg.content-type=application/json
spring.cloud.stream.bindings.inboundUserMsg.group=productGroup
spring.cloud.stream.kafka.binder.brokers=localhost
spring.cloud.stream.kafka.binder.defaultBrokerPort=9092
spring.cloud.stream.kafka.binder.zkNodes=localhost


定义通道接口
public interface SpringCloudBookChannels {//可看出 inboundUserMsg 同配置文件内的某节点
    @Input("inboundUserMsg")
    SubscribableChannel userMsgs();

    @Output("inboundUserMsg")
    MessageChannel userMsgSender();
}


@EnableBinding(SpringCloudBookChannels.class)
public class UserMsgListener {
    protected Logger logger = LoggerFactory.getLogger(this.getClass());

    @Autowired
    protected UserRedisRepository userRedisRepository;

    @StreamListener("inboundUserMsg")
    public void onUserMsgSink(UserMsg userMsg) {
        if (UserMsg.UA_UPDATE.equalsIgnoreCase(userMsg.getAction())) {
            this.logger.debug("收到用户更新消息，所要更新用户的ID: {}", userMsg.getUserId());
            this.userRedisRepository.delete(userMsg.getUserId());
        } else if (UserMsg.UA_DELETE.equalsIgnoreCase(userMsg.getAction())) {
            this.logger.debug("收到用户删除消息，所删除用户的ID: {}", userMsg.getUserId());
            this.userRedisRepository.delete(userMsg.getUserId());
        } else {
            this.logger.debug("收到未知用户消息，用户的ID: {}", userMsg.getUserId());
        }
    }
}

注意这个错误 Dispatcher has no subscribers for channel，未定义生产者

@EnableBinding(SpringCloudBookChannels.class)
public class UsrMsgSend {

    @Autowired
    private SpringCloudBookChannels source;

    public void send(String message) {
        source.userMsgs().send(MessageBuilder.withPayload(message)
                .setHeader(MessageHeaders.CONTENT_TYPE, MimeTypeUtils.TEXT_PLAIN_VALUE).build());
    }
}


------------
另一方的配置：
spring.cloud.stream.bindings.output.destination=cd826-cloud-usertopic
spring.cloud.stream.bindings.output.content-type=application/json
spring.cloud.stream.kafka.binder.brokers=localhost
spring.cloud.stream.kafka.binder.defaultBrokerPort=9092
spring.cloud.stream.kafka.binder.zkNodes=localhost

import org.springframework.cloud.stream.annotation.Output;
import org.springframework.messaging.MessageChannel;

public interface Source {//就是一个接口
    String OUTPUT = "output";

    @Output("output")
    MessageChannel output();
}


@Component
public class UserMsgSender {
    protected Logger logger = LoggerFactory.getLogger(this.getClass());

    private Source source;

    @Autowired
    public UserMsgSender(Source source) {
        this.source = source;
    }

    public void sendMsg(UserMsg userMsg) {
        this.logger.debug("发送用户变更消息:{} ", userMsg);

        // 发送消息
        this.source.output().send(MessageBuilder.withPayload(userMsg).build());
    }
}


windows下 curl同linux 不一样 ，注意转义 \

curl -H "Content-Type:application/json" -X POST -d "{\"id\":3,\"nickname\":\"yxh1\",\"avatar\":\"yhy1\"}"  http://localhost:2100/users/3



type=Method Not Allowed, status=405 这种错误 一般是get/post方法访问的问题


curl -X POST  http://localhost:2100/bus/events/upddd



spring-cloud-starter-bus-kafka 为kafka的消息总线 ，前者包含后者 spring-cloud-starter-stream-kafka
同时也包含了 spring-cloud-bus

-------------------
发送自定义的消息总线：

public class MyBusEvent extends RemoteApplicationEvent {
    private static final long serialVersionUID = 1L;
。。。。


/**
 * 发布一个事件
 * @param eventType
 * @return
 */
@RequestMapping(value = "/{eventType}", method = RequestMethod.POST)
public Boolean publishEvent(@PathVariable String eventType){
    ApplicationContext applicationContext = ApplicationContextHolder.getApplicationContext();
    MyBusEvent myBusEvent = new MyBusEvent(this, applicationContext.getId(), "*:**", eventType);
    applicationContext.publishEvent(myBusEvent);
    this.logger.debug("publish remote application event = {}", myBusEvent);
    return Boolean.TRUE;
}



接收自定义的消息总线：

@Component
public class MyBusEventListener implements ApplicationListener<MyBusEvent> {
    protected Logger logger = LoggerFactory.getLogger(this.getClass());

    @Override
    public void onApplicationEvent(MyBusEvent event) {
        this.logger.debug("Recived an remote event... ");
        this.logger.debug("Event type = {}", event.getEventType());
    }
}


-----------------------
ssh-keygen -t rsa -C "yxhhc@hotmail.com"

cat /c/Users/Administrator/.ssh/id_rsa.pub


指定刷新：
#刷新所有的哈  一秒后 客户端的配置信息就跟着变化了 ok 明白了
curl -X POST http://localhost:8888/bus/refresh
#指定刷新某个服务的配置信息
curl -X POST http://localhost:8888/bus/refresh?destination=productservice:2200


http://localhost:8888/productservice/dev   此种方式可查看 配置信息已变化了，但内存内的还未刷新

还得  curl -X POST http://localhost:8888/bus/refresh 类似这样刷新

git remote -v

git remote set-url origin https://github.com/yuxhe/SpringcloudSamplesConfig.git

注意：有个坑 如果前后github url 不同则不会触发相关刷新哈

spring-cloud-config-server 内部包含了 actuator 哈
-----------------------------------------------------

ssh-keygen -t rsa -C "yxhhc@hotmail.com"   然后一路回车就看可以了

cat ...显示  复制  - setting  内 ssh 粘贴过去就可以了，主要是 连接需对应 ssh 哈


spring cloud config 后端也可以是数据库方式哈

-----------------------------------------------

#WebSecurityConfigurerAdapter 和 ResourceServerConfigurerAdapter 都是Spring 使用的过滤器，用来对访问进行过滤的。都是对http访问资源的控制   ConfigurerAdapter

SecurityConfigurer


@EnableWebSecurity
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

#Spring Boot的自动配置自动启用Web安全性并检索WebSecurityConfigurerAdapter类型的所有bean，以在满足特定条件（类路径上的 spring-boot-starter-security 等）时自定义配置 . 在类 org.springframework.boot.autoconfigure.security.SpringBootWebSecurityConfiguration 中启用了Web安全性的自动配置, 因此spring  boot 中用户可不用配置 @EnableWebSecurity 这个了


重写匹配权限策略

@Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable();
        http.authorizeRequests()
                .antMatchers(HttpMethod.DELETE, "/products/**")
                .hasRole("ADMIN")
                .anyRequest()
                .authenticated()
        .and()
        .httpBasic();

        http.authorizeRequests().antMatchers(SHOW_RECORD_P).hasRole("PROVIDER");
		http.authorizeRequests().antMatchers(UPDATE_RECORD).hasRole("CLIENT");
		http.authorizeRequests().antMatchers(SHOW_ALL_CLIENTS).hasRole("ADMIN");
    }

可自定义登录页面：
http.formLogin().loginPage("/login.do").loginProcessingUrl(loginProcessingUrl).and().authorizeRequests()

还有一种不是301、302重定向 而是直接输出登录页面给前端
spring  security

重要特点覆写 UserDetailsService：

@Configuration
public class AccountingCheck implements UserDetailsService{
	
	@Autowired
	IAccounts accounts;
	@Override
	public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {//此处核查身份 及获取用户角色哈
		String password=accounts.getPassword(username);
		if(password==null)
			throw new UsernameNotFoundException("");
		return new User(username,password ,
				AuthorityUtils.createAuthorityList(accounts.getRoles(username)));//获取角色
	}

}

![](https://img-blog.csdn.net/20180406152002371?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L215X2xlYXJuaW5nX3JvYWQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

security 基于username、password、roles 三者实现的

@Configuration 与 @EnableWebSecurity 起到了同等作用某种情况下


inMemoryAuthentication为其中之一的内存模式，模拟内存用户，其他还有jdbc模式用户等

protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("zhangsan")
                .password("password")
                .roles("USER")
                .and()
                .withUser("superAdmin")
                .password("superPwd")
                .roles("ADMIN");
    }

--------------------------------------------------------
spring-cloud-starter-zuul 内置如下功能：

spring-cloud-starter
spring-boot-starter-web
spring-boot-starter-actuator
spring-cloud-starter-hystrix
spring-cloud-starter-ribbon
spring-cloud-starter-archaius
zuul-core

spring-cloud-security 整合集成依赖 spring-security-oauth2



@Configuration
public class ResourceServerConfiguration extends ResourceServerConfigurerAdapter {
    @Override
    public void configure(HttpSecurity http) throws Exception{
        http.authorizeRequests().anyRequest().authenticated();
    }
}


------------
@Autowired
    private OAuth2RestTemplate restTemplate;


OAuth2RestTemplate 扩展了RestTemplate 这个东东

@EnableAuthorizationServer   开启认证获取令牌

http://localhost:8900/auth/oauth/token


https://blog.csdn.net/qq_41853447/article/details/106598361



springclouddemo   scdsecret


1)base64编码 最基本的 放到header头上,最基本的 Basic auth
springclouddemo:scdsecret  ->  c3ByaW5nY2xvdWRkZW1vOnNjZHNlY3JldA==

curl -X POST -H "Authorization:Basic c3ByaW5nY2xvdWRkZW1vOnNjZHNlY3JldA==" -i -d "grant_type=client_credentials" http://localhost:8900/auth/oauth/token


2)password方式获取token
curl -X POST -H "Authorization:Basic c3ByaW5nY2xvdWRkZW1vOnNjZHNlY3JldA==" -i -d "grant_type=password&username=zhangsan&password=password" http://localhost:8900/auth/oauth/token

下面这句还是过不去呀
http://localhost:8900/auth/oauth/token?username=zhangsan&password=password&grant_type=password&client_id=springclouddemo&client_secret=scdsecret

3)refresh_token方式刷新access_token

curl -X POST -H "Authorization:Basic c3ByaW5nY2xvdWRkZW1vOnNjZHNlY3JldA==" -i -d "grant_type=refresh_token&refresh_token=37704b3b-a5e3-41fc-9c7e-b80eba4690ab" http://localhost:8900/auth/oauth/token


4)访问商品库 利用上面的access_token访问 商品信息  dos窗口下 chcp 65001 转换中文乱码问题
curl  -H "Authorization: Bearer b731d086-64fd-4d7b-b868-e01f4b837b5a" -i  http://localhost:2200/products/3/comments 





http://localhost:8900/auth/user 验证token的有效性


配置资源的访问策略
@Configuration
public class ResourceServerConfiguration extends ResourceServerConfigurerAdapter {
    @Override
    public void configure(HttpSecurity http) throws Exception{
        http.authorizeRequests().anyRequest().authenticated();
    }
}


访问另一微服务

@Autowired
private OAuth2RestTemplate restTemplate;

重启动auth服务则 token失效

![](https://img2018.cnblogs.com/blog/253045/201902/253045-20190209165028711-1789615754.png)


@ComponentScan 默认扫描的同 Application 包下面的东东哈

日志追踪 logging.level.org.springframework=debug
OAuth2AuthenticationProcessingFilter 过滤器内会调用check token接扣校验token是否有效;


注意有时为了直接按照服务名称访问，必须得加上 @LoadBalanced 才行哈：

@EnableDiscoveryClient
@EnableResourceServer
@SpringBootApplication
@EnableJpaRepositories({"com.cd826dong.**.repository"})
@ComponentScan("com.cd826dong.**")
public class Application {

    @LoadBalanced
    @Bean
    public OAuth2RestTemplate oauth2RestTemplate(OAuth2ClientContext oauth2ClientContext,
                                                 OAuth2ProtectedResourceDetails details) {
        return new OAuth2RestTemplate(details, oauth2ClientContext);
    }

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}




@Autowired
    private UserFeginService  userFeginService; 前提条件是开启@EnableFeignClients 同时定义接口

1）@EnableFeignClients


2）@FeignClient("USERSERVICE")
public interface UserFeginService {
    @RequestMapping(value = "/users", method = RequestMethod.GET)
    List<UserDto> findAll();

    @RequestMapping(value = "/users/{id}", method = RequestMethod.GET)
    UserDto load(@PathVariable("id") Long id);
}

#Fegin中使用auth2  则需要传递器传递必要的头信息

import feign.RequestInterceptor;
import feign.RequestTemplate;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContext;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.oauth2.provider.authentication.OAuth2AuthenticationDetails;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FeignOauth2RequestInterceptor implements RequestInterceptor {

    private final String AUTHORIZATION_HEADER = "Authorization";
    private final String BEARER_TOKEN_TYPE = "Bearer";

    @Override
    public void apply(RequestTemplate requestTemplate) {
        SecurityContext securityContext = SecurityContextHolder.getContext();
        Authentication authentication = securityContext.getAuthentication();
        if (authentication != null && authentication.getDetails() instanceof OAuth2AuthenticationDetails) {
            OAuth2AuthenticationDetails details = (OAuth2AuthenticationDetails) authentication.getDetails();
            requestTemplate.header(AUTHORIZATION_HEADER, String.format("%s %s", BEARER_TOKEN_TYPE, details.getTokenValue()));
        }

    }
}

#网关zuul中要玩auth2  则也需要传递必要的头信息，需要配置一下
zuul.sensitiveHeaders=Cookie,Set-Cookie


auth2 + redis 实现分布式认证，或单点登录sso ,原理是相通的


/**
 * OAuth2 配置
 *
 * @author CD826(CD826Dong@gmail.com)
 * @since 1.0.0
 */
@Configuration
public class OAuthConfig extends AuthorizationServerConfigurerAdapter {
    @Autowired
    private AuthenticationManager authenticationManager;
    @Autowired
    private UserDetailsService userDetailsService;

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
                .withClient("springclouddemo")
                .secret("scdsecret")
                .authorizedGrantTypes("refresh_token", "password", "client_credentials")
                .scopes("webclient", "mobileclient");
    }

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints
                .authenticationManager(authenticationManager)
                .userDetailsService(userDetailsService);
    }
}


/**
 * OAuth2 Web安全配置
 *
 * @author CD826(CD826Dong@gmail.com)
 * @since 1.0.0
 */
@Configuration
@Order(org.springframework.boot.autoconfigure.security.SecurityProperties.ACCESS_OVERRIDE_ORDER)
public class OAuthWebSecurityConfigurer extends WebSecurityConfigurerAdapter {
    @Override
    @Bean
    public AuthenticationManager authenticationManagerBean() throws Exception{
        return super.authenticationManagerBean();
    }

    @Override
    @Bean
    public UserDetailsService userDetailsServiceBean() throws Exception {//这个可改写 来自数据库的处理
        return super.userDetailsServiceBean();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
                .withUser("zhangsan")
                .password("password")
                .roles("USER")
                .and()
                .withUser("superAdmin")
                .password("superPwd")
                .roles("USER", "ADMIN");
    }
}

//-----------------------------------------------------------------------------------

项目路径：server.contextPath

BCryptPasswordEncoder 加密  --- 注意有加盐部分哈

 通过tokenStore来定义Token的存储方式和生成方式：
     InMemoryTokenStore
     JdbcTokenStore
     JwtTokenStore
     RedisTokenStore


// 授权码模式,这个配置的是客户端申请的app_id和app_secrete

// 这个格式要在 Spring Security 5.0开始
// String finalSecret = "{bcrypt}" + new BCryptPasswordEncoder().encode("123456");


 /**
     * 使用Md5加密方式，也可以采用BCryptPasswordEncoder
     */
    @Bean
    public Md5PasswordEncoder passwordEncoder(){
        return new Md5PasswordEncoder();//PasswordEncoder
    }


/**
     * 使用Redis作为token的缓存
     */
    @Bean
    public TokenStore tokenStore(RedisConnectionFactory connectionFactory){
        return new RedisTokenStore(connectionFactory);
    }

@Configuration
public class TokenConfig {

    private String SIGNING_KEY = "uaa123";

    @Bean
    public TokenStore tokenStore() {
        //JWT令牌存储方案
        return new JwtTokenStore(accessTokenConverter());
    }

    @Bean
    public JwtAccessTokenConverter accessTokenConverter() {
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        converter.setSigningKey(SIGNING_KEY); //对称秘钥，资源服务器使用该秘钥来验证
        return converter;
    }

    /*@Bean
    public TokenStore tokenStore() {
        //使用内存存储令牌（普通令牌）
        return new InMemoryTokenStore();
    }*/
}


#-------------jwt 坐标 ，看坐标jwt是建立在auth2基础之上哈
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.security.oauth</groupId>
            <artifactId>spring-security-oauth2</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-jwt</artifactId>
        </dependency>


#jwt.signing.key = springcloud-dong 


你要实现TokenEhancer（令牌增强器）中的 ehance方法

public class JWTTokenEnhancer implements TokenEnhancer {//令牌增强器
    @Override
    public OAuth2AccessToken enhance(OAuth2AccessToken accessToken, OAuth2Authentication authentication) {
        Map<String, Object> additionalInfo = new HashMap<>();
        additionalInfo.put("shopId", "cd826");
        ((DefaultOAuth2AccessToken) accessToken).setAdditionalInformation(additionalInfo);
        return accessToken;
    }
}

jwt  是一个开放标准（RFC 7519）

JwtAccessTokenConverter  

#JwtAccessTokenConverter：TokenEnhancer的子类，帮助程序在JWT编码的令牌值和OAuth身份验证信息之间进行转换（在两个方向上），同时充当TokenEnhancer授予令牌的时间。
自定义的JwtAccessTokenConverter（把自己设置的jwt签名加入accessTokenConverter中）


@Configuration
public class JWTTokenConfig {
    @Autowired
    private ServiceConfig serviceConfig;

    @Bean
    public TokenStore tokenStore() {//这个是处理令牌的json部分哈
        return new JwtTokenStore(jwtAccessTokenConverter());
    }

    @Bean
    @Primary
    public DefaultTokenServices tokenServices() {//采用 jwt思路处理
        DefaultTokenServices defaultTokenServices = new DefaultTokenServices();
        defaultTokenServices.setTokenStore(tokenStore());
        defaultTokenServices.setSupportRefreshToken(true);
        return defaultTokenServices;
    }

    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {//这个主要是加盐
        JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
        converter.setSigningKey(serviceConfig.getJwtSigningKey());
        return converter;
    }

    @Bean
    public TokenEnhancer jwtTokenEnhancer() {//令牌增强加入附加的一些信息
        return new JWTTokenEnhancer();
    }
}


@Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        TokenEnhancerChain tokenEnhancerChain = new TokenEnhancerChain();
        tokenEnhancerChain.setTokenEnhancers(Arrays.asList(this.jwtTokenEnhancer, this.jwtAccessTokenConverter));

        endpoints.tokenStore(this.tokenStore)  //加入存储 jwt存储
                .accessTokenConverter(this.jwtAccessTokenConverter) //加入转换器
                .tokenEnhancer(tokenEnhancerChain) //加入增强器
                .authenticationManager(authenticationManager)
                .userDetailsService(userDetailsService);
    }


注意首字母的大小写问题  bearer 

@EnableResourceServer 开启了这个一定会向资源服务器发送一次信息 哈，主要是令牌核查



#拦截器转发的请求加上 必要的token信息：
public class JWTOAuthTokenInterceptor implements ClientHttpRequestInterceptor {
    @Override
    public ClientHttpResponse intercept(
            HttpRequest request, byte[] body, ClientHttpRequestExecution execution)
            throws IOException {

        HttpHeaders headers = request.getHeaders();
        headers.add(UserContext.AUTH_TOKEN, UserContext.getAuthToken());

        return execution.execute(request, body);
    }
}


ClientHttpRequestInterceptor 这个也是ribbon的拦截器哈

同理Fegin也采用 相似手段处理加上头 实现扩展Fegin的拦截器 RequestInterceptor


    //上下文  三个线程变量
    private static final ThreadLocal<String> authToken= new ThreadLocal<String>();
    private static final ThreadLocal<String> userId = new ThreadLocal<String>();
    private static final ThreadLocal<String> shopId = new ThreadLocal<String>();


#总之服务之间有堵墙，这个就是首先要拿去验证真伪，同时调转访问下一个的时候也得带上头去访问，及其他传递头信息
#肯定是拦截器的作用哈

#访问链中加入 拦截器 
@Bean
    @Primary
    public RestTemplate getCustomRestTemplate() {
        RestTemplate template = new RestTemplate();
        List interceptors = template.getInterceptors();
        if (interceptors == null) {
            template.setInterceptors(Collections.singletonList(new JWTOAuthTokenInterceptor()));
        } else {
            interceptors.add(new JWTOAuthTokenInterceptor());
            template.setInterceptors(interceptors);
        }
        return template;
    }

------------------

#jjwt可解密客户端，传入解盐
Claims claims = Jwts.parser()
                    .setSigningKey(serviceConfig.getJwtSigningKey().getBytes("UTF-8"))
                    .parseClaimsJws(authToken).getBody();
            filterUtils.setShopId((String) claims.get("shopId"));
            filterUtils.setUserId((String) claims.get("user_name"));
            System.out.println("shopId:"+(String) claims.get("shopId"));
            System.out.println("user_name:"+(String) claims.get("user_name"));



------------
#数据库方式  主要就是获取令牌，性能应该可以控制的，注意可以配置过期时间等

@Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {

        //默认值InMemoryTokenStore对于单个服务器是完全正常的（即，在发生故障的情况下，低流量和热备份备份服务器）。大多数项目可以从这里开始，也可以在开发模式下运行，以便轻松启动没有依赖关系的服务器。
        //这JdbcTokenStore是同一件事的JDBC版本，它将令牌数据存储在关系数据库中。如果您可以在服务器之间共享数据库，则可以使用JDBC版本，如果只有一个，则扩展同一服务器的实例，或者如果有多个组件，则授权和资源服务器。要使用JdbcTokenStore你需要“spring-jdbc”的类路径。

        //这个地方指的是从jdbc查出数据来存储
        clients.withClientDetails(clientDetails());


    }


import javax.sql.DataSource;
@Resource
private DataSource dataSource;


 @Bean
    public ClientDetailsService clientDetails() {
        return new JdbcClientDetailsService(dataSource);
    }


源码：

public JdbcClientDetailsService(DataSource dataSource) {
        this.updateClientDetailsSql = DEFAULT_UPDATE_STATEMENT;
        this.updateClientSecretSql = "update oauth_client_details set client_secret = ? where client_id = ?";
        this.insertClientDetailsSql = "insert into oauth_client_details (client_secret, resource_ids, scope, authorized_grant_types, web_server_redirect_uri, authorities, access_token_validity, refresh_token_validity, additional_information, autoapprove, client_id) values (?,?,?,?,?,?,?,?,?,?,?)";
        this.selectClientDetailsSql = "select client_id, client_secret, resource_ids, scope, authorized_grant_types, web_server_redirect_uri, authorities, access_token_validity, refresh_token_validity, additional_information, autoapprove from oauth_client_details where client_id = ?";
        this.passwordEncoder = NoOpPasswordEncoder.getInstance();
        Assert.notNull(dataSource, "DataSource required");
        this.jdbcTemplate = new JdbcTemplate(dataSource);
        this.listFactory = new DefaultJdbcListFactory(new NamedParameterJdbcTemplate(this.jdbcTemplate));
    }

-------------------------------------------

OAuth2RestTemplate 这个不加入过滤器 也可正常向前传递 Authorization 头信息

#注意  jwt是无状态的哈，服务重启不影响 之前获取到的token

