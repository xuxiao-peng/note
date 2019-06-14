#  Java 容器



## 一、概览

容器主要包括 Collection 和 Map 两种，Collection 存储着对象的集合，而 Map 存储着键值对（两个对象）的映射表。

# Map

<div align="center"> <img src="../../Markdown/pics/774d756b-902a-41a3-a3fd-81ca3ef688dc.png" width="500px"> </div><br>

## HashMap





[源码分析链接]: https://juejin.im/post/5aa5d8d26fb9a028d2079264#heading-22	"源码分析链接"



存储数据采用的哈希表结构，元素的存取顺序不能保证一致。由于要保证键的唯一、不重复，需 要重写键的hashCode()方法、equals()方法。



hashMap 是 Node<k,v>和链表组成的符合结构

```java
// node 结构图
Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }
```

当 HashMap 里面的元素高于8时，就会链表就会变成红黑树，低于6时就会退化成链表。

```java
/**
     * The bin count threshold for using a tree rather than list for a
     * bin.  Bins are converted to trees when adding an element to a
     * bin with at least this many nodes. The value must be greater
     * than 2 and should be at least 8 to mesh with assumptions in
     * tree removal about conversion back to plain bins upon
     * shrinkage.
     */
    static final int TREEIFY_THRESHOLD = 8;

    /**
     * The bin count threshold for untreeifying a (split) bin during a
     * resize operation. Should be less than TREEIFY_THRESHOLD, and at
     * most 6 to mesh with shrinkage detection under removal.
     */
    static final int UNTREEIFY_THRESHOLD = 6;
```

![1559281552331](C:\Users\P\Desktop\笔记本\img\HashMap-Put方法逻辑.png)





## HashTable

### ConcurrentHashMap

## TreeMap

## LinkedHashMap

# Collection

<div align="center"> <img src="../../Markdown/pics/73403d84-d921-49f1-93a9-d8fe050f3497.png" width="800px"> </div><br>

## Set

### HashSet

### TreeSet

### LinkedHashSet



## List

### ArrayList

### Vector

### LinkedList

## Queue

### LinkedList

### PriorityQueue



# 面试题：

## 1、HashMap、HashTable、ConccurentHashMap 区别

## 2、ArrayList,HashMap,LinkedList 初始化大小和 扩容机制

ArrayList 初始化时，如果不指定容量默认为10，插入数据时，当发现数组不够时自动扩容为1.5倍，然后拷贝到扩容数组中去。

```java
// 扩容代码
private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

HashMap 初始化时 容量为 16 ，装载因子为 0.75，当容量为 0.75*16时，扩容为2倍

LinkedList 不存在扩容。

## 3、HashMap链表转红黑树是怎么实现的