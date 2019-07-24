## LinkedList
&emsp; &emsp;LinkedList是一种可以在任何位置进行高效地插入和删除操作的有序序列。  
&emsp; &emsp;它的最基本存储结构是一个节点：每个节点将存储对象，以及前后节点的引用。


### 结构图
![LinkedList 结构体](https://github.com/suifeng412/JCKTree/blob/master/xmind/collection/05-LinkedList结构图.jpg)



&emsp; &emsp;从上面的结构图中，我们可以了解到 ListedList 底层是基于双向链表实现的。  
&emsp; &emsp;围起来的可以看成 LinkedList 类，它定义了三个 transient 成员变量：first、last、size。这三个变量是整个 LinkedList 类的关键点。
1. 由于是双向链表（每个node都有保存前后节点的引用），因此我们不管是由 first 还是 last 节点开始迭代，都可以将整个链表的数据找出来；
2. 在查询、随机插入以及set等操作都有涉及 size 判断；
3. 由于 LinkedList 是双向链表，类中只存储了首尾两个节点，因此查询第n个元素都要从头遍历进行查找。


### 源码分析

#### add(E e) 方法分析
```java
    /**
     * Appends the specified element to the end of this list.
     *
     * <p>This method is equivalent to {@link #addLast}.
     *
     * @param e element to be appended to this list
     * @return {@code true} (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        linkLast(e);
        return true;
    }
    
    /**
     * Links e as last element.
     */
    void linkLast(E e) {
        final Node<E> l = last;                             // 将当前最后一个元素寄存在 l
        final Node<E> newNode = new Node<>(l, e, null);     // new 一个新节点：pre的引用为l；存储元素为e；next的引用为null
        last = newNode;                                     // 将新节点引用覆盖成员变量 last
        if (l == null)                                      
            first = newNode;                                // 若l为null，说明之前链表为空，此时新节点为首个元素
        else
            l.next = newNode;                               // 否则，更新l的next引用
        size++;                                             // size+1
        modCount++;                                         // 非查询操作 modCount 都会 +1
    }
```

#### add(int index, E element) 方法分析
```java
    /**
     * Inserts the specified element at the specified position in this list.
     * Shifts the element currently at that position (if any) and any
     * subsequent elements to the right (adds one to their indices).
     *
     * @param index index at which the specified element is to be inserted
     * @param element element to be inserted
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public void add(int index, E element) {
        checkPositionIndex(index);  // 检查 index 是否大于 size

        if (index == size)
            linkLast(element);      // 直接在链表末尾追加
        else
            linkBefore(element, node(index));   // 插入index 节点前面
    }
    
    
    // 检查 index 是否超出范围 超出则抛出 IndexOutOfBoundsException
    private void checkPositionIndex(int index) {
        if (!isPositionIndex(index))
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    /**
     * Tells if the argument is the index of a valid position for an
     * iterator or an add operation.
     */
    private boolean isPositionIndex(int index) {
        return index >= 0 && index <= size;
    }
    
    
    
    /**
     * 根据 index 查找 node
     * 该方法利用了双向链表的特性，index 距离哪个链表头近就从哪边开始开始遍历
     * 时间复杂度为 O(n/2)；
     * 当 index 接近 size 的中间值时，效率最低
     * Returns the (non-null) Node at the specified element index.
     */
    Node<E> node(int index) {
        // assert isElementIndex(index);

        if (index < (size >> 1)) {          // size 右移一位（除以2）
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }

```


#### 优点
+ 增删元素效率高（只需要更新节点附近的引用即可）


#### 缺点
+ 由于查询需要进行遍历，因此效率低

### 知识脑图
![LinkedList 脑图](https://github.com/suifeng412/JCKTree/blob/master/xmind/collection/05-LinkedList源码分析.png)
