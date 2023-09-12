# 0. 框架总览

- 特性总览
- 版本特性
- 模块化设计
- 技术整合
  - Java语言特性运用
  - JDK API实践
  - Java EE API 整合
- 编程模型
  - 面向对象编程
    - 契约接口
  - 面向切面编程
    - 动态代理
    - 字节码提升
  - 面向元编程
    - 配置元信息
    - 注解
    - 属性配置
  - 面向模块编程
    - Maven Artifacts
    - ~~OSGI Bundles~~
    - Java 9 Automatic Modules
    - Spring @EnableXXX 注解
  - 面向函数编程
    - Lambda
    - Reactive
- IOC容器
  - 重新认识 IOC
  - Spring IOC 容器
  - 依赖查找
  - 依赖注入
  - 依赖来源
  - Spring IOC 生命周期
- Bean
  - Bean实例
  - Bean作用域
  - Bean生命周期
- 元信息
  - 注解
  - 配置元信息
  - 外部化属性
- 基础设施
  - 类型转换
  - 数据绑定
  - 校验
  - 国际化
  - 事件
  - 范型处理



# 1. 课前准备

## 心态

1. 戒骄戒躁
2. 谨慎豁达
3. 如履薄冰

## 方法

**基础**：夯实基础，了解动态
**思考**：保持怀疑，验证一切
**分析**：不拘小节，观其大意
**实践**：思辨结合，学以致用

## 工具

- JDK: Oracle JDK 8
- Spring Framework: 5.2.2.RELEASE
- IDE: IntelliJ IDEA 2019 (Community)
- Maven: 3.2+



# 2. Spring 特性总览

## 核心特性（Core）

- IoC 容器（IoC Container）
- Spring 事件（Events）
- 资源管理（Resources）
- 国际化（i18n）
- 校验（Validation）
- 数据绑定（Data Binding）
- 类型转换（Type Conversion）
- Spring 表达式（Spring Express Language）
- 面向切面编程（AOP）

## 数据存储（Data Access）

- JDBC
- 事务抽象
- DAO 支持（DAO Support）
- O/R 映射（O/R Mapping）
- XML 编列（XML Marshalling）

## Web 技术（Web）

- Web Servlet 技术

  - Spring MVC

  - WebSocket

  - SockJS

- Web Reactive 技术栈

  - Spring WebFlux

  - WebClient

  - WebSocket

## 技术整合

- 远程调用（Remoting）
- Java 消息服务（JMS）
- Java 连接架构（JCA）
- Java 管理扩展（JMX）
- Java 邮件客户端（Email）
- 本地任务（Tasks）
- 本地调度（Scheduling）
- 缓存抽象（Caching）
- Spring 测试（Testing）

# 3. Spring 版本特性

| Spring Framework版本 | Java标准版本 | Java企业版本      | Spring Boot | 说明                                                         |
| -------------------- | ------------ | ----------------- | ----------- | ------------------------------------------------------------ |
| 1.x                  | 1.3+         | J2EE1.3+          |             | Java1.3 引入动态代理，J2EE1.3对应的Servlet 2.3引入了Servlet事件 |
| 2.x                  | 1.4.2+       | J2EE1.3+          |             | Java 1.4.2 支持NIO                                           |
| 3.x                  | 5+           | J2EE1.4和Java EE5 |             | Java5 支持注解                                               |
| 4.x                  | 6+           | Java EE 6和7      | 1.x         | 支持Spring Boot 1.x                                          |
| 5.x                  | 8+           | Java EE7          | 2.x         | Java支持lambda语法，支持Spring Boot 2.x                      |
| 6.x                  | 17+          | Java EE7          | 3.x         | 支持Spring Boot 3.x                                          |

下面是 Spring 各个版本的主要特性：

- Spring 1.0：Spring的第一个版本，引入了IoC（Inversion of Control，控制反转）和AOP（Aspect-Oriented Programming，面向切面编程）的核心概念。它提供了一个轻量级的容器，用于管理Java对象的生命周期和依赖关系。
- Spring 1.1：提供了对 JNDI、JavaMail 和 JMS 的支持，增强了 AOP 功能，引入了新的 Web 框架 Spring Web Flow。
  - AOP框架改进：Spring 1.1引入了更强大和灵活的AOP（Aspect-Oriented Programming，面向切面编程）框架。它支持基于代理的AOP和基于字节码的AOP，并提供了更多的切入点表达式和增强类型。
  - JDO（Java Data Objects）集成：Spring 1.1增加了对JDO的支持，使得使用JDO持久化框架的开发更加便捷。它提供了对JDO的集成模块，并简化了使用JDO进行数据访问的配置和编码工作。
  - JMX（Java Management Extensions）集成：Spring 1.1引入了对JMX的支持，使得应用程序的管理和监控变得更容易。它提供了JMX集成模块，可以将Spring管理的Bean暴露为JMX MBean，并通过JMX进行管理和监控。
  - 拦截器链改进：Spring 1.1改进了拦截器链的处理方式，使得在拦截器链中添加和管理拦截器变得更简单。它引入了更灵活的拦截器链配置方式，并提供了更多的拦截器链处理选项。
  - 校验框架改进：Spring 1.1对校验框架进行了改进，提供了更丰富的校验功能和更灵活的校验配置选项。它引入了校验框架的集成模块，并支持基于注解的校验和自定义校验器。
  - Web框架改进：Spring 1.1对Web框架进行了改进，提供了更强大和灵活的Web开发功能。它引入了更多的Web组件和增强选项，包括新的控制器抽象、请求参数绑定增强、表单处理增强等。

