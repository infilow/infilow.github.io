# Datetimes

日期时间操作方法。

```java
Datetimes.nowInMills();
Datetimes.nowInNanos();
Datetimes.nowInMillsOfNano();
Datetimes.now();
Datetimes.from("1942", Datetimes.Patterns.AtYears);
Datetimes.from("1980-01", Datetimes.Patterns.AtMonths);
Datetimes.from("1980-11-01", Datetimes.Patterns.AtDays);
Datetimes.from("1980-11-01 18", Datetimes.Patterns.AtHours);
Datetimes.from("1980-11-01 18:42", Datetimes.Patterns.AtMinutes);
Datetimes.from("1980-11-01 18:42:33", Datetimes.Patterns.AtSeconds);
Datetimes.from(1638363075578L);
Datetimes.asMills(localdatetime);
Datetimes.asSeconds(localdatetime);
```
