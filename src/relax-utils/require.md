# Require

类似 Guava 的 `Precondition.checkArgument`，避免引入 Guava。

同时提供了一些常见的检查类型，校验条件失败时抛出 `IllegalArgumentException` 异常。

```java
Require.check(1==2);
Require.check(1==2, "1 not equals 2");
Require.check(1==2, "%d not equals %d", 1, 2);

Require.checkNotNull(object);
Require.checkNotNull(object, "object must not be null");
Require.checkNotNull(object), "object named %s must not be null", name);

Require.checkNotBlank(string);
Require.checkNotBlank(string, "string must not be blank");
Require.checkNotBlank(string), "string named %s must not be blank", name);

Require.checkNotEmpty(collectionOrArray);
Require.checkNotEmpty(collectionOrArray, "collection must not be empty");
Require.checkNotEmpty(collectionOrArray), "collection named %s must not be empty", name);
```
