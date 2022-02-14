# Relax Refine

> Check [Github release](https://github.com/infilow/relax-refine/releases) to get the latest version.

```xml
<dependency>
    <groupId>com.infilos</groupId>
    <artifactId>relax-refine</artifactId>
    <version>0.2.0</version>
</dependency>
```

## 应用场景

通常在开发 HTTP 或 RPC 接口时，需要对请求参数执行必要的校验以避免非法值引起的异常或错误逻辑。

### 前端请求

前端在实现需求时，基于产品设计文档中指定的规则对用户参数执行校验，然后再发起后端调用。这样可以在用户输入时就能通过一些美观的 UI 来提示用户修正非法输入，进一步避免无效的后端调用请求。

**一定要避免的是前端不执行任何用户输入校验，完全依赖于后端的接口参数校验，并将后端校验信息直接暴露给用户。**

**后端校验仅用于确保请求参数的合法性，避免异常和业务逻辑错误，在遇到非法输入时返回有效的错误信息来帮助排查问题。**

### 远程调用

后端远程调用场景中，完备的校验逻辑和详细的错误信息可以有效降低沟通成本。

## 校验方式

1. 最简单的方式是基于 if-else 表达式的校验。
2. 然后可以通过 Guava 提供的 `Preconditions.checkArgument` 来规范校验逻辑。
3. 更进一步是使用 JSR 380 标准校验框架(*Bean Validation 2.0*)来实现自动化校验，但功能有限。
4. 还有一些其他框架实现，用于弥补 JSR 380 内置功能的不足，但有些实现过于复杂。

基于多年开发经验来看，非常现实的问题是：**如果校验逻辑需要大量重复代码且非常冗余，则不够优雅且难以维护；如果校验框架太过复杂则会导致仅会去实现非 NULL 校验或根本不再校验......**

因此，如果我们能够明确应用场景(仅用于后端校验不面向用户)，能够复用校验逻辑，应用过程非常简单，那么校验逻辑是非常值得一写的。

## 功能特性

- 应用过程简单
- 校验逻辑可复用、可扩展
- 一次校验多个变量并打包返回结果，避免反复报错
- 非常简单......

## 开始使用

### 1. 校验变量

```java
RefineResult result1 = Refine.check("Julia", Strings.NotBlank);
```

校验结果 `RefineResult` 封装了成功或失败的信息，可以进一步执行额外的处理，比如转换为统一的错误码和本地化错误信息。

```java
RefineResult result2 = Refine.check(
    Refine.of("case-1").expect("Julia").match(Strings.NotBlank),
    Refine.of("case-2").expect("").match(Strings.IsBlank)
);

Integer val = 12;
RefineResult result3 = Refine.of("case-3").when(() -> val >10).expect(val).match(Numbers.Modulo(5,2)).check();

RefineResult result4 = Refine.of("case-4").expect("value").matchAll(Objects.NotNull(),Strings.NotBlank).check();

RefineResult result5 = Refine.of("case-5").expect("abcde").matchAny(Strings.Startswith("a"),Strings.Startswith("b")).check();

RefineResult result6 = Refine.of("case-6").expect("abcde").matchOne(Strings.Startswith("a"),Strings.Startswith("b")).check();

RefineResult result7 = Refine.of("case-6").expect("abcde").matchNone(Strings.Startswith("f"),Strings.Startswith("g")).check();
```

### 2. 校验 Bean

如果参数较多且有可能重复使用校验逻辑，则应该将参数封装为 Bean。

```java
public class Person implements RefinedType<Person> {
    String name;
    String gender;

    public Person(String name, String gender) {
        this.name = name;
        this.gender = gender;
    }

    @Override
    public RefineResult refine(Person bean) {
        return Refine.check(
            Refine.of("Person.name").expect(name).match(Strings.NotBlank),
            Refine.of("Person.gender").expect(gender).match(Strings.NotBlank)
        );
    }
}
```

注意 `refine` 方法仅仅是对多个变量校验的封装，没有额外逻辑。

### 3. 校验 Bean：附带外部输入

有些校验逻辑需要传入外部变量，比如注入系统环境变量、从配置中心或数据库查询某个值，然后才能执行校验逻辑。

比如实现基于枚举值切换校验逻辑：

```java
public enum CriteriaType {
    Keyword, Content
}

public class Criteria implements RefinedType1<Criteria, CriteriaType> {
    String keyword;
    String content;

    @Override
    public RefineResult refine(Criteria bean, CriteriaType param) {
        CriteriaType checkType = Optional.ofNullable(param).orElse(CriteriaType.Keyword);

        switch (checkType) {
            case Keyword:
                return Refine.of("Criteria.keyword").expect(keyword).match(Strings.NotBlank).check();
            case Content:
                return Refine.of("Criteria.content").expect(content).match(Strings.NotBlank).check();
        }

        throw new UnsupportedOperationException();
    }
}
```

## 逻辑扩展

已经内置了的几种常用类型的数十段校验逻辑：

- Object
- String
- Number
- Time
- Map
- List

如果需要扩展，仅需要实现接口 `Refined` 然后直接在校验代码中使用：

```java
public static Refined<String> IsAlpha = new Refined<String>() {
  @Override
  public String name() {
    return "IsAlpha";
  }

  @Override
  public RefineResult check(String name, String value) {
    if (/*Your Check Logic*/StringUtils.isAlpha(value)) {
			return RefineResult.succed(name, value, this);
		}

		return RefineResult.failed(name, value, this);
  }
};

RefineResult result = Refine.check("Julia", IsAlpha);
```

