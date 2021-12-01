# Shutdown

用于注册 JVM 关闭时的回调函数。如果需要处理的逻辑较多且有前后依赖，则难于管理。

- `order` 参数用于设置执行顺序，值较小的回调更早执行。
- `order` 值不能重复使用。
- 其中一个回调函数报错不影响其他函数的执行。

```java
Shutdown.setup(1, () -> System.out.println("hook 1"));
Shutdown.setup(2, () -> System.out.println("hook 2"));

Shutdown.isShutingdown();
Shutdown.remove(runnable);
Shutdown.contains(runnable);
```
