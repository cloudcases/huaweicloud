## SpringCloud之Resilience4J用法精讲

在微服务中，经常会出现一些故障，而一些故障会直接或者间接的拖垮其它的服务，造成服务器雪崩，系统就会死掉。

什么是服务雪崩？我们可以通过下面一张图来看：
![在这里插入图片描述](http://img.5iqiqu.com/images5/07/076c689b4221b63b504d1a09636d24fb.png)
假如现在有很多的用户同时请求订单微服务去执行下单的操作，那么会调用我们的支付微服务，如果支付微服务现在挂掉了，而订单调用一直没有响应，由于很多的用户执行相同的操作，属于高并发，那么服务器上积累的订单越来越多，那么原来没有问题的订单微服务，也会被拖垮，这就是服务雪崩。

我们需要做的就是，**当某一个微服务发生蔓延当时候，不能发生故障蔓延**，整个系统还能以其它某种方式正常运行，这个就是我们需要解决的。

# 断路器

我们耳熟能详的就是Netflix Hystrix,这个断路器是SpringCloud中最早支持的一种容错方案，现在这个断路器已经处于维护状态，已经不再更新了，你仍然可以使用这个断路器，但是呢，我不建议你去使用，因为这个已经不再更新，所以Spring官方已经出现了Netflix Hystrix的替换方案。 如下图：
![在这里插入图片描述](http://img.5iqiqu.com/images3/7b/7b48e607bb32040ec02887d2f9345805.png)
在 Spring Cloud Greenwich 版中，对于 Hystrix 以及 Hystrix Dashboard 官方都给出了替代方案。我们整个教程虽然基于最新的 Spring Cloud Greenwich 版，但是考虑到现实情况，本文中我还是先向大家大致介绍一下 Hystrix 的功能，后面我们会详细介绍 Resilience4j 的用法。

# 服务熔断

什么是服务熔断呢？服务熔断就是当A服务去调用B服务，如果A服务迟迟没有收到B服务的响应，那么就终断当前的请求，而不是一直等待下去，一直等待下去的结果就是拖垮其它的服务。当系统发生熔断的时候，我们还要去监控B服务，当B服务恢复正常使用时，A服务就发起重新调用的请求。

# 服务降级

当我们的服务发生熔断的时候，那么就需要降级了，那么什么是降级？降级指的是A服务调用B服务，没有调用成功，发生熔断，那么A服务就不要死板的一直请求B服务，而是去服务上哪一个缓存先顶着，避免给我们的用户，响应一些错误的页面，这个就是服务降级。

# 请求缓存

请求缓存是指对接口进行缓存，这样可以大大降低服务提供者的压力，当然我们要选择缓存使用的场景，是那种更新频率低，但是访问又比较频繁的数据。

我们这里说的缓存啊，指的是Hystrix的缓存，但是在实际的开发中，我们可能会配合其它的缓存来实现更好的效果，如redis。

# 请求合并

我们知道SpringCloud中的微服务之间的调用都是通过HTTP来实现的，但是我们通过HTTP协议调用微服务时候，如果是高并发数量小的话，那么效率很低，那么我们可以通过合并请求来实现，也就是将客户端多个请求合并成一个请求，也就是只发送一个HTTP请求，服务器拿到请求结果后，再将请求结果分发给不同的请求，这样就可以提供传输效率。

# Resilience4J

Resilience4J是我们Spring Cloud G版本 推荐的容错方案，它是一个轻量级的容错库。它呢借鉴了Hystrix而设计，并且采用JDK8 这个函数式编程，也就是我们的lambda表达式，为什么说它是轻量级的呢？因为它的库只使用 Vavr （以前称为 Javaslang ），它没有任何其他外部库依赖项。相比之下， Netflix Hystrix 对Archaius 具有编译依赖性，这导致了更多的外部库依赖，例如 Guava 和 Apache Commons 。而如果使用Resilience4j，你无需引用全部依赖，可以根据自己需要的功能引用相关的模块即可。

Resilience4J 提供了一系列增强微服务的可用性功能：

1. 断路器
2. 限流
3. 基于信号量的隔离
4. 缓存
5. 限时
6. 请求重启

那么我们接下来就讲解Resilience4J的几种功能的使用方法，由于是基本用法所以我们不用创建SpringBoot工程，我们只需要创建一个叫Resilience4j的普通maven工程并加入 junit 单元测试依赖，这样准备工作就完成了。

junit单元测试

```
1  <dependency>
2            <groupId>junit</groupId>
3            <artifactId>junit</artifactId>
4            <version>4.12</version>
5  </dependency>
6
7
```

# 断路器初始化

我们使用的是Resilience4J 提供的断路器功能，那么就要加入依赖

```
1
2  <dependency>
3         <groupId>io.github.resilience4j</groupId>
4         <artifactId>resilience4j-circuitbreaker</artifactId>
5         <version>0.13.2</version>
6  </dependency>
7
8
```

这个依赖它提供的是一个基于ConcurrentHashMap 的 CircuitBreakerRegistry ，CircuitBreakerRegistry 是线程安全的，并且是原子操作。开发者可以使用 CircuitBreakerRegistry 来创建和检索 CircuitBreaker 的实例 ，开发者可以直接使用默认的全局CircuitBreakerConfig 为所有 CircuitBreaker 实例创建 CircuitBreakerRegistry ，如下所示：



```
1CircuitBreakerRegistry circuitBreakerRegistry = CircuitBreakerRegistry.ofDefaults();
2
3
4
```

我们使用自定义的CircuitBreakerConfig，可以配置如下参数：

1. 故障率阈值百分比，超过这个阈值，断路器就会打开
2. 断路器保持打开的时间，在到达设置的时间之后，断路器会进入到 half open 状态
3. 当断路器处于 half open 状态时，环形缓冲区的大小
4. 当断路器关闭时，环形缓冲区的大小
5. 自定义断路器中的事件操作
6. 自定义 Predicate 以便计算异常是否被记录为失败事件

具体的定义如下：

```
1CircuitBreakerConfig circuitBreakerConfig = CircuitBreakerConfig.custom()
2    .failureRateThreshold(50)
3    .waitDurationInOpenState(Duration.ofMillis(1000))
4    .ringBufferSizeInHalfOpenState(2)
5    .ringBufferSizeInClosedState(2)
6    .build();
7CircuitBreakerRegistry circuitBreakerRegistry = CircuitBreakerRegistry.of(circuitBreakerConfig);
8CircuitBreaker circuitBreaker2 = circuitBreakerRegistry.circuitBreaker("otherName");
9CircuitBreaker circuitBreaker = circuitBreakerRegistry.circuitBreaker("uniqueName", circuitBreakerConfig);
10
11
```

上面这段代码解释如下：
我们首先定义了一个CircuitBreakerConfig 对象，在定义CircuitBreakerConfig对象时，设置故障率为50%，断路器 保持打开时间为2秒，断路器处于half open的时候，缓冲区大小为2，当对象处于关闭时，缓冲区的大小也是2，然后根据CircuitBreakerConfig对象创建
CircuitBreakerRegistry，然后再根据CircuitBreakerRegistry 创建两个断路器CircuitBreaker。



如果不想使用CircuitBreakerRegistry来管理断路器 那么可以直接创建CircuitBreaker对象



```
1CircuitBreaker defaultCircuitBreaker = CircuitBreaker.ofDefaults("testName");
2CircuitBreaker customCircuitBreaker = CircuitBreaker.of("testName", circuitBreakerConfig);
3
4
```

# 断路器的使用案例

断路器使用了装饰者模式，开发者可以使用 CircuitBreaker.decorateCheckedSupplier(), CircuitBreaker.decorateCheckedRunnable() 或者 CircuitBreaker.decorateCheckedFunction() 来装饰 Supplier / Runnable / Function 或者 CheckedRunnable / CheckedFunction，然后使用 Try.of(…) 或者 Try.run(…) 来进行调用操作，也可以使用 map、flatMap、filter、recover 或者 andThen 进行链式调用，但是调用这些方法断路器必须处于 CLOSED 或者 HALF_OPEN 状态。例如下面一个例子，创建一个断路器出来，首先装饰了一个函数，这个函数返回一段字符串，然后使用 Try.of 去执行，执行完后再进入到 map 中去执行。如果第一个函数正常执行第二个函数才会执行，如果第一个函数执行失败，那么 map 函数将不会执行：

```
1CircuitBreaker circuitBreaker = CircuitBreaker.ofDefaults("testName");
2CheckedFunction0<String> decoratedSupplier = CircuitBreaker
3        .decorateCheckedSupplier(circuitBreaker, () -> "This can be any method which returns: 'Hello");
4Try<String> result = Try.of(decoratedSupplier)
5        .map(value -> value + " world'");
6System.out.println(result.isSuccess());
7System.out.println(result.get());
8
9
```

你可以将不同的断路器连接起来：

```
1CircuitBreaker circuitBreaker = CircuitBreaker.ofDefaults("testName");
2CircuitBreaker anotherCircuitBreaker = CircuitBreaker.ofDefaults("anotherTestName");
3CheckedFunction0<String> decoratedSupplier = CircuitBreaker
4        .decorateCheckedSupplier(circuitBreaker, () -> "Hello");
5CheckedFunction1<String, String> decoratedFunction = CircuitBreaker
6        .decorateCheckedFunction(anotherCircuitBreaker, (input) -> input + " world");
7Try<String> result = Try.of(decoratedSupplier)
8        .mapTry(decoratedFunction::apply);
9System.out.println(result.isSuccess());
10System.out.println(result.get());
11
12
```

# 断路器的打开

这里创建了两个 CircuitBreaker ，装饰了两个函数，第二次使用了 mapTry 方法来连接。前面给大家演示的几种情况，都是执行成功的，即断路器一直处于关闭的状态，接下来给大家再来演示一个断路器打开的例子，如下：

```
1CircuitBreakerConfig circuitBreakerConfig = CircuitBreakerConfig.custom()
2        .ringBufferSizeInClosedState(2)
3        .waitDurationInOpenState(Duration.ofMillis(1000))
4        .build();
5CircuitBreaker circuitBreaker = CircuitBreaker.of("testName", circuitBreakerConfig);
6circuitBreaker.onError(0, new RuntimeException());
7System.out.println(circuitBreaker.getState());
8circuitBreaker.onError(0, new RuntimeException());
9System.out.println(circuitBreaker.getState());
10Try<String> result = Try.of(CircuitBreaker.decorateCheckedSupplier(circuitBreaker, () -> "Hello"))
11        .map(value -> value + " world");
12System.out.println(result.isSuccess());
13System.out.println(result.get());
14
15
16
```



这里手动模拟错误，首先设置了断路器关闭状态下的环形缓冲区大小为 2 ，即当有两条数据时就可以去统计故障率了，这里没有设置故障率，默认的故障率是 50% ，当第一次调用 onError 方法后，打印断路器当前状态，发现断路器还是处于关闭状态，并未打开，接下来再次调用 onError 方法，然后再去查看断路器状态，此时发现断路器已经打开了，因为满足了 50% 的故障率了。

# 断路器重置

断路器的重置就是清空数据

```
1circuitBreaker.reset();
2
3
```

# 服务器请求降级

服务器降级操作如下：

```
1CircuitBreaker circuitBreaker = CircuitBreaker.ofDefaults("testName");
2CheckedFunction0<String> checkedSupplier = CircuitBreaker.decorateCheckedSupplier(circuitBreaker, () -> {
3    throw new RuntimeException("BAM!");
4});
5Try<String> result = Try.of(checkedSupplier)
6        .recover(throwable -> "Hello Recovery");
7System.out.println(result.isSuccess());
8System.out.println(result.get());
9
10
```

如果需要使用服务降级，可以使用 Try.recover() 链接，当 Try.of() 返回 Failure 时服务降级会被触发。

# 状态监听

状态监听可以获取到熔断器当前的运行数据，例如：

```
1CircuitBreaker.Metrics metrics = circuitBreaker.getMetrics();
2// 获取故障率
3float failureRate = metrics.getFailureRate();
4// 获取调用失败次数
5int failedCalls = metrics.getNumberOfFailedCalls();
6
7
```

# 限流

RateLimiter 和我们前面提到的断路器实际上非常类似，它也有一个基于内存的 RateLimiterRegistry 和 RateLimiterConfig 可以配置，我们可以配置如下一些参数：

1. 限流之后的冷却时间
2. 阈值刷新时间
3. 阈值刷新频次

使用限流我们要引入下面的依赖：

```
1<dependency>
2    <groupId>io.github.resilience4j</groupId>
3    <artifactId>resilience4j-ratelimiter</artifactId>
4    <version>0.13.2</version>
5</dependency>
6
7
```

**基本用法**

例如，想限制某个请求的频率为 2QPS（每秒处理两个请求），为什么给一个这样的频率呢？主要是为了大家一会儿测试方便，代码如下：

```
1RateLimiterConfig config = RateLimiterConfig.custom()
2        .limitRefreshPeriod(Duration.ofMillis(1000))
3        .limitForPeriod(2)
4        .timeoutDuration(Duration.ofMillis(1000))
5        .build();
6RateLimiterRegistry rateLimiterRegistry = RateLimiterRegistry.of(config);
7RateLimiter rateLimiterWithDefaultConfig = rateLimiterRegistry.rateLimiter("backend");
8RateLimiter rateLimiterWithCustomConfig = rateLimiterRegistry.rateLimiter("backend#2", config);
9RateLimiter rateLimiter = RateLimiter.of("NASDAQ :-)", config);
10
11
```



和前面的一样，我们也可以使用 RateLimiterRegistry 来统一管理 RateLimiter ，也可以通过 RateLimiter.of 方法来直接创建一个 RateLimiter。创建好了，就可以直接使用了，代码如下：

```
1CheckedRunnable restrictedCall = RateLimiter
2        .decorateCheckedRunnable(rateLimiter,()->{
3            System.out.println(new Date());
4        });
5Try.run(restrictedCall)
6        .andThenTry(restrictedCall)
7        .andThenTry(restrictedCall)
8        .andThenTry(restrictedCall)
9        .onFailure(throwable -> System.out.println(throwable.getMessage()));
10
11
```

执行结果如下：

![在这里插入图片描述](http://img.5iqiqu.com/images5/45/4580d248cc45ce922195ac795615f855.png)
可以观察上面，我们发现可以知道限流一次执行了两个方法，另外两个方法在1s过后执行的。并且限流参数是可以随便修改的，修改后，本次的限流周期内不会生效，下次限流才会生效执行。

修改限流如下：

```
1rateLimiter.changeLimitForPeriod(100);
2rateLimiter.changeTimeoutDuration(Duration.ofMillis(100));
3
4
```

# 事件监听

在限流中，我们可以获取所有允许和拒绝执行的事件信息，获取方式如下：

```
1rateLimiter.getEventPublisher()
2        .onSuccess(event -> {
3            System.out.println(new Date()+">>>"+event.getEventType()+">>>"+event.getCreationTime());
4        })
5        .onFailure(event -> {
6            System.out.println(new Date()+">>>"+event.getEventType()+">>>"+event.getCreationTime());
7        });
8
9
```

# 请求隔离

这里的请求隔离,主要是基于信号量的请求隔离，不包含基于线程的请求隔离，具体用法和前面两个类似，不过在使用之前，需要先添加请求隔离相关的依赖，如下：

```
1<dependency>
2    <groupId>io.github.resilience4j</groupId>
3    <artifactId>resilience4j-bulkhead</artifactId>
4    <version>0.13.2</version>
5</dependency>
6
7
```

定义最大并行数和饱和状态Bulkhead时 线程的最大阻塞时间 如下：

```
1  BulkheadConfig config = BulkheadConfig.custom()
2                .maxConcurrentCalls(150)
3                .maxWaitTime(100)
4                .build();
5        BulkheadRegistry registry = BulkheadRegistry.of(config);
6        Bulkhead bulkhead1 = registry.bulkhead("foo");
7        BulkheadConfig custom = BulkheadConfig.custom()
8                .maxWaitTime(0).build();
9
10
11        Bulkhead bulkhead2 = registry.bulkhead("bar", custom);
12        System.out.println(bulkhead1 + ">>>>>" + bulkhead2);
13    }
14
15
```

输出结果为：
![在这里插入图片描述](http://img.5iqiqu.com/images3/94/94becad76d78b5e9ba1a8303af9b3187.png)

如果不想通过BulkheadRegistry来管理Bulkhead的实例，那么我们可以直接创建Bulkhead如下：

```
1  Bulkhead bulkhead1 = Bulkhead.ofDefaults("foo");
2        Bulkhead bulkhead2 = Bulkhead.of("bar",
3                BulkheadConfig.custom().maxConcurrentCalls(50).build());
4        System.out.println(bulkhead1 + ">>>>>" + bulkhead2);
5
6
```

运行效果如下：
![在这里插入图片描述](http://img.5iqiqu.com/images5/94/94becad76d78b5e9ba1a8303af9b3187.png)
创建好来后，使用的方法和断路器是一样的：

```
1 BulkheadConfig config = BulkheadConfig.custom()
2                .maxConcurrentCalls(1)
3                .maxWaitTime(100)
4                .build();
5        Bulkhead bulkhead = Bulkhead.of("testName", config);
6        CheckedFunction0<String> decoratedSupplier = Bulkhead.decorateCheckedSupplier(
7                bulkhead,()-> "this is bulkhead: love "
8        );
9
10        Try<String> result = Try.of(decoratedSupplier).map(
11                value -> value + "小蕾"
12        );
13
14        System.out.println(result.isSuccess());
15        System.out.println(result.get());
16    }
17
18
```

执行结果如下：
![在这里插入图片描述](http://img.5iqiqu.com/images6/ea/eae09c5af61a28efe5349520e6e6e395.png)

# 请求重试

当我们的服务失败的时候，那么就需要请求重试，可以说请求重试是一个非常常用的功能，要使用请求 重试的话，那么要引入下面这个依赖：

```
1<dependency>
2    <groupId>io.github.resilience4j</groupId>
3    <artifactId>resilience4j-retry</artifactId>
4    <version>0.13.2</version>
5</dependency>
6
7
```

那么引入依赖后，我们创建一个重试的测试用例：

```
1  RetryConfig config = RetryConfig.custom()
2   .maxAttempts(3) //重试次数为3次
3   .waitDuration(Duration.ofMillis(500)) //每次重试间隔500毫秒
4   .build();
5  Retry retry = Retry.of("id", config);
6
7
```

创建好 Retry的实例后，我们就可以使用了，使用的步骤和断路器是一样的如下：

```
1 CheckedFunction0<String> retryAllSupplier = Retry.decorateCheckedSupplier(
2                retry, () -> {
3                    System.out.println("date:" + new Date() + ":" + Math.random());
4                    return "love 小蕾";
5                });
6
7        Try<String> result = Try.of(retryAllSupplier).recover((throwable -> "Hello world from recovery function"));
8        System.out.println(result.isSuccess());
9        System.out.println(result.get());
10
11
```

运行的结果如下：
![在这里插入图片描述](http://img.5iqiqu.com/images3/94/94bc9ada7d17281b49f41c1c13dcd933.png)

如果抛出了异常那么就会触发重试机制。

# 缓存

Resilience4J 提供了 JCache 缓存，但是我们实际开发用的是Redis缓存，这里就不多讲了，感兴趣的朋友，可以自己去学习。

# 限时

Resilience4j 中的限时器是要结合 Future 一起来使用，开发者需要提前配置过期时间，在过期时间内要是没有获取到value，那么 Future 将会被取消，使用步骤如下：
先引入依赖

```
1<dependency>
2    <groupId>io.github.resilience4j</groupId>
3    <artifactId>resilience4j-timelimiter</artifactId>
4    <version>0.13.2</version>
5</dependency>
6
7
```

使用代码如下：

```
1   TimeLimiterConfig config = TimeLimiterConfig.custom()
2                .timeoutDuration(Duration.ofSeconds(60))
3                .cancelRunningFuture(true)
4                .build();
5        TimeLimiter timeLimiter = TimeLimiter.of(config);
6        ExecutorService executorService = Executors.newSingleThreadExecutor();
7        Supplier<Future<Integer>> futureSupplier = () -> executorService.submit(userService::doSomething);
8        Callable restrictedCall = TimeLimiter
9                .decorateFutureSupplier(timeLimiter, futureSupplier);
10        Try.of(restrictedCall.call)
11                .onFailure(throwable -> System.out.println(throwable.getMessage()));
12
13
```

这里首先创建了一个 TimeLimiter，然后将任务放到线程池中，获取到一个 Supplier 对象，然后使用限时器包装该对象，当调用超时， onFailure 方法就会被触发。

也可以将限时器和断路器结合使用，当调用超时次数过多，直接熔断，如下：

```
1Callable restrictedCall = TimeLimiter
2    .decorateFutureSupplier(timeLimiter, futureSupplier);
3Callable chainedCallable = CircuitBreaker.decorateCallable(circuitBreaker, restrictedCall);
4Try.of(chainedCallable::call)
5    .onFailure(throwable -> LOG.info("We might have timed out or the circuit breaker has opened."));
6
7
```

# 总结

本文首先向大家介绍了传统的容错方案 Hystrix 的一些大致功能，这个读者作为了解即可；然后向读者介绍了 Resilience4j 的一些基本功能，这些基本功能涵盖了请求熔断、限流、限时、缓存、隔离以及重试，这里我们只是介绍了 Resilience4j 的一些基本用法。上文中所有的案例都是在一个普通的 JavaSE 项目中写的，这里并未涉及到微服务，下篇文章我将和大家分享，这六个功能如何在微服务中使用，进而实现微服务系统的高可用。

# 项目地址

github



[SpringCloud之Resilience4J用法精讲 (daimajiaoliu.com)](https://www.daimajiaoliu.com/daima/47985ee7a1003e4)



#### 相关文章

[Resilience4j 基本用法详解](https://www.daimajiaoliu.com/daima/4796354e6100403)

[SpringCloud之在微服务中使用Resilience4J](https://www.daimajiaoliu.com/daima/4798c42c790040c)

[关于服务熔断你不得不知道的知识](https://www.daimajiaoliu.com/daima/569fc827054cc06)

[限流 熔断：Hystrix、 Sentinel](https://www.daimajiaoliu.com/daima/47618961b900406)

[spring cloud 简介](https://www.daimajiaoliu.com/daima/4794a52851003fc)

[微服务概述与SpringCloud概述](https://www.daimajiaoliu.com/daima/569e73f814af802)

[Java流控的各种实现方案](https://www.daimajiaoliu.com/daima/479d153ba900404)

[Spring Cloud Alibaba之服务容错组件 - Sentinel 简介（十一）](https://www.daimajiaoliu.com/daima/4796ebc84100400)

[【源码分析】Resilience4j 框架源码解析（2）：浅析框架设计原理](https://www.daimajiaoliu.com/daima/485f024279003ec)

[使用Spring Boot + Resilience 4j实现断路器](https://www.daimajiaoliu.com/daima/4797ff475100420)

[resilience4j-简单使用](https://www.daimajiaoliu.com/daima/487016230100404)

[稳定性五件套-熔断的原理和实现](https://www.daimajiaoliu.com/daima/56a78dac2e25006)

[08 服务容错-Resilience4j](https://www.daimajiaoliu.com/daima/569b2eaf1eb1803)

[SpringCloud —— Hystrix 断路器](https://www.daimajiaoliu.com/daima/569428745d4cc08)

[springboot-springcloud总结(一)](https://www.daimajiaoliu.com/daima/4eddd9db2900400)

[聊聊微服务的隔离和熔断（送10本书）](https://www.daimajiaoliu.com/daima/486f98d451003f4)

[SpringCloud小Demo](https://www.daimajiaoliu.com/daima/485b8c2f9900408)

[关于SpringCloud、SpringBoot](https://www.daimajiaoliu.com/daima/4ee0244c2100406)

[微服务熔断原理](https://www.daimajiaoliu.com/daima/485cf5a11100400)