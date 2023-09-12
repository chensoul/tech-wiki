

# 1、Servlet 简介

Servlet 是一种基于 Java 技术的 Web 组件，用于生成动态内容，由容器管理。类似于其他 Java 技术组件，Servlet 是平台无关的 Java 类组成，并且由 Java Web 服务器加载执行。 通常情况，由 Servlet 容器提供运行时环境。Servlet 容器，有时候也称作为 Servlet 引擎， 作为Web服务器或应用服务器的一部分。通过请求和响应对话，提供Web 客户端与Servlets 交互的能力。容器管理Servlets实例以及它们的生命周期。 

# 2、Servlet 主要版本

![image.png](../assets/ct96goktzy8yu3k0/1676179703876-9ab50333-534e-499e-b8d6-4df1f3faa66d.png) 



# 3、Servlet 设计模式



## 设计模式

GOF 23 设计模式


通用面向对象设计模式


核心 J2EE 设计模式

- <http://www.corej2eepatterns.com/index.htm>

Presentation Tier

- Intercepting Filter
  - Servlet 中的 Filter
  - Tomcat 中的 Value
  - Spring WebMVC 中的 HandlerInterceptor
- Context Object：上下文对象，主要作为视图所引用的对象
  - Spring WebMVC：Model 或 ModelAndView
- Front Controller：前端总控制器，将请求通过映射逻辑（Request Mapping）转发响应的 ApplicationController
  - Spring WebMVC: DispatcherServlet
- Application Controller：应用控制器，具体处理请求对应资源处理，执行业务逻辑，并且存储视图需要上下文。可以作为 View 中的 Model
  - Spring WebMVC: @Controller，命令模式实现
- View Helper：视图辅助对象
  - Spring WebMVC: JSTL标签，比如：<spring:form>
  - Struts：JSTL标签
  - 内置上下文
- Composite View： 组合视图，相反的是，单一视图。模版引擎实现模式
  - Layout：布局、head、body、footer
- Dispatcher View：分发视图
  - Servlet：RequestDispatcher#forward，本质从一个 Servlet 资源转发到另外一个 Sevlet 资源，在同义词 HTT： 请求内完成（统一线程处理）
  - Spring WebMVC:
- Service To Worker：将具体的处理逻辑交给相关工作组件完成



## Model 1 架构

![image.png](assets/ct96goktzy8yu3k0/1676179775561-397fb681-fe74-4822-abb1-759eee1e72f1.png)Model 2 架构


角色：

- JSP：接受用户 HTTP 请求，并且数据转发到 JavaBeans
- JavaBeans：处理业务逻辑，并且执行数据库存储
- DB：数据持久化



## Model 2 架构（MVC）

![image.png](assets/ct96goktzy8yu3k0/1676179868437-3bea8c60-1724-4110-85f7-35deac7910d7.png)&#x9;


角色：

- Servlet 作为控制器
- JavaBeans 作为模型
- JSP 作为视图 

## Front Controller 模式

![image.png](assets/ct96goktzy8yu3k0/1676179841015-1d7019b2-5ea0-4630-8f77-3abe07c668e8.png)



# 4、Servlet 扩展

Struts Web MVC


![image.png](assets/ct96goktzy8yu3k0/1676179933843-b88d9894-f9f1-4efb-a5c1-efc34c943749.png)


&#x20;Spring Web MVC


&#x20;![image.png](assets/ct96goktzy8yu3k0/1676180002238-0a296b2d-0894-426c-ae3e-4d00f9e618e9.png) 

# 5、Servlet 规范



## Servlet 核心接口

- 服务组件
  - javax.servlet.Servlet
  - javax.servlet.Filter(since Servlet 2.3)
