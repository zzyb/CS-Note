## 	RDD编程

### 1.1-RDD基础

**Spark中的RDD就是一个不可变的分布式对象集合。**

​	--每个RDD被分为多个分区，这些分区运行在集群的不同节点上。

​	--RDD可以包含Rython、Java、Scala中任意类型的对象，甚至可以包含用户自定义的对象。

**创建RDD的两种方法：**

​	--读取一个外部数据集；

```scala
lines = sc.textFile("/user/local/a.txt")
```

​	--驱动器程序里分发驱动器程序中的对象集合（比如list、set）

```scala
lines = sc.parallelize(Set(1,2,3,4,5))
```

**RDD支持两种类型的操作：**

​	--转化操作	<u>由一个RDD生成一个新的RDD</u>

​	--行动操作	<u>对RDD计算出一个结果，并把结果返回到驱动器程序中，或把结果存储到外部存储系统（HDFS）中。</u>

**转化操作和行动操作的区别在于Spark计算RDD的方式不同。**

​	任何时候定义新的RDD，Spark只会惰性计算这些RDD。只有第一次行动操作中用到时，才会真正的计算。

```txt
	如果 Spark 在我们运行 lines = sc.textFile(...) 时就把文件中所有的行都读取并存储起来，就会消耗很多存储空间，而我们马上就要筛选掉其中的很多数据。相反，  一旦 Spark 了解了完整的转化操作链之后，它就可以只计算求结果时真正需要的数据。事实上，在行动操作 first() 中，Spark 只需要扫描文件直到找到第一个匹配的行为止，而不需要读取整个文件。
```

**默认情况下，Spark的RDD会在你每次对它们进行行动操作时重新计算。**

​	如果想多个行动操作重用一个RDD，可以使用RDD.persist()把RDD缓存下来。

```txt
	在实际操作中，你会经常用 persist() 来把数据的一部分读取到内存中，并反复查询这部分数据。
	在第一次对持久化的 RDD 计算之后，Spark 会把 RDD 的内容保存到内存中（以分区方式存储到集群中的各机器上） ，这样在之后的行动操作中，就可以重用这些数据了。我们也可以把 RDD 缓存到磁盘上而不是内存中。默认不进行持久化可能也显得有些奇怪，不过这对于大规模数据集是很有意义的：如果不会重用该 RDD，我们就没有必要浪费存储空间，Spark 可以直接遍历一遍数据然后计算出结果。
```

[cache()与使用默认存储级别调用persist()是一样的。](https://blog.csdn.net/houmou/article/details/52491419 )

### 1-2创建RDD

**创建RDD的两种方式**

​	--读取外部数据集

​	--在驱动器对一个集合进行并行化

### 1-3RDD操作

RDD的**转化操作**时<u>返回一个新的RDD</u>的操作，比如map、filter



RDD的**行动操作**是向驱动器程序<u>返回结果或把结果写入外部操作系统，会触发实际的计算</u>，比如count、first



#### 转化操作

**转化出来的RDD是惰性求值的，只有在行动操作中用到这些RDD才会被计算。**

```scala
/**
*选出log.txt文件里的错误消息
*/
val inputRDD = sc.textFile("log.txt")
val errorsRDD = inputRDD.filter(line => line.contains("error"))
//filter操作不会改变已有的RDD，实际上，会返回一个新的RDD
```

**转化操作可以操作任意数量的输入RDD。**

```scala
/**
*选出log.txt文件里的错误消息和警告
*/
val inputRDD = sc.textFile("log.txt")
val errorsRDD = inputRDD.filter(line => line.contains("error"))
val warningsRDD = inputRDD.filter(line => line.contains("warning"))
val badLinesRDD = errorsRDD.union(warningsRDD)//union操作两个RDD
```

**Spark会使用谱系图（lineage graph）来记录不同RDD之间的依赖关系。**

​	spark需要使用这些信息来<u>按需计算每个RDD</u>，也可以依靠谱系图在持久化的RDD<u>丢失部分数据时恢复所丢失的数据</u>。

![谱系图](/Users/zhangyanbo/Yber/文档/Sp/谱系图.png)

#### 行动操作

**行动操作的RDD会把最终求的的结果返回到驱动器程序，或者写入外部存储系统中。**

​	会<u>强制执行那些求值必须用到的RDD的转化操作</u>。

```scala
/**
*输出badLineRDD的相关信息
*/
println("Input had "+ badLinesRDD.count() + "concerning lines")
println("Here are 10 examples:")
badLinesRDD.take(10).foreach(println)
//take()获取了RDD的少量元素。
//RDD还有一个collect()函数，可以用来获取整个RDD中的数据。
```

**RDD的collect()函数，可以用来获取整个RDD中的数据。**

​	如果你的程序把<u>RDD筛选到一个很小的规模，并且你想在本地处理这些数据</u>，就可以使用它。

​	记住，当你的整个数据集能在单台机器的内存中放得下时，才能使用collect()。因此，<u>collect不能用在大规模数据集上</u>。

​	**通常，我们要把数据写到诸如HDFS这样的分布式的存储系统中。**

​	可以使用<u>saveAsTextFile()、saveAsSequenceFile()</u>,或者<u>任意其他行动操作</u>把RDD的数据内容以各种自带格式保存起来。

**需要注意，每当我们调用一个新的行动操作时，整个RDD都会从头开始计算。**

​	要避免这种低效率的行为，我们可以把中间结果持久化。

#### 惰性求值

**RDD的转化操作都是惰性求值的。这意味着在被调用行动操作之前Spark不会开始计算。**

​	--当我们对RDD调用转化操作（例如map）时，操作不会立即执行；Spark会在内部记录下所要执行的操作的相关信息。

​	--最好把每个RDD看作是我们通过转化操作构建出来的、记录着如何计算数据的指令列表。

**数据读取到RDD的操作也是惰性的。**

​	--当我们调用sc.textFile()时，数据并没有读取进来。

​	--和转化操作一样，读取数据的操作也可能会多次执行。

​	--虽然转化操作是惰性的，但是可以随时通过一个行动操作算子来强制Spark执行RDD的转化操作。

### 1-4向Spark传递函数

Spark的大部分转化操作和一部分行动操作，都需要依赖用户传递的函数来计算。

#### Scala的函数传递

**我们可以把定义的内联函数、方法的引用或静态方法传递给Spark。**

​	--我们需要考虑一些细节，比如所传递的函数及其引用的数据是可序列化的（实现了Java的Serializable接口）

​	--除此之外，传递一个对象的方法或字段时，会包含对整个对象的引用。

```scala

```





### 1-5常见的转化操作和行动操作

#### 基本RDD

##### 针对各个元素的转化操作

###### map

map()接收一个函数，把这个函数用于RDD的每个元素，将函数返回的结果作为结果RDD中对应元素的值。

###### filter

filter()接收一个函数，并将RDD中满足该函数的元素放入新的RDD中返回。

###### flatMap

flatMap()应用到RDD的每个元素上，返回的不是一个元素，而是一个返回值序列的迭代器。

​	--得到的是一个包含各个迭代器可以访问的所有元素的RDD。

​	--简单用途就是把输入字符串切分为单词。

```scala
val lines = sc.parallelize(List("hello world", "hi")) 
val words = lines.flatMap(line => line.split(" ")) 
words.first() // 返回"hello"
```

###### map和flatMap区别

![map和flatMap](/Users/zhangyanbo/Yber/文档/Sp/map和flatMap.png)

##### 伪集合的转换操作

**RDD不是严格意义上的集合，但是也支持数学上的集合操作，例如合并和相交。**

<img src="/Users/zhangyanbo/Yber/文档/Sp/伪集合转换RDD.png" alt="伪集合转换RDD" style="zoom:50%;" />

###### distinct()

distinct转化操作生成一个<u>只包含不同元素的新RDD</u>。

​	注意：distinct()<u>开销很大</u>，它需要将所有数据通过网络进行混洗（shuffle），确保每个元素只有一份。

###### union(other)

返回包含<u>两个RDD中**所有**元素</u>的新RDD。（**不去重**）

###### intersection(other)

只返回<u>两个RDD都有的</u>元素。（去重）

​	注意：性能差，因为需要网络混洗数据来发现所有共有元素。

###### subtract(other)

返回<u>只存在第一个RDD不存在第二个RDD的所有元素</u>组成的RDD。

​	注意：也需要数据混洗。

###### cartesian(other)

返回<u>所有可能的（a,b）对</u>，其中a是源RDD中的元素，b是另一个RDD。

​	在考虑所有可能组合相似度时，比较有用；但是求大规模RDD的笛卡尔积开销巨大。

<img src="/Users/zhangyanbo/Yber/文档/Sp/笛卡尔积.png" alt="笛卡尔积" style="zoom:50%;" />

##### 行动操作

###### reduce()

接收一个函数作为参数，这个函数要<u>操作两个RDD的元素类型的数据并返回一个同样类型的新元素</u>。

```scala
//一个简单的例子就是函数+，可以对RDD进行累加。计算所有元素的总和、元素个数、以及其他类型的剧和操作。
val sum = rdd.reduce((x, y) => x + y)
```

###### fold()

接收一个<u>与reduce()接收的函数签名相同的函数，再加上一个“初始值”来作为每个分区第一次调用时的结果</u>。

​	你所提供的初始值应该是你提供的操作的单位元素；比如，使用函数对初始值进行多次计算不会改变结果（+对应的0，*对应的1，或拼接操作对应的空列表）。

**fold、reduce要求函数返回值类型和我们需要操作的RDD中的元素类型相同。**

​	在计算平均值时，需要记录遍历过程中的计数以及元素数量，这就需要我们返回一个二元组。可以先对数据使用map()操作，来把元素转为该元素和1的二元组，也就是我们希望返回的类型，这样reduce()就可以以二元组的形式进行规约了。

###### aggregate()

从返回值类型必须和操作的RDD类型相同的限制中解放出来。

需要提供我们<u>期待返回的类型的初始值</u>。然后通过<u>一个函数把RDD中的元素合并起来放进累加器</u>。考虑到每个节点是在本地进行累加的，<u>还需要提供第二个函数将累加器两两合并</u>。

```scala
/**
*(0,0)定义了期望返回的类型初始值
*(acc, value)定义了累加值并计数
*(acc1, acc2)定义了不同节点之间值合并、计数合并
*最后返回的result就是（value的和,value个数的和）
*/
val result = input.aggregate((0, 0))(
  (acc, value) => (acc._1 + value, acc._2 + 1),
  (acc1, acc2) => (acc1._1 + acc2._1, acc1._2 + acc2._2)) 
val avg = result._1 / result._2.toDouble
```

###### collect()

会将<u>整个RDD内容返回</u>。通常用于单元测试。collect要求<u>所有数据都必须能一同放入单台机器内存中</u>。

###### take(n)

返回<u>RDD的n个元素，并且尝试只返回尽量少的分区</u>，操作返回的元素顺序可能和预期的不一样。

###### top(n)

从RDD中<u>获取前几个元素</u>。top会使用<u>数据的默认顺序</u>，但我们也<u>可以提供自己的比较函数</u>，来提取前几个元素。

###### takeSample(withReplacement,num,seed)

从数据中<u>获取一个采样</u>，并指定是否替换。

###### foreach()

对RDD的<u>每个元素进行操作</u>，而不需要把RDD发回本地。

​	比如可以使用JSON格式把数据发送到一个网络服务器，或把数据存储到数据库中。

###### count()

返回<u>元素个数</u>。

###### countByValue()

返回一个<u>各值到值对应的计数的映射表</u>。

#### 在不同RDD类型间转换

**有些函数只能用在特定类型的RDD。**

​	--join()只能用在键值对RDD上

​	--mean()和variance()只能用在数值RDD上

**Scala中，将RDD转为有特定函数的RDD（比如RDD[Double]上进行数值操作）是由<u>隐式转换</u>来自动处理的。**

```scala
//使用这些隐式转换。
import org.apache.spark.SparkContext._ 
```

这些隐式转换可以隐试的将一个RDD转换为各种封装类。

隐式转换很强大，但是会让阅读代码的人感到困惑。所以，当我们在Scala文档查找函数的时候，不要忘记了那些封装了专用类中的函数。



### 1-6持久化（缓存）

**为了避免多次计算同一个RDD，可以让Spark对数据进行持久化。**

spark持久化存储一个RDD时，计算出RDD的节点会分别保存它们所求出的分区数据。

如果一个持久化数据节点发生故障，Spark会在需要用到缓存的数据时重算丢失的数据分区

**如果希望节点故障不会拖累我们的执行速度，也可以把数据备份到多个节点上。**



**出于不同的目的，我们可以为RDD选择不同的持久化级别。**

默认情况下，persist()会把数据以序列化的形式 缓存在JVM的堆空间中。

![持久化级别](/Users/zhangyanbo/Yber/文档/Sp/持久化级别.png)

RDD有一个方法叫做**unpersist()，该方法手动的将持久化的RDD从缓存中移除。**

## 键值对操作

**键值对类型的RDD被称为pairRDD。它们提供了并行操作各个键或跨节点重新进行数据分组的操作接口。**

​	--pairRDD提供的reduceByKey()方法，可以分别归约每个键对应的数据。

​	--join方法，可以把两个RDD中键相同的元素组合到一起。

### 2-1创建Pari RDD

 **创建键值对RDD的方式：**

​	--很多存储键值对的数据格式在读取时直接返回由其键值对组成的Pari RDD。

​	--当需要把一个普通的RDD转为pair RDD的时候，可以调用map()函数来实现，传递的函数需要返回键值对。

```scala
val pairs = lines.map(x => (x.split(" ")(0), x))
```

### 2-2Pair RDD的转化操作

**Pair RDD 可以使用所有标准RDD上的可用的转化操作。**

​	由于pair RDD包含二元组，所以需要传递的函数应该操作二元组而不是独立的元素。

**Pair RDD也还是RDD，因此同样支持RDD所支持的函数。**

```scala
//删除值超过20个字符的行
pairs.filter{case (key, value) => value.length < 20}
```

**有时，我们只想访问pair RDD的值的部分，这时操作二元组很麻烦。可以使用mapValue(func)函数。**

​	类似于map{case (x, y): (x, func(y))}。

#### 聚合操作

###### reduceByKey()

和reduce()类似，接收一个函数，使用该函数对值进行合并。

reduceByKey()会为数据集中的<u>每个键进行并行的归约操作</u>，每个归约操作会将<u>键相同的值合并起来</u>。返回一个由各键和对应键鬼月出来的结果值组成的新的RDD。

###### foldByKey()

和fold()类似，都使用一个与RDD和合并函数中的数据类型相同的零值作为初始值。

###### **计算每个键对应的平均值**

```scala
/**
*mapValues不改变键，将值变为二元组（值,1）
*reduceByKey根据键进行聚合，并传入值操作的函数【x、y代表了两个值（这里是两个二元组）】
*/
rdd.mapValues(x => (x, 1)).reduceByKey((x, y) => (x._1 + y._1, x._2 + y._2))
```

<img src="/Users/zhangyanbo/Yber/文档/Sp/PairRDD1.png" alt="PairRDD1" style="zoom:50%;" />

###### **单词计数**

```scala
val input = sc.textFile("s3://...") 
val words = input.flatMap(x => x.split(" ")) 
val result = words.map(x => (x, 1)).reduceByKey((x, y) => x + y)
```

###### countByValue()

我们可以对第一个RDD使用<u>countByValue()更快的实现 单词计数</u>。

```scala
input.flatMap(x => x.split(" ")).countByValue()
```

###### combineByKey()

最为常用的基于键进行聚合的操作。<u>可以让用户返回与输入类型不同的返回值</u>。

**conbineByKey会遍历分区中的每个元素。**

<u>有多个参数，分别对应聚合操作的各个阶段。</u>

​	如果是一个新的元素，conbineByKey会使用一个叫做createCombiner()的函数创建键对应的累加器的初始值。这一过程在<u>每个分区</u>第一次出现各个键时发生。

​	如果这是一个处理当前分区之前已经遇到的键，则会使用mergeValue()方法将该键的累加器对应的当前值与这个新的值进行合并。

​	由于每个分区是独立的，同一个键可能有多个累加器。多个分区都有对应同一个键的累加器，就需要使用用户提供的mergeCombiners()方法将各个分区的结果进行合并。

​	**如果已知数据在进行conbineByKey时无法从map端聚合中获益的话，可以禁用它。**如果希望禁用map端组合，需要指定分区方式。

可以通过传递rdd.partitioner来直接使用源RDD的分区方式。

```scala
/**
*参数1:(v) => (v, 1)	
--	createCombiner()的函数创建键对应的累加器的初始值
*参数2:(acc: (Int, Int), v) => (acc._1 + v, acc._2 + 1)	
--	当前分区之前已经遇到的键，则会使用mergeValue()方法将该键的累*加器对应的当前值与这个新的值进行合并
*参数3:(acc1: (Int, Int), acc2: (Int, Int)) => (acc1._1 + acc2._1, acc1._2 + acc2._2)   
--	多个分区都有对应同一个键的累加器，mergeCombiners()方法将各个分区的结果进行合并
*/
val result = input.combineByKey(
  (v) => (v, 1),   
  (acc: (Int, Int), v) => (acc._1 + v, acc._2 + 1),   
  (acc1: (Int, Int), acc2: (Int, Int)) => (acc1._1 + acc2._1, acc1._2 + acc2._2)   
	).map{ case (key, value) => (key, value._1 / value._2.toFloat) }   
result.collectAsMap().map(println(_))
```

<img src="/Users/zhangyanbo/Yber/文档/Sp/combineByKey.png" alt="combineByKey" style="zoom:50%;" />

###### 并行度调优

**每个RDD都有固定数目的分区，分区数决定了在RDD上执行操作时的并行度。**

Spark始终尝试根据集群大小推断出一个有意义的默认值，但是有时候你可能要对并行度调优来获得更好的性能表现。

**大多数操作符都能接受第二参数，这个参数用来指定分组结果或聚合结果的RDD的分区数。**

```scala
val data = Seq(("a", 3), ("b", 4), ("a", 1)) 
sc.parallelize(data).reduceByKey((x, y) => x + y)    // 默认并行度 
sc.parallelize(data).reduceByKey((x, y) => x + y,10)    // 自定义并行度
```

##### [ coalesce 与 repartition的区别](https://www.cnblogs.com/jiangxiaoxian/p/9539760.html)

###### repartition()

repartition()函数，把数据通过网络进行混洗，并创建出新的分区集合。repartition只是coalesce接口中shuffle为true的实现

**重分区是代价比较大的操作。**

###### coalesce()

优化版的repartition。

可以使用rdd.partitions.size()查看RDD分区数。

#### 数据分组

###### groupByKey()

使用RDD的键对数据进行分组。k-v型RDD得到的结果RDD类型是[ k-Iterable[v] ].

###### groupBy()

用于未成对的数据上，也可以根据键相同以外的条件分组。

​	可以接收一个函数，对源RDD中的每个元素使用该函数，将返回结果作为键在进行分组。

```scala
//
```

###### cogroup()

**对多个共享同一个键的RDD进行分组。**

​	对于两个键类型为k而值类型分别为V、W的RDD进行cogroup时，得到的RDD类型为[ (K, (Iterable[V], Iterable[W])) ]

​	其中一个RDD对于另一个RDD中存在的键没有对应的记录，那么对应的迭代器则为空。

**cogroup()提供了为多个RDD进行数据分组的方法。**

**可以 1: 实现连接操作、2:求键的交集、3:同时应用于3个以上RDD。**

#### 连接

1. 左外连接
2. 右外连接
3. 交叉连接
4. 内连接

###### join

join表示内连接。只有两个PairRDD都存在的键，才能成功。

 ```scala
//左侧
storeAddress = {   
  (Store("Ritual"), "1026 Valencia St"), 
  (Store("Philz"), "748 Van Ness Ave"),   
  (Store("Philz"), "3101 24th St"), 
  (Store("Starbucks"), "Seattle")}  
//右侧
storeRating = {   
  (Store("Ritual"), 4.9), 
  (Store("Philz"), 4.8))}  
//内连接
storeAddress.join(storeRating) == {   
  (Store("Ritual"), ("1026 Valencia St", 4.9)),   
  (Store("Philz"), ("748 Van Ness Ave", 4.8)),   
  (Store("Philz"), ("3101 24th St", 4.8))}
 ```

###### leftOuterJoin()

左外连接。

```scala
//左侧
storeAddress = {   
  (Store("Ritual"), "1026 Valencia St"), 
  (Store("Philz"), "748 Van Ness Ave"),   
  (Store("Philz"), "3101 24th St"), 
  (Store("Starbucks"), "Seattle")}  
//右侧
storeRating = {   
  (Store("Ritual"), 4.9), 
  (Store("Philz"), 4.8))}  
//左外连接
storeAddress.leftOuterJoin(storeRating) == 
{(Store("Ritual"),("1026 Valencia St",Some(4.9))),   
 (Store("Starbucks"),("Seattle",None)),   
 (Store("Philz"),("748 Van Ness Ave",Some(4.8))),   
 (Store("Philz"),("3101 24th St",Some(4.8)))}
```



###### rightOuterJoin()

右外连接。

```scala
//左侧
storeAddress = {   
  (Store("Ritual"), "1026 Valencia St"), 
  (Store("Philz"), "748 Van Ness Ave"),   
  (Store("Philz"), "3101 24th St"), 
  (Store("Starbucks"), "Seattle")}  
//右侧
storeRating = {   
  (Store("Ritual"), 4.9), 
  (Store("Philz"), 4.8))}  
//右外连接
storeAddress.rightOuterJoin(storeRating) == 
{(Store("Ritual"),(Some("1026 Valencia St"),4.9)),   
 (Store("Philz"),(Some("748 Van Ness Ave"),4.8)),   
 (Store("Philz"), (Some("3101 24th St"),4.8))}
```

#### 数据排序

###### sortByKey()

**作用于Key-Value形式的RDD，并对Key进行排序。**

```scala
def sortByKey(ascending: Boolean = true, numPartitions: Int = self.partitions.size)
    : RDD[(K, V)] =
{
  val part = new RangePartitioner(numPartitions, self, ascending)
  new ShuffledRDD[K, V, V](self, part)
    .setKeyOrdering(if (ascending) ordering else ordering.reverse)
}
//ascending	-- true 生序 ,false 降序
//numPartitions	--	分区数（全局排序应该设置为1）
```

**在OrderedRDDFunctions类中有个变量ordering它是隐形的：`private val ordering = implicitly[Ordering[K]]`。它是默认的排序规则，我们可以对它进行重写**

```scala
implicit val sortIntegersByString = new Ordering[Int] {
  override def compare(a: Int, b: Int) = a.toString.compare(b.toString) 
}
//然后再调用SortByKey即可。
```

### 2-3Pair RDD行动操作

###### countByKey()

###### collectAsMap()

###### lookup(key)

<img src="/Users/zhangyanbo/Yber/文档/Sp/PairRDD行动操作1.png" alt="PairRDD行动操作1" style="zoom:50%;" />

### 2-4数据分区

**在分布式程序中，通讯的代价是很大的，因此控制数据分布以获得最少的网络传输可以极大的提升整体性能。**



**分区并不是对所有应用都有好处。**

​	--如果给定的RDD只需要被扫描一次，完全没必要预先进行分区处理。

​	--当数据集多次在诸如连接这种基于键的操作中使用时，分区才会有帮助。

**Spark中所有键值对RDD都可以进行分区。**

​	--spark没有给出显示控制每个键具体落在哪个工作节点上的方法（部分原因是Spark即使在某些节点失败时依然可以工作）

**Spark可以确保同一组键出现在同一个节点上。**



##### 应用：用户信息表关联用户访问表

​	我们分析这样一个应用，它在内存中保存着一张很大的用户信息表——也就是一个由 (UserID, UserInfo) 对组成的 RDD，其中 UserInfo 包含一个该用户所订阅的主题的列表。该应用会周期性地将这张表与一个小文件进行组合，这个小文件中存着过去五分钟内发生的事件——其实就是一个由 (UserID, LinkInfo) 对组成的表，存放着过去五分钟内某网站各用户的访问情况。例如，我们可能需要对用户访问其未订阅主题的页面的情况进行统计。我们可以使用 Spark 的 join() 操作来实现这个组合操作，其中需要把UserInfo 和 LinkInfo 的有序对根据 UserID 进行分组。

```scala
// 初始化代码； 从HDFS商的一个Hadoop SequenceFile中读取用户信息 
// userData中的元素会根据它们被读取时的来源， 即HDFS块所在的节点来分布 
// Spark此时无法获知某个特定的UserID对应的记录位于哪个节点上 
val sc = new SparkContext(...) 
val userData = sc.sequenceFile[UserID, UserInfo]("hdfs://...").persist()  
// 周期性调用函数来处理过去五分钟产生的事件日志 
// 假设这是一个包含(UserID, LinkInfo)对的SequenceFile 
def processNewLogs(logFileName: String) {   
  val events = sc.sequenceFile[UserID, LinkInfo](logFileName)   
  val joined = userData.join(events)// RDD of (UserID, (UserInfo, LinkInfo)) pairs   
  val offTopicVisits = joined.filter {
    case (userId, (userInfo, linkInfo)) => // Expand the tuple into its components       
    !userInfo.topics.contains(linkInfo.topic)   }.count()   
  println("Number of visits to non-subscribed topics: " + offTopicVisits) 
}
```

**正确运行，但是不够高效。**

默认情况下，连接操作会将两个数据集中的所有键的哈希值都求出来，将该哈希值相同的记录通过网络传到同一台机器上，然后在那台机器上对所有键相同的记录进行连接操作（见图 4-4） 。因为 userData 表比每五分钟出现的访问日志表 events 要大得多，所以要浪费时间做很多额外工作：在<u>每次调用时都对 userData 表进行哈希值计算和跨节点数据混洗，虽然这些数据从来都不会变化</u>。

<img src="/Users/zhangyanbo/Yber/文档/Sp/数据分区1.png" alt="数据分区1" style="zoom:50%;" />

**要解决这一问题也很简单**：在程序开始时，对 userData 表使用 partitionBy() 转化操作，将这张表转为哈希分区。可以通过向 partitionBy 传递一个 spark.HashPartitioner 对象来实现该操作。

```scala
val sc = new SparkContext(...) 
val userData = sc.sequenceFile[UserID, UserInfo]("hdfs://...")                  
.partitionBy(new HashPartitioner(100))   // 构造100个分区                  
.persist()
```

由于在<u>构建 userData 时 调 用 了 partitionBy()，Spark 就 知 道 了 该 RDD 是 根 据 键 的 哈 希 值 来 分区的，这样在调用 join() 时，Spark 就会利用到这一点</u>。具体来说，当调用 userData.join(events) 时，Spark 只会对 events 进行数据混洗操作，将 events 中特定 UserID 的记录发送到 userData 的对应分区所在的那台机器上（见图 4-5） 。这样，需要通过网络传输的数据就大大减少了，程序运行速度也可以显著提升了。

<img src="/Users/zhangyanbo/Yber/文档/Sp/数据分区2.png" alt="数据分区2" style="zoom:50%;" />

###### partitionBy()

**partitionBy()是一个转化操作，因此它的返回值是一个新的RDD。**

​	--不会改变原来的RDD，RDD一旦创建就无法修改。

​	--<u>应该对partitionBy的结果进行持久化并保存</u>。

​	--传递给partitionBy的分区数目会控制之后对这个RDD的进一步操作时有多少任务会并行执行。（比如连接操作）。总的来说，这个值至少应该和集群中的总核心数一样。

**如果没有将 partitionBy() 转化操作的结果持久化，那么后面每次用到这个RDD 时都会重复地对数据进行分区操作。**不进行持久化会导致整个 RDD 谱系图重新求值。那样的话，partitionBy() 带来的好处就会被抵消，导致重复对数据进行分区以及跨节点的混洗，和没有指定分区方式时发生的情况十分相似。

#### 获取RDD的分区方式

##### **使用RDD的partitioner属性来获取RDD的分区方式。**

​	--会返回一个scala.Option对象；这是Scala中用来存放可能存在的对象的容器类。如果存在值的话，这个值是一个spark.Partitioner对象。这本质上是一个告诉我们RDD中各个键分别属于哪个分区的函数。

​		--可以对这个Option对象调用isDefined()来检查其中是否有值。

​		--调用get()来获取其中的值。

```scala
//创建出了一个由 (Int, Int) 对组成的 RDD
scala> val pairs = sc.parallelize(List((1, 1), (2, 2), (3, 3))) 
pairs: spark.RDD[(Int, Int)] = ParallelCollectionRDD[0] at parallelize at <console>:12  

//初始时没有分区方式信息（一个值为 None 的 Option 对象）
scala> pairs.partitioner 
res0: Option[spark.Partitioner] = None  

//对第一个 RDD 进行哈希分区，创建出了第二个 RDD。
scala> val partitioned = pairs.partitionBy(new spark.HashPartitioner(2)) 
partitioned: spark.RDD[(Int, Int)] = ShuffledRDD[1] at partitionBy at <console>:14  

scala> partitioned.partitioner 
res1: Option[spark.Partitioner] = Some(spark.HashPartitioner@5147788d)
```

**如果要在后续操作中使用partitionBy定义好的RDD。应该加上persist()。**

​	如果不加上的话，后续RDD操作会对RDD整个谱系重新求值。

#### 从分区中获益的操作

##### 将数据根据键跨节点进行混洗的，所有这些操作都会从数据分区中获益。

cogroup()、groupWith()、join()、leftOuterJoin()、rightOuterJoin()、groupByKey()、reduceByKey()、combineByKey() 以及 lookup()。

​	-<u>-reduceByKey()这样只作用于单个RDD的操作，如果运行在未分区的RDD上的时候会导致每个键的所有对应值都在每台机器上进行本地计算，只需要把本地规约出的结果从各个工作节点传回主节点，所以原本的网络开销就不算大</u>。

​	--<u>对于cogroup()和join()这样的二元操作，预先进行数据分区会导致其中至少一个RDD不发生数据混洗</u>。

​		如果两个RDD使用同样的分区方式，并且它们还缓存在同样的机器上（比如一个RDD是通过mapValues()从另个RDD创建出来的，它们会拥有同样的键和分区方式）

​		或者其中一个RDD还没有被计算出来

​	<u>那么跨节点数据混洗就不会发生了</u>。

#### 影响分区方式的操作

**Spark内部知道各操作会如何影响分区方式，并将会对数据进行分区的操作的结果RDD自动设置为对应的分区器。**

​	例如，你调用join来连接两个RDD；由于键相同的元素会被哈希到同一台机器上，Spark就知道输出结果也是哈希分区的，这样对结果进行诸如reduceByKey()这样的操作就会明显变快。

**不过，转化操作的结果并不一定会按已知的分区方式分区，这时候输出的RDD可能就会没有设置分区器。**

​	例如，你对一个哈希分区的键值对RDD调用map时，由于传递给map()的函数理论上可以改变元素的键，因此结果就不会有固定的分区方式。Spark不会分析你的函数来判断键是否会被保留下来。

​	但是Spark提供了另外两个操作mapValues()和flatMapValues()作为替代方法，它们可以保证二元组的键保持不变。

**所有会为生成的结果RDD设好分区方式的操作：**

​	cogroup()、groupWith()、join()、leftOuterJoin()、rightOuterJoin()、groupByKey()、reduceByKey()、combineByKey()、partitionBy()、sort()、mapValues()（如果父 RDD 有分区方式的话） 、flatMapValues()（如果父 RDD 有分区方式的话） ，以及 filter()（如果父 RDD 有分区方式的话）

**对于二元操作，输出数据的分区方式取决于父RDD的分区方式。**

​	--默认情况下，结果采用哈希分区，分区的数量和操作的并行度一样。

​	--如果一个父RDD已经设置过分区方式，那么结果就会采用那种分区方式；

​	--如果两个父类RDD都设置过分区方式，结果RDD会采用第一个父RDD的分区方式。

#### PageRank算法

**用来根据外部文档指向一个文档的链接，对集合中每个文档的重要程度赋一个度量值。**

​	可以用于对网页进行排序；

​	也可以用于排序科技文章或社交网络中有影响的用户。

**PageRank是执行多次连接的一个迭代算法。**

...



#### 自定义分区方式



## 数据读取与保存

**有时候，数据量可能大到无法放到一台机器中。**



**Spark支持很多种输入输出源。常见的3种数据源：**

​	--文件格式与文件系统

​	--Spark Sql中的结构化数据源

​	--数据库与键值存储



### 3-1文件格式

**Spark会根据文件扩展名选择对应的处理方式。这一过程是封装好的，对用户透明。**

![文件格式1](/Users/zhangyanbo/Yber/文档/Sp/文件格式1.png)

#### 文本文件

- 当一个文本文件读取为RDD时，输入的每一行都会成为RDD的一个元素。

- 也可以将多个完整的文本文件一次性读取为一个pair RDD，键是文件名，值是文件内容。

##### 读取文本文件

###### textFile()

调用SparkContext中的textFile()函数，就可以读取一个文本文件。

```scala
val input = sc.textFile("file:///home/holden/repos/spark/README.md")
```

###### wholeTextFiles()

**方法会返回一个pair RDD，其中键是输入文件的文件名。**

通过将数据加载到PairRDD中（每个输入文件一个记录），wholeTextFiles保留了数据与包含数据的文件之间的关系。记录将具有格式（fileName，fileContent）。这意味着<u>加载大文件是有风险的</u>（可能会导致性能下降或OutOfMemoryError，因为每个文件必须存储在单个节点上）。根据用户输入或Spark的配置进行分区-多个文件可能会加载到单个分区中。

###### 通配符读取文件

Spark支持读取给定目录中的所有文件，以及在输入路径中使用通配符。

##### 保存文本文件

###### saveAsTextFile()

方法接收一个路径，并将RDD中的内容都输入到路径对应的文件中。

​	--spark将传入的路径当作目录对待，会在目录下输出多个文件。这样Spark就可以在多个节点并行输出了。

​	--不能控制数据的那一部分输出到那个文件中。

​	--可以控制输出格式。

#### JSON

一种适用广泛的半结构化数据格式。

##### 读取JSON

<u>将数据作文文本读取，然后对JSON数据进行解析。</u>

​	--该方法假设每一行都是一个JSON记录。

<u>如果有跨行的JSON数据，只能读入整个文件，然后对每个文件进行解析。</u>

<u>如果JSON解析器开销比较大，可以使用mapPartitions()重用解析器。			？？？</u>



**对于大规模数据来说，格式错误是家常便饭。如果选择跳过不正确的数据，你应该使用累加器来追踪错误的个数。**



##### 保存JSON

将结构化数据组成的RDD转化为字符串RDD，然后使用Spark的文本文件API写出去。

#### 逗号分隔值与制表符分隔值

##### 读取SCV



##### 保存CSV



#### SequenceFile



#### 对象文件



#### 文件压缩



### 3-2文件系统

#### 本地/“常规”文件系统

spark支持从本地文件系统中读取文件，不过它要求<u>文件在集群中所有节点的相同路径下**都**可以找到</u>。

```scala
val rdd = sc.textFile("file:///home/holden/happypandas.gz")
```

如果文件还没有放在集群的所有节点上，你可以在驱动器程序中从本地读取该文件而无需整个集群，然后调用paralleize将内容发送到工作节点。（可能会慢）

#### HDFS

在 Spark 中使用 HDFS 只需要将输入输出路径指定为 **hdfs://master:port/path** 就够了。



### 3-3 Spark SQL中的结构化数据

**我们把一条sql查询给Spark SQL，让它对一个数据源执行查询（选出一些字段或者对字段使用一些函数），然后得到有Row对象组成的RDD，每个Row对象表示一条记录。**

​	--Java和Scala中，Row对象的访问是基于下标的。

​	--每个Row都有一个get()方法，会返回一个一般类型让我们可以进行类型转换。另外还有针对常见基本类型的专用get方法（getFloat()、getInt()、getLong()、getString()、getShort()、getBoolean()）



#### Hive

hive是Hadoop上一种常见的结构化数据源。**Spark SQL可以读取Hive支持的任何表。**



**要把Spark SQL连接到已有的Hive上，你需要提供Hive的配置文件。**

​	--将hive-site.xml文件复制到Spark的./conf/目录下。

​	--然后创建出HiveContext对象，也就是Spark SQL的入口；

​	--最后可以使用HQL对你的表进行查询，并以组成RDD的形式拿回数据。

```scala
import org.apache.spark.sql.hive.HiveContext  

val hiveCtx = new org.apache.spark.sql.hive.HiveContext(sc) 
val rows = hiveCtx.sql("SELECT name, age FROM users") 
val firstRow = rows.first() println(firstRow.getString(0)) // 字段0是name字段
```

#### JSON



### 3-4 数据库

#### Java数据库连接



#### HBase



#### Elasticsearch



## Spark编程进阶

**两种类型的共享变量：**

​	--**累加器**（accumulator）	用来对信息进行聚合。

​	--**广播变量**（broadcast variable）	用来高效分发较大的对象。



**通常再向Spark传递函数时，比如使用map()或者filter()传递条件时，可以使用驱动器程序中定义的变量，但是集群中运行的每个任务都会得到这些变量的一份新的副本；更新这些副本的值不会影响到驱动器中对应的变量。**



### 4-1累加器

提供了**将工作节点的值聚合到驱动器程序中**的简单语法。

​	--一个常见用途就是在调试时对作业<u>执行过程中的事件进行计数</u>。

```scala
val sc = new SparkContext(...) 
val file = sc.textFile("file.txt")  

val blankLines = sc.accumulator(0) // 创建Accumulator[Int]并初始化为0  

val callSigns = file.flatMap(line => {
		if (line == "") {     
      blankLines += 1 // 累加器加1   
    }
  	line.split(" ") 
})  

callSigns.saveAsTextFile("output.txt") 
println("Blank lines: " + blankLines.value)
```



**注意：只有在运行了saveAsTextFile()行动操作之后才能看到正确的计数，因为行动操作之前的转化操作flatMap()是惰性的，所以作为计算副产品的累加器只有在惰性操作flatMap()被行动操作强制触发才会开始求值。**



<u>当然，我们也可以使用reduce()这演的行动操作将整个RDD中的值都聚合到驱动器中。</u>**但是我们有时候希望能用更简单的方法，来对那些RDD本身的范围和粒度不一样的值进行聚合。聚合可以发生在RDD进行转化操作的过程中。**



##### 累加器的用法：

1. 通过在驱动器中调用 SparkContext.accumulator(initialValue) 方法，创建出存有初始值的累加器。返回值为 org.apache.spark.Accumulator[T] 对象，其中 T 是初始值initialValue 的类型。
2. Spark 闭包里的执行器代码可以使用累加器的 += 方法 （在 Java 中是 add） 增加累加器的值。
3. 驱动器程序可以调用累加器的 value 属性（在 Java 中使用 value() 或 setValue()）来访问累加器的值。

**注意：工作节点不能访问累加器的值。**

​	--从任务角度来看，累加器是一个只写变量。

​	--这种模式下，累加器的实现可以更加高效，不需要对每次更新操作进行复杂的通信。

**累加器的值只有在驱动器程序中可以访问，所以检查也应该在驱动器程序中完成。**



##### 累加器与容错性：

**Spark会自动重新执行失败的或较慢的任务来应对有错误或比较慢的机器。**

​	--例如，如果对某分区执行 map() 操作的节点失败了，Spark 会在另一个节点上重新运行该任务。

​	--即使该节点没有崩溃，而只是处理速度比别的节点慢很多，Spark 也可以抢占式地在另一个节点上启动一个“投机” （speculative）型的任务副本，如果该任务更早结束就可以直接获取结果。

​	--即使没有节点失败，Spark 有时也需要重新运行任务来获取缓存中被移除出内存的数据。

因此，<u>同一个函数可能对同一个数据运行了多次，这取决于集群发生了什么。</u>



**如何处理累加器的这种问题：**

- ​	对于在行动操作中使用的累加器，Spark只会把每个任务对累加器的修改应用一次。
  - 如果想要一个无论在失败还是重复计算时都绝对可靠的累加器，我们必须把它放在foreach()这样的行动操作中。
- ​    对于在RDD转化操作中使用的累加器，无法保证只更新一次。
  - 转化操作中，累加器通常只用于调试目的。

##### 自定义累加器：

Spark还支持Double、Long和Float型的累加器。同时也引入了自定义累加器和聚合操作的API。

**自定义累加器要扩展AccumulatorParam。**

​	只要操作同时满足交换律和结合律，就可以使用任意的操作来代替数值上的加法。



### 4-2广播变量

**可以让程序高效地向所有工作节点发送一个较大的只读值，以供一个或多个 Spark 操作使用。**

​	--比如，如果你的应用需要向所有节点发送一个较大的只读查询表。

**广播变量就是类型为：spark.broadcast.Broadcast[ T ]的一个对象，其中存放着类型为T的值。**

​	--可以在任务中<u>通过对Broadcast对象调用value来获取该对象的值</u>。

​	--这个<u>值只会被发送到各节点一次</u>，使用的是一种高效的类似BitTorrent的通信机制。

##### 广播变量的使用：

​	(1) 通过对一个类型 T 的对象调用 SparkContext.broadcast 创建出一个 Broadcast[T] 对象。
任何可序列化的类型都可以这么实现。
​	(2) 通过 value 属性访问该对象的值（在 Java 中为 value() 方法） 。
​	(3) 变量只会被发到各个节点一次，应作为只读值处理（修改这个值不会影响到别的节点） 。

**满足只读要求的最容易的使用方式是，广播<u>基本类型的值</u>或者<u>引用不可变对象</u>。**

​	--有时候需要<u>传一个可变对象可能更加方便与高效。此时，注意维护只读条件</u>。

##### 广播的优化：

当广播一个比较大的值时，选择**既快又好的序列化格式是很重要的**，因为如果序列化对象的时间很长或者传送花费的时间太久，这段时间很容易就成为性能瓶颈。

​	--Scala和Java API默认使用的序列化库是Java序列化库；它对于除了基本类型的数组以外都比较低效。

​	--可以使用spark.serializer属性来选择另一个序列化库来优化序列化过程。

### 4-3基于分区进行操作

基于分区对数据进行操作可以让我们**避免为每个数据元素进行重复的配置工作**。

​	--Spark 提供基于分区的 map 和 foreach，让你的部分代码只对 RDD 的每个分区运行一次，这样可以帮助降低这些操作的代价。

![基于分区的操作](/Users/zhangyanbo/Yber/文档/Sp/基于分区的操作.png)

### 4-4数值RDD的操作

**Spark 对包含数值数据的 RDD 提供了一些描述性的统计操作。**

Spark 的数值操作是通过**流式算法实现**的，允许以每次一个元素的方式构建出模型。<u>这些统计数据都会在调用 stats() 时通过一次遍历数据计算出来，并以 StatsCounter 对象返回</u>。

![数值RDD操作](/Users/zhangyanbo/Yber/文档/Sp/数值RDD操作.png)

```scala
// 现在要移除一些异常值， 因为有些地点可能是误报的 
// 首先要获取字符串RDD并将它转换为双精度浮点型 
val distanceDouble = distance.map(string => string.toDouble) 
val stats = distanceDoubles.stats() 
val stddev = stats.stdev 
val mean = stats.mean 
val reasonableDistances = distanceDoubles.filter(x => math.abs(x-mean) < 3 * stddev) println(reasonableDistance.collect().toList)
```

## Spark Streaming

**Spark Streaming 使用离散化流（discretized stream）作为抽象表示，叫作 DStream。**

​	--DStream 是随时间推移而收到的数据的序列。

​	--在内部，每个时间区间收到的数据都作为 RDD 存在，而 DStream 是由这些 RDD 所组成的序列（因此得名“离散化” ） 。

**DStream 可以从各种输入源创建，比如 Flume、Kafka 或者 HDFS。**

**创建出来的 DStream 支持两种操作**

​	一种是转化操作（transformation） ，会生成一个新的DStream；

​	另一种是输出操作（output operation） ，可以把数据写入外部系统中。

从创建 StreamingContext 开始，它是流计算功能的主要入口。StreamingContext 会在底层创建出 SparkContext，用来处理数据。其构造函数还接收用来指定多长时间处理一次新数据的批次间隔（batch interval）作为输入。

#### 例子：流式筛选，打印包含“error”的行

```scala
/**
* 在终端中输入nc命令发送数据
*	nc -lk 7777
*	然后运行程序，即可。
*/

import org.apache.spark.streaming.{Seconds, StreamingContext}
import org.apache.spark.{SparkConf, SparkContext}

object SparkStreamingTest {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setMaster("local[3]").setAppName("SparkStreamingTest")

    val sc = new SparkContext(conf)

    val ssc = new StreamingContext(sc,Seconds(5))


    val lines = ssc.socketTextStream("localhost",7777)

    val errorlines = lines.filter(t => t.contains("error"))

    errorlines.print()
    
// 启动流计算环境StreamingContext并等待它"完成" ssc.start() 
// 等待作业完成 ssc.awaitTermination()
    
    ssc.start()
    ssc.awaitTermination()
  }
}
```

![DStream转化关系](/Users/zhangyanbo/Yber/文档/Sp/DStream转化关系.png)

### 8-1 架构与抽象

Spark Streaming 使用**“微批次”的架构，把流式计算当作一系列连续的小规模批处理**来对待。

​	--每个输入批次都形成一个 RDD，以 Spark 作业的方式处理并生成其他的 RDD。处理的结果可以以批处理的方式传给外部系统。

<img src="/Users/zhangyanbo/Yber/文档/Sp/Streaming高层次架构.png" alt="Streaming高层次架构" style="zoom:50%;" />

Spark Streaming 的编程抽象是**离散化流**，也就是 **DStream** 。它**是一个 RDD 序列，每个 RDD 代表数据流中一个时间片内的数据**。

![DStream](/Users/zhangyanbo/Yber/文档/Sp/DStream.png)

​	--你可以从外部输入源**创建 DStream**;

​	--也可以对其他 DStream 应用进行**转化操作得到新的DStream**。DStream 支持许多第 3 章中所讲到的 RDD 支持的转化操作;

​	--另外，DStream 还有“**有状态”的转化操作，可以用来聚合不同时间片内的数据**。



**Spark Streaming 对DStream提供的容错性与Spark为RDD提供的容错性一致：**

​	--只要输入的数据还在，就可以使用RDD系谱算出任意状态。

​		--默认情况下，收到的数据分别存在于两个节点上，这样 Spark 可以容忍一个工作节点的故障。不过，如果只用谱系图来恢复的话，重算有可能会花很长时间，因为需要处理从程序启动以来的所有数据。

**Spark Streaming 也提供了检查点机制，可以把状态阶段性地存储到可靠文件系统中（例如 HDFS 或者 S3） 。**

​	--一般来说，你需要每处理 5-10 个批次的数据就保存一次。在恢复数据时，Spark Streaming 只需要回溯到上一个检查点即可。

### 8-2 转化操作

**DStream 的转化操作可以分为无状态（stateless）和有状态（stateful）两种。**

	• 在无状态转化操作中，每个批次的处理不依赖于之前批次的数据。
		例如 map()、 filter()、 reduceByKey() 等， 都是无状态转化操作。

```
• 相对地，有状态转化操作需要使用之前批次的数据或者是中间结果来计算当前批次的数据。
	有状态转化操作包括基于滑动窗口的转化操作和追踪状态变化的转化操作。
```

#### 无状态转化操作

把简单的 RDD 转化操作<u>应用到每个批次上</u>，也就是**转化 DStream 中的每一个 RDD**。

​	--RDD 转 化 操 作 中 有 不 少 都 可 以 用 于DStream。

​	--针对键值对的 DStream 转化操作（比如 reduceByKey()）要添加 import StreamingContext._ 才能在 Scala 中使用。

![无状态转化操作](/Users/zhangyanbo/Yber/文档/Sp/无状态转化操作.png)

尽管这些函数看起来像作用在整个流上一样，但事实上<u>每个 DStream 在内部是由许多 RDD（批次）组成，且**无状态转化操作是分别应用到每个 RDD 上的**</u>。

​	--reduceByKey() 会归约每个时间区间中的数据，但不会归约不同区间之间的数据。



**无状态转化操作也能在多个 DStream 间整合数据，不过也是<u>在各个时间区间内</u>。**

​	--cogroup()、join()、leftOuterJoin() 等。我们可以在 DStream 上使用这些操作，这样就对每个批次分别执行了对应的 RDD 操作。



**可以使用 StreamingContext.union() 来合并多个流。**



如果这些无状态转化操作不够用，DStream 还提供了一个叫作 **transform() 的高级操作符**，可以让你直接操作其内部的 RDD。这个 transform() 操作**允许你对 DStream 提供任意一个 RDD 到 RDD 的函数。这个函数会在数据流中的每个批次中被调用，生成一个新的流。**

​	--一个常见应用就是重用你为 RDD 写的批处理代码。

```scala
/**
*	例如，如果你有一个叫作 extractOutliers() 的函数，用来从一个日志记录的 RDD 中提取出异常值的RDD（可能通过对消息进行一些统计） ，
*	你就可以在 transform() 中重用它
*/
val outlierDStream = accessLogsDStream.transform { rdd =>
   extractOutliers(rdd) 
}
```

#### 有状态转换操作

**是跨时间区间跟踪数据的操作；也就是说，一些先前批次的数据也被用来在新的批次中计算结果。**



**主要的两种类型：**

<u>滑动窗口</u>和 <u>updateStateByKey()</u>，前者以一个时间阶段为滑动窗口进行操作，后者则用来跟踪每个键的状态变化（例如构建一个代表用户会话的对象） 。



**有状态转化操作需要在你的 StreamingContext 中打开检查点机制来确保容错性。**

```scala
//传递一个目录作为参数给ssc.checkpoint() 来打开它
ssc.checkpoint("hdfs://...")
```



##### 基于窗口的转化操作

会在一个<u>比 StreamingContext 的批次间隔更长的时间范围内，通过整合多个批次的结果，计算出整个窗口的结果</u>。



所有基于窗口的操作**都需要两个参数**，分别为<u>窗口时长</u>以及<u>滑动步长</u>，**两者都必须是StreamContext 的批次间隔的整数倍**。

​	--窗口时长控制每次计算最近的多少个批次的数据，其实就是最近的 windowDuration/batchInterval 个批次。如果有一个以 10 秒为批次间隔的源DStream，要创建一个最近 30 秒的时间窗口（即最近 3 个批次） ，就应当把 windowDuration 设为 30 秒。

​	--而滑动步长的默认值与批次间隔相等，用来控制对新的 DStream 进行计算的间隔。如果源 DStream 批次间隔为 10 秒，并且我们只希望每两个批次计算一次窗口结果，就应该把滑动步长设置为 20 秒。



###### window()

最简单窗口操作是 window()，它**返回一个新的 DStream 来表示所请求的窗口操作的结果数据**。

​	<u>window() 生成的 DStream 中的每个 RDD 会包含多个批次中的数据</u>，可以对这些数据进行 count()、transform() 等操作。

<img src="/Users/zhangyanbo/Yber/文档/Sp/window.png" alt="window" style="zoom:50%;" />

```scala
val accessLogsWindow = accessLogsDStream.window(Seconds(30), Seconds(10)) 
val windowCounts = accessLogsWindow.count()
```

**Spark Streaming 还是提供了一些其他的窗口操作，让用户可以高效而方便地使用。**

首先，reduceByWindow() 和 reduceByKeyAndWindow() 让我们可以对每个窗口更高效地进行归约操作。

###### reduceByWindow() 

###### reduceByKeyAndWindow() 

​	--它们接收一个归约函数，在整个窗口上执行，比如+。

​	--除此之外，它们还有一种特殊形式，通过只考虑进入窗口的数据和离开窗口的数据，让Spark增量计算归约操作。这种特殊形式需要提供归约函数的一个逆函数。

​		--例如+对应的逆函数为-。

​		--对于较大窗口，提供逆函数可以大大提高执行效率。

![reduceByWindow](/Users/zhangyanbo/Yber/文档/Sp/reduceByWindow.png)

```scala
//每个 IP 地址的访问量计数
val ipDStream = accessLogsDStream.map(logEntry => (logEntry.getIpAddress(), 1)) 
val ipCountDStream = ipDStream.reduceByKeyAndWindow(
  {(x, y) => x + y}, // 加上新进入窗口的批次中的元素   
  {(x, y) => x - y}, // 移除离开窗口的老批次中的元素   
  Seconds(30),       // 窗口时长   
  Seconds(10))       // 滑动步长
```

**最后，DStream 还提供了 countByWindow() 和 countByValueAndWindow() 作为对数据进行计数操作的简写。**

###### countByWindow() 

返回一个表示每个窗口中元素个数的 DStream。

######  countByValueAndWindow()

返回的 DStream 则包含窗口中每个值的个数。



##### UpdateStateByKey转化操作

有时，我们需要在 DStream 中跨批次维护状态（例如跟踪用户访问网站的会话） 。

###### updateStateByKey() 

**updateStateByKey() 为我们提供了对一个状态变量的访问，用于键值对形式的DStream。**

<u>	给定一个由（键，事件）对构成的 DStream，并传递一个指定如何根据新的事件更新每个键对应状态的函数</u>，它可以<u>构建出一个新的 DStream，其内部数据为（键，状态）对</u>。

```
例如，在网络服务器日志中，事件可能是对网站的访问，此时键是用户的 ID。使用updateStateByKey() 可以跟踪每个用户最近访问的 10 个页面。这个列表就是“状态”对象，我们会在每个事件到来时更新这个状态。
```

**要使用 updateStateByKey()，提供了一个 update(events, oldState) 函数，接收与某键相关的事件以及该键之前对应的状态，返回这个键对应的新状态。**

​	--events 当前批次接收到的事件列表（可能为空）。

​	--oldState 一个可选的状态对象，存放Option内；如果一个键没有之前的状态，这个值可以空缺。

​	--newState 由函数返回，以Option形式存在；我们可以返回一个空的Option来表示想要删除该状态。

**结果会是一个新的Dstream，内部RDD序列是由每个时间区间内对应的（键，状态）对组成的。**

```scala
def updateRunningSum(values: Seq[Long], state: Option[Long]) = {   
  Some(state.getOrElse(0L) + values.size) 
}  
val responseCodeDStream = accessLogsDStream.map(log => (log.getResponseCode(), 1L)) 
val responseCodeCountDStream = responseCodeDStream.updateStateByKey(updateRunningSum _)
```

### 8-3输出操作

**输出操作指定了对流数据经转化操作得到的数据所要执行的操作。**

​	--例如把结果推入外部数据库或输出到屏幕上。

**与 RDD 中 的 惰 性 求 值 类 似， 如 果 一 个 DStream 及 其 派 生 出 的 DStream 都 没 有 被 执 行 输 出 操 作， 那 么 这 些 DStream 就 都 不 会 被 求 值。** 

​	--如 果StreamingContext 中没有设定输出操作，整个 context 就都不会启动。

​	--调试性输出操作是 print()。会在每个批次中抓取 DStream 的前十个元素打印出来。

**DStream 有与 Spark 类似的 save() 操作，它们接受一个目录作为参数来存储文件，还支持通过可选参数来设置文件的后缀名。**

​	--每个批次的结果被保存在给定目录的子目录中，且文件名中含有时间和后缀名。

```scala
ipAddressRequestCount.saveAsTextFiles("outputDir", "txt")
```

还有一个更为通用的 saveAsHadoopFiles() 函数，接收一个 Hadoop 输出格式作为参数。

```scala
val writableIpAddressRequestCount = ipAddressRequestCount.map {   
  (ip, count) => (new Text(ip), new LongWritable(count)) } 
writableIpAddressRequestCount.saveAsHadoopFiles[   
  SequenceFileOutputFormat[Text, LongWritable]]("outputDir", "txt")
```

**还有一个通用的输出操作 foreachRDD()，它用来对 DStream 中的 RDD 运行任意计算。**

​	--这和 transform() 有些类似，都可以让我们访问任意 RDD。<u>在 foreachRDD() 中，可以重用我们在 Spark 中实现的所有行动操作</u>。

​	--比如，常见的用例之一是把数据写到诸如MySQL 的外部数据库中。对于这种操作，Spark 没有提供对应的 saveAs() 函数，但可以使用 RDD 的 foreachPartition() 方法来把它写出去。为了方便，foreachRDD() 也可以提供给我们当前批次的时间，允许我们把不同时间的输出结果存到不同的位置。

```scala
ipAddressRequestCount.foreachRDD { rdd =>   
  rdd.foreachPartition { partition =>     // 打开到存储系统的连接 （比如一个数据库的连接）      
    partition.foreach { item =>       // 使用连接把item存到系统中     
    }     // 关闭连接   
  } 
}
```

### 8-4 输入源

#### 核心数据源

##### 文件流



##### Akka actor 流



#### 附加数据流

##### Kafka



##### Flume



##### 自定义输入源



#### 多数据源与集群规模

**每个接收器都以 Spark 执行器程序中一个长期运行的任务的形式运行，因此会占据分配给应用的CPU 核心。此外，我们还需要有可用的 CPU 核心来处理数据。**

​	--这意味着如果要运行多个接收器，就必须至少有和接收器数目相同的核心数，还要加上用来完成计算所需要的核心数。

​		--例如，如果我们想要在流计算应用中运行 10 个接收器，那么至少需要为应用分配 11 个 CPU 核心。



### 8-5 24/7不间断运行

#### 检查点机制

**使 Spark Streaming 阶段性地把应用数据存储到诸如 HDFS 或 Amazon S3 这样的可靠存储系统中，以供恢复时使用。**

​	--控制发生失败时需要重算的状态数。

​	--提供驱动器程序容错。

**通过向ssc.checkpoint() 方法传递一个路径参数（HDFS、S3 或者本地路径均可）来配置检查点机制。**

```scala
ssc.checkpoint("hdfs://...")
```

#### 驱动器程序容错

**驱 动 器 程 序 的 容 错 要 求 我 们 以 特 殊 的 方 式 创 建 StreamingContext。 我 们 需 要 把 检 查点 目 录 提 供 给 StreamingContext。** 

​	--与 直 接 调 用 new StreamingContext 不 同， 应 该 <u>使 用StreamingContext.getOrCreate() 函数</u>。

```scala
def createStreamingContext() = {   
  ... 
  val sc = new SparkContext(conf)   
  // 以1秒作为批次大小创建StreamingContext   
  val ssc = new StreamingContext(sc, Seconds(1))   
  ssc.checkpoint(checkpointDir) } 
... 
val ssc = StreamingContext.getOrCreate(checkpointDir, createStreamingContext _)
```

**除了用 getOrCreate() 来实现初始化代码以外，你还需要编写在驱动器程序崩溃时重启驱动器进程的代码。**

​	--Spark 在独立集群管理器中提供了更丰富的支持，可以在<u>提交驱动器程序时使用 --supervise 标记来让 Spark 重启失败的驱动器程序</u>。你还要<u>传递 --deploy-mode cluster 参数来确保驱动器程序在集群中运行</u>，而不是在本地机器上.



## SCALA

### 常用方法

mkString()





Option有两个子类别，Some和None。当程序回传Some的时候，代表这个函式成功地给了你一个String，而你可以透过get()函数拿到那个String，如果程序返回的是None，则代表没有字符串可以给你。
在返回None，也就是没有String给你的时候，如果你还硬要调用get()来取得 String 的话，Scala一样是会抛出一个**NoSuchElementException异常**给你的。
我们也可以选用另外一个方法，getOrElse。这个方法在这个Option是Some的实例时返回对应的值，而在是None的实例时返回传入的参数。换句话说，传入getOrElse的参数实际上是默认返回值。



## Spark官方文档

