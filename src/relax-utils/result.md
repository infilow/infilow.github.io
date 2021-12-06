# Result

结果类型封装，成功或失败：

```java
// succed or failed
Result<Integer> succed = Result.succed(123);
Result<Integer> failed = Result.failed(new IllegalArgumentException());

// from supplier
long now = System.currentTimeMillis();
Result<Integer> succedOrFailed = Result.of(() -> {
    if (now % 2 == 0) {
        return 123;
    } else {
        throw new IllegalArgumentException("not a good time");
    }
});

// from optional
Optional<Integer> optional;
if (now % 2 == 0) {
    optional = Optional.of(123);
} else {
    optional = Optional.empty(); 
}
Result<Integer> succedOrNothing = Result.of(optional);  // NoSuchElementException

// from nullable
Integer integerValue;
if (now % 2 == 0) {
    integerValue = 123;
} else {
    integerValue = null;
}
Result<Integer> succedOrNull_1 = Result.ofNullable(integerValue); // NullPointerException
Result<Integer> succedOrNull_2 = Result.ofNullable(integerValue, IllegalArgumentException::new);
```

## Succed or Failed

```java
// check
Result.of(() -> someValue).isSucced();
Result.of(() -> someValue).isFailed();

// transform
Result.of(() -> someValue).ifSucced(System.out::println);
Result.of(() -> someValue).switchIfFailed(e -> {
    if (e instanceof IllegalArgumentException) {
        return Result.succed(123);
    } else {
        return Result.succed(456);
    }
});

// map
Result.of(() -> someValue).map(v -> v * 100);
Result.of(() -> someValue).flatMap(v -> Result.of(() -> v * 100));
Result.of(() -> someValue).mapFalied(e -> {
    if (e instanceof IllegalArgumentException) {
        return new CustomException();
    } else {
        return new RuntimeException();
    }
});
```
