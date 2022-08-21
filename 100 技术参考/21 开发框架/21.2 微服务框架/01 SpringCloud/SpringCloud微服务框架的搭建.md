# SpringCloud微服务框架的搭建

[![img](https://upload.jianshu.io/users/upload_avatars/26464854/757974e8-60a1-48b2-84e7-0d869f74bab5.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96/format/webp)](https://www.jianshu.com/u/b9516bd71dc1)

[Java如何学](https://www.jianshu.com/u/b9516bd71dc1)关注

0.1072021.06.18 15:42:28字数 2,885阅读 237

![img](https://upload-images.jianshu.io/upload_images/26464854-c83e2cab4e165e92?imageMogr2/auto-orient/strip|imageView2/2/w/854/format/webp)

image

# 一、微服务架构

## 1.1什么是分布式

不同模块部署在不同服务器上
作用：分布式解决网站高并发带来问题

## 1.2什么是集群

多台服务器部署相同应用构成一个集群
作用：通过负载均衡设备共同对外提供服务

## 1.3什么是RPC

RPC 的全称是 Remote Procedure Call 是一种进程间通信方式。
它允许程序调用另一个地址空间（通常是共享网络的另一台机器上）的过程或函数，而不用程序员显式编码这个远程调用的细节。即无论是调用本地接口/服务的还是远程的接口/服务，本质上编写的调用代码基本相同。
比如两台服务器A，B，一个应用部署在A服务器上，想要调用B服务器上应用提供的函数或者方法，由于不在一个内存空间，不能直接调用，这时候需要通过就可以应用RPC框架的实现来解决

### 1.3.1restful、soap、rpc

（1）RESTful是一种架构设计风格，提供了设计原则和约束条件，而不是架构。而满足这些约束条件和原则的应用程序或设计就是 RESTful架构或服务。
（2）SOAP，简单对象访问协议是一种数据交换协议规范，
是一种轻量的、简单的、基于XML的协议的规范。SOAP协议和HTTP协议一样，都是底层的通信协议，只是请求包的格式不同而已，SOAP包是XML格式的。
SOAP的消息是基于xml并封装成了符合http协议，因此，它符合任何路由器、 防火墙或代理服务器的要求。
soap可以使用任何语言来完成，只要发送正确的soap请求即可，基于soap的服务可以在任何平台无需修改即可正常使用。
（3）RPC就是从一台机器（客户端）上通过参数传递的方式调用另一台机器（服务器）上的一个函数或方法（可以统称为服务）并得到返回的结果。
RPC 会隐藏底层的通讯细节（不需要直接处理Socket通讯或Http通讯）
RPC 是一个请求响应模型。客户端发起请求，服务器返回响应（类似于Http的工作方式）
RPC 在使用形式上像调用本地函数（或方法）一样去调用远程的函数（或方法）。

### 1.3.2rpc远程调用框架

几种比较典型的RPC的实现和调用框架。
（1）RMI实现，利用java.rmi包实现，基于Java远程方法协议(Java Remote Method Protocol)
和java的原生序列化。
（2）Hessian，是一个轻量级的remoting onhttp工具，使用简单的方法提供了RMI的功能。 基于HTTP协议，采用二进制编解码。
（3）thrift是一种可伸缩的跨语言服务的软件框架。thrift允许你定义一个描述文件，描述数据类型和服务接口。依据该文件，编译器方便地生成RPC客户端和服务器通信代码。
（4）SpringCloud 为开发人员提供了快速构建分布式系统的一些工具，包括配置管理、服务发现、断路器、路由、微代理、事件总线、全局锁、决策竞选、分布式会话等等。

## 1.4什么是SOA

业务系统分解为多个组件，让每个组件都独立提供离散，自治，可复用的服务能力
通过服务的组合和编排来实现上层的业务流程
作用：简化维护,降低整体风险,伸缩灵活

## 1.5什么是微服务

架构设计概念,各服务间隔离（分布式也是隔离）,自治（分布式依赖整体组合）其它特性(单一职责,边界,异步通信,独立部署)是分布式概念的跟严格执行
SOA到微服务架构的演进过程
作用：各服务可独立应用，组合服务也可系统应用(巨石应用[monolith]的简化实现策略-平台思想)

## 1.6使用RPC http技术实现会员与订单系统通讯

# 二、微服务架构

微服务架构是一项在云中部署应用和服务的新技术。大部分围绕微服务的争论都集中在容器或其他技术是否能很好的实施微服务，而红帽说API应该是重点。

微服务可以在“自己的程序”中运行，并通过“轻量级设备与HTTP型API进行沟通”。关键在于该服务可以在自己的程序中运行。通过这一点我们就可以将服务公开与微服务架构（在现有系统中分布一个API）区分开来。在服务公开中，许多服务都可以被内部独立进程所限制。如果其中任何一个服务需要增加某种功能，那么就必须缩小进程范围。在微服务架构中，只需要在特定的某种服务中增加所需功能，而不影响整体进程的架构。

# 三、SpringCloud

pringCloud 为开发人员提供了快速构建分布式系统的一些工具，包括配置管理、服务发现、断路器、路由、微代理、事件总线、全局锁、决策竞选、分布式会话等等。它运行环境简单，可以在开发人员的电脑上跑。

# 四、服务提供者与消费关系

服务提供者:提供服务被人调用
消费者:调用被人服务

# 五、服务的注册与发现(Eureka )

在这里，我们需要用的的组件上Spring Cloud Netflix的Eureka ,eureka是一个服务注册和发现模块。

### 4.1 服务注册

### 4.1.1创建eurekaserver 项目

### 4.1.2引入maven依赖

**以上都是概念。**

![img](https://upload-images.jianshu.io/upload_images/26464854-15d7bfa416b427fa?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image



以上是我的springcloud项目，eureka是注册中心，zuul是注册网关，ribbon和feign都是cloud的rpc远程调用。
zuul主要是用来配置网关



```c
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.2.RELEASE</version>
        <relativePath /> <!-- lookup parent from repository -->
    </parent>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <!--eureka server -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka-server</artifactId>
        </dependency>
        <!-- spring boot test -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Dalston.RC1</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
    <repositories>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>
```

### 4.3配置application.yml



```c
server:
  port: 8761
eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

### 4.3配置application.yml



```c
server:
  port: 8761
eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

### 4.4启动EurekaServer



```c
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
       SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

1. eureka.client.registerWithEureka=true #是否将自身注册
2. eureka.client.fetchRegistry=false #如果为true，启动时报警

### 4.5打开eureka server 界面的

![img](https://upload-images.jianshu.io/upload_images/26464854-c7bba0c96cc89645?imageMogr2/auto-orient/strip|imageView2/2/w/601/format/webp)

image

## 4.2 服务提供者

创建一个服务提供者 (eureka client),当client向server注册时，它会提供一些元数据，例如主机和端口，URL，主页等。Eureka server 从每个client实例接收心跳消息。 如果心跳超时，则通常将该实例从注册server中删除。

### 4.2.1 创建项目eurekaclient (百度)

### 4.2.2 引入maven依赖



```c
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.2.RELEASE</version>
        <relativePath /> <!-- lookup parent from repository -->
    </parent>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Dalston.RC1</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
    <repositories>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>
```

### 4.2.3 application.yml配置



```c
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
server:
  port: 8762
spring:
  application:
    name: service-hi
```

### 4.2.4 发布服务

通过注解@EnableEurekaClient 表明自己是一个eurekaclient.



```c
@SpringBootApplication
@EnableEurekaClient
@RestController
public class ServiceHiApplication {

    public static void main(String[] args) {
        SpringApplication.run(ServiceHiApplication.class, args);
    }

    @Value("${server.port}")
    String port;

    @RequestMapping("/hi")
    public String home(@RequestParam String name) {
        return "hi " + name + ",i am from port:" + port;
    }

}
```

### 4.2.5演示效果

需要指明[spring.application.name](https://links.jianshu.com/go?to=http%3A%2F%2Fspring.application.name),这个很重要，这在以后的服务与服务之间相互调用一般都是根据这个name 。 启动工程，打开[http://localhost:8761](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A8761) ，即eureka server 的网址：

![img](https://upload-images.jianshu.io/upload_images/26464854-4edf3bb3dfa47243?imageMogr2/auto-orient/strip|imageView2/2/w/601/format/webp)

image


你会发现一个服务已经注册在服务中了，服务名为SERVICE-HI ,端口为7862
这时打开`http://localhost:8762/hi?name=forezp` ，你会在浏览器上看到 :
`hi forezp,i am from port:8762`



# 服务消费者（Feign）

## 什么是Feign

> Feign是一个声明式的伪Http客户端，它使得写Http客户端变得更简单。使用Feign，只需要创建一个接口并注解。
> 它具有可插拔的注解特性，可使用Feign 注解和JAX-RS注解。Feign支持可插拔的编码器和解码器。
> Feign默认集成了Ribbon，并和Eureka结合，默认实现了负载均衡的效果。

**简而言之：**

- Feign 采用的是基于接口的注解
- Feign 整合了ribbon

**准备工作**

继续用上一节的工程， 启动eureka-server，端口为8761; 启动service-hi 两次，端口分别为8762 、8773.
准备工创建一个feign的服务
新建一个spring-boot工程，取名为serice-feign，在它的pom文件引入Feign的起步依赖spring-cloud-starter-feign、Eureka的起步依赖
spring-cloud-starter-eureka、Web的起步依赖spring-boot-starter-web，代码如下：



```c
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-feign</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Dalston.RC1</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

    <repositories>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>
```

## application.yml配置

在工程的配置文件application.yml文件，指定程序名为service-feign，端口号为8765，服务注册地址为[http://localhost:8761/eureka/](https://links.jianshu.com/go?to=http%3A%2F%2Flocalhost%3A8761%2Feureka%2F) ，代码如下：



```c
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
server:
  port: 8765
spring:
  application:
    name: service-feign
```

## 定义一个feign接口



```c
@FeignClient(value = "service-hi")
public interface SchedualServiceHi {
    @RequestMapping(value = "/hi", method = RequestMethod.GET)
    String sayHiFromClientOne(@RequestParam(value = "name") String name);
}
```

**一个”/hi”的API接口**



```c
@RestController
public class HiController {
    @Autowired
    SchedualServiceHi schedualServiceHi;
    @RequestMapping(value = "/hi",method = RequestMethod.GET)
    public String sayHi(@RequestParam String name){
        return schedualServiceHi.sayHiFromClientOne(name);
    }
}
```

**启动方式**



```c
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class SericeFeign {

    public static void main(String[] args) {
        SpringApplication.run(SericeFeign.class, args);
    }

}
```

演示效果



```c
启动程序，多次访问http://localhost:8765/hi?name=forezp(http://localhost:8765/hi?name=forezp),浏览器交替显示：

hi forezp,i am from port:8762

hi forezp,i am from port:8763
```

# Hystrix断路器

在微服务架构中，根据业务来拆分成一个个的服务，服务与服务之间可以相互调用（RPC），在Spring Cloud可以用RestTemplate+Ribbon和Feign来调用。为了保证其高可用，单个服务通常会集群部署。由于网络原因或者自身的原因，服务并不能保证100%可用，如果单个服务出现问题，调用这个服务就会出现线程阻塞，此时若有大量的请求涌入，Servlet容器的线程资源会被消耗完毕，导致服务瘫痪。服务与服务之间的依赖性，故障会传播，会对整个微服务系统造成灾难性的严重后果，这就是服务故障的“雪崩”效应。

为了解决这个问题，业界提出了断路器模型。

## 7.1 什么是Hystrix

Netflix开源了Hystrix组件，实现了断路器模式，SpringCloud对这一组件进行了整合。 在微服务架构中，一个请求需要调用多个服务是非常常见的，如下图：

![img](https://upload-images.jianshu.io/upload_images/26464854-d4cc268fd8e5d2fa?imageMogr2/auto-orient/strip|imageView2/2/w/601/format/webp)

image


底层的服务如果出现故障，会导致连锁故障。当对特定的服务的调用的不可用达到一个阀值（Hystric 是5秒20次） 断路器将会被打开。

![img](https://upload-images.jianshu.io/upload_images/26464854-378e34fe5791e478?imageMogr2/auto-orient/strip|imageView2/2/w/601/format/webp)

在这里插入图片描述


断路打开后，可用避免连锁故障，fallback方法可以直接返回一个固定值。
**准备工作**



这篇文章基于上一篇文章的工程，首先启动上一篇文章的工程，启动eureka-server 工程；启动service-hi工程，它的端口为8762。
在ribbon使用断路器
改造serice-ribbon 工程的代码，首先在pox.xml文件中加入spring-cloud-starter-hystrix的起步依赖：



```c
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
```

**改造service**

改造HelloService类，在hiService方法上加上@HystrixCommand注解。该注解对该方法创建了熔断器的功能，并指定了fallbackMethod熔断方法，熔断方法直接返回了一个字符串，字符串为”hi,”+name+”,sorry,error!”，代码如下：



```c
@Service
public class HelloService {

    @Autowired
    RestTemplate restTemplate;

    @HystrixCommand(fallbackMethod = "hiError")
    public String hiService(String name) {
        return restTemplate.getForObject("http://SERVICE-HI/hi?name=" + name, String.class);
    }

    public String hiError(String name) {
        return "hi," + name + ",sorry,error!";
    }
}
```

**在启动类上加入**



```c
@SpringBootApplication
@EnableEurekaClient
@EnableHystrix   //断路器
public class App {

    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }

    @Bean
    @LoadBalanced
    RestTemplate restTemplate() {
        return new RestTemplate();
    }

}
```

**演示效果**

![img](https://upload-images.jianshu.io/upload_images/26464854-79ae83a72060e97b?imageMogr2/auto-orient/strip|imageView2/2/w/554/format/webp)

image



> 如果文章有错的地方欢迎指正，大家可以在评论区互相交流。



https://www.jianshu.com/p/e8ceb79443a6