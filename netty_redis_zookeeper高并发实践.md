getResource(name)   //注意中文及特殊符号问题


URL url = IOUtil.class.getResource(resName);

file:/H:/java%e4%bb%a3%e7%a0%81/netty_redis_zookeeper_source_code/netty+redis+zookeeper%e5%ae%9e%e6%88%98%e7%ac%ac2-8%e7%ab%a0%e2%80%94%e2%80%94%e7%ac%ac10-11%e7%ab%a0%e6%ba%90%e7%a0%81/NioDemos/target/classes/system.properties

path = IOUtil.class.getResource(resName).toURI().getPath();

/H:/java代码/netty_redis_zookeeper_source_code/netty+redis+zookeeper实战第2-8章——第10-11章源码/NioDemos/target/classes/system.properties

注意编码 内容 ，还有作为 url访问则需要编码

URLEncoder.encode(path, "UTF-8"); //编码发到服务端，服务端会进行decode
decodePath = URLDecoder.decode(path, "UTF-8");


编码对应：
空格	!	#	$	%	+	@	:	=	?
%20	%21	%23	%24	%25	%2B	%40	%3A	%3D	%3F

base64 也不适合url ，唯一具有可阅读性


POST提交
对于POST方式，表单中的参数值对是通过request body发送给服务器，此时浏览器会根据网页的ContentType("text/html; charset=GBK")中指定的编码进行对表单中的数据进行编码，然后发给服务器


为什么要进行URL编码
我们都知道Http协议中参数的传输是"key=value"这种简直对形式的，如果要传多个参数就需要用“&”符号对键值对进行分割。如"?name1=value1&name2=value2"，这样在服务端在收到这种字符串的时候，会用“&”分割出每一个参数，然后再用“=”来分割出参数值。

 

针对“name1=value1&name2=value2”我们来说一下客户端到服务端的概念上解析过程: 
  上述字符串在计算机中用ASCII吗表示为： 
  6E616D6531 3D 76616C756531 26 6E616D6532 3D 76616C756532。 
   6E616D6531：name1 
   3D：= 
   76616C756531：value1 
   26：&
   6E616D6532：name2 
   3D：= 
   76616C756532：value2 
   服务端在接收到该数据后就可以遍历该字节流，首先一个字节一个字节的吃，当吃到3D这字节后，服务端就知道前面吃得字节表示一个key，再想后吃，如果遇到26，说明从刚才吃的3D到26子节之间的是上一个key的value，以此类推就可以解析出客户端传过来的参数。

   现在有这样一个问题，如果我的参数值中就包含=或&这种特殊字符的时候该怎么办。 
比如说“name1=value1”,其中value1的值是“va&lu=e1”字符串，那么实际在传输过程中就会变成这样“name1=va&lu=e1”。我们的本意是就只有一个键值对，但是服务端会解析成两个键值对，这样就产生了奇异。

如何解决上述问题带来的歧义呢？解决的办法就是对参数进行URL编码 
   URL编码只是简单的在特殊字符的各个字节前加上%，例如，我们对上述会产生奇异的字符进行URL编码后结果：“name1=va%26lu%3D”，这样服务端会把紧跟在“%”后的字节当成普通的字节，就是不会把它当成各个参数或键值对的分隔符。


另外一个问题，就是为什么我们要用ASCII传输，可不可以用别的编码？ 
    当然可以用别的编码，你自己可以开发一套编码，然后自己解析。就像大部分国家都有自己的语言一样。那国家之间要交流，怎么办？  用英语把，英语的使用范围最广。

 

 

通常如果一样东西需要编码，说明这样东西并不适合传输。原因多种多样，如Size过大，包含隐私数据，对于Url来说，之所以要进行编码，是因为Url中有些字符会引起歧义。

　　例如，Url参数字符串中使用key=value键值对这样的形式来传参，键值对之间以&符号分隔，如/s?q=abc&ie=utf-8。如果你的value字符串中包含了=或者&，那么势必会造成接收Url的服务器解析错误，因此必须将引起歧义的&和=符号进行转义，也就是对其进行编码。

　　又如，Url的编码格式采用的是ASCII码，而不是Unicode，这也就是说你不能在Url中包含任何非ASCII字符，例如中文。否则如果客户端浏览器和服务端浏览器支持的字符集不同的情况下，中文可能会造成问题。

Url编码的原则就是使用安全的字符（没有特殊用途或者特殊意义的可打印字符）去表示那些不安全的字符。

