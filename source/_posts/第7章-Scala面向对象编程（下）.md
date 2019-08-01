---
title: 第7章 Scala面向对象编程（下）
date: 2018-08-05 21:58:06
tags: Scala开发快速入门
---
## 7.1 trait简介
- 一个类只能有一个父类但可以混入多个trait

## 7.2 trait使用
- trait中可以有抽象成员、成员方法、具体成员和具体方法
- 构造函数执行顺序：
  + 如果由超类，先调用超类的函数
  + 如果混入的trait有父trait，按照继承层次先调用父trait的构造函数
  + 如果有多个父trait，按顺序从左到右执行
  + 所有父类构造函数和父trait被构造完之后，才会构造本类的构造函数


```scala211
/**
  * 只有抽象方法的trait
  */
object Example7_01 extends App {
    case class Person(var id: Int, var name: String, var age: Int)

    trait PersonDAO{
        def add(p:Person)
        def update(p:Person)
        def delete(id:Int)
        def findById(id:Int):Person
    }

    class PersonDAOImpl extends PersonDAO{
        override def add(p:Person): Unit = {
            println("Invoking add Method ... adding " + p)
        }
        override def update(p: Person): Unit = {
            println("Invoking update Method, updating " + p)
        }
        override def findById(id: Int): Person = {
            println("Invoking findById Method, id = " + id)
            Person(1, "John", 18)
        }
        override def delete(id:Int): Unit = {
            println("Invoking delete Method, id = " + id)
        }
    }

    val p:PersonDAO = new PersonDAOImpl
    p.add(Person(1, "John", 18))
}
```


```scala211
/**
  * trait构造顺序
  */
object Example7_02 extends App {
    import java.io.PrintWriter
    trait Logger {
        println("Logger")
        def log(msg: String): Unit
    }

    trait FileLogger extends Logger{
        println("FileLogger")
        val fileOutput = new PrintWriter("file.log")
        fileOutput.println("#")

        def log(msg: String):Unit={
            fileOutput.print(msg)
            fileOutput.flush()
        }
    }

    new FileLogger {}.log("trait")
}
```


```scala211
/**
  * trait构造顺序02
  */
object Example7_03 extends App {
    trait Logger{
        println("Logger")
    }

    trait FileLogger extends Logger{
        println("FileLogger")
    }

    trait Closable{
        println("Closable")
    }

    class Person{
        println("Constructing Person...")
    }
    class Student extends Person with FileLogger with Closable{
        println("Constructing Student...")
    }

    new Student
}
```


```scala211
/**
  * 在常规构造之前将变量初始化
  */
object Example7_04 extends App {
    import java.io.PrintWriter

    trait Logger {
        def log(msg: String):Unit
    }
    trait FileLogger extends Logger {
        val filename: String
        val fileOutput = new PrintWriter(filename:String)
        fileOutput.println("#")

        def log(msg:String):Unit = {
            fileOutput.print(msg)
            fileOutput.flush()
        }
    }

    class Person

    class Student extends Person with FileLogger {
        val filename = "file.log"
    }

    val s = new {
        override val filename = "file.log"
    } with Student
    s.log("predifined variable ")
}
```


```scala211
/**
  * lazy val
  */
object Example7_05 extends App{
    import java.io.PrintWriter

    trait Logger {
        def log(msg:String):Unit
    }

    trait FileLogger extends Logger {
        val filename:String
        lazy val fileOutput = new PrintWriter(filename: String)

        def log(msg:String):Unit = {
            fileOutput.print(msg)
            fileOutput.flush()
        }
    }
    class Person
    class Student extends Person with FileLogger {
        val filename = "file.log"
    }

    val s = new Student
    s.log("#")
    s.log("lazy demo")
}
```

## 7.3 trait与类
- 相同点
  + 更接近抽象类
  + 定义trait可以extends继承类
- 不同点
  + trait不能有主构造函数定义的成员变量
  + 类不能继承多个类，但可以混入多个trait

## 7.4 多重继承问题
- 为解决__菱形继承__冲突，Scala会对类进行__线性化__，_最右深度优先遍历算法_

## 7.5 自身类型
- 将大类分成若干特质


```scala211
trait B
trait A {this:B => }
class C extends A with B
```
