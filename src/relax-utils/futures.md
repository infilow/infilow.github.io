# Futures

## 合并

CompletableFuture 内置 API 提供了 `future.thenCombine(otherFuture, function)` 用于结合两个 Future，Futures 提供了如下 API 用于同时结合更多的 Future。

### Futures.ofList

```java
List<CompletableFuture<String>> futures = asList(completedFuture("a"), completedFuture("b"));
CompletableFuture<List<String>> joined = Futures.ofList(futures);
```

### Futures.ofMap

```java
Map<String, CompletableFuture<String>> futures = new HashMap<>() {{
    put("key", completedFuture("value"));
}};
CompletableFuture<Map<String, String>> joined = Futures.ofMap(futures);
```

### Futures.ofSuccedList

与 `Futures.ofList` 类似，但如果其中一个 Future 失败时，并不会导致 join 失败，而是由 `defaultValueMapper` 为失败的 Future 提供一个默认值。该默认值可以任意设置，可以使 `null` 或 `Optional.empty()`。

```java
List<CompletableFuture<String>> input = Arrays.asList(
    completedFuture("a"),
    exceptionallyCompletedFuture(new RuntimeException("boom"))
);
CompletableFuture<List<String>> joined = Futures.ofSuccedList(input, t -> "default");
```

### Future.joinList

Stream Collector，当为列表中的每个元素应用异步操作时，可以直接将结果收集为一个列表。

```java
collection.stream()
    .map(this::someAsyncFunction)
    .collect(CompletableFutures.joinList())
    .thenApply(this::consumeList)
```

### Futures.joinMap

与 `Future.joinList` 类似，应用于 Map 结构。

```java
collection.stream()
    .collect(joinMap(this::toKey, this::someAsyncFunc))
    .thenApply(this::consumeMap)
```

### Futures.combine

用于合并多个不同类型的 Future。

```java
CompletableFutures.combine(f1, f2, (a, b) -> a + b);
CompletableFutures.combine(f1, f2, f3, (a, b, c) -> a + b + c);
CompletableFutures.combine(f1, f2, f3, f4, (a, b, c, d) -> a + b + c + d);
CompletableFutures.combine(f1, f2, f3, f4, f5, (a, b, c, d, e) -> a + b + c + d + e);
```

### Futures.combineFutures

用于将多个不同类型的 Future 合并为一个新的 Future。

```java
CompletableFutures.combineFutures(f1, f2, (a, b) -> completedFuture(a + b));
CompletableFutures.combineFutures(f1, f2, f3, (a, b, c) -> completedFuture(a + b + c));
CompletableFutures.combineFutures(f1, f2, f3, f4, (a, b, c, d) -> completedFuture(a + b + c + d));
CompletableFutures.combineFutures(f1, f2, f3, f4, f5, (a, b, c, d, e) -> completedFuture(a + b + c + d + e));
```

### Futures.combine(function, vararg)

如果需要合并多于 6 个 Future，使用该多变量参数版本的 `combine` 方法。第一个函数用于结果多个 Future 的值，这种方式使得求值过程从合并过程中分离。

```java
CompletionStage<String> f1;
CompletionStage<String> f2;
CompletionStage<String> result = combine(combined -> combined.get(f1) + combined.get(f2), f1, f2);
```

### Futures.combineFutures(function, vararg)

类似于 `Futures.combine(function, vararg)`。

```java
CompletionStage<String> f1;
CompletionStage<String> f2;
CompletionStage<String> result = dereference(combine(combined -> completedFuture(combined.get(f1) + combined.get(f2)), f1, f2));
```

## 调度

如果有一个长时间轮询以从外部获取资源的接口，可以将其转换为一个 Future。

```java
Supplier<Optional<T>> pollingTask = () -> Optional.ofNullable(resource.result());
Duration frequency = Duration.ofSeconds(2);
CompletableFuture<T> result = CompletableFutures.poll(pollingTask, frequency, executor);
```

## 工具

### Futures.handleCompose

类似于 `CompletableFuture.handle`，但可以返回一个新的 `CompletionStage`，而非直接返回一个值。

```java
CompletionStage<String> composed = handleCompose(future, (value, throwable) -> completedFuture("hello"));
```

### Futures.exceptionallyCompose

类似于 `CompletableFuture.exceptionally`，返回新的 `CompletionStage`。

```java
CompletionStage<String> composed = CompletableFutures.exceptionallyCompose(future, throwable -> completedFuture("fallback"));
```

### Futures.dereference

将 `CompletionStage<CompletionStage<T>>` 解包为 `CompletionStage<T>`。

```java
CompletionStage<CompletionStage<String>> wrapped = completedFuture(completedFuture("hello"));
CompletionStage<String> unwrapped = CompletableFutures.dereference(wrapped);
```

### Futures.ofFailed

创建一个已经失败的 Future。

```java
return CompletableFutures.ofFailed(new RuntimeException("boom"));
```





