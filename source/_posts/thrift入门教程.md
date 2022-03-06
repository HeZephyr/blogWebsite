---
title: thrift教程
date: 2022-2-5
top: true
tags: [thrift,RPC框架]
categories: 开发
---
# 1 thrift简介
## 1.1 RPC
RPC(Remote Procedure Call， 远程过程调用)，它是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。通俗讲，有两台服务器A，B，一个应用部署在A服务器上，想要调用B服务器上应用提供的函数/方法，由于不在一个内存空间，不能直接调用，需要通过网络来表达调用的语义和传达调用的数据。
RPC协议假定某些传输协议的存在，如TCP或UDP，为通信程序之间携带信息数据。RPC实现了服务消费调用方Client与服务提供实现方Server之间的点对点调用流程，即包括了stub、通信、数据的序列化/反序列化。且Client与Server一般采用直连的调用方式。
在OSI网络通信模型中，RPC跨越了传输层和应用层。RPC使得开发包括网络分布式多程序在内的应用程序更加容易。
其工作原理如下：
![](https://img-blog.csdnimg.cn/fca6247dc6c54411aa383be58c422b8b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAdW5pcXVlX3B1cnN1aXQ=,size_13,color_FFFFFF,t_70,g_se,x_16)
1)调用客户端以本地方式调用远程服务
2)client stub将请求（方法和参数）组装成网络消息
3)client stub找得到服务器地址，将消息传送到远程主机
4)server stub得到传送过来的请求，进行解码
5)server stub 调用本地服务，处理请求
6)本地服务处理请求，并将处理结果返回给server stub
7)server stub将请求处理结果组装成网络消息
8)server stub找到客户端地址，将请求处理结果传送给客户端
9)client stub 接收到请求处理结果，进行解码
10)客户端最终接收到请求处理结果

而RPC框架的目的就是将2-9步骤封装起来，对使用者透明，客户端只需要执行第一步调用接口，然后就能够得到结果。这样是不是很方便，而且省去了很多麻烦。
常见的RPC技术和框架有：
* 应用级的服务框架：阿里的Dubbo/Dubbox、Google gPRC、Spring Boot/Spring Cloud。
* 远程通信协议：RMI、Socket、SOAP(HTTP XML)、REST(HTTP JSON)。
* 通信框架：MINA和Netty。

目前流行的开源 RPC 框架还是比较多的，有阿里巴巴的 Dubbo、Facebook 的 Thrift、Google 的 gRPC、Twitter 的 Finagle 等。

