# 集合介绍


![ArrayList 基础用法](https://github.com/suifeng412/JCKTree/blob/master/xmind/collection/02-ArrayList基础用法.png)

### demo coding
```java
public class MyTest {

    public static void main(String[] args) throws Exception {
        
        ArrayList<String> arrayList = new ArrayList<>();
        arrayList.add("add element 1");
        arrayList.add("add element 2");
        arrayList.add("aa");

        ArrayList<String> arrayList2 = new ArrayList<>();
        arrayList2.add("22-aaa");
        arrayList2.add("22-bbb");
        
        // 把另一个容器所有对象都加进来
        arrayList.addAll(arrayList2);
        
        // 查看是否存在
        arrayList.contains("aa");

        // 获取指定位置元素
        arrayList.get(0);

        // 获取元素所处位置
        arrayList.indexOf("add element 2");

        // 删除
        arrayList.remove("aa");

        // 更新
        arrayList.set(0, "set");

        // 长度
        arrayList.size();

        // 生成数组
        Object[] strings = arrayList.toArray();

        // 清空
        arrayList.clear();
    }
    
}

```