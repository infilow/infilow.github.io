# Scheduler

基于 `ScheduledThreadPoolExecutor` 实现的简单调度器：

```java
// 创建调度器实例：2 线程
Scheduler scheduler = Scheduler.create(2).startup();

// 调度一次：延迟 5 秒
scheduler.scheduleOnce("AAA", () -> System.out.println("AAA"), 5000);
scheduler.scheduleOnce("BBB", () -> System.out.println("BBB"), 5, TimeUnit.SECONDS);

// 周期调度：延迟 2 秒，每 4 秒一次
scheduler.schedule("AAA", () -> System.out.println("AAA"), 2, 4, TimeUnit.SECONDS);
```

