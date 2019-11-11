# 控制抽象

## 柯里化

**柯里化的函数被用于多个参数列表，而不仅仅是一个。**

普通函数：

```scala
scala> def plainOldSum(x:Int,y:Int) = x+y
//未柯里化的函数
scala> plainOldSum(1,2)
//res0: Int = 3
```

柯里化函数：

```scala
scala> def curriedSum(x:Int)(y:Int) = x+y
//柯里化函数
scala> curriedSum(1)(2)
//res1: Int = 3
```

- 当我们调用这个柯里化的curriedSum函数的时候，实际上接连调用了两个传统函数。

​	第一个：`def firse(x:Int) = (y:Int) => x+y`

调用第一个函数，并传入参数，会产生第二个函数

​	第二个：`val second = first(1)`

first和second函数只是柯里化过程的一个演示。

- 对于柯里化函数我们还可以这么做：

```SCALA
scala> val twoPlus = curriedSum(2)_
//下划线是第二个参数列表的占位符。结果指向第一个函数的参考。
scala> twoPlus(2)
//res2: Int = 4
```



