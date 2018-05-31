## HashMap源码解析(JDK1.7)

HashMap是基于Map接口的非同步实现，允许使用null键和null值。HashMap其实是一个“散列的链表”，即数组和链表的结合体。  
HashMap底层是一个数组结构，数组中的每一项又是一个链表。
这里附上一张JDK1.7的HashMap的结构图
![](https://swapp-images.oss-cn-hangzhou.aliyuncs.com/user-head-img/20170704/f8e362f3b86095fc214281688d35ddc6.png)

### 添加一个元素
下面是向HashMap中put一个元素时的代码
```
    public V put(K key, V value) {
        if (table == EMPTY_TABLE) {
            inflateTable(threshold);
        }
        if (key == null)
            return putForNullKey(value);
        int hash = hash(key);
        int i = indexFor(hash, table.length);
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }

        modCount++;
        addEntry(hash, key, value, i);
        return null;
    }
```
从上面的代码中可以看出，先通过hash函数计算KEY的哈希值，接着如果是第一次进入这个函数，那么table一定是null，此时会调用resize初始化数组的大小。
根据哈希值得到这个元素在数组中的下标
```
i = (n-1) & hash
```
如果数组在该位置已经有其他元素了，那么在这个位置的元素将以链表的形式存放，新加入的放在链表头部

### 下标计算
每当要确定一个元素的位置，首先要确定它在数组中的下标，确定下标的代码如下
```
i = (n-1) & hash
```
这个方法很巧妙，通过位运算来获取数组下标。HashMap数组的长度总是2的幂次方，这是一个很巧妙的方法。我们举个例子。
假设数组的长度分别是15和16，则哈希后的数字是8和9，那么&运算后的结果如下：

| (table.length - 1) & hash | table.length - 1 | hash | 结果 |
| :-: | :-: | :-: | :-: |
| (15-1) & 8 | 1110 | 0100 | 0100 |
| (15-1) & 9 | 1110 | 0101 | 0100 |
| (16-1) & 8 | 1111 | 0100 | 0100 |
| (16-1) & 9 | 1111 | 0101 | 0101 | 

从上面的结果可以看出来，当数组长度是15的时候产生了相同的结果，也就是说他们会定位到同一个数组中的同一个位置上，这就产生了碰撞，8和9会被放到数组中的同一个位置上形成链表，这就会降低查询时的效率。另外我们发现任何一个数与(15-1)(1110)与的结果的最优以为永远是0，这样一来0001,0011,0101,1011,0111这几个位置永远都不能存放元素了，空间浪费相当大，同时数组中可用的位置少了很多，进一步增加了碰撞的几率。

所以说，当数组长度为2的幂次方的时候，所有的位置都以相同概率被使用到，数据再数组上的分布较均匀，碰撞几率较小。

### Fail-Fast机制
HashMap不是线程安全的，所以在使用迭代器的过程中如果有其他线程修改了Map，那么HashMap将尽最大努力抛出ConcurrentModificationException异常，这就是常说的Fail-Fast机制，该机制是通过在Map中维护一个modCount域来实现的。

### 面试点
根据代码可以看出，当程序试图将一个键值对放入HashMap中，首先根据该KEY的hashCode的返回值确定该Entry在数组中的存放位置，如果2个Entry的KEY的hashCode相同,那么他们在数组中的存储位置相同。如果这2个Entry的KEY通过equals比较的结果为true，那么新添加的Entry的VALEU将覆盖集合中原有的Entry的VALUE。如果比较的结果为false，则新添加的Entriy将与集合中原有的Entry形成Entry链。
