---
title: 第3章 Iterable
date: 2018-08-05 22:09:10
tags: Scala集合技术手册
---
+ foreach  被动迭代
+ next    主动迭代

#### Iterable 对象分组
+ grouped 元素数量
+ groupBy 计算的键值

#### 滑动窗口（大小固定）分组
+ sliding(size, step //默认为1)

#### zip   生成Tuple2   以较短的集合为准   无次序的集合每次运行可能生成不一样的结果

#### zipAll 以较长的集合为准

#### zipWithIndex
+ 索引值作为Tuple2的第二个值
+ .view 延迟执行

#### 检查两个Iterables包含相同元素  sameElements

#### takeRight  两个指针遍历

#### dropRight  