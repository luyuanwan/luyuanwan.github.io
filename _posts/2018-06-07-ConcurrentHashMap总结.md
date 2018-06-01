## ConcurrentHashMap总结

ConcurrentHashMap1.7和1.8版本有以下一些不同
- 结构上，1.7采用segment数组+链表的结构，1.8采用Node数组+链表/红黑树的结构
- 1.7中对每一个segment加锁，而1.8中对每一个Node加锁，且使用synchronized，可以被JVM更好地优化，锁粒度更小  
- 1.8中链表中的数据个数大于8的时候，转换为红黑树
- 1.7中的size函数是精确值，而1.8中的size函数不是，因为在高并发下获取精确的瞬时值没有有意义
