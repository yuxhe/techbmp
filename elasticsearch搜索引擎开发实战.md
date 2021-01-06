# #！/bin/sh  脚本以这个开头
Shell脚本中用“#”表示注释，相当于C语言的“//”注释。但如果“#”位于第一行开头并且是“#！”（称为Shebang）则例外，它表示该脚本使用后面指定的解释器/bin/sh解释执行。每个脚本程序必须在开头包含Shebang语句。



bash -n  /usr/local/mysql/support-files/mysqld_multi.server

如果Shell脚本中有语法错误，则会提示错误所在行；否则不输出任何信息。

#判断某个命令是否存在
if which programname >/dev/null; then
    echo exists
else
    echo does not exist
fi


#case语法
extension="png"
case "$extension" in
  "jpg"|"jpeg")
    echo "It's image with jpeg extension."
  ;;
  "png")
    echo "It's image with png extension."
  ;;
  "gif")
    echo "Oh, it's a giphy!"
  ;;
  *)
    echo "Woops! It's not image!"
  ;;
esac


#gradle也需要安装客户端
# unzip -d /opt/gradle gradle-3.5-bin.zip

Lucene是由一个Java语言开发的开源全文检索引擎工具包。把Lucene用Netty封装成服务，使用JSON访问就是Elasticsearch。

默认情况下Elasticsearch的RESTful服务只有本机才能访问，也就是说无法从主机访问虚拟机中的服务。为了方便调试，可以修改config/elasticsearch.yml文件，加入以下两行命令：
http.host: 0.0.0.0
transport.host: 127.0.0.1
echo >> ./elasticsearch.yml http.host: 0.0.0.0
echo >> ./elasticsearch.yml transport.host: 127.0.0.1

# adduser ops 创建用户
# passwd ops  指定密码

解压缩
$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-
5.6.2.tar.gz
$ tar -xvf elasticsearch-5.6.2.tar.gz

# curl "-XPOST" "http://localhost:9200/_search" -d'
  {
     "query": {
       "match_all": {}
     }
  }'

为了避免出现跨域问题，在文件elasticsearch.yml中添加如下配置：
http.cors.enabled: true
http.cors.allow-origin: "*"


查看linux 运行方式  runlevel

·0：系统停机模式，系统默认运行级别不能设置为0，否则不能正常启动，机器关闭。
·1：单用户模式，root权限，用于系统维护，禁止远程登录，类似Windows下的安全模式登录。
·2：多用户模式，没有NFS网络支持。
·3：完整的多用户文本模式，有NFS，登录后进入控制台命令行模式。
·4：系统未使用，保留，一般不用。
·5：图形化模式，登录后进入图形GUI模式，X Window系统。
·6：重启模式，默认运行级别不能设为6，否则不能正常启动。运行init 6机器就会重启。

将Elasticsearch服务开启并设置启动级别：
# chkconfig --level 3 es on

#向死而生，并非向阳而生
古来征战几人回?

报君黄金台上意，提携玉龙为君死。

琵琶、胡琴、羌笛

所以古羌人与羌族不是同一个概念

elasticsearch 中的json有处理包

XContentBuilder b = XContentFactory.jsonBuilder().startObject();
		b.field("title", "新闻标题");
		b.field("body", "内容");
		b.endObject();

		// from XContent to JSON
		String json = b.bytes().utf8ToString();


_source 宇段存储的是索引的原始内容

从每一个存储的列中获取值时都需要一次磁盘 ，如果想获取多个列的值，就需要多次磁盘 1/0 。但是
如果要从 source 获取多个列的值，则只需要一次磁盘 ，因为 ource 只是 字段
所以在大多数情况下，从 ource 获取多个列的值是快速而高效的。

使用动态 Mapping

HTTP 协议的 Put 方法用来更新数据。 Put 方法和 Get 方法一样，都是幂等性

Post方法用于创建资源，每次请求都会产生新的资源，因此不具备幕等性。

#Mapping是对索引中索引的宇段名称及其数据类型进行定义，类似数据库中的表的结构定义，但无主键的说法。

#往lucene放的是文档中拆分的词，返回的是文档 。

#用CRC32算法生成,检验文件的完整性

#volatile变量，会总是从内存中读取它，并把它的值立即放入内存中

#对必须以特定顺序执行的操作设置内存屏障

可以通过如下命令查看线程地情况
#curl http : //l ocalhost:9200/_nodes/stats?pretty 
例如，返回的搜索线程池相关信息如下
” search" : { 
” threads ”: 2 , 
” queue " : 0 , 
” active ”: 0 , 
" rejected”: 0 , 
” l argest”: 2 , 
” completed”: 2

最需要关注 reje 。当某 线程 active 的值 threads 时，示所有

线程都在忙，那么后续新的请求就会进入队列中，一旦队列大小超出限制

NettyReactor 设计模式的 个实现。服务处理器多路复用输入

Elasticsearch 支持以下 类型的缓存 节点查询缓存、分片请求缓存和宇段数据缓存。

#有1个主节点，但是为了避免单点失败，需要有多个候选主节点。

Elasticsearch集群默认使用 Zen Discovery （Zen 发现机 ）管理。


例如， 主节点 配置如下
node .master :true 
node . data :false 
discovery . zen .minimum master nodes : 2 
数据节点的配置如下
node.master :false 
node . data : true 
discovery .z en .minimum ma ster nodes:2


Guice 为google的依赖注入，且已开源

可以 使用 Luke （网址 s: // github.com/DmitryKey/luke ）分析硬盘上
Elasticsearch索引，使用luke.bat luke.sh 运行 Luke 然后就可以在/indexname/O/index
这样的路径中打开索引

Ant 可以自动化打包逻辑， Maven 也可以 自动化打包 相比于 nt. Maven 多做的事
是帮你下载 jar 包，而 radle 既能自动下 jar 包， 又能自己写脚本

静态页面或者 JS css 图片之类的静态资源可以放入 resources 录的 public

<groupid>com.lietu . example</groupid> 
<artifactid>boot- war</artifactid> 
<version>l SNAPSHOT </vers ion>
#<packaging>war</packaging>  ,pom,jar,war

#热启动 spring-boot-devtools


REST 述性状态传递（ Representational State Transfer ），是 2000 Roy Fielding 
博士在他的博士论文中提出的 种软件架构风格 ST 是一种针对网络应用的设计和开

先生成 package.json 文件：
# npm init - f

然后生成一个 四”Pr ct 的项目。
# vue init webpack Vue-project

因为 Axios 使用了 Promise 对象，所以可以将它与异步／等待结合起来，得到一个简洁
且易于使用的 API

Vue js Paginator 个简单但功能强大的插件

String input E6 B5 7% E6 8A A5 E7 BD 91 ”；
String codingName=getE odi ng （工 nput) 判断编码
System . out . print ln(URLDe coder.decode(input , codingName)) ; II 用正确的编码 解码

判断字符编码

#tar -xjf price.tar.bz2

#tar -zxvf mongodb-linux-x86_64-3.4.4.tgz

org.jsoup 为分析html内容包的组织   jsoup包




佛：这个字创造挺有意思，左边人为人，右边为弗(不的意思)，希望人不要有啥，不要有欲望、放弃欲望，讲究出世。
道：追求物我合一、天人合一、顺其自然，讲究自然。
儒：左边为人，右边为需，就是希望人们留到啥、学到啥，比如：修身、齐家、治国平天下，讲究入世。




