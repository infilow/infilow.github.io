# Resource

用于读取或写入 `src/main/resources` 或 `src/test/resources` 下的文件。

> 注意：文件路径仅包含 `resources` 之后的部分，`/` 可以省略。

```java
File file = Resource.readAsFile("/config.json");
InputStream inputStream = Resource.readAsStream("/config.json");
List<String> lines = Resource.readAsLines("/config.json");
String string = Resource.readAsString("/config.json");
byte[] bytes = Resource.readAsBytes("/config.json");

// File would appear in "/target/classes" or "/target/test-classes".
File writed = Resource.writeResource("/config-updated.json", updatedBytes);
```
