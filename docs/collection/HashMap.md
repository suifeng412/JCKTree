## HashMap

>一种存储键/值关联的数据结构  
>HashMap 的 key 是不可以重复的，保证元素唯一的依据是对象的hashCode跟equals方法  
>HashMap 底层数据结构是散列表  
>HashMap 继承于AbstractMap，实现了Map、Cloneable、java.io.Serializable接口  
>HashMap 的实现是异步的，这意味着它是非线程安全的；它的key、value都可以为null  

### 散列表
&emsp;&emsp;散列表（Hash table，也叫哈希表），是根据键（Key）而直接访问在内存存储位置的数据结构。  
&emsp;&emsp;主要构造是数组（物理存储结构：顺序存储结构），因此对于指定下标的查找，时间复杂度为O(1)。  

&emsp;&emsp;散列表是通过一个函数来计算键（key）的散列值来映射到表中的一个位置来访问记录的。  
&emsp;&emsp;这个映射函数叫散列函数；存放记录的数组叫做散列表。  

**通俗的例子**   
    &emsp;&emsp;需要新增或者查找某个元素，我们把键（key）通过散列函数计算得出一个散列值，然后映射到具体的位置（index）即可。   
>    &emsp;&emsp;index = f(key)    
>    &emsp;&emsp;f(x) 为散列函数    

图片    


**常见的Hash函数有以下几个：**   
1. （加法）直接定址法：取关键字或关键字的某个线性函数值为散列地址。即 hash(k) = k 或者 hash(k) = a * k + b，其中 a b 为常数。（这种散列函数叫做自身函数）
2. （取模）除留余数法：取关键字被某个不大于散列表表长m的数p除后所得的余数为散列地址。即  hash(k) = k mod p , p ≤ m 不仅可以对关键字直接取模。也可在折叠法、平方取中法等运算之后取模。对p的选择很重要，一般取素数或m，若p选择不好，容易产生冲突。
3. （分布均匀：直接取较均匀的数字）数字分析法：假设关键字是以r为基的数，并且哈希表中可能出现的关键字都是事先知道的，则可取关键字的若干数位组成哈希地址。
4. （分布不均匀：先求平方值在按需取）平方取中法：取关键字平方后的中间几位为哈希地址。通常在选定哈希函数时不一定能知道关键字的全部情况，取其中的哪几位也不一定合适，而一个数平方后的中间几位数和数的每一位都相关，由此使随机分布的关键字得到的哈希地址也是随机的。取的位数由表长决定。
5. （分成位数相同几段，几部分相加，舍去最高位）折叠法：将关键字分割成位数相同的几部分（最后一部分的位数可以不同），然后取这几部分的叠加和（舍去进位）作为哈希地址。
6. （随机）随机数法：采用一个伪随机数当作哈希函数。


**哈希冲突**  
&emsp;&emsp;当两个不同的 key 通过散列函数 f(x) 得到的散列值（实际的存储地址）相同，在进行插入的时候，发现该地址已经被其他元素占用，这将会造成哈希冲突，也叫哈希碰撞。
   
**常见的哈希冲突的解决方案有以下几个：**   
1. 开放定址法（open addressing）：一旦发生了冲突，就去寻找下一个空的散列地址，只要散列表足够大，空的散列地址总能找到，并将记录存入
2. 链地址法：将哈希表的每个单元作为链表的头结点，所有哈希地址为i的元素构成一个同义词链表。即发生冲突时就把该关键字链在以该单元为头结点的链表的尾部。
3. 再哈希法：当哈希地址发生冲突用其他的函数计算另一个哈希函数地址，直到冲突不再产生为止。
4. 建立公共溢出区：将哈希表分为基本表和溢出表两部分，发生冲突的元素都放入溢出表中。


**而 HashMap 则采用了链地址法来解决哈希冲突的问题，在JDK 1.8中引入了红黑树，在链表的长度大于等于8并且 hash 桶的长度大于等于64的时候，会将链表进行树化**



### 源码分析

类常量分析  
```java
    // 默认hash桶初始长度16
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16  

    // hash表最大容量2的30次幂
    static final int MAXIMUM_CAPACITY = 1 << 30;

    // 默认负载因子为 0.75
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    // 链表的数量大于等于8个并且桶的数量大于等于64时链表树化 
    static final int TREEIFY_THRESHOLD = 8;

    // hash表某个节点链表的数量小于等于6时树拆分
    static final int UNTREEIFY_THRESHOLD = 6;

    // 树化时最小桶的数量
    static final int MIN_TREEIFY_CAPACITY = 64;

```

实例变量分析
```java
    // hash 桶
    transient Node<K,V>[] table;

    // 保留缓存的 entryset
    transient Set<Map.Entry<K,V>> entrySet;

    // 键值对的数量
    transient int size;

    // HashMap结构修改的次数
    transient int modCount;

    // 扩容的阀值，当键值对的数量超过这个阀值会产生扩容
    // 当table == {}时，该值为初始容量（初始容量默认为16）
    int threshold;

    // 负载因子,代表了table的填充度有多少,默认是0.75
    final float loadFactor;

```

构造函数分析
> HashMap有4个构造器，其他构造器如果用户没有传入initialCapacity 和loadFactor这两个参数，会使用默认值  
> 若没有传入 initialCapacity 和loadFactor这两个参数 则分别默认为 16、0.75  
```java
    // 从下面的构造函数中发现，hash桶没有在构造函数中初始化，而它是在第一次存储键值对的时候才进行初始化
    public HashMap(int initialCapacity, float loadFactor) {
        // 校验初始容量不能小于 0
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        // 校验初始容量不能大于 2的30次幂
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        // 负载因子校验
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        // 负载因子赋值
        this.loadFactor = loadFactor;
        // 根据 initialCapacity 计算扩容阀值
        this.threshold = tableSizeFor(initialCapacity);
    }

    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }


    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }

    public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }


```
tableSizeFor 计算扩容阀值
> 这个方法的作用就是大于或等于 cap 的最小2的幂
> 例如：cap 为6 ，该方法的计算的计算结果为8
```java
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }

```

hash(key) 方法分析
```java


```


put() 方法分析
```java

    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }


    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; 
        Node<K,V> p; 
        int n, i;   // n 用于记录 table 的长度
        // 判断table是否为空，为空则进行初始化（table不是通过构造函数进行初始化，而是通过扩容进行初始化）
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;    // resize() 扩容函数
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }

// TODO 源码分析先放一段时间  完成时间：2019.06 前 随风  
```




