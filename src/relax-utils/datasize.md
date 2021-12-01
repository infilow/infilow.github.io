# DataSize

数据大小的可读表示。

```java
DataSize.ofSuccinct(123).toString();                                    // 123B
DataSize.ofSuccinct(Double.valueOf(5.5 * 1024).longValue()).toString(); // 5.50kB
DataSize.ofSuccinct(3L * 1024 * 1024).toString();                       // 3MB
DataSize.ofSuccinct(3L * 1024 * 1024 * 1024).toString();                // 3GB
DataSize.ofSuccinct(3L * 1024 * 1024 * 1024 * 1024).toString();         // 3TB
DataSize.ofSuccinct(3L * 1024 * 1024 * 1024 * 1024 * 1024).toString();  // 3PB
```

```java
DataSize.of(1234);
DataSize.parse("1234");
DataSize.parse("1234B");
DataSize.parse("1234kB");
DataSize.parse("1234MB");
DataSize.parse("1234GB");
DataSize.parse("1234TB");
DataSize.parse("1234PB");
```
