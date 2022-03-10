# Retry

<!-- toc -->

该功能抄录重构自 [retry4j](https://github.com/elennick/retry4j)，主要为了学习其实现过程，支持以较为简单直观的方式执行重试。

类似的库还有：

- [retry in Scala](https://github.com/softwaremill/retry)
- [failsafe](https://github.com/failsafe-lib/failsafe)
- [resilience4j](https://github.com/resilience4j/resilience4j)
- [guava-retrying](https://github.com/rholder/guava-retrying)

## 1. 配置

> `RetryConfig<T>` 的类型参数 `T` 为返回值的类型，在设置基于返回结果的重试策略时能够确保类型安全。

```java
RetryConfig<String> config = Retry.<String>config()
    // 基于异常重试
    .retryOnError(IllegalArgumentException.class)
    .retryOnErrors(Arrays.asList(IndexOutOfBoundsException.class, IllegalStateException.class))
    .retryOnErrorMatch(e -> e.getMessage().contains("CODE"))
    .retryOnAnyErrorExclude(SQLException.class)
    .retryOnAnyError()
    
    // 匹配异常类型时使用根源异常类型
    .retryOnErrorOfCausedBy()
    
    // 出现任意异常直接失败
    .failOnAnyError()
    
    // 基于返回结果重试
    .retryOnValue("ILLEGAL_VALUE")
    .retryOnValues("ILLEGAL_VALUE_1", "ILLEGAL_VALUE_2")
    .retryOnAnyValueExclude("LEGAL_VALUE_3")
    .retryOnValueMatch(v -> v.startsWith("LEGAL"))
    
    // 重试次数或无限重试
    .withMaxAttempts(2)
    .withInfiniteAttempts()
    
    // 重试之间的延迟间隔
    .withDelayDuration(Duration.ofSeconds(3))
    
    // 重试时的回退策略
    .withFixedBackoff()
    .withExponentialBackoff()
    .withFibonacciBackoff()
    .withNoDelayBackoff()
    .withRandomBackoff()
    .withRandomExponentialBackoff()
    .withCustomBackoff(new BackoffStrategy() {
        @Override
        public Duration nextDelayToWait(int failedAttempts, Duration delayDuration) {
            return null;
        }

        @Override
        public Duration nextDelayToWait(int failedAttempts, Duration delayDuration, Object lastValue, Exception lastError) {
            // 基于当前重试次数、结果、异常，返回下次重试的延迟间隔
            return Duration.seconds(10);
        }
    })
    
    // 监听不同重试时机的状态
    .listenSuccedTry(System.out::println)
    .listenFailedTry(System.out::println)
    .listenCompleteTry(System.out::println)
    .listenBeforeNextTry(System.out::println)
    .listenAfterFailTry(System.out::println)
    
    // 设置异步重试时使用的线程池
    .asyncThreadPool(Executors.newFixedThreadPool(6))
    
    .build();
    
// 或者直接使用预定义好的常见策略
Retry.<String>config().fixedBackoff2Tries3Sec();
Retry.<String>config().fixedBackoff5Tries10Sec();
Retry.<String>config().exponentialBackoff5Tries5Sec();
Retry.<String>config().fiboBackoff7Tries5Sec();
Retry.<String>config().randomExpBackoff10Tries60Sec();
```

## 2. 执行

```java
// 同步重试，将会阻塞当前执行线程
RetryStatus<String> syncStatus = Retry.runSync(config, () -> "result");

// 异步重试，创建新线程或使用提供的线程池
CompletableFuture<RetryStatus<String>> asyncStatus = Retry.runAsync(config, () -> "result");
```
