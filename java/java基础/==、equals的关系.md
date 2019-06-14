> 本篇笔记比较 在Java语言中，**==**、**equals()**两个个方法的用处，也就是说为什么要设计三种对象的比较方法呢？

### ==

 `==` 设计目的是为了比较这两个对象是否是同一个对象。

作用：

1. 用于基本数据类型的比较（比较是否相等）

2. 判断引用是否指向堆内存的同一块地址。

### equals

由类自己实现，在`String` 中

```java
 public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```

1、 如果是同一个对象，那么就直接相等

2、如果是其他对象，但值也相等那么他们也相等

### 