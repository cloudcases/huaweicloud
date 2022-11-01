# 高可用框架Resilience4j使用指南

30 人赞同了该文章

## **介绍**

Hystrix停更之后，Netflix官方推荐移步至[resilience4j](https://link.zhihu.com/?target=https%3A//github.com/resilience4j/resilience4j)，它是一个轻量、易用、可组装的高可用框架，支持***熔断***、***高频控制***、***隔离***、***限流***、***限时***、***重试***等多种高可用机制。



与Hystrix相比，它有以下一些主要的区别：

1. Hystrix调用必须被封装到HystrixCommand里，而resilience4j以装饰器的方式提供对函数式接口、lambda表达式等的嵌套装饰，因此你可以用简洁的方式组合多种高可用机制
2. Hystrix的频次统计采用滑动窗口的方式，而resilience4j采用环状缓冲区的方式
3. 关于熔断器在半开状态时的状态转换，Hystrix仅使用一次执行判定是否进行状态转换，而resilience4j则采用可配置的执行次数与阈值，来决定是否进行状态转换，这种方式提高了熔断机制的稳定性
4. 关于隔离机制，Hystrix提供基于线程池和信号量的隔离，而resilience4j只提供基于信号量的隔离



下面依次介绍各类高可用机制的使用方式【选译官方使用文档】。



## **熔断 CircuitBreaker**

## 初始化熔断器

CircuitBreakerRegistry负责创建和管理熔断器实例CircuitBreaker，它是线程安全的，提供原子性操作。

```text
CircuitBreakerRegistry circuitBreakerRegistry = CircuitBreakerRegistry.ofDefaults();
```

该方式使用默认的全局配置CircuitBreakerConfig创建熔断器实例，你也可以选择使用定制化的配置，可选项有：

1. 触发熔断的失败率阈值
2. 熔断器从打开状态到半开状态的等待时间
3. 熔断器在半开状态时环状缓冲区的大小
4. 熔断器在关闭状态时环状缓冲区的大小
5. 处理熔断器事件的定制监听器CircuitBreakerEventListener
6. 评估异常是否被记录为失败事件的定制谓词函数Predicate

```text
// 创建定制化熔断器配置
CircuitBreakerConfig circuitBreakerConfig = CircuitBreakerConfig.custom()
    .failureRateThreshold(50)
    .waitDurationInOpenState(Duration.ofMillis(1000))
    .ringBufferSizeInHalfOpenState(2)
    .ringBufferSizeInClosedState(2)
    .build();

// 使用定制化配置创建熔断器注册中心
CircuitBreakerRegistry circuitBreakerRegistry = CircuitBreakerRegistry.of(circuitBreakerConfig);

// 从注册中心获取使用默认配置的熔断器
CircuitBreaker circuitBreaker2 = circuitBreakerRegistry.circuitBreaker("otherName");

// 从注册中心获取使用定制化配置的熔断器
CircuitBreaker circuitBreaker = circuitBreakerRegistry.circuitBreaker("uniqueName", circuitBreakerConfig);
```

你也可以选择不经过注册中心，直接创建熔断器实例：

```text
CircuitBreaker defaultCircuitBreaker = CircuitBreaker.ofDefaults("testName");

CircuitBreaker customCircuitBreaker = CircuitBreaker.of("testName", circuitBreakerConfig);
```



## 熔断器使用方式

熔断器采用装饰器模式，你可以使用CircuitBreaker.decorateCheckedSupplier() / CircuitBreaker.decorateCheckedRunnable() / CircuitBreaker.decorateCheckedFunction() 装饰 Supplier / Runnable / Function / CheckedRunnable / CheckedFunction，然后使用Vavr的Try.of(…) / Try.run(…) 调用被装饰的函数，也可以使用map / flatMap / filter / recover / andThen链接更多的函数。函数链只有在熔断器处于关闭或半开状态时才可以被调用。

```text
// 创建熔断器
CircuitBreaker circuitBreaker = CircuitBreaker.ofDefaults("testName");

// 用熔断器包装函数
CheckedFunction0<String> decoratedSupplier = CircuitBreaker
        .decorateCheckedSupplier(circuitBreaker, () -> "This can be any method which returns: 'Hello");

// 链接其它的函数
Try<String> result = Try.of(decoratedSupplier)
                .map(value -> value + " world'");

// 如果函数链中的所有函数均调用成功，最终结果为Success<String>
assertThat(result.isSuccess()).isTrue();
assertThat(result.get()).isEqualTo("This can be any method which returns: 'Hello world'");
```



函数链中可以包含被不同熔断器包装的多个函数：

```text
// 两个熔断器
CircuitBreaker circuitBreaker = CircuitBreaker.ofDefaults("testName");
CircuitBreaker anotherCircuitBreaker = CircuitBreaker.ofDefaults("anotherTestName");

// 用两个熔断器分别包装Supplier 和 Function
CheckedFunction0<String> decoratedSupplier = CircuitBreaker
        .decorateCheckedSupplier(circuitBreaker, () -> "Hello");

CheckedFunction1<String, String> decoratedFunction = CircuitBreaker
        .decorateCheckedFunction(anotherCircuitBreaker, (input) -> input + " world");

// 链接函数
Try<String> result = Try.of(decoratedSupplier)
        .mapTry(decoratedFunction::apply);

assertThat(result.isSuccess()).isTrue();
assertThat(result.get()).isEqualTo("Hello world");
```



下面我们模拟熔断器被触发的情况，在熔断器打开的状态，Try.of 返回 Failure<Throwable>，链接函数将不会被调用：

```text
// 创建一个环状缓冲区大小为2的熔断器
CircuitBreakerConfig circuitBreakerConfig = CircuitBreakerConfig.custom()
        .ringBufferSizeInClosedState(2)
        .waitDurationInOpenState(Duration.ofMillis(1000))
        .build();
CircuitBreaker circuitBreaker = CircuitBreaker.of("testName", circuitBreakerConfig);

// 模拟一次失败调用
circuitBreaker.onError(0, new RuntimeException());
// 没有触发熔断，熔断器仍处于关闭状态
assertThat(circuitBreaker.getState()).isEqualTo(CircuitBreaker.State.CLOSED);
// 模拟第二次失败调用
circuitBreaker.onError(0, new RuntimeException());
// 失败率超过百分之五十，熔断器被触发
assertThat(circuitBreaker.getState()).isEqualTo(CircuitBreaker.State.OPEN);

// 由于熔断器处于打开状态，调用失败
Try<String> result = Try.of(CircuitBreaker.decorateCheckedSupplier(circuitBreaker, () -> "Hello"))
        .map(value -> value + " world");

assertThat(result.isFailure()).isTrue();
assertThat(result.failed().get()).isInstanceOf(CircuitBreakerOpenException.class);
```



熔断器支持RxJava/Reactor，下面是简单的示例：

```text
CircuitBreaker circuitBreaker = CircuitBreaker.ofDefaults("backendName");
Observable.fromCallable(backendService::doSomething)
.lift(CircuitBreakerOperator.of(circuitBreaker))

CircuitBreaker circuitBreaker = CircuitBreaker.ofDefaults("backendName");
Mono.fromCallable(backendService::doSomething)
    .transform(CircuitBreakerOperator.of(circuitBreaker))
```



## 熔断器重置

熔断器支持重置，重置之后所有状态数据清空，恢复初始状态。

```text
circuitBreaker.reset();
```



## 熔断器降级

你可以为熔断函数设置降级措施，链接Try.recover()，当Try.of() 返回 Failure<Throwable>时，使用降级措施：

```text
// 创建熔断器
CircuitBreaker circuitBreaker = CircuitBreaker.ofDefaults("testName");

// 模拟失败调用，并链接降级函数
CheckedFunction0<String> checkedSupplier = CircuitBreaker.decorateCheckedSupplier(circuitBreaker, () -> {
 throw new RuntimeException("BAM!");
});
Try<String> result = Try.of(checkedSupplier)
        .recover(throwable -> "Hello Recovery");

// 降级函数被调用，最终调用结果为成功
assertThat(result.isSuccess()).isTrue();
assertThat(result.get()).isEqualTo("Hello Recovery");
```



## 熔断器失败判定

默认所有异常都被认定为失败事件，你可以定制Predicate的test检查，实现选择性记录，只有该函数返回为true时异常才被认定为失败事件。下例展示了如何忽略除WebServiceException外的所有异常：

```text
CircuitBreakerConfig circuitBreakerConfig = CircuitBreakerConfig.custom()
        .ringBufferSizeInClosedState(2)
        .ringBufferSizeInHalfOpenState(2)
        .waitDurationInOpenState(Duration.ofMillis(1000))
        .recordFailure(throwable -> Match(throwable).of(
                Case($(instanceOf(WebServiceException.class)), true),
                Case($(), false)))
        .build();
```



## 监听熔断器事件

熔断器事件CircuitBreakerEvent包含状态转换、重置、成功、失败异常、忽略异常事件，所有事件包含发生时间、处理时长的信息。你可以使用如下方式注册事件监听器：

```text
// 分类监听事件
circuitBreaker.getEventPublisher()
    .onSuccess(event -> logger.info(...))
    .onError(event -> logger.info(...))
    .onIgnoredError(event -> logger.info(...))
    .onReset(event -> logger.info(...))
    .onStateTransition(event -> logger.info(...));
// 监听所有事件
circuitBreaker.getEventPublisher()
    .onEvent(event -> logger.info(...));
```

你也可以使用CircularEventConsumer将事件存储在缓存中：

```text
CircularEventConsumer<CircuitBreakerEvent> ringBuffer = new CircularEventConsumer<>(10);
circuitBreaker.getEventPublisher().onEvent(ringBuffer);
List<CircuitBreakerEvent> bufferedEvents = ringBuffer.getBufferedEvents()
```

你还可以将EventPublisher转换为RxJava/Reactor的事件流，该方法的优势在于，可以进一步指定调度器进行异步化处理：

```text
RxJava2Adapter.toFlowable(circuitBreaker.getEventPublisher())
    .filter(event -> event.getEventType() == Type.ERROR)
    .cast(CircuitBreakerOnErrorEvent.class)
    .subscribe(event -> logger.info(...))
```



## 熔断器状态

```text
CircuitBreaker.Metrics metrics = circuitBreaker.getMetrics();
float failureRate = metrics.getFailureRate();
int bufferedCalls = metrics.getNumberOfBufferedCalls();
int failedCalls = metrics.getNumberOfFailedCalls();
```



## **高频控制 RateLimiter**

## 初始化高频控制器

高频控制的配置方式与熔断类似，有对应的RateLimiterRegistry 和 RateLimiterConfig，自定义配置的可选项有：

1. 频次阈值
2. 阈值刷新时间
3. 限流后的冷却时间

```text
// 创建高频控制配置，控制频率为1QPS
RateLimiterConfig config = RateLimiterConfig.custom()
    .limitForPeriod(1)
    .limitRefreshPeriod(Duration.ofMillis(1000))
    .timeoutDuration(Duration.ofMillis(500))
    .build();

// 创建高频控制注册中心
RateLimiterRegistry rateLimiterRegistry = RateLimiterRegistry.of(config);

// 从注册中心创建高频控制器实例
RateLimiter rateLimiterWithDefaultConfig = rateLimiterRegistry.rateLimiter("backend");
RateLimiter rateLimiterWithCustomConfig = rateLimiterRegistry.rateLimiter("backend#2", config);

// 直接创建高频控制器实例
RateLimiter rateLimiter = RateLimiter.of("NASDAQ :-)", config);
```



## 高频控制器使用方式

```text
// 使用上面定义的高频控制器装饰函数调用
CheckedRunnable restrictedCall = RateLimiter
    .decorateCheckedRunnable(rateLimiter, () -> System.out.println("Do something"));

// 第一次调用成功，第二次调用被高频限制
Try.run(restrictedCall)
    .andThenTry(restrictedCall)
    .onFailure(throwable -> System.out.println("Wait before call it again :)"));

你可以在运行时动态修改高频控制器配置，但新的冷却时间不会影响当前处于冷却状态的线程，新的阈值也不会影响处于当前一轮控制的线程：

// 在下一轮控制中，阈值变更为100
rateLimiter.changeLimitForPeriod(100);
```

与熔断类似，高频控制也支持RxJava/Reactor.

```text
RateLimiter rateLimiter = RateLimiter.ofDefaults("backendName");
Observable.fromCallable(backendService::doSomething)
    .lift(RateLimiterOperator.of(rateLimiter));

RateLimiter rateLimiter = RateLimiter.ofDefaults("backendName");
Mono.fromCallable(backendService::doSomething)
    .transform(RateLimiterOperator.of(rateLimiter));
```



## 监听高频控制事件

高频控制器事件RateLimiterEvent包含允许执行和拒绝执行事件，所有事件包含发生时间、相关高频控制器名称的信息。

```text
rateLimiter.getEventPublisher()
    .onSuccess(event -> logger.info(...))
    .onFailure(event -> logger.info(...));
```

将EventPublisher转换为RxJava/Reactor的事件流：

```text
RxJava2Adapter.toFlowable(rateLimiter.getEventPublisher())
    .filter(event -> event.getEventType() == FAILED_ACQUIRE)
    .subscribe(event -> logger.info(...))
```



## 高频控制器状态

```text
RateLimiter limit;
RateLimiter.Metrics metrics = limit.getMetrics();
int numberOfThreadsWaitingForPermission = metrics.getNumberOfWaitingThreads();
int availablePermissions = metrics.getAvailablePermissions();

AtomicRateLimiter atomicLimiter;
long nanosToWaitForPermission = atomicLimiter.getNanosToWait();
```



## **隔离&流控 Bulkhead**

## 介绍

Bulkhead意指船舶中的隔舱板，它将船体分割为多个船舱，在船部分受损时可避免沉船。框架中的Bulkhead通过信号量的方式隔离不同种类的调用，并进行流控，这样可以避免某类调用异常危及系统整体。（注：不同于Hystrix，该框架不提供基于线程池的隔离）



## 初始化Bulkhead

Bulkhead的配置方式与熔断类似，有对应的BulkheadRegistry 和 BulkheadConfig，自定义配置的可选项有：

1. 最大并行度
2. 尝试进入饱和态的Bulkhead时，线程的最大阻塞时间

```text
// 创建自定义的Bulkhead配置
BulkheadConfig config = BulkheadConfig.custom()
                                      .maxConcurrentCalls(150)
                                      .maxWaitTime(100)
                                      .build();

// 创建Bulkhead注册中心
BulkheadRegistry registry = BulkheadRegistry.of(config);

// 从注册中心获取默认配置的Bulkhead
Bulkhead bulkhead1 = registry.bulkhead("foo");

// 从注册中心获取自定义配置的Bulkhead
Bulkhead bulkhead2 = registry.bulkhead("bar", config);
```

你也可以不经过注册中心，直接创建Bulkhead：

```text
Bulkhead bulkhead1 = Bulkhead.ofDefaults("foo");
Bulkhead bulkhead2 = Bulkhead.of(
                         "bar",
                         BulkheadConfig.custom()
                                       .maxConcurrentCalls(50)
                                       .build()
                     );
```



## Bulkhead使用方式

与熔断类似，不再赘述，下面是代码示例：

```text
// 创建Bulkhead实例
Bulkhead bulkhead = Bulkhead.of("testName", config);

// 用Bulkhead包装函数调用
CheckedFunction0<String> decoratedSupplier = Bulkhead.decorateCheckedSupplier(bulkhead, () -> "This can be any method which returns: 'Hello");

// 链接其它函数
Try<String> result = Try.of(decoratedSupplier)
                        .map(value -> value + " world'");

assertThat(result.isSuccess()).isTrue();
assertThat(result.get()).isEqualTo("This can be any method which returns: 'Hello world'");
assertThat(bulkhead.getMetrics().getAvailableConcurrentCalls()).isEqualTo(1);
```



函数链中可以包含被不同Bulkhead或熔断器包装的多个函数：

```text
// 两个Bulkhead
Bulkhead bulkhead = Bulkhead.of("test", config);
Bulkhead anotherBulkhead = Bulkhead.of("testAnother", config);

// 用两个Bulkhead分别包装Supplier 和 Function
CheckedFunction0<String> decoratedSupplier
    = Bulkhead.decorateCheckedSupplier(bulkhead, () -> "Hello");
CheckedFunction1<String, String> decoratedFunction
    = Bulkhead.decorateCheckedFunction(anotherBulkhead, (input) -> input + " world");

// 链接函数
Try<String> result = Try.of(decoratedSupplier)
                        .mapTry(decoratedFunction::apply);

assertThat(result.isSuccess()).isTrue();
assertThat(result.get()).isEqualTo("Hello world");
assertThat(bulkhead.getMetrics().getAvailableConcurrentCalls()).isEqualTo(1);
assertThat(anotherBulkhead.getMetrics().getAvailableConcurrentCalls()).isEqualTo(1);
```



下面我们模拟饱和态Bulkhead的行为：

```text
// 创建并行度为2的Bulkhead
BulkheadConfig config = BulkheadConfig.custom().maxConcurrentCalls(2).build();
Bulkhead bulkhead = Bulkhead.of("test", config);
// 该方法会进入Bulkhead
bulkhead.isCallPermitted();
bulkhead.isCallPermitted();

// 经过上面两次调用，Bulkhead已饱和
CheckedRunnable checkedRunnable = Bulkhead.decorateCheckedRunnable(bulkhead, () -> {throw new RuntimeException("BAM!");});
Try result = Try.run(checkedRunnable);

assertThat(result.isFailure()).isTrue();
assertThat(result.failed().get()).isInstanceOf(BulkheadFullException.class);
```



你可以在运行时动态修改Bulkhead配置，但新的最大阻塞时间不会影响当前正在等待的线程。

与熔断类似，Bulkhead也支持RxJava/Reactor.

```text
Bulkhead bulkhead = Bulkhead.ofDefaults("backendName");
Observable.fromCallable(backendService::doSomething)
          .lift(BulkheadOperator.of(bulkhead));

Bulkhead bulkhead = Bulkhead.ofDefaults("backendName");
Mono.fromCallable(backendService::doSomething)
          .transform(BulkheadOperator.of(bulkhead));
```



## 监听Bulkhead事件

BulkHeadEvent包含允许执行、拒绝执行、执行结束事件，可以用如下方式监听：

```text
bulkhead.getEventPublisher()
    .onCallPermitted(event -> logger.info(...))
    .onCallRejected(event -> logger.info(...))
    .onCallFinished(event -> logger.info(...));
```



## Bulkhead状态

```text
Bulkhead.Metrics metrics = bulkhead.getMetrics();
in remainingBulkheadDepth = metrics.getAvailableConcurrentCalls()
```



## **限时 TimeLimiter**

## 使用方式

限时器与Future配合使用，限时器将Future Supplier转换为Callable，它将尝试在给定时间内获取Future的值，如果超时未获取到，Future将会被取消：

```text
// 创建限时器配置，最大执行时间为60s，超时将取消Future
TimeLimiterConfig config = TimeLimiterConfig.custom()
    .timeoutDuration(Duration.ofSeconds(60))
    .cancelRunningFuture(true)
    .build();

// 从该配置创建限时器
TimeLimiter timeLimiter = TimeLimiter.of(config);

ExecutorService executorService = Executors.newSingleThreadExecutor();

// 将待执行任务提交到线程池，获取Future Supplier
Supplier<Future<Integer>> futureSupplier = () -> executorService.submit(backendService::doSomething)

// 使用限时器包装Future Supplier
Callable restrictedCall = TimeLimiter
    .decorateFutureSupplier(timeLimiter, futureSupplier);

// 若任务执行超时，onFailure会被触发
Try.of(restrictedCall.call)
    .onFailure(throwable -> LOG.info("A timeout possibly occurred."));
```

你可以将限时器与熔断器配合使用，在超时次数过多时触发熔断：

```text
Callable restrictedCall = TimeLimiter
    .decorateFutureSupplier(timeLimiter, futureSupplier);

Callable chainedCallable = CircuitBreaker.decorateCallable(circuitBreaker, restrictedCall);

Try.of(chainedCallable::call)
    .onFailure(throwable -> LOG.info("We might have timed out or the circuit breaker has opened."));
```



## **重试 Retry**

## 使用方式

可配置选项有：

1. 最大重试次数
2. 重试间隔
3. 评估是否重试的Predicate

```text
// 最多重试2次
// 重试间隔为100ms
// 当执行出现WebServiceException时触发重试
RetryConfig config = RetryConfig.custom()
    .maxAttempts(2)
    .waitDuration(Duration.ofMillis(100))
    .retryOnException(throwable -> API.Match(throwable).of(
            API.Case($(Predicates.instanceOf(WebServiceException.class)), true),
            API.Case($(), false)))
    .build();

// 从重试配置创建重试器
Retry retry = Retry.of("id", config); 
// 使用重试器包装函数调用
CheckedFunction0<String> retryableSupplier = Retry.decorateCheckedSupplier(retry, () -> {throw new WebServiceException("BAM!");});
// 调用将会自动重试
Try<String> result = Try.of(retryableSupplier).recover((throwable) -> "Hello world from recovery function");

assertThat(result.get()).isEqualTo("Hello world from recovery function");
```



## **Maven依赖**

```text
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-circuitbreaker</artifactId>
    <version>0.13.1</version>
</dependency>
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-ratelimiter</artifactId>
    <version>0.13.1</version>
</dependency>
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-retry</artifactId>
    <version>0.13.1</version>
</dependency>
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-bulkhead</artifactId>
    <version>0.13.1</version>
</dependency>
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-cache</artifactId>
    <version>0.13.1</version>
</dependency>
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-timelimiter</artifactId>
    <version>0.13.1</version>
</dependency>
```



编辑于 2018-11-28 08:36

[高可用](https://www.zhihu.com/topic/19671201)

[分布式系统](https://www.zhihu.com/topic/19570823)