- Spring 2.0：引入了许多新功能，其中包括对Java 5的支持。主要特性包括注解驱动的开发，通过注解减少XML配置的需求，引入了@Component和@Autowired注解等。还引入了Spring MVC框架，用于构建Web应用程序。
  - 注解驱动开发：Spring 2.0引入了对注解的广泛支持，使得开发人员可以使用注解来简化配置和开发。通过使用注解，可以将依赖注入、切面配置、事务管理等功能与代码紧密集成，减少了对XML配置的需求。- 
  - Java 5支持：Spring 2.0对Java 5进行了全面支持，包括泛型（Generics）、枚举（Enums）、注解（Annotations）等特性。这使得在Spring应用程序中能够更好地利用Java 5的语言特性，提高代码的可读性和类型安全性。
  - 模块化设计：Spring 2.0引入了模块化设计，使得开发人员可以根据需求选择所需的模块，并根据需要进行组合。这样可以减少不必要的依赖，并提供更轻量级的部署和更快的启动时间。
  - Spring MVC改进：Spring 2.0对Spring MVC进行了改进，引入了更多的注解和便利的配置选项，使得构建Web应用程序更加简单和灵活。它提供了更好的RESTful风格的支持，并提供了更丰富的数据绑定和验证功能。
  - AOP改进：Spring 2.0在AOP方面进行了改进，提供了更强大和灵活的AOP功能。它引入了更多的切入点表达式、增强类型和织入选项，使得开发人员能够更好地利用AOP来解耦和增强应用程序。
  - JPA集成：Spring 2.0提供了对Java持久化API（Java Persistence API，JPA）的集成支持。它使得在Spring应用程序中使用JPA变得更加容易，提供了对JPA实体管理器的集成和事务管理的支持。
  - 增强的测试支持：Spring 2.0引入了更强大的测试支持，包括对单元测试、集成测试和端到端测试的支持。它提供了Spring测试框架和Mock对象，使得在Spring环境中进行测试变得更加简单和可靠。
- Spring 2.1：引入了 Spring Aspects，提供了对 JPA 2.0 和 Java EE 6 的支持，引入了对 JSR-303 Bean Validation 的支持。
  - Java 6支持：Spring 2.1开始官方支持Java 6，并充分利用Java 6的新特性。这包括对Java 6的注解、并发工具和改进的I/O等方面的支持。
  - Spring EL改进：Spring 2.1引入了对Spring Expression Language（SpEL）的改进。SpEL是一种强大的表达式语言，用于在Spring应用程序中进行属性访问、方法调用和逻辑计算等操作。在2.1版本中，SpEL得到了性能和功能方面的提升。
  - JPA 2.0支持：Spring 2.1提供了对Java Persistence API（JPA）2.0规范的支持。JPA 2.0带来了一些重要的改进，包括标准查询语言（JPQL）的改进、Criteria查询API的引入和更好的集合处理支持等。
  - Spring MVC改进：Spring 2.1对Spring MVC进行了一些改进和增强。它引入了更多的注解和配置选项，使得开发者可以更轻松地构建RESTful风格的Web应用程序。此外，它还提供了更好的文件上传和异常处理支持。
  - 缓存抽象和支持：Spring 2.1引入了缓存抽象和支持，使得在应用程序中使用缓存变得更加简单和灵活。它提供了对各种缓存提供商（如Ehcache、GemFire、Redis等）的集成，并提供了简单的注解配置和缓存管理功能。
  - 异步支持改进：Spring 2.1对异步编程支持进行了改进，引入了对Java 5 `java.util.concurrent` 包的更好集成，使得在应用程序中使用异步方法和任务变得更加方便。

- Spring 2.5：提供了对 JPA 的支持，引入了基于注解的配置方式，支持对 Java 5 特性的利用。
  - 注解驱动开发的增强：Spring 2.5进一步增强了对注解驱动开发的支持。它引入了更多的注解，如@Component、@Repository、@Service和@Controller等，用于标识不同类型的组件。这样可以更容易地进行组件扫描和自动装配。
  - Spring MVC改进：Spring 2.5对Spring MVC进行了一些改进和增强。它引入了`@RequestMapping`注解，用于更灵活地处理请求映射。此外，它还提供了更好的对RESTful风格的支持，并增强了表单处理和数据绑定功能。
  - Bean生命周期的改进：Spring 2.5提供了更细粒度的控制和扩展Bean生命周期的方式。通过引入`@PostConstruct`和`@PreDestroy`注解，开发人员可以在Bean初始化和销毁时执行自定义的逻辑。
  - 基于注解的配置：Spring 2.5进一步推进了基于注解的配置方式。除了支持对Bean的注解配置外，它还支持对依赖注入和AOP的注解配置。这样可以减少XML配置的需求，简化配置文件。
  - JSP标签库：Spring 2.5引入了Spring的JSP标签库，用于在JSP页面中使用Spring的功能。这包括表单标签、数据绑定标签、消息标签等，使得在JSP页面中使用Spring更加方便。
  - 事务管理的增强：Spring 2.5对事务管理进行了一些增强。它提供了对基于注解的声明性事务管理的支持，通过`@Transactional`注解可以轻松地将事务逻辑应用于方法。此外，它还增强了对程序式事务管理的支持。

