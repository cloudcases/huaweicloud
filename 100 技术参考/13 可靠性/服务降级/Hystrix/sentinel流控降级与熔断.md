# sentinel流控降级与熔断

发布于2022-08-16 19:20:37阅读 690

>  人们因为能忘却，所以自己能渐渐的脱离了受过的苦痛，也因为能忘却，所以照样得再犯前人的错误。——鲁迅 

引入依赖

```javascript
<!--    sentinel 降级熔断    -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
<!--    与sentinel控制台进行通信    -->
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-transport-simple-http</artifactId>
    <version>1.7.1</version>
</dependency>
```

复制

下载客户端[jar包](https://waibi.oss-cn-chengdu.aliyuncs.com/picGo/sentinel-dashboard.jar)

然后输入命令运行

```javascript
java -Dserver.port=9000 -Dcsp.sentinel.dashboard.server=localhost:9000 -Dproject.name=sentinel-dashboard -jar sentinel-dashboard.jar
```

复制

>  若您的应用使用了 Spring AOP，您需要通过配置的方式将 `SentinelResourceAspect` 注册为一个 Spring Bean： @Configuration public class SentinelAspectConfiguration {   	@Bean  	public SentinelResourceAspect sentinelResourceAspect() {             return new SentinelResourceAspect();  	} } 

然后在配置文件配置

```javascript
sentinel:
  transport:
    port: 9001
    dashboard: localhost:9000
  filter:
    enabled: true
    url-patterns: /**
```

复制

因为我们可以看到运行后输出的

>  2021-03-16 23:24:49.986  INFO 18604 — [           main] c.a.c.s.SentinelWebAutoConfiguration     : [Sentinel Starter] register SentinelWebInterceptor with urlPatterns: [/*]. 

所以需要配置`url-patterns: /**`否则只有一层`url`会被监控到

```javascript
/**
     * 登录成功时调用该接口，传入一句话
     *
     * @param word 参数 例子：?word=登录成功！
     * @return 返回json格式的map
     */
    @GetMapping("say")
    public AjaxJson say(@RequestParam String word) {
        return AjaxJson.success().put("data", word);
    }
```

复制

访问控制台`localhost:9000`(上面配置的9000端口)

输入默认用户名密码`sentinel`

然后找到我们需要限流的接口点击流控

![img](https://ask.qcloudimg.com/http-save/yehe-9968198/bfeab4b7dffca28a184b3fb1a0b493df.png?imageView2/2/w/1620);

![img](https://ask.qcloudimg.com/http-save/yehe-9968198/06be124807ad0498c5e9b95f88606587.png?imageView2/2/w/1620);

然后我们每秒就只能访问一次了

![img](https://ask.qcloudimg.com/http-save/yehe-9968198/d9a7063c6aafb2975ae9941dae5f9b56.png?imageView2/2/w/1620);

接下来是服务降级

我们可以在配置文件开启

```javascript
feign:
  sentinel:
    enabled: true
```

复制

然后实现我们的`feign`接口

```javascript
package com.ruben.feign.fallback;

import com.ruben.feign.ConsumerService;
import com.ruben.pojo.dto.PageDTO;
import com.ruben.utils.AjaxJson;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

/**
 * @ClassName: ConsumerServiceFallback
 * @Description: 我还没有写描述
 * @Date: 2021/3/16 0016 22:14
 * *
 * @author: <achao1441470436@gmail.com>
 * @version: 1.0
 * @since: JDK 1.8
 */
@Slf4j
@Service
public class ConsumerServiceFallback implements ConsumerService {
    @Override
    public AjaxJson list(PageDTO pageDTO) {
        log.error("服务挂了");
        return AjaxJson.error("");
    }

    @Override
    public AjaxJson dropWare() {
        log.error("服务挂了");
        return AjaxJson.error("");
    }
}
```

复制

最后在`feign`接口使用`@FeignClient`的`fallback`参数指定降级实现

```javascript
package com.ruben.feign;

import com.ruben.feign.fallback.ConsumerServiceFallback;
import com.ruben.pojo.dto.PageDTO;
import com.ruben.utils.AjaxJson;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestBody;

@FeignClient(value = "ruben-consumer", fallback = ConsumerServiceFallback.class)
public interface ConsumerService {
    @GetMapping("study/list")
    AjaxJson list(@RequestBody PageDTO pageDTO);

    @GetMapping("ware")
    AjaxJson dropWare();
}
```

复制

然后访问我们的接口如果出现异常，则会调用我们的降级实现

![img](https://ask.qcloudimg.com/http-save/yehe-9968198/1e20f7282dffa30afb3b63fee964c44d.png?imageView2/2/w/1620);

我们再配置熔断

点击我们远程接口的降级按钮

![img](https://ask.qcloudimg.com/http-save/yehe-9968198/1b5fc3206a6748bf1923f2a60cc6458e.png?imageView2/2/w/1620);

![img](https://ask.qcloudimg.com/http-save/yehe-9968198/316dade40eb48f0bf78333050f083618.png?imageView2/2/w/1620);

配置完成后如果我们再远程调用在5秒内异常比例超过百分之八十，则之后都会直接调用我们的降级实现了

自定义受保护的资源

可以在接口上加`@SentinelResource`注解

也可以在代码中

```javascript
    @Override
    @Transactional
    @GlobalTransactional
    public AjaxJson order() {
        try (Entry entry = SphU.entry("resourceName")) {
            consumerService.dropWare();
//        mpOrderMapper.insert(OrderPO.builder().id(1L).build());
        } catch (BlockException e) {
            return AjaxJson.error("慢点");
        }
        return AjaxJson.success();
    }
}
```

复制

加上`try(Entry entry = SphU.entry("{资源名}")){}catch(BlockException e){}`

后即可把这段代码作为一个受保护的资源

我们可以在`catch`中编写我们的降级方法

然后我们需要在流控规则中新建

![img](https://ask.qcloudimg.com/http-save/yehe-9968198/0afc2012331eda7367e23b05858c93f3.png?imageView2/2/w/1620);

![img](https://ask.qcloudimg.com/http-save/yehe-9968198/1928f3f93f9885fcfb158c38e35ab382.png?imageView2/2/w/1620);

然后可以看到我们成功实现流控我们的受保护资源

![img](https://ask.qcloudimg.com/http-save/yehe-9968198/f0348be08515beb6165d0d233ec6a256.png?imageView2/2/w/1620)



https://cloud.tencent.com/developer/article/2074128?from=article.detail.1843914