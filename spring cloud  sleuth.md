Google开源的 Dapper链路追踪组件，并在2010年发表了论文《Dapper, a Large-Scale Distributed Systems Tracing Infrastructure》，这篇文章是业内实现链路追踪的标杆和理论基础，具有非常大的参考价值。

目前，链路追踪组件有Google的Dapper，Twitter 的Zipkin，以及阿里的Eagleeye （鹰眼）等，它们都是非常优秀的链路追踪开源组件。



服务端要开启@eurekaclent


服务端

	<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-sleuth-zipkin-stream</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-stream-rabbit</artifactId>
		</dependency>


在程序的启动类ZipkinServerApplication上@EnableZipkinStreamServer注解，开启ZipkinStreamServer。代码如下：

zipkin支持 rabitmq、es、mysql

客户端
<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-sleuth-zipkin-stream</artifactId>

		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-stream-rabbit</artifactId>
		</dependency>

http://api/account/withAccounts/2

consul go语言写的，感觉不错



@EnableZuulProxy  网关代理

@Bean
	AddResponseIDHeaderFilter filter() {
		return new AddResponseIDHeaderFilter(); //响应头处理
	}
	
	@Bean
	AccountFallbackProvider fallback() {
		return new AccountFallbackProvider();  //处理某服务回调处理哈
	}

------------------

所以feign其实是通过http请求找到对应的服务而不是网关

@EnableFeignClients 是开启

@FeignClient(name = "account-service")
public interface AccountClient { //代理另一服务的访问方式

	@GetMapping("/customer/{customerId}")
	List<Account> findByCustomer(@PathVariable("customerId") Long customerId);
	
}


此种写法可以呀 list 演化出来的...表达式

public Customer findById(Long id) {
		Optional<Customer> customer = customers.stream().filter(p -> p.getId().equals(id)).findFirst();
		if (customer.isPresent())
			return customer.get();
		else
			return null;
	}


logstash 日志可以试一把

CommonsRequestLoggingFilter 可以优雅的记录日志信息  debug级别的

@Bean
	public CommonsRequestLoggingFilter requestLoggingFilter() {
	    CommonsRequestLoggingFilter loggingFilter = new CommonsRequestLoggingFilter();
	    loggingFilter.setIncludePayload(true);
	    loggingFilter.setIncludeHeaders(true);
	    loggingFilter.setMaxPayloadLength(1000);
	    loggingFilter.setAfterMessagePrefix("REQ:");
	    return loggingFilter;
	}


logging:
  level:
    org.springframework.web.filter.CommonsRequestLoggingFilter: DEBUG
    

-----------------------------------------------
docker exec -it fb7a78201d31 /bin/bash

进入容器开启
rabbitmq-plugins enable rabbitmq_management


# cat /etc/rabbitmq/conf.d/management_agent.disable_metrics_collector.conf 
management_agent.disable_metrics_collector = true


import org.springframework.cloud.stream.annotation.Output;
import org.springframework.messaging.MessageChannel;

public interface Source {
    String OUTPUT = "output";

    @Output("output")
    MessageChannel output();
}



import com.fasterxml.jackson.databind.ObjectMapper;

keytool -list -rfc -keystore server.jks -storepass 87654321

keytool -list -rfc -keystore order.jks -storepass 123456

keytool -list -rfc -keystore discovery.jks -storepass 123456

trust-store   key-store

----------------------
ip策略 

netsh interface ipv6 show prefixpolicies

netsh int ipv6 set prefix ::/96 50 0
netsh int ipv6 set prefix ::ffff:0:0/96 40 1
netsh int ipv6 set prefix 2002::/16 35 2
netsh int ipv6 set prefix 2001::/32 30 3
netsh int ipv6 set prefix ::1/128 10 4
netsh int ipv6 set prefix ::/0 5 5
netsh int ipv6 set prefix fc00::/7 3 13
netsh int ipv6 set prefix fec0::/10 1 11
netsh int ipv6 set prefix 3ffe::/16 1 12

-------------------------------------------------
删除证书

keytool -delete -alias discovery -keystore "C:\Program Files\Java\jdk1.8.0_271\jre\lib\security\cacerts"  -storepass changeit

生成证书
keytool -export -alias discovery -keystore discovery.p12 -rfc -file discovery.cer


验证是否已创建过同名的证书
keytool -list -v -alias tomcat -keystore "%JAVA_HOME%/JRE/LIB/SECURITY/CACERTS" -storepass changeit
删除已创建的证书
keytool -delete -alias tomcat -keystore "%JAVA_HOME%/JRE/LIB/SECURITY/CACERTS" -storepass changeit


keytool -list -rfc -alias customer -keystore "C:\Program Files\Java\jdk1.8.0_271\jre\lib\security\cacerts" -storepass  changeit
	 
1）保持密钥对
2）证书导入 cacerts内
3）路径地址是个坑，尤其window 与 linux 上

