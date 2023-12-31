# 1. IoC发展简介

## 什么是IoC

IoC（Inversion of Control，控制反转）是一种设计思想，它指的是将对象的创建、依赖关系的维护和对象的生命周期的管理等任务交给容器负责，而不是由对象自己负责。在传统的编程模型中，对象间的依赖关系是由对象自己管理的，而在 IoC 模型中，容器负责创建和管理对象，并将对象之间的依赖关系注入到对象中。

在 Spring 框架中，IoC 是实现依赖注入（Dependency Injection，DI）的核心机制。依赖注入是指容器将一个对象所需的依赖关系注入到该对象中，使得该对象能够正常工作。依赖注入可以通过构造函数、Setter 方法和字段注入等方式实现。Spring 框架提供了丰富的依赖注入方式，使得开发者可以选择最适合自己的方式来实现依赖注入。



## IoC简史

- 1983年，Richard E. Sweet 在《The Mesa Programming Environment》中提出"Hollywood Principle"（好莱坞原则）
- 1988年，Ralph E. Johnson & Brian Foote 在《Designing Reusable Classes》中提出"Inversion of control"（控制反转）
- 1996年，Michael Mattson 在 《Object-Oriented Frameworks， A survey of methodological
  issues》中将'Inversion of controT 命名为"Hollywood principle"
- 2000 年，Martin Fowler 首次在他的文章中提到了 IoC 的概念，并将其称为“依赖注入”（Dependency Injection，DI）。从那时起，DI 就成为了 IoC 的一个重要实现方式。
- 2004 年，Rod Johnson 在他的著作《Expert One-on-One J2EE Development without EJB》中介绍了 Spring 框架。Spring 框架是一个基于 IoC 和 AOP（Aspect-Oriented Programming，面向切面编程）的轻量级应用程序框架，它的成功推广和广泛应用使得 IoC 和 DI 的概念被越来越多的开发者所认识和使用。
- 2004年，Martin Fowler 在《Inversion of Control Containers and the Dependency Injection pattern》中提出了自己对loC以及DI的理解
- 2005年，Martin Fowler 在《InversionOfControl》对 IoC 做出进一步的说明

# 2. IoC主要实现策略

在面向对象编程中，有几种基本技术来实现控制反转。这些技术包括：

- 使用服务定位器。服务定位器是一种中央注册表的概念，用于查找和获取对象的实例。在服务定位器模式中，应用程序通过服务定位器来获取所需的服务或对象实例，而不是直接依赖于具体的实现类。通过一个服务定位器来管理对象之间的依赖关系，客户端可以通过服务定位器来获取所需的对象。服务定位器可以是一个单例对象，也可以是一个可配置的对象。

- 使用依赖注入：依赖注入是IoC的一种具体实现方式。它通过将对象的依赖关系从代码中移出，由容器负责创建和管理对象之间的依赖关系。依赖注入可以通过构造函数、属性注入或方法注入来实现。

- 使用上下文化查找。一个对象通过查找上下文或环境来获取它所需要的依赖项或服务，而不是直接实例化或创建这些依赖项或服务。

- 钩子方法（Hook Methods）：钩子方法是一种在父类中定义的虚拟方法，子类可以通过重写这些方法来插入自定义的行为。通过使用钩子方法，父类可以控制和管理子类的行为，实现了对子类的控制反转。

- 工厂模式（Factory Pattern）：工厂模式是一种常见的创建对象的设计模式。在工厂模式中，通过将对象的创建逻辑委托给工厂类，将对象的实例化过程与使用对象的代码分离。这样可以通过工厂来管理对象的生命周期和依赖关系。

  

《Expert One-on-One™ J2EE™ Development without EJB™》提到的主要实现策略：

