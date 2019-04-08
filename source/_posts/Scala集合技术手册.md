---
title: Scala集合技术手册
date: 2018-12-01 23:45:04
tags: Scala Collection
---

## 第2章 Traversable

+ Traversable
  - immutable.Traversable
  - mutable.Traversable
      - 单线程  非线程安全

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

## 第3章 Iterable
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

## 第4章 Seq
+ 一定顺序  可迭代  可重复
+ 每个元素对应一个索引值
+ 偏函数
+ IndexedSeq 快速随机访问
+ LinearSeq  快速访问head、tail

#### indices
+ 0 until length

#### size、length、lengthCompare
+ size  Traversable的方法
+ length  Seq的方法
+ .lengthCompare(len)
  - 序列长度等于len，返回0
  - 序列长度小于len，返回seq.length - len
  - 序列长度大于len，返回1
  - immutable.List长度等于len，总是返回-1

#### Seq(index)  or  Seq.apply(index)

#### indexOf   lastIndexOf

#### indexWhere  lastIndexWhere
+ indexWhere 返回索引值
+ find     返回元素

#### indexOfSlice  lastIndexOfSlice  搜索集合，返回索引值

#### segmentLength  prefixLength   返回长度值

#### 增加元素
+ +:   序列头部
+ :+   序列尾部
+ padTo(len, elem)


```scala211
val t = Seq(1 to 5: _*)
println(t :+ 3)
println(3 +: t)
println(t.padTo(8,6))
```

#### 替换元素 patch(from 要替换的位置, that 要替换后的元素, replaced 要替换的元素数量)

#### 更新元素  update
+ immutable.Seq.updated  生成新序列
+ mutable.Seq.update  原地更改

#### 排序
+ sorted  math.Ordering  将元素复制到数组，调用java.util.Arrays.sort,将数组复制到新集合
+ sortBy
+ sortWith

#### 反转
+ reverse 对于List，通过逐个获取元素重新生成一个列表
+ reverseIterator  返回反转的迭代器
+ reverseMap  比seq.reverse.map更有效，在一次遍历中实现反转和映射

#### 序列是否包含某个前缀或后缀
- startsWith
- endsWith

#### 序列是否包含某子序列
- contains  元素
- containsSlice  子序列

#### 检查两个序列对应的元素是否满足断言
- corresponds

#### 集合操作
- intersect
- union
- diff
  + Seq.fill(times)(elems)

#### 去掉重复的元素
- distinct

#### 得到元素的各种排列
- permutations

#### 得到序列的指定长度的元素的组合
- combinations(length)
  + length == Seq.size: 返回只含一个集合的迭代器
  + length < 0 or length > Seq.size: 返回空的迭代器
  + length == 0: 返回只含空集合的迭代器

#### 将序列进行转换
- mutable.Seq.transform
  + transform 原地转换， map返回新的序列
  + transform后的元素类型和原来一致， map有可能不同

#### 偏函数的应用
- addThen
- applyOrElse
- orElse
- lift  将偏函数转换成一个普通函数，也就是返回一个Option[A]类型的函数
- runWith


```scala211
val s1 = Seq(1 to 5: _*)
val s2 = Seq(6 to 10: _*)
val s3: PartialFunction[Int, String] = {
    case x if x > 0 => "hello" + x
}

val pf = s1.asInstanceOf[PartialFunction[Int, Int]]
```


```scala211
// andThen 先应用序列
println(s1.andThen(s2)(2))  // s2(s1(2))
println(s1.andThen(s3)(2))  // s3(s1(2))
// compose 先调用函数
println(s1.compose((x: String) => x.length - 1)("hello"))

println(s1.applyOrElse(2, (_: Int) => 2))
println(s1.applyOrElse(10, (_: Int) => Int.MaxValue))

println(s1.orElse(s2)(2))
println(s1.orElse(s3)(8))

println(s1.lift(2))
println(s1.lift(8))

println(s1.runWith(println)(2))
println(s1.runWith(println)(8))
```

#### IndexedSeq和LinearSeq
- IndexedSeq  索引序列
  + immutable.Vector
  + immutable.NumericRange
  + immutable.Range
  + mutable.ArraySeq
  + mutable.StringBuilder
  + mutable.ArrayBuffer
  + 可以隐式转换成immutable.IndexedSeq[Char]的String类型
  + 可以隐式转换成mutable.IndexedSeq的数组类型Array
- LinearSeq  线性序列
  + immutable.List
  + immutable.Stream
  + immutable.Queue
  + immutable.Stack
  + mutable.MutableList
  + mutable.Queue

#### Range和NumericRange
- Range  等差整数队列
- NumericRange  可以生成Int、BigInt、Short、Byte、Char、Long、Float、BigDecimal、Double类型的序列

