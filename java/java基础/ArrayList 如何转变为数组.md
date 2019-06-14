## ArrayList 如何转变为数组

```java
// 方案1
String[] a = new String[mset.size()];
        mset.toArray(a);
```

这时候我试过 `String[] array= (String[]) list.toArray();`的写法，但是会抛出 ` ava.lang.ClassCastException: [Ljava.lang.Object; cannot be cast to [Ljava.lang.String;` 的异常。

```java
String[] array= (String[]) list.toArray();
```

这是因为  java 不能自动将 Object 数组转换为 String 数组。