- 依赖查找（Dependency Lookup）：容器提供回调方法和查找上下文给组件使用，这是 EJB 和 Apache Avalon 的实现方法。它让每个组件使用容器 API 来查找资源和协作者。控制反转仅限于容器调用回调方法，以便应用程序代码可以获取资源。
- 依赖注入（Dependency Injection）：组件不再进行查找，它们提供了普通的 Java 方法来让容器解析依赖关系。容器完全负责将组件连接起来，将解析后的对象传递给 JavaBean 属性或构造函数。使用 JavaBean 属性的方式称为 Setter 注入；使用构造函数参数的方式称为 Constructor 注入。

# 3. IoC容器的职责

控制反转（Inversion of Control，IoC）的设计目的是解耦和增强可扩展性。它通过将对象的创建、依赖关系的管理和控制的责任转移到容器或框架中，以实现以下目标：

1. 解耦：IoC的主要目的是解耦应用程序的各个组件和模块之间的依赖关系。传统的编程方式中，对象之间通常直接相互依赖，导致紧耦合的代码。而使用IoC，依赖关系被反转，对象不再直接依赖于特定的实现类，而是通过接口或抽象来定义依赖关系，由容器负责提供具体的实现。这种解耦使得代码更加灵活、可维护和可测试。
2. 可扩展性：IoC提供了可扩展性和灵活性，使得应用程序更容易进行扩展和变更。通过将对象的创建和依赖关系的管理交给容器，新增或替换组件变得更加简单。可以通过配置文件或注解来定义依赖关系，而不需要修改源代码。这种灵活性使得应用程序可以更容易地适应变化的需求，支持模块化和可插拔的架构。
3. 可测试性：IoC提升了应用程序的可测试性。由于依赖关系被外部化，可以使用模拟对象或测试替身来替代真实的依赖项，从而实现更容易的单元测试和集成测试。测试时可以灵活地注入不同的依赖项或模拟对象，以验证组件的行为。
4. 可维护性：IoC使得应用程序的维护更加简化。由于依赖关系被集中管理，当需要修改或替换某个组件时，只需要修改配置或注解，而不需要修改大量的代码。这降低了维护成本，并减少了引入错误的风险。



**通用职责**

- 依赖处理
  - 依赖查找
  - 依赖注入
- 生命周期管理
  - 容器
  - 托管的资源（Java Bean 或其他资源）
- 配置
  - 容器
  - 外部化配置
  - 托管的资源（Java Bean 或其他资源）

# 4. IoC容器的实现

- Java SE
  - Java Beans
  - Java ServiceLoader SPI
  - JNDI (Java Naming and Directory Interface)
- Java EE
  - EJB (Enterprise Java Beans)
  - Servlet