#### Vector
- 每个非叶节点有32个子节点
- Vector的updated方法会生成一个新的Vector对象

## 第5章 Set 

#### 检查Set集合是否包含元素
- contains
- apply
- subsetOf  检查一个集合是否包含某个集合的子集合

#### 增加一个元素或者一组元素到Set集合中
- +
  + 向集合中增加一个元素，并返回新的集合，此时原集合不变
  + 可以增加1或多个元素
- ++
  + 将一个集合中所有的元素都加入到Set集合中
  + 参数类型并不要求必须是Set类型
- ++:
  + 将左边的集合添加到右边的Set集合
- mutable.Set.+=
  + 原地添加
  + __(s += 4) eq s返回true__
- mutable.Set.++=
  + 原地添加
- mutable.Set.add
  + 原集合中包含这个元素，则返回false，否则返回true

#### 从Set集合中去掉一个元素或一组元素
- -
  + 一次可以移除一个或者多个元素
  + 去掉一个集合中不存在的元素时不会报错，而且返回结果还是原来的集合
- `--`
  + 从原集合中去掉一个指定的集合所包含的元素，这组元素不一定是Set集合
- mutable.Set.-=
  + 原地删除
- mutable.Set.--=
  + 原地删除
- mutable.Set.remove
  + 这个元素存在于原集合，则返回true，否则返回false
- mutable.Set.retain
  + 移除集合中所有不满足断言的元素
- mutable.Set.clear
  + 清除集合中的所有元素

#### 二元Set集合运算
- &
- intersect
- |
- union
- &~
- diff
  + 差集操作符的左右集合是有顺序的

#### 更新一个可变Set集合的元素
- mutable.Set.update

#### 克隆可变Set集合
- mutable.Set.clone

#### SortedSet
- collection.SortedSet
  + firstKey
  + lastKey
  + keySet
  + 默认实现结果是TreeSet
- collection.BitSet
  + 更有效的计算交、并、差集的方法，它可以使用位操作来实现
- immutable.SortedSet
- immutable.BitSet
- immutable.TreeSet
- mutable.SortedSet
  + 元素的增加、删除、更改后的结果都是按照顺序排列的
- mutable.BitSet
  + 原地位操作的方法：&=、|=、&~、^=
- mutable.TreeSet

#### HashSet
- immutable.HashSet是以Hash Trie树的方式实现的
- Trie，又称前缀树或字典树，是一种有序树
- Vector的底层实现，它也是以分支因子为32的Trie树实现的
- 子类
  + EmptyHashSet
  + HashSet1
  + HashTrieSet
  + HashSetCollision1
    - 只有在哈希冲突的时候才会生成
    - 以链表的方式（ListSet）存放哈希值重复的对象
- 树的每一个节点最多只能容纳32个元素或者子节点

#### ListSet
- 通过ListSetBuilder生成集合，它主要包含两个数据结构：ListBuffer和HashSet
- 总是以最后加入的元素作为头指针head

#### LinkedHashSet
- 以hashtable数据结构存储数据，同时元素的顺序保持插入时的顺序，它是可变的Set集合

## 第6章 Map

#### 根据键值查找值
- apply
- get
- getOrElse
- withDefault
- withDefaultValue
- getOrElseUpdate

#### 包含
- contains
- isDefinedAt

#### 增加新的键值对
- +
  + 增加一个或者多个键值对，返回新的Map对象
- ++
  + 增加一个键值对的集合
- ++:
- 可变Map原地添加
  + +=
  + ++=
  + put

#### 删除键
- -
  + 如果传入的参数为null，则不会报错，但是没有意义
- `--`
- -=
- --=
- remove
- clear

#### 根据键更新它的值
- updated
  + 返回新的Map对象
- mutable.Map.update
- mutable.Map.put

#### 得到键的集合
- keys
  + 每次调用都会返回一个新的ImmutableDefault-KeySet对象
- keySet
  + 每次调用都会返回一个新的ImmutableDefault-KeySet对象
- keysIterator

#### 得到值的集合
- values
- valuesIterator

#### 转换函数
- filterKeys 对键进行筛选
- mapValues 对值进行映射，返回新的key-mapped(value)键值对集合
- transforms
  + 为键生成新的值
  + 与mapValues不同的是，键和值都会传给函数参数，参与到新的值的计算中
  + 可变Map的transform方法要求所生成的值的类型要与原始Map的值的类型相同，这是因为它所执行的原地映射，不会生成新的Map
  + 不可变Map的transform方法所生成的值的类型可以和原始Map的值的类型不同，因为它所返回的是新的Map集合

#### 偏函数
- andThen
- applyOrElse
- lift
- orElse
- runWith

