## ConcurrentHashMap源码解析(JDK1.7)

ConcurrentHashMap和HashMap差不多，但它是线程安全的，支持并发操作，所以实现上稍微复杂一些。

整个ConcurrentHashMap由一个一个Segment组成，在很多地方这个词被翻译成“段”、“槽”。简单理解，ConcurrentHashMap是一个Segment数组，Segment通过  
继承ReentrantLock来加锁，所以每次需要加锁的操作锁住的是一个segment，这样只要保证每一个segment是线程安全的，也就是实现了ConcurrentHashMap是线程安全的。

![](https://javadoop.com/blogimages/map/3.png)

concurrencyLevel，有的书上称为并发级别，默认是16，也就是ConcurrentHashMap有16个segment，所以理论上，最多支持16个线程并发写。
具体到每一个segment内部，其实很像之前的HashMap，不过它要保证线程安全，所以做的工作要多一些，也要复杂一些。

参考网址：http://www.importnew.com/28263.html