- Spring 3.0：在这个版本中，Spring引入了Java配置（JavaConfig）作为替代XML配置的选项。它还引入了基于注解的Bean定义，支持基于Java的容器配置。Spring 3.0还增强了对RESTful Web服务的支持，并提供了对Servlet 3.0、JPA 2.0和Java EE 6的集成支持。
  - Java 5和Java 6支持：Spring 3.0开始完全基于Java 5进行编译，并对Java 6进行了全面支持。这包括对Java 5的泛型、枚举、注解和Java 6的改进特性的广泛应用。
  - Java配置（JavaConfig）：Spring 3.0引入了基于Java的配置方式，即JavaConfig。开发人员可以使用Java代码来定义Spring Bean的配置信息，而不再依赖于XML配置文件。这提供了一种更可读性强、类型安全的配置方式。
  - REST支持：Spring 3.0加强了对REST（Representational State Transfer）风格的支持。它提供了`@PathVariable`注解用于处理RESTful风格的URL路径参数，以及`@RequestBody`注解用于处理请求体的内容。这使得开发RESTful Web服务更加方便。
  - Spring MVC改进：Spring 3.0对Spring MVC进行了一系列的改进和增强。它引入了`@RequestMapping`注解的更多用法，包括支持Ant风格的URL路径匹配和正则表达式的支持。此外，它还提供了更好的表单处理和数据绑定功能。
  - RESTful客户端支持：Spring 3.0提供了对RESTful Web服务的客户端支持。通过`RestTemplate`类，开发人员可以方便地发送HTTP请求，并处理响应结果。这简化了与RESTful服务的交互。
  - Bean验证（Bean Validation）：Spring 3.0集成了Java Bean验证（Bean Validation）规范，使得在Spring应用程序中可以更方便地进行数据验证和约束。开发人员可以使用注解来定义验证规则，并通过Spring提供的验证器进行验证。
  - 缓存抽象和支持的增强：Spring 3.0改进了对缓存的支持。它提供了更强大的缓存抽象，支持多种缓存提供商，如Ehcache、GemFire和Redis等。此外，它还提供了对注解驱动的缓存配置，简化了缓存的使用。
  - 异步支持改进：Spring 3.0对异步编程支持进行了改进。它引入了`DeferredResult`和`Callable`等新的异步处理方式，使得在处理大量并发请求时能够更高效地利用系统资源。

- Spring 3.1：提供了对 Java 7 特性的支持，引入了基于 Java 注解的缓存管理。
  - 编程模型的改进：Spring 3.1引入了一些编程模型的改进，使得开发人员可以更轻松地使用Spring框架。其中包括对注解的更多支持，如`@ComponentScan`用于自动扫描组件、`@Profile`用于配置不同环境下的Bean等。
  - 缓存抽象的改进：Spring 3.1对缓存抽象进行了改进和扩展。它引入了`@Cacheable`和`@CacheEvict`等注解，使得开发人员可以更方便地在方法级别上进行缓存控制。此外，它还提供了对JSR-107（Java Caching API）的支持。
  - Spring MVC改进：Spring 3.1对Spring MVC进行了一些改进和增强。它引入了`@RequestMapping`注解的更多选项，如`consumes`和`produces`用于指定请求的内容类型和响应的内容类型。此外，它还增强了对RESTful Web服务的支持。
  - 异步支持改进：Spring 3.1对异步编程支持进行了改进。它引入了`@Async`注解，用于标识异步方法。这样可以更方便地在应用程序中使用异步方法，提高应用程序的性能和响应能力。
  - 数据访问改进：Spring 3.1对数据访问进行了一些改进。它提供了更好的JDBC支持，包括对简单JDBC操作的简化、对命名参数的支持和对批处理操作的增强。此外，它还加强了对NoSQL数据库的支持，如MongoDB和Redis等。
  - Java 7支持：Spring 3.1开始官方支持Java 7，并充分利用Java 7的新特性。这包括对Java 7的异常处理、try-with-resources语句和类型推断等方面的支持。
  - WebSocket支持：Spring 3.1引入了对WebSocket的支持。它提供了WebSocket消息传递的抽象和编程模型，使得在Spring应用程序中可以更方便地实现实时通信功能。

- Spring 3.2：引入了对 WebSocket 和 Spring MVC Test 的支持，提供了更好的 Java 8 支持。
  - Java配置（JavaConfig）的增强：Spring 3.2进一步增强了Java配置的功能。它引入了`@Configuration`注解的派生注解，如`@Import`和`@ComponentScan`，使得配置类之间的引入和组件扫描更加方便。此外，它还提供了对`@Bean`方法的条件化配置。
  - Spring MVC的改进：Spring 3.2对Spring MVC进行了一系列的改进和增强。它引入了`@ControllerAdvice`注解，用于定义全局的控制器增强器。此外，它还提供了更好的异步请求处理和文件上传处理的支持。
  - WebSocket的改进：Spring 3.2对WebSocket的支持进行了改进。它提供了更强大的WebSocket消息传递功能，包括对消息编码和解码、消息拦截器、WebSocket握手处理等的支持。
  - Bean验证（Bean Validation）的增强：Spring 3.2增强了对Bean验证的支持。它提供了对JSR-303（Bean Validation）规范的更全面的支持，包括对验证组、验证序列和验证器的支持。
  - 缓存抽象的改进：Spring 3.2对缓存抽象进行了改进和扩展。它引入了`@CacheConfig`注解，用于对缓存的统一配置。此外，它还提供了对注解驱动的缓存配置的更多选项，如条件化缓存和自定义缓存解析器。
  - SpEL（Spring Expression Language）的增强：Spring 3.2增强了SpEL的功能。它引入了更多的SpEL表达式语法和操作符，使得在配置文件和注解中可以更灵活地使用SpEL表达式。
  - 数据访问的改进：Spring 3.2对数据访问进行了一些改进。它提供了对JPA 2.0的更好支持，包括对JPA Criteria查询的支持和对JPA命名查询的增强。此外，它还增强了对JDBC和ORM框架的支持。

