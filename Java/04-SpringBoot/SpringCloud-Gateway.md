## 1. 概述

### 1.1 什么是 Spring Cloud Gateway

Spring Cloud Gateway 是一个基于 Spring Framework 5、Spring Boot 2 和 Project Reactor 等开源技术栈构建的网关服务，用于处理 API 请求的路由、流量控制、安全性、监控等方面的需求。Spring Cloud Gateway 的设计目标是提供一个简单、高效、可扩展、易于使用的 API 网关解决方案。

Spring Cloud Gateway 主要特点包括：

- 基于 Spring Framework 5 和 Reactor 的非阻塞式设计，具有高性能和低延迟的优势。
- 支持多种路由策略，包括 URI 路由、断言路由等方式，能够满足不同场景的需求。
- 支持集成多种服务发现和负载均衡组件，如 Eureka、Consul、Zookeeper 等。
- 提供可扩展的过滤器机制，能够对请求和响应进行处理，实现流量控制、安全性、监控等需求。
- 支持熔断器、限流等高级特性，能够提高系统的可用性和稳定性。
- 具有灵活的部署和管理选项，支持容器化部署、Docker 镜像等方式。



### 1.2 Spring Cloud Gateway 的优势

Spring Cloud Gateway 相比其他 API 网关解决方案具有以下优势：

1. 基于 Spring Framework 5 和 Reactor 的非阻塞式设计，具有高性能和低延迟的优势。
2. 支持多种路由策略，包括 URI 路由、断言路由等方式，能够满足不同场景的需求。
3. 支持集成多种服务发现和负载均衡组件，如 Eureka、Consul、Zookeeper 等。
4. 提供可扩展的过滤器机制，能够对请求和响应进行处理，实现流量控制、安全性、监控等需求。
5. 支持熔断器、限流等高级特性，能够提高系统的可用性和稳定性。
6. 具有灵活的部署和管理选项，支持容器化部署、Docker 镜像等方式。

除此之外，Spring Cloud Gateway 还具有以下优势：

- 与 Spring Cloud 微服务架构紧密集成，能够更好地满足微服务架构的需求。
- 提供了一套完整的 API 管理和监控功能，包括路由信息、流量监控、错误日志等，可帮助开发者更好地管理和维护 API 网关。
- 基于开源技术栈构建，社区活跃，文档丰富，生态系统完整，能够更好地满足开发者的需求。



### 1.3 Spring Cloud Gateway 与 Zuul 的区别

Spring Cloud Gateway 和 Zuul 都是 Spring Cloud 微服务架构中的 API 网关组件，它们的主要区别如下：

1. 基于不同的技术栈

Spring Cloud Gateway 基于 Spring Framework 5 和 Reactor 的非阻塞式设计，具有高性能和低延迟的优势，支持多种路由策略和可扩展的过滤器机制，能够满足不同场景的需求。

Zuul 基于 Netflix 开源技术栈构建，包括 Ribbon、Hystrix、Eureka 等组件，提供了路由、负载均衡、熔断器等功能，适合于构建复杂的微服务架构。

2) 路由策略的不同

Spring Cloud Gateway 支持多种路由策略，包括 URI 路由、断言路由等方式，能够满足不同场景的需求。

Zuul 使用基于 Servlet 的路由策略，需要将请求转发到 Zuul Servlet 上进行处理，然后再根据路由规则进行转发。

3) 过滤器机制的不同

Spring Cloud Gateway 提供可扩展的过滤器机制，能够对请求和响应进行处理，实现流量控制、安全性、监控等需求。

Zuul 提供了一套基于 Groovy 的过滤器机制，能够实现类似于 Spring MVC 的拦截器功能。

4) 功能的不同

Spring Cloud Gateway 提供了一套完整的 API 管理和监控功能，包括路由信息、流量监控、错误日志等，可帮助开发者更好地管理和维护 API 网关。

Zuul 则提供了一些高级的功能，如动态路由、自定义路由、熔断器等。



## 2. 快速入门

### 2.1 环境配置

要使用 Spring Cloud Gateway，需要进行以下环境配置：

1. 安装 JDK

首先需要安装 Java Development Kit（JDK），Spring Cloud Gateway 需要运行在 Java 虚拟机（JVM）上。可以从 Oracle 官网或 OpenJDK 官网下载并安装 JDK。

2) 安装 Maven

Spring Cloud Gateway 使用 Maven 进行项目构建和依赖管理，因此需要安装 Maven。可以从 Maven 官网下载并安装 Maven，也可以使用系统包管理器进行安装。



### 2.2 创建Spring Cloud Gateway项目

接下来需要创建一个 Spring Cloud Gateway 项目。可以使用 Spring Initializr 在线工具创建一个新的 Spring Boot 项目。

在 `pom.xml` 文件中添加 Spring Cloud Gateway 依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

### 2.3 配置路由规则

