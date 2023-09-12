

# 1、Apache Tomcat 架构总览



## 设计意图

Tomcat 被设计成一个快速高效的Servlet规范实现。Tomcat 是作为本规范的参考实现 而出现的，并且一直严格遵守规范。同时，Tomcat 的性能也受到了极大的关注，它现在 与其他 Servlet 容器(包括商业容器)不相上下。&#x20;


在 Tomcat 的最新版本中，主要从 Tomcat 5 开始，我们已经开始努力通过 JMX 管理 Tomcat 的更多方面。此外，Manager 和 Admin webapps 也得到了极大的增强和改进。 随着产品的成熟和规范的稳定，可管理性是我们关注的主要领域。



## 目录结构

- bin - Tomcat 脚本目录，包含 *nix 和 Windows，如 startup.sh、startup.bat
- conf - Tomcat 配置文件目录，包括服务器配置、Web 上下文配置、安全配置等
- lib - Tomcat 运行时类库目录
- logs - Tomcat 日志目录，包括服务器日志、HTTP 访问日志、Web 应用日志
- temp - Tomcat 临时目录，重定向了 Java 系统属性 “java.io.tmpdir”
- webapps - Web 应用部署目录
- work - Tomcat JSP 翻译和变异存储目录 

## 术语解释

- Server - 在Tomcat世界中，Server 代表整个容器。Tomcat提供了 Server 接口的默认实现，很少由用户自定义

- Service - 是一个中间组件，它位于 Server 内部，并将一个或多个 Connector 连接 到一个 Engine 上。服务元素很少由用户自定义，因为默认的实现是简单而充分的Service 接口

- Engine - 表示特定 Service 的请求处理管道。由于一个 Service 可能有多个 Connector ，Engine 接收并处理来自这些连接器的所有请求，将响应传递回相应的 连接器以传输给客户端。 Engine 接口可以实现为提供定制 Engine ，尽管这并不常见。

- Host - 是网络名称的关联，例如。www.yourcompany.com，到Tomcat服务器。一 个 Engine 可以包含多个 Host ，并且Host元素还支持网络别名，例如 yourcompany.com和abc.yourcompany.com。用户很少创建自定义 Host ，因为 StandardHost 实现提供了重要的附加功能

- Connector - 处理与客户机的通信。Tomcat 有多个可用的 Connector。其中包括用 于大多数 HTTP 通信的 HTTP 连接器，特别是在将 Tomcat 作为独立服务器运行时， 以及实现将 Tomcat 连接到web服务器(如 Apache Httpd服务器)时使用的 AJP 协议的 AJP 连接器。创建自定义连接器是一项重要的工作。

- Context - 表示 web 应用程序。 Host 可以包含多个 Context，每个 Context 都有 一个唯一的路径。可以实现 Context 接口来创建自定义 Context，但这种情况很少发 生，因为 StandardContext 提供了重要的附加功能。



# 2、Apache Tomcat 服务器启动过程

过程一：命令行启动&#x20;


关联类：org.apache.catalina.startup.Bootstrap&#x20;


明细：

- a). 设置 ClassLoader
  - commonLoader (common)-> System Loader
  - sharedLoader (shared)-> commonLoader -> System Loader
  - catalinaLoader(server) -> commonLoader -> System Loader

- b). 加载启动类(通过反射)- org.apache.catalina.startup.Catalina
  - setParentClassloader -> sharedLoader
  - Thread.contextClassloader -> catalinaLoader

           			 						 					<br />过程二：处理命令行参数 <br />关联类:org.apache.catalina.startup.Bootstrap (assume command->start) <br />明细: 

- a). Catalina.setAwait(true)

- b). Catalina.load()
  - initDirs()
  - initNaming()
  - createStartDigester()
  - 加载server.xml并使用 Digester 对其进行解析
  - 将 System.out and System.err 分配到类 SystemLogHandler
  - 调用所有组件的 initialize 方法，使每个对象向 JMX 代理注册自己。

- c). Catalina.start()
  - 启动 NamingContext 并将所有JNDI引用绑定到其中
  - 启动<Server>下的服务:StandardService -> Engine(ContainerBase -> Realm,Cluster,...)
  - Service 启动 StandardHost
    - 配置 ErrorReportValve，输出 HTTP 错误代码对应的 HTML 页面
    - 启动 Pipeline 中的 Valve
    - 配置 StandardHostValve
    - 启动 HostConfig
  - 在容器(StandardEngine)的生命周期内，有一个后台线程不断检查上下文是否已更改。如果上下 文发生更改(war文件的时间戳，上下文xml文件，web.xml)，然后发出重新加载 (stop/remove/deploy/start)

- d). Tomcat通过HTTP端口接收请求
  - 请求由一个单独的线程接收，该线程正在 ThreadPoolExecutor 等待。它正在常规 ServerSocket.accept() 方法中等待请求。当收到请求时，此线程将被唤醒
  - ThreadPoolExecutor分配一个TaskThread来处理请求。它还为catalina容器提供了一个JMX对象名
  - 本例中处理请求的处理器是Coyote Http11Processor，然后调用process方法。这个处理器也在继续检 查套接字的输入流，直到达到保持活动点或连接断开
  - 使用内部缓冲区类(Http11InputBuffer)解析HTTP请求 buffer 类解析请求行、报头等，并将结果存储 在 Coyote 请求(不是HTTP请求)此请求包含所有HTTP信息，例如服务器名、端口、方案等
  - 处理器包含对适配器的引用，它是 CoyoteAdapter 。一旦请求被解析，Http11Processor 在适配器上 调用 service()。在服务方法中，请求包含 CoyoteRequest 和 CoyoteResponse(首次为空) CoyoteRequest(Response)实现HttpRequest(Response)和 HttpServletRequest(Response)
  - 解析完成后，CoyoteAdapter调用其容器(StandardEngine)并调用invoke(请求，响应)方法。这 将从引擎级别开始向Catalina容器发起HTTP请求
  - StandardEngine.invoke() 只调用容器 Pipeline.invoke()
  - 默认情况下，Engine 只有一个 StandardEngineValve，这个 Valve 简单调用 Host Pipeline.invoke() 方法(StandardHost.getPipeLine())
  - 默认情况下，StandardHost有两个 Valve 实现:StandardHostValve 和 ErrorReportValve
  - 标准主阀将正确的类装入器与当前线程相关联，它还检索与请求相关联的管理器和会话(如果有)，如 果存在会话，则调用 access() 使会话保持活动状态
  - 之后，StandardHostValve 调用关联上下文上的管道和请求一起。
  - Context Pipeline 调用的第一个 Vavle 是FormAuthenticator Vavle。然后 StandardContextValve 被 调用。StandardContextValve 调用与上下文关联的任何上下文 Listener。接下来，它调用包装器组件 (StandardWrapperValve)上的管道
  - 在调用 StandardWrapperValve 期间，JSP 包装器(Jasper)被调用这将导致JSP的实际编译。然后 调用实际的 Servlet

- e). 调用 Servlet 类