- 开源
  - [Apache Avalon](https://avalon.apache.org/closed.html) 和 EJB类似，是开源轻量级依赖查找框架
  - [PicoContainer](http://picocontainer.com/) 《Expert One-on-One™ J2EE™ Development without EJB™》第六章提到Spring Framework技术灵感来自PicoContainer
  - [Google Guice](https://github.com/google/guice) 当前国外除了Spring Framework比较流行的之一
  - [Spring Framework](https://spring.io/projects/spring-framework)

> 面试题：Spring 作为 IoC 容器有什么优势？
>
> 答：
>
> 典型的 IoC 管理，依赖查找和依赖注入
>
> AOP 抽象
>
> 事务抽象
>
> 事件机制
>
> SPI 扩展
>
> 强大的第三方整合
>
> 易测试性
>
> 更好的面向对象

# 5. 传统IoC容器实现

**Java Beans作为IoC容器**

- 特性
  - 依赖查找
  - 生命周期管理
  - 配置元信息
  - 事件
    Spring 也是基于Java的事件机制来做的
  - 自定义
  - 资源管理
  - 挺久化
- 规范
  - [JavaBeans](https://docs.oracle.com/javase/tutorial/javabeans/)
  - [Bean Context](https://docs.oracle.com/javase/8/docs/technotes/guides/beans/spec/beancontext.html)

# 6. 轻量级IoC容器

轻量级IoC容器通常具有以下特征：
1. 简洁性：轻量级IoC容器的设计目标之一是保持简洁性。它们通常提供简单而精练的API，以减少配置和学习成本。这使得容器的使用和配置变得更加直观和容易上手。
2. 易用性：轻量级IoC容器注重提供友好的开发体验。它们通常提供简单的注解或配置方式来定义对象的依赖关系和注入方式，而无需繁琐的XML配置。容器会自动处理对象的创建、依赖注入和生命周期管理，使开发者能够专注于业务逻辑而不必过多关注底层容器的细节。
3. 轻量级：轻量级IoC容器的实现相对较小、轻便，并且对资源消耗较少。它们通常不依赖于大型框架或库，并且只提供IoC容器所需的最基本功能。这使得容器在性能和资源方面具有较低的开销，并且适合在资源受限的环境下使用。
4. 快速启动：轻量级IoC容器通常具有快速启动的特点。它们旨在在应用程序启动时快速初始化和配置对象，以便尽快地提供依赖注入的功能。这对于需要快速启动和响应时间敏感的应用程序非常重要。
5. 可扩展性：尽管轻量级IoC容器功能相对较简单，但它们通常具备一定的扩展性。它们提供了可插拔的机制，允许开发者通过扩展或自定义来满足特定需求。这使得容器能够适应不同的应用场景，并支持定制化的配置。



使用轻量级IoC容器带来以下好处：

1. 解耦与模块化：IoC容器能够解耦对象之间的依赖关系。通过容器管理对象的创建和依赖注入，对象只需要关注自身的功能，而不需要关心如何获取所需的依赖对象。这样可以将系统拆分为独立的模块，提高代码的可维护性和可测试性。
2. 简化配置：轻量级IoC容器通常提供简洁的配置方式，如注解或简单的配置文件。相比手动管理对象的依赖关系，使用容器可以减少繁琐的配置工作，提高开发效率和代码可读性。
3. 可替换性与灵活性：通过使用IoC容器，可以轻松替换依赖的实现。容器负责管理对象的创建和注入，可以根据配置或条件选择不同的实现类。这样在更换或升级依赖时，只需调整容器的配置，而无需修改大量的代码。
4. 生命周期管理：IoC容器通常提供对象的生命周期管理功能。它可以在适当的时候创建对象、注入依赖、销毁对象等，确保对象的生命周期得到正确管理，避免资源泄漏和内存溢出等问题。
5. 依赖注入：IoC容器通过依赖注入将对象之间的依赖关系注入到目标对象中。这种方式使得对象之间的耦合度降低，提高了代码的可测试性和可维护性。同时，依赖注入也使得代码更加灵活，可以根据需求动态切换依赖对象。
6. 提供AOP支持：一些轻量级IoC容器还提供面向切面编程（AOP）的支持。AOP可以在不修改目标对象代码的情况下，通过横切关注点的方式实现一些横向的功能，如日志记录、事务管理等。通过容器的AOP支持，可以方便地应用和管理横切关注点。

# 7. 依赖查找 VS 依赖注入

| 类型     | 依赖注入 | 实现便利性 | 代码侵入性   | API依赖性     | 可读性 |
| -------- | -------- | ---------- | ------------ | ------------- | ------ |
| 依赖查找 | 主动获取 | 相对繁琐   | 侵入业务逻辑 | 依赖容器API   | 良好   |
| 依赖注入 | 被动提供 | 相对便利   | 低侵入性     | 不依赖容器API | 一般   |

依赖查找（Dependency Lookup）和依赖注入（Dependency Injection）是两种不同的依赖管理方式。

1. 依赖查找：依赖查找是指在需要使用依赖对象时，通过显式地向容器请求获取所需的依赖对象。在依赖查找中，对象需要主动查询容器或使用特定的查找方法来获取它所需要的依赖对象。这通常涉及到与容器的接口进行交互，以获取依赖对象的引用。依赖查找模式需要对象了解容器的存在，增加了对象与容器之间的耦合。
2. 依赖注入：依赖注入是指通过容器自动将依赖对象注入到目标对象中，而无需目标对象显式地获取依赖对象。在依赖注入中，对象不需要知道容器的存在，而是通过容器在创建对象时自动注入所需的依赖。依赖注入可以通过构造函数、属性注入或方法注入的方式实现。这样可以降低对象之间的耦合度，使代码更加清晰、可测试和可维护。

下面是一些比较依赖查找和依赖注入的特点：

依赖查找：

- 对象需要主动参与依赖对象的获取，增加了对象与容器之间的耦合。
- 对象需要了解容器的接口或特定的查找方法。
- 可以动态地获取依赖对象，但需要自己管理依赖对象的生命周期。
- 更加灵活，可以根据需要选择不同的依赖对象实例。
- 适用于需要动态获取依赖对象或基于特定条件选择依赖对象的场景。

依赖注入：

- 对象无需了解容器的存在，通过容器自动注入依赖对象。
- 降低了对象与容器之间的耦合度，使代码更加清晰、可测试和可维护。
- 容器负责管理依赖对象的生命周期，确保对象的依赖关系正确注入。
- 更加符合面向对象设计原则，如单一职责、开闭原则等。
- 适用于大多数情况下，特别是对于简化对象之间的依赖关系和提高代码可测试性的需求。

综上所述，依赖查找和依赖注入是两种不同的依赖管理方式，各自有其适用的场景和特点。依赖注入通过自动注入依赖对象来降低对象间的耦合度，提高代码的可维护性和可测试性，是一种常用的依赖管理方式。

> 面试题：依赖查找和依赖注入的区别？
>
> 答：依赖查找是主动或手动的依赖查找方式，通常需要依赖容器或标准 API 实现。而依赖注入则是手动或自动依赖绑定的方式，无需依赖特定的容器和 API

# 8. 构造器注入 VS Setter 注入﻿

构造器注入（Constructor Injection）和Setter注入（Setter Injection）是依赖注入中两种常见的方式。

1. 构造器注入：构造器注入是通过对象的构造器（构造函数）来注入依赖对象。在构造器注入中，依赖对象作为构造器的参数传递给对象，在对象创建时就完成了依赖的注入。这意味着对象在创建后就具有了完备的依赖关系，并且依赖对象是不可变的。通常，构造器注入要求所有的依赖都在构造器中声明，并且在对象创建时就要一次性提供所有的依赖。
2. Setter注入：Setter注入是通过对象的Setter方法来注入依赖对象。在Setter注入中，对象提供一系列的Setter方法，用于设置依赖对象的引用。通过调用这些Setter方法，容器将依赖对象注入到目标对象中。相对于构造器注入，Setter注入允许在对象创建后动态地注入依赖，可以灵活地修改依赖对象。

下面是一些比较构造器注入和Setter注入的特点：

构造器注入：

- 依赖对象在对象创建时通过构造器参数传递，使对象具有不可变性。
- 依赖关系在对象创建时就被确定，使得对象的依赖关系更加清晰和可靠。
- 对象在创建后就具备完备的依赖关系，避免了对象创建后状态不一致的问题。
- 构造器注入要求所有的依赖都在构造器中声明，可能导致构造器参数较多的情况。

Setter注入：

- 允许在对象创建后动态地注入依赖，灵活性更高。
- 可以选择性地设置依赖对象，允许部分依赖可选或可变。
- Setter方法提供了更加灵活的依赖注入方式，可以在任何时候修改依赖对象。
- 对象在创建后可能处于不完全初始化的状态，可能需要额外的逻辑来确保对象的正确使用。

通常情况下，构造器注入更适合在对象创建时确定完备的依赖关系，使对象更加稳定和一致。Setter注入更适合在对象创建后动态地修改依赖关系，以及处理部分依赖可选或可变的情况。实际使用时，可以根据具体情况选择适合的注入方式，或者在需要时结合两种方式使用。

需要注意的是，无论是构造器注入还是Setter注入，它们都是依赖注入的不同实现方式，旨在实现依赖关系的解耦和灵活性。选择合适的注入方式取决于具体的需求和设计考虑。
