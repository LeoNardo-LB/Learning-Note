# 05 - Map

### Map概述：

Map可以联想到地图，一个地图上的点位对应着一个现实中的地方。如果把地图上的点位看作key，现实中的地方看作Value，这就是Java中的Map了。这种一一对应的关系，就叫做映射。Java提供了专门的集合类用来存放这种对象关系的对象，即` java.util.Map<K,V> `接口。

通过查看 Map 接口描述，发现`Map<K,V> `接口下的集合与 `Collection<E>` 接口下的集合，它们存储数据的形式不同。

-   Collection 中的集合，元素是孤立存在的（理解为单身），向集合中存储元素采用一个个元素的
    方式存储。
-   Map 中的集合，元素是成对存在的(理解为夫妻)。每个元素由键与值两部分组成，通过键可以找对所对应的值。
-   Collection 中的集合称为单列集合， Map 中的集合称为双列集合。

>   需要注意的是， Map 中的集合不能包含重复的键，值可以重复；每个键只能对应一个值（这个值可以是单个值，也可以是个数组或集合值）。

![image-20200912183617544](_images/image-20200912183617544.png)

### Map常用方法

#### 添加操作

| 方法名 | 描述 |
| ------ | ---- |
|V put(K key,V value)|存一个键值对|
|void putAll(Map<? extends K,? extends V> m)|存一个Map接口的键值对|

#### 删除
| 方法名 | 描述 |
| ------ | ---- |
|void clear()|清除所有键值对|
|V remove(Object key)|移除一个键对应的键值对|

#### 元素查询的操作
| 方法名 | 描述 |
| ------ | ---- |
|V get(Object key)|通过一个键（key）获取值（value）|
|boolean containsKey(Object key)|是否包含某key|
|boolean containsValue(Object value)|是否包含某value|
|boolean isEmpty()|判断map是否为空|

#### 元视图操作的方法：

| 方法名 | 描述 |
| ------ | ---- |
|Set keySet()|返回key的集合（Set形式）|
|Collection values()|返回value的集合（Collection形式）|
|Set<Map.Entry<K,V>> entrySet()|返回所有键值对（Set形式）|

#### 其他方法

| 方法名 | 描述 |
| ------ | ---- |
|int size()|获取Map键值对数量|

##### 代码演示

```java
public class MapDemo {
    public static void main(String[] args) {
        //创建 map对象
        HashMap<String, String> map = new HashMap<String, String>();
        //添加元素到集合
        map.put("黄晓明","杨颖");
        map.put("文章","马伊琍");
        map.put("邓超","孙俪");
        System.out.println(map);
        //String remove(String key)
        System.out.println(map.remove("邓超"));
        System.out.println(map);
        // 想要查看 黄晓明的媳妇 是谁
        System.out.println(map.get("黄晓明"));
        System.out.println(map.get("邓超"));
    }
}
```

put说明：

使用put方法时，若指定的键(key)在集合中没有，则没有这个键对应的值，返回null，并把指定的键值添加到集合中；

若指定的键(key)在集合中存在，则返回值为集合中键对应的值（该值为替换前的值），并把指定键所对应的值，替换成指定的新值。

### Map集合的遍历

Map的遍历，不能支持foreach，因为Map接口没有继承java.lang.Iterable接口，也没有实现iterator()方法。只能用如下方式遍历：

-   单独遍历

    通过Set keySet()与Collection values()方法获取键的集合与值的集合单独遍历

-   使用EntrySet遍历

    Map.Entry保存 key—value 的映射关系，实际上Map.Entry是Map接口的内部接口。在Map中存储数据，实际上是通过将Key—value的数据存储在Map.Entry接口的实例中，然后再Map集合中插入Map.Entry的实例对象实现的。

    ![image-20200912184703588](_images/image-20200912184703588.png)

```java
@Test
public void test(){
    HashMap<String,String> map = new HashMap<>();
    map.put("许仙","白娘子");
    map.put("董永","七仙女");
    map.put("牛郎","织女");
    map.put("许仙","小青");
    System.out.println("所有的key:");
    Set<String> keySet = map.keySet();
    for (String key : keySet) {
        System.out.println(key);
    }
    System.out.println("所有的value：");
    Collection<String> values = map.values();
    for (String value : values) {
        System.out.println(value);
    }
    System.out.println("所有的映射关系");
    Set<Map.Entry<String,String>> entrySet = map.entrySet();
    for (Map.Entry<String,String> entry : entrySet) {
        // System.out.println(entry);
        System.out.println(entry.getKey()+"->"+entry.getValue());
    }
}
```



## Map的实现类

Map接口的常用实现类：HashMap、TreeMap、LinkedHashMap和Properties。其中**HashMap**是Map 接口使用频率最高的实现类。

