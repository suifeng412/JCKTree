# ListIterator 源码分析

### Iterator 对比
&emsp;&emsp;IteratorIterator（迭代器）是一种涉及模式，是一个对象，用于遍历集合中的所有元素。  
&emsp;&emsp;IteratorIterator 包含四个方法，分别是：next()、hasNext()、remove()、forEachRemaining(Consumer<? super E> action)   

&emsp;&emsp;IteratorCollection 接口继承 java.lang.Iterable，因此所有 Collection 实现类都拥有 Iterator 迭代能力。  
&emsp;&emsp;Iterator逆向思考，Iterable 面向众多的 Collection 类型实现类，定义的方法就不可能太定制化，因此 Iterator 定义的功能比较简单。  
&emsp;&emsp;Iterator仅有如上所列出来的四种方法，并且该迭代器只能够单向移动。  

&emsp;&emsp;Iterator由于 List 类型的 Collection 是一个有序集合，对于拥有双向迭代是很有意义的。  
&emsp;&emsp;IteratorListIterator 接口则在继承 Iterator 接口的基础上定义了：add(E newElement)、set(E newElement)、hasPrevious()、previous()、nextIndex()、previousIndex() 等方法，使得 ListIterator 迭代能力增强，能够进行双向迭代、迭代过程中可以进行增删改操作。


### 外部现象
TODO 待完善 随风 2019-03-30~31





### 源码分析
TODO 待完善 随风 2019-03-30~31