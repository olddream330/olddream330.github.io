---
title: 第6章 Scala面向对象编程（上）
date: 2018-08-05 21:53:53
tags: Scala开发快速入门
---
## 6.1 类与对象
- 类的成员变量在声明时必须初始化（var：null或 _ ），抽象类除外
- 统一访问原则：调用者并不知道是直接对成员变量进行访问，还是通过setter方法进行访问的方式
- main函数必须定义在单例对象中
- 伴生类与伴生对象开放了对彼此成员的访问权限，还可以实现无new创建对象及析构对象(外部不允许访问私有成员)


```scala211
import scala.beans.BeanProperty

class Person {
    // private
    @BeanProperty
    var name: String = _
}
```





```scala211
class Person2 {
    // private final
    val name: String = null
}
```





```scala211
val p = new Person
p.name_= ("john")
p.name = "John"
p.setName("Johnson")
p.getName
```





```scala211
object Example6_01 {
    object Student {
        private var studentNo: Int = 0
        def uniqueStudentNo() = {
            studentNo += 1
            studentNo
        }
    }
    def main(args: Array[String]) {
        println(Student.uniqueStudentNo())
    }
}
```





```scala211
object Example6_01_2 extends App {
    object Student {
        private var studentNo: Int = 0
        def uniqueStudentNo() = {
            studentNo += 1
            studentNo
        }
    }
    println(Student.uniqueStudentNo())
}
```





```scala211
object Example6_02 extends App {
    // Companion class
    class Student {
        private var name: String = null
        def getStudentNo = {
            Student.uniqueStudentNo()
            Student.studentNo
        }
    }
    // Copmpanion object
    object Student {
        // def apply() = new Student()
        private var studentNo: Int = 0
        def uniqueStudentNo() = {
            studentNo += 1
            studentNo
        }
        def printStudentName = println(new Student().name)
    }
    
    println(new Student().getStudentNo)
    println(Student.printStudentName)
}
```




## 6.2 主构造函数


```scala211
// class Person3 private(val name: String, ...
class Person3(val name: String, val age: Int = 32) {
    println("Constructing Person ...")
    override def toString() = name + ":" + age
}
```





```scala211
val p = new Person3("phil")
```




## 6.3 辅助构造函数
- this
- 必须先调用主构造函数this()


```scala211
object Example6_03 extends App {
    class Person {
        private var name: String = null
        private var age: Int = 32
        private var sex: Int = 0
        
        def this(name: String) {
            this()
            this.name = name
        }
        def this(name: String, age: Int) {
            this(name)
            this.age = age
        }
        def this(name: String, age:Int, sex: Int) {
            this(name, age)
            this.sex = sex
        }
        
        override def toString = {
            val sexStr = if(sex == 1) "female" else "male"
            s"name=$name, age=$age, sex=$sexStr"
        }
    }
    println(new Person("John", 19, 1))
    println(new Person("John", 19))
    println(new Person("John"))
}
```





```scala211
object Example6_03_2 extends App {
    class Person private {
        private var name: String = null
        private var age: Int = 32
        private var sex: Int = 0
        
        def this(name: String="", age:Int=16, sex: Int=1) {
            this()
            this.name = name
            this.age = age
            this.sex = sex
        }
        
        override def toString = {
            val sexStr = if(sex == 1) "female" else "male"
            s"name=$name, age=$age, sex=$sexStr"
        }
    }
    println(new Person("John", 19, 1))
    println(new Person("John", 19))
    println(new Person("John"))
}
```




## 6.4 继承与多态
- 子类可以拥有父类的属性和方法，可以在子类中添加新的类成员，可以重写父类的方法
- 多态：
    + 允许不同类的对象对同一消息做出响应，即同一消息可以根据发送对象的不同而采用多种不同的行为方式
    + 动态绑定、延迟绑定，指在执行期间而非编译期间确定所引用对象的实际类型，根据其实际类型调用其相应的方法
    + aka. 子类的引用可以赋给父类
- 成员变量从父类继承不能使用var或val
- 构造函数执行顺序：如果两个类之间存在继承关系，创建子类对象时会先调用父类的构造函数，然后再调用子类的构造函数来完成对象的创建


```scala211
object Example6_04 extends App {
    class Person(var name:String, var age:Int) {
        override def toString: String = "name=" + name + ", age=" + age
    }
    class Student(name:String, age:Int, var studentNo:String) extends Person(name, age) {
        override def toString: String = super.toString + ", studentNo=" + studentNo
    }
    println(new Person("John",18))
    println(new Student("Nancy", 19, "140116"))
}
```




```scala211
/**
  * 构造函数执行顺序
  */
object Exmaple6_05 extends App {
    class Person(var name:String, var age:Int) {
        println("Person's")
    }
    class Student(name:String, age:Int, var studentNo:String) extends Person(name, age) {
        println("Student's")
    }
    new Student("Nancy", 19, "140116")
}
```





```scala211
/**
  * 方法重写
  */
object Exmaple6_06 extends App {
    class Person(var name:String, var age:Int) {
        override def toString = s"Person($name,$age)"
    }
    class Student(name:String, age:Int,var studentNo:String) extends Person(name,age){
        override def toString = s"Student($name,$age,$studentNo)"
    }
    println(new Student("Nancy", 19, "140116"))
}
```


```scala211
/**
  * 动态绑定与多态
  */
object Exmaple6_07 extends App {
    class Person(var name:String, var age:Int) {
        def walk():Unit = println("Walk() method in Person")
        def talkTo(p:Person):Unit = println("talkTo() method in Person")
    }
    class Student(name:String, age:Int) extends Person(name, age) {
        private var studentNo:Int = 0
        override def walk() = println("Walk like an elegant swan")
        override def talkTo(p: Person) = {
            println("talkTo() method in Student")
            println(this.name + " is talking to " + p.name)
        }
    }
    class Teacher(name:String, age:Int) extends Person(name,age) {
        private var teacherNo:Int = 0
        override def walk() = println("Walk like an elegant swan")
        override def talkTo(p: Person) = {
            println("talkTo() method in Teacher")
            println(this.name + " is talking to " + p.name)
        }
    }

    val p1:Person = new Teacher("Albert", 38)
    val p2:Person = new Student("Jack", 18)

    p1.walk()
    p1.talkTo(p2)

    println("-----------------")

    p2.walk()
    p2.talkTo(p1)
}
```

## 6.5 成员访问控制
- Scala没有public关键字
- protected 类、子类可以访问
- private 类访问
  + @BeanProperty 只能作用于非private成员变量
- private[this] 类的私有成员
  + private 伴生类和伴生对象可以访问
  + private[this] 只能伴生类访问
- 主构造函数参数没有val或var，默认为private[this]

## 6.6 抽象类
- Scala的抽象类可以有抽象方法、抽象成员变量
- 子类继承抽象类时，需要在子类中对父类的抽象成员变量进行override(可省略)__初始化__，否则子类也必须声明为抽象类


```scala211
/**
  * 抽象类的使用
  */
object Example6_08 extends App {
    abstract class Person {
        var age: Int = 0
        var name: String
        def walk()
        override def toString = name
    }

    class Student extends Person {
        override var name: String = _
        override def walk(): Unit = println("Walk like a Student")
    }

    val p = new Student
    p.walk()
}
```

## 6.7 内部类与内部对象

## 6.8 匿名类
- 只使用一次


```scala211
/**
  * 匿名类
  */
object Example6_09 extends App {
    abstract class Person(var name: String, var age: Int) {
        def print: Unit
    }

    val p = new Person("John", 18) {
        override def print: Unit = println(s"Person($name,$age)")
    }

    p.print
}
```