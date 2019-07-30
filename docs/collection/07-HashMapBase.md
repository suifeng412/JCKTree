# HashMap 基本用法


![HashMap 基本用法](https://github.com/suifeng412/JCKTree/blob/master/xmind/collection/07-HashMap基本用法.png)

### demo coding
```java
public class MyTest {

    public static void main(String[] args) throws Exception {
                HashMap<String, String> hashMap = new HashMap<>();
                hashMap.put("Java", "I am Java");
                hashMap.put("PHP", "I am PHP");
                hashMap.put("C++", "I am C++");
                hashMap.put("C#", "I am C#");
                hashMap.put("Go", "I am Go");
        
                hashMap.remove("C#");
                hashMap.remove("C++", "I am C++");
        
                hashMap.size();
                // hashMap.clear();
                hashMap.isEmpty();
                hashMap.get("Java");
                hashMap.containsKey("PHP");
                hashMap.containsValue("I am Go");
        
                for (Map.Entry<String, String> entry : hashMap.entrySet()) {
                    entry.getKey();
                    entry.getValue();
                }
        
                Set<String> keySet = hashMap.keySet();
                keySet.stream().forEach(s -> System.out.println(s));
        
                Collection<String> valueSet = hashMap.values();
                valueSet.stream().forEach(s -> System.out.println(s));
        
                
        
                // 根据 key value 计算 value；不为空就覆盖，或者添加；为空则删除原键值对
                hashMap.compute("Java", (key, value) -> (value + key));
        
                // key 对应的 value 不为空函数才起作用
                // 根据 key 计算出 value，如果 value 不为 null，则赋值或者添加；value 为空则方法失效，不做任何改变
                hashMap.computeIfAbsent("Py", key -> (key + key + "666"));
        
                // key 对应的 value 不为空函数才起作用
                // 根据 key value 计算出新的 value，不为空则赋值，为空就删去原有的键值对
                hashMap.computeIfPresent("Go", (key, value) -> (value + key));
        
                
                // 遍历
                hashMap.forEach((key, value) -> System.out.println(key + " => " + value));      
        
                
                // 得到指定 key 的 value 值，若不存在返回默认值
                hashMap.getOrDefault("noKey", "this is defaultValue");
        
                // 如果 key 在集合中的 value 为空或则键值对不存在，则用参数 value 覆盖或新增
                hashMap.putIfAbsent("GG", "GG");
        
        
        
                // 通过 key 获取 value
                // 若 value 不为空，根据 oldVal 和 newVal 计算覆盖 value
                // 若 value 为空，直接用 newVal 进行添加
                // 若计算结果为 null，则删除原有键值对
                hashMap.merge("Go", "new value", (oldVal, newVal) -> (oldVal + newVal));
        
        
        
                // 替换 key 在集合中的 value。如果 key 不存在不会添加新的键值对。
                hashMap.replace("PHP", "世界上最好的语言");
        
                // key-value 完全相等存在才进行替换，否则不改变
                hashMap.replace("Py", "PyPy666", "替换了PY");
        
                // 对集合中的所有键值对执行计算，并将返回结果作为value覆盖
                hashMap.replaceAll((key, value) -> (value + "--尾巴------>" + key));
        
        
                System.out.println(hashMap);
    }
}
```
