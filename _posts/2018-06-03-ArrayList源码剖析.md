## ArrayList源码剖析

ArrayList是List接口的可变数组实现，底层是以数组形式实现的，如果数组大小不够了，它会自动扩容。本博客对核心的函数进行分析，对一些边角的、大家容易理解的函数不做分析。

首先内部它是一个数组，存储的是Object对象
```java
 /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * will be expanded to DEFAULT_CAPACITY when the first element is added.
     */
transient Object[] elementData; // non-private to simplify nested class access
```
