## ArrayList源码剖析

ArrayList是List接口的可变数组实现，底层是以数组形式实现的，如果数组大小不够了，它会自动扩容。本博客对核心的函数进行分析，对一些边角的、大家容易理解的函数不做分析。

首先它的内部是一个数组，存储的是Object对象
```java
   transient Object[] elementData; 
```

其次，它也会有容量不够的时候，在加入一个元素之前，它会判断容量是否够，不够了，扩容。下面的代码第一行是确保容量是足够的。size表示当前的数组中有效的元素个数。从第二行代码可以看出每次加入的元素都放在最后面。
```java
    public boolean add(E e) {  
        ensureCapacityInternal(size + 1);  // Increments modCount!!  
        elementData[size++] = e;  
        return true;  
    }  
```  
