---
title: java中Map遍历的四种方法
date: 2019-07-15 15:55:24
tags:
    - map
categories:
    - java
---

在java中所有的map都实现了Map接口，因此所有的Map（如HashMap, TreeMap, LinkedHashMap, Hashtable等）都可以用以下的方式去遍历。

方法一：在for循环中使用entries实现Map的遍历：
```java
/**
* 最常见也是大多数情况下用的最多的，一般在键值对都需要使用
 */
Map <String,String>map = new HashMap<String,String>();
map.put("熊大", "棕色");
map.put("熊二", "黄色");
for(Map.Entry<String, String> entry : map.entrySet()){
    String mapKey = entry.getKey();
    String mapValue = entry.getValue();
    System.out.println(mapKey+":"+mapValue);
}
```

方法二：在for循环中遍历key或者values，一般适用于只需要map中的key或者value时使用，在性能上比使用entrySet较好；

```java
Map <String,String>map = new HashMap<String,String>();
map.put("熊大", "棕色");
map.put("熊二", "黄色");
//key
for(String key : map.keySet()){
    System.out.println(key);
}
//value
for(String value : map.values()){
    System.out.println(value);
}
```

方法三：通过Iterator遍历；
```java
Iterator<Entry<String, String>> entries = map.entrySet().iterator();
while(entries.hasNext()){
    Entry<String, String> entry = entries.next();
    String key = entry.getKey();
    String value = entry.getValue();
    System.out.println(key+":"+value);
}
```

方法四：通过键找值遍历，这种方式的效率比较低，因为本身从键取值是耗时的操作；

```java
for(String key : map.keySet()){
    String value = map.get(key);
    System.out.println(key+":"+value);
}
```