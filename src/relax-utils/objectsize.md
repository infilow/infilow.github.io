# ObjectSize

计算 Java 对象大小。

```java
ObjectSize.ofMaxJvmHeap();
ObjectSize.ofUsedJvmHeap();
ObjectSize.ofFreeJvmHeap();
ObjectSize.of(object);
```

可以结合 DataSize 使用：

```java
DataSize.ofSuccinct(ObjectSize.of(object)).toString();  
```