#### 克隆
- clone
  + 可变Map
  + 生成一个与原Map相同的Map对象
  + 一旦克隆完成，对原始Map对象的更改将不会影响新生成的Map对象

## 第7章 数组

#### 数组的长度
- length 优先使用length方法
- size 调用数组的size方法会产生隐式转换

#### 更新数组
- t.update(1,200)可以简写为t(1)=200

#### 连接两个数组
- concat
  + 连接两个包含相同类型的数组
  + 通过ArrayBuilder将两个数组中的元素复制到一个新的数组中

#### 填充数组
- fill
```
val t = Array.fill(3,2)(10)
// deep方法可以将多维数组转化成一个嵌套的IndexedSeq对象
println(t.deep.toString())
```

#### 增加元素
- :+  将元素放在新数组的最后
- +:  将元素放在新数组的第一个位置

## 第8章 字符串

#### 字符串比较
- ==  值
- eq  对象

#### 按照换行符分割字符串
- linesWithSeparators

#### 正则表达式
```
val regex = """(\d\d)-(\d\d)-(\d\d\d\d)""".r("month", "day", "year")
```

#### 分割字符串
- split
- strip
  + stripLineEnd 用来去除尾部换行符
  + stripPrefix 如果字符串以指定的前缀开头，则返回除去前缀后的字符串
  + stripSuffix 如果字符串以指定的后缀结尾，则返回除去后缀后的字符串
  + stripMargin 如果多行字符串中的行以空格或者其他控制符作前缀，然后以分隔符“|”与其他字符分隔，则这个方法会删除这些空格和其他控制符，以及这个分隔符“|”。每一行的换行符还会保留，只是删除分隔符“|”前面的前导字符串

#### String Interpolation
- s
- f
  + 类型安全，如果格式和变量的类型不匹配，则编译的时候会出错
  + 没有“%”定义格式，应该按照s进行处理
- raw

## 第9章 缓冲器

#### 增加元素
- 返回的是新的Buffer
    - :+ 增加到尾部
    - +: 增加到头部
    - ++ 增加集合到尾部
    - ++: 增加集合到头部
- 对原始Buffer的原地操作
    - +=
    - +=:
    - ++=
    - ++=:
- 原地操作符等价的方法
    - clear
    - 尾部
        - append
        - appendAll
    - 头部
        - prepend
        - prependAll
    - 指定位置
        - insert
        - insertAll

#### 移除元素
- 如果Buffer中有重复元素，则移除第一个匹配的元素

#### Trim、clear和clone
- trimStart
- trimEnd
- clone 返回一个新的Buffer

## 第10章 列表

#### 初始化
- :: 调用的是右边的对象的方法
- ::: 操作符左边的列表增加到右边的列表的头部
- 后进先出（last-in-first-out，LIFO），类似栈的访问模式

#### 模式匹配
```
l match { 
  case ::(firstValue,restValues) => println(firstValue) 
  case List() => println("empty") 
}
```

#### MutableList
- 在Scala内部用来实现队列Queue
- 没有定义移除元素的操作

## 第11章 栈和队列

#### Stack
- 先入后出（FILO）的数据结构
- 不可变， 用List代替
  + 入栈  x :: list
  + 出栈  list.head和list.tail
- mutable.Stack
  + top 返回栈顶的元素，但是不会把这个元素从栈顶移除
  + push 将元素压入栈顶
  + pushAll 将集合中的所有元素都压入栈
  + pop 将栈顶的元素移除，并返回栈顶的元素
  + elems 返回当前的底层数据结构List对象
  + update
    + stack(n) = A
  + clear
- mutable.ArrayStack
  + top、push、pop
    + 没有pushAll方法
    + 没有压入多个元素的push重载方法
  + dup 复制栈顶的元素（duplicate的缩写），然后将这个复制的元素再压入栈
  + drain 逐个移除栈顶的元素，直到为空，移除的元素会传递给一个函数去处理
  + preserving 在表达式执行完毕后恢复这个栈，使这个栈的内容和调用前的一致

#### Queue
- 先入先出（FIFO）的数据结构
- immutable.Queue
  + 出队和入队都会返回新的队列
  + enqueue
  + dequeue
  + front
- mutable.Queue
  + dequeueAll 移除所有满足断言的元素
  + dequeueFirst 移除第一个满足断言的元素，返回一个Option对象
- PriorityQueue
  + 内部以堆（heap）的数据结构实现
  + dequeueAll 并不需要断言作参数，它直接按照优先级将所有的元素移除队列
  + 没有front方法，而是使用head方法实现

## 第12章 流

#### 初始化
- #::
- #::: 连接合并两个流
- head 流的第一个元素
- tail 除去流的第一个元素的其他元素，__在调用时可能会需要即时计算__
