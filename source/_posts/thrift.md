---
title: thrift 深入理解
date: 2018-12-29 16:14:47
tags: thrift
categories: 技术
comment: true
---


### 简介

RPC（Remote Procedure Call Protocol），在各大互联网公司中被广泛使用，如阿里巴巴的hsf、dubbo（开源）、Facebook的thrift（开源）、Google grpc（开源）、Twitter的finagle等。

![image-20181221150904725.png](https://github.com/leolinf/hexo-blog/blob/master/source/_posts/images/image-20181221150904725.png?raw=true)

### 基础架构

在多种不同的语言之间通信thrift可以作为二进制的高性能的通讯中间件，支持数据(对象)序列化和多种类型的RPC服务。Thrift是IDL(interface definition language)描述性语言的一个具体实现，Thrift适用于程序对程序静态的数据交换，需要先确定好他的数据结构，他是完全静态化的，当数据结构发生变化时，必须重新编辑IDL文件，代码生成，再编译载入的流程，跟其他IDL工具相比较可以视为是Thrift的弱项，Thrift适用于搭建大型数据交换及存储的通用工具，对于大型系统中的子系统间数据传输相对于JSON和xml无论在性能、传输大小上有明显的优势。

如上图所示是thrift的协议栈整体的架构，thrift是一个客户端和服务器端的架构体系（c/s），在最上层是用户自行实现的业务逻辑代码。第二层是由thrift编译器自动生成的代码，主要用于结构化数据的解析，发送和接收。TServer主要任务是高效的接受客户端请求，并将请求转发给Processor处理。Processor负责对客户端的请求做出响应，包括RPC请求转发，调用参数解析和用户逻辑调用，返回值写回等处理。从TProtocol以下部分是thirft的传输协议和底层I/O通信。TProtocol是用于数据类型解析的，将结构化数据转化为字节流给TTransport进行传输。TTransport是与底层数据传输密切相关的传输层，负责以字节流方式接收和发送消息体，不关注是什么数据类型。底层IO负责实际的数据传输，包括socket、文件和压缩数据流等。

![image-20181217171353673.png](https://github.com/leolinf/hexo-blog/blob/master/source/_posts/images/image-20181217171353673.png?raw=true)

###  数据类型

Thrift 脚本可定义的数据类型包括以下几种类型：

- 基本类型：

- - bool：布尔值，true 或 false，对应 Java 的 boolean

  - byte：8 位有符号整数，对应 Java 的 byte
  - i16：16 位有符号整数，对应 Java 的 short
  - i32：32 位有符号整数，对应 Java 的 int
  - i64：64 位有符号整数，对应 Java 的 long
  - double：64 位浮点数，对应 Java 的 double
  - string：未知编码文本或二进制字符串，对应 Java 的 String

- 枚举类型：

  - enmu:  不同于protocal buffer，thrift不支持枚举类嵌套，枚举常量必须是32位的正整数

- 结构体类型：

  - struct：定义公共的对象，类似于 C 语言中的结构体定义，在 Java 中是一个 JavaBean

- 常量：const i32 INT_CONST = 1234;

- 容器类型:

  - list：对应 Java 的 ArrayList
  - set：对应 Java 的 HashSet
  - map：对应 Java 的 HashMap

- 异常类型：

  - exception：对应 Java 的 Exception

- 服务类型：service: 对应服务的类

- 名字空间:  namespace:


Thrift之所以被称为一种高效的RPC框架，其中一个重要的原因就是它提供了高效的数据传输，在传输协议上总体划分为文本 (text) 和二进制 (binary) 传输协议，为节约带宽，提高传输效率，一般情况下使用二进制类型的传输协议为多数，有时还会使用基于文本类型的协议。
以下是Thrift的传输格式种类：

### Thrift的数据传输格式（Protocol）

**TBinaryProtocol**: 二进制格式。效率显然高于文本格式。
**TCompactProtocol**：非常高效、密集的二进制编码格式进行数据传输
**TJSONProtocol**：使用 JSON 的数据编码协议进行数据传输。
**TSimpleJSONProtocol**：提供JSON只写协议(缺少元数据信息), 适用于脚本语言解析。
**MultiplexProtocol**: 多路复用协议 
**TDebugProtocol**：使用易懂的可读文本格式，以便于调试。(当前版本：就golang和cpp可能还支持)

以上可以看到，使用**TCompactProtocol**格式效率是最高的，同等数据传输占用网络带宽是最少的。

### Thrift的数据传输方式(Transport)

**TFramedTransport**：以帧的形式发送数据，每个帧之前有一个长度，以帧为单位进行传输。非阻塞式服务中使用。同TBufferedTransport类似，也会对相关数据进行buffer，同时，它支持定长数据发送和接收。
**TBufferTransport**: 以缓冲的形式发送数据。
**TZlibTransport**：使用zlib进行压缩传输，与其他传输方式联合使用。
**THttpTransport**: 以Http的形式发送数据

### Thrift底层数据传输方法(Low-Level Transport)

**TSocket**：阻塞式socket I/O进行传输。
**TSSLSocket**: 阻塞式的带证书的 socket I/O 进行传输。
**TMemoryTransport**：将内存用于I/O.
**UnixDomainSocket**: 域socket的方式进行 I/O
**TFileTransport**:  文件（日志）传输类，允许client将文件传给server，允许server将收到的数据写到文件中。
**TPipeTransport**: 管道 I/O

### Thrift的服务模型

**TSimpleServer**：简单的单线程服务模型，常用于测试。只在一个单独的线程中以阻塞I/O的方式来提供服务。所以它只能服务一个客户端连接，其他所有客户端在被服务器端接受之前都只能等待。
**TThreadPoolServer**：它使用的是一种多线程服务模型，使用标准的阻塞式I/O。它会使用一个单独的线程来接收连接。一旦接受了一个连接，它就会被放入Queue中的一个worker线程里处理。worker线程被绑定到特定的客户端连接上，直到它关闭。一旦连接关闭，该worker线程就又回到了线程池中。这意味着，如果有1万个并发的客户端连接，你就需要运行1万个线程。所以它对系统资源的消耗不像其他类型的server一样那么“友好”。此外，如果客户端数量超过了线程池中的最大线程数，在有一个worker线程可用之前，请求将被一直阻塞在那里。如果提前知道了将要连接到服务器上的客户端数量，并且不介意运行大量线程的话，可以采用此方案。
**TNonblockingServer**：它使用了非阻塞式I/O， (Java实现使用NIO通道)的多线程服务器，TFramedTransport此服务器一起使用。使用了java.nio.channels.Selector，通过调用select()，它使得程序阻塞在多个连接上，而不是单一的一个连接上。TNonblockingServer处理这些连接的时候，要么接受它，要么从它那读数据，要么把数据写到它那里，然后再次调用select()来等待下一个准备好的可用的连接。通用这种方式，server可同时服务多个客户端，而不会出现一个客户端把其他客户端全部“饿死”的情况。缺点是所有消息是被调用select()方法的同一个线程处理的，服务端同一时间只会处理一个消息，并没有实现并行处理。
**TForkingServer**： 通过fork子进程的服务模型。
**TThreadedServer**: 通过构建线程的服务模型 同上的 TThreadPoolServer 一样的缺点，这个没有线程的大小限制，会无线增加，可能导致内存崩掉，甚至down机的危险。

注意事项

1. Thrift生成的server端是thread safe的. 但是client端不是thread safe. 所以需要多个thread和server端通信,则每个thread需要initiate一个自己的client实例.
2. 如果服务器采用TNonblockingServer的话，客户端必须采用TFramedTransport。程序链接的时候需要thriftnb。
3. 默认TServerSocket和TSocket都设置了NoDelay为1，使得报文尽快发送出去，如果客户端和服务器间传输数据量较大，通过可以设置NoDelay为0来开启Nagel算法，缓存一段数据后再进行发送，减少报文数量。
   TSocket默认开启了Linger，并设置linger time为0，这样close会丢弃socket发送缓冲区中的数据，并向对端发送一个RST报文，close不会被阻塞，立即返回。
   TServerSocket默认关闭了Linger，close不会被阻塞，立即返回。
4. fb303作为handler的基类，里面预置了一些rpc方法，用于监控，包括系统状态，请求次数等状态信息。
   thrift文件中需要include "fb303.thrift"这样来将service导入目标thrift文件中。thrift编译后的代码只需要相应的Handler多重继承facebook::fb303::FacebookBase就好了。

#### 通过 python thrift 源码 详解 thrift 调用链路

client 调用过程

socket 连接 -> 选择传输数据方式->选择传输协议->调用定义好的相应的方法

```python
from thrift import Thrift
from thrift.transport import TSocket
from thrift.transport import TTransport, TZlibTransport
from thrift.protocol import TBinaryProtocol, TCompactProtocol, TJSONProtocol
from hello import HelloService, ttypes
# 监听端口
transport = TSocket.TSocket('0.0.0.0', 9090)
transport.setTimeout(3000)
# 选择传输层
transport = TTransport.TBufferedTransport(transport)
# 选择传输协议
protocol = TCompactProtocol.TCompactProtocol(transport)  # 压缩格式
# 调用服务端handler
client = HelloService.Client(protocol)
transport.open()
# 调用server 端相应的方法
msg = client.say("Hello World!")
transport.close()
```

generaterted code 里面的调用，

send_xxx 方法 发送数据: 

怎个数据序列化的过程: writeMessageBegin->writeStructBegin->writeFieldBegin->writeFieldend->writeStructEnd->writeMessageEnd 最后调用 flush 通过 socket 发送数据到服务端

recv_xxx 方法 接受返回数据:

和 send_xxx 相反的一个过程，反序列化数据

server 启用过程

```python
from hello import HelloService
from thrift.transport import TSocket
from thrift.transport import TTransport, TZlibTransport
from thrift.protocol import (
    TBinaryProtocol, TCompactProtocol, TJSONProtocol
)
from thrift.server import TServer, TNonblockingServer, THttpServer, TProcessPoolServer


class HelloServiceHandler:
    def say(self, msg):
        return msg
# 创建对应的服务处理
handler = HelloServiceHandler()
processor = HelloService.Processor(handler)
# 监听端口
transport = TSocket.TServerSocket("0.0.0.0", 9090)

# 选择传输层
tfactory = TTransport.TBufferedTransportFactory()
# tfactory = TTransport.TFramedTransportFactory()
# tfactory = TZlibTransport.TZlibTransportFactory()

# 选择传输协议
pfactory = TBinaryProtocol.TBinaryProtocolFactory()  # 二进制格式
# pfactory = TBinaryProtocol.TBinaryProtocolAcceleratedFactory()  # 二进制格式加速版
# pfactory = TCompactProtocol.TCompactProtocolFactory()  # 压缩格式
# pfactory = TJSONProtocol.TJSONProtocolFactory()  # json格式

# 选择服务模型
server = TServer.TSimpleServer(processor, transport, tfactory, pfactory)  # 单进程
# server = Tserver.TThreadPoolServer(processor, transport, tfactory, pfactory)  # 线程池
# server = TNonblockingServer.TNonblockingServer(processor, transport, pfactory, pfactory)  # 非阻塞(client 必须使用TFramedTransport 传输)
# 启动
server.serve()
```

socket listen 端口-> 选择传输协议和传输数据的方式->实现相应的 handler

generaterted code 里面的调用 process 对应的 xxx 方法 process_xxx方法

readMessageBegin->readStructBegin->readFieldBegin->readFieldEnd->readStructEnd->readMessageEnd获取数据然后逻辑处理数据->writeMessageBegin->writeSturtBegin->writeFieldBegin->writeFieldend->writeStructEnd->writeMessageEnd-> fulsh 返回处理过后的数据

根据源码分期了整个数据的处理过序列号和反序列化的过程
