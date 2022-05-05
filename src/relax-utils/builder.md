# Builder

简单的通用构建器：

```java
Friend friend = Builder.of(Friend::new)
    .with(Friend::setName, "name")
    .with(Friend::setAge, 22)
    .with(Friend::setVitalStatistics, 33, 44, 55)
    .with(Friend::addHobby, "Study")
    .with(Friend::setBirthday, "2000-01-01")
    .with(Friend::setAddress, "Beijing")
    .with(Friend::setEmail, "name@mail.com")
    .with(Friend::setHairColor, "Gray")
    .with(Friend::addGift, "2000-01-01", "Hat")
    .build();
```