- Spring 4.0：这个版本主要集中在增强Java 8的支持，包括对Lambda表达式和Stream API的支持。它还引入了条件化的配置，允许开发人员根据特定条件选择性地加载或排除配置。Spring 4.0还提供了对WebSocket通信的支持，并改进了对RESTful Web服务的支持。
  - Java 8支持：Spring 4.0增加了对Java 8的支持，可以充分利用其新特性，如Lambda表达式和新的日期和时间API。这使得开发人员可以在Spring应用程序中使用Java 8的强大功能和便利性。
  - Groovy支持：Spring 4.0改进了对Groovy编程语言的支持。它提供了更好的Groovy集成，允许开发人员在Spring应用程序中使用Groovy脚本和表达式。
  - WebSocket支持：Spring 4.0引入了全面的WebSocket支持。它提供了WebSocket协议的实现和抽象，使得在Spring应用程序中可以更方便地实现实时通信功能。
  - REST支持改进：Spring 4.0对REST（Representational State Transfer）支持进行了改进。它提供了更好的RESTful Web服务开发支持，包括对JAX-RS（Java API for RESTful Web Services）的集成和对HATEOAS（Hypermedia as the Engine of Application State）的支持。
  - HTML5支持：Spring 4.0增强了对HTML5的支持。它提供了对HTML5表单和输入类型的支持，以及对WebSockets、Server-Sent Events和WebSockets的集成。
  - 条件化配置支持：Spring 4.0引入了条件化配置的支持。开发人员可以使用条件化注解来根据特定条件选择性地加载和配置Bean，从而提高应用程序的灵活性和可扩展性。

- Spring 4.1：提供了对 JCache 的支持，引入了对 Java 8 时间 API 的支持。
  - 缓存抽象的改进：Spring 4.1对缓存抽象进行了改进和扩展。它引入了`@CacheConfig`注解，用于对缓存的统一配置。此外，它还提供了对注解驱动的缓存配置的更多选项，如条件化缓存和自定义缓存解析器。
  - Groovy 2.3支持：Spring 4.1增强了对Groovy 2.3的支持。它允许开发人员在Spring应用程序中使用Groovy 2.3的新特性和语法。
  - WebSocket支持的改进：Spring 4.1对WebSocket支持进行了改进。它提供了更好的WebSocket消息编码和解码的支持，以及对WebSocket拦截器的增强。
  - Spring MVC改进：Spring 4.1对Spring MVC进行了一系列改进和增强。它提供了更好的Servlet 3.0+异步请求处理的支持，以及对HTTP协议的HTTP Patch方法的支持。
  - HTTP/2支持：Spring 4.1引入了对HTTP/2协议的支持。它提供了对HTTP/2的原生支持，使得在Spring应用程序中可以更高效地利用HTTP/2协议的特性。
  - JCache支持：Spring 4.1增加了对JCache的支持。它提供了对JSR-107（JCache）规范的支持，使得在Spring应用程序中可以更方便地使用缓存的标准API。
  - 测试改进：Spring 4.1对测试支持进行了改进。它提供了更好的测试环境配置和管理的支持，以及对测试数据的加载和清理的更灵活的方式。

- Spring 4.2：提供了对 HTTP/2、Server-Sent Events 和 JSON 响应的支持，引入了 Spring Session。
  - HTTP/2支持的改进：Spring 4.2对HTTP/2支持进行了改进。它提供了更好的HTTP/2客户端支持，并优化了在Servlet容器中使用HTTP/2的性能。
  - Spring MVC的改进：Spring 4.2对Spring MVC进行了一系列的改进和增强。它提供了更好的RESTful Web服务开发支持，包括对HTTP方法的约束和对请求和响应的内容协商的支持。此外，它还提供了对SSE（Server-Sent Events）的支持。
  - Groovy 2.4支持：Spring 4.2增强了对Groovy 2.4的支持。它允许开发人员在Spring应用程序中使用Groovy 2.4的新特性和语法。
  - 条件化配置的增强：Spring 4.2进一步增强了条件化配置的功能。它引入了`@Conditional`注解，允许根据特定条件决定是否加载和配置Bean。
  - WebSocket支持的改进：Spring 4.2对WebSocket支持进行了改进。它提供了更好的WebSocket会话管理和错误处理的支持，以及对WebSocket消息转换器的增强。
  - Spring Test的改进：Spring 4.2改进了Spring Test模块的功能。它提供了更好的测试环境配置和管理的支持，以及对测试数据的加载和清理的更灵活的方式。
  - 缓存抽象的改进：Spring 4.2对缓存抽象进行了改进和扩展。它提供了对Cache 2.0规范的支持，包括对注解驱动缓存的异步操作和条件化缓存的支持。

