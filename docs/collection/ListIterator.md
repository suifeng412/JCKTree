# ListIterator 源码分析

### Iterator 对比
&emsp;&emsp;Iterator（迭代器）是一种涉及模式，是一个对象，用于遍历集合中的所有元素。  
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


**很多书本都有给出的结论：**
+ 链表有 n 个元素，则有 n+1 个位置可以添加新元素；
+ add() 方法只依赖迭代器的+位置；remove() 和 set() 方法依赖于迭代器的状态（此时迭代的方向）；
+ 连续两个 remove() 会出错，remove() 前应先执行 next() 或 previous()。


**迭代时进行修改的问题：**  
&emsp;&emsp;一个迭代器指向另一个迭代器刚刚删除的元素，则现在这个迭代器就变成无效的了（节点删除被回收；即使没被回收，该节点的前后引用也被重置为null）。
链表迭代器有能够检测到这种修改的功能，当发现集合被修改了，将会抛出一个 ConcurrentModificationException 异常

&emsp;&emsp;为什么出现上面的这些现象与问题呢，我们还是从源码中寻找答案吧


### 源码分析
TODO 待完善 随风 2019-03-30~31