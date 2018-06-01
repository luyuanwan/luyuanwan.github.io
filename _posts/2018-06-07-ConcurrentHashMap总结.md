## ConcurrentHashMap总结

ConcurrentHashMap1.7和1.8版本有以下一些不同
- 1.7中对每一个segment加锁，而1.8中对每一个Node加锁，且使用synchronized，可以被JVM更好地优化，锁粒度更小  
- 1.8中链表中的数据个数大于8的时候，转换为红黑树
