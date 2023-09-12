

# 1、依赖查找



## 概念

容器提供对组件的回调和查找上下文。这是 EJB 和 Apache Avalon方法。它让每个组件都 有责任使用容器 API 来查找资源和协作者。控制反转仅限于调用回调方法的容器，应用程 序代码可以使用这些方法获取资源。&#x20;


典型实现

- Java Beans
- Java ServiceLoader SPI
- JNDI(Java Naming and Directory Interface)

        			 					<br />J2EE 核心设计模式 - Service Locator<br />使用服务定位器来实现和封装服务和组件查找。服务定位器隐藏查找机制的实现细节，并封装相关的依赖关系。 <br />[http://corej2eepatterns.com/ServiceLocator.htm](http://corej2eepatterns.com/ServiceLocator.htm)<br /> 			



# 2、依赖注入



## 概念

依赖注入(dependency injection)的意思为，给予调用方它所需要的事物。“依赖”是 指可被方法调用的事物。依赖注入形式下，调用方不再直接指使用“依赖”，取而代之是 “注入” 。“注入”是指将“依赖”传递给调用方的过程。在“注入”之后，调用方才会 调用该“依赖”。传递依赖给调用方，而不是让让调用方直接获得依赖，这个是该设计的 根本需求。在编程语言角度下，“调用方”为对象和类，“依赖”为变量。在提供服务的 角度下，“调用方”为客户端，“依赖”为服务。
