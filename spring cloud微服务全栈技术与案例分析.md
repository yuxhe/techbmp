
Spring Cloud 就是用于构建微服务开发和治理的框架集合（井不是具体的一个框架），
主要贡献来自 Netflix OSS

 Feign: Web 服务客户端，能够简 Http 接口的调用

Spring Boot Starter 是用来简化 jar 包依赖的，集成一个框架我 们只需要引人 一个
Starter ，然后在属性文件中配置一些值，整个集成的过程就结束了

添加 spring-boot-devtools 的依赖即可 现热部署功能

Spring Boot 是构建 pring Cloud 的基础

Zoo keeper Leader ，而且在 Leader 无法使用的时候通过 Paxos ( ZAB 算法
举出一个新的 Leader 这个 Leader 的任务就是保证写数据的时候只向这个 Leader 写人
Leader 会同步信息到其他节点 通过这个操作就可以保证数据的一致性


eureka.instance.preferipAddress=true 配置这个才可跳转

eureka.instance.status-page-url=http://cxytiandi.com  自定义跳转页

主次服务器端关闭自我保护：

开发环境下使用，生产 不推荐使用
首先在我们的 fan ia- ur ka 中增加两 置，分 是关闭自我保护和清理
eureka.server.enable-self -preservation=false 
默认 6000 毫秒
eureka.server.evi ction-interval-tirner- in-ms=SOOO

客户端 维护心跳：
eureka.client.healthcheck.enabled=true 
默认 30
eureka.instance.lease-renewal-interval-in-seconds=5  表示eureka client发送心跳给server端的频率
默认 90
eureka.instance.lease-expiration-duration-in-seconds=5 表示eureka server至上一次收到client的心跳之后，等待下一次心跳的超时时间，在这个时间内若没收到下一次心跳，则将移除该instance 


restTemplate 不能兼顾两种方式：二者必须分开 ，无语了

//非负载平衡是前者
return restTemplate.getForObject(
			"http://localhost:8083/user/hello", String.class);

负载平衡是后者
return restTemplate.getForObject(
			"http://eureka-client-user-service/user/hello", String.class); 	

@Configuration
public class BeanConfiguration {

	@Bean("BalanceRestTemplate")
	@LoadBalanced
	public RestTemplate getBalanceRestTemplate() {
		return new RestTemplate();
	}


	@Bean("RestTemplate")
	public RestTemplate getRestTemplate() {
		return new RestTemplate();
	}
}


-----------------------
这个 显示到管理页面还有用哈
eureka.instance.instance-id=${spring.application.name}:${spring.cloud.client.ip-address}:${server.port}


eureka.instance.status-page-url=http://cxytiandi.com 实质是info页

eureka.instance.preferIpAddress=true  //会有ip信息注册到eureka服务器 ，info链接跳转的ip 

#注意 2.x后要 开启 两个 一个是 enabled,一个是暴漏哈
management.endpoint.env.enabled=true
management.endpoints.web.exposure.include=*  注意不能有引号


management:
  endpoint.shutdown.enabled: true # 开启shutdown
  endpoints.web.exposure.include: "*" # 暴露所有端点  此种方式配置有引号
  endpoint.health.show-details: always # health健康检查显示详细信息


#第四章
Ribbon Netflix 开源的一款用于客户端负载均衡的工具软件
GitHub 地址： https://github.com/Netflix/ribbon


打包时引入 

<plugin>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-maven-plugin</artifactId>
</plugin>


<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
		</dependency>

内部包含：

ribbon、 ribbon-core 、  ribbon-httpclient 、 ribbon-loadbalancer、  rxjava

-----------------------------------
//ribbon加载模式 
ribbon.eager-load.enabled=true
ribbon.eager-load.clients=ribbon-config-demo

#ribbon默认配置规则 @RibbonClients ，比如配置随机访问
import org.springframework.cloud.netflix.ribbon.RibbonClients;

/**
 * 默认配置
 * @author yinjihuan
 *
 */
@RibbonClients(defaultConfiguration = DefaultRibbonConfig.class)
public class RibbonClientDefaultConfigurationTestsConfig {

  
}

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import com.netflix.loadbalancer.IRule;
import com.netflix.loadbalancer.RandomRule;

@Configuration
public class DefaultRibbonConfig {
	
	@Bean
	public IRule ribbonRule() {
		return new RandomRule();
	}

}



#ribbon 配置某个服务具体的访问规则
import org.springframework.cloud.netflix.ribbon.RibbonClient;

/**
 * 为某个客户端指定配置
 * @author yinjihuan
 *
 */
@RibbonClient(name = "ribbon-config-demo", configuration = BeanConfiguration.class) 
public class RibbonClientConfig {  

}



import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

import com.cxytiandi.ribbon_eureka_demo.rule.MyRule;

@Configuration
public class BeanConfiguration {

	@Bean
	@LoadBalanced
	public RestTemplate getRestTemplate() {
		return new RestTemplate();
	}

	//@Bean     可开启测试
	public MyRule rule() { 	    
	    return new MyRule();     
	} 

}


import java.util.List;

import com.netflix.loadbalancer.ILoadBalancer;
import com.netflix.loadbalancer.IRule;
import com.netflix.loadbalancer.Server;

public class MyRule implements IRule {

	private ILoadBalancer lb;

	@Override
	public Server choose(Object key) {
		List<Server> servers = lb.getAllServers();
		for (Server server : servers) {
			System.out.println(server.getHostPort());
		}
		return servers.get(0);
	}

	@Override
	public void setLoadBalancer(ILoadBalancer lb) {
		this.lb = lb;
	}

	@Override
	public ILoadBalancer getLoadBalancer() {
		return lb;
	}

}
----------------------------------

@Bean
	@LoadBalanced
	public RestTemplate getRestTemplate() {
		return new RestTemplate();
	}

#ribbon访问策略  假如有服务或网络不可用时的访问策略

对当前实例的重试次数 <br>
ribbon.maxAutoRetries=l <br>
切换实例的重试次数 <br>
ribbon.maxAutoRetriesNextServer=3 <br>
对所有操作请求都进行重试  <br>
ribbon.okToRetryOnAllOperations=true  <br>

#pom坐标管理 -> 管理依赖、管理版本、管理冲突

#Fegin原始调用
HelloRemote helloRemote = RestApiCallUtils.getRestClient(HelloRemote.class,"http://localhost:8081");


import feign.Feign;

public class RestApiCallUtils {
	/**
	 * 获取 API 接口代理对象
	 * 
	 * @param apiType 接 口 类
	 * @param url     API 地址
	 * @return
	 */
	public static <T> T getRestClient(Class<T> apiType, String url) {
		return Feign.builder().target(apiType, url);
	}

}


#接口代理类
import feign.RequestLine;

public interface HelloRemote {
	
	@RequestLine("GET /user/hello") 
	String hello();
	
}


------
@FeignClient(value = "feign-server",configuration = FeignConfig.class)  //需要一个配置文件
public interface TestService {
    @RequestLine("POST /feign/test")    //对应请求方式和路径
    String feign(@RequestBody UserDO userDO);
}


@EnableFeignClients
@SpringBootConfiguration
public class FeignConfig {
    @Bean
    public Contract contract(){
        return new feign.Contract.Default();//  feign 配置
    } 
}

-------------

@RestController
public class DemoController {

	@Autowired
	private UserRemoteClient userRemoteClient; //调用引入方式

	@GetMapping("/callHello")
	public String callHello() {
		String result = userRemoteClient.hello(); //直接调用了
		System.out.println(" 调用结果：" + result);
		return result;
	}

}



import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;

import com.cxytiandi.feign_demo.config.FeignConfiguration;
#可以引入配置哈 ，比如拦截器等
@FeignClient(value="eureka-client-user-service", configuration=FeignConfiguration.class)
public interface UserRemoteClient {

	@GetMapping("/user/hello") 
	String hello();
	
}



import feign.RequestInterceptor;
import feign.RequestTemplate;

public class FeignBasicAuthRequestInterceptor implements RequestInterceptor {
	
	public FeignBasicAuthRequestInterceptor() {

	}

	public void apply(RequestTemplate template) {
		System.err.println("进入拦截器了");
	}

	
}



import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.cxytiandi.feign_demo.auth.FeignBasicAuthRequestInterceptor;

import feign.Logger;
import feign.Request;
import feign.auth.BasicAuthRequestInterceptor;

@Configuration
public class FeignConfiguration {
	/**
	 * 日志级别
	 * 
	 * @return
	 */
	@Bean
	Logger.Level feignLoggerLevel() {//日志级别
		return Logger.Level.FULL;
	}

	@Bean
	public BasicAuthRequestInterceptor basicAuthRequestInterceptor() {//http基本认证
		return new BasicAuthRequestInterceptor("user", "password");
	}

	@Bean
	public FeignBasicAuthRequestInterceptor feignBasicAuthRequestInterceptor() {//拦截器
		return new FeignBasicAuthRequestInterceptor();
	}

	@Bean
	public Request.Options options() {//参数
		return new Request.Options(5000, 10000);
	}

}

#fegin四种日志级别
NONE，无记录（DEFAULT）。

BASIC，只记录请求方法和URL以及响应状态代码和执行时间。

HEADERS，记录基本信息以及请求和响应标头。

FULL，记录请求和响应的头文件，正文和元数据。

作者：咔啡
链接：https://www.jianshu.com/p/5d6d576eeb25
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。


logging.level.com.cxytiandi.feign_demo.remote.UserRemoteClient=DEBUG

feign.httpclient.enabled=false 
feign.okhttp.enabled=true

feign.compression.request.enabled=true
feign.compression.response.enabled=true
feign.compression.request.mime-types=text/xml,application/xml,application/json
feign.compression.request.min-request-size=2048

-------------------------------------------------



@EnableDiscoveryClient
@EnableFeignClients(basePackages= {"com.cxytiandi.feignapi"})  注意非本包下则需要指出扫描范围
@SpringBootApplication
public class App {
	public static void main(String[] args) {
		SpringApplication.run(App.class, args);
	}
}


生产提供方 实现接口的思路

import java.util.Map;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import com.cxytiandi.feignapi.user.User;
import com.cxytiandi.feignapi.user.UserRemoteClient;

@RestController
public class DemoController implements UserRemoteClient {

	@Override
	public String getName() {
		return "yinjihuan";
	}


服务消费方

@RestController
public class DemoController {

	@Autowired
	private UserRemoteClient userRemoteClient;



共享接口的思路

import java.util.Map;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestParam;

@FeignClient("feign-inherit-provide")
public interface UserRemoteClient {
	
	@GetMapping("/user/name")
	String getName();
	
	@GetMapping("/user/info")
	String getUserInfo(@RequestParam("name")String name);
	
	@GetMapping("/user/detail")
	String getUserDetail(@RequestParam Map<String, Object> param);
	
	@PostMapping("/user/add")
	String addUser(@RequestBody User user);
	
}


服务提供方、服务消费方包含 接口的思路，接口共享：

		<dependency>
			<groupId>com.cxytiandi</groupId>
			<artifactId>feign-inherit-api</artifactId>
			<version>0.0.1-SNAPSHOT</version>
		</dependency>


---------------------------

#Hystrix 通过 HystrixCommand 对调用进行隔

Hystrix 支持将多个请求自动合并为一个请求

在启动类上添加＠Enab eHystr 或者＠EnableCirc tBreaker 注意，＠EnableHystrix
中包含了＠EnableCircuitBreaker

＠HystrixCommand 注解


hystrix.command default.execution.isolation.strategy
具体策略有下面两种
• THREAD ：线程隔离，在单独的线程上执行，并发请求受线程池大小的控制
• SEMAPHORE ：信号量隔离，在调用线程上执行，并发请求受信号量计数器限制


hystrix. command.default. execution. isolation. thread. timeoutlnMilliseconds 该配置用
hystrix Command 执行的超时时间设置，当 HystrixCommand 执行的时间超过了该所设的数值后就会进入服务降级处理，单位是毫秒，默认值为 1000

https://github.com/Netflix/Hystrix/wiki/Configuration

feign.hystrix.enabled=true 开启fegin对hystrix的支持

在fegin中有两种方式来实现哈
1）Fallback 方式  2）FallbackFactory 方式

在启动类上添@EnableHystrixDashboard注解，单独的一个服务，能在web控制面版上看到具体的调用图形信息


Hystrix 只能监控单个节点，Turbine 来监控集群，Turbine 是聚合服务器发送事件流数据的一个工具；

Hystrix 熔断降级、监控


请求合并 HystrixCollapser

public class MyHystrixCollapser extends HystrixCollapser<List<String>, String, String> {


import java.util.ArrayList;
import java.util.Collection;
import java.util.List;

import com.netflix.hystrix.HystrixCollapser;
import com.netflix.hystrix.HystrixCommand;
import com.netflix.hystrix.HystrixCommandGroupKey;
import com.netflix.hystrix.HystrixCommandKey;

public class MyHystrixCollapser extends HystrixCollapser<List<String>, String, String> {
	private final String name;

	public MyHystrixCollapser(String name) {
		this.name = name;
	}

	@Override
	public String getRequestArgument() {
		return name;
	}

	@Override
	protected HystrixCommand<List<String>> createCommand(final Collection<CollapsedRequest<String, String>> requests) {
		return new BatchCommand(requests);
	}

	@Override
	protected void mapResponseToRequests(List<String> batchResponse,
			Collection<CollapsedRequest<String, String>> requests) {
		int count = 0;
		for (CollapsedRequest<String, String> request : requests) {
			request.setResponse(batchResponse.get(count++));
		}
	}

	private static final class BatchCommand extends HystrixCommand<List<String>> {

		private final Collection<CollapsedRequest<String, String>> requests;

		private BatchCommand(Collection<CollapsedRequest<String, String>> requests) {
			super(Setter.withGroupKey(HystrixCommandGroupKey.Factory.asKey("ExampleGroup"))
					.andCommandKey(HystrixCommandKey.Factory.asKey("GetValueForKey")));
			this.requests = requests;
		}

		@Override
		protected List<String> run() {//这才是执行业务逻辑的地方哈
			System.out.println("真正执行请求.......");
			ArrayList<String> response = new ArrayList<String>();
			for (CollapsedRequest<String, String> request : requests) {//加入到结果数组中哈
				response.add("返回结果: " + request.getArgument());
			}
			return response;
		}
	}
}

-------------------------

import java.util.concurrent.ExecutionException;
import java.util.concurrent.Future;

import com.netflix.hystrix.strategy.concurrency.HystrixRequestContext;

public class CollapserApp {
	public static void main(String[] args) throws InterruptedException, ExecutionException {
		HystrixRequestContext context = HystrixRequestContext.initializeContext();//初始上下文
		Future<String> f1 = new MyHystrixCollapser("yinjihuan").queue();       
		Future<String> f2 = new MyHystrixCollapser("yinjihuan333").queue();      
		System.out.println(f1.get()+"="+f2.get()); 	
		context.shutdown();

	}
}


-----------------------
spring-configuration-metadata.json 中有关于fegin的参数设置  <br>
# feign - hystrix
feign默认对hystrix是关闭的，要支持则需要开启哈 <br>
feign.hystrix.enabled=true <br>

management.endpoints.web.exposure.include=* <br>

---------------监控数据采集 ，同时有个监控页面展示哈，有这样的服务 其他客户端应@EnableHystrix用很自然的会发送采集的信息给 turbine服务

<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-turbine</artifactId>
		</dependency>


@EnableDiscoveryClient
@EnableTurbine
@EnableHystrixDashboard
@SpringBootApplication
public class DashboardApplication {
	public static void main(String[] args) {
		SpringApplication.run(DashboardApplication.class, args);
	}
}


turbine.appConfig=hystrix-feign-demo
turbine.aggregator.clusterConfig=default
turbine.clusterNameExpression=new String("default")
---------------------

fegin中使用hystrix

#实现接口时回退 FallbackFactory，接口参数内传入接口名


import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;

@FeignClient(value="eureka-client-user-service", fallbackFactory=UserRemoteClientFallbackFactory.class)
public interface UserRemoteClient {

	@GetMapping("/user/hello") 
	String hello();
	
}

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

import feign.hystrix.FallbackFactory;

@Component
public class UserRemoteClientFallbackFactory implements FallbackFactory<UserRemoteClient> {

	private Logger logger = LoggerFactory.getLogger(UserRemoteClientFallbackFactory.class);
	
	@Override
	public UserRemoteClient create(Throwable cause) {//书写上更灵活，可动态处理配置问题、返回上可下功夫哈
		logger.error("UserRemoteClient回退：", cause);
		return new UserRemoteClient() {
			
			@Override
			public String hello() {
				return "fail";
			}
		};
	}

}


#fallback 时回退 ，同样也是直接实现接口；同样注解配置上有细微差别，就是 fallback=UserRemoteClientFallback.class

import org.springframework.stereotype.Component;

@Component
public class UserRemoteClientFallback implements UserRemoteClient {

	@Override
	public String hello() {
		return "fail";
	}
}


import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;

@FeignClient(value="eureka-client-user-service", fallback=UserRemoteClientFallback.class)
public interface UserRemoteClient {

	@GetMapping("/user/hello") 
	String hello();
	
}

#fegin下配置hystrix策略参数 两种方式：

hystrix:
    command:
        "RemoteProductService#getProduct(int)":
            execution:
                isolation:
                    thread:
                        timeoutInMilliseconds: 50
或
hystrix:
    command:
        default:
            execution:
                isolation:
                    thread:
                        timeoutInMilliseconds: 50
或代码方式

@Bean
    public Feign.Builder feignHystrixBuilder() {
        return HystrixFeign.builder().setterFactory(new SetterFactory() {
            @Override
            public HystrixCommand.Setter create(Target<?> target, Method method) {
                return HystrixCommand.Setter
                        .withGroupKey(HystrixCommandGroupKey.Factory.asKey(RemoteProductService.class.getSimpleName()))// 控制 RemoteProductService 下,所有方法的Hystrix Configuration
                        .andCommandPropertiesDefaults(
                                HystrixCommandProperties.Setter().withExecutionTimeoutInMilliseconds(10000) // 超时配置
                        );
            }
        });
    }

---------------------------

#非fegin中使用 hystrix,采用注解 @HystrixCommand ， 可见未采用接口方式，而是挂载其他普通方法

@RestController
public class HellController {

	@Autowired
	private RestTemplate restTemplate;

	@GetMapping("/callHello")
	@HystrixCommand(fallbackMethod = "defaultCallHello", commandProperties = { //可配置策略                 
			@HystrixProperty(
					name="execution.isolation.strategy",      
					value = "THREAD")    
			} 
	)
	public String callHello() {
		String result = restTemplate.getForObject("http://localhost:8088/house/hello", String.class);
		return result;
	}

	public String defaultCallHello() {
		return "fail";
	}

}

----------------------------------------
zuul

#Zuul 的核心是过滤器:动态路由\请求监控\认证鉴权\压力测试\灰度发布

＠EnableZuulProxy 已经自带了＠EnableDiscoveryCl ent


#Zuul 集成 ureka 后，其实就可以为 ureka 中所 的服务进行路由操作了， 默认的转发规则就是"API网关地址＋访问的服务名称＋接口URI"

#1)指定具体服务路由 每一个服务都配置一个路由转发规则 zuul.routes.fsh-house.path;/api-house/**    后面一定要配置两个星号，两个星号表示可以转发任意层级的URL

#2)路由前缀  zuul.prefix=/rest   http://cxytiandi.com/rest/user/login

#3)本地跳转

zuul.routes.fsh-substitution.path=/api/** 
zuul.routes.fsh-substitution.url=/forward:/local


#Zuul 中的过滤器总共有 类型，每种类型都有对应的使用场  https://github.com/Netflix/zuul
• pre ：可以在请求被路由之前调用 适用于身份认证的场景，认证通过后再继续执行下面的流程 <br>
• route ：在路由请求时被调用 适用于灰度发布场景，在将要路由的时候可以做一些自定义的逻辑 <br>
• post ：在 route error 过滤器之后被调用 速种过滤器将请求路由到达具体的服务之后执行 适用于需要添加响应头，记录响应日志等应用场景 <br>
• error ：处理请求 发生错误时被调用 在执行过程中发送错误时会进入 error 过滤器，可以用来统一记录错误信息  <br>

public class pFilter extends ZuulFilter { <p>
1)shouldFilter是否执行该过滤器 true 为执行， false 为不执行  <p>
2)filterType ：过滤器类型，可选值有 pre route post error <p>
3)filterOrder ：过滤器的执行顺序，数值越小，优先级越高 ；还有同过滤器类型也有关系哈 <p>
4)run ：执行自己的业务逻辑，本段代码中是通过判断请求 IP 否在黑名单中，决定是否进行拦截的 BasicConf 是一个配置类，里面有个字段是 IP 的黑名单，判断件成立之后，通过设置 ctx .setSendZuu!Response( false），告诉 Zuul 需要将当前请求转发到后端的服务了 通过 setResponseBody 返回数据给客户端 <p>
5)定义完 ZuulFilter 还需要进行配置 <p>

@Configuration  <br>
public class FilterConfig { <br>
@Bean <br>
  public IpFilter ipFilter() { <br>
        return  new IpFilter(); <br>
 } <br>
} <p>


有的场景下，我们需要禁用过滤器，此时可以采取下面的两种方式来实现：<p>
·1)利用 shouldFilter 方法中的 return false 让过滤器不再执行 <br>
·2)过配置方式来禁用过滤器，格式为“ zuul.过滤器的类名.过滤器类型 .disabl”如果我们需要禁用 7.4.3节中的 IpFilter ，可以用下面的配 <br>
zuul. pFilter.pre.disable=true


通过 RequestContext set 方法进行传递， RequestContext的原理就是 ThreadLocal

RequestContext ctx = RequestContext.getCurrentContext(); 
ctx.set("msg","你好吗"）； <p>
后面的过滤就可以通过 RequestContext get 方法来获取数据：
RequestContext ctx = RequestContext.getCurrentContext(); 
ctx.get("msg")


RequestContext ctx = RequestContext.getCurrentContext(); 
ctx.setSendZuulResponse(false);  //不往后传递
ctx.setResponseBody("返回信息"）；
return null;   //同时需要返回 null 这个也重要哈


#spring  boot 异常统一处理  

@RestController 
public class ErrorHandlerController implements ErrorController {


#zuul开启重试机制要依赖spring retry

<dependency> 
<group d>org.springframework.retry</group d>
<artifactid>spring-retry</artifact d>
</dependency>

zuul.retryable=true 
ribbon.connectTimeout=SOO 
ribbon.readTimeout=SOOO 
ribbon.maxAutoRetries=l 
ribbon.maxAutoRetriesNextServer=3 
ribbon.okToRetryOnAllOperations=true


• zuul.retryable ：开启重试 <br>
• ribbon connectTim out 请求连接的超时时间（ ms <br>
• ribbon readTimeout 请求处理的超时时间（ ms   <br>
• ribbon maxAutoRetries ：对 实例的重试次数    <br>
• ribbon.maxAutoRetriesNextServer ：切换实例的最大重试次数   <br>
• ribbon.okToRetryOnAllOperations ：对所有操作请求都进行重试  <br>

#Zuu 默认整合了 Hystrix

@Component 
public class ServiceConsumerFallbackProvider implements ZuulFallbackProvider {

getRoute 方法中返回＊表示对所有服务进行回退操作，如果只想对某个服务进行回退，
那么就返回需要回退的服务名称，这个名称 定要是注册到 Eur 中的名称

通过 getS tatusCode 返回响应的状态码；通过 getStatusText 返回响应状态码对应的文本；通过 getBody 返回回退的内容；通过getHeaders 返回响应的请求头信息


#Zuul 高可用  ,注册到 Eureka 中，通过 Ribbon 来进行负载均衡

Nginx upstream ，然后重载（ reload ）配置来达到网关的动态扩容



这个就是肉的价钱： 假如后续处理为 非压缩的内容时，需要处理这个地方

List<Pair<String, String>>  headerList1 = RequestContext.getCurrentContext().getZuulResponseHeaders(); <br>
        for (Pair<String, String> pair : headerList1) {     <br>
            if  ("gzip".equalsIgnoreCase(pair.second())) {  <br>
                pair.setSecond("");
            }
            System.err.println("RESPONSE HEADER1:: > " +pair.second());
        }

#注意 ZuulResponseHeaders 是 OriginResponseHeaders的克隆版本，后续处理按照 ZuulResponseHeaders走的哈

#ResponseGZipped 这个东东 不过是过滤器间的一个传递标记吧了，标记内容是否加密的一个变量，主要针对的 setResponseDataStream，且是过略器流之间的流转，setResponseBody本身是字符串，不存在压不压缩的问题；同时注意下字符串的优先级高于流的优先级别，注意两个不是同一个东西哈

setResponseBody 设置内容 ，或设置流 setResponseDataStream

#内容gzip的加解密

 public String streamToString2(InputStream in) throws IOException {//内容解密
        
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream(); //定义一个内存输出流
        
        GZIPInputStream gis = new GZIPInputStream(in); //将流转换成字符串
        int len1 = -1;
        byte[] b1 = new byte[1024];
        while ((len1 = gis.read(b1)) != -1) {
            byteArrayOutputStream.write(b1, 0, len1);
        }
        byteArrayOutputStream.close();
        //转化为数组
        // byte[] bytes = byteArrayOutputStream.toByteArray();
        return byteArrayOutputStream.toString();
    }

    public  byte[] compress(String str, String encoding) {//内容压缩
        if (str == null || str.length() == 0) {
            return null;
        }
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        GZIPOutputStream gzip;
        try {
            gzip = new GZIPOutputStream(out);
            gzip.write(str.getBytes(encoding));
            gzip.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return out.toByteArray();
    }



RequestContext.getCurrentContext().get("zuulResponse"); //这个也是获取响应结果


假如只有一个zuulfilter 则header受下面这个的影响

List<Pair<String, String>>  headerList1 = RequestContext.getCurrentContext().getZuulResponseHeaders();
        for (Pair<String, String> pair : headerList1) {
            if  ("gzip".equalsIgnoreCase(pair.second())) {//这个地方是核心点位
                pair.setSecond("");
            }
            System.err.println("RESPONSE HEADER1:: > " +pair.second());
        }


//--------------------------------------还有注意传递内容包体或内容流的一致性哈

public boolean shouldFilter() {
        return RequestContext.getCurrentContext().sendZuulResponse();
}


cxt.setSendZuulResponse(false);//请求到此为止  ******
cxt.setResponseStatusCode(401);//响应码      ********
cxt.set("sendForwardFilter.ran", true);     *****这个呢  只是对forward的映射有啥吧，其他则不是
cxt.addZuulResponseHeader("content-type","text/html;charset=utf-8");//响应头
#cxt.setResponseBody("非法访问");//响应体   本版本这个也是个必要条件 这个也是一个坑，赋值 setResponseDataStream(stream2)也不管用哈

这个run方法的返回值在当前版本(Dalston.SR3)中暂时没有任何意义，可以返回任意值

假如要中途返回 则还是通过值来判断----还是定位到shouldFilter 进行处理
-------------------------------------------------------


spring.servlet.multipart.max-file-size=1000Mb
spring.servlet.multipart.max-request-size=1000Mb


#这是一个文件上传哈 


import java.io.File;
import java.io.IOException;

import org.springframework.util.FileCopyUtils;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

@RestController
public class FileController {

	@PostMapping("/file/upload")
	public String fileUpload(@RequestParam(value = "file") MultipartFile file) throws IOException {
		byte[] bytes = file.getBytes();
		File fileToSave = new File(file.getOriginalFilename());
		FileCopyUtils.copy(bytes, fileToSave);
		return fileToSave.getAbsolutePath();
	}
	
}

#注意zuulfilters使用时注意下，存在各种坑，注意跳出来



#解决乱码问题：
RequestContext.getCurrentContext().getResponse().setContentType("text/html;charset=UTF-8");


#spring-cloud-gateway 功能更强大


＜！一 输出 Json 格式日志 一〉
<dependency> 
<group d>net. logsta sh.logback</group d>
<artifact d>logstash-logback-encoder</artifac d>
<version>4.8</version> 
<scope>runtime</scope> 
</dependency> 
然后创建一个 logback-spring nl 文件



Zipkin Twitter 一个开源项

<dependency> 
<group d>io.zipkin.java</groupld>
<artifactld>zipkin-server</artifactld> 
</dependency> 
<dependency> 
<groupld>io.zipkin .java</groupld> 
<artif actld>zipkin-autocon gure-u </artifactld>
</dependency>


客户端依赖

<dependency> 
<groupid>org.springframework .cloud</ groupid> 
<artifactid>spring-cloud-starter-zipkin</artifactid> 
</dependency>

#zipkin 抽样比例
spring.sleuth.sampler.percentage=l

配置 zipKin Server 的地址
spring.zipkin.base-url=http://localhost:9411
spring.sleuth.sampler.probability=1.0
spring.zipkin.sender.type=RABBIT
spring.rabbitmq.addresses=amqp://192.168.10.124:5672
spring.rabbitmq.username=yinjihuan
spring.rabbitmq.password=123456

#spring mvc中过滤器 GenericFilterBean

import java.io.IOException;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletResponse;

import org.springframework.cloud.sleuth.instrument.web.TraceWebServletAutoConfiguration;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.GenericFilterBean;

import brave.Span;
import brave.Tracer;

@Component
@Order(TraceWebServletAutoConfiguration.TRACING_FILTER_ORDER + 1)
class MyFilter extends GenericFilterBean {

	private final Tracer tracer;

	MyFilter(Tracer tracer) {
		this.tracer = tracer;
	}

	@Override 
	public void doFilter(ServletRequest request, ServletResponse response,
			FilterChain chain) throws IOException, ServletException {
		Span currentSpan = this.tracer.currentSpan();
		if (currentSpan == null) {
			chain.doFilter(request, response);
			return;
		}
		((HttpServletResponse) response).addHeader("ZIPKIN-TRACE-ID",
						currentSpan.context().traceIdString());
		currentSpan.tag("custom", "tag");
		chain.doFilter(request, response);
	}
}

------------
后在启动类上加上 EnableZipkinStreamServer 注解 ，把之前的 nableZipkinServer
去掉 属性文件中 RabbitMq 连接配置
# rabbitrnq 配置
spring.zipkin.base-url=http://localhost:9411
spring.sleuth.sampler.probability=1.0
spring.zipkin.sender.type=RABBIT
spring.rabbitmq.addresses=amqp://192.168.10.124:5672
spring.rabbitmq.username=yinjihuan
spring.rabbitmq.password=123456

#rabbitmq-plugins.bat enable rabbitmq_management  插件是个坑 网页端口地址15672
要安装erl语言，并配置环境变量，默认用户名guest 默认密码guest
创建用户、同时赋予访问的权限

amqp://localhost:5672

接下来我 就要改造需要跟踪的具体服务了，也就是要加入 RabbitMq 的依赖信息，采用
RabbitMq 来替换之前的 Http 发送数据的方式，依赖如下：
<dependency> 
<groupid>org .springfrarnework .cloud</groupid> 
<artifactid>spring -cloud- sleuth-zipkin -stre n</ artifact d>
</dependency> 
<dependency> 
<groupid>org.springfrarnework .cloud</groupid> 
<artifactid>spring -cloud- starter-strearn-rabbit</artifact d>
</dependency>



的zipkin 项目，增加 astic search 存储的依赖
<dependency> 
<groupid>io. z ipkin . java</ groupid> 
<artifact Id> 
zipkin-autoconfigure-storage-elasticsearch- http 
</arti factid> 
<version>l . 24.0</version> 
<optional>true</optional> 
</ dependency> 
属性配置中指定 ipkin 的存储方式以及配置 as ticsearc 的连接信息
zipkin.storage.StorageComponent=e lasticsea rch 
zipkin.storage . type=elasticsearch 
zipkin.storage.elasticsearch.cluster=elasticsearch- zipkin-cluster 
zipkin.storage. e lasticsearch . hosts=12 7. 0.0.1 : 9300 
zipkin.storage . elasticsearch.max- r equests=6 4 
zipkin.storage.elasticsearch . index=zipkin 
zipkin.storage . elasticsearch.index- shards=S 
zipkin.storage.elasticsearch.index- replicas=l


官网下载启动
zipkin-server 

java -jar zipkin-server-2.19.2-exec.jar --RABBIT_ADDRESSES=127.0.0.1:5672


#配置异步线程池：

import java.util.concurrent.Executor;

import org.springframework.beans.factory.BeanFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.cloud.sleuth.instrument.async.LazyTraceExecutor;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.AsyncConfigurerSupport;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

@Configuration
@EnableAutoConfiguration
public class CustomExecutorConfig extends AsyncConfigurerSupport {

	@Autowired BeanFactory beanFactory;

	@Override 
	public Executor getAsyncExecutor() {
		ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
		executor.setCorePoolSize(7);
		executor.setMaxPoolSize(42);
		executor.setQueueCapacity(11);
		executor.setThreadNamePrefix("yinjihuan-");
		executor.initialize();
		return new LazyTraceExecutor(this.beanFactory, executor);
	}
}


拦截标签的一个应用：

import brave.ScopedSpan;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.sleuth.annotation.NewSpan;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;

import brave.Tracer;

@Service
public class ArticleServiceImpl implements ArticleService {

	private Logger logger = LoggerFactory.getLogger(ArticleServiceImpl.class);

	@Autowired
	Tracer tracer;

	@Async
	@Override
	public void saveLog(String log) {
		logger.info("异步线程执行");

	}

	@NewSpan(name = "saveLog2")     //注意这儿哈
	@Override
	public void saveLog2(String log) {
		ScopedSpan span = tracer.startScopedSpan("saveLog2");
		try {
			System.out.println("---="+span.context().traceIdString());
			Thread.sleep(1000);
		} catch (Exception | Error e) {
			//span.error(e);
		} finally {
			//span.finish(); 
		}
	}

------------------------------

监测mysql查询时间

    @Autowired
    private Tracer tracer; //用于访问Spring Cloud Sleuth跟踪信息
public Organization getOrg
            (String organizationId) {
        Span newSpan = tracer.createSpan("getOrgDBCall");//创建一个新的自定义跨度，名为getOrgDBCall
        logger.debug("In the organizationService.getOrg() call");
        try {
            return orgRepository.findById(organizationId);
        }
        finally{
          newSpan.tag("peer.service", "mysql");//将标签信息添加到跨度中，提供了将要被Zipkin捕获的服务名称
          newSpan.logEvent(org.springframework.cloud.sleuth.Span.CLIENT_RECV);//记录事件，告诉Spring Cloud Sleuth捕获调用完成的时间
          tracer.close(newSpan);//关闭跟踪，否则报错
        }
    }

-------------------
添加自定义标签

tracer.currentSpan().tag("用户", "yinjihuan");


可创建新标签方式

@NewSpan(name = "saveLog2")
	@Override
	public void saveLog2(String log) {
		ScopedSpan span = tracer.startScopedSpan("saveLog2");
		try {
			System.out.println("---="+span.context().traceIdString());
			span.tag("key", "firstBiz");
			Thread.sleep(1000);
		} catch (Exception | Error e) {
			span.error(e);
		} finally {
			span.finish();
		}
	}

----------------------------------------------

#spring.zipkin.base-url=http://localhost:9411
spring.sleuth.sampler.probability=1.0
spring.zipkin.sender.type=RABBIT
spring.rabbitmq.addresses=amqp://localhost:5672
spring.rabbitmq.username=yinjihuan
spring.rabbitmq.password=123456


-------------
属性配置中指定zipkin 的存储方式以及配置 as ticsearc 的连接信息
zipkin.storage.StorageComponent=e lasticsea rch 
zipkin.storage.type=elasticsearch 
zipkin.storage.elasticsearch.cluster=elasticsearch-zipkin-cluster 
zipkin.storage.elasticsearch.hosts=127.0.0.1:9300 
zipkin.storage.elasticsearch.max-requests=6 4 
zipkin.storage.elasticsearch.index=zipkin 
zipkin.storage.elasticsearch.index-shards=S 
zipkin.storage.elasticsearch.index-replicas=l


--------------------------------

#JWT 部分构成，第 部分称为头部（Header），第二部分称为消息体（Pay load),部分是签名（Signature 一 JWT 生成的 Token 格式为：
token = encodeBase64 (header) +'·'+ encodeBase64(payload) +'.'+ encodeBase64(signature)

JWT Github 址是 https://github.com/jwtk/jjwt

第12章是 jwt的一个落地实现


management.security.enabled fa lse 要禁用 ctuator 的安全控制，在 Spring Boot 
1.5 以上版本中默认是安全的，不禁用我们访问不了 ctuator 信息

spring.security.user.name=yinjihuan
spring.security.user.password=123456


#Swagger 的目标是对 REST API 定义 个标准且和语言无关的接口

spring-boot tart swagger

@Api 用在类上、@ApiModel用在类上， 表示 进行说明用于类中参数接说明

@ApiModelProperty 用于字段，表示model属性

@ApiParam 方法的参数说明

@ApiOperation 用在Controller里的方法上

@ApiResponse 方法上，

@ApilmplicitParam 手日 @ApilmplicitParams用于方法上，为单独的请求参数进行说明




#集群限流，可通过redis处理，实现zuulFilter

@Override 
public Object run() { 
RequestContext ctx = RequestContext.getCurrentContext(); 
Long currentSecond = System.currentTimeMillis() I 1000; 
String key = "fsh-api-rate-limit-" + currentSecond; 
try { 
if ( ! redisTemplate. has Key (key) ) { 
redisTemplate.opsForValue().set(key, OL, 
10 0 I TimeUni t. SECONDS) ; 
int rate = Integer.parseint( 
System.getProperty (” api.clusterLimitRate”}}; 
／／ 当集群 当前秒的并发量达到 设定的值 不进行处理
／／ 注意集群中的网关与所 服务器时间的必须同步
if (redisTemplate. psForValue().increment(key, 1) >rate) { 
ctx.setSendZuulResponse(false); 
ctx.set(”isSuccess”, false);


#可实现具体某服务的限流
RequestContext ctx = RequestContext.getCurrentContex t(); 
Object service = ctx .get("serviceid"};

@ApiRateLimit(confKey open .api hosue nf）
@GetMapping (” / { houseid}") 
public ResponseData hosueinfo(@PathVariable （” house ”） Long houseid, 
HttpServletRequest request) { 
String uid = request.getHeader(” uid”); 
System.err.println (”==="+uid); 
return ResponseData ok(houseService.getHouse nfo(houseid));

RibbonFilterContextHolder 是基于 Inheritable ThreadLocal 来传输数据的 工具类，为
么要用 InheritableThreadLocal 而不是 ThreadLocal ？在 Spring Cloud 中我们用 Hystrix
实现断路器，默认是用信号量来进行隔离的 信号量的隔离方式用 ThreadLocal 线程
传递数据是没问题的，当隔离模式为线程时， Hystrix 将请求放入 Hystrix 线程池中执
行，这时候某个请求就由 程变成 线程了 ThreadLocal 然没有效果了


#13章 -代码对应15章  灰度发布      本章暂缓 重要
根据IP做灰度发布或用户发布


#14章 -16 暂缓  重要
Guava Cache 一个全内存的本 缓存实现，Guava Cache 不是一个单独的缓存框架，而是 Guava 中的一个模块


LoadingCache<String, Person> cahceBuilder = CacheBuilder.newBuilder() 
.expireAfterWrite(l, TirneUnit.M NUTES)
.build(new CacheLoader<String, Person>() { 
@Override 
public Person load(String key) throws Exception { 
return dao.findBy (id); 
}); 
public Person get(String id) throws Exception { 
return cahceBuilder.get(id);
}


CacheBuilder 为基于时间的回收提供了两种方式，见代码清单 14-2:
• expireAfterAccess(long, TimeUnit) 
当缓存项在指定的时间段内没有被读或写就会被回收 这种回收策略类似于基于容
回收策略
• expireAfterWrite(long, TimeUnit) 
当缓存项在指定的时间段内没有更新就会被回收 如果我们认为缓存数据在一段时
间后不再可用，那么可以使用该种策略



雪崩 某段时间 key大量失效，缓存失效、穿透--直接打到数据库






<dependency> 
groupid org.springfr nework.boot< groupid
<artifactid>spring-boot-starter-data-mongodb</artifact d>
</dependency>



Mongodb 除了能够存 据外，还内 个非 系统 基于
Mongodb 群的优势， GridFS 当然 是分布式的 而且 份也方便 用户把 件上传到
GridFS 割成 256KB 的块， 井单


public static void uploadFile() throws Exception { 
File file = new File {” / Users/yi njihuan/Downlaods/logo.png”); 
nputStream content = new FileinputStrearn （丑le);
存储文件的额外信息 比如用户 ID 后面要查询某个用户的所有文件时就可以直接查询
DBObject rnetadata = new BasicDBObject （” user ”，” 1001 ”）；
GridFSFile gridFSFile = gridFsTernplate.store( 
content, file.getN ne （），” irnag e/ png ”, rne t adata) ; 
String file = gridFSFile get d() .toStri ng(); 
Systern.out.println( file d);
根据文 ID 查询文件，见代码清单 15-34:
代码清单 15-34 根据文件 ID 查询文件
public static GridFSDBFile getFile (String fil e !d) throws Ex ception { 
return gridFsTernplate.findOne ( 
Query.query(Crite ria.where (” id”) .is(fi leid)));


public static void rernoveFile(String file!d) throws Except ion { 
gridFsTernplate.delete (Query.query(Criteria .where (” id"). is (fileid)));


@RequestMapping(value ＝” rnage {file ｝”）
@ResponseBody 
public void getirnage(@PathVariable String file id, 
HttpServletResponse response) { 
try { 
GridFSDBFile g r idfs = filesService.getFile( fileid); 
response.setContentType(gridfs.getContentType()); 
OutputStream out= response.getOutputStrearn(); 
gridfs.writeTo(out);
out.flush(); 
out.close(); 
} catch (Exc e ption e) { 
e.printStackTra ce();


自增序列
@Document(col l e ction = ” s equ e nce ”) 
public class Sequence { 
＠工
private String id ; 
@Field {”seq i d ”) 
private long seq d;
• seqld : 自增 ID
• ollN me 集合名称
下面我 定义个注解来标识 字段要自动增长 ID 有些场景下可能不需要自动增长，
需要自动增长的 候我们需要加上这个注解，见代码清单 5-48
代码清 15-48 定义
@Retention(RetentionPolicy . RUNTIME) 
@Target ( { Element Type . FIELD } ) 
public @interface GeneratedValue { 
来定义我们测试的实体类，注意自增 ID 的类型不要定义成 Long 种包装类，
mongotemp late 源码里面对主键 ID 的类型有限制，见代码清单 15-
码清 15-49 使用
@Document 
public c l ass Studen t { 
@GeneratedVa l u e 
＠工
private long i d; 
private Strin g n ame ; 1

findAndModify（）是原子操


<dependency> 
<group d>org.springfrarnework .boot</group d>
<artifactid>spring-boot-starter-jdbc</artifactid> 
</dependency>


<dependency> 
<group d>org.springfrarnework.boot</groupid>
<artifactid>spring-boot-starter-data-elasticsearch</artifactid> 
</dependency> 
<dependency> 
<group d>net. java. dev. jna</group d>
<artifactid>jna</artifactid> 
<scope>runtime</scope> 
</dependency>

#关于打包 目标第三方可直接引入使用，组件项目打包一定要把build 元素注释掉，否则别人无法引入 jar包

#ApplicationContextAware接口实现，Spring上下文环境，ApplicationContext 是一个高级形态的 IoC 容器，它在 BeanFactory 基础上附加了好多其他功能， 提供了更多面向框架的功能，因此我们一般使用 ApplicationContext 进行开发

public interface ApplicationContextAware extends Aware


#信号量初始化，初始化变量放到一个map中，初始化许可证多少个，键值对

/**
 * 对 API 进行访问速度限制 <br>
 * 限制的速度值在 Apollo 配置中通过 key 关联
 * 
 * @author yinjihuan
 *
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ApiRateLimit {
	/**
	 * Apollo 配置中的 key
	 * 
	 * @return
	 * 
	 */
	String confKey();
}

@Component
public class InitApiLimitRateListener implements ApplicationContextAware {

	public void setApplicationContext(ApplicationContext ctx) throws BeansException {
		Environment environment = ctx.getEnvironment();
		String defaultLimit = environment.getProperty("open.api.defaultLimit");
		Object rate = defaultLimit == null ? 100 : defaultLimit; 
		ApiLimitAspect.semaphoreMap.put("open.api.defaultLimit", new Semaphore(Integer.parseInt(rate.toString())));
		Map<String, Object> beanMap = ctx.getBeansWithAnnotation(RestController.class);
		Set<String> keys = beanMap.keySet();
		for (String key : keys) {
			Class<?> clz = beanMap.get(key).getClass();
			String fullName = beanMap.get(key).getClass().getName();
			if (fullName.contains("EnhancerBySpringCGLIB") || fullName.contains("$$")) {
				fullName = fullName.substring(0, fullName.indexOf("$$"));
				try {
					clz = Class.forName(fullName);
				} catch (ClassNotFoundException e) {
					throw new RuntimeException(e);
				}
			}
			Method[] methods = clz.getMethods();
			for (Method method : methods) {
				if (method.isAnnotationPresent(ApiRateLimit.class)) {
					String confKey = method.getAnnotation(ApiRateLimit.class).confKey();
					if (environment.getProperty(confKey) != null) {//这儿有个bug
						int limit = Integer.parseInt(environment.getProperty(confKey));
						ApiLimitAspect.semaphoreMap.put(confKey, new Semaphore(limit));
					}
				}
			}
		}
	}
}

#拿到信号量，才可继续访问，并发数控制

/**
 * 具体 API 并发控制
 * 
 * @author yinjihuan
 *
 */
@Aspect
@Order(value = Ordered.HIGHEST_PRECEDENCE)
public class ApiLimitAspect {
	public static Map<String, Semaphore> semaphoreMap = new ConcurrentHashMap<String, Semaphore>();

	@Around("execution(* com.cxytiandi.eureka_client.controller.*.*(..))")
	public Object around(ProceedingJoinPoint joinPoint) {
		Object result = null;
		Semaphore semap = null;
		Class<?> clazz = joinPoint.getTarget().getClass();
		String key = getRateLimitKey(clazz, joinPoint.getSignature().getName());
		if (key != null) {
			semap = semaphoreMap.get(key);
		} else {
			semap = semaphoreMap.get("open.api.defaultLimit");
		}
		try {
			semap.acquire();
			result = joinPoint.proceed();
		} catch (Throwable e) {
			throw new RuntimeException(e);
		} finally {
			semap.release();
		}
		return result;
	}

	private String getRateLimitKey(Class<?> clazz, String methodName) {
		for (Method method : clazz.getDeclaredMethods()) {
			if (method.getName().equals(methodName)) {

				if (method.isAnnotationPresent(ApiRateLimit.class)) {
					String key = method.getAnnotation(ApiRateLimit.class).confKey();
					return key;
				}
			}
		}
		return null;
	}
}


#转换方法  数组转 list  , Arrays.asList

String[] uids = userIds.split(",");
                     if (Arrays.asList(uids)

负载均衡原则：

配置负载规则

zuul-extend-article-service.ribbon.NFLoadBalancerRuleClassName=com.cxytiandi.zuul_demo.rule.GrayPushRule


/**
 * 灰度发布转发规则，基于RoundRobinRule规则改造
 *
 * @author yinjihuan
 *
 **/
public class GrayPushRule extends AbstractLoadBalancerRule {//在配置文件中配置使用
    private AtomicInteger nextServerCyclicCounter;
    private static final boolean AVAILABLE_ONLY_SERVERS = true;
    private static final boolean ALL_SERVERS = false;
    private static Logger log = LoggerFactory.getLogger(RoundRobinRule.class);

    public GrayPushRule() {
        this.nextServerCyclicCounter = new AtomicInteger(0);
    }

    public GrayPushRule(ILoadBalancer lb) {
        this();
        this.setLoadBalancer(lb);
    }

    public Server choose(ILoadBalancer lb, Object key) {//选出灰度用户、灰度服务器
        if (lb == null) {
            log.warn("no load balancer");
            return null;
        } else { //看来是灰度的用户 遇到 灰度的服务器 ，首先是灰度用户，然后再选出灰度的服务器
            // 当前有灰度的用户和灰度的服务配置信息，并且灰度的服务在所有服务中则返回该灰度服务给用户
            String curUserId = RibbonFilterContextHolder.getCurrentContext().get("userId");
            String userIds = RibbonFilterContextHolder.getCurrentContext().get("userIds");
            String servers = RibbonFilterContextHolder.getCurrentContext().get("servers");
            System.out.println(Thread.currentThread().getName()+":"+servers);
            System.out.println("所有服务器："+lb.getAllServers());
            if (StringUtils.isNotBlank(servers)) {
            	 List<String> grayServers = Arrays.asList(servers.split(","));
                 if (StringUtils.isNotBlank(userIds) && StringUtils.isNotBlank(curUserId)) {
                     String[] uids = userIds.split(",");
                     if (Arrays.asList(uids).contains(curUserId)) {
                         List<Server> allServers = lb.getAllServers();//所有服务器
                         for (Server server : allServers) {//host:port  那么配置也得是 houst:port
                             if (grayServers.contains(server.getHostPort())) {
                                 return server;
                             }
                         }
                     }
                 }
			}

            Server server = null;
            int count = 0;

            while(true) {//非灰度用户选择，非灰度的服务器
                if (server == null && count++ < 10) {
                    List<Server> reachableServers = lb.getReachableServers();
                    List<Server> allServers = lb.getAllServers();
                    // 移除已经设置为灰度发布的服务信息
                    if (StringUtils.isNotBlank(servers)) {//获取可用的Server集合
                    	 reachableServers = removeServer(reachableServers, servers);
                         allServers = removeServer(allServers, servers);
                    }
                    int upCount = reachableServers.size();
                    int serverCount = allServers.size();
                    if (upCount != 0 && serverCount != 0) {//循环选择  从数组中选择
                        int nextServerIndex = this.incrementAndGetModulo(serverCount);
                        server = (Server)allServers.get(nextServerIndex);//排除了灰度服务器之后的服务器中选择
                        if (server == null) {
                            Thread.yield();
                        } else {
                            if (server.isAlive() && server.isReadyToServe()) {
                                return server;
                            }

                            server = null;
                        }
                        continue;
                    }

                    log.warn("No up servers available from load balancer: " + lb);
                    return null;
                }

                if (count >= 10) {
                    log.warn("No available alive servers after 10 tries from load balancer: " + lb);
                }

                return server;
            }
        }
    }


    private List<Server> removeServer(List<Server> allServers, String servers) {
        List<Server> newServers = new ArrayList<Server>();
        List<String> grayServers = Arrays.asList(servers.split(","));
        for (Server server : allServers) {
            String hostPort = server.getHostPort();
            if (!grayServers.contains(hostPort)) {
                newServers.add(server);
            }
        }
        return newServers;
    }

    private int incrementAndGetModulo(int modulo) {
        int current;
        int next;
        do {
            current = this.nextServerCyclicCounter.get();
            next = (current + 1) % modulo;
        } while(!this.nextServerCyclicCounter.compareAndSet(current, next));

        return next;
    }

    public Server choose(Object key) {
        return this.choose(this.getLoadBalancer(), key);
    }

    public void initWithNiwsConfig(IClientConfig clientConfig) {
    }
}

----------------------------------------------------------------
#聚群限流，网关上可以控制(通过redis很好控制)


String key = "fsh-api-rate-limit-" + currentSecond;
try {
    if (!redisTemplate.hasKey(key)) {
        redisTemplate.opsForValue().set(key, 0L, 100, TimeUnit.SECONDS);
    }

    // 当集群中当前秒的并发量达到了设定的值，不进行处理，注意集群中的网关所在服务器时间必须同步
    if (redisTemplate.opsForValue().increment(key, 1) > basicConf.getClusterLimitRate()) {
        ctx.setSendZuulResponse(false);
        ctx.set("isSuccess", false);
        ResponseData data = ResponseData.fail("当前负载太高，请稍后重试", ResponseCode.LIMIT_ERROR_CODE.getCode());
        ctx.setResponseBody(JsonUtils.toJson(data));
        ctx.getResponse().setContentType("application/json; charset=utf-8");
        return null;
    }
} 

-------------------------------

#认证过滤器：

/**
 * 认证过滤器
 *
 * @author yinjihuan
 * 
 **/
public class AuthFilter extends ZuulFilter {

    @Autowired
    private BasicConf basicConf;

    public AuthFilter() {
        super();
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public String filterType() {
        return "pre";
    }

    @Override
    public int filterOrder() {
        return 1;
    }

    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        String token = ctx.getRequest().getHeader("Authorization");
        JWTUtils jwtUtils = JWTUtils.getInstance();
        String apis = basicConf.getApiWhiteStr();
        //白名单，放过
        List<String> whileApis = Arrays.asList(apis.split(","));
        String uri = ctx.getRequest().getRequestURI();
        if (whileApis.contains(uri)) {
            return null;
        }
        // path uri 处理
        for (String wapi : whileApis) {
            if (wapi.contains("{") && wapi.contains("}")) {
                if (wapi.split("/").length == uri.split("/").length) {
                    String reg = wapi.replaceAll("\\{.*}", ".*{1,}");
                    System.err.println(reg);
                    Pattern r = Pattern.compile(reg);
                    Matcher m = r.matcher(uri);
                    if (m.find()) {
                        return null;
                    }
                }
            }
        }

        //验证TOKEN
        if (!StringUtils.hasText(token)) {
            ctx.setSendZuulResponse(false);
            ctx.set("isSuccess", false);
            ResponseData data = ResponseData.fail("非法请求【缺少Authorization信息】", ResponseCode.NO_AUTH_CODE.getCode());
            ctx.setResponseBody(JsonUtils.toJson(data));
            ctx.getResponse().setContentType("application/json; charset=utf-8");
            return null;
        }

        JWTUtils.JWTResult jwt = jwtUtils.checkToken(token);
        if (!jwt.isStatus()) {
            ctx.setSendZuulResponse(false);
            ctx.set("isSuccess", false);
            ResponseData data = ResponseData.fail(jwt.getMsg(), jwt.getCode());
            ctx.setResponseBody(JsonUtils.toJson(data));
            ctx.getResponse().setContentType("application/json; charset=utf-8");
            return null;
        }
        System.err.println("用戶ID"+jwt.getUid());
        ctx.addZuulRequestHeader("uid", jwt.getUid());
        return null;
    }
}
-------------------------------------------------------------------------------------

@Value("${grayPushServers:192.168.247.1:8082}") //default
	private String grayPushServers;


// API接口白名单，多个用逗号分隔
	@Value("${apiWhiteStr:/zuul-extend-user-service/user/login}")
	private String apiWhiteStr;

@Value("${grayPushServers:192.168.247.1:8082}") //default
	private String grayPushServers;


@ApolloConfig     携程阿波罗 ，看来还可以嘛
private Config config;
	
	@ApolloConfigChangeListener   //监听数据变化
	public void onChange(ConfigChangeEvent changeEvent) {
		if (changeEvent.isChanged("limitRate")) {
			// 更 新 RateLimiter
			System.err.println(config.getDoubleProperty("limitRate", 10.0));
			LimitFilter.rateLimiter = RateLimiter.create(config.getDoubleProperty("limitRate", 10.0));
		}
	}
---------------------------------------------

#注意信息存储传递  采用的方式


import java.util.Map;

public interface RibbonFilterContext {
    RibbonFilterContext add(String key, String value);

    String get(String key);

    RibbonFilterContext remove(String key);

    Map<String, String> getAttributes();
}


public class DefaultRibbonFilterContext implements RibbonFilterContext {//实现类 实现接口
    private final Map<String, String> attributes = new HashMap<String, String>();

    @Override
    public RibbonFilterContext add(String key, String value) {
        attributes.put(key, value);
        return this;
    }

    @Override
    public String get(String key) {
        return attributes.get(key);
    }

    @Override
    public RibbonFilterContext remove(String key) {
        attributes.remove(key);
        return this;
    }

    @Override
    public Map<String, String> getAttributes() {
        return Collections.unmodifiableMap(attributes);
    }
}


import com.alibaba.ttl.TransmittableThreadLocal;

public class RibbonFilterContextHolder {
    private static final TransmittableThreadLocal<RibbonFilterContext> contextHolder = new TransmittableThreadLocal<RibbonFilterContext>() {
        @Override
        protected RibbonFilterContext initialValue() {//这儿有个具体的实现类 挂载上来
            return new DefaultRibbonFilterContext();
        }
    };


    public static RibbonFilterContext getCurrentContext() {//返回类  里面是个map
        return contextHolder.get();
    }


    public static void clearCurrentContext() {//销毁
        contextHolder.remove();
    }
}


#TransmittableThreadLocal的使用，很好的传递了变量到子线程中
#//子线程每次new 所以会复制线程的InheritableThreadLocal,结果正确
#//因线程池复用线程,不会每次new 所以不会更新父线程InheritableThreadLocal 的值,导致结果错误

阿里开源了  TransmittableThreadLocal
<dependency>
	<groupId>com.alibaba</groupId>
	<artifactId>transmittable-thread-local</artifactId>
	<version>2.2.0</version>
</dependency>

---------------------fegin 拦截器，传递头到下一个服务器

@Bean
	public FeignBasicAuthRequestInterceptor basicAuthRequestInterceptor() {
		return new FeignBasicAuthRequestInterceptor();
	}
	

/**
 * 传递用户信息到被调用的服务
 * 
 * @author yinjihuan
 *
 */
public class FeignBasicAuthRequestInterceptor implements RequestInterceptor {
	public FeignBasicAuthRequestInterceptor() {

	}

	@Override
	public void apply(RequestTemplate template) {
		Map<String, String> attributes = RibbonFilterContextHolder.getCurrentContext().getAttributes();
		for (String key : attributes.keySet()) {
			String value = attributes.get(key);
			template.header(key, value);
		}
	}

}
---------------------------------------------------

#普通链接加入 过滤器，接收请求 同时加入自己的过滤器，然后把想要挂载进来的东西 挂载进来 RibbonFilterContextHolder.getCurrentContext().add("uid", uid);

@Bean
	public FilterRegistrationBean<HttpHeaderParamFilter> filterRegistrationBean() {
		FilterRegistrationBean<HttpHeaderParamFilter> registrationBean = new FilterRegistrationBean<HttpHeaderParamFilter>();
		HttpHeaderParamFilter httpHeaderParamFilter = new HttpHeaderParamFilter();
		registrationBean.setFilter(httpHeaderParamFilter);
		List<String> urlPatterns = new ArrayList<String>(1);
		urlPatterns.add("/*");
		registrationBean.setUrlPatterns(urlPatterns);
		return registrationBean;
}


/**
 * 接收Zuul过来的用户信息
 * 
 * @author yinjihuan
 *
 */
public class HttpHeaderParamFilter implements Filter {
	
	@Override
	public void init(FilterConfig filterConfig) throws ServletException {
		
	}

	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
			throws IOException, ServletException {
		HttpServletRequest httpRequest = (HttpServletRequest) request;
		HttpServletResponse httpResponse = (HttpServletResponse) response;
		httpResponse.setCharacterEncoding("UTF-8");
		httpResponse.setContentType("application/json; charset=utf-8");
		String uid = httpRequest.getHeader("uid");
		RibbonFilterContextHolder.getCurrentContext().add("uid", uid);//此处挂靠进来 暂存
		chain.doFilter(httpRequest, response);
	}

	@Override
	public void destroy() {

	}
}

--------------------------------------------

@Bean
	@LoadBalanced
	public RestTemplate RestTemplate() {
		return  new  RestTemplate();
	}


#服务降级特殊处理  zuulfilter

@Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        Object serviceId = ctx.get("serviceId");
        System.out.println(serviceId);
        if (serviceId != null && basicConf != null) {
            List<String> serviceIds = Arrays.asList(basicConf.getDownGradeServiceStr().split(","));
            if (serviceIds.contains(serviceId.toString())) {//配置的服务为逗号分割的，如果用户访问的这个服务则 返回必要的信息而终止掉
                ctx.setSendZuulResponse(false);//后续filter会用到
                ctx.set("isSuccess", false); //后续filter会用到
                ResponseData data = ResponseData.fail("服务降级中", ResponseCode.DOWNGRADE.getCode());
                ctx.setResponseBody(JsonUtils.toJson(data));
                ctx.getResponse().setContentType("application/json; charset=utf-8");
                return null;
            }
        }
        return null;
    }

---------------------------------------------------------------

#本地缓存 guava ，缓存只是 guava中的部分功能，可以设置过期时间
  <dependency>
			<groupId>com.google.guava</groupId>
			<artifactId>guava</artifactId>
			<version>15.0</version>
		</dependency>


import java.util.concurrent.TimeUnit;
import com.google.common.cache.CacheBuilder;
import com.google.common.cache.CacheLoader;
import com.google.common.cache.LoadingCache;

public class App {
	public static void main(String[] args) {
		final PersonDao dao = new PersonDao();
		LoadingCache<String, Person> cahceBuilder = CacheBuilder.newBuilder().expireAfterWrite(1, TimeUnit.SECONDS)
				.build(new CacheLoader<String, Person>() {//建立提取数据 同时设置过期时间  expireAfterWrite
					@Override
					public Person load(String key) throws Exception {
						return dao.findById(key);
					}
				});
		
		try {
			for(;;) {
				Person person = cahceBuilder.get("1");//这个是提取数据 实质执行的上一个 内部封装 build
				System.out.println(person.getName());
				Thread.sleep(200);
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	

}


#redis  分布式锁

<dependency>
		    <groupId>org.redisson</groupId>
		    <artifactId>redisson-spring-boot-starter</artifactId>
		    <version>3.10.1</version>
		</dependency>

public void testReentrantLock(RedissonClient redisson){

        RLock lock = redisson.getLock("anyLock");
        try{
            // 1. 最常见的使用方法
            //lock.lock();

            // 2. 支持过期解锁功能,10秒钟以后自动解锁, 无需调用unlock方法手动解锁
            //lock.lock(10, TimeUnit.SECONDS);

            // 3. 尝试加锁，最多等待3秒，上锁以后10秒自动解锁
            boolean res = lock.tryLock(3, 10, TimeUnit.SECONDS);
            if(res){    //成功
                // do your business

            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }

    }


public void testAsyncReentrantLock(RedissonClient redisson){
        RLock lock = redisson.getLock("anyLock");
        try{
            lock.lockAsync();
            lock.lockAsync(10, TimeUnit.SECONDS);
            Future<Boolean> res = lock.tryLockAsync(3, 10, TimeUnit.SECONDS);

            if(res.get()){
                // do your business

            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }

    }

#框架存储 缓存数据
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-redis</artifactId>
		</dependency>


@Data
@RedisHash("persons")
public class Person {
	@Id
	String id;
	String firstname;
	String lastname;
}


import org.springframework.data.repository.CrudRepository;
import org.springframework.stereotype.Repository;

import com.cxytiandi.cache_data_redis.po.Person;

@Repository
public interface PersonRepository extends CrudRepository<Person, String> {

}


@GetMapping("/test2")
	public void basicCrudOperations() {
		Person person = new Person();
		person.setFirstname(" 尹吉欢 ");
		person.setLastname("yinjihuan");
		repo.save(person);
		Optional<Person> personObj = repo.findById(person.getId());
		System.err.println(personObj.get().getFirstname());
		System.err.println(repo.count());
		//repo.delete(person);
	}


#@Cacheable 自动管理存储
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

import com.cxytiandi.cache_data_redis.po.Person;

@Service
public class PersonServiceImpl implements PersonService {
	
	@Cacheable(value = "get", key = "#id")
	public Person get(String id) {
		Person p = new Person();
		p.setFirstname("xxx");
		p.setLastname("bbb");
		p.setId("111");
		return p;
	}
	
}

@GetMapping("/get")
	public Person get() {
		return personService.get("1001");
	}


#redis自定义缓存处理

package com.cxytiandi.cache_data_redis.service;

import java.util.concurrent.TimeUnit;

public interface CacheService {//接口
	/**
	 * 设置缓存
	 * 
	 * @param key      缓存 KEY
	 * @param value    缓存值
	 * @param timeout  缓存过期时间
	 * @param timeUnit 缓存过期时间单位
	 */
	public void setCache(String key, String value, long timeout, TimeUnit timeUnit);

	/**
	 * 获取缓存
	 * 
	 * @param key 缓 存 KEY
	 * @return
	 */
	public String getCache(String key);

	public <V, K> String getCache(K key, Closure<V, K> closure);

	public <V, K> String getCache(K key, Closure<V, K> closure, long timeout, TimeUnit timeUnit);

	/**
	 * 删除缓存
	 * 
	 * @param key 缓 存 KEY
	 */
	public void deleteCache(String key);
}



import java.util.concurrent.TimeUnit;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.stereotype.Service;

import com.cxytiandi.cache_data_redis.config.CacheAutoConfiguration;

@Service
public class CacheServiceImpl implements CacheService {//实现
	private Logger logger = LoggerFactory.getLogger(CacheAutoConfiguration.class);
	@Autowired
	private StringRedisTemplate stringRedisTemplate;

	private long timeout = 1L;

	private TimeUnit timeUnit = TimeUnit.HOURS;

	@Override
	public void setCache(String key, String value, long timeout, TimeUnit timeUnit) {
		try {
			stringRedisTemplate.opsForValue().set(key, value, timeout, timeUnit);
		} catch (Exception e) {
			logger.error("",e);
		}
	}

	@Override
	public String getCache(String key) {
		try {
			return stringRedisTemplate.opsForValue().get(key);
		} catch (Exception e) {
			logger.error("",e);
			return null;
		}
	}

	@Override
	public void deleteCache(String key) {
		stringRedisTemplate.delete(key);
	}

	@Override
	public <V, K> String getCache(K key, Closure<V, K> closure) {
		return doGetCache(key, closure, this.timeout, this.timeUnit);
	}

	@Override
	public <V, K> String getCache(K key, Closure<V, K> closure, long timeout, TimeUnit timeUnit) {
		return doGetCache(key, closure, timeout, timeUnit);
	}

	private <K, V> String doGetCache(K key, Closure<V, K> closure, long timeout, TimeUnit timeUnit) {
		String ret = getCache(key.toString());
		if (ret == null) {
			Object r = closure.execute(key);
			setCache(key.toString(), r.toString(), timeout, timeUnit);
			return r.toString();
		}
		return ret;
	}

}


接口

public interface Closure<O, I> {//这个接口定义 比较高
	public O execute(I input);
}

@GetMapping("/getCallback")
	public String getCallback() {
		String cacheKey = "1001";
		return cacheService.getCache(cacheKey, new Closure<String, String>() {//
			@Override
			public String execute(String id) {//输出很模糊，需要加工的，  但就是这个输出结果作为存在redis中的value 
				// 执行你的业务逻辑
				return id+"hello";
			}
		});
	}



参考下面方法可见 key是对应的关系 即上面的 cacheKey, id  ---------- 整个自定义框架感觉还可以呀
@Override
public <V, K> String getCache(K key, Closure<V, K> closure) {//泛型定义 注意后面的泛型 前面一定得出现，且注意各自的对应位置
		return doGetCache(key, closure, this.timeout, this.timeUnit);
}

#以上的泛型思路得留心学习下，比较好
----------------------------

#mongdb
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-mongodb</artifactId>
		</dependency>


import java.util.Date;
import java.util.List;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;
import org.springframework.data.mongodb.core.mapping.Field;

import com.cxytiandi.mongodb.autoid.GeneratedValue;

import lombok.Data;

@Data
@Document(collection = "article_info")
public class Article {
	@Id
	@GeneratedValue
	private Long id;

	@Field("title")
	private String title;

	@Field("url")
	private String url;

	@Field("author")
	private String author;

	@Field("tags")
	private List<String> tags;

	@Field("visit_count")
	private Long visitCount;

	@Field("add_time")
	private Date addTime;

}


import org.springframework.data.mongodb.core.index.CompoundIndex;
import org.springframework.data.mongodb.core.index.CompoundIndexes;
import org.springframework.data.mongodb.core.index.Indexed;
import org.springframework.data.mongodb.core.mapping.Document;

import lombok.Data;

@Data
@Document
@CompoundIndexes({ @CompoundIndex(name = "city_region_idx", def = "{'city': 1, 'region': 1}") })
public class Person {
	private String id;

	@Indexed(unique = true)
	private String name;

	@Indexed(background = true)
	private int age;

	private String city;

	private String region;
}


import java.util.List;

import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.data.repository.PagingAndSortingRepository;
import org.springframework.stereotype.Repository;

import com.cxytiandi.mongodb.po.Article;

@Repository("ArticleRepositor")
public interface ArticleRepositor extends PagingAndSortingRepository<Article, String> {
	// 分页查询
	public Page<Article> findAll(Pageable pageable);

	// 根据 author 查询
	public List<Article> findByAuthor(String author);

	// 根据作者和标题查询
	public List<Article> findByAuthorAndTitle(String author, String title);

	// 忽略参数大小写
	public List<Article> findByAuthorIgnoreCase(String author);

	// 忽略所有参数大小写
	public List<Article> findByAuthorAndTitleAllIgnoreCase(String author, String title);

	// 排序
	public List<Article> findByAuthorOrderByVisitCountDesc(String author);

	public List<Article> findByAuthorOrderByVisitCountAsc(String author);

	// 自带排序条件
	public List<Article> findByAuthor(String author, Sort sort);
}



import org.springframework.data.mongodb.core.MongoTemplate;

@Autowired
	private MongoTemplate mongoTemplate;


mongoTemplate.save(article);

mongoTemplate.insert(articles, Article.class);

#mongodb操作  知识点滴

package com.cxytiandi.mongodb.controller;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.Date;
import java.util.List;

import org.bson.Document;
import org.bson.types.ObjectId;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.data.mongodb.core.query.Update;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import com.cxytiandi.mongodb.batchupdate.BathUpdateOptions;
import com.cxytiandi.mongodb.batchupdate.MongoBaseDao;
import com.cxytiandi.mongodb.po.Article;
import com.cxytiandi.mongodb.repository.ArticleRepositor;
import com.mongodb.client.ListIndexesIterable;

@RestController
public class ArticleController {

	@Autowired
	private MongoTemplate mongoTemplate;

	@Autowired
	private ArticleRepositor articleRepositor;
	
	@GetMapping("/save")
	public String save() {
		// 循环添加
		for (int i = 0; i < 10; i++) {
			Article article = new Article();
			article.setTitle("MongoTemplate 的基本使用 ");
			article.setAuthor("yinjihuan");
			article.setUrl("http://cxytiandi.com/blog/detail/" + i);
			article.setTags(Arrays.asList("java", "mongodb", "spring"));
			article.setVisitCount(0L);
			article.setAddTime(new Date());
			mongoTemplate.save(article);
		}
		return "success";
	}

	@GetMapping("/batchSave")
	public String batchSave() {
		// 批量添加
		List<Article> articles = new ArrayList<>(10);
		for (int i = 0; i < 10; i++) {
			Article article = new Article();
			article.setTitle("MongoTemplate 的基本使用 ");
			article.setAuthor("yinjihuan");
			article.setUrl("http://cxytiandi.com/blog/detail/" + i);
			article.setTags(Arrays.asList("java", "mongodb", "spring"));
			article.setVisitCount(0L);
			article.setAddTime(new Date());
			articles.add(article);
		}
		mongoTemplate.insert(articles, Article.class);
		return "success";
	}
	
	@GetMapping("/indexList")
	public String indexList() {
		ListIndexesIterable<Document> list = mongoTemplate.getCollection("person").listIndexes();//返回索引
		for (Document document : list) {
			System.out.println(document.toJson());
		}
		return "success";
	}
	
	@GetMapping("/update")
	public String update() {
		Query query = Query.query(Criteria.where("author").is("yinjihuan")); 
		Update update = Update.update("title", "MongoTemplate")
				.set("visitCount", 10); 
		mongoTemplate.updateFirst(query, update, Article.class);
		
		query = Query.query(Criteria.where("author").is("yinjihuan"));
		update = Update.update("title", "MongoTemplate").set("visitCount", 10); 
		mongoTemplate.updateMulti(query, update, Article.class);
		
		query = Query.query(Criteria.where("author").is("jason"));
		update = Update.update("title", "MongoTemplate").set("visitCount", 10); 
		mongoTemplate.upsert(query, update, Article.class);//有则更新 无则插入数据
		
		query = Query.query(Criteria.where("author").is("jason"));
		update = Update.update("title", "MongoTemplate").set("money", 100); 
		mongoTemplate.updateMulti(query, update, Article.class);
		
		query = Query.query(Criteria.where("author").is("jason"));
		update = Update.update("title", "MongoTemplate").inc("money", 100);//增加操作
		mongoTemplate.updateMulti(query, update, Article.class);
		
		query = Query.query(Criteria.where("author").is("jason")); 
		update = Update.update("title", "MongoTemplate")
				.rename("visitCount", "vc"); //重命名键值  如果重命名键不存在  则什么也不做
		mongoTemplate.updateMulti(query, update, Article.class);
		
		query = Query.query(Criteria.where("author").is("jason")); 
		update = Update.update("title", "MongoTemplate").unset("vc");// unset删除键
		mongoTemplate.updateMulti(query, update, Article.class);
		
		query = Query.query(Criteria.where("author").is("yinjihuan"));
		update = Update.update("title", "MongoTemplate").pull("tags", "java");//pull删除值为啥得行
		mongoTemplate.updateMulti(query, update, Article.class);
		return "success";
	}
	
	@GetMapping("/delete")
	public String delete() {
		Query query = Query.query(Criteria.where("author").is("yinjihuan")); 
		mongoTemplate.remove(query, Article.class);//删除行
		
		query = Query.query(Criteria.where("author").is("yinjihuan")); 
		mongoTemplate.remove(query, "article_info");//删除指定集合中得行
		
		query = Query.query(Criteria.where("author").is("yinjihuan")); 
		Article article = mongoTemplate.findAndRemove(query, Article.class);//查询出符合条件的第一个结果，并将符合条件的数据删除,只会删除第一条
		
		query = Query.query(Criteria.where("author").is("yinjihuan")); 
		List<Article> articles =
				mongoTemplate.findAllAndRemove(query, Article.class);//找出并删除  多个
		
		mongoTemplate.dropCollection(Article.class); //删除集合，可传实体类，也可以传名称
		mongoTemplate.dropCollection("article_info");
		
		mongoTemplate.getDb().drop();
		return "success";
	}
	
	@GetMapping("/query")
	public String query() {
		Query query = Query.query(Criteria.where("author").is("yinjihuan")); 
		List<Article> articles = mongoTemplate.find(query, Article.class);
		
		query = Query.query(Criteria.where("author").is("yinjihuan")); 
		Article article = mongoTemplate.findOne(query, Article.class);
		articles = mongoTemplate.findAll(Article.class);
		
		query = Query.query(Criteria.where("author").is("yinjihuan")); 
		long count = mongoTemplate.count(query, Article.class);
		
		article = mongoTemplate.findById(new ObjectId("57c6e1601e4735b2c306cdb7"), Article.class);
		
		List<String> authors = Arrays.asList("yinjihuan", "jason"); 
		query = Query.query(Criteria.where("author").in(authors)); 
		articles = mongoTemplate.find(query, Article.class);
		
		query = Query.query(Criteria.where("author").ne("yinjihuan")); 
		articles = mongoTemplate.find(query, Article.class);
		
		query = Query.query(Criteria.where("visitCount").lt(10)); 
		articles = mongoTemplate.find(query, Article.class);
		
		query = Query.query(Criteria.where("visitCount").gt(5).lt(10)); 
		articles = mongoTemplate.find(query, Article.class);
		
		query = Query.query(Criteria.where("author").regex("a")); 
		articles = mongoTemplate.find(query, Article.class);
		
		query = Query.query(Criteria.where("tags").size(3)); 
		articles = mongoTemplate.find(query, Article.class);
		
		query = Query.query(Criteria.where("").orOperator( Criteria.where("author").is("jason"), Criteria.where("visitCount").is(0)));
		articles = mongoTemplate.find(query, Article.class);
		return "success";
	}
	
	@GetMapping("/findAll")
	public Object findAll() {
		return articleRepositor.findByAuthor("yinjihuan");
	}
	
	@GetMapping("/batchUpdate")
	public Object batchUpdate() {
		List<BathUpdateOptions> list = new ArrayList<BathUpdateOptions>();
		list.add(new BathUpdateOptions(Query.query(Criteria.where("author").is("yinjihuan")),Update.update("title", "批量更新"), true, true));
		list.add(new BathUpdateOptions(Query.query(Criteria.where("author").is("jason")),Update.update("title", "批量更新"), true, true));
		int n = MongoBaseDao.bathUpdate(mongoTemplate, "article_info", list, true);
		System.out.println("受影响的行数："+n);
		return n;
	}
}


#gridfs 文件操作哈

import java.io.File;
import java.io.FileInputStream;
import java.io.InputStream;
import java.io.OutputStream;

import javax.servlet.http.HttpServletResponse;

import org.apache.tomcat.util.http.fileupload.IOUtils;
import org.bson.types.ObjectId;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.data.mongodb.gridfs.GridFsResource;
import org.springframework.data.mongodb.gridfs.GridFsTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import com.mongodb.BasicDBObject;
import com.mongodb.DBObject;
import com.mongodb.client.gridfs.GridFSBucket;
import com.mongodb.client.gridfs.GridFSBuckets;
import com.mongodb.client.gridfs.GridFSDownloadStream;
import com.mongodb.client.gridfs.model.GridFSFile;

@RestController
public class GridFsController {

	@Autowired
	private GridFsTemplate gridFsTemplate;

	@Autowired
	private MongoTemplate mongoTemplate;
	
	/**
	 * 上传文件
	 * 
	 * @author yinjihuan
	 * @throws Exception
	 */
	@GetMapping("/image/upload")
	public String uploadFile() throws Exception {
		File file = new File("/Users/yinjihuan/Downloads/logo.png");
		InputStream content = new FileInputStream(file);
		// 存储文件的额外信息，比如用户ID,后面要查询某个用户的所有文件时就可以直接查询
		DBObject metadata = new BasicDBObject("userId", "1001");//这个思路好
		ObjectId fileId = gridFsTemplate.store(content, file.getName(), "image/png", metadata);
		System.out.println(fileId.toString());
		return "success";
	}

	/**
	 * 根据文件ID查询文件
	 * 
	 * @author yinjihuan
	 * @param fileId
	 * @return
	 * @throws Exception
	 */
	@GetMapping("/image/get")
	public String getFile(String fileId) throws Exception {
		GridFSFile file = gridFsTemplate.findOne(Query.query(Criteria.where("_id").is(fileId)));
		System.err.println(file.getFilename());
		return "success";
	}

	/**
	 * 根据文件ID删除文件
	 * 
	 * @author yinjihuan
	 * @param fileId
	 * @throws Exception
	 */
	@GetMapping("/image/remove")
	public String removeFile(String fileId) throws Exception {
		gridFsTemplate.delete(Query.query(Criteria.where("_id").is(fileId)));
		return "success";
	}

	/**
	 * 下载文件
	 * @param fileId
	 * @param response
	 */
	@GetMapping("/image/{fileId}")
	public void getImage(@PathVariable String fileId, HttpServletResponse response) {
		try {
			GridFSFile gridfs = gridFsTemplate.findOne(Query.query(Criteria.where("_id").is(fileId)));
			response.setHeader("Content-Disposition", "attachment;filename=\"" + gridfs.getFilename() + "\"");
			GridFSBucket gridFSBucket = GridFSBuckets.create(mongoTemplate.getDb());
			GridFSDownloadStream gridFSDownloadStream = gridFSBucket.openDownloadStream(gridfs.getObjectId());
			GridFsResource resource = new GridFsResource(gridfs, gridFSDownloadStream);
			InputStream inp = resource.getInputStream();
			OutputStream out = response.getOutputStream();
			IOUtils.copy(inp, out);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}

------------------------------------

#elaticsearch

<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-elasticsearch</artifactId>
		</dependency>
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<optional>true</optional>
		</dependency>

注意版本间得应用差异大

spring.data.elasticsearch.repositories.enabled=true
spring.elasticsearch.rest.uris=http://192.168.247.1:9200
spring.data.elasticsearch.client.reactive.connection-timeout=3000
spring.data.elasticsearch.client.reactive.socket-timeout=3000



import lombok.Data;

注意配置文档时配置了库名称 indexName = "cxytiandi" ，同时配置了文档名称
@Data
@Document(indexName = "cxytiandi", indexStoreType = "article")
public class Article {
	
	@Id    
    @Field(type = FieldType.Integer)    
	private Integer id;
	
	@Field(type = FieldType.Keyword)   
	private String sid;
	
	@Field(type = FieldType.Keyword,  analyzer = "ik_max_word", searchAnalyzer = "ik_max_word") 
	private String title;
	
	@Field(type = FieldType.Keyword) 
	private String url;
	
	@Field(type = FieldType.Keyword) 
	private String content;
	
}


#jpa访问方式

import java.util.List;

import org.springframework.data.repository.CrudRepository;
import org.springframework.stereotype.Repository;
import com.cxytiandi.elasticsearch.po.Article;

@Repository
public interface ArticleRepository extends CrudRepository<Article, Long> {
	
	List<Article> findByTitleContaining(String title);
	
} 


#springboot 模板访问方式 ElasticsearchRestTemplate
@Autowired
	private ElasticsearchRestTemplate   elasticsearchRestTemplate;


@Test
	public void testQueryByOr() {
		SearchHits<Article> search= elasticsearchRestTemplate.search(new CriteriaQuery(Criteria.where("title").is("java").or("sid").is("dak219dksd1")), Article.class);
		search.forEach(searchHit-> System.out.println(searchHit.getContent()));

	}


#处理日期问题
@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
或
spring.jackson.time-zone=GMT+8
spring.jackson.date-format=yyyy-MM-dd HH:mm:ss

# 这也是一个思路
@EnableDiscoveryClient
@EnableFeignClients(basePackages = "com.fangjia.mqclient")
@SpringBootApplication
public class TransactionTaskApplication {
	private static final Logger LOGGER = LoggerFactory.getLogger(TransactionTaskApplication.class);
	
	public static void main(String[] args) {
		SpringApplication application = new SpringApplication(TransactionTaskApplication.class);
		ConfigurableApplicationContext content = application.run(args);
		ProcessMessageTask task = content.getBean(ProcessMessageTask.class);//拿出来 开始执行
		task.start();

----------

AmqpTemplate

#注意资源数、线程数  最多放多少过去竞争、拿到令牌的 再去竞争执行


@Service
public class ProcessMessageTask {
	private static final Logger LOGGER = LoggerFactory.getLogger(ProcessMessageTask.class);
	
	@Autowired
	private TransactionMqRemoteClient transactionMqRemoteClient;
	
	@Autowired
	private Producer producer;
	
	@Autowired
	private RedissonClient redisson;
	
	private ExecutorService fixedThreadPool = Executors.newFixedThreadPool(10);
	
	private Semaphore semaphore = new Semaphore(20);//20个怼10个
	
	public void start() {//这个名字无所谓
		Thread th = new Thread(new Runnable() {//这是启动的主线程
			
			public void run() {
				while(true) {//非定时器方式 而是 直接线程  或者说是主线程  主线程再次执行一定是各子线程执行完毕后
					final RLock lock = redisson.getLock("transaction-mq-task");//分布式锁
					try {//分布式锁
						lock.lock();
						System.out.println("开始发送消息:" + DateFormatUtils.format(new Date(), "yyyy-MM-dd HH:mm:ss"));
						int sleepTime = process();
						if (sleepTime > 0) {//延迟10秒
							Thread.sleep(10000);
						}
					} catch (Exception e) {
						LOGGER.error("", e);
					} finally {
						lock.unlock();
					}
				}
			}
		});
		th.start();
	}
	
	private int process() throws Exception {
		int sleepTime = 10000;	//默认执行完之后等等10秒
		List<TransactionMessage> messageList = transactionMqRemoteClient.findByWatingMessage(5000);
		if (messageList.size() == 5000) {
			sleepTime = 0;
		}
		final CountDownLatch latch = new CountDownLatch(messageList.size());
		for (final TransactionMessage message : messageList) {
			semaphore.acquire();//满了信号量 将被锁住等待
			fixedThreadPool.execute(new Runnable() {//这个才是真正的执行线程  超过线程池数量 也将会在无界队列等待等待
				
				public void run() {
					try {
						doProcess(message);
					} catch (Exception e) {
						LOGGER.error("", e);
					} finally {
						semaphore.release();
						latch.countDown();
					}
				}
			});
		}
		latch.await();
		return sleepTime;
	}
	
	private void doProcess(TransactionMessage message) {
		//检查此消息是否满足死亡条件
		if (message.getSendCount() > message.getDieCount()) {//超过次数 进入死信队列
			transactionMqRemoteClient.confirmDieMessage(message.getId());
			return;
		}
		
		//距离上次发送时间超过一分钟才继续发送
		long currentTime = System.currentTimeMillis();
		long sendTime = 0;
		if (message.getSendDate() != null) {//首次为空 则会立即发送
			sendTime = message.getSendDate().getTime();
		}
		if (currentTime - sendTime > 60000) {//控制可能会 在晚些时候被处理掉
			System.out.println("发送具体消息：" + message.getId());
			
			//向MQ发送消息
			MessageDto messageDto = new MessageDto();
			messageDto.setMessageId(message.getId());
			messageDto.setMessage(message.getMessage());
			producer.send(message.getQueue(), JsonUtils.toJson(messageDto));
			
			//修改消息发送次数以及最近发送时间
			transactionMqRemoteClient.incrSendCount(message.getId(), DateFormatUtils.format(new Date(), "yyyy-MM-dd HH:mm:ss"));
			
		}
	}
}


#可靠消费、数据库方式

/**
 * 可靠消息服务消费类
 * @author yinjihuan
 *
 */
@Component
public class ActiveMqConsumer {


	@Autowired
	private TransactionMqRemoteClient transactionMqRemoteClient;
	
	// 小区名称修改操作
	//@JmsListener(destination = "house_queue")

	@RabbitListener(queues ="house_queue")
	@RabbitHandler
	public void process(String content, Channel channel, Message message) throws IOException {
		try {
			System.out.println("可靠消息服务消费消息："+ content);
			//MessageDto dto = JsonUtils.toBean(MessageDto.class, text.getText());

			ObjectMapper mapper = new ObjectMapper();

			MessageDto dto=mapper.readValue(content, MessageDto.class);

			//UpdateHouseNameDto houseInfo =mapper.readValue(dto.getMessage(), UpdateHouseNameDto.class);  ;//JsonUtils.toBean(UpdateHouseNameDto.class, dto.getMessage());

			//数据库处理逻辑 省略

			//mapper.readValue(file, MessageDto.class);

			// 执行修改操作 ....
			//System.out.println(2/0);
			// service.update(houseInfo)  处理数据库
			//修改成功后调用消息确认消费接口，确认该消息已被消费
			System.out.println(dto.getMessageId());
			boolean result = transactionMqRemoteClient.confirmCustomerMessage("substitution-service", dto.getMessageId());// 数据库消息处理
			//手动确认ACK
			System.out.println(result);
			if (result) {//这个是控制  后端rabbitmq的接收确认标记 哈
				//text.acknowledge();
				System.out.println("是否到达这里。。。。。。。。。。。。。。");
				channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
			}
		} catch (Exception e) {
			// 异常时会触发重试机制，重试次数完成之后还是错误，消息会进入DLQ队列中
		    e.printStackTrace();
			channel.basicNack(message.getMessageProperties().getDeliveryTag(),
					false, true);
			//throw new RuntimeException(e);
		}
		
	}
}


# rabbitmq配置
spring.rabbitmq.addresses=amqp://localhost:5672
spring.rabbitmq.username=yinjihuan
spring.rabbitmq.password=123456

spring.rabbitmq.listener.direct.acknowledge-mode=manual
spring.rabbitmq.listener.simple.acknowledge-mode=manual
spring.rabbitmq.listener.simple.retry.enabled=true

#还有种spring cloud 总线流方式

# exchange name
spring.cloud.stream.bindings.addHouseInput.destination=houseExchange
# 分组，确保多个实例时消息只消费一次
spring.cloud.stream.bindings.addHouseInput.group=houseGroup

----------------------------
#Feign 做成单独可共享方式的项目jar

@FeignClient(value = "transaction-mq-service", path = "/message", fallback = TransactionMqRemoteClientHystrix.class)
public interface TransactionMqRemoteClient {
	
	/**
	 * 发送消息，只存储到消息表中，发送逻辑有具体的发送线程执行
	 * @param message  消息内容
	 * @return true 成功 | false 失败
	 */
	@PostMapping("/send")
	public boolean sendMessage(@RequestBody TransactionMessage message);

----------------------------

#elaticjob分布式调度系统 当当开源的，是按节点分片或分配资源、定义分片规则 ，注册中心zookeeper 

注意另外有个单独的监控中心，能定义修改任务，能查看任务执行情况

<dependency>
			<groupId>com.dangdang</groupId>
			<artifactId>elastic-job-lite-core</artifactId>
			<version>2.1.5</version>
		</dependency>

		<dependency>
			<groupId>com.dangdang</groupId>
			<artifactId>elastic-job-lite-spring</artifactId>
			<version>2.1.5</version>
		</dependency>
		
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
		</dependency>
		
		<dependency>
			<groupId>commons-dbcp</groupId>
			<artifactId>commons-dbcp</artifactId>
			<version>1.4</version>
		</dependency>


public class MySimpleJob implements SimpleJob {

	public void execute(ShardingContext context) {
		String shardParamter = context.getShardingParameter();
		System.out.println("分片参数："+shardParamter);
		int value = Integer.parseInt(shardParamter);
		for (int i = 0; i < 1000000; i++) {
			if (i % 2 == value) {
				String time = new SimpleDateFormat("HH:mm:ss").format(new Date());
				System.out.println(Thread.currentThread().getName()+ "  " + time + ":开始执行简单任务" + i);
			}
		}
	}

}


public class MyDataflowJob implements DataflowJob<String> {

	public List<String> fetchData(ShardingContext context) {
		return Arrays.asList("1", "2", "3");
	}

	public void processData(ShardingContext context, List<String> data) {
		System.out.println("处理数据：" + data.toString());
	}

}

------------------------------------
#sharding-jdbc

@ImportResource(locations = { "classpath:sharding.xml" })

解压缩文件
tar -xvJf  mysql-8.0.21-linux-glibc2.17-x86_64-minimal.tar.xz

#主从库建立过程：mysql组及用户 安装时都是存在的哈
1)创建目录
 mkdir /data/mysql/data
 mkdir /data/mysql/slavedata

2)授权

chown -R mysql:mysql /data/mysql/data
chown -R mysql:mysql /data/mysql/slavedata

3)建立数据库
mysqld --initialize-insecure --lower-case-table-names=1 --user=mysql --datadir=/data/mysql/data
mysqld --initialize-insecure --lower-case-table-names=1 --user=mysql --datadir=/data/mysql/slavedata

4)文件授权
chown -R mysql.mysql /data

5)汇报工作  
mysqld_multi --defaults-extra-file=/labs/my.cnf report

6)开始工作启动
mysqld_multi --defaults-extra-file=/labs/my.cnf start 1-2   mysqld_multi --defaults-extra-file=/usr/local/mysql/etc/my.cnf start 1,2

mysqld_multi --defaults-extra-file=/labs/my.cnf start 1

mysqld_multi --defaults-extra-file=/labs/my.cnf report  1

7)停机

mysqld_multi --defaults-extra-file=/labs/my.cnf stop 1-2

8)注意事项 配置文件中加入 --lower-case-table-names=1  初始化数据库加入 --lower-case-table-names=1

9)连接数据库注意事项
mysql --socket=/tmp/mysql1.sock   -uroot -p

use mysql;
 
select host from user where user='root';
update user set host = '%' where user ='root';
flush privileges;

grant all privileges on *.* to 'root'@'%' ;

8.0加密方式是（caching_sha2_password）有些 mysql workbench 客户端还不支持，因此需要修改 mysql 用户密码的加密方式
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '';

10)主从数据库配置
CREATE USER 'repl'@'%' IDENTIFIED WITH mysql_native_password BY 'yxhyhy';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
flush privileges;
SHOW MASTER STATUS;

从库上执行
CHANGE MASTER TO
MASTER_HOST='192.168.220.153',
MASTER_USER='repl',
MASTER_PASSWORD='yxhyhy',
MASTER_LOG_FILE='mysql-bin.000002',
MASTER_LOG_POS=1934;

show slave status ; 从库状态查看
注意观察是否一样 主从若一样 则删除从库data目录下auto.cnf 再重启动
show variables like '%uuid%';


#可查看当当配置策略  数据源 及jdbc管理源以及分片策略

<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:rdb="http://www.dangdang.com/schema/ddframe/rdb" 
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
                        http://www.springframework.org/schema/beans/spring-beans.xsd
                        http://www.springframework.org/schema/context 
                        http://www.springframework.org/schema/context/spring-context.xsd 
                        http://www.dangdang.com/schema/ddframe/rdb 
                        http://www.dangdang.com/schema/ddframe/rdb/rdb.xsd 
                        ">
   <!-- inline表达式报错 -->
   <context:property-placeholder  ignore-unresolvable="true"/> 
                       
    <!-- 主数据 -->
    <bean id="ds_2" class="com.alibaba.druid.pool.DruidDataSource" destroy-method="close" primary="true">
        <property name="driverClassName" value="com.mysql.jdbc.Driver" />
        <property name="url" value="jdbc:mysql://192.168.220.153:3306/ds_2?characterEncoding=utf-8" />
        <property name="username" value="root" />
        <property name="password" value="" />
    </bean>
    

    <!-- algorithm-class="com.fangjia.sharding.UserSingleKeyTableShardingAlgorithm" -->
     <!-- user_0,user_1,user_2,user_3 -->
    <rdb:strategy id="userTableStrategy" sharding-columns="id" algorithm-expression="user_${id.longValue() % 4}"/>
    <rdb:data-source id="dataSource">
        <rdb:sharding-rule data-sources="ds_2">
            <rdb:table-rules>
                <rdb:table-rule logic-table="user" actual-tables="user_${0..3}" table-strategy="userTableStrategy"/>
            </rdb:table-rules>
            <rdb:default-database-strategy sharding-columns="none" algorithm-class="com.dangdang.ddframe.rdb.sharding.api.strategy.database.NoneDatabaseShardingAlgorithm"/>
        </rdb:sharding-rule>
    </rdb:data-source>
    
    
    <!-- 增强版JdbcTemplate -->
    <bean id="cxytiandiJdbcTemplate" class="com.cxytiandi.jdbc.CxytiandiJdbcTemplate">
    	<property name="dataSource" ref="dataSource"/>
    	<constructor-arg>
    		<value>com.fangjia.sharding.po</value>
    	</constructor-arg>
    </bean>
    
</beans>