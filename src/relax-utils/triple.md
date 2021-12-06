# Triple

三元组：

```java
Triple<String, Integer, Boolean> triple = Triple.of("string", 123, true);
String string = triple.item1();
Integer integer = triple.item2();
Boolean bool = triple.item3();
```
