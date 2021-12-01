# Loggable

基于 slf4j-api 获取当前类的 `Logger` 实例，避免手动创建。

```java
public class Service implements Loggable {
    
    public void execute() {
        log().info("start execute...");
    }
}
```

同时可以切换日志级别：

```java
Loggable.switchRootLevel(Level.ERROR);

Loggable.switchLoggerLevel("com.infilos", Level.ERROR);

Loggable.loggers().forEach(logger -> System.out.println(logger.getName()));
```
