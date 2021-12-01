# Encoder

用于计算 MD5、Base64、进制转换，避免每次重复编写。进制转换在一些场景中用于缩短数值型字符串的长度。

```java
Encoder.md5("string");

Encoder.asBase64("string");
Encoder.ofBase64("MTU2MTk0NzU3MDYx00==");

String b62 = Encoder.decimalToNumeric(1561947570614L, 62);
System.out.println(b62);            // ruW0PlQ
System.out.println(b62.length());   // 7

long str = Encoder.numericToDecimal(b62, 62);
assertEquals(1561947570614L, str);
```
