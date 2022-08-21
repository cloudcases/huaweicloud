# 纯干货！Spring Cloud Gateway整合OAuth2.0 实现分布式统一认证授权

[![img](https://upload.jianshu.io/users/upload_avatars/23866222/b665db5d-7d55-4676-a41e-00fb6067ca73.jpg?imageMogr2/auto-orient/strip|imageView2/1/w/96/h/96/format/webp)](https://www.jianshu.com/u/4f2c19bd668c)

[马小莫QAQ](https://www.jianshu.com/u/4f2c19bd668c)[![  ](https://upload.jianshu.io/user_badge/19c2bea4-c7f7-467f-a032-4fed9acbc55d)](https://www.jianshu.com/mobile/creator)关注

22022.02.09 16:27:54字数 2,633阅读 1,501

今天这篇文章介绍一下**Spring Cloud Gateway**整合**OAuth2.0**实现认证授权，涉及到的知识点有点多，有不清楚的可以看下陈某的往期文章。

文章目录如下：

![img](https://upload-images.jianshu.io/upload_images/23866222-62674ffdfaef6c92.png?imageMogr2/auto-orient/strip|imageView2/2/w/919/format/webp)

### 微服务认证方案

微服务认证方案目前有很多种，每个企业也是大不相同，但是总体分为两类，如下：

1. 网关只负责转发请求，认证鉴权交给每个微服务商控制
2. 统一在网关层面认证鉴权，微服务只负责业务

**你们公司目前用的是哪种方案？**

先来说说第一种方案，有着很大的弊端，如下：

- 代码耦合严重，每个微服务都要维护一套认证鉴权
- 无法做到统一认证鉴权，开发难度太大

第二种方案明显是比较简单的一种，优点如下：

- 实现了统一的认证鉴权，微服务只需要各司其职，专注于自身的业务
- 代码耦合性低，方便后续的扩展

下面陈某就以第二种方案为例，整合**Spring Cloud Gateway+Spring Cloud Security** 整合出一套统一认证鉴权案例。

### 案例架构

开始撸代码之前，先来说说大致的认证鉴权流程，架构如下图：

![img](https://upload-images.jianshu.io/upload_images/23866222-5c9fb398be7d2c92.png?imageMogr2/auto-orient/strip|imageView2/2/w/888/format/webp)

**大致分为四个角色，如下**：

- **客户端**：需要访问微服务资源
- **网关**：负责转发、认证、鉴权
- **OAuth2.0授权服务**：负责认证授权颁发令牌
- **微服务集合**：提供资源的一系列服务。

**大致流程如下**：

1、客户端发出请求给网关获取令牌

2、网关收到请求，直接转发给授权服务

3、授权服务验证用户名、密码等一系列身份，通过则颁发令牌给客户端

4、客户端携带令牌请求资源，请求直接到了网关层

5、网关对令牌进行校验（**验签**、**过期时间校验**....）、鉴权（对当前令牌携带的权限）和访问资源所需的权限进行比对，如果权限有交集则通过校验，直接转发给微服务

6、微服务进行逻辑处理

针对上述架构需要新建三个服务，分别如下：

![img](https://upload-images.jianshu.io/upload_images/23866222-ff7431fc480781de.png?imageMogr2/auto-orient/strip|imageView2/2/w/423/format/webp)

案例源码目录如下：

![img](https://upload-images.jianshu.io/upload_images/23866222-2fe14ce56b439522.png?imageMogr2/auto-orient/strip|imageView2/2/w/949/format/webp)

### 认证授权服务搭建

很多企业是将认证授权服务直接集成到网关中，这么做耦合性太高了，这里陈某直接将认证授权服务抽离出来。

认证服务的搭建这里就不再细说了，上一篇文章中已经介绍的很清楚了：OAuth2.0实战！使用JWT令牌认证！

新建一个**oauth2-cloud-auth-server**模块，目录如下：

![img](https://upload-images.jianshu.io/upload_images/23866222-d91cad03be73a50b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1074/format/webp)

和上篇文章不同的是创建了**JwtTokenUserDetailsService**这个类，用于从数据库中加载用户，如下：

![img](https://upload-images.jianshu.io/upload_images/23866222-ee0bbf87208c0a3b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1188/format/webp)

为了演示只是模拟了从数据库中查询，其中存了两个用户，如下：

- **user**：具有ROLE_user权限
- **admin**：具有ROLE_admin、ROLE_user权限

要想这个生效，还要在security的配置文件**SecurityConfig**中指定，如下图：

![img](https://upload-images.jianshu.io/upload_images/23866222-ec23245cc5eff36c.png?imageMogr2/auto-orient/strip|imageView2/2/w/823/format/webp)

另外还整合了注册中心Nacos，详细配置就不贴了，可以看源码。

### 网关服务搭建

网关使用的是**Spring Cloud Gateway**，网关如何搭建这里就不再细说了，有不清楚的可以看之前文章：Spring Cloud Gateway夺命连环10问？

新建一个**oauth2-cloud-gateway**模块，目录如下图：

![img](https://upload-images.jianshu.io/upload_images/23866222-4c25131c1e8cd329.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

#### 1、添加依赖

需要添加几个OAuth2.0相关的依赖，如下：

![img](https://upload-images.jianshu.io/upload_images/23866222-f48ba3dca455db28.png?imageMogr2/auto-orient/strip|imageView2/2/w/796/format/webp)

#### 2、JWT令牌服务配置

使用JWT令牌，配置要和认证服务的令牌配置相同，代码如下：

![img](https://upload-images.jianshu.io/upload_images/23866222-b3bd1787de93ad9b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1011/format/webp)

#### 3、认证管理器自定义

新建一个**JwtAuthenticationManager**，需要实现**
ReactiveAuthenticationManager**这个接口。

认证管理的作用就是获取传递过来的令牌，对其进行**解析**、**验签**、**过期时间**判定。

详细代码如下：

![img](https://upload-images.jianshu.io/upload_images/23866222-bba8e9e137b0e646.png?imageMogr2/auto-orient/strip|imageView2/2/w/1199/format/webp)

逻辑很简单，就是通过JWT令牌服务解析客户端传递的令牌，并对其进行校验，比如上传三处校验失败，抛出令牌无效的异常。

**抛出的异常如何处理？如何定制返回的结果？**

这里抛出的异常可以通过**Spring Cloud Gateway**的全局异常进行捕获，这个内容在Spring Cloud Gateway夺命连环10问？这篇文章有详细的介绍。下面只贴出关键代码，如下：

![img](https://upload-images.jianshu.io/upload_images/23866222-008ab72ab677e3e4.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

#### 4、鉴权管理器自定义

经过认证管理器**JwtAuthenticationManager**认证成功后，就需要对令牌进行鉴权，如果该令牌无访问资源的权限，则不允通过。

新建**JwtAccessManager**，实现**
ReactiveAuthorizationManager**，代码如下：

![img](https://upload-images.jianshu.io/upload_images/23866222-19b1ccc5f971a13a.png?imageMogr2/auto-orient/strip|imageView2/2/w/1198/format/webp)

这里的逻辑很简单，就是**取出令牌中的权限和当前请求资源URI的权限对比**，如果有交集则通过。

**①处的代码什么意思？**

这里是直接从Redis中取出资源URI对应的权限集合，因此实际开发中需要维护**资源URI和权限的对应关系**，这里不细说，为了演示，陈某直接在项目启动的时候向Redis中添加了两个资源的权限，代码如下：

![img](https://upload-images.jianshu.io/upload_images/23866222-e479e5a14e51df06.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

> 注意：实际开发中需要维护资源URI和权限的对应关系。

**②处的代码什么意思？**

这处代码就是取出令牌中的权限集合

**③处的代码什么意思？**

这处代码就是比较两者权限了，有交集，则放行。

#### 5、令牌无效或者过期时定制结果

在第4步，如果令牌失效或者过期，则会直接返回，这里需要定制提示信息。

新建一个**
RequestAuthenticationEntryPoint**，实现**
ServerAuthenticationEntryPoint**，代码如下：

![img](https://upload-images.jianshu.io/upload_images/23866222-2c9204eb44444b8f.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

#### 6、无权限时定制结果

在第4步鉴权的过程中，如果无该权限，也是会直接返回，这里也需要定制提示信息。

新建一个**
RequestAccessDeniedHandler**，实现**ServerAccessDeniedHandler**，代码如下：

![img](https://upload-images.jianshu.io/upload_images/23866222-48550ecd626370c4.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

#### 7、OAuth2.0相关配置

经过上述6个步骤，相关组件已经准备就绪，现在直接配置到OAuth2.0中。

新建SecurityConfig这个配置类，标注注解 **@EnableWebFluxSecurity**，注意不是 **@EnableWebSecurity**，因为Spring Cloud Gateway是基于Flux实现的。详细代码如下：

需要配置的内容如下：

- 认证过滤器，其中利用了认证管理器对令牌进行校验
- 鉴权管理器、令牌失效异常处理、无权限访问异常处理
- 白名单配置
- 跨域过滤器的配置

#### 8、全局过滤器定制

试想一下：**网关层面认证鉴权成功后，下游微服务如何获取到当前用户的详细信息？**

陈某这里是将令牌携带的用户信息解析出来，封装成**JSON**数据，然后通过**Base64**加密，放入到请求头中，转发给下游微服务。

这样一来，下游微服务只需要解密请求头中的JSON数据，即可获取用户的详细信息。

因此需要在网关中定义一个全局过滤器，用来拦截请求，解析令牌，关键代码如下：

上述代码逻辑如下：

- 检查是否是白名单，白名单直接放行
- 检验令牌是否存在
- 解析令牌中的用户信息
- 封装用户信息到JSON数据中
- 加密JSON数据
- 将加密后的JSON数据放入到请求头中

好了，经过上述8个步骤，完整的网关已经搭建成功了。

### 订单微服务搭建

由于在网关层面已经做了鉴权了（细化到每个URI），因此微服务就不用集成Spring Security单独做权限控制了。

因此这里的微服务也是相对比较简单了，只需要将网关层传递的加密用户信息解密出来，放入到Request中，这样微服务就能随时获取到用户的信息了。

新建一个**oauth2-cloud-order-service**模块，目录如下：

![img](https://upload-images.jianshu.io/upload_images/23866222-5c0e55cb4b4ae336.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

新建一个过滤器**AuthenticationFilter**，用于解密网关传递的用户数据，代码如下：

![img](https://upload-images.jianshu.io/upload_images/23866222-76def481b131fb2b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

新建两个接口，返回当前登录的用户信息，如下：

![img](https://upload-images.jianshu.io/upload_images/23866222-8884cdf4740e5ac8.png?imageMogr2/auto-orient/strip|imageView2/2/w/898/format/webp)

注意：以上两个接口所需要的权限已经放入到了Redis中，权限如下：

- **/order/login/info**：ROLE_admin和ROLE_user都能访问
- **/order/login/admin**：ROLE_admin权限才能访问

![img](https://upload-images.jianshu.io/upload_images/23866222-b049597cb212e04b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

### 为什么要将URI和权限放入Redis？

在网关的**鉴权管理器**那里是直接从Redis中获取URI对应的权限，然后和令牌中的权限比较，为什么要这样做？

这也是目前企业中比较常用的一种方式，将鉴权完全放在了网关层面，也实现了**动态权限**校验。**当然有些是直接将接口的权限控制在每个微服务中。**

> 采用陈某的这种方案需要另外维护URI和权限的对应关系，当然这种难度很低，便于实现。

只是一种方案，具体是否选用还要考虑到架构层面。

### 测试

同时启动上述三个服务，如下：

![img](https://upload-images.jianshu.io/upload_images/23866222-ac0905067af65537.png?imageMogr2/auto-orient/strip|imageView2/2/w/705/format/webp)

**1、用密码模式登录user，获取令牌，如下**：

![img](https://upload-images.jianshu.io/upload_images/23866222-710b2ab2ed493889.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

**2、使用user用户的令牌访问/order/login/info接口，如下**：

![img](https://upload-images.jianshu.io/upload_images/23866222-7c7c09c6ccdf4a9b.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

可以看到成功返回了，因为具备ROLE_user权限。

**3、使用user用户的令牌访问/order/login/admin接口，如下**：

![img](https://upload-images.jianshu.io/upload_images/23866222-6dd4f874654e3c67.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

可以看到直接返回了无权限访问，直接在网关层被拦截了。

### 总结

本篇文章只是简单的整合了**网关+OAuth2.0**，实际开发中还有一些细节待完善，由于文章篇幅限制，后续介绍......