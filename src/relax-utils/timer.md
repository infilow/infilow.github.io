# Timer

计时器实现，用于度量执行时长。

```java
Timer timer = Timer.init();

Thread.sleep(300);
timer.span("S1");

Thread.sleep(1100);
timer.span("SS2");

Thread.sleep(700);
timer.stop("SSS3");
```

最终调用 `timer.costSummary()` 得到一个可读的度量视图：

```
Timer include 3 spans, cost 2 Seconds 100 Millis:
   S1├─────────────┐14%: 300 Millis
  SS2              └───────────────────────────────────────────────────┐52%: 1 Seconds 100 Millis
SSSS3                                                                  └───────────────────────────────┤33%: 700 Millis
```

