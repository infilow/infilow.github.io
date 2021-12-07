# Parallel Stream

> - 这部分功能 fork 自 [Parallel Stream Support](https://github.com/ferstl/parallel-stream-support) 以学习其实现方式。
> - 类似的库还有：[Java Stream API Parallel Collectors](https://github.com/pivovarit/parallel-collectors)

并行 Stream 默认是由公用的 [ForkJoinPool](https://docs.oracle.com/en/java/javase/15/docs/api/java.base/java/util/concurrent/ForkJoinPool.html#commonPool()--) 执行。这在大多数情况下没有问题，但无法控制在池中还会执行什么其他类型的任务。比如一些三方库会使用该池执行一些 IO 密集型任务，这会严重影响应用中并行 Stream 的执行性能。但 Stream 并没有提供一个接口以用户自定义池来执行并行 Stream。

本库用于解决该问题，为并行 Stream 提供了专用的 ForkJoinPool。与标准 Stream 接口类似，同时为 Object、int、long、double 提供了操作接口。

```java
public static <T> Stream<T> submit(T[] array, ForkJoinPool workerPool);

public static <T> Stream<T> submit(Spliterator<T> spliterator, ForkJoinPool workerPool);

public static <T> Stream<T> submit(Supplier<? extends Spliterator<T>> supplier, int characteristics, ForkJoinPool workerPool);

public static <T> Stream<T> submit(Builder<T> builder, ForkJoinPool workerPool);

public static <T> Stream<T> iterate(T seed, UnaryOperator<T> operator, ForkJoinPool workerPool);

public static <T> Stream<T> generate(Supplier<T> supplier, ForkJoinPool workerPool);

public static <T> Stream<T> concat(Stream<? extends T> a, Stream<? extends T> b, ForkJoinPool workerPool);
```

## 实例

以指定的并行度执行并行调用：

```java
private static final ForkJoinPool pool2 = new ForkJoinPool(2);

private static final Random random = new Random();
private static final Function<List<Integer>, String> executor = value -> {
    Threads.sleep(random.nextInt(3));

    String result = value.stream().map(String::valueOf).collect(Collectors.joining(","));
    System.out.printf("Execute Chunk(%s): %s%n", System.currentTimeMillis(), result);

    return result;
};

@Test
public void test() {
    executeParallel(executor, oneTo(10), 3, pool2);
}

private <I, R> List<R> executeParallel(Function<List<I>, R> executor, List<I> items, int chunckSize, ForkJoinPool pool) {
    System.out.println("Start:");
    if (items.size() < chunckSize) {
        return Collections.singletonList(executor.apply(items));
    }

    List<List<I>> chunks = CommonStream.splitChunk(items, chunckSize);

    return ParallelStream.submit(chunks, pool).map(executor).collect(Collectors.toList());
}

private static List<Integer> oneTo(Integer stop) {
    return IntStream.range(1, stop +1).boxed().collect(Collectors.toList());
}
```