在项目的配置文件中，可以使用 `spring.cloud.gateway.routes` 属性来配置路由规则。例如：

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: example
          uri: http://example.com
          predicates:
            - Path=/example/**
```

这个配置表示将所有以 `/example` 开头的请求路由到 `http://example.com`。

在 Spring Cloud Gateway 中，还可以使用代码配置路由，可以更加灵活地定义路由规则。以下是一个示例代码：

```java
import org.springframework.cloud.gateway.route.RouteLocator;
import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class GatewayConfig {

    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
                .route("example", r -> r.path("/example/**")
                        .filters(f -> f.addRequestHeader("X-Example-Header", "example-value"))
                        .uri("http://example.com"))
                .build();
    }

}
```

在这个示例中，使用 `@Configuration` 注解将配置类声明为 Spring Bean，并使用 `@Bean` 注解定义一个名为 `customRouteLocator` 的方法，返回一个 `RouteLocator` 对象。

在 `customRouteLocator` 方法中，使用 `RouteLocatorBuilder` 构建器定义路由规则。使用 `route` 方法定义一个名为 `example` 的路由规则，使用 `path` 方法指定路由的匹配规则。使用 `filters` 方法添加过滤器，使用 `addRequestHeader` 方法添加一个名为 `X-Example-Header` 的请求头部，值为 `example-value`。使用 `uri` 方法指定路由的目标地址。

最后，通过使用 `build` 方法构建一个 `RouteLocator` 对象，并返回。

需要注意的是，在使用代码配置路由时，可以使用 `RouteLocatorBuilder` 构建器来定义路由规则，也可以使用 `RouteDefinitionLocator` 接口来加载外部的路由定义。例如：

```java
@Bean
public RouteLocator customRouteLocator(RouteDefinitionLocator locator) {
    return new RouteDefinitionRouteLocator(locator);
}
```

在这个示例中，使用 `RouteDefinitionLocator` 接口来加载外部的路由定义，然后使用 `RouteDefinitionRouteLocator` 类将路由定义转换成 `RouteLocator` 对象。这种方式可以通过读取外部文件或数据库来动态加载路由定义。

### 2.4 添加过滤器

在 Spring Cloud Gateway 中，可以使用过滤器来对请求进行处理。可以使用 `spring.cloud.gateway.routes.filters` 属性来添加过滤器。例如：

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: example
          uri: http://example.com
          predicates:
            - Path=/example/**
          filters:
            - AddRequestHeader=X-Example-Header, example-value
```

这个配置表示在路由到 `http://example.com` 之前，添加一个名为 `X-Example-Header` 的请求头部，值为 `example-value`。

### 2.5 运行和测试

配置完成后，可以运行项目，并测试路由和过滤器是否生效。如果一切正常，应该能够看到请求被正确路由到目标地址，并且请求头部中包含了添加的自定义请求头部。



## 3. 路由配置

### 3.1 URI路由

URI 路由是指将请求的 URI 路径映射到目标网址的过程。在 Spring Cloud Gateway 中，可以使用 URI 路由来实现简单的请求路由，例如将 /api/user 路径的请求转发到 http://user-service/api/user 上。

URI 路由的配置可以使用 YAML 或者 Java 代码进行配置。以下是一个基本的 YAML 配置文件示例：

```
spring:
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: http://user-service
          predicates:
            - Path=/api/user/**
```

在上面的配置中，配置了一个名为 user-service 的路由规则，将 /api/user/** 路径的请求转发到 [http://user-service](http://user-service/) 上。其中，id 属性指定了路由规则的 ID，uri 属性指定了目标服务的地址，predicates 属性指定了路由规则的匹配条件，这里使用了 Path 匹配条件。

### 3.2 配置断言工厂和过滤器工厂

配置断言和过滤器有两种方法：快捷方式和完全扩展的参数。下面的大多数例子都使用快捷方式。

名称和参数名称在每个部分的第一句或第二句中列为 `code` 。这些参数通常按照快捷方式配置所需的顺序列出。

#### 3.2.1 快捷方式配置

快捷方式配置由过滤器名称识别，后跟等号 ( `=` )，后跟用逗号分隔的参数值 ( `,` )。

application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - Cookie=mycookie,mycookievalue
```

前面的示例使用两个参数定义 `Cookie` 路由谓词工厂：cookie 名称 `mycookie` 和匹配 `mycookievalue` 的值。

#### 3.2.2 完全扩展的参数

完全扩展的参数看起来更像是带有名称/值对的标准 yaml 配置。通常，会有一个 `name` 键和一个 `args` 键。 `args` 键是用于配置谓词或过滤器的键值对的映射。

application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: after_route
        uri: https://example.org
        predicates:
        - name: Cookie
          args:
            name: mycookie
            regexp: mycookievalue
```

这是上面显示的 `Cookie` 谓词的快捷方式配置的完整配置。

### 3.3 路由断言工厂

Spring Cloud Gateway 的断言路由是指根据请求的属性（例如请求头、请求参数、路径等）来匹配路由规则，进而将请求路由到目标网址的过程。在 Spring Cloud Gateway 中，可以使用断言路由来实现更复杂的请求路由，例如根据请求头的值、请求参数的值等匹配路由规则。

#### 3.3.1 内置断言工厂

Spring Cloud Gateway 提供了多种内置的断言工厂，用于实现常见的路由匹配条件。这些断言工厂可以直接在路由规则中使用，也可以基于它们进行二次开发和定制。

官方文档：https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gateway-request-predicates-factories

以下是一些常用的内置断言工厂：

- After：判断请求时间是否在指定时间之后。

- Before：判断请求时间是否在指定时间之前。

- Between：判断请求时间是否在指定时间段之内。

- Cookie：判断请求中是否包含指定名称的 Cookie。

- Header：判断请求头中是否包含指定名称和值的请求头。

- Host：判断请求的 Host 是否匹配指定的 Host。

- Method：判断请求的方法是否匹配指定的方法。

- Path：判断请求的路径是否匹配指定的路径。

- Query：判断请求的查询参数是否匹配指定的参数名和值。

- ReadBody：根据请求体的内容进行路由匹配。

- RemoteAddr：判断请求的远程地址是否匹配指定的地址段。

  ```yaml
  spring:
    cloud:
      gateway:
        routes:
        - id: remoteaddr_route
          uri: https://example.org
          predicates:
          - RemoteAddr=192.168.1.1/24
  ```

  如果请求的远程地址是 `192.168.1.10` ，则此路由匹配。

- XForwardedRemoteAddr：使用 X-Forwarded-For 头部信息来获取客户端的真实 IP 地址。

  ```yml
  spring:
    cloud:
      gateway:
        routes:
        - id: xforwarded_remoteaddr_route
          uri: https://example.org
          predicates:
          - XForwardedRemoteAddr=192.168.1.1/24
  ```

  如果 `X-Forwarded-For` 标头包含 `192.168.1.10` ，则此路由匹配。

- Weight：基于服务实例的权重进行路由选择。

  ```yaml
  spring:
    cloud:
      gateway:
        routes:
        - id: weight_high
          uri: https://weighthigh.org
          predicates:
          - Weight=group1, 8
        - id: weight_low
          uri: https://weightlow.org
          predicates:
          - Weight=group1, 2
  ```

  该路由会将约 80% 的流量转发到 Weighthigh.org，将约 20% 的流量转发到 Weightlow.org

- CloudFoundryRouteService：判断请求是否来自 Cloud Foundry Route Service。

#### 3.3.2 自定义断言工厂

Spring Cloud Gateway 提供了自定义断言工厂的功能，可以根据具体的业务需求编写自定义的路由匹配条件。自定义断言工厂需要实现 RoutePredicateFactory 接口，并重写两个方法：

- shortcutFieldOrder：指定该断言工厂支持的参数名称及其顺序。
- apply：根据参数值创建断言对象，用于进行路由匹配。

以下是一个自定义断言工厂的示例：

```java
@Component
public class CustomRoutePredicateFactory implements RoutePredicateFactory<CustomRoutePredicateFactory.Config> {

    @Override
    public List<String> shortcutFieldOrder() {
        return Arrays.asList("paramName", "expectedValue");
    }

    @Override
    public Predicate<ServerWebExchange> apply(Config config) {
        return exchange -> {
            String paramValue = exchange.getRequest().getQueryParams().getFirst(config.getParamName());
            return paramValue != null && paramValue.equals(config.getExpectedValue());
        };
    }
  
		@Data
    public static class Config {
        private String paramName;
        private String expectedValue;
    }
}
```

在上面的示例中，定义了一个名为 CustomRoutePredicateFactory 的自定义断言工厂，并实现了 RoutePredicateFactory 接口。Config 类用于存储断言工厂的参数值，其中 paramName 用于指定要匹配的查询参数名，expectedValue 用于指定要匹配的查询参数值。shortcutFieldOrder 方法指定了该断言工厂支持的参数名称及其顺序。apply 方法根据传入的参数值创建出一个 Predicate 对象，用于进行路由匹配。

使用自定义断言工厂时，可以在路由规则中使用对应的参数名和值，例如：

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: custom-route
          uri: http://localhost:8080
          predicates:
            - Custom=foo,bar
```

在上面的示例中，配置了一个名为 custom-route 的路由规则，将匹配请求中的 paramName 参数值为 foo，且其对应的值为 bar 的请求转发到 [http://localhost:8080](http://localhost:8080/) 上。其中，Custom 是自定义断言工厂的名称，paramName 和 expectedValue 是自定义断言工厂支持的参数名。

### 3.4 GatewayFilter 工厂

Spring Cloud Gateway 的过滤器是指在请求被路由到目标服务之前或之后，对请求和响应进行处理的组件。通过过滤器可以实现请求的验证、修改、转发、重定向、缓存等功能。

Spring Cloud Gateway 的过滤器分为两种类型：局部过滤器和全局过滤器。局部过滤器只对某个路由规则生效，而全局过滤器对所有路由规则生效。过滤器可以使用 GatewayFilterFactory 或者 GlobalFilter 进行配置。

官方文档：https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gatewayfilter-factories

#### 3.4.1 内置过滤器工厂

Spring Cloud Gateway 提供了多种内置的过滤器工厂，用于实现常见的请求预处理、响应处理等功能。这些过滤器工厂可以直接在路由规则中使用，也可以基于它们进行二次开发和定制。

以下是一些常用的内置过滤器工厂：

- AddRequestHeaderGatewayFilterFactory

- AddRequestParameterGatewayFilterFactory

- CacheRequestBodyGatewayFilterFactory

- DedupeResponseHeaderGatewayFilterFactory

- FallbackHeadersGatewayFilterFactory

- JsonToGrpcGatewayFilterFactory

- MapRequestHeaderGatewayFilterFactory

- PrefixPathGatewayFilterFactory

- PreserveHostHeaderGatewayFilterFactory

- RedirectToGatewayFilterFactory

- RemoveRequestHeaderGatewayFilterFactory

- RemoveRequestParameterGatewayFilterFactory

- RemoveResponseHeaderGatewayFilterFactory

- RequestHeaderSizeGatewayFilterFactory

- RequestHeaderToRequestUriGatewayFilterFactory

- RequestSizeGatewayFilterFactory

- RequestRateLimiterGatewayFilterFactory

- RetryGatewayFilterFactory

- RewriteLocationResponseHeaderGatewayFilterFactory

- RewriteResponseHeaderGatewayFilterFactory

- SaveSessionGatewayFilterFactory

- SetPathGatewayFilterFactory

- SetRequestHeaderGatewayFilterFactory

- SetRequestHostHeaderGatewayFilterFactory

- SetStatusGatewayFilterFactory

- SpringCloudCircuitBreakerResilience4JFilterFactory

- StripPrefixGatewayFilterFactory

- TokenRelayGatewayFilterFactory

- ModifyRequestBodyGatewayFilterFactory

- ModifyResponseBodyGatewayFilterFactory

  

举例：

```yaml
# 其他的内置过滤器
- AddRequestHeader=X-Request-color, red # 添加请求头参数
# - AddRequestParameter=color, blue # 添加请求参数
# - PrefixPath=/mall‐order # 添加前缀 对应微服务需要配置context‐path: /mall‐order
# - RedirectTo=302, https://www.baidu.com/ # 重定向到百度
```

#### 3.4.2 自定义过滤器工厂

Spring Cloud Gateway 提供了自定义过滤器工厂的功能，可以根据具体的业务需求编写自定义的过滤器。自定义过滤器工厂需要实现 GatewayFilterFactory 接口，并重写两个方法：

- shortcutFieldOrder：指定该过滤器工厂支持的参数名称及其顺序。
- apply：根据参数值创建过滤器对象，用于实现过滤逻辑。

以下是一个自定义过滤器工厂的示例：

```java
@Component
public class CustomGatewayFilterFactory implements GatewayFilterFactory<CustomGatewayFilterFactory.Config> {

    @Override
    public GatewayFilter apply(Config config) {
        return (exchange, chain) -> {
            ServerHttpRequest request = exchange.getRequest();
            HttpHeaders headers = request.getHeaders();
            headers.add("X-Custom-Header", config.getMessage());
            return chain.filter(exchange);
        };
    }

    @Override
    public List<String> shortcutFieldOrder() {
        return Collections.singletonList("message");
    }

  	@Data
    public static class Config {
        private String message;
    }
}
```

在上面的示例中，定义了一个名为 CustomGatewayFilterFactory 的自定义过滤器工厂，并实现了 GatewayFilterFactory 接口。Config 类用于存储过滤器工厂的参数值，其中 message 用于指定要添加的自定义头部信息的值。shortcutFieldOrder 方法指定了该过滤器工厂支持的参数名称及其顺序。apply 方法根据传入的参数值创建出一个 GatewayFilter 对象，用于实现过滤逻辑。

使用自定义过滤器工厂时，可以在路由规则中使用对应的参数名和值，例如：

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: custom-route
          uri: http://localhost:8080
          filters:
            - CustomFilter=message:Hello World
```

在上面的示例中，配置了一个名为 custom-route 的路由规则，将请求转发到 [http://localhost:8080](http://localhost:8080/) 上，并添加一个名为 X-Custom-Header，值为 Hello World 的自定义头部信息。其中，CustomFilter 是自定义过滤器工厂的名称，message 是自定义过滤器工厂支持的参数名。

### 3.5 全局过滤器

Spring Cloud Gateway 提供了全局过滤器的功能，可以在所有路由规则之前或之后统一处理请求和响应。全局过滤器可以用于添加公共的头部信息、记录请求日志、进行鉴权等操作。

要创建一个全局过滤器，只需要实现 GlobalFilter 接口，并重写 filter 方法即可。`GlobalFilter` 接口与 `GatewayFilter` 具有相同的签名。这些是有条件地应用于所有路由的特殊过滤器。



以下是一个简单的全局过滤器的示例：

```java
@Component
public class CustomGlobalFilter implements GlobalFilter {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        HttpHeaders headers = request.getHeaders();
        headers.add("X-Custom-Header", "Global Filter");
        return chain.filter(exchange);
    }
}
```

在上面的示例中，定义了一个名为 CustomGlobalFilter 的全局过滤器，并实现了 GlobalFilter 接口。filter 方法根据传入的 ServerWebExchange 对象和 GatewayFilterChain 对象，对请求进行处理，并将请求转发到下一个过滤器或者路由规则。

以下是Spring Cloud Gateway 内置的全局过滤器：

- AdaptCachedBodyGlobalFilter：适配缓存请求体过滤器，用于适配缓存请求体的过滤器。

- NettyWriteResponseFilter：Netty 写响应过滤器，用于将响应写回客户端。

- ForwardPathFilter：请求转发过滤器，用于将请求转发到后端服务。

- ForwardRoutingFilter：用于将请求转发到后端服务的实例，并处理一些转发相关的逻辑。

- GatewayMetricsFilter：度量过滤器工厂，用于收集请求和响应的度量信息。

- NoLoadBalancerClientFilter

- LoadBalancerServiceInstanceCookieFilter：用于将选择的负载均衡实例信息保存到 Cookie 中，以便在后续的请求中使用。这个过滤器的作用是为了在多个请求之间保持会话的一致性，从而提高负载均衡的效果。

- NettyRoutingFilter：使用 Netty 客户端将请求转发到后端服务的实例。它是 Spring Cloud Gateway 中的核心过滤器之一，负责将请求路由到后端服务，并且支持负载均衡、重试、限流等功能。该过滤器使用 Netty 的异步非阻塞网络通信框架，提高了网关的性能和吞吐量，并支持HTTP/2协议。

- NettyWriteResponseFilter：Netty 写响应过滤器，用于将响应写回客户端。

- ReactiveLoadBalancerClientFilter：负载均衡过滤器，用于选择后端服务的实例

- RemoveCachedBodyFilter：用于移除请求体缓存。

- RouteToRequestUrlFilter：路由请求 URL 过滤器，用于将请求路由到后端服务的实例。

- WebClientHttpRoutingFilter：使用 WebClient 将请求转发到后端服务的实例。与 NettyRoutingFilter 相比，WebClientHttpRoutingFilter 使用 WebClient 进行网络通信，支持更多的协议和数据格式，并且支持响应式编程模型，提供了更好的性能和扩展性。该过滤器还支持负载均衡、重试、限流等功能。同时，WebClientHttpRoutingFilter 也可以配置连接池和超时时间等参数，以满足不同的业务需求。

- WebClientWriteResponseFilter：用于将响应写回客户端。与 NettyWriteResponseFilter 相比，WebClientWriteResponseFilter 使用 WebClient 进行响应的写入操作，支持更多的协议和数据格式，并且支持响应式编程模型，提供了更好的性能和扩展性。该过滤器还支持设置响应头、响应状态码等操作，以满足不同的业务需求。

- WebsocketRoutingFilter：用于将 WebSocket 请求转发到后端服务的实例。WebSocket 是一种双向通信协议，可以在客户端和服务器之间建立持久化连接，实现实时通信。该过滤器会检查请求是否为 WebSocket 请求，如果是，则将请求路由到后端服务的实例，并建立 WebSocket 连接。该过滤器还支持负载均衡和重试等功能，以提高网关的性能和可靠性。

   

### 3.6 网关指标过滤器

要启用网关指标，请将 `spring-boot-starter-actuator` 添加为项目依赖项。然后，默认情况下，只要 `spring.cloud.gateway.metrics.enabled` 属性未设置为 `false` ，网关指标过滤器就会运行。此过滤器添加一个名为 `spring.cloud.gateway.requests` 的计时器指标，并带有以下标签：

- `routeId`: 路线ID。
- `routeUri`: API 路由到的 URI。
- `outcome`: 结果，按 HttpStatus.Series 分类。
- `status`: 返回给客户端的请求的 HTTP 状态。
- `httpStatusCode`: 返回给客户端的请求的 HTTP 状态。
- `httpMethod`: 用于请求的 HTTP 方法。

此外，通过 `spring.cloud.gateway.metrics.tags.path.enabled` 属性（默认情况下为 `false` ），您可以使用路径标记激活额外的指标：

- `path`: 请求的路径。

然后可以从 `/actuator/metrics/spring.cloud.gateway.requests` 中抓取这些指标，并且可以轻松地与 Prometheus 集成以创建 Grafana 仪表板。

> 要启用 prometheus 端点，请将 `micrometer-registry-prometheus` 添加为项目依赖项。

### 3.7 本地响应缓存过滤器

如果启用了关联属性，则 `LocalResponseCache` 运行：

- `spring.cloud.gateway.global-filter.local-response-cache.enabled`:激活所有路由的全局缓存
- `spring.cloud.gateway.filter.local-response-cache.enabled`: 激活关联的过滤器以在路由级别使用

此功能为满足以下条件的所有响应启用使用 Caffeine 的本地缓存：

- 该请求是无实体的 GET。
- 响应具有以下状态代码之一：HTTP 200（正常）、HTTP 206（部分内容）或 HTTP 301（永久移动）。
- HTTP `Cache-Control` 标头允许缓存（这意味着它不具有以下任何值：请求中存在 `no-store` 以及 `no-store` 或 `private`

它接受两个配置参数：

- `spring.cloud.gateway.filter.local-response-cache.size`: 设置缓存的最大大小以逐出该路由的条目（以 KB、MB 和 GB 为单位）。
- `spring.cloud.gateway.filter.local-response-cache.time-to-live` 设置缓存条目的过期时间（以 s 表示秒、m 表示分钟、h 表示小时）表示。

如果未配置这些参数，但启用了全局过滤器，则默认情况下，它会为缓存的响应配置 5 分钟的生存时间。

此过滤器还实现了 HTTP `Cache-Control` 标头中 `max-age` 值的自动计算。如果原始响应中存在 `max-age` ，则会使用 `timeToLive` 配置参数中设置的秒数重写该值。在后续调用中，将使用响应到期之前剩余的秒数重新计算该值。

将 `spring.cloud.gateway.global-filter.local-response-cache.enabled` 设置为 `false` 会停用所有路由的本地响应缓存，LocalResponseCache 过滤器允许在路由级别使用此功能。

> 要启用此功能，请添加 `com.github.ben-manes.caffeine:caffeine` 和 `spring-boot-starter-cache` 作为项目依赖项。
>
> 如果您的项目创建自定义 `CacheManager` bean，则需要用 `@Primary` 标记或使用 `@Qualifier` 注入。

## 4. 高级特性

### 4.1 服务注册和发现

#### 4.1.1 集成Nacos

Nacos 是阿里巴巴开源的一款服务注册和配置中心。它提供了服务注册、服务发现、配置管理等功能，可以方便地管理微服务应用的配置和服务注册。

在 Spring Cloud Gateway 中集成 Nacos，可以实现自动化的服务注册和发现，以及动态的路由配置。以下是集成步骤：

1. 添加依赖

在 pom.xml 文件中添加以下依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-nacos-discovery</artifactId>
</dependency>
```

其中，`spring-cloud-starter-gateway` 是 Spring Cloud Gateway 的核心依赖，`spring-cloud-alibaba-nacos-discovery` 是 Nacos 的服务注册和发现客户端依赖。

2) 配置 Nacos 服务注册和发现

在 application.yml 中添加以下配置：

```yaml
spring:
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true # 启用基于服务注册和发现的路由
          lower-case-service-id: true # 忽略服务名的大小写
  nacos:
    discovery:
      server-addr: ${nacos.server-addr:localhost:8848} # Nacos 服务地址
      register-enabled: true # 启用服务注册
      namespace: ${nacos.namespace:} # Nacos 命名空间
```

在以上配置中，`spring.cloud.gateway.discovery.locator.enabled` 属性启用基于服务注册和发现的路由。`spring.cloud.gateway.discovery.locator.lower-case-service-id` 属性忽略服务名的大小写，可以避免由于服务名大小写不一致导致的路由失败问题。`nacos.discovery.server-addr` 属性指定了 Nacos 服务地址。`nacos.discovery.register-enabled` 属性启用服务注册。`nacos.discovery.namespace` 属性指定了 Nacos 命名空间。

3) 配置路由规则

在 application.yml 中添加以下配置，定义路由规则：

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: service-a # 路由 ID，可以任意指定
          uri: lb://service-a # 目标服务名，可以是服务名或者 LoadBalancerClient bean 名称
          predicates:
            - Path=/service-a/** # 路由匹配规则，匹配 /service-a/** 的请求
        - id: service-b
          uri: lb://service-b
          predicates:
            - Path=/service-b/**
```

在以上配置中，`spring.cloud.gateway.routes` 属性定义了路由规则，可以定义多个路由规则。每个路由规则包括 `id`、`uri` 和 `predicates` 三个属性。`id` 属性是路由 ID，可以任意指定。`uri` 属性是目标服务名，可以是服务名或者 LoadBalancerClient bean 名称。`predicates` 属性是路由匹配规则，可以根据请求路径、请求头等属性进行匹配。

4) 启动应用程序

配置完成后，启动应用程序即可。Spring Cloud Gateway 将自动注册到 Nacos 服务中心，并根据路由规则将请求转发到指定的服务中。如果服务发生变化，例如服务实例

### 4.2 熔断器

#### 4.2.1 集成 Sentinel

Sentinel 是阿里巴巴开源的一款流量控制和熔断降级框架，可以实现实时监控和限流、熔断降级等功能，有效地保护系统的稳定性和可靠性。在微服务架构中，Sentinel 可以作为服务网关或者应用程序内部的流量控制和熔断降级组件。

在 Spring Cloud 中集成 Sentinel，可以实现对微服务架构中的流量进行实时监控和控制，以及处理异常情况，提高服务的可靠性和稳定性。以下是集成步骤：

1. 添加依赖

在 pom.xml 文件中添加以下依赖：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

其中，`spring-cloud-starter-alibaba-sentinel` 是 Sentinel 的 Spring Cloud Starter 包。

2) 配置 Sentinel

在 application.yml 中添加以下配置：

```yaml
spring:
  cloud:
    sentinel:
      transport:
        dashboard: localhost:8858 # Sentinel 控制台地址
        port: 8719 # Sentinel 客户端监听端口
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

在以上配置中，`spring.cloud.sentinel.transport.dashboard` 属性指定了 Sentinel 控制台地址。`spring.cloud.sentinel.transport.port` 属性指定了 Sentinel 客户端监听端口。`management.endpoints.web.exposure.include` 属性开启了所有 Actuator 端点的暴露，方便在 Sentinel 控制台中监控应用程序的运行状态、性能指标等。

3) 配置 Sentinel 规则

在应用程序中添加 Sentinel 规则，例如流量控制规则、熔断降级规则等。可以使用注解或者 API 的方式定义规则。以下是一个使用注解定义流量控制规则的示例：

```java
@RestController
public class DemoController {

    @GetMapping("/hello")
    @SentinelResource(value = "hello", blockHandler = "handleHelloBlock")
    public String hello() {
        return "Hello, World!";
    }

    public String handleHelloBlock(BlockException ex) {
        return "Blocked by Sentinel: " + ex.getMessage();
    }
}
```

在以上代码中，`@SentinelResource` 注解定义了一个 Sentinel 资源，名称为 `hello`，同时指定了 `blockHandler` 属性，表示当资源被流量控制限制时，调用 `handleHelloBlock` 方法进行处理。

通过配置文件完成

```yaml
server:
  port: 8088

spring:
  application:
    name: api-gateway
  cloud:
    # 整合sentinel
    sentinel:
      transport:
        # 添加sentinel控制台
        dashboard: 127.0.0.1:8858
      # 自定义异常
      scg:
        fallback:
          mode: response
          response-body: '{"code":"403", "msg":"限流了"}'
```

通过 GatewayCallbackManager 代码实现：

```java
@Configuration
public class GatewaySentinelConfig {

    @PostConstruct
    public void init() {
        BlockRequestHandler blockRequestHandler = (serverWebExchange, throwable) -> {
            // 自定义异常处理
            Map<String, String> result = new HashMap<>();
            if (throwable instanceof FlowException) {
                // 限流

            } else if (throwable instanceof DegradeException) {
                // 降级

            } 
            result.put("code", String.valueOf(HttpStatus.TOO_MANY_REQUESTS.value()));
            result.put("message", HttpStatus.TOO_MANY_REQUESTS.getReasonPhrase());

            return ServerResponse.status(HttpStatus.OK)
                    .contentType(MediaType.APPLICATION_JSON)
                    .body(BodyInserters.fromValue(result));
        };
    }
}
```

代码实现限流降级操作。**用户可以通过 GatewayRuleManager.loadRules(rules) 手动加载网关规则 GatewayConfiguration中添加:**

```java
@Configuration
public class GatewaySentinelConfig {
    @PostConstruct
    public void init() {
        // 自定义异常信息
        initBlockRequestHandler();

        // 自定义网关限流规则
        initGatewayRules();
    }

    private void initBlockRequestHandler() {
        BlockRequestHandler blockRequestHandler = (serverWebExchange, throwable) -> {
            // 自定义异常处理
            Map<String, String> result = new HashMap<>();

            if (throwable instanceof FlowException) {
                // 限流
            } else if (throwable instanceof DegradeException) {
                // 降级
            } 
            result.put("code", String.valueOf(HttpStatus.TOO_MANY_REQUESTS.value()));
            result.put("message", HttpStatus.TOO_MANY_REQUESTS.getReasonPhrase());

            return ServerResponse.status(HttpStatus.OK)
                    .contentType(MediaType.APPLICATION_JSON)
                    .body(BodyInserters.fromValue(result));
        };
    }

    private void initGatewayRules() {
        Set<GatewayFlowRule> rules = new HashSet<>();
        //resource：资源名称，可以是网关中的 route 名称或者用户自定义的 API 分组名称。
        //count：限流阈值
        //intervalSec：统计时间窗口，单位是秒，默认是 1 秒。
        rules.add(new GatewayFlowRule("order_route").setCount(2).setIntervalSec(1));
        rules.add(new GatewayFlowRule("user_server_api").setCount(2).setIntervalSec(1));

        // 加载网关规则
        GatewayRuleManager.loadRules(rules);
    }
}
```

其中网关限流规则 GatewayFlowRule 的字段解释如下：

| 字段                      | 解释                                                         |
| ------------------------- | ------------------------------------------------------------ |
| resource                  | 资源名称，可以是网关中的 route 名称或者用户自定义的 API 分组名称。 |
| resourceMode              | 规则是针对 API Gateway 的 route（`RESOURCE_MODE_ROUTE_ID`）还是用户在 Sentinel 中定义的 API 分组（`RESOURCE_MODE_CUSTOM_API_NAME`），默认是 route。 |
| grade                     | 限流指标维度，同限流规则的 `grade` 字段。                    |
| count                     | 限流阈值                                                     |
| intervalSec               | 统计时间窗口，单位是秒，默认是 1 秒。                        |
| controlBehavior           | 流量整形的控制效果，同限流规则的 controlBehavior 字段，目前支持快速失败和匀速排队两种模式，默认是快速失败。 |
| burst                     | 应对突发请求时额外允许的请求数目。                           |
| maxQueueingTimeoutMs      | 匀速排队模式下的最长排队时间，单位是毫秒，仅在匀速排队模式下生效。 |
| paramItem                 | 参数限流配置。若不提供，则代表不针对参数进行限流，该网关规则将会被转换成普通流控规则；否则会转换成热点规则。 |
| paramItem - parseStrategy | 从请求中提取参数的策略，目前支持提取来源 IP（PARAM_PARSE_STRATEGY_CLIENT_IP）、Host（PARAM_PARSE_STRATEGY_HOST）、任意 Header（PARAM_PARSE_STRATEGY_HEADER）和任意 URL 参数（PARAM_PARSE_STRATEGY_URL_PARAM）四种模式。 |
| paramItem - fieldName     | 若提取策略选择 Header 模式或 URL 参数模式，则需要指定对应的 header 名称或 URL 参数名称。 |
| paramItem - pattern       | 参数值的匹配模式，只有匹配该模式的请求属性值会纳入统计和流控；若为空则统计该请求属性的所有值。（1.6.2 版本开始支持） |
| paramItem - matchStrategy | 参数值的匹配策略，目前支持精确匹配（PARAM_MATCH_STRATEGY_EXACT）、子串匹配（PARAM_MATCH_STRATEGY_CONTAINS）和正则匹配（PARAM_MATCH_STRATEGY_REGEX）。（1.6.2 版本开始支持） |

用户可以通过 `GatewayRuleManager.loadRules(rules)` 手动加载网关规则，或**通过 `GatewayRuleManager.register2Property(property)` 注册动态规则源动态推送（推荐方式）**

4) 启动应用程序

配置完成后，启动应用程序即可。应用程序将自动连接 Sentinel 控制台，并将应用程序的运行状态、性能指标等数据发送给 Sentinel 控制台。在 Sentinel 控制台中，可以查看应用程序的运行状态、性能指标、规则配置等信息，以及手动修改规则配置、触发流量控制、熔断降级等操作。

### 4.3 限流

限流是一种常见的流量控制技术，可以在高并发的场景下保护系统的可靠性和稳定性，避免系统崩溃或者瘫痪。在微服务架构中，限流可以针对服务、接口、用户等维度进行控制，保护系统的稳定性和可用性。

在 Spring Cloud 中，可以使用多种方式实现限流，例如使用 Redis、Guava、Bucket4j 等限流组件，或者使用 Sentinel 等流量控制框架。以下是一些常见的限流实现方式：

#### 4.3.1 基于 Redis 的限流

Redis 是一款高性能的内存数据库，可以用于实现分布式限流。在 Spring Cloud 中，可以使用 RedisTemplate 和 Redisson 等 Redis 客户端库，将 Redis 作为限流组件。可以使用 Redis 的计数器、分布式锁等机制实现限流。例如，可以使用 Redis 的 INCRBY 命令实现令牌桶算法，对请求进行限流。

基于 Redis 的限流可以使用 Redis 的计数器和分布式锁等机制实现。以下是使用 Redis 实现令牌桶算法的示例：

1. 添加 Redis 依赖

在 pom.xml 文件中添加 Redis 相关依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
```

1. 配置 RedisTemplate

在 application.yml 文件中添加 Redis 相关配置：

```yaml
spring:
  redis:
    host: localhost
    port: 6379
    password: xxxx
```

在 RedisConfig.java 中配置 RedisTemplate：

```java
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new Jackson2JsonRedisSerializer<>(Object.class));
        return template;
    }
}
```

在以上配置中，使用了 RedisTemplate 将 Redis 作为限流组件。使用了 Jackson2JsonRedisSerializer 将对象序列化为 JSON 格式存储。

1. 编写限流代码

在需要进行限流的方法中，可以使用 Redis 的 INCRBY 命令实现令牌桶算法。例如：

```java
@Service
public class DemoService {

    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    public boolean tryAcquire(String key, int max, int rate) {
        String redisKey = "rate.limiter:" + key;
        long now = System.currentTimeMillis();
        String script = "local rate = tonumber(redis.call('get', KEYS[1])) or 0\n" +
                "local expire = tonumber(redis.call('ttl', KEYS[1])) or -2\n" +
                "local interval = 1000 / tonumber(ARGV[2])\n" +
                "local last = tonumber(redis.call('get', KEYS[2])) or 0\n" +
                "local tokens = math.floor(((now - last) + expire * 1000) / interval)\n" +
                "if tokens > 0 then\n" +
                "    redis.call('setex', KEYS[2], interval / 1000, now)\n" +
                "    local count = redis.call('incrby', KEYS[1], tokens)\n" +
                "    if count > tonumber(ARGV[1]) then\n" +
                "        return 0\n" +
                "    end\n" +
                "end\n" +
                "return 1";
        List<String> keys = Arrays.asList(redisKey, redisKey + ":last");
        Long result = (Long) redisTemplate.execute(new DefaultRedisScript<>(script, Long.class), keys, max, rate);
        return result != null && result > 0;
    }
}
```

在以上代码中，使用了 Lua 脚本实现令牌桶算法。首先获取当前令牌桶中的令牌数，然后计算时间间隔内应该生成的令牌数，将令牌数累加到令牌桶中，如果令牌数超过阈值，则限流。

1. 在 Controller 中使用限流

在 Controller 中使用 tryAcquire() 方法进行限流。例如：

```java
@RestController
public class DemoController {

    @Autowired
    private DemoService demoService;

    @GetMapping("/hello")
    public String hello() {
        if (demoService.tryAcquire("hello", 10, 1)) {
            return "Hello, World!";
        } else {
            return "Blocked by Redis";
        }
    }
}
```

在以上代码中，使用 DemoService 的 tryAcquire() 方法进行限流。限制每秒最多处理 10 个请求。

#### 4.3.2 基于 Guava 的限流

Guava 是 Google 开源的一款 Java 工具库，其中包含了限流组件 RateLimiter。RateLimiter 可以根据指定的速率限制请求的处理速度，例如每秒处理 10 个请求。在 Spring Cloud 中，可以使用 Guava 的 RateLimiter 实现限流。例如，在 Controller 中使用 RateLimiter：

```java
@RestController
public class DemoController {

    private final RateLimiter rateLimiter = RateLimiter.create(10.0);

    @GetMapping("/hello")
    public String hello() {
        if (rateLimiter.tryAcquire()) {
            return "Hello, World!";
        } else {
            return "Blocked by RateLimiter";
        }
    }
}
```

在以上代码中，创建了一个速率为 10.0 的 RateLimiter，表示每秒最多处理 10 个请求。在 Controller 中使用 tryAcquire() 方法尝试获取令牌，如果获取成功，则处理请求，否则返回限流提示信息。

#### 4.3.3 基于 Bucket4j 的限流

Bucket4j 是一个基于 Token Bucket 算法实现的 Java 限流库，可以用于实现基于令牌桶的限流。Bucket4j 支持多种限流模式，例如固定窗口、滑动窗口等模式。在 Spring Cloud 中，可以使用 Bucket4j 实现限流。例如，在 Controller 中使用 Bucket4j：

```java
@RestController
public class DemoController {

    private final Bucket bucket = Bucket4j.builder()
            .addLimit(Bandwidth.classic(5, Refill.intervally(5, Duration.ofSeconds(1))))
            .build();

    @GetMapping("/hello")
    public String hello() {
        if (bucket.tryConsume(1)) {
            return "Hello, World!";
        } else {
            return "Blocked by Bucket4j";
        }
    }
}
```

在以上代码中，创建了一个限流桶，设置了速率为 5 个请求/秒。在 Controller 中使用 tryConsume() 方法尝试获取令牌，如果获取成功，则处理请求，否则返回限流提示信息。

#### 4.3.4 基于 Sentinel 的限流

Sentinel 是阿里巴巴开源的一款流量控制框架，可以实现实时监控和限流、熔断降级等功能，可以作为微服务架构中的流量控制和熔断降级组件。在 Spring Cloud 中，可以使用 Sentinel 实现限流。例如，在 Controller 中使用 @SentinelResource 注解：

```java
@RestController
public class DemoController {

    @GetMapping("/hello")
    @SentinelResource(value = "hello", blockHandler = "handleHelloBlock")
    public String hello() {
        return "Hello, World!";
    }

    public String handleHelloBlock(BlockException ex) {
        return "Blocked by Sentinel: " + ex.getMessage();
    }
}
```

在以上代码中，使用 @SentinelResource 注解定义了一个 Sentinel 资源，名称为 hello，同时指定了 blockHandler 属性，表示当资源被流量控制限制时，调用 handleHelloBlock() 方法进行处理。

#### 4.3.4 内置的限流

RequestRateLimiterGatewayFilterFactory 是 Spring Cloud Gateway 中的一个限流过滤器工厂，可以基于 Redis、Guava 等限流组件实现请求速率限制（Request Rate Limiting），保护服务的稳定性和可用性。

RequestRateLimiterGatewayFilterFactory 提供了多种限流算法和限流配置方式，可以根据具体场景选择合适的限流策略。以下是使用 RequestRateLimiterGatewayFilterFactory 实现基于 Redis 的请求速率限制的示例：

1. 添加 Redis 依赖

在 pom.xml 文件中添加 Redis 相关依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

2) 配置 RedisTemplate

在 application.yml 文件中添加 Redis 相关配置：

```yaml
spring:
  redis:
    host: localhost
    port: 6379
    password: xxxx
```

在 RedisConfig.java 中配置 RedisTemplate：

```jaav
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new Jackson2JsonRedisSerializer<>(Object.class));
        return template;
    }
}
```

3) 配置限流规则

在 application.yml 文件中配置限流规则：

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: example
          uri: http://example.com
          predicates:
            - Path=/example/**
          filters:
            - name: RequestRateLimiter
              args:
                key-resolver: "#{@apiKeyResolver}"
                redis-rate-limiter.replenishRate: 1
                redis-rate-limiter.burstCapacity: 10
```

在以上配置中，定义了一个名为 example 的路由规则，匹配 /example/** 的请求。使用 RequestRateLimiter 过滤器工厂，配置了 key-resolver、replenishRate 和 burstCapacity 三个参数，表示使用 apiKeyResolver 解析限流 key，限制每秒最多处理 1 个请求，令牌桶容量为 10。

4) 实现 KeyResolver

在 GatewayConfig.java 中实现 KeyResolver 接口：

```java
@Configuration
public class GatewayConfig {

    @Bean
    public KeyResolver apiKeyResolver() {
        return exchange -> Mono.just(exchange.getRequest().getPath().value());
    }
}
```

在以上代码中，使用请求路径作为限流 key。

5) 启动应用程序

在启动应用程序后，访问 /example/** 的请求将会进行请求速率限制，保护服务的稳定性和可用性。



6) RequestRateLimiterGatewayFilter实现

`RequestRateLimiterGatewayFilter` 工厂使用 `RateLimiter` 实现来确定是否允许当前请求继续。如果不是，则返回 `HTTP 429 - Too Many Requests` 状态（默认情况下）。

此筛选器采用可选 `keyResolver` 参数和特定于速率限制器的参数（本节稍后将介绍）。

`keyResolver` 是实现接口的 `KeyResolver` Bean。在配置中，使用 SpEL 按名称引用 Bean。 `#{@myKeyResolver}` 是一个 SpEL 表达式，它引用名为 `myKeyResolver` 的 Bean。以下清单显示了该 `KeyResolver` 接口：

```java
public interface KeyResolver {
    Mono<String> resolve(ServerWebExchange exchange);
}
```

该 `KeyResolver` 接口允许可插拔策略派生用于限制请求的密钥。在未来的里程碑版本中，将有一些 `KeyResolver` 实现。

默认 `KeyResolver` 实现是 `PrincipalNameKeyResolver` ，它 `Principal` 从 `ServerWebExchange` 和 调用 `Principal.getName()` 中检索 。

默认情况下，如果找不到 `KeyResolver` 密钥，则拒绝请求。您可以通过设置 `spring.cloud.gateway.filter.request-rate-limiter.deny-empty-key` （ `true` 或 `false` ） 和 `spring.cloud.gateway.filter.request-rate-limiter.empty-key-status-code` 属性来调整此行为。

>  `RequestRateLimiter` 不能使用“快捷方式”表示法进行配置。以下示例无效：
>
> ```properties
> spring.cloud.gateway.routes[0].filters[0]=RequestRateLimiter=2, 2, #{@userkeyresolver}
> ```



RateLimiter 的 Redis 实现基于 [Stripe](https://stripe.com/blog/rate-limiters) 所做的工作。它需要使用 `spring-boot-starter-data-redis-reactive` Spring Boot 启动器。使用的算法是令牌桶算法。

该 `redis-rate-limiter.replenishRate` 属性定义每秒允许的请求数（没有任何丢弃的请求）。这是填充令牌桶的速率。

该 `redis-rate-limiter.burstCapacity` 属性是用户在一秒钟内允许的最大请求数（没有任何丢弃的请求）。这是令牌存储桶可以容纳的令牌数。将此值设置为零会阻止所有请求。

该 `redis-rate-limiter.requestedTokens` 属性是请求的成本。这是每个请求从存储桶中获取的令牌数，默认为 `1` 。

通过在 和 `burstCapacity` 中 `replenishRate` 设置相同的值来实现稳定速率。通过设置 `burstCapacity` 高于 ，可以允许临时突发 `replenishRate` 。在这种情况下，需要在突发之间允许速率限制器一段时间（根据 `replenishRate` ），因为连续两次突发会导致请求丢弃 （ `HTTP 429 - Too Many Requests` ）。以下清单配置： `redis-rate-limiter`。

以下 `1 request/s` 速率限制是通过设置为 `replenishRate` 所需的请求数、 `requestedTokens` 以秒为单位的时间跨度以及 `burstCapacity` 和 `requestedTokens` 的乘积来实现的 `replenishRate` 。例如，设置 `replenishRate=1` 、 `requestedTokens=60` 和 `burstCapacity=60` 会导致限制为 `1 request/min` 。

application.yml

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: requestratelimiter_route
        uri: https://example.org
        filters:
        - name: RequestRateLimiter
          args:
            redis-rate-limiter.replenishRate: 10
            redis-rate-limiter.burstCapacity: 20
            redis-rate-limiter.requestedTokens: 1
```

以下示例在 Java 中配置  `KeyResolver` ：

```java
@Bean
KeyResolver userKeyResolver() {
    return exchange -> Mono.just(exchange.getRequest().getQueryParams().getFirst("user"));
}
```

这定义了每个用户 10 个的请求速率限制。允许突发 20 个，但在下一秒，只有 10 个请求可用。 `KeyResolver` 这是一个获取 `user` 请求参数的简单参数 注意：不建议将其用于生产。

您还可以将速率限制器定义为实现 `RateLimiter` 接口的 Bean。在配置中，您可以使用 SpEL 按名称引用 Bean。 `#{@myRateLimiter}` 是一个 SpEL 表达式，它引用名为 的 `myRateLimiter` Bean。下面的清单定义了一个速率限制器，该限制器使用前面清单中 `KeyResolver` 定义的 ：

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: requestratelimiter_route
        uri: https://example.org
        filters:
        - name: RequestRateLimiter
          args:
            rate-limiter: "#{@myRateLimiter}"
            key-resolver: "#{@userKeyResolver}"
```



### 4.4 日志

#### 4.4.1日志过滤器

以下是一个示例，展示了如何在 Gateway 中记录请求和响应的信息：

```java
public class LoggingGatewayFilterFactory extends AbstractGatewayFilterFactory<LoggingGatewayFilterFactory.Config> {
    public LoggingGatewayFilterFactory() {
        super(Config.class);
    }

    @Override
    public GatewayFilter apply(Config config) {
        return (exchange, chain) -> {
            ServerHttpRequest request = exchange.getRequest();
            ServerHttpResponse response = exchange.getResponse();

            if (config.isRequestLoggingEnabled()) {
                // 记录请求信息
                log.info("Request URI: {}", request.getURI());
                log.info("Request Method: {}", request.getMethod());
                log.info("Request Headers: {}", request.getHeaders());
                log.info("Request Remote Address: {}", request.getRemoteAddress());
            }

            // 调用下一个过滤器
            return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                if (config.isResponseLoggingEnabled()) {
                    // 记录响应信息
                    log.info("Response Status Code: {}", response.getStatusCode());
                    log.info("Response Headers: {}", response.getHeaders());
                }
            }));
        };
    }

  	@Data
    public static class Config {
        private boolean requestLoggingEnabled;
        private boolean responseLoggingEnabled;
    }
}
```

在 `LoggingGatewayFilterFactory` 的 `Config` 类中，定义了两个 boolean 类型的参数 `requestLoggingEnabled` 和 `responseLoggingEnabled`，用于控制是否开启请求日志和响应日志的记录。在 `apply` 方法中，根据这两个参数的值来决定是否记录请求和响应的信息。在 Spring Cloud Gateway 的配置文件中，可以通过 `args` 属性来设置这两个参数的值，例如：

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: example
          uri: http://example.com
          filters:
            - Logging
          args:
            requestLoggingEnabled: true
            responseLoggingEnabled: true
      default-filters:
        - name: Logging
          args:
            requestLoggingEnabled: true
            responseLoggingEnabled: true
```

> `default-filters` 是 Spring Cloud Gateway 的一个全局过滤器属性，用于指定在所有路由中都应用的过滤器。
>
> 当定义了 `default-filters` 属性时，这些过滤器将自动应用于所有路由，无需在每个路由中单独指定。这有助于简化配置，并确保所有路由都遵循相同的安全和性能标准。

#### 4.4.2 Logback记录日志

Spring Cloud Gateway 使用 Spring Boot 默认的日志框架 Logback 进行日志记录。通过配置 Logback 的 XML 文件，可以定义日志的输出格式、级别和目标，以满足不同的需求。

在日志配置中，可以设置网关的请求日志和响应日志，以便记录请求和响应的详细信息，例如请求 URL、响应状态码、请求头、响应体等。此外，Spring Cloud Gateway 还支持使用 Sleuth 和 Zipkin 进行分布式跟踪和日志聚合，以方便开发者进行分布式系统的调试和排错。

#### 4.4.3 Reactor Netty 访问日志

https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#reactor-netty-access-logs

要启用 Reactor Netty 访问日志，请设置 `-Dreactor.netty.http.server.accessLogEnabled=true`，还可以在Logback中将日志记录系统配置为具有单独的访问日志文件。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <property name="LOG_PATTERN" value="%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n"/>

    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
        </encoder>
    </appender>

    <appender name="access" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/access.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/access.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>7</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%msg%n</pattern>
        </encoder>
    </appender>

    <appender name="async" class="ch.qos.logback.classic.AsyncAppender">
        <queueSize>1048576</queueSize>
        <discardingThreshold>0</discardingThreshold>
        <includeCallerData>true</includeCallerData>
        <appender-ref ref="access"/>
    </appender>

    <logger name="reactor.netty.http.server.AccessLog" level="INFO" additivity="false">
        <appender-ref ref="async"/>
    </logger>

    <root level="INFO">
        <appender-ref ref="console"/>
    </root>
</configuration>
```



### 4.5 监控

Spring Cloud Gateway 提供了多种监控方式，包括：

1. Actuator：Spring Boot Actuator 是一个用于监控和管理 Spring Boot 应用程序的框架，提供了多种监控和管理功能，包括应用程序信息、健康状况、请求跟踪、环境信息等。Spring Cloud Gateway 集成了 Spring Boot Actuator，可以通过 HTTP 或 JMX 接口来访问这些监控信息。
2. Prometheus：Prometheus 是一款开源的监控系统，可以帮助开发者收集、存储和分析系统的监控数据。Spring Cloud Gateway 集成了 Micrometer，可以将网关的监控数据输出为 Prometheus 格式，以方便集成到 Prometheus 中进行监控。
3. Grafana：Grafana 是一款流行的开源监控和数据可视化平台，可以帮助开发者将监控数据进行可视化展示和分析。通过将 Spring Cloud Gateway 的监控数据导入 Grafana 中，可以更加直观地了解网关的性能和健康状况。
4. Zipkin：Zipkin 是一款分布式跟踪系统，可以帮助开发者跟踪和分析分布式系统的请求链路。Spring Cloud Gateway 集成了 Spring Cloud Sleuth 和 Zipkin，可以将请求和响应的跟踪数据输出到 Zipkin 中，以方便进行分布式系统的调试和排错。





## 5. 开发向导



## 6. 部署和管理

### 6.1 部署选项

Spring Cloud Gateway 可以部署在各种环境中，例如物理机、虚拟机、容器等。以下是 Spring Cloud Gateway 的部署方式：

1. 物理机部署

在物理机上部署 Spring Cloud Gateway 需要安装 Java 运行环境和 Spring Boot 应用程序。可以通过以下步骤进行部署：

- 下载和解压 Spring Boot 应用程序。
- 安装 Java 运行环境。
- 在命令行中进入应用程序目录。
- 执行命令 `./mvnw spring-boot:run` 启动应用程序。

在以上步骤中，需要将应用程序打包成可执行的 Jar 包，或者使用 Maven 或 Gradle 进行构建。

2) 虚拟机部署

在虚拟机上部署 Spring Cloud Gateway 需要安装虚拟化软件和操作系统。可以通过以下步骤进行部署：

- 安装虚拟化软件，例如 VirtualBox、VMware 等。
- 创建虚拟机，选择适合的操作系统，例如 Ubuntu、CentOS 等。
- 在虚拟机中安装 Java 运行环境和 Spring Boot 应用程序。
- 启动应用程序。

在以上步骤中，需要将应用程序打包成可执行的 Jar 包，并将 Jar 包上传到虚拟机中。

3) 容器部署

在容器中部署 Spring Cloud Gateway 可以使用 Docker、Kubernetes 等容器技术。可以通过以下步骤进行部署：

- 编写 Dockerfile 文件，配置应用程序环境和依赖。
- 使用 Docker 命令构建 Docker 镜像。
- 使用 Docker 命令启动容器。

在以上步骤中，需要将应用程序打包成可执行的 Jar 包，并将 Jar 包复制到 Docker 镜像中。

4) 部署到云平台

Spring Cloud Gateway 也可以部署到云平台，例如 AWS、Azure、GCP 等。可以通过以下步骤进行部署：

- 创建云服务器、容器或云函数。
- 在服务器、容器或云函数中安装 Java 运行环境和 Spring Boot 应用程序。
- 启动应用程序。

在以上步骤中，需要将应用程序打包成可执行的 Jar 包，并将 Jar 包上传到云平台中。



### 6.2 容器化部署

容器化部署是将应用程序打包成一个独立的可运行的容器镜像，然后在容器环境中运行的方式。容器化部署的优点包括：便于构建、移植、扩展和管理，可以提高应用程序的可靠性和可维护性。以下是容器化部署 Spring Cloud Gateway 的步骤：

1. 编写 Dockerfile 文件

Dockerfile 文件是容器镜像的构建脚本，用于定义容器镜像的构建过程。可以通过以下 Dockerfile 文件创建一个 Spring Cloud Gateway 的容器镜像：

```dockerfile
FROM openjdk:8-jre-slim

LABEL maintainer="Your Name <your.email@example.com>"

# Set environment variables
ENV APP_HOME=/data TZ=Asia/Shanghai JAVA_OPTS="-Xms128m -Xmx256m -Djava.security.egd=file:/dev/./urandom"

# Create application directory
RUN mkdir -p $APP_HOME

# Set application directory as working directory
WORKDIR $APP_HOME

# Copy the application jar file to the container
COPY target/*.jar $APP_HOME/app.jar

# Expose port 8080 for the application
EXPOSE 8080

# Run the application
CMD ["java","-Xms128m -Xmx256m -Djava.security.egd=file:/dev/./urandom", "-jar", "app.jar"]
```

在以上 Dockerfile 文件中，定义了容器镜像的基础镜像、维护者、应用程序目录、工作目录、应用程序 jar 文件、暴露端口和运行命令。

创建一个 .dockerignore文件忽略不需要编译的文件：

```bash
.git
.gitignore
*.iml
.idea

logs
docs
target/*-javadoc.jar
target/*-sources.jar
```

如果使用分布编译，则 Dockerfile 文件如下：

```docker
FROM openjdk:8-jdk-alpine AS builder
WORKDIR /build
COPY target/*.jar app.jar
RUN java -Djarmode=layertools -jar app.jar extract && rm app.jar

FROM openjdk:8-jdk-alpine

ENV TZ=Asia/Shanghai JAVA_OPTS="-Xms128m -Xmx256m -Djava.security.egd=file:/dev/./urandom"

WORKDIR /data

COPY --from=builder /build/dependencies/ /build/snapshot-dependencies/ /build/spring-boot-loader/ /build/application/ ./

EXPOSE 8080

CMD sleep 5; java $JAVA_OPTS org.springframework.boot.loader.JarLauncher
```

2) 构建容器镜像

使用 Docker 命令构建容器镜像：

```bash
docker build -t gateway:1.0.0 .
```

在以上命令中，`-t` 参数指定容器镜像的名称和标签，`.` 表示 Dockerfile 文件所在目录。

3) 运行容器

使用 Docker 命令运行容器：

```
docker run -p 8080:8080 gateway:1.0.0
```

在以上命令中，`-p` 参数指定容器端口和主机端口的映射关系，`gateway:1.0.0` 是容器镜像的名称和标签。

4) 访问服务

在容器运行后，可以通过主机的 IP 地址和容器映射的端口访问 Spring Cloud Gateway 提供的服务。

5) 使用 docker-compose 构建和运行容器

docker-compose.yml

```yaml
version: '3'
services:
  cocktail-gateway:
    build:
      context: ./gateway
    container_name: gateway
    image: gateway
    hostname: gateway
    restart: always
    env_file:
      - .env
    ports:
      - 8080:8080
```

.env 保存一些环境变量，如：Redis配置、数据库配置、Nacos地址、spring profile 等等：

```bash
SPRING_PROFILES_ACTIVE=dev

NACOS_HOST=XXXX

MYSQL_HOST=XXXX
MYSQL_USER=root
MYSQL_PASS=XXXX

REDIS_HOST=XXXX
REDIS_DB=15
REDIS_PASS=XXXX
```

运行容器：

 ```bash
 docker-compose -f docker-compose.yml up -d
 ```



### 6.3 管理和监控



## 7. 故障排除

### 7.1 常见问题

### 7.2 日志分析





## 8. 总结和参考资料

### 8.1 总结

### 8.2 参考资料

- https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/

