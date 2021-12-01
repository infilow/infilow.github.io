# Hash

用于生成不同位数的哈希值。

## string 32

```java
String key = "test";
int seed = 123;
Hash.hash32(key.getBytes(), key.getBytes().length, seed);
```

## int 32

```java
byte[] data = ByteBuffer.allocate(4).putInt(1122).array();
Hash.hash32(data, data.length, seed);
```

## long 32

```java
byte[] data = ByteBuffer.allocate(8).putLong(3344L).array();
Hash.hash32(data, data.length, seed);
```

## double 32

```java
byte[] data = ByteBuffer.allocate(8).putDouble(3344D).array();
Hash.hash32(data, data.length, seed);
```

## string 64

```java
byte[] origin = "value".getBytes();
Hash.hash64(origin, 0, origin.length);
```

## string 128

```java
String key = "test";
int seed = 123;
Hash.hash128(key.getBytes(), 0, key.getBytes().length, seed);
```
