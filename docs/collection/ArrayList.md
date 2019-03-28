# ArrayList
    ArrayList 是通过一个数组来实现的，因此它是在连续的存储位置存放对象的引用，只不过它比 Array 更智能，能够根据集合长度进行自动扩容。 

假设让我们来实现一个简单的能够自动扩容的数组，我们最容易想到的点就是：
1. add()的时候需要判断当前数组size+1是否等于此时定义的数组大小；
2. 若小于直接添加即可；否则，需要先扩容再进行添加。 

实际上，ArrayList的内部实现原理也是这样子，我们可以来研究分析一下ArrayList的源码

### `add(E e)`源码分析
```java
    /**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return <tt>true</tt> (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);   // 进行扩容校验
        elementData[size++] = e;            // 将值添加到数组后面，并将 size+1
        return true;
    }



    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * will be expanded to DEFAULT_CAPACITY when the first element is added.
     */
    transient Object[] elementData; // non-private to simplify nested class access
    
    private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));    // elementData 数组
    }



    /**
     * Default initial capacity.
     */
    private static final int DEFAULT_CAPACITY = 10;
    
    /**
     * Shared empty array instance used for default sized empty instances. We
     * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
     * first element is added.
     */
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    // 返回最大的 index
    private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {   //  与空数组实例对比
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }



    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
```

**扩容调用方法，实际也就是数组复制的过程**
```java
    /**
     * Increases the capacity to ensure that it can hold at least the
     * number of elements specified by the minimum capacity argument.
     *
     * @param minCapacity the desired minimum capacity
     */
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

### `add(int index, E element)`源码分析
```java
    /**
     * Inserts the specified element at the specified position in this
     * list. Shifts the element currently at that position (if any) and
     * any subsequent elements to the right (adds one to their indices).
     *
     * @param index index at which the specified element is to be inserted
     * @param element element to be inserted
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public void add(int index, E element) {
        rangeCheckForAdd(index);    // 校验index是否超过当前定义的数组大小范围，超过则抛出 IndexOutOfBoundsException

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);     // 复制，向后移动
        elementData[index] = element;
        size++;
    }
    

    /**
     * A version of rangeCheck used by add and addAll.
     */
    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
```


从上面的源码分析可知，扩容和随机插入元素的消耗比较大，因此在实际开发中，应尽量指定ArrayList大小，减少在随机插入操作。


### 优点
+ 封装了一个动态再分配的对象数组
+ 使用索引进行随机访问效率高

### 缺陷
+ 在数组中增删一个元素，所有元素都要往后往前移动，效率低下



### 知识脑图
![ArrayList](https://suifeng-blog.oss-cn-shenzhen.aliyuncs.com/java-collection/ArrayList.png)




