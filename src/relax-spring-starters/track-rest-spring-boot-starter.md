# relax-track-rest-spring-boot-starter

## 请求响应接口日志

引入该模块将会开启 Spring MVC 接口的日志功能，自动记录请求响应日志，并提供以下可配置参数：

```properties
track.record.rest.include-headers=true
track.record.rest.include-bodies=true
track.record.rest.exclude-patterns[0]=^/user.*
```

- 日志中是否记录请求头
- 日志中是否记录请求体
- 需要忽略日志的接口正则，上例中 `/user` 开头的接口将不会记录日志，内置的正则有：
    - `^/swagger-ui.*`
    - `^/v3/api-docs.*`
    - `^/actuator.*`

## 基于 MDC 的追踪信息

该模块将会尝试从接口请求头中解析两个用于追踪的参数，如果请求头中未提供，则会自动生成一组追踪参数并在该请求链路中使用：

- `X-Request-ID`
- `X-Correlation-ID`

在当前请求链路中，基于 slf4j 的 MDC 将会持有这组参数，用于日志打印中携带追踪信息，以 logback 配置为例，在 appender 的 pattern 中引用并打印追踪参数：

```xml
<appender name="console" class="ch.qos.logback.core.ConsoleAppender">
    <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
        <level>ALL</level>
    </filter>
    <layout class="ch.qos.logback.classic.PatternLayout">
        <pattern>[%X{X-Request-ID},%X{X-Correlation-ID}] %date{ISO8601} %-5level [%thread] %logger{32}:%L > %message%n
        </pattern>
    </layout>
</appender>
```

配置后的日志效果如下：

```properties
[4391c8432aa845dab1c0fe3e18fbc611,adbe088f32a54d009ef1115c95ac4c3b] 2024-05-09 11:01:10,633 INFO  [http-nio-8080-exec-1] ....
```

## MDC Context 线程传递

### 1. 直接子线程

直接在当前请求中创建子线程，能够正确传递 MDC Context。

```java
@RestController
public class DemoController {
    
    @GetMapping("/test")
    public void test() {
        new Thread() {
          @Override
          public void run() {
            log.info("This log can get the correct MDC Context.");
          }
        }.join();
    }
}
```

### 2. 手动提交线程池

对于任意一个线程池，在提交任务到线程池时手动设置好 MDC Context，完成传递：

```
import com.infilos.spring.track.TrackSupport;

ExecutorService PLAIN_EXECUTOR = Executors.newCachedThreadPool();

PLAIN_EXECUTOR.submit(TrackSupport.wrapMDCContext(() -> {
    log.info("This log can get the correct MDC Context.");
    return 0;
}));
```

### 3. 扩展线程池行为

手动提交线程池时需要每次都手动传递上下文，可以选择直接扩展线程池的行为，在线程池内实现传递逻辑，避免每次提交时操作：

```
import com.infilos.spring.track.concurrent.MDCAwareThreadPoolExecutor;

ThreadPoolExecutor MDC_EXECUTOR = new MDCAwareThreadPoolExecutor(
    4, 4, 1, TimeUnit.MINUTES,
    new SynchronousQueue<>(),
    Executors.defaultThreadFactory(),
    new ThreadPoolExecutor.AbortPolicy()
);
CompletableFuture.supplyAsync(() -> {
    log.info("This log can get the correct MDC Context.");
    return 0;
}, MDC_EXECUTOR);
```

### 4. Spring `@Async`

使用 Spring `@Async` 注解的异步方法，需要给 Spring 使用的线程池设置上传递 MDC Context 的装饰器：

```java
import com.infilos.spring.track.TrackSupport;

@EnableAsync
@Configuration
public class AsyncExecutorConfigure extends AsyncConfigurerSupport {
    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new TaskExecutorBuilder()
                // omitted add other configs here...
                .taskDecorator(TrackSupport.getMDCContextTaskDecorator())
                .build();
        executor.initialize();
  
        return executor;
    }
}
```

### 5. CompletableFuture

`CompletableFuture` 的情况较复杂，目前未提供支持，更多信息可以参考[这篇文章](https://medium.com/asyncparadigm/logging-in-a-multithreaded-environment-and-with-completablefuture-construct-using-mdc-1c34c691cef0).

## MDC Context 跨服务传递

### 1. Spring RestTemplate

提供了一个拦截器用于在发起请求时，将当前 MDC Context 中的信息设置到请求头中。

```java
import com.infilos.spring.track.config.RestTemplateInterceptor;

@Configuration
public class RestTemplateConfigure {
  @Bean
  public RestTemplate restTemplate() {
    RestTemplate restTemplate = new RestTemplate();
    restTemplate.setInterceptors(Arrays.asList(new RestTemplateInterceptor()));
    return restTemplate;
  }
}
```

### 2. Unirest

使用 Unirest 发起请求时提供了对应的拦截器来传递 MDC Context，更多细节请查看[这篇介绍](track-unirest.md)。
