---
title: 第2章 Traversable
date: 2018-08-05 22:07:04
tags: Scala集合技术手册
---

+ Traversable
  - immutable.Traversable
  - mutable.Traversable
  
  
> 单线程  非线程安全

#### 初始化Traversable对象
+ 不能直接实例化trait，通过具体实现类的伴生对象： .toTraversable
+ 直接赋值给Traversable类型的变量
+ 直接得到Traversable的实例
  - Traversable的伴生对象的apply


```scala211
val s = Traversable(1,2,3)
```




#### 集合的静态类型和类型擦除
+ Scala的类型参数信息在编译时被擦除
+ 通过scala.reflect.runtime.universe.typeTag保存编译时的类型信息

#### foreach
+ higher-order function 高阶函数
  - 一个或多个函数作为参数
  - 返回一个函数
+ foreach不返回值
+ 语法糖
  - 零参数的方法省略括号
  - 中缀表示法
  - 后缀表示法


```scala211
val s = Traversable(1,2,3)
s.foreach(x=>x*x)
s.foreach(x=>println(x*x))
```




#### flatten

###### 所包含的元素类型不一致仍可平展


```scala211
val s3 = Traversable(Traversable(1,2,3),Traversable(2L,3L,4L),Traversable(5:Short,6:Short,7:Short))
val s4 = s3.flatten
```




###### 普通类型、集合类型不可平展

###### 只作浅层平展


```scala211
val s5 = Traversable( Traversable( 1, 2, 3), Traversable( 2, 3, 4), Traversable( Traversable( 5, 6, 7), Traversable( 8, 9, 10)))
val s6 = s5. flatten
```




###### 可将Traversable[Option[A]]平展为Traversable[A]

#### transpose
+ 元素长度不一致抛出异常

#### unzip、unzip3
+ 接受隐式参数asPair

#### ++
+ mangled技术，将方法、类编码为$name<br>  `=, $eq`
+ ++ 类型与左边一致
+ ++: 类型与右边一致

#### mutable.Traversable.concat

#### PartialFunction
+ collect
+ collectFirst

#### map、flatMap

#### scan 计算阶乘   返回中间结果


```scala211
// scan is scanLeft
val t = Traversable(1 to 5 :_*)
val result1 = t.scan(1)(_*_)
val result2 = t.scanRight(1)(_*_)
```




#### fold  返回值
+ foldLeft   /:
+ foldRight   :\

#### 判断集合非空  isEmpty   nonEmpty   size   hasDefiniteSize
+ isEmpty 不推荐
+ size   不推荐
+ nonEmpty 推荐
+ hasDefiniteSize
  - 对strict集合：true
  - 对Stream：<br>到终点    true<br>未到终点   false

#### 得到特定元素
+ head、last    空集合抛出异常
+ headOption、lastOption
+ find

#### 尾部
+ tail
+ tails   生成Iterator
+ init   去除尾部
+ inits

#### 选择一段子集
+ slice[from,until)  左闭右开

#### 选择前N个元素
+ take(n) == slice(0,n)  元素数量小于n，返回整个集合
+ takeWhile

#### 跳过前N个元素
+ drop
  - 小于N，返回空集合
  - N <= 0, 返回整个集合
+ dropWhile

#### 条件筛选
+ filter、filterNot
+ withFilter 返回FilterMonadic
  - Monad 计算序列  管道式

#### 分组
+ splitAt == (c.take(n), c.drop(n))
+ span   == (c.takeWhile(p), c.dropWhile(p))
+ partition == (true, false)
+ groupBy


```scala211
val t = Traversable(1 to 10: _*)
println(t.splitAt(5))
println(t.span(_ < 6))
println(t.partition(_ % 2 == 0))
println(t.groupBy(_ % 3))
```




#### 检查元素  forall、exist

#### 统计满足断言的元素个数
+ count
+ filter  额外生成Traversable对象

#### 规约 reduce  不需要初始值

#### 聚合 sum、product、min、max、aggregate、maxBy、minBy
+ aggregate不要求返回结果的类型是集合元素类型的父类

#### 基于Traversable对象生成字符串
+ mkString  利用addString实现
+ addString 分隔符
+ stringPrefix 返回对象的实际类型的名称  总是返回"List"


```scala211
val t = Traversable(1 to 5: _*)
println(t.mkString("^",",","&"))
// addString
```




#### 类型转换
+ toArray
+ toParArray
+ toBuffer
+ toIndexedSeq
+ toIterable  返回scala.collection.Iterable  toSeq == toStream
+ toIterator  返回scala.collection.Iterator  实际调用toStream.iterator
+ toList
+ toSeq
+ toSet
+ toStream
+ toVector
+ toTraversable  返回原集合
+ toMap  要求Tuple2
+ to

#### 复制元素到一个数组  copyToArray  copyToBuffer

#### 返回一个Traversable对象的视图
+ view[from:until)  返回non-strict,调用force返回strict
+ slice 立即产生新的集合

#### 得到Traversable对象的底层实现 repr

#### 使用一个相同的元素填充


```scala211
val t = Traversable.fill(5,2)("A")
val t1 = Traversable.iterate(1, 5)(_ * 10)
```




#### range(start, end, step)

#### tabulate


```scala211
val t = Traversable.tabulate(5,2)(_ * 10 + _)
```




#### 生成空Traversable对象
+ Traversable.empty[Int]
+ Traversable`[Int]()`
+ Nil

#### 串行seq    并行par