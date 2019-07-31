# HashMap

>一种存储键/值关联的数据结构  
>HashMap 的 key 是不可以重复的，保证元素唯一的依据是对象的hashCode跟equals方法  
>HashMap 底层数据结构是散列表  
>HashMap 继承于AbstractMap，实现了Map、Cloneable、java.io.Serializable接口  
>HashMap 的实现是异步的，这意味着它是非线程安全的；它的key、value都可以为null  

## 散列表
&emsp;&emsp;散列表（Hash table，也叫哈希表），是根据键（Key）而直接访问在内存存储位置的数据结构。  
&emsp;&emsp;主要构造是数组（物理存储结构：顺序存储结构），因此对于指定下标的查找，时间复杂度为O(1)。  

&emsp;&emsp;散列表是通过一个函数来计算键（key）的散列值来映射到表中的一个位置来访问记录的。  
&emsp;&emsp;这个映射函数叫散列函数；存放记录的数组叫做散列表。  

**通俗的例子**   
    &emsp;&emsp;需要新增或者查找某个元素，我们把键（key）通过散列函数计算得出一个散列值，然后映射到具体的位置（index）即可。   
>    &emsp;&emsp;index = f(key)    
>    &emsp;&emsp;f(x) 为散列函数    

图片    


### 常见的Hash函数有以下几个：
1. （加法）直接定址法：取关键字或关键字的某个线性函数值为散列地址。即 hash(k) = k 或者 hash(k) = a * k + b，其中 a b 为常数。（这种散列函数叫做自身函数）
2. （取模）除留余数法：用关键字 k 除以某个不大于哈希表长度 m 的数 p，所得余数为散列地址。即  hash(k) = k mod p , p ≤ m 不仅可以对关键字直接取模。也可在折叠法、平方取中法等运算之后取模。对p的选择很重要，一般取素数或m，若p选择不好，容易产生冲突。
3. （分布均匀：直接取较均匀的数字）数字分析法：提取关键字中取值比较均匀的数字作为哈希地址。
4. （分布不均匀：先求平方值在按需取）平方取中法：如果关键字各个部分分布都不均匀的话，可以先求出它的平方值，然后按照需求取中间的几位作为哈希地址。
5. （分成位数相同几段，几部分相加，舍去最高位）折叠法：将关键字分割成位数相同的几部分（最后一部分的位数可以不同），然后取这几部分的叠加和（舍去进位）作为哈希地址。
6. （随机）随机数法：采用一个伪随机数当作哈希函数。


### 哈希冲突  
&emsp;&emsp;当两个不同的 key 通过散列函数 f(x) 得到的散列值（实际的存储地址）相同，在进行插入的时候，发现该地址已经被其他元素占用，这将会造成哈希冲突，也叫哈希碰撞。
   
**常见的哈希冲突的解决方案有以下几个：**   
1. 开放定址法（open addressing）：当发生哈希冲突时，寻找一下个空闲位置进行存放，存在查找性能差、扩容频繁等缺点
2. 再哈希法：当哈希地址发生冲突用其他的函数计算另一个哈希函数地址，直到冲突不再产生为止，缺点就是增加了计算的时间
3. 建立公共溢出区：将哈希表分为基本表和溢出表两部分，发生冲突的元素都放入溢出表中。
4. 链地址法：采用数组+链表数据结构，当发生哈希冲突时，使用一个链表来存储相同的 Hash 值数据，查找一个元素时间复杂度为 O(1+n)。



**而 HashMap 则采用了链地址法来解决哈希冲突的问题，在JDK 1.8中引入了红黑树，在链表的长度大于等于8并且 hash 桶的长度大于等于64的时候，会将链表进行树化**



## 源码分析

### 类常量分析  
```java
    // 默认hash桶初始长度16 即 Node<K,V>[] 的长度
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16  

    // hash 桶数最大值为2的30次幂
    static final int MAXIMUM_CAPACITY = 1 << 30;

    // 默认负载因子为 0.75
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    // 链表的数量大于等于8个并且桶的数量大于等于64时链表树化 
    static final int TREEIFY_THRESHOLD = 8;

    // 桶中的链表长度小于等于6时树拆分
    static final int UNTREEIFY_THRESHOLD = 6;

    // 树化时最小桶的数量
    static final int MIN_TREEIFY_CAPACITY = 64;

```

