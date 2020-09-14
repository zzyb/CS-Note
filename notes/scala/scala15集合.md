# 集合类型

## 集合库概览

![集合类型层级](../../png/集合类型的类层级.png)

**Iterable是主要特质。是以下几种的超特质。**

- 可变和不可变的序列（Seq）
- 集（Set）
- 映射（Map）

<u>命名为Iterable是为了说明集合对象可以通过名为elements的方法产生Iterator（枚举器）。</u>

```scala
def elements:Iterator[A]
//A是类型参数，指的是集合包含的元素类型。
```

...



### 序列

**序列继承自特质Seq类，可以处理一组线性分布的数据。**

- ​	元素是有序的，你可请求第一个、第二个、第108个元素，乃至更多。

#### 列表

##### List类（List）

1. 列表支持在表的头部快速添加和删除。
2. 不能提供对任意索引值的快速访问。

```scala
scala> val list = List("tom","jack","lihua")
list: List[String] = List(tom, jack, lihua)

scala> list.head
res0: String = tom

scala> list.tail
res1: List[String] = List(jack, lihua)
```

##### 数组（Array）

1. 保留一组元素序列并可以用基于零（0）的索引高效访问（添加或者获取）处于任意位置的元素。

```scala
scala> val fiveInt = new Array[Int](5)
fiveInt: Array[Int] = Array(0, 0, 0, 0, 0)

scala> val fiveToOne = Array(5,4,3,2,1)
fiveToOne: Array[Int] = Array(5, 4, 3, 2, 1)

scala> fiveInt.size
res2: Int = 5

scala> fiveInt(0) = fiveToOne(0)

scala> println(fiveInt)
[I@52657d5f

scala> println(fiveInt(0))
5

scala> fiveInt
res6: Array[Int] = Array(5, 0, 0, 0, 0)
```

**Scala数组与Java数组表达方式一样。可以无缝的使用现存的返回数组的Java方法。**

##### 列表缓存（ListBuffer）

- List支持的是对列表头部的快速访问。

  如果需要通过向尾部添加对象的方式构造列表。考虑先已对表头前置元素的方式反向构造列表，完成之后再调用reverse使元素反转为你需要的顺序。

- **ListBuffer**

  - ListBuffer是可变对象，包含在collection.mutable中；可以更高效的通过添加元素的方式构造列表。
  - 支持常量时间的添加（O(1)）和前缀操作。
    - 元素添加使用+=操作符。
    - 前缀使用+:操作符。
  - 完成之后，可以调用toList方法获得List。

  ```scala
  scala> import scala.collection.mutable.ListBuffer
  import scala.collection.mutable.ListBuffer
  
  scala> val listbuffer = new ListBuffer[Int]
  listbuffer: scala.collection.mutable.ListBuffer[Int] = ListBuffer()
  
  //向后添加元素
  scala> listbuffer += 1
  res8: listbuffer.type = ListBuffer(1)
  scala> listbuffer += 2
  res9: listbuffer.type = ListBuffer(1, 2)
  //前缀添加元素
  scala> 3 +: listbuffer
  res10: scala.collection.mutable.ListBuffer[Int] = ListBuffer(3, 1, 2)
  //toList转化为List
  scala> res10.toList
  res14: List[Int] = List(3, 1, 2)
  
  //这里不知道为什么没转化出来前缀的3
  scala> listbuffer.toList
  res11: List[Int] = List(1, 2)
  
  scala> res10
  res13: scala.collection.mutable.ListBuffer[Int] = ListBuffer(3, 1, 2)
  ```

##### 数组缓存（ArrayBuffer）

- ArrayBuffer与数组类似。
- 但是，允许你在序列开始或结束的地方添加和删除元素。
  - 新的添加和删除操作平均为常量时间。
  - 偶尔因为申请分配新数组，保留缓存内容而需要线性处理时间。
- 创建ArrayBuffer必须指定类型参数，但是不用指定长度。
  - ArrayBuffer可以自动分配空间。
  - 可以使用+=添加元素。
  - 所有数组能用的方法，数组缓存都能用。

```scala
scala> import scala.collection.mutable.ArrayBuffer
import scala.collection.mutable.ArrayBuffer

scala> val abf = new ArrayBuffer[Int]()
abf: scala.collection.mutable.ArrayBuffer[Int] = ArrayBuffer()

scala> abf.size
res21: Int = 0

scala> abf += 1
res22: abf.type = ArrayBuffer(1)

scala> abf += 2
res23: abf.type = ArrayBuffer(1, 2)

scala> abf(1)
res24: Int = 2
```

##### 队列（Queue）



##### 栈（Stack）



##### 字符串（Seq[Char]）



### 集（Set）和映射（Map）

- Set 和 Map 也有可变和不可变的版本。
- 默认使用的 Set 和 Map 都是不可变版本；如果需要可变版本，需要声明引用。

```scala
//默认的Set
scala> Set(1,2,3)
res26: scala.collection.immutable.Set[Int] = Set(1, 2, 3)

//导入可变Set的包
scala> import scala.collection.mutable.Set
import scala.collection.mutable.Set

//可变的Set
scala> Set(1,2,3,4)
res27: scala.collection.mutable.Set[Int] = Set(1, 2, 3, 4)
```

