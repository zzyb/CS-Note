# spark-RDD

## RDD解释

- **弹性**：数据主要基于内存存储，如果内存不够，磁盘顶上。

1. 出错后可自动重新计算（通过血缘自动容错）
2. 可checkpoint（设置检查点，用于容错），可persist或cache（缓存）
3. 里面的数据是分片的（也叫分区，partition），分片的大小可自由设置和细粒度调整

- **数据集**：就是一个普通的scala的不可变的集合(Array, Map，Set)

- **分布式**：这个集合是分布式的，这个集合RDD被拆分成多个Partition(分区)存储在不同的机器里面。RDD不存储数据，数据存储在各个partition中。

  ​	

  **RDD**只是这些partition的抽象ADT(abstract data type),在这个ADT上面我们定义了非常多的操作，比如flatMap、比如map、filter等等高阶函数。

## RDD特性

![](../png/RDD特性.jpg)

## RDD算子分类

#### Transformation

| Transformation（转换算子且lazy的）          |
| ------------------------------------------- |
| map                                         |
| flatMap                                     |
| filter                                      |
| sample                                      |
| union                                       |
| groupByKey                                  |
| join                                        |
| reduceByKey                                 |
| sortByKey                                   |
| combineBykey（combine的意思是**联合**）     |
| aggregateByKey（aggregate的意思是**计数**） |

#### Action

| Action       |                                        |
| ------------ | -------------------------------------- |
| foreach      |                                        |
| count        |                                        |
| countBykey   |                                        |
| countByValue |                                        |
| collect      |                                        |
| take         |                                        |
| first        |                                        |
| reduce       | 容易混淆，reduce是一个action算子       |
| saveXXX      | saveAsTextFile、saveAsNewAPIHadoopFile |
| top          |                                        |
| takeOrdered  |                                        |

## RDD算子编程实例

### transformation算子

- map算子

为RDD中每一条记录执行一次map中的操作，操作是one-2-one的，最后返回新的RDD。

```scala
object Demo01 {
  def main(args: Array[String]): Unit = {
    Logger.getLogger("org.apache.spark").setLevel(Level.WARN)
    Logger.getLogger("org.apache.hadoop").setLevel(Level.WARN)
    Logger.getLogger("org.spark_project").setLevel(Level.WARN)
    val conf = new SparkConf()
      .setAppName("Demo01").setMaster("local[*]")
    val sc = new SparkContext(conf)
    
    val list = 1 to 9
    val listRDD:RDD[Int] = sc.parallelize(list)
    //给集合中每一个元素乘上2
    val mapRDD = listRDD.map(t=>t*2)
    mapRDD.foreach(println)
    
    sc.stop()
  }
}
```

输出：

```
14
4
6
12
18
16
2
8
10
```

- flatMap算子

为RDD中每一条记录执行一次flatMap中的操作，操作时one-2-many的，最终返回一个新的RDD。

```scala
val list2 = List(
      "zhang yan bo",
      "jiao xu yang",
      "zhang mei mei"
    )
    val list2RDD:RDD[String] = sc.parallelize(list2)
		//对一行字符串进行拆分，转化为多个单词。
    val flatRDD = list2RDD.flatMap(t => t.split("\\s+"))
    flatRDD.foreach(println)
```

结果：

```
jiao
xu
yang
zhang
yan
bo
zhang
mei
mei
```

- filter算子

为RDD中每一条记录执行一次filter中的函数操作，函数返回结果为boolean，最终保留结果为true记录，过滤掉结果为false的记录。

```scala
val list2 = List(
      "zhang yan bo",
      "jiao xu yang",
      "zhang mei mei"
    )
    val list2RDD:RDD[String] = sc.parallelize(list2)
    val flatRDD = list2RDD.flatMap(t => t.split("\\s+"))
    val filterRDD:RDD[String] = flatRDD.filter(t=>{
      t.length>=5
    })
    filterRDD.foreach(println)
```

结果：

```
zhang
zhang
```

- sample算子

将RDD中的所有元素进行采样操作，返回新的RDD，最终得到所有元素的子集。

