# relax-track-unirest

Unirest 是一个简单易用的 HTTP 客户端，其底层是对 Apache HTTP Client 的封装，稳定可靠。

这里提供一些扩展用于简化应用过程。

## MDC Context 传递

用于在发起请求前将当前 MDC Context 设置到请求头中，实现跨服务传递追踪信息。

```java
import com.infilos.spring.track.TrackUnirestMDCInterceptor;
```

## 请求响应日志打印

用于将请求内容和响应内容打印到日志，便于排查问题。

```java
import com.infilos.spring.track.TrackUnirestLoggingInterceptor;
```

## 请求响应耗时日志打印

用于将请求响应耗时打印到日志。

```java
import com.infilos.spring.track.TrackUnirestMetricLogger;
```

## 完整配置示例

```java
@Configuration
public class GlobalUnirestConfigure implements InitializingBean {

    private static final int TIMEOUT = 30 * 1000;

    @Override
    public void afterPropertiesSet() throws Exception {
        SSLContext sslContext = new SSLContextBuilder().loadTrustMaterial(null, new TrustSelfSignedStrategy() {
            public boolean isTrusted(X509Certificate[] chain, String authType) {
                return true;
            }
        }).build();

        HttpClient customHttpClient = HttpClients.custom()
            .setMaxConnTotal(30)
            .setMaxConnPerRoute(30)
            .setSSLContext(sslContext)
            .setSSLHostnameVerifier(new NoopHostnameVerifier())
            .build();
        Unirest.config()
            .verifySsl(false)
            .httpClient(ApacheClient.builder(customHttpClient))
            .socketTimeout(TIMEOUT)
            .connectTimeout(TIMEOUT)
            .followRedirects(false)
            .setDefaultHeader("Content-Type", "application/json")
            .setDefaultHeader("Accept", "application/json")
            .setObjectMapper(new JacksonObjectMapper(Json.underMapper()))
            .interceptor(new TrackUnirestMDCInterceptor())
            .interceptor(new TrackUnirestLoggingInterceptor())
            .instrumentWith(new TrackUnirestMetricLogger());
    }
}
```

## 日志效果示例

```
[5d5166ea03c249ceb1c5566c75d1f1e4,d6cdace9b2ab4164b6fb11b8ac9abc2a] 2024-05-09 13:41:29,109 INFO  [http-nio-8080-exec-1] c.i.s.t.TrackUnirestMetricLogger:17 > RequestMetrics==>: uri: https://httpbin.org/headers, cost: 1814ms, status: OK
[5d5166ea03c249ceb1c5566c75d1f1e4,d6cdace9b2ab4164b6fb11b8ac9abc2a] 2024-05-09 13:41:29,133 INFO  [http-nio-8080-exec-1] c.i.s.t.TrackUnirestLoggingInterceptor:34 > HTTPInvokeLogging: 
ExecuteRequest==>: GET https://httpbin.org/headers 
AcceptResponse<==: 200, Succed, {"headers":{"Accept":"application/json","Accept-Encoding":"gzip","Content-Type":"application/json","Host":"httpbin.org","User-Agent":"unirest-java/3.1.00","X-Amzn-Trace-Id":"Root=1-663c6208-70dc5ceb0531bf4d61cb3c1e","X-Correlation-Id":"d6cdace9b2ab4164b6fb11b8ac9abc2a"}}
[5d5166ea03c249ceb1c5566c75d1f1e4,d6cdace9b2ab4164b6fb11b8ac9abc2a] 2024-05-09 13:41:29,133 INFO ....
```