- 上下文组件：
  - javax.servlet.ServletContext：Servlet 应用上下文范围，不宜存放短生命周期对象，比如：当前请求相关，底层存储类似于 Map
    - 操作方法：
      - ServletContext#setAttribute：对象需要实现序列化接口
      - ServletContext#getAttribute
      - ServletContext#removeAttribute
      - ServletContext#getAttributeNames
    - 容错设计
      - 假设 Servlet 引擎意外宕机，Servlet 应用中保存了不少的对象，下次启动时，如何恢复这些对象状态？
  - javax.servlet.http.HttpSession
    - 跟踪方式，保存会话的状态
      - Cookie（JSESSIONID）
      - URL rewrite（JSESSIONID）
      - SSL
    - 容错设计
      - 假设 Servlet 引擎意外宕机，Servlet 应用中保存了不少的对象，下次启动时，如何恢复这些对象状态？
    - 状态属性
      - 时间相关
        - getCreationTIme()：创建时间，布标
        - getLastAccessedTIme()：最后访问时间，动态变化
        - getMaxInactiveInterval()：最大活跃间隔时间（单位：秒）
      - 有效性
        - invalidate()：主动失效
        - isNew() ：是否为新的 HttpSession
  - javax.servlet.http.HttpServletRequest
    - HttpSession 相关
      - getSession()：获取当前请求所关联的 HttpSession，如果不存在，则创建一个新的，永远不会返回 null
      - getSession()：获取当前请求所关联的 HttpSession，如果参数为 false 并且请求没有关联 HttpSession 的话，name 就返回 null
    - HttpSession ID 相关
      - getRequestedSessoionId()：获取客户端请求中的 Session Id，可能为 null
      - isRequestedSessionIdValid()：请求 Session Id 是否合法
      - isRequestedSessionIdFromCookie()：请求 Session Id 是否来自与 Cookie
      - isRequestedSessionIdFromURL()：请求 Session Id 是否来自与 URL
  - javax.servlet.http.HttpServletResponse
  - javax.servlet.http.Cookie(客户端)
- 配置
  - javax.servlet.ServletConfig
  - javax.servlet.FilterConfig(since Servlet 2.3 )
- 输入输出
  - javax.servlet.ServletInputStream
    - 来源：ServletRequest#getInputStream
    - 默认实现来自于 Servlet 容器，并且是一次性使用
    - 可能由 ServletRequestWrapper 实现，比如提供多次数据读取
      - Wrapper 实现通常与 Filter 实现配合使用，因为 FIlter 组件执行优先于 Servlet
  - javax.servlet.ServletOutputStream
- 异常
  - javax.servlet.ServletException
- 事件(since Servlet 2.3 )
  - 生命周期类型
    - javax.servlet.ServletContextEvent
    - javax.servlet.http.HttpSessionEvent
    - java.servlet.ServletRequestEvent
  - 属性上下文类型
    - javax.servlet.ServletContextAttributeEvent
    - javax.servlet.http.HttpSessionBindingEvent
    - javax.servlet.ServletRequestAttributeEvent
- 监听器(since Servlet 2.3)
  - 生命周期类型
    - javax.servlet.ServletContextListener
    - javax.servlet.http.HttpSessionListener
    - javax.servlet.http.HttpSessionActivationListener
    - javax.servlet.ServletRequestListener
  - 属性上下文类型
    - javax.servlet.ServletContextAttributeListener
    - javax.servlet.http.HttpSessionAttributeListener
    - javax.servlet.http.HttpSessionBindingListener
    - javax.servlet.ServletRequestAttributeListener
- 组件申明注解(since Servlet 3.0)
  - @javax.servlet.annotation.WebServlet
  - @javax.servlet.annotation.WebFilter
  - @javax.servlet.annotation.WebListener
  - @javax.servlet.annotation.ServletSecurity
  - @javax.servlet.annotation.HttpMethodConstraint
  - @javax.servlet.annotation.HttpConstraint
- 配置申明
  - @javax.servlet.annotation.WebInitParam
- 上下文
  - javax.servlet.AsyncContext
- 事件
  - javax.servlet.AsyncEvent
- 监听器
  - javax.servlet.AsyncListener
- Servlet 组件注册
  - javax.servlet.ServletContext#addServlet()
  - javax.servlet.ServletRegistration
- Filter 组件注册
  - javax.servlet.ServletContext#addFilter()
  - javax.servlet.FilterRegistration
- 监听器注册
  - javax.servlet.ServletContext#addListener()
  - javax.servlet.AsyncListener 

## Servlet 3.1 规范重点章节

- CHAPTER 2 The Servlet Interface
- CHAPTER 3 The Request
- CHAPTER 4 Servlet Context
- CHAPTER 5 The Response
- CHAPTER 9 Dispatching Requests
- CHAPTER 11Application Lifecycle Events
- CHAPTER 12 Mapping Requests to Servlets