### 实例变量分析
```java
    // hash 桶数组（Node数组）
    transient Node<K,V>[] table;

    // 保留缓存的 entryset
    transient Set<Map.Entry<K,V>> entrySet;

    // table 已存元素数量（实际已使用的桶数量）
    transient int size;

    // HashMap结构修改的次数
    transient int modCount;

    // 扩容的阀值，当已使用桶的数量超过这个阀值会进行扩容 resize()
    // 计算公式为：capacity * load factor（总桶数 * 负载因子）
    int threshold;

    // 负载因子,代表了 table 的填充度有多少,默认是0.75
    final float loadFactor;

```

### 内部类 Node 分析
```java
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        // next 指针；当发生哈希碰撞时新 Node 元素的引用将存储在 next 指针，形成一个链表
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }
```

### 构造函数分析
> HashMap有4个构造器，若用户没有传入initialCapacity 和loadFactor这两个参数，将分别使用 16、0.75 默认参数
```java
    // 无参构造方法
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
    }

    // 带初始元素的构造方法，执行 putMapEntries 方法进行添加
    public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }
    
    // 带初始桶数（容量）构造方法 =》增加默认参数，调用另一个构造方法
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    } 

    // 带初始桶数（容量）、负载因子的构造方法
    public HashMap(int initialCapacity, float loadFactor) {
        // 校验初始桶数不能小于 0
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        // 校验初始桶数不能大于 2的30次幂
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        // 负载因子校验
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        // 负载因子赋值
        this.loadFactor = loadFactor;
        
        // tableSizeFor 方法的作用就是返回大于或等于 初始桶数（容量） 的最小2次幂 =》 即不管传入的 initialCapacity 为何值，最终的 table 长度都是2的幂次方（原因下文解答）
        // 为什么会将 初始桶数 存放在扩容阈值中呢？
        // 因为 HashMap 没有定义桶数的属性，只能暂时寄存在 threshold 中，直到第一次进行键值对存储容量扩容时，才会对 table、threshold 初始化
        this.threshold = tableSizeFor(initialCapacity);
    }
```


### tableSizeFor
> 这个方法的作用就是返回大于或等于 cap 的最小2次幂
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


### 获取数组下标源码分析（hash(key)源码分析、`(n - 1) & hash`分析）
以 put(key, value) 路径进行分析  
1. put(K key, V value)   
2. hash(key)  
3. putVal() =》i = (n - 1) & hash

**put(K key, V value) 源码**
```java
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
```

**hash(Object key) 方法分析**
```java
    static final int hash(Object key) {
        int h;
        // 获取 key 的 hashCode，与 hashCode 无符号右移16位进行异或操作
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
    
    // 最后在 putVal() 方法中使用 (n - 1) & hash 获取 key 的下标
     
```
+ 为什么要通过 hash(Object key) 方法再次处理 hashCode 值，而不是直接使用 hashCode？  
&emsp;&emsp;在最后一步获取下标中`(n - 1）& hash`，n 的一般取值（数组长度）相对于 int 范围来说是很小的，当 n 取值为 65535（2的16次方） 的时候，hash 的高16位才会参与获取下标的计算。    
&emsp;&emsp;在 `hash(Object key)` 方法中，将 key 的 hashCode，与 hashCode 无符号右移16位进行异或操作 =》 即将 hashCode 的高16位与低16位进行异或操作，高位16位不变，返回的 hash 再进行下标的'与'操作。    
&emsp;&emsp;这样做的目的就是，充分混合高位低位的特征，减少哈希碰撞。  
    
+ 为何不使用'取模'操作`i = hash % n`，而是通过'与'操作 `i = (n - 1) & hash`来获取下标呢？    
&emsp;&emsp;这两者的操作是等价，由于计算机底层运算是基于二进制的，因此 `&` 比 `%` 效率更高  
  
+ 为什么桶的数量（table数组长度）必须是2次幂呢？  
&emsp;&emsp;2的幂次方数减1的二进制的每一位都是1，保证数组中的每一个位置都能添加到元素 => (n -1) & hash  
  
**图解**
123132313



