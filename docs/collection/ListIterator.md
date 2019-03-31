# ListIterator 源码分析

### Iterator 对比
&emsp;&emsp;IteratorIterator（迭代器）是一种涉及模式，是一个对象，用于遍历集合中的所有元素。  
&emsp;&emsp;IteratorIterator 包含四个方法，分别是：next()、hasNext()、remove()、forEachRemaining(Consumer<? super E> action)   

&emsp;&emsp;IteratorCollection 接口继承 java.lang.Iterable，因此所有 Collection 实现类都拥有 Iterator 迭代能力。  
&emsp;&emsp;Iterator逆向思考，Iterable 面向众多的 Collection 类型实现类，定义的方法就不可能太定制化，因此 Iterator 定义的功能比较简单。  
&emsp;&emsp;Iterator仅有如上所列出来的四种方法，并且该迭代器只能够单向移动。  

&emsp;&emsp;Iterator由于 List 类型的 Collection 是一个有序集合，对于拥有双向迭代是很有意义的。  
&emsp;&emsp;IteratorListIterator 接口则在继承 Iterator 接口的基础上定义了：add(E newElement)、set(E newElement)、hasPrevious()、previous()、nextIndex()、previousIndex() 等方法，使得 ListIterator 迭代能力增强，能够进行双向迭代、迭代过程中可以进行增删改操作。


### 现象与问题
1. add() 方法在迭代器位置前面添加一个新元素
2. next() 与 previous() 返回越过的对象
3. set() 方法替换的是 next() 和 previous() 方法返回的上一个元素
4. next() 后，再 remove() 则删除前面的元素；previous() 则会删除后面的元素

**很多书本都有给出的结论：**
+ 链表有 n 个元素，则有 n+1 个位置可以添加新元素
+ add() 方法只依赖迭代器的+位置；remove() 和 set() 方法依赖于迭代器的状态（此时迭代的方向）


**迭代时进行修改的问题：**
一个迭代器指向另一个迭代器刚刚删除的元素，则现在这个迭代器就变成无效的了（元素被删除被回收；即使没被删除，仍存在引用指向，那么将会导致元素混乱的状态）。
链表迭代器有能够检测到这种修改的功能，当发现集合被修改了，将会抛出一个 ConcurrentModificationException 异常

TODO 待完善 随风 2019-03-30~31





### 源码分析
TODO 待完善 随风 2019-03-30~31