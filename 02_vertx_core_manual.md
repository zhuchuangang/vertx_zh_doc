#vert.x core手册
vert.x核心的java api我们成为vert.x core.

源码库:https://github.com/eclipse/vert.x

vert.x核心提供的功能有:

- Writing TCP clients and servers

- Writing HTTP clients and servers including support for WebSockets

- The Event bus

- Shared data - local maps and clustered distributed maps

- Periodic and delayed actions

- Deploying and undeploying Verticles

- Datagram Sockets

- DNS client

- File system access

- High availability

- Clustering

核心的功能是相当低的水平,你不会找到数据库访问、授权或高级的web功能——这些东西,你会发现它们在vert.x ext(扩展)。

vert.x核心是小巧的.你可以只使用你想要的部分.它也可以被集成到你现有的应用中.我们不强制你使用特定的方式结构化你的应用.

你可以使用vert.x支持的其他语言来使用vert.x core.但是我们不强制你直接使用java API.不同的语言有不同的习惯和约定,让一个Ruby开发人员使用java的语法习惯会很别扭。相反,我们自动生成一个惯用的每种语言的核心Java api。

maven:
```
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-core</artifactId>
  <version>3.3.2</version>
</dependency>
```

gradle:
```
compile io.vertx:vertx-core:3.3.2
```

##1 vert.x初探
首先你需要创建一个Vertx对象,它是Vert.x的控制中心,它可以做很多事情,包括创建客户端和服务端,获取事件总线的引用,设置定时任务等等.

下面我们来创建一个Vertx实例.

如果你是想嵌入Vert.x,最简单方法就是:
```java
Vertx vertx = Vertx.vertx();
```

> 大多数的应用只需要单个Vert.x实例,但是如果你需要它可以创建多个Vert.x实例,例如事件总线之间的隔离或不同组的服务器和客户端。

###1.1 指定选项
如果默认配置不是你需要的,那么在Vertx对象创建的是需要指定选项:
```java
Vertx vertx = Vertx.vertx(new VertxOptions().setWorkerPoolSize(40));
```
VertxOptions对象有很多设置,允许你配置像集群,高可用,池大小等配置.javadoc中有详细描述.

###1.2 创建一个Vert.x集群
如果你要创建一个集群Vert.x(在 [event bus](http://vertx.io/docs/vertx-core/java/#event_bus) 获取更多信息),你通常会使用异步变体创建Vertx对象.

这是因为通常需要一段时间(也许几秒钟)不同的Vert.x实例在集群实例集合。在此期间,我们不想阻止调用线程,所以我们异步的返回结果。

##2 are you fluent?
你可能已经注意到,在前面的例子使用流利的API。

流利的API是多种方法调用可以链接在一起的地方。例如:
```java
request.response().putHeader("Content-Type", "text/plain").write("some text").end();
```
在vert.x中很常见,所以要去适应它.

这样的链接调用可能会让你编写代码有点冗长。当然,如果你不喜欢流利的方法我们不强迫你这么做,你可以快乐地忽略它如果你喜欢写这样的代码:
```java
HttpServerResponse response = request.response();
response.putHeader("Content-Type", "text/plain");
response.write("some text");
response.end();
```

##3 不要调用我们,我们会调用你
Vert.x api是一个庞大的事件驱动.这个意味着在Vert.x中事情发生可能是你感兴趣的,Vert.x将会发送给你事件.

这些事件例如有:

- 定时器被触发
- socket接受到数据
- 从磁盘上读取数据
- 抛出一个异常
- HTTP服务接受到一个请求

你处理事件通过提供Vert.x提供的API处理。例如接收一个计时器事件每一秒你会做的事:
```java
vertx.setPeriodic(1000, id -> {
  // This handler will get called every second
  System.out.println("timer fired!");
});
```
或者接收到一个HTTP请求:
```
server.requestHandler(request -> {
  // This handler will be called every time an HTTP request is received at the server
  request.response().end("hello world!");
});
```

















