### put() 方法分析
```java
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; 
        Node<K,V> p; 
        int n, i;   // n 用于存储 tab 长度   
        
        // 判断当 table 为 null 或者 tab 的长度为 0 时，即 table 尚未初始化，此时通过 resize() 方法得到初始化的 table
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;    // n 存储最新 tab 的长度
            
        // 通过（n - 1） & hash 计算出的值作为 tab 的下标 i，
        // 并将 tab[i] 引用赋值到 p 中（该链表第一个节点的位置）
        // 并判断 p 是否为 null
        // 当 p 为 null 时，表明 tab[i] 上没有任何元素，那么接下来就 new 第一个 Node 节点，调用 newNode 方法返回新节点引用给 tab[i]
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            // p 不为 null 有四种情况：链表头、链表中、链表尾、红黑树
            Node<K,V> e;    // 定义一个 Node 节点，用于存放需要更新的节点引用
            K k;
            // 第一种情况，链表头 => 新 key 与 p 节点中的key相等,且 hash 也相等 =》直接更新当前节点的 value 即可（这里先将 p 引用赋值到 e 后续再处理） 
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            // 第二种情况，p 是红黑树节点 => 那么肯定插入后仍然是红黑树节点，所以我们直接强制转型 p 后调用 TreeNode.putTreeVal 方法，返回的引用赋给 e
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);     // TreeNode 是 Node 的一个子类
            else {
                // 遍历当前链表； binCount 用于记录链表个数
                for (int binCount = 0; ; ++binCount) {
                    // 第三种情况，链表末尾 => 当前节点的 next 为 null => 可将新元素存入
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        
                        // 插入成功后，需判断是否需要转换为红黑树
                        // 因为插入后链表长度加 1，而 binCount 并不包含新节点，因此判断时要将临界阈值减 1
                        if (binCount >= TREEIFY_THRESHOLD - 1) 
                            treeifyBin(tab, hash);  // 调用 treeifyBin 方法，将该链表转换为红黑树
                        break;
                    }
                    
                    // 第四种情况，链表中 => 判断新 key 是否与链表中节点的 key 完全相同，若是直接更新该节点 value 即可
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    
                    p = e;
                }
            }
            
            // 根据条件处理 e 节点的 value 值；返回旧 value
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                // 回调函数
                afterNodeAccess(e); 
                return oldValue;
            }
        }
        
        // 当往 table 数组添加实际元素的时候 modCount 加 1、size 加 1
        ++modCount;
        // size 加1、根据阈值判断是否需要扩容
        if (++size > threshold)
            resize();
        // 回调函数
        afterNodeInsertion(evict);
        return null;
    }


```


### resize() 扩容分析
```java
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;     // 用于暂存 table
        int oldCap = (oldTab == null) ? 0 : oldTab.length;  // 旧 table 桶数（容量、数组长度）
        int oldThr = threshold;         // 旧扩容阈值
        int newCap, newThr = 0;         // 定义新的 table 桶数、新的扩容阈值 局部变量
        
        // 判断旧 table 桶数，是否初始化
        if (oldCap > 0) {
            // 若旧 table 桶数已经超过定义的最大长度，则将阈值设置为最大值，不再进行扩容
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            // 新的 table 桶数（容量）扩容为原来的两倍、扩容阈值也扩大为原来的两倍
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        // 扩容阈值已经初始化过，例如：带初始桶数（容量）、负载因子的构造方法
        // 首次插入数据初始化扩容
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        // 没有任何初始化值，例如：无参构造方法
        // 首次插入数据初始化扩容
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        
        // 新的扩容阈值 局部变量赋值
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;     // 为当前阀值赋值
        
        // 初始化 table 的桶数（容量）
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        
        // 如果旧的 table 不为空，需要将里面的所有数据重新映射到 新的 table 中
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                
                // 判断该下标元素是否为 null
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    // 只有一个节点，直接使用新桶数 n 获取下标直接存储
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    // 如果是红黑树，需要进行树拆分然后映射
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    
                    // 多个节点链表拆分      
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            // 根据 (e.hash & oldCap) == 0 拆分为两个链表，
                            // 索引位置，一个为原索引，一个为原索引加上旧 Hash 桶长度的偏移量
                            // 具体原因下面分析
                            next = e.next;
                            // 链表 1
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            // 链表 2
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        // 链表1存于原索引
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        // 链表2存于原索引加上原 hash 桶长度的偏移量
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }

```

+ 为什么可以根据`(e.hash & oldCap) == 0`就可以快速将原链表拆分，并知道在新 table 中的下标？  
&emsp;&emsp;计算下标公式为：`index = (table.length - 1) & hash`，由于 `table.length` 即 `capacity` 肯定是2的N次方，在扩容中新桶数为原来两倍即`capacity << 1`。在使用`&`运算符，意味只需计算新增的一位 + 原下标 即可。  
图解。。。。。


## 小结
+ HashMap 运行 NULL 值，NULL 键  
+ 不要轻易改变负载因子，过大，链表会过长，增加查找时间复杂度；过小，桶数量过多，增加空间复杂度  
+ 每次扩容桶数为原来两倍  
+ HashMap 线程不安全  
+ 尽量设置HashMap的初始容量，尤其在数据量大的时候，防止多次resize   



参考资料：
https://juejin.im/post/5ab99afff265da23a2291dee  
https://zhuanlan.zhihu.com/p/34280652  