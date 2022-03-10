# Differ

<!-- toc -->

## 对比逻辑

1. 如果两者是基本类型，则直接使用 `Object.equals` 执行对比。
2. 如果两者是集合类型，则直接使用 `Object.deepEquals` 执行对比。
3. 否则，基于 Getter 或 Field 选择，通过反射获取属性值，执行对比。
4. 支持对比不同类型的对象，默认仅对比二者共有的属性。

> `Arrays.deepEquals` looks really deep. From the source, we could understand that Arrays.deepEquals:
>
> 1. Loops through the input arrays, gets each pair
> 2. Analyses the type of each pair
> 3. Delegates the equal deciding logic to one of the overloaded Arrays.equals if they are one of the primitive arrays
> 4. Delegates recursively to Arrays.deepEquals if it is an Object array
> 5. Calls the respective object’s equals, for any other object
>

如果以上功能不能满足的需要，可以尝试使用  [java-object-differ](https://github.com/SQiShER/java-object-diff)。

## 应用示例

```java
String val1 = "val1";
String val2 = "val2";
List<Integer> list1 = Arrays.asList(1,2);
List<Integer> list2 = Arrays.asList(3,4);
Person person1 = new Person("Anna", 22);
Person person2 = new Person("Luna", 23);

Differ fieldDiffer = Differ.builder()
    .include()
    .exclude()
    .onlyBothExist(true)
    .baseField();

Differ getterDiffer = Differ.builder()
    .include()
    .exclude()
    .onlyBothExist(true)
    .baseGetter();

List<FieldInfo> diff1 = fieldDiffer.diff(val1, val2); 
List<FieldInfo> diff2 = fieldDiffer.diff(list1, list2); 
List<FieldInfo> diff3 = fieldDiffer.diff(person1, person2); 
List<FieldInfo> diff4 = getterDiffer.diff(person1, person2); 
```

