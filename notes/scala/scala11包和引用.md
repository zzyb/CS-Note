# 包和引用

## 包

Scala的代码采用了Java平台完整的包机制。

```SCALA
package com.yber.spark
class HelloWorld
//代码将HelloWorld放入com.yber.spark包中。
```

- 遵循反域名习惯设置Scala包名。

- 一种想C#的命名空间方式：

- ```scala
  package yber{
    package spark{
      class HelloWorld//在yber.spark包中
      package test{
        class Test//在yber.spark.test包中
      }
    }
  }
  ```

- Scala的嵌套包

- ```SCALA
  package com {
    package yber{
      class Hello
    }
    
    package spark{
      class Hi{
        val hello = new yber.Hello
        //这里不用写 com.ybe.Hello 
      }
    }
  }
  ```

- 内部作用域的包可以隐匿被定义在外部作用域的同名包。

- ```scala
  //文件1：
  package launch{
    class Booster3
  }
  //文件2：
  package com{
    package yber{
      package launch{
        class Booster1
      }
      class MissionControl {
        val booster1 = new launch.Booster1
        val booster2 = new com.launch.Booster2
        val booster3 = new _root_.launch.Booster3//!!!
        //Scala在所有用户所创建的包之外，提供了名为_root_的包。
        //也就是说，任何你能写出来的顶级包都被当做_root_包的成员。
      }
    }
    package launch{
      class Booster2
    }
  }
  ```

## 引用

Scala使用import子句来引用。

```scala
package yber

abstract class Fruit{
  val name:String,
  val color:String
}
Object Fruits{
  object Apple extends Fruit("apple","red")
  object Orange extends Fruit("orange","orange")
  object Pear extends Fruit("pear","yellow")
  val menu = List(Apple,Orange,Pear)
}
```

### Import

```SCALA
import yber.Fruit
//1、易于访问Fruit
import yber._
//2、易于访问yber的所有成员
import yber.Fruits_
//3、易于访问Fruits的所有成员
```

- 第一个，单类型引用

- 第二个，按需引用，与Java的区别是Java使用`*`而Scala使用`_`

- 第三个，与Java的静态类字段引用一致。

- ```SCALA
  def showFruit(fruit:Fruit){
    import fruit._
    println(name+"'s are"+color)
    //注意这里，可以直接使用name、color，等价于fruit.name、fruit.color
  }
  //当把对象当做模块使用的时候，这用语法尤其有用。
  ```



### Scala的灵活引用

- 可以出现在任何地方。
- 可以指的是（单例或正统的）对象及包。
- 可以重命名或隐藏一些被引用的成员。