## 1.2 thrift
Apache Thrift 是由 Facebook 开发的一种RPC框架，是用来进行可伸缩的、跨语言的服务开发。它最大的特点就是支持多语言，在各个服务之间的RPC通信领域应用非常广泛。
Thrift通过一个中间语言（IDL，接口定义语言）来定义RPC的接口和数据类型，然后通过一个编译器生成不同语言的代码（目前支持的语言有C++, Java, Python, PHP, Ruby, Erlang, Perl, Haskell, C#, Cocoa, JavaScript, Node.js, Smalltalk, OCaml and Delphi 等），并由生成的代码负责RPC协议层和传输层的实现。
Thrift协议栈：
![](https://img-blog.csdnimg.cn/7a25799083fd4a6bb7b3ef66e8523127.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAdW5pcXVlX3B1cnN1aXQ=,size_10,color_FFFFFF,t_70,g_se,x_16)
* **第一部分（Your Code）**
为开发者的业务逻辑代码。
* **第二部分（ServiceClient）**
Thrift自动生成的代码，包含Processor、TServer、ServiceClient。
TServer负责接收Client的请求，并将请求转发到Processor进行处理。TServer主要任务就是高效的接受Client的请求，特别是在高并发请求的情况下快速完成请求。Processor(或者TProcessor)负责对Client的请求做出相应，包括RPC请求转发，调用参数解析和用户逻辑调用，返回值写回等处理步骤。Processor是服务器端从Thrift框架转入用户逻辑的关键流程。Processor同时也负责向Message结构中写入数据或者读出数据。
ServiceClient就是客户端，包含可以调用的请求方法和发送客户端请求
* **第三部分（TProtocol）**
TProtocol主要负责结构化数据组装成Message，或者从Message结构中读取结构化数据。TProtocol将一个有类型的数据转换为字节流以交给TTransport进行传输，或者从TTransport中读取一定长度的字节数据转化为特定类型的数据。如int32会被TBinaryProtocol Encode为一个四字节的字节数据，或者TBinaryProtocol从TTransport中取出四个字节的数据Decode为int32。
* **第四部分（TTransport）**
TTransport负责以字节流方式发送和接收Message，是底层IO模块在Thrift框架中的实现，每一个底层IO模块都会有一个对应TTransport来负责Thrift的字节流(Byte Stream)数据在该IO模块上的传输。例如TSocket对应Socket传输，TFileTransport对应文件传输。

* **第五部分（Underlying I/O）**
底层IO模块，负责实际的数据传输，包括Socket，文件，或者压缩数据流等。

通过这个协议栈，我们使用Thrift只需要做三件事
1.通过IDL定义数据结构和服务
即编写thrift文件。Thrift 采用IDL（Interface Definition Language）来定义通用的服务接口，然后通过Thrift提供的编译器，可以将服务接口编译成不同语言编写的代码，通过这个方式来实现跨语言的功能。
2.利用代码生成工具生成代码
利用Thrift编译器将thrift文件生成对应语言的源代码，使用命令为：
`thrift -r --gen <language> <Thrift filename>`
3.编写你的业务逻辑

在实际编程中，我们可以参照[thrift官网](https://thrift.apache.org/tutorial/)提供的教程来进行编写，其实现了一个计算器服务功能，容易实现。


# 2 thrift安装配置（ubuntu）
## 2.1 boost安装
thrift依赖boost库，所以我们需要先安装boost库。安装教程参照：[ubuntu安装boost](https://blog.csdn.net/faihung/article/details/88128928)。
## 2.2 thrift安装配置
先下载thriftt压缩包，[下载地址](https://thrift.apache.org/download)，然后直接解压即可。
关键在于环境配置。

# 3 thrift的IDL语法
IDL语法都会与对应的编程语言相对应生成，所以我们实现可以参考生成关系，然后根据需求使用我们的IDL语法。
## 3.1 基本类型
>bool: 布尔值 对应Java中的boolean
byte: 有符号字节 对应Java中的byte
i16: 16位有符号整型 对应Java中的short
i32: 32位有符号整型 对应Java中的int
i64: 64位有符号整型 对应Java中的long
double: 64位浮点型 对应Java中的double
string: 字符串 对应Java中的String
binary: Blob 类型 对应Java中的byte[]
## 3.2 struct结构体
类似C语言中的struct，只不过有以下规定：
* struct不能继承，但是可以嵌套，不能嵌套自己。
* 其成员都是有明确类型
* 成员必须是被正整数编号过的，其中的编号使不能重复的，这个是为了在传输过程中编码使用。
* 成员分割符可以是逗号（,）或是分号（;），而且可以混用
* 字段会有optional和required之分，但是如果不指定则为无类型–可以不填充该值，但是在序列化传输的时候也会序列化进去，optional是不填充则不序列化，required是必须填充也必须序列化。
* 每个字段可以设置默认值
* 同一文件可以定义多个struct，也可以定义在不同的文件，进行include引入。

定义User结构体如下：

```c
struct User {
	1: required string name, // 表明该字段必须填写
	2: optional i32 age = 0; // 设置默认值为0
	3: i32 id // 默认类型为optional
}
```
## 3.3 Container(容器)
有3种可用容器类型：
* `list<type>`：元素类型为type的有序表，允许元素重复。对应C++的vector，java的ArrayList。
* `set<type>`：元素类型为type的无序表，不允许元素重复。对应C++的set，java的HashSet。
* `map<type,type>`：键类型为t，值类型为t的kv对，键不容许重复。对用c++中的map, Java的HashMap。

例子：

```c
struct Test {
	1: map<string, User> userMap,
	2: list<i32> intList,
	3: set<double> doubleSet
}
```
## 3.4 enum(枚举)
类似C++中的enum。有如下规则遵守：
* 编译器默认从0开始赋值
* 可以赋予某个常量某个整数
* 允许常量是十六进制整数
* 末尾没有分号
* 给常量赋缺省值时，使用常量的全称
* Thrift不支持枚举类嵌套，枚举常量必须是32位的正整数

例子：

```c
enum Test{
	OK = 200,
	FALSE = 300
}
```
## 3.5 定义常量
和C++一样，在变量的前面加上const即可。
## 3.6 typedef 
Thrift支持C/C++类型定义，例子如下：

```c
typedef i32 int
typedef i64 long
```
## 3.7 Service(服务定义类型)
服务的定义方法在语义上等同于面向对象语言的接口。

```cpp
service HelloService{
	i32 calInt(1: i32 param1, 2: i32 param2),
	bool checkString(1: string param)
}
```
## 3.8 namespace(命名空间)
Thrift中的命名空间类似于C++中的namespace和java中的package，它们提供了一种组织（隔离）代码的简便方式。名字空间也可以用于解决类型定义中的名字冲突。
由于每种语言均有自己的命名空间定义方式（如python中有module）, thrift允许开发者针对特定语言定义namespace：`namespace <language> <名称>`

## 3.9 include
便于管理、重用和提高模块性/组织性，我们常常分割Thrift定义在不同的文件中。包含文件搜索方式与c++一样。Thrift允许文件包含其它thrift文件，用户需要使用thrift文件名作为前缀访问被包含的对象。
thrift文件名要用双引号包含，末尾没有逗号或者分号。
`include "test.thrift"`

# 4 thrift实战—双人游戏匹配系统
## 4.1 前期准备
根据实际需要作业务逻辑图如下：
![](https://img-blog.csdnimg.cn/3afddf786a39482fa53b678319aee6f3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAdW5pcXVlX3B1cnN1aXQ=,size_20,color_FFFFFF,t_70,g_se,x_16)
## 4.2 教程
[教程](https://www.acwing.com/blog/content/7852/)
## 4.3 源码地址
[源码地址](https://github.com/unique-pure/thriftStudy-gameMatch/blob/master/thrift/save.thrift)