- Spring 4.3：提供了对 HTTP PATCH 方法的支持，引入了对 CORS 的支持。
  - ore容器改进：Spring 4.3对核心容器进行了改进。它提供了更简化的配置选项，包括对@Configuration类的自动检测和解析，以及对@Bean方法的条件化处理。此外，它还提供了更好的泛型注入支持，使得依赖注入更加简洁和类型安全。
  - 缓存抽象的改进：Spring 4.3进一步改进了缓存抽象。它引入了对JSR-107（JCache）2.0的支持，包括对JCache注解的支持和对JCache缓存管理器的集成。同时，它还提供了更好的缓存清除和更新策略的支持。
  - Spring MVC的改进：Spring 4.3对Spring MVC进行了一系列改进。它提供了更好的注解驱动的控制器方法返回类型处理，包括对集合和Optional类型的支持。此外，它还改进了HTTP消息转换器的支持，支持HTTP PATCH方法，以及提供了更好的跨域请求处理。
  - 测试改进：Spring 4.3改进了Spring Test模块的功能。它引入了对JUnit 5的支持，包括对JUnit 5扩展模型和JUnit 5注解的支持。此外，它还提供了更好的测试环境配置和管理的支持，以及对测试数据的加载和清理的更灵活的方式。
  - WebSocket支持的改进：Spring 4.3对WebSocket支持进行了改进。它提供了更好的WebSocket会话管理和错误处理的支持，以及对STOMP（Simple Text Oriented Messaging Protocol）的支持。

- Spring 5.0：这个版本的主要特性是对响应式编程的支持，引入了Spring WebFlux框架，用于构建响应式Web应用程序。它还对核心容器进行了改进，提供了对Java 9和Java EE 8的支持。此外，Spring 5.0还引入了对函数式端点和反应式数据访问的支持。
  - 响应式编程支持：Spring 5.0引入了响应式编程模型，通过Reactor项目提供对反应式流和异步编程的支持。它可以帮助开发人员构建高性能、高吞吐量的响应式应用程序，适用于处理大量并发请求和事件流的场景。
  - 支持Java 8及以上版本：Spring 5.0完全支持Java 8及以上版本，并充分利用了Java 8的新特性，如Lambda表达式和流式API。这使得开发人员可以编写更简洁、可读性更强的代码。
  - 核心容器改进：Spring 5.0对核心容器进行了改进。它提供了对Java 8的函数式风格Bean定义的支持，以及对泛型注入的改进。此外，它还简化了条件化配置和自动装配的方式。
  - WebFlux：Spring 5.0引入了WebFlux，它是Spring框架的反应式Web编程模型。WebFlux提供了对响应式HTTP和WebSocket的支持，可以构建高性能、非阻塞的Web应用程序。
  - Spring Web MVC改进：Spring 5.0对Spring Web MVC进行了改进。它提供了对函数式风格的Web编程的支持，通过注解驱动的方式实现更简洁、灵活的控制器定义。
  - 安全性改进：Spring 5.0改进了安全性功能。它引入了对OAuth 2.0的原生支持，提供了更简化的OAuth 2.0客户端和资源服务器配置。
  - 测试改进：Spring 5.0对测试支持进行了改进。它提供了对JUnit 5的支持，并简化了集成测试和模拟对象的使用方式。

- Spring 5.1：这个版本主要关注于改进性能和稳定性。它引入了基于Kotlin的扩展函数，用于简化Spring框架的使用。此外，Spring 5.1还增强了Spring WebFlux框架，并改进了对Reactive Stream规范的支持。
  - Kotlin支持改进：Spring 5.1对Kotlin的支持进行了改进。它提供了更好的Kotlin扩展函数支持，并优化了与Kotlin协程的集成。
  - 响应式Web客户端：Spring 5.1引入了响应式Web客户端。它提供了一种非阻塞的、基于反应式流的方式来进行HTTP请求，并支持对响应流的处理。
  - WebFlux改进：Spring 5.1对WebFlux进行了改进。它提供了更多的注解驱动编程模型，包括对函数式端点的支持，以及对响应式数据访问的增强。
  - JUnit 5扩展支持：Spring 5.1增强了对JUnit 5扩展模型的支持。它提供了更丰富的扩展点和API，使得在Spring应用程序中可以更方便地编写自定义扩展。
  - WebSocket改进：Spring 5.1对WebSocket支持进行了改进。它提供了对WebSocket会话的更细粒度的控制，以及对WebSocket消息编解码器的增强。
  - Spring Security改进：Spring 5.1改进了Spring Security模块。它引入了对OAuth 2.0的更多改进和扩展，包括对OAuth 2.0授权服务器和资源服务器的支持。
  - RSocket支持：Spring 5.1引入了对RSocket协议的支持。RSocket是一种面向消息的、跨语言的网络通信协议，Spring框架提供了对RSocket的原生支持。