System.setProperty("javax.net.ssl.keyStore", "H:\\java代码\\Mastering-Spring-Cloud\\Chapter12\\sample-spring-cloud-security-oauth2\\customer-service\\src\\main\\resources\\customer.p12");
		System.setProperty("javax.net.ssl.keyStorePassword", "123456");

		System.setProperty("javax.net.ssl.trustStore", "C:\\Program Files\\Java\\jdk1.8.0_271\\jre\\lib\\security\\cacerts");
		System.setProperty("javax.net.ssl.trustStorePassword", "changeit");

---------------------------------------------------------------------------

SSL和TLS都是加密协议，有网络请求的地方就可以使用这两种协议在传输层进行加密，确保数据传输的安全，SSL是TLS的前身，网景在1995年发布了直接发布了SSL 2.0版本，1.0版本没有对外发布。由于漏洞的原因，版本2.0也只是昙花一现，网景在1996年就发布了SSL3.0。随后在1999年的时候，基于SSL3.0版本，网景发布了TLS1.0版本(虽然TLS1.0在SSL3.0基础上的改动不太大，但是这些改动都是非常重要的)。

协议	年份
SSL 2.0	1995
SSL 3.0	1996
TLS 1.0	1999
TLS 1.1	2006
TLS 1.2	2008
TLS 1.3	2018


一、作用

不使用SSL/TLS的HTTP通信，就是不加密的通信。所有信息明文传播，带来了三大风险。

（1） 窃听风险（eavesdropping）：第三方可以获知通信内容。

（2） 篡改风险（tampering）：第三方可以修改通信内容。

（3） 冒充风险（pretending）：第三方可以冒充他人身份参与通信。

SSL/TLS协议是为了解决这三大风险而设计的，希望达到：

（1） 所有信息都是加密传播，第三方无法窃听。

（2） 具有校验机制，一旦被篡改，通信双方会立刻发现。

（3） 配备身份证书，防止身份被冒充。


（1）如何保证公钥不被篡改？

解决方法：将公钥放在数字证书中。只要证书是可信的，公钥就是可信的。

（2）公钥加密计算量太大，如何减少耗用的时间？

解决方法：每一次对话（session），客户端和服务器端都生成一个"对话密钥"（session key），用它来加密信息。由于"对话密钥"是对称加密，所以运算速度非常快，而服务器公钥只用于加密"对话密钥"本身，这样就减少了加密运算的消耗时间。


因此，SSL/TLS协议的基本过程是这样的：

（1） 客户端向服务器端索要并验证公钥。

（2） 双方协商生成"对话密钥"。

（3） 双方采用"对话密钥"进行加密通信。

首先，客户端（通常是浏览器）先向服务器发出加密通信的请求，这被叫做ClientHello请求。

在这一步，客户端主要向服务器提供以下信息。

（1） 支持的协议版本，比如TLS 1.0版。

（2） 一个客户端生成的随机数，稍后用于生成"对话密钥"。

（3） 支持的加密方法，比如RSA公钥加密。

（4） 支持的压缩方法。

服务器收到客户端请求后，向客户端发出回应，这叫做SeverHello。服务器的回应包含以下内容。

（1） 确认使用的加密通信协议版本，比如TLS 1.0版本。如果浏览器与服务器支持的版本不一致，服务器关闭加密通信。

（2） 一个服务器生成的随机数，稍后用于生成"对话密钥"。

（3） 确认使用的加密方法，比如RSA公钥加密。

（4） 服务器证书。


4.3 客户端回应

客户端收到服务器回应以后，首先验证服务器证书。如果证书不是可信机构颁布、或者证书中的域名与实际域名不一致、或者证书已经过期，就会向访问者显示一个警告，由其选择是否还要继续通信。

如果证书没有问题，客户端就会从证书中取出服务器的公钥。然后，向服务器发送下面三项信息。

（1） 一个随机数。该随机数用服务器公钥加密，防止被窃听。

（2） 编码改变通知，表示随后的信息都将用双方商定的加密方法和密钥发送。

（3） 客户端握手结束通知，表示客户端的握手阶段已经结束。这一项同时也是前面发送的所有内容的hash值，用来供服务器校验。


4.4 服务器的最后回应

服务器收到客户端的第三个随机数pre-master key之后，计算生成本次会话所用的"会话密钥"。然后，向客户端最后发送下面信息。

（1）编码改变通知，表示随后的信息都将用双方商定的加密方法和密钥发送。

（2）服务器握手结束通知，表示服务器的握手阶段已经结束。这一项同时也是前面发送的所有内容的hash值，用来供客户端校验。

至此，整个握手阶段全部结束。接下来，客户端与服务器进入加密通信，就完全是使用普通的HTTP协议，只不过用"会话密钥"加密内容。


springfox基于swagger2，兼容老版本

以前的版本是swagger + springmvc，现在springfox中整合了swagger-ui