#### 集（Set）

Set可以使用对象的==操作检查，确保每个对象只在集（Set）中保留最多一个副本。

```scala
//默认不可变的Set
scala> val set = Set(1,2,3)
set: scala.collection.immutable.Set[Int] = Set(1, 2, 3)
//获取大小
scala> set.size
res0: Int = 3
//添加元素（返回新Set）
scala> set + 5
res1: scala.collection.immutable.Set[Int] = Set(1, 2, 3, 5)
//删除元素（返回新Set）
scala> set - 1
res2: scala.collection.immutable.Set[Int] = Set(2, 3)
//原Set没变
scala> set
res3: scala.collection.immutable.Set[Int] = Set(1, 2, 3)
//添加多个元素（）
scala> set ++ List(7,8,9)
res4: scala.collection.immutable.Set[Int] = Set(1, 9, 2, 7, 3, 8)
//删除多个元素（）
scala> set -- List(1,2)
res5: scala.collection.immutable.Set[Int] = Set(3)
//** 不能求交集
scala> set ** Set(1,2)
<console>:13: error: value ** is not a member of scala.collection.immutable.Set[Int]
       set ** Set(1,2)
           ^
// & 用来求Set交集
scala> set & Set(1,2)
res11: scala.collection.immutable.Set[Int] = Set(1, 2)
//判断是否包含，返回boolean
scala> set.contains(3)
res12: Boolean = true
//不可变Set不能使用clear
scala> set.clear
<console>:13: error: value clear is not a member of scala.collection.immutable.Set[Int]
       set.clear
           ^
//导入可变包（可变Set在mutable包内）
scala> import scala.collection.mutable
import scala.collection.mutable

scala> val mset = mutable.Set(1,2,3,4,5)
mset: scala.collection.mutable.Set[Int] = Set(1, 5, 2, 3, 4)
//清空可变Set
scala> mset.clear
//查看清空后的Set
scala> mset
res15: scala.collection.mutable.Set[Int] = Set()
```

#### 映射（Map）

- 创建Map的时候，必须制定两个类型。
  - 第一个类型用来定义映射的键（key）
  - 第二个类型用来定义映射的值（value）
- Map的使用和数组（Array）相近，不过你可以使用不同类型的键，而不仅仅是从0开始的整数做索引。

```scala
//创建 空的 不可变Set（要指定键值类型）
scala> val map = Map.empty[String,Int]
map: scala.collection.immutable.Map[String,Int] = Map()
//添加元素（返回新的map）
scala> map + ("tom" -> 1)
res0: scala.collection.immutable.Map[String,Int] = Map(tom -> 1)

scala> map + ("jack" -> 2)
res1: scala.collection.immutable.Map[String,Int] = Map(jack -> 2)
//删除元素（）
scala> res1 - "jack"
res2: scala.collection.immutable.Map[String,Int] = Map()
//添加多个元素（）
scala> map ++ List("A"->3,"B"->4)
res6: scala.collection.immutable.Map[String,Int] = Map(A -> 3, B -> 4)
//删除多个元素（）
scala> res6 -- List("B","A")
res7: scala.collection.immutable.Map[String,Int] = Map()
//查看Map大小
scala> res6.size
res8: Int = 2
//是否包含某个键，返回boolean
scala> res6.contains("A")
res9: Boolean = true
//返回key的迭代器
scala> res6.keys
res10: Iterable[String] = Set(A, B)
//返回key的Set
scala> res6.keySet
res12: scala.collection.immutable.Set[String] = Set(A, B)
//返回value的迭代器
scala> res6.values
res13: Iterable[Int] = MapLike(3, 4)
//判断是否为空
scala> res6.isEmpty
res15: Boolean = false


---------------------------------------------------------------------------------------------------------------
//导入可变Map的包
scala> import scala.collection.mutable.Map
import scala.collection.mutable.Map
//创建可变的Map（记住格式）
scala> val mmap = Map("tom" -> 100,"jack" -> 101)
mmap: scala.collection.mutable.Map[String,Int] = Map(jack -> 101, tom -> 100)
//添加元素（+=）
scala> mmap += ("lihua" -> 102)
res17: mmap.type = Map(jack -> 101, tom -> 100, lihua -> 102)
//删除元素（-=）
scala> mmap -= ("tom")
res18: mmap.type = Map(jack -> 101, lihua -> 102)
//添加多个元素（++= List(k-v)）
scala> mmap ++= List("A" -> 200,"B" -> 201)
res19: mmap.type = Map(A -> 200, jack -> 101, lihua -> 102, B -> 201)
//删除多个元素（++= List(k)）
scala> mmap --= List("lihua","jack")
res20: mmap.type = Map(A -> 200, B -> 201)
//查看元素（如同数组，通过key）
scala> mmap("A")
res21: Int = 200
//更改元素的value（如同数组，通过key获取，并赋值）
scala> mmap("A") = 300
//查看更改之后
scala> mmap
res23: scala.collection.mutable.Map[String,Int] = Map(A -> 300, B -> 201)

```

#### 默认的集和映射



#### 有序的集和映射



#### 同步的集和映射





### 可变（mutable）集合 VS 不可变（immutable）集合







### 元组