- Spring 5.2：这个版本引入了许多新特性，包括对Java 13和Java EE 8的支持。它还提供了对Spring Boot 2.2的支持，并改进了Spring WebFlux、Spring Data和Spring Security等模块。
  - 非阻塞编程模型改进：Spring 5.2进一步改进了非阻塞编程模型。它引入了对Kotlin协程的支持，使得在响应式应用程序中使用Kotlin协程更加方便。
  - WebFlux改进：Spring 5.2对WebFlux进行了一些改进。它提供了更好的函数式编程模型，包括对函数式Web请求处理的增强，以及对WebClient的改进和扩展。此外，它还增加了对RSocket的支持。
  - Spring Boot集成改进：Spring 5.2改进了与Spring Boot的集成。它提供了更好的自动配置和启动器支持，简化了使用Spring Boot的开发流程。
  - Spring Security改进：Spring 5.2改进了Spring Security模块。它引入了更多的安全特性和改进，包括对OAuth 2.0的更好支持，以及对WebFlux的安全性增强。
  - Micrometer集成：Spring 5.2引入了对Micrometer的集成。Micrometer是一个用于应用程序度量的通用度量库，Spring框架通过集成Micrometer，提供了更好的应用程序度量和监控的支持。
  - Kotlin支持改进：Spring 5.2进一步改进了对Kotlin的支持。它提供了更好的Kotlin协程支持，并优化了与Kotlin协程的集成。

- Spring 5.3：这个版本主要关注于改进开发人员的生产力和开发体验。它引入了更简洁的编程模型，提供了对Java 14和Java EE 9的支持。此外，Spring 5.3还增强了对响应式编程的支持，并提供了更好的集成测试支持。
  - 集成了Reactor 3.3：Spring 5.3集成了Reactor 3.3，这是一个用于响应式编程的流式编程库。通过与Reactor的集成，Spring 5.3提供了更强大的反应式编程能力和更好的性能。
  - 数据访问改进：Spring 5.3对数据访问模块进行了改进。它提供了对R2DBC（Reactive Relational Database Connectivity）的支持，这是一种针对关系型数据库的响应式编程模型。此外，它还增强了对JDBC和JPA的支持。
  - WebFlux改进：Spring 5.3对WebFlux进行了改进。它提供了更好的函数式Web请求处理的支持，并增加了对WebSocket的改进和扩展。此外，它还引入了对HTTP/2的支持。
  - 自动配置改进：Spring 5.3改进了自动配置功能。它提供了更好的条件化配置支持，通过条件注解和条件匹配器，可以更精确地控制配置的加载和生效条件。
  - Kotlin支持改进：Spring 5.3进一步改进了对Kotlin的支持。它提供了更好的Kotlin协程支持，并优化了与Kotlin协程的集成。
  - Spring Security改进：Spring 5.3改进了Spring Security模块。它引入了更多的安全特性和改进，包括对OAuth 2.0的改进和扩展，以及对WebFlux的安全性增强。

- Spring 5.4：针对反应式编程支持做了大幅提升,同时也提升了对Kotlin和验证等其他功能的支持。重点是简化Spring框架中的反应式编程开发。
  - 反应式网络应用支持得到增强。采用改进的WebFlux和非阻塞API支持反应式编程。包含了更好的反应式网络应用与容器集成。
  - 升级到Reactor Core 3,采用最新的反应式流支持库。带来多订阅发布者等新特性。
  - 对Kotlin支持进行了进一步改进,包括对空值的检查。
  - 支持Jackson 2.12,改进JSON内容处理能力。
  - 验证API升级到Jakarta Bean Validation 3。
  - 任务调度支持新增TaskSchedulerBuilder API,更方便配置任务调度。
  - 消息转换配置增加定制请求/响应转换的选项。
  - 核心容器更新,如DisposableBean接口变更和@Profile功能改进。
  - 测试框架针对测试反应式应用做了增强。
  - Docker支持得到优化,改进自动配置和Docker环境支持。

- Spring 5.5：重点改进了Kotlin、WebFlux以及任务执行和测试等方面的支持能力。同时针对Jakarta EE和Spring Boot等常见场景提供更友好的集成。
  - 对Kotlin的支持更加完善,例如属性委托支持和调用 conveniences等新特性。
  - 支持Jakarta EE 9规范,如Bean Validation 3.1和JSON Binding 1.0。
  - WebFlux升级到Reactor Core 3.4,支持聚合请求和顺序保证。
  - Spring Boot 2.4引入,提供了一体化的Spring开发体验。
  - 数据访问支持升级,包括JDBC 模板4.3和数据 MongoDB 4.2。
  - Configuration Processor支持高级功能如条件化配置。
  - 支持GRESS基准测试,可以进行基准比较和性能调优。
  - 鉴权服务支持OpenID Connect流程。
  - 响应式任务(Reactive Streams)调度重构,性能和易用性均有提升。
  - spring-shell提供了交互shell的支持。
  - 集成测试支持Mock Server,可以mock配置的HTTP交互。
  - 日志支持改进,例如custom logging system。

- Spring 5.6：针对最新Java版本、响应式编程模型、Kotlin和数据访问等领域做了升级,同时也支持Quarkus、Micronaut等新的云原生技术栈。
  - 支持Java 17,调整了对新的Java版本的支持。
  - WebFlux升级到Reactor Core 3.5,支持flow和hook了更多响应式操作。
  - Kotlin升级到1.6,优化对协程和更多Kotlin特性的支持。
  - 数据访问框架升级到最新版本,例如JDBC 5.4和Data MongoDB 4.3。
  - 注解处理器支持更复杂的编程模型和第三方注解。
  - JPA支持元模型检查和[@Id] generatedValue策略。
  - Webflux支持原生Servlet和Undertow。
  - 支持Quarkus和Micronaut框架。
  - 支持 GraalVM基于AOT的Spring应用部署。
  - 任务执行支持函数式编程和自定义异步执行策略。
  - 日志改进,支持SLF4J 1.8和Log4j 2.17。
  - 自动配置模型更强大,支持内嵌属性和容器值注入。

