

## **1. 概述**

在本教程中，我们将探讨 Spring 中的异步执行支持和 `@Async` 注解。

简单地说，用 `@Async` 注解 bean 的方法将使其在单独的线程中执行。换句话说，调用者不会等待被调用方法的完成。



## 2.示例程序

### 2.1.启用异步支持

让我们首先通过 Java 注解启用异步处理。


我们将通过将 `@EnableAsync` 添加到配置类来完成此操作：

```java
@Configuration
@EnableAsync
public class SpringAsyncConfig { ... }
```


启用注解就足够了。但也有一些简单的配置选项：

- **annotation** 默认情况下，`@EnableAsync` 检测 Spring 的 `@Async` 注解和 EJB 3.1 `javax.ejb.Asynchronous`。我们也可以使用此选项来检测其他用户定义的注解类型。
- **mode** 指示应使用的建议类型 - 基于 `JDK` 代理或 `AspectJ` 编织。
- ***proxyTargetClass*** 指示应使用的代理类型 — `CGLIB` 或 `JDK`。仅当模式设置为 `AdviceMode.PROXY` 时，此属性才有效。
- ***order*** 设置应用 `AsyncAnnotationBeanPostProcessor` 的顺序。默认情况下，它最后运行，以便它可以考虑所有现有代理。

### 2.2.@Async注解


首先，让我们回顾一下规则。 `@Async` 有两个限制：

- 它必须仅应用于公共方法。
- 自调用（从同一个类中调用异步方法）将不起作用。


原因很简单：该方法需要公开，以便可以被代理。并且自调用不起作用，因为它绕过代理并直接调用底层方法。

#### 2.2.1.返回类型为 void 的方法


这是配置具有 void 返回类型的方法以异步运行的简单方法：

```java
		@Async
    public void asyncMethodWithVoidReturnType() {
        log.info("Execute method asynchronously. " + Thread.currentThread().getName());
    }
```

#### 2.2.2.具有返回类型的方法


我们还可以通过将实际返回包装在 `Future` 中来将 `@Async` 应用于具有返回类型的方法：

```java
		@Async
    public Future<String> asyncMethodWithReturnType() {
        log.info("Execute method asynchronously " + Thread.currentThread().getName());
        try {
            Thread.sleep(5000);
            return new AsyncResult<>("hello world !!!!");
        } catch (final InterruptedException e) {
        }
        return null;
    }
```


Spring还提供了一个实现`Future`的`AsyncResult`类。我们可以用它来跟踪异步方法执行的结果。


现在让我们调用上述方法并使用 `Future` 对象检索异步过程的结果。

```java
	@Test
	public void testAsyncAnnotationForMethodsWithReturnType() throws InterruptedException, ExecutionException {
		log.info("Start - invoking an asynchronous method. " + Thread.currentThread().getName());
		final Future<String> future = asyncService.asyncMethodWithReturnType();

		while (true) {
			if (future.isDone()) {
				log.info("Result from asynchronous process - " + future.get());
				break;
			}
			log.info("Continue doing something else. ");
			Thread.sleep(1000);
		}
	}
```

### 2.3. Executor

默认情况下，Spring 会寻找一个 thread pool Bean：

- 一个唯一的 `org.springframework.core.task.TaskExecutor` Bean。如果存在多个名称的 Bean，则会使用名称为 taskExecutor 的 `java.util.concurrent.Executor `Bean 
- 或者一个名称为 `taskExecutor` 的 `java.util.concurrent.Executor` Bean

如果上面两个都没有找到，则使用 `org.springframework.core.task.SimpleAsyncTaskExecutor` 处理异步调用。

此外，带有void返回类型的注解方法无法将任何异常传递回调用者。默认情况下，这样的未捕获异常只会被记录在日志中。

为了自定义异步操作的配置，你可以实现`AsyncConfigurer`接口并提供以下内容：

- 通过`getAsyncExecutor()`方法提供自定义的执行器。

- 通过`getAsyncUncaughtExceptionHandler()`方法提供自定义的异步未捕获异常处理器（`AsyncUncaughtExceptionHandler`）。



#### 2.3.1.在方法级别重写Executor

1、方法一：在配置类中声明一个执行器：

```java
@Configuration
@EnableAsync
public class SpringAsyncConfig {
    
    @Bean(name = "threadPoolTaskExecutor")
    public Executor threadPoolTaskExecutor() {
        return new ThreadPoolTaskExecutor();
    }
}
```


然后我们应该在 `@Async` 中提供执行程序名称作为属性：

```java
@Async("threadPoolTaskExecutor")
public void asyncMethodWithConfiguredExecutor() {
    System.out.println("Execute method with configured executor - "
      + Thread.currentThread().getName());
}
```

@Async 注解如果指定了 Bean 的名称，则会查找该名称的执行器；如果没有指定 Bean 的名称，则会查找默认的执行器。



2、方法二：声明一个名称为 `taskExecutor` 的执行器，即 `AsyncAnnotationBeanPostProcessor` 中常量 `DEFAULT_TASK_EXECUTOR_BEAN_NAME` 的值。

```java
@Configuration
@EnableAsync
public class SpringAsyncConfig {
    
    @Bean(name = "taskExecutor")
    public Executor threadPoolTaskExecutor() {
        return new ThreadPoolTaskExecutor();
    }
}
```



#### 2.3.2.在应用程序级别覆盖Executor


配置类应实现 `AsyncConfigurer` 接口。因此，它必须实现 `getAsyncExecutor()` 方法。在这里，我们将返回整个应用程序的执行器。现在，它成为运行用 `@Async` 注解的方法的默认执行器：

```java
@Configuration
@EnableAsync
public class SpringAsyncConfig implements AsyncConfigurer {
   @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor threadPoolTaskExecutor = new ThreadPoolTaskExecutor();
        threadPoolTaskExecutor.initialize();
        return threadPoolTaskExecutor;
    }
}
```

### 2.4. 异常处理


当方法返回类型是 `Future` 时，异常处理很容易。 `Future.get()` 方法将抛出异常。


但如果返回类型为 void，异常将不会传播到调用线程。因此，我们需要添加额外的配置来处理异常。


我们将通过实现 `AsyncUncaughtExceptionHandler` 接口来创建自定义异步异常处理程序。当存在任何未捕获的异步异常时，将调用 `handleUncaughtException()` 方法：

```java
public class CustomAsyncExceptionHandler
  implements AsyncUncaughtExceptionHandler {

    @Override
    public void handleUncaughtException(
      Throwable throwable, Method method, Object... obj) {
 
        System.out.println("Exception message - " + throwable.getMessage());
        System.out.println("Method name - " + method.getName());
        for (Object param : obj) {
            System.out.println("Parameter value - " + param);
        }
    }
    
}
```


在上一节中，我们了解了配置类实现的 `AsyncConfigurer` 接口。作为其中的一部分，我们还需要重写 `getAsyncUncaughtExceptionHandler() ` 方法以返回我们的自定义异步异常处理程序：

```java
@Override
public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
    return new CustomAsyncExceptionHandler();
}
```



## 参考文档

- https://blog.chensoul.com/posts/2023/08/25/spring-async/