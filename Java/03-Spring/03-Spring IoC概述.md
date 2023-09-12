## Spring IoC 依赖查找

- 根据 Bean 名称查找

  - 实时查找

  - 延迟查找

- 根据 Bean 类型查找

  - 单个 Bean 对象

  - 集合 Bean 对象

- 根据 Bean 名称 + 类型查找

- 根据 Java 注解查找
  - 单个 Bean 对象
  - 集合 Bean 对象



## Spring IoC 依赖注入

- 根据 Bean 名称注入

- 根据 Bean 类型注入

  - 单个 Bean 对象

  - 集合 Bean 对象

- 注入容器內建 Bean 对象

- 注入非 Bean 对象

- 注入类型
  - 实时注入
  - 延迟注入



## Spring IoC 依赖来源

- 自定义 Bean

- 容器內建 Bean 对象

- 容器內建依赖



## Spring IoC 配置元信息

- Bean 定义配置

  - 基于 XML 文件

  - 基于 Properties 文件

  - 基于 Java 注解
  - 基于 Java API（专题讨论）

-  IoC 容器配置

  - 基于 XML 文件

  - 基于 Java 注解

  - 基于 Java API （专题讨论）

- 外部化属性配置
  - 基于 Java 注解

## Spring IoC 容器

BeanFactory 和 ApplicationContext 谁才是 Spring IoC 容器？

## Spring 应用上下文

- ApplicationContext 除了 IoC 容器角色，还有提供：

  - 面向切面（AOP）

  - 配置元信息（Configuration Metadata）

  - 资源管理（Resources）

  - 事件（Events）

  - 国际化（i18n）

  - 注解（Annotations）

  - Environment 抽象（Environment Abstraction）

## 使用 Spring IoC 容器

BeanFactory 是 Spring 底层 IoC 容器

ApplicationContext 是具备应用特性的 BeanFactory 超集



## Spring IoC 容器生命周期



## 面试题

1、什么是 Spring IoC 容器？

2、BeanFactory 与 FactoryBean？

3、Spring IoC 容器启动时做了哪些准备？