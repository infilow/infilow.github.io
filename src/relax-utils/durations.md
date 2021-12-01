# Durations

解析表示时间间隔的字符串。`Duration` 的默认格式中包含一些特殊字符，该方法会自动补充这些字符，以简化应用过程。

```java
/*
 * Y: year
 * M: month
 * D: day
 * H: hour
 * M: minute
 * S: second
 */
Duration duration = Durations.parse("1d2h3s");
```
