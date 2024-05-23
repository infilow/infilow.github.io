# relax-router-spring-boot-starter

有些业务场景需要将接收到的请求转发到其他服务。

```java
import com.infilos.spring.RestRouter;

@RestController
public class RouteController {
    private static final Logger logger = LoggerFactory.getLogger(RouteController.class);

    @GetMapping("/route/source")
    public void routingSource(HttpServletRequest request, HttpServletResponse response) throws IOException {
        RestRouter.of(request, response)
            .withHost("http://localhost:8080")
            .withHeader("key", "value")
            .withPath("/route/target")
            .withLogger(logger)
            .execute();
    }
}
```

`RestRouter` 提供了一个简单封装，可以直接将接收到的请求包装、执行并返回响应。
