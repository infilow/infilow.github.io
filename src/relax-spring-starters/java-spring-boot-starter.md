# relax-java-spring-boot-starter

Spring boot 常用配置的基本封装，简化新建项目时的重复配置。

## ContextConfigure

提供一个静态的用于获取 `ApplicationContext` 的方法。

```java
package com.demo;

import com.infilos.spring.config.ContextConfigure;

public class DemoService {

    public static void demoMethod() {
        // 通过静态方法使用
        ApplicationContext context = ContextConfigure.context();
        
        // 同时提供了简化的注入 bean 的方法
        SomeBean bean1 = ContextConfigure.inject("someBeanName");
        SomeBean bean2 = ContextConfigure.inject(someBean.class);
    }
}
```

## JsonConfigure

统一设置 Spring 接口中使用的 Jackson `ObjectMapper`，通过 `MappingJackson2HttpMessageConverter` 设置为全局统一的 `ObjectMapper`。

用于确保全局一致的序列化、反序列化行为。可以通过 `com.infilos.relax.Json.underMapper()` 获取该全局 `ObjectMapper` 或直接用于执行序列化或反序列化。

同时提供了几种时间类型的序列化反序列化器：

- `LocalDate`：`yyyy-MM-dd`
- `LocalDatetime`：`yyyy-MM-dd HH:mm:ss`
- `ZonedDatetime`：`yyyy-MM-dd HH:mm:ss`
- `Timestamp`：`yyyy-MM-dd HH:mm:ss`

基于 `name` 或 `code` 的枚举序列化反序列化器，继承 `NameBasedEnum` 或 `CodeBasedEnum` 的枚举可以直接用于类字段类型，并能自动使用 `name` 或 `code` 实现序列化或反序列化。

## ShutdownConfigure

用于注册 Spring 服务终止时的回调逻辑，当 Spring 服务终止时，将按注册顺序逐个执行已注册的回调逻辑，实现自定义清理逻辑。

```java
import com.infilos.spring.config.ShutdownConfigure;

@Service
class DemoService {
    @Autowired
    private ShutdownConfigure shutdownConfigure;
    
    public void demoMethod() {
        shutdownConfigure.register(() -> {
            // custom shutdown logic here
        });
    }
}
```

## SwaggerConfigure

基于 `springdoc-openapi-ui` 的预设 Swagger 配置，可以直接通过地址 `http://localhost:8080/swagger-ui.html` 访问 Swagger 文档。

基本的接口配置：

```java
package com.demo;

import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;

@Tag(name="用户管理接口")
@RestController("/user")
public class UserController {
    
    @Operation(summary="创建用户")
    @PostMapping("/create")
    public Long createUser(@RequestBody UserCreation creation) {
        return 1L;
    }
}
```

生产环境关闭 Swagger，如 `application-prod.properties`：

```properties
springdoc.api-docs.enabled=false
springdoc.swagger-ui.enabled=false
```

## Respond

Spring 接口响应封装类。

Spring 接口的客户端可能是前端、APP 或他后端服务(用作 RPC)，可能会跨越多层内外网关，为了避免混淆，常见做法是将 HTTP 协议层的状态码与业务状态码分离，接口客户端对接口响应的判断则分为两层：

- HTTP 协议状态码：`200 <= code < 300` 则表示网络层成功，否则均为网络层失败
- 业务状态码：在网络层成功的前提下，如果 `code==0` 则表示业务成功，否则均为业务失败

基本结构：

```java
public final class Respond<T> {
    private final Integer code;
    private final T data;
    private final String message;
}
```

常用静态方法：

- `<T> Respond<T> succed()`
- `<T> Respond<T> succed(T data)`
- `<T> Respond<T> succed(int code, T data)`
- `<T> Respond<T> succed(int code)`
- `<T> Respond<T> failed(String message)`
- `<T> Respond<T> failed(int code, String message)`
- `<T> Respond<T> failed(RespondEnum<?> error)`
- `<T> Respond<T> failed(RespondEnum<?> error, String message)`
- `<T> Respond<T> failed(RespondEnum<?> error, Object... messageTemplateArgs)`
- `<T> Respond<T> failed(Throwable cause)`
- `<T> Respond<T> failed(int code, Throwable cause)`
- `<T> Respond<T> failed()`
- `<T> Respond<T> of(Either<?, T> either)`
- `ResponseEntity<byte[]> ofBytes(HttpStatus status, HttpHeaders headers, byte[] bytes)`
- `ResponseEntity<byte[]> ofFileBytes(String name, byte[] bytes)`
- `ResponseEntity<byte[]> ofFileBytes(String name, String format, byte[] bytes)`

继承 `RespondEnum<E>` 可用于实现业务自定义错误码枚举，比如:

```java
@AllArgsConstructor
public enum BizError implements RespondEnum<BizError> {
    PARAMS_INVALID(10001, "请求参数无效"),
    ENTITY_NOTFOUND(10002, "请求实体不存在: %s"),
    ;

    private final int code;
    private final String message;

    @Override
    public int getCode() {
        return code;
    }

    @Override
    public String getMessage() {
        return message;
    }
}
```

在接口中使用：

```
return Respond.failed(BizError.PARAMS_INVALID);
return Respond.failed(BizError.ENTITY_NOTFOUND, id);
```
