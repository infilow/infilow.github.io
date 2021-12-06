# Pool

简单对象池封装。

## 创建

```java
// Const pool
Pool.fixedConsts(Arrays.asList("a", "b"));

// SimplePool
Pool.builder()
  .capacity(1)
  .creator(Object::new)
  .build();

// ExpiringPool
Pool.builder()
  .capacity(1)
  .creator(Object::new)
  .maxIdleTime(Duration.ofSeconds(1))
  .build();

// Weak reference pool, collect by GC
Pool.builder()
  .capacity(1)
  .creator(Object::new)
  .refType(PoolRefType.Weak)
  .build();

// Soft reference pool, notify by GC but collect by OOM
Pool.builder()
  .capacity(1)
  .creator(Object::new)
  .refType(PoolRefType.Soft)
  .build();

// fully build
AtomicInteger counter = new AtomicInteger(0);
Pool.builder()
  .capacity(1)                                  // 总容量
  .creator(Object::new)                         // 新元素创建起 
  .refType(PoolRefType.Soft)                    // 引用类型
  .reseter(obj -> counter.incrementAndGet())    // 用于重置池中元素状态
  .disposer(obj -> counter.incrementAndGet())   // 用于销毁池中元素
  .checker(obj -> Objects.nonNull(obj))         // 用于检查池中元素可用性
  .maxIdleTime(Duration.ofMillis(100))          // 最大空闲时间，之后被逐出
  .build();
```

## 操作

```
pool.fill(); 		// 填充元素
pool.clear(); 		// 清空元素
pool.close();		// 销毁元素并关闭池

pool.acquire().get();               // 获取值
pool.acquire().release();           // 释放值
pool.acquire().invalidate();        // 销毁-不再返回池
pool.acquire().map(function);       // 执行函数并释放

pool.refType();		// 引用类型
pool.capacity();	// 容量
pool.live();		// 已创建数量：池中+已租借
pool.size();		// 池中数量
pool.leased();		// 已租借数量
```

