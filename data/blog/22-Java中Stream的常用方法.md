---
title: 'Java中Stream的常用方法' # 标题
date: '2023/3/28' # 发布时间
# lastmod: '2022/3/10'
tags: [Java] # 标签
draft: false
summary: 'Java中Stream的常用方法'
images: # 文章封面 不要就留个''空字符串
  [
    'https://cn.bing.com/images/search?view=detailV2&ccid=%2ft5hyNwN&id=533828F2F7239AEBA3C74904F613D62307670672&thid=OIP._t5hyNwNH3cI1_m049rLcwHaD5&mediaurl=https%3a%2f%2fwallpapercave.com%2fwp%2fwp6933338.jpg&exph=1076&expw=2048&q=Java&simid=607990537215691976&FORM=IRPRST&ck=CE42F4C49C0351B4852F4FB34D4580FF&selectedIndex=3',
  ]
layout: PostLayout
---

`Java` 中的 `Stream` 是一种用于处理集合的 API。它提供了一种高效且易于使用的方式来对集合进行过滤、映射、排序、约简等操作。使用 Stream API 可以使代码更加简洁、易于维护和可读性更高

这里列举了一些 `Stream` 常用的方法

## map

```java
List<String> words = Arrays.asList("hello", "world", "java", "stream");

// 使用map方法将每个字符串转换为大写
List<String> upperCaseWords = words.stream()
                                   .map(String::toUpperCase)
                                   .collect(Collectors.toList());
// 输出结果：[HELLO, WORLD, JAVA, STREAM]
```

## filter

```java
List<String> words = Arrays.asList("hello", "world", "java", "stream");

// 使用filter方法过滤出长度大于4的字符串
List<String> longWords = words.stream()
                              .filter(s -> s.length() > 4)
                              .collect(Collectors.toList());
// 输出结果：[world, stream]
```

## reduce

```java
List<String> words = Arrays.asList("hello", "world", "java", "stream");

// 使用reduce方法计算所有字符串的长度之和
int totalLength = words.stream()
                       .mapToInt(String::length)
                       .reduce(0, Integer::sum);
// 输出结果：20

// reduce方法还可以用于查找集合中的最大值、最小值等聚合操作。以下是一个使用reduce方法查找集合中最大值的例子
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

int max = numbers.stream()
                 .reduce(Integer.MIN_VALUE, Integer::max);

System.out.println(max); // 输出结果：5

```

## sorted

```java
List<String> words = Arrays.asList("hello", "world", "java", "stream");

// 使用sorted方法按字母顺序排序字符串
List<String> sortedWords = words.stream()
                                .sorted()
                                .collect(Collectors.toList());
// 输出结果：[hello, java, stream, world]
```

## collect

将流中的元素收集到列表中：

```java
List<T> list = stream.collect(Collectors.toList());
```

将流中的元素收集到集合中：

```java
Set<T> set = stream.collect(Collectors.toSet());
```

将流中的元素收集到映射中：

```java
Map<K, V> map = stream.collect(Collectors.toMap(keyMapper, valueMapper));
```

例子：将 List 转为 Map

- 的第三个参数`(key1, key2) -> key2`是一个合并函数，用于解决在将流中的元素收集到映射中时，如果存在相同的键，如何处理值的问题。在这种情况下，它表示如果存在相同的键，则保留后一个值

```java
Map<String, CourseCategoryTreeDto> mapTemp = courseCategoryTreeDtos.stream()
.filter(item -> !id.equals(item.getId()))
.collect(Collectors.toMap(key -> key.getId(), value -> value, (key1, key2) -> key2));
```
