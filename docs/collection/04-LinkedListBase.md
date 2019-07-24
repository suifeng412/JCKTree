# LinkedList 基本用法


![LinkedList 基本用法](https://github.com/suifeng412/JCKTree/blob/master/xmind/collection/04-LinkedList基本用法.png)

### demo coding
```java
public class MyTest {

    public static void main(String[] args) throws Exception {
        
          LinkedList linkedList = new LinkedList();
                linkedList.add("aa");
                linkedList.add("bb");
        
                LinkedList linkedList2 = new LinkedList();
                linkedList2.add("bb");
                linkedList2.add("cc");
        
                linkedList.addAll(linkedList2);
        
                System.out.println(linkedList);
        
                linkedList.addFirst("first");
                linkedList.addLast("last");
        
                // [first, aa, bb, bb, cc, last]
                System.out.println(linkedList)
        
                linkedList.removeFirstOccurrence("bb");
                linkedList.pollFirst();
                linkedList.poll();
        
                // [bb, cc, last]
                System.out.println(linkedList);
    }
    
}

```