该算子有三个参数：

| 参数 | 参数类型---含义 |
| -------------- | ------------------------------------- |
| withReplacement：是否有放回抽样 | Boolean---true有放回 |
| fraction：百分比 | Double---（0，1）           |
| seed：随机种子  | Long---有默认值，可以不设置     |

```scala
 val list3 = 1 to 100000
    val list3RDD:RDD[Int] = sc.parallelize[Int](list3)
    val sampleRDD:RDD[Int] = list3RDD.sample(true,0.5)
    println(sampleRDD.count())
    val sampleRDD2:RDD[Int] = list3RDD.sample(false,0.5)
    println(sampleRDD2.count())
```

结果：

```
50478
49703
```

- reducebykey算子

按照key进行reduce操作。

输入：RDD[K,V]

输出：RDD[K,V]

```scala
 val list7 = List(
      "zhang mei mei",
      "wang ming ming",
      "xiao ming",
      "jiao xu yang"
    )
    val list7RDD:RDD[String] = sc.parallelize(list7)
    val resultRDD:RDD[(String,Int)] = list7RDD.flatMap(_.split("\\s+")).map((_,1)).reduceByKey(_+_)
    resultRDD.foreach(println)
```

结果：

```
(mei,2)
(xu,1)
(ming,3)
(wang,1)
(yang,1)
(jiao,1)
(xiao,1)
(zhang,1)
```

[scala – reduceByKey：它在内部如何工作？](http://www.voidcn.com/article/p-rfwyfvgx-bte.html)

- groupbykey算子

groupByKey(**numPartitions**):按照**key进行分组**，**默认有8个线程**去处理，这个操作**要求数据的格式为RDD[K, V]**,处理之后的**结果,RDD[K, Iterable[V]]**。

注意：改变了RDD的类型！！！

```scala
 val list6 = List(
      "zy-fy-2019 王明明 男 20",
      "zy-xf-2019 德华 女 23",
      "zy-xf-2019 马超 男 33",
      "zy-fy-2019 孙磊 女 13",
      "zy-lyq-2019 小红 男 23",
      "zy-lyq-2019 赵英 女 25",
      "zy-lyq-2019 小可 男 27"
    )
		//加载集合元素并通过map转化为（String，String）二元组类型的RDD
    val storeRDD:RDD[(String,String)] = sc.parallelize(list6).map(t=>{
      val store = t.substring(0,t.indexOf(" "))
      val info = t.substring(t.indexOf(" ")+1)
      (store,info)
    })
		//使用groupbykey算子（注意返回的RDD类型）
    val storesRDD:RDD[(String,Iterable[String])] = storeRDD.groupByKey()
		//直接输出
    storesRDD.foreach(println)
		//通过匹配+mkString函数输出
    storesRDD.foreach(
      {
        case (store,infos)=>{
          println(s"$store===>${infos.mkString("[",",","]")}")
        }
      }
    )
```

结果：

```
(zy-xf-2019,CompactBuffer(德华 女 23, 马超 男 33))
(zy-fy-2019,CompactBuffer(王明明 男 20, 孙磊 女 13))
(zy-lyq-2019,CompactBuffer(小红 男 23, 赵英 女 25, 小可 男 27))
//
zy-lyq-2019===>[小红 男 23,赵英 女 25,小可 男 27]
zy-fy-2019===>[王明明 男 20,孙磊 女 13]
zy-xf-2019===>[德华 女 23,马超 男 33]
```

说明：

1、**groupBy**，这里和groupByKey区别，可以**基于的没有K的数据类型进行分组**。底层还是groupByKey。

2、groupByKey的**性能相对较差**，能不用则不用，能取代则一定要代替。性能较差的**原因**主要和reduceByKey进行比较，**reduceByKey在执行过程当中有一个本地预聚合mapSideCombine，而groupByKey没有**，很多情况下，groupByKey能够完成的工作，同样也可以使用reduceByKey.

- union算子

union(otherRDD):就是前面了解到的**联合操作**，把**两个rdd的数据整合**到一起，成为一个**新的RDD**。

​	类似sql中的union，sql中的union有两种：

​	union: 联合去重

​	union all :**联合不去重**

**spark中**的union操作，**相当于union all**。

```scala
//第一个集合（5个单词）
val list4 = List(
      "hello",
      "java",
      "hadoop",
      "spark",
      "python"
    )
//第二个集合（4个单词）
val list5 = List("hello","java","python","hive")
    val list4RDD = sc.parallelize(list4)
    val list5RDD = sc.parallelize(list5)
//使用union算子
    val unionRDD:RDD[String] = list4RDD.union(list5RDD)
//输出union之后的RDD元素数量
    println(unionRDD.count())
//输出union之后RDD中的每一个元素
    unionRDD.foreach(println)
```

结果：

```
9
hadoop
python
hello
java
spark
hello
java
python
hive
```



- join算子

rdd：RDD[K, V]
otherRDD:RDD[K, W]
rdd.join(otherRDD),这个join操作就是sql中的join操作。

**join的rdd的类型必须是k-v
join的结果RDD[K, (V, W)]**
**leftOuterJoin**的结果RDD[K, (V, Option[W])]
**rightOuterJoin**的结果RDD[K, (Option[V], W)]
**fullOuterJoin**的结果RDD[K, (Option[V], Option[W])]

```scala
val map = Map[String,String](
      "张三"->"男 188 23",
      "李四"->"男 178 24",
      "王五"->"男 177 19",
      "赵六"->"女 165 17"
    )
val map2 = Map[String,Int](
      "张三"->99,
      "王五"->89,
      "赵六"->85
    )

    val map1RDD:RDD[(String,String)] = sc.parallelize(map.toList)
    val map2RDD:RDD[(String,Int)] = sc.parallelize(map2.toList)

    val leftjoinRDD:RDD[(String,(String,Option[Int]))] = map1RDD.leftOuterJoin(map2RDD)

    leftjoinRDD.foreach(
      t=>{
        println(s"${t._1}===>(${t._2._1},${t._2._2.getOrElse(0)}])")
      }
    )
    println("************************")
    leftjoinRDD.foreach{
      case(name,(info,scoreOption))=>{
        println(s"${name}===>(${info}---${scoreOption.getOrElse(null)})")
      }
    }
```

输出：

```
赵六===>(女 165 17,85])
张三===>(男 188 23,99])
李四===>(男 178 24,0])
王五===>(男 177 19,89])
************************
张三===>(男 188 23---99)
赵六===>(女 165 17---85)
李四===>(男 178 24---null)
王五===>(男 177 19---89)
```

- sortbyKey算子

按照key进行排序。

源码：

```scala
/**
   * Sort the RDD by key, so that each partition contains a sorted range of the elements. Calling
   * `collect` or `save` on the resulting RDD will return or output an ordered list of records
   * (in the `save` case, they will be written to multiple `part-X` files in the filesystem, in
   * order of the keys).
   */
  // TODO: this currently doesn't work on P other than Tuple2!
  def sortByKey(ascending: Boolean = true, numPartitions: Int = self.partitions.length)
      : RDD[(K, V)] = self.withScope
  {
    val part = new RangePartitioner(numPartitions, self, ascending)
    new ShuffledRDD[K, V, V](self, part)
      .setKeyOrdering(if (ascending) ordering else ordering.reverse)
  }
```

案例：

```scala
val list1 = List(
      "xiao tian tian",
      "heng ha heng heng heng",
      "shu ke shu ke ke"
    )
//拆分每行字符串为多个单词
    val words:RDD[String] = sc.parallelize(list1).flatMap(_.split("\\s+"))
//转化为String，Int二元组形式
    val pairs:RDD[(String,Int)] = words.map((_,1))
//按照key进行聚合
    val rbk:RDD[(String,Int)] = pairs.reduceByKey(_+_)
//通过map操作调换key-value位置（为了按照单词数量排序）
    val temp1:RDD[(Int,String)] = rbk.map(t=>(t._2,t._1))
//调用sortbyKey，参数一：升序，参数二：一个分区。
    val temp2:RDD[(Int,String)] = temp1.sortByKey(true,1)
//再次通过map调换key-value位置。（重新调整二元组格式为String，Int）
    val results = temp2.map(t=>(t._2,t._1)).foreach(println)
```

结果：

```
(xiao,1)
(ha,1)
(shu,2)
(tian,2)
(ke,3)
(heng,4)
```

- combinebyKey算子

1. 模拟groupbyKey

   ```scala
   //定义集合
   val list6 = List(
         "zy-fy-2019 王明明 男 20",
         "zy-xf-2019 德华 女 23",
         "zy-xf-2019 马超 男 33",
         "zy-fy-2019 孙磊 女 13",
         "zy-lyq-2019 小红 男 23",
         "zy-lyq-2019 赵英 女 25",
         "zy-lyq-2019 小可 男 27"
       )
   //加载集合并转化为二元组形式的RDD
   val store2RDD:RDD[(String,String)] = sc.parallelize(list6).map(t=>{
         val store1 = t.substring(0,t.indexOf(" "))
         val info1 = t.substring(t.indexOf(" ")+1)
         (store1,info1)
       })
   //使用combinebyKey算子（三个参数createCombiner、mergeValue、mergeCombiners）
       val stores2RDD:RDD[(String,ArrayBuffer[String])]
       =store2RDD.combineByKey(createCombiner,mergeValue,mergeCombiners)
   //输出方式一
       stores2RDD.foreach{
         case(store1,infos)=>{
           println(s"$store1===>$infos")
         }
       }
       println("**************************")
   //输出方式二
       stores2RDD.foreach{
         t=>{
           println(s"${t._1}===>${t._2.mkString("{","-","}")}")
         }
       }
   
   ------------分割线（下面在main方法外定义combinebyKey的三个参数）---------------
   //定义combinebyKey中的三个参数（函数）
     def createCombiner(info:String):ArrayBuffer[String] = {
       val ab = ArrayBuffer[String]()
       ab.append(info)
       ab
     }
     def mergeValue(ab:ArrayBuffer[String],info:String):ArrayBuffer[String] = {
       ab.append(info)
       ab
     }
     def mergeCombiners(ab1:ArrayBuffer[String],ab2:ArrayBuffer[String]):ArrayBuffer[String] = {
       ab1.++(ab2)
     }
   ```

   输出：

   ```
   zy-fy-2019===>ArrayBuffer(王明明 男 20, 孙磊 女 13)
   zy-xf-2019===>ArrayBuffer(德华 女 23, 马超 男 33)
   zy-lyq-2019===>ArrayBuffer(小红 男 23, 赵英 女 25, 小可 男 27)
   **************************
   zy-xf-2019===>{德华 女 23-马超 男 33}
   zy-fy-2019===>{王明明 男 20-孙磊 女 13}
   zy-lyq-2019===>{小红 男 23-赵英 女 25-小可 男 27}
   ```

   

2. 模拟reducebyKey

```scala
//定义集合元素
val list7 = List(
      "zhang mei mei",
      "wang ming ming",
      "xiao ming",
      "jiao xu yang"
    )
//加载集合元素并转化为（String，Int）二元组形式的RDD
val combineRDD:RDD[(String,Int)] = sc.parallelize(list7).flatMap(_.split("\\s+")).map((_,1))
//使用combinebyKey算子（三个参数createCombiner1,mergeValue1,mergeCombiners1）
val result1 = combineRDD.combineByKey(createCombiner1,mergeValue1,mergeCombiners1)
    result1.foreach(println)

------------分割线（下面在main方法外定义combinebyKey的三个参数）---------------

//定义combinebyKey中的三个参数（函数）
def createCombiner1(value:Int):Int = {
    value
  }
  def mergeValue1(sum:Int,value:Int):Int = {
    sum + value
  }
  def mergeCombiners1(sum:Int,sumi:Int):Int = {
    sum+sumi
  }
```

输出：

```
(wang,1)
(xiao,1)
(zhang,1)
(yang,1)
(jiao,1)
(ming,3)
(mei,2)
(xu,1)
```



- aggregatebyKey算子

1. 模拟groupbyKey

   

2. 模拟reducebyKey



说明：

**combineByKey** **aggregateByKey** 按照key进行combine操作和按照key进行aggregate操作

这二者其实说的都是同一回事，同时aggregateByKey和combineByKey**底层都是通过combineByKeyWithClassTag来进行实现的**，其中combineByKey是combineByKeyWithClassTag的一个简写方式，从本质上说combineByKey和aggregateByKey没有区别，只不过一般建议在不同的场景下使用不同的算子。



### Action算子

- foreach算子

遍历RDD集合中的每一个元素，返回为null。

- count算子

统计RDD中共有多少条记录。

```scala
val list7 = List(
      "zhang mei mei",
      "wang ming ming",
      "xiao ming",
      "jiao xu yang"
    )

		val list7ss = sc.parallelize(list7).flatMap(_.split("\\s+"))

    val num = list7ss.count()

    println(s"全部的单词数量为：${num}")
```

结果：

```
全部的单词数量为：11
```

- countbyKey算子

统计每个key出现的次数。

```scala
val list7 = List(
      "zhang mei mei",
      "wang ming ming",
      "xiao ming",
      "jiao xu yang"
    )

		val list7sRDD:RDD[String] = sc.parallelize(list7).flatMap(_.split("\\s+"))

    val words = list7sRDD.map((_,1))

    val countbykey = words.countByKey()

    countbykey.foreach(println)
    println("--------------------------")
    for((word,count) <- countbykey){
      println(s"$word==>$count")
    }
```

输出：

```
(xiao,1)
(mei,2)
(zhang,1)
(jiao,1)
(ming,3)
(yang,1)
(xu,1)
(wang,1)
------------------------------
xiao==>1
mei==>2
zhang==>1
jiao==>1
ming==>3
yang==>1
xu==>1
wang==>1
```



- countbyValue算子

统计每个value出现的次数。

```scala
val list9 = List("hello","marry","tom","cici","hello","hello","java")

    val list9RDD:RDD[String] = sc.parallelize(list9)

    val countbyvalue = list9RDD.map((_,1)).countByValue()//

    println(countbyvalue)
```

结果：

```
Map((tom,1) -> 1, (cici,1) -> 1, (marry,1) -> 1, (java,1) -> 1, (hello,1) -> 3)
```



- collect算子

收集，将该RDD中**各个分区中的数据收集起来**，存储在driver的内存中成为一个本地集合进行计算。这就很**容易会造成driver由于内存不足造成driver内存溢出的异常**(OutOfMemory OOM),所以在使用该算子的时候要**慎重**，**最好在collect之前执行一下filter**。

- take算子

take(num),获取该集合（RDD）中的前num个元素，**如果该RDD是一个有序的集合**，那么take(N)得到结果是**topN**。

```scala
val list7 = List(
      "zhang mei mei",
      "wang ming ming",
      "xiao ming",
      "jiao xu yang"
    )

		val list7ss = sc.parallelize(list7).flatMap(_.split("\\s+"))

    val take3 = list7ss.take(3)

    take3.foreach(println)
```

结果：

```
zhang
mei
mei
```

- first算子

获取RDD的第一个元素。（等同于take（1））

```scala
		val list7 = List(
      "zhang mei mei",
      "wang ming ming",
      "xiao ming",
      "jiao xu yang"
    )

		val list7ss = sc.parallelize(list7).flatMap(_.split("\\s+"))

    val first = list7ss.first()

    println(first)
```

结果：

```
zhang
```

- reduce算子

对RDD进行聚合操作。（要保证集合类型**不是**Key-value类型）

```scala
		val nums = sc.parallelize(1 to 100)
    val sum = nums.reduce((a,b)=>a+b)
    println(s"sum:$sum")
```

输出：

```
sum:5050
```

- saveXXXX算子

将RDD中的数据落地到本地磁盘或HDFS中。sz