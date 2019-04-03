# ListIterator 源码分析

### Iterator 对比
&emsp;&emsp;Iterator（迭代器）是一种设计模式，是一个对象，用于遍历集合中的所有元素。  
&emsp;&emsp;Iterator 包含四个方法，分别是：next()、hasNext()、remove()、forEachRemaining(Consumer<? super E> action)   

&emsp;&emsp;Collection 接口继承 java.lang.Iterable，因此所有 Collection 实现类都拥有 Iterator 迭代能力。  
&emsp;&emsp;逆向思考，Iterable 面向众多的 Collection 类型实现类，定义的方法就不可能太定制化，因此 Iterator 定义的功能比较简单。  
&emsp;&emsp;仅有如上所列出来的四种方法，并且该迭代器只能够单向移动。  

&emsp;&emsp;由于 List 类型的 Collection 是一个有序集合，对于拥有双向迭代是很有意义的。  
&emsp;&emsp;ListIterator 接口则在继承 Iterator 接口的基础上定义了：add(E newElement)、set(E newElement)、hasPrevious()、previous()、nextIndex()、previousIndex() 等方法，使得 ListIterator 迭代能力增强，能够进行双向迭代、迭代过程中可以进行增删改操作。


### 现象与问题
1. add() 方法在迭代器位置前面添加一个新元素
2. next() 与 previous() 返回越过的对象
3. set() 方法替换的是 next() 和 previous() 方法返回的上一个元素
4. next() 后，再 remove() 则删除前面的元素；previous() 则会删除后面的元素

```java
        List<String> list = new LinkedList<>();
        list.add("aaa");
        list.add("bbb");
        list.add("ccc");

        ListIterator<String> listIterator = list.listIterator();

        //迭代器位置： add-1 | aaa bbb ccc
        listIterator.add("add-1");
        // add-1 add-1 | aaa bbb ccc
        listIterator.add("add-2");

        // 返回： aaa
        // add-1 add-1 aaa | bbb ccc
        listIterator.next();
        // add-1 add-1 aaa-set | bbb ccc
        listIterator.set("aaa-set");
        // bbb
        // add-1 add-1 aaa-set bbb | ccc
        listIterator.next();

        // 返回： bbb
        // add-1 add-1 aaa-set | bbb ccc
        listIterator.previous();
        // add-1 add-1 aaa-set | bbb-set ccc
        listIterator.set("bbb-set");

        // 删除 bbb-set
        listIterator.remove();
        listIterator.remove();

        System.out.println(list);
```


**很多书本都有给出这样子的结论：**
+ 链表有 n 个元素，则有 n+1 个位置可以添加新元素；
+ add() 方法只依赖迭代器的+位置；remove() 和 set() 方法依赖于迭代器的状态（此时迭代的方向）；
+ 连续两个 remove() 会出错，remove() 前应先执行 next() 或 previous()。


**迭代同时修改问题：**  
&emsp;&emsp;一个迭代器指向另一个迭代器刚刚删除的元素，则现在这个迭代器就变成无效的了（节点删除被回收；即使没被回收，该节点的前后引用也被重置为null）。
链表迭代器有能够检测到这种修改的功能，当发现集合被修改了，将会抛出一个 ConcurrentModificationException 异常

&emsp;&emsp;为什么出现上面的这些现象与问题呢，我们还是从源码中寻找答案吧

### 源码分析
&emsp;&emsp;有多个集合类根据自己的特点实现了 ListIterator 接口，其实现都大同小异，这里我们主要分析 LinkedList 中所实现的 ListIterator。  

&emsp;&emsp;首先我们来分析 LinkedList 的 listIterator() 和 listIterator(int index) 方法获取 ListIterator 迭代器过程。  

```java
// AbstractList.java
// listIterator() 方法 LinkedList 类本身并没有重写，需要追溯到 AbstractList 抽象类

    // 获取 ListIterator 迭代器
    public ListIterator<E> listIterator() {
        return listIterator(0);
    }

    public ListIterator<E> listIterator(final int index) {
        rangeCheckForAdd(index);    // 检查 index 范围是否超出

        return new ListItr(index);  // 该抽象类也有实现 ListItr 类
    }

    private void rangeCheckForAdd(int index) {
        if (index < 0 || index > size())
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
```

```java
// LinkedList.java
// LinkedList 类重写了 listIterator(int index) 方法

    public ListIterator<E> listIterator(int index) {
        checkPositionIndex(index);  // 同理 检查 index 范围；相关代码就不贴了
        return new ListItr(index);
    }


    private class ListItr implements ListIterator<E> {
        private Node<E> lastReturned;   // 上一次处理的节点
        private Node<E> next;   // 即将要处理的节点
        private int nextIndex;  // 即将要处理的节点的 index
        // modCount 表示集合和迭代器修改的次数；expectedModCount 表示当前迭代器对集合修改的次数
        private int expectedModCount = modCount;

        ListItr(int index) {
            // assert isPositionIndex(index);
            next = (index == size) ? null : node(index);
            nextIndex = index;
        }

        public boolean hasNext() {
            return nextIndex < size;
        }

        /**
        * 处理对象：迭代器当前的 next 节点
        * 将处理目标储到 lastReturned 变量中
        * 然后将当前的 next.next 节点保存起来，用于下一次迭代处理
        * nextIndex 同时 +1
        * 返回 lastReturned.item 元素
        * 执行后：lastReturned 指向该次处理的节点；next、nextIndex 指向该次处理节点的后一个节点
        */
        public E next() {
            // 检查 modCount 与 expectedModCount 是否相等
            // 实际检查该链表是否被其他迭代器或者集合本身修改
            checkForComodification();
            // 判断是否存在 next 节点
            if (!hasNext())
                throw new NoSuchElementException();
            
            lastReturned = next;    // 将这次返回的 node 节点更新到迭代器中的 lastReturned 变量
            next = next.next;   // 将下一次需要处理 node 节点更新会 next 变量
            nextIndex++;    // 变量 nextIndex +1
            return lastReturned.item;   // 返回元素
        }

        public boolean hasPrevious() {
            return nextIndex > 0;
        }

        /**
        * 处理对象：迭代器当前的 next.prev 节点
        * 将处理目标储到 lastReturned 变量中
        * 然后将当前的 next.prev 节点保存起来，用于下一次迭代处理
        * nextIndex 同时 -1
        * 返回当前的 next.item 元素
        * 执行后：next、lastReturned、nextIndex 指向该次处理节点的前一个节点
        */
        public E previous() {
            checkForComodification();
            // 判断是否存在 prev 节点
            if (!hasPrevious())
                throw new NoSuchElementException();

            // 处理当前 next 的 prev 节点
            // 特殊情况：next = null 时，则它的 prev 节点为 last 节点  
            lastReturned = next = (next == null) ? last : next.prev;    
            nextIndex--;    // nextIndex -1
            return lastReturned.item;
        }

        public int nextIndex() {
            return nextIndex;
        }

        public int previousIndex() {
            return nextIndex - 1;
        }

        /**
        * 处理对象：lastReturned
        * 删除 lastReturned 指向的节点，并置为 null
        * 同时保证 next 和 nextIndex 指向同一个节点
        */
        public void remove() {
            checkForComodification();   // 同理， 检查 modCount 与 expectedModCount 是否相等
            if (lastReturned == null)
                throw new IllegalStateException();

            Node<E> lastNext = lastReturned.next;  // 暂存 lastReturned 的 next 节点，用于恢复迭代状态
            unlink(lastReturned);   // 删除最后返回的节点    modCount++;
            
            // 分迭代方向处理（因为删除一个节点后，需要恢复迭代状态：next 和 nextIndex 指向同一个节点）
            if (next == lastReturned)   // next 与 lastReturned 节点相同则表明最近一次迭代操作是 previous()
                next = lastNext;        // 删除了原有 next 指向的节点，因此 nextIndex 相对指向的节点变为 next.next，需要更新 next 变量的指向
            else
                nextIndex--;    // next() 迭代方向；删除了next前面的节点，因此next的相对位置发生变化，需要 nextIndex -1
            lastReturned = null;    
            expectedModCount++;     // 同时 expectedModCount++
        }

        /**
        * 处理对象：lastReturned
        */
        public void set(E e) {
            if (lastReturned == null)
                throw new IllegalStateException();
            checkForComodification();
            lastReturned.item = e;
        }

        /**
        * 分位置进行添加
        */
        public void add(E e) {
            checkForComodification();
            lastReturned = null;
            if (next == null)
                linkLast(e);
            else
                linkBefore(e, next);
            nextIndex++;
            expectedModCount++;
        }

        public void forEachRemaining(Consumer<? super E> action) {
            Objects.requireNonNull(action);
            while (modCount == expectedModCount && nextIndex < size) {
                action.accept(next.item);
                lastReturned = next;
                next = next.next;
                nextIndex++;
            }
            checkForComodification();
        }

        /**
        * 检查 modCount 与 expectedModCount 是否相等，否则抛出错误
        * ListIterator 迭代器进行增删操作时，都会同时对这两个变量 +1
        * 目的：
        * 使用 ListIterator 迭代器期间，LinkedList 对象有且只能当前这一个迭代器可以进行修改
        * 避免 LinkedList 对象本身以及其他迭代器进行修改导致链表混乱
        */
        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
    }

```

### 小结
>&emsp;&emsp;总的来说 ListIterator 是记录 List 位置的一个对象，它主要的成员变量是 lastReturned、next、nextIndex 以及 expectedModCount。
>1. next() 处理的是 next 节点，返回 next.item
>2. previous() 处理的是 next.prev 节点 返回 next.prev.item
>3. remove() 处理的是 lastReturned 节点，并置为null，但要注意的是，删除节点后的 next 与 nextIndex 需分情况处理。
>4. set() 处理的是 lastReturned 节点，lastReturned.item = e
>5. add() 添加，并将 lastReturned 置为null
>
>&emsp;&emsp;这就很好地解释上面所提到的一些现象与问题了。  
>&emsp;&emsp;典型的就是连续两个 remove() 会报错，那是因为第一个 reomve() 之后 lastReturned 被置为null；第二个 remove() 处理的对象是null，因此炮锤 IllegalStateException


### 知识脑图
![ListIterator](https://suifeng-blog.oss-cn-shenzhen.aliyuncs.com/java-collection/ListIterator%E8%84%91%E5%9B%BE.png)