- Spring 5.7：持续跟进最新Java和响应式技术栈的发展,同时也优化Web、监控和测试等开发场景的支持能力。
  - 支持Java 18,继续跟进最新Java版本变化。
  - WebFlux升级到Reactor Core 3.6,支持更丰富的响应式功能。
  - Spring Boot 2.7,提供一站式Spring开发体验。
  - Kotlin升级到1.7版本,继续增强对Kotlin特性的支持。
  - 数据访问框架升级各个组件到最新版本。
  - Web开发支持WebClient/multipart文件上传。
  - 支持Quarkus 1.16/Micronaut 3作为云原生框架。
  - Spring Shell新增CommandLineRunner支持。
  - 监控支持Metrics和Prometheus exporter。
  - 日志完善记录堆栈跟踪信息。
  - 增强的类加载器支持和资源注入。
  - 响应式流支持各种调度器。
  - 自动配置支持属性同步动作。
  - 测试支持调用WebClient和WebTestClient。
- Spring 6.0：

# 4. Spring 模块化设计

- spring-aop：
- spring-aspects
- spring-beans
- spring-context-indexer
- spring-context-support
- spring-context
- spring-core
- spring-expression：Spring3 引入
- spring-instrument：Spring2 引入
- spring-jcl：Spring5 引入，统一日志管理
- spring-jdbc
- spring-jms：消息服务
- spring-messaging：消息服务的统一实现
- spring-orm
- spring-oxm：XML编列
- spring-test：测试
- spring-tx：事物抽象
- spring-web
- spring-webflux
- spring-webmvc
- spring-websocket

# 5. Spring 对 Java 语言特性用