　　预备知识：URI是统一资源标识的意思，通常我们所说的URL只是URI的一种。典型URL的格式如下所示。下面提到的URL编码，实际上应该指的是URI编码。

foo://example.com:8042/over/there?name=ferret#nose

   \_/ \______________/ \________/\_________/ \__/

    |         |              |         |        |

  scheme     authority                path      query   fragment

　　哪些字符需要编码
　　RFC3986文档规定，Url中只允许包含英文字母（a-zA-Z）、数字（0-9）、-_.~4个特殊字符以及所有保留字符。RFC3986文档对Url的编解码问题做出了详细的建议，指出了哪些字符需要被编码才不会引起Url语义的转变，以及对为什么这些字符需要编码做出了相应的解释。

　　US-ASCII字符集中没有对应的可打印字符：Url中只允许使用可打印字符。US-ASCII码中的10-7F字节全都表示控制字符，这些字符都不能直接出现在Url中。同时，对于80-FF字节（ISO-8859-1），由于已经超出了US-ACII定义的字节范围，因此也不可以放在Url中。

　　保留字符：Url可以划分成若干个组件，协议、主机、路径等。有一些字符（:/?#[]@）是用作分隔不同组件的。例如：冒号用于分隔协议和主机，/用于分隔主机和路径，?用于分隔路径和查询参数，等等。还有一些字符（!$&'()*+,;=）用于在每个组件中起到分隔作用的，如=用于表示查询参数中的键值对，&符号用于分隔查询多个键值对。当组件中的普通数据包含这些特殊字符时，需要对其进行编码。

　　RFC3986中指定了以下字符为保留字符：! * ' ( ) ; : @ & = + $ , / ? # [ ]

　　不安全字符：还有一些字符，当他们直接放在Url中的时候，可能会引起解析程序的歧义。这些字符被视为不安全字符，原因有很多。

空格：Url在传输的过程，或者用户在排版的过程，或者文本处理程序在处理Url的过程，都有可能引入无关紧要的空格，或者将那些有意义的空格给去掉。
引号以及<>：引号和尖括号通常用于在普通文本中起到分隔Url的作用
#：通常用于表示书签或者锚点
%：百分号本身用作对不安全字符进行编码时使用的特殊字符，因此本身需要编码
{}|\^[]`~：某一些网关或者传输代理会篡改这些字符
　　需要注意的是，对于Url中的合法字符，编码和不编码是等价的，但是对于上面提到的这些字符，如果不经过编码，那么它们有可能会造成Url语义的不同。因此对于Url而言，只有普通英文字符和数字，特殊字符$-_.+!*'()还有保留字符，才能出现在未经编码的Url之中。其他字符均需要经过编码之后才能出现在Url中。

　　但是由于历史原因，目前尚存在一些不标准的编码实现。例如对于~符号，虽然RFC3986文档规定，对于波浪符号~，不需要进行Url编码，但是还是有很多老的网关或者传输代理会进行编码。

　　如何对Url中的非法字符进行编码
　　Url编码通常也被称为百分号编码（Url Encoding，also known as percent-encoding），是因为它的编码方式非常简单，使用%百分号加上两位的字符——0123456789ABCDEF——代表一个字节的十六进制形式。Url编码默认使用的字符集是US-ASCII。例如a在US-ASCII码中对应的字节是0x61，那么Url编码之后得到的就是%61，我们在地址栏上输入http://g.cn/search?q=%61%62%63，实际上就等同于在google上搜索abc了。又如@符号在ASCII字符集中对应的字节为0x40，经过Url编码之后得到的是%40。

　　对于非ASCII字符，需要使用ASCII字符集的超集进行编码得到相应的字节，然后对每个字节执行百分号编码。对于Unicode字符，RFC文档建议使用utf-8对其进行编码得到相应的字节，然后对每个字节执行百分号编码。如"中文"使用UTF-8字符集得到的字节为0xE4 0xB8 0xAD 0xE6 0x96 0x87，经过Url编码之后得到"%E4%B8%AD%E6%96%87"。

　　如果某个字节对应着ASCII字符集中的某个非保留字符，则此字节无需使用百分号表示。例如"Url编码"，使用UTF-8编码得到的字节是0x55 0x72 0x6C 0xE7 0xBC 0x96 0xE7 0xA0 0x81，由于前三个字节对应着ASCII中的非保留字符"Url"，因此这三个字节可以用非保留字符"Url"表示。最终的Url编码可以简化成"Url%E7%BC%96%E7%A0%81" ，当然，如果你用"%55%72%6C%E7%BC%96%E7%A0%81"也是可以的


------------------------
https://www.jianshu.com/p/59dd0489248f

Post请求的两种编码格式 

application/x-www-form-urlencoded和multipart/form-data

[https://www.jianshu.com/p/59dd0489248f](https://www.jianshu.com/p/59dd0489248f)


Google 的 AngularJS 中的 Ajax 功能，默认就是提交 JSON 字符串。例如下面这段代码：

var data = {'title':'test', 'sub' : [1,2,3]};
$http.post(url, data).success(function(result) {
    ...
});
最终发送的请求是：

POST http://www.example.com HTTP/1.1
Content-Type: application/json;charset=utf-8
 
{"title":"test","sub":[1,2,3]}

-------------

text/xml
我的博客之前提到过 XML-RPC（XML Remote Procedure Call）。它是一种使用 HTTP 作为传输协议，XML 作为编码方式的远程调用规范。典型的 XML-RPC 请求是这样的：

POST http://www.example.com HTTP/1.1
Content-Type: text/xml
 
<!--?xml version="1.0"?-->
<methodcall>
    <methodname>examples.getStateName</methodname>
    <params>
        <param>
            <value><i4>41</i4></value>
         
    </params>
</methodcall>

-----------------

Range / Accept-Range
按范围取数据
Accept-Range: bytes 响应报⽂中出现，表示服务器⽀持按字节来取范围数据
Range: bytes=<start>-<end> 请求报⽂中出现，表示要取哪段数据
Content-Range:<start>-<end>/total 响应报⽂中出现，表示发送的是哪段数据
作⽤：断点续传、多线程下载。
-------
[https://www.jianshu.com/p/b0aa797608c0](https://www.jianshu.com/p/b0aa797608c0)


抽象 - >get、open等返回一个创建好的对象哈，抽象类及本地方法构成哈

--------------------------------FutureTask 的方式
有返回值的线程处理哈哈

MyThread2 myThread2 = new MyThread2();
FutureTask<Integer> futureTask = new FutureTask<>(myThread2);
new Thread(futureTask, "线程名：有返回值的线程2").start();

thread  callable   
结构关系
public class FutureTask<V> implements RunnableFuture<V> {

另外 google的线程池封装可以时 异步非阻塞的
-----------------------------------------
服务接收端的处理逻辑

b.childHandler(new ChannelInitializer<SocketChannel>() {
                //有连接到达时会创建一个channel
                protected void initChannel(SocketChannel ch) throws Exception {
                    // pipeline管理子通道channel中的Handler
                    // 向子channel流水线添加3个handler处理器
                    ch.pipeline().addLast(new ProtobufVarint32FrameDecoder());
                    ch.pipeline().addLast(new ProtobufDecoder(MsgProtos.Msg.getDefaultInstance()));
                    ch.pipeline().addLast(new ProtobufBussinessDecoder());
                }
            });


 //服务器端业务处理器
    static class ProtobufBussinessDecoder extends ChannelInboundHandlerAdapter {
        @Override
        public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
            MsgProtos.Msg protoMsg = (MsgProtos.Msg) msg;
            //经过pipeline的各个decoder，到此Person类型已经可以断定
            Logger.info("收到一个 MsgProtos.Msg 数据包 =》");
            Logger.info("protoMsg.getId():=" + protoMsg.getId());
            Logger.info("protoMsg.getContent():=" + protoMsg.getContent());
        }
    }

---------------------------------------------
客户端的发送逻辑
//构建ProtoBuf对象
    public MsgProtos.Msg build(int id, String content) {
        MsgProtos.Msg.Builder builder = MsgProtos.Msg.newBuilder();
        builder.setId(id);
        builder.setContent(content);
        return builder.build();
    }

//发送 Protobuf 对象
            for (int i = 0; i < 1000; i++) {
                MsgProtos.Msg user = build(i, i + "->" + content);
                channel.writeAndFlush(user);
                Logger.info("发送报文数：" + i);
            }
            channel.flush();

//5 装配通道流水线
            b.handler(new ChannelInitializer<SocketChannel>() {
                //初始化客户端channel
                protected void initChannel(SocketChannel ch) throws Exception {
                    // 客户端channel流水线添加2个handler处理器
                    ch.pipeline().addLast(new ProtobufVarint32LengthFieldPrepender());//编码
                    ch.pipeline().addLast(new ProtobufEncoder());//编码
                }
            });

----------------------------------------------------------
logback日志 是log4j的改进的更好开源版本


路径需要从 / 出发哈 ；注意zookeeper版本不同，会有命令不同哈 **注意**

[zk: localhost:2181(CONNECTED) 13] ls -s /test
[CRUD]
cZxid = 0x5                           创建事务号
ctime = Tue Nov 03 11:30:19 CST 2020  创建时间
mZxid = 0x5                           修改事务号
mtime = Tue Nov 03 11:30:19 CST 2020  修改时间
pZxid = 0x6          子节点最后的修改事务号
cversion = 1         子节点版本号
dataVersion = 0      数据版本号
aclVersion = 0       权限版本号
ephemeralOwner = 0x0  //代表永久节点
dataLength = 0        该节点数据长度
numChildren = 1       该节点子节点数量


ls -s  /test/CRUD         ls -s -R  /test/CRUD 能递归显示其子孙
-s 显示比较细节即stat  -w显示节点的子目录或节点内数据(get -w) 或者说添加watch事件

stat 命令有点类似 get命令

事务节点只能由leader产生、从节点只能同步，保证事务的一致性

set /test  1124124214   #更新节点数据操作  再次查询可看到相关状态已变化


[zk: localhost:2181(CONNECTED) 53] getAcl /test
'world,'anyone
: cdrwa    #CREATE、READ、WRITE、DELETE、ADMIN 也就是 增、删、改、查、管理权限，这**5种权限**简写为crwda(即：每个单词的首字符缩写)


create -s -e /a 123  s表示顺序节点 -e表示临时节点

slf4j 带格式打印日志  注意 {} 号的应用
log.info("read data:{}",data);



curator java的zookeeper客户端组件 apache组织的

curator  InterProcessMutex 基于zookeeper实现的分布式锁

curator 观察者或发布订阅模式

----------------
通过别名引用时  @Resource 指定别名 =  @Qualifier("别名") + @Autowired注解

------------------------------
compile

​ 默认的scope，表示 dependency 都可以在生命周期中使用。而且，这些dependencies 会传递到依赖的项目中。适用于所有阶段，会随着项目一起发布

provided

​ 跟compile相似，但是表明了dependency 由JDK或者容器提供，例如Servlet AP和一些Java EE APIs。这个scope 只能作用在编译和测试时，同时没有传递性。

runtime

​ 表示dependency不作用在编译时，但会作用在运行和测试时，如JDBC驱动，适用运行和测试阶段。

test

​ 表示dependency作用在测试时，不作用在运行时。 只在测试时使用，用于编译和运行测试代码。不会随项目发布。

system

​ 跟provided 相似，但是在系统中要以外部JAR包的形式提供，maven不会在repository查找它


persons.stream().filter(it -> it.getId().equals(id)).findFirst().get() 
数组转流的操作模式哈，注意

private static final Logger log = LoggerFactory.getLogger(PersonController.class);

persons.stream().forEach(it -> log.info("werwe:{},{}",it.getAge(),it.getFirstName()));//格式化日志打印输出





--------------------
docker run --name  microservices  -p 27017:27017  -d mongo

mongo
use  microservices

db.createUser({user:"micro",pwd:"micro",roles:[{role:'dbAdmin',db:'microservices'}]}) 


mongo  192.168.220.1/microservices  -umicro -pmicro

----------------------

http://127.0.0.1:2222/swagger-ui.html


配置中的隔板    注意不能夹杂无关属性哈 ,只要保证相对层级不能出错，那么配置就ok了
---  

日志打印方式哈
import org.slf4j.Logger;
import org.slf4j.LoggerFactory ;
Logger logger=LoggerFactory.getLogger(this.getClass());


spring boot 有点复杂 打印出所有的配置信息,实质还是spring的配置信息输出 ：

		StreamSupport.stream(environment.getPropertySources().spliterator(), false)
				.filter(propertySource -> (propertySource instanceof EnumerablePropertySource))
				.map(propertySource -> ((EnumerablePropertySource) propertySource).getPropertyNames())
				.flatMap(Arrays::stream).distinct().collect(Collectors.toMap(Function.identity(), environment::getProperty))
				.entrySet().stream().forEach(entry -> {
			logger.info("-------- {} ===> {}", entry.getKey(), entry.getValue());

注释快捷方式哈
ctrl +/ ,ctr + shift +/ ,   /** +enter

代码格式化方式 ctrl + shift + F

alt + 向上或向下箭头是块移动

插入行  上一行Ctrl+Alt+enter下一行shift+enter