![Java语法变化](http://image.geekidentity.com/Java%E8%AF%AD%E6%B3%95%E5%8F%98%E5%8C%96_1633429778738.png)

Java语法变化

- Java 5语法特性

| 语法特性                  | Spring支持版本 | 代表实现                   |
| ------------------------- | -------------- | -------------------------- |
| 注解（Annotation）        | 1.2+           | @Transactional             |
| 枚举（Enumeration）       | 1.2+           | Propagation                |
| for-each语法              | 3.0+           | AbstractApplicationContext |
| 自动装箱（AutoBoxing）    | 3.0+           |                            |
| 泛型（Generic）           | 3.0+           | ApplicationListener        |
| 静态导入（Static import） |                |                            |
| 可变参数（Varargs）       |                |                            |

- Java 6语法特性

| 语法特性      | Spring支持版本 | 代表实现 |
| ------------- | -------------- | -------- |
| 接口@Override | 4.0+           |          |

- Java 7语法特性

| 语法特性               | Spring支持版本 | 代表实现                    |
| ---------------------- | -------------- | --------------------------- |
| Diamond语法            | 5.0+           | DefaultListableBeanFactory  |
| try-with-resources语法 | 5.0+           | ResourceBundleMessageSource |
| 多 Catch               |                |                             |
| switch语句支持字符串   |                |                             |
| Fork/Join框架          |                |                             |

- Java 8语法特性

| 语法特性   | Spring支持版本 | 代表实现                       |
| ---------- | -------------- | ------------------------------ |
| Lambda语法 | 5.0+           | PropertiyEditorRegistrySupport |
| 可重复注解 |                |                                |
| 类型注解   |                |                                |

- Java 9语法特性

| 语法特性 | Spring支持版本 | 代表实现 |
| -------- | -------------- | -------- |
| 模块化   | 5.0+           |          |

# 6. Spring 对 JDK API 实践

- < Java 5 API

| API类型                   | Spring支持版本 | 代表实现                  |
| ------------------------- | -------------- | ------------------------- |
| 反射（Reflection）        | 1.0+           | MethodMatcher             |
| Java Beans                | 1.0+           | CacheIntrospectionResults |
| 动态代理（Dynamic Proxy） | 1.0+           | JdkDynamicAopProxy        |

- Java 5 API

| API类型                 | Spring支持版本 | 代表实现                       |
| ----------------------- | -------------- | ------------------------------ |
| XML 处理（DOM、SAX...） | 1.0+           | XmlBeanDefinitionReader        |
| Java 管理扩展（JMX）    | 1.2+           | @ManagedResource、@Transaction |
| Instrumentation         | 2.0+           | InstrumentationSavingAgent     |
| 并发框架（J.U.C）       | 3.0+           | ThreadPoolTaskScheduler        |
| 格式化（Fromatter）     | 3.0+           | DateFormatter                  |

- Java 6 API

| API类型                       | Spring支持版本 | 代表实现                                                     |
| ----------------------------- | -------------- | ------------------------------------------------------------ |
| JDBC4.0（JSR 221）            | 1.0+           | JdbcTemplate                                                 |
| Common Annotations（JSR 250） | 2.5+           | @Resource、@PostConstruct、CommonAnnotationBeanPostProcessor |
| JAXB 2.0（JSR 222）           | 3.0+           | Jaxb2Marshaller                                              |
| Scripting in JVM（JSR 223）   | 4.2+           | StandardScriptFactory                                        |
| 可插拔注解处理 API（JSR 269） | 5.0+           | @Indexed                                                     |
| Java Compiler API（JSR 199）  | 5.0+           | TestCompiler（单元测试）                                     |

- Java 7 API

| API类型                        | Spring支持版本 | 代表实现                |
| ------------------------------ | -------------- | ----------------------- |
| Fork/Join框架（JSR 166）       | 3.1+           | ForkJoinPoolFactoryBean |
| NIO 2（JSR 203）               | 4.0+           | PathResource            |
| invokedynamic字节码（JSR 292） | 无             |                         |

- Java 8 API

| API类型                             | Spring支持版本 | 代表实现                             |
| ----------------------------------- | -------------- | ------------------------------------ |
| Date and Time API（JSR 310）        | 4.0+           | DateTimeContext                      |
| 可重复Annotations（JSR 237）        | 4.0+           | @PropertySources                     |
| Stream API（JSR 335）               | 4.2+           | StreamConverter                      |
| CompletableFuture（J.U.C）          | 4.2+           | CompletableToListenableFutrueAdapter |
| Annotation on Java Types（JSR 308） | 无             |                                      |
| JavaScript 运行时（JSR 223）        | 无             |                                      |

- Java 9 API

| API类型                           | Spring支持版本 | 代表实现 |
| --------------------------------- | -------------- | -------- |
| Reactive Streams FlowAPI（J.U.C） | 无             |          |
| Process API Updates（JEP 102）    | 无             |          |
| Variable Handles（JEP 193）       | 无             |          |
| Method Handles（JEP 277）         | 无             |          |
| Spin-Wait Hints（JEP 285）        | 无             |          |
| Stack-Walking API（JEP 259）      | 无             |          |

# 7. Spring 对 Java EE API 整合

- Java EE Web技术相关

| JSR规范                     | Spring支持版本 | 代表实现                          |
| --------------------------- | -------------- | --------------------------------- |
| Servlet + JSP（JSR 035）    | 1.0+           | DispatherServlet                  |
| JSTL（JSR 052）             | 1.0+           | JstlView                          |
| JavaServer Faces（JSR 127） | 1.1+           | FacesContextUtils                 |
| Portlet（JSR 168）          | 2.0 - 4.2      | DispatherPortlet                  |
| SOAP（JSR 067）             | 2.5+           | SoapFaultException                |
| WebServices（JSR 109）      | 2.5+           | CommonAnnotationBeanPostProcessor |
| WebSocket（JSR 356）        | 4.0            | WebSocketHandler                  |

- Java EE 数据存储相关

| JSR规范                      | Spring支持版本 | 代表实现              |
| ---------------------------- | -------------- | --------------------- |
| JDO（JSR 12）                | 1.0 - 4.2      | JdoTemplate           |
| JTA（JSR 907）               | 1.0+           | JtaTransactionManager |
| JPA（EJB 3.0 JSR 220的成员） | 2.0            | JpaTransactionManager |
| Java Caching API（JSR 107）  | 3.2+           | JCacheCache           |

- Java EE Bean技术相关

| JSR规范                               | Spring支持版本 | 代表实现                             |
| ------------------------------------- | -------------- | ------------------------------------ |
| JMS（JSR 914）                        | 1.1+           | JmsTemplate                          |
| EJB2.0（JSR 19）                      | 1.0            | AbstractStatefulSessionBean          |
| Dependency Inject for Java（JSR 330） | 2.5            | AutowiredAnnotationBeanPostProcessor |
| Bean Validation（JSR 303）            | 3.0            | LocalValidationFactoryBean           |

# 8. Spring 编程模型

- 面向对象编程

  - 契约接口：Aware、BeanPostProcessor ...

  - 设计模式：观察者模式、组合模式、模板模式 ...

  - 对象继承：Abstract* 类

- 面向切面编程

  - 动态代理：JdkDynamicAopProxy

  - 字节码提升：ASM、CGLib、AspectJ ...

- 面向元编程

  - 注解：模式注解（@Component、@Service、@Respository）

  - 配置：Environment抽象、PropertySources、BeanDefinition ...

  - 泛型：GenericTypeResolver、ResolvableType ...

- 函数驱动

  - 函数接口：ApplicationEventPublisher

  - Reactive: Spring WebFlux

- 模块驱动

  - Maven Artifacts

  - OSGI Bundles

  - Java9 Automatic Modules

  - Spring @Enable*

# 9. Spring 核心价值

## 生态系统

1. Spring Boot
2. Spring Cloud
3. Spring Security
4. Spring Data
5. 其他

## API抽象设计

1. AOP抽象
2. 事务抽象
3. Environment抽象
4. 生命周期

## 编程模型

内容同上一节

## 设计思想

1. Object Oriented Programming(OOP)
2. IoC/DI
3. Domain-Driven Development(DDD)
4. Test-Driven Development(TDD)
5. Event-Driven Programming(EDP)
6. Functional Programming(FP)

## 设计模式

- 专属模式
  - 前缀模式
    - Enable模式
    - Configurable模式
  - 后缀模式
    - 处理器模式
      - Processor
      - Resolver
      - Handler
    - 意识模式
      - Aware
    - 配置模式
      - Configuror
    - 选择器模式
      - ImportSelector

- 传统 GoF 23

## 用户基础

1. Spring Framework
2. Spring Boot
3. Spring Cloud

## 传统用户

1. Java SE
2. Java EE

