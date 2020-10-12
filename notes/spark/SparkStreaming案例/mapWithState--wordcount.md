# 有状态运算符mapWithState

mapWithState它会按照时间线在每一个批次间隔返回之前的发生改变的或者新的key的状态，不发生变化的不返回。

- 有状态的计算wordcount

```scala
package com.yber.hotcenter.test2

import org.apache.spark.streaming.{Seconds, State, StateSpec, StreamingContext}
import org.apache.spark.{SparkConf, SparkContext}

object SparkMapWithState {
  def main(args: Array[String]): Unit = {
    if (args.length < 2) {
      System.err.println("Usage:SparkMapWithState <hostname> <port>")
      System.exit(1)
    }

    //创建StreamingContext对象
    val sparkConf = new SparkConf().setAppName("SparkMapWithState").setMaster("local[3]")
    val sc = new SparkContext(sparkConf)
    val ssc = new StreamingContext(sc, Seconds(5))
		//设置检查点
    ssc.checkpoint(".")
		//初始状态RDD
    val initRDD = ssc.sparkContext.parallelize(List(("hello", 1), ("hi", 1)))

    //从socket接口接受数据
    val lines = ssc.socketTextStream(args(0), args(1).toInt)

    //将数据转化为k,1格式
    val words = lines.flatMap(_.split(" "))
    val wordDstream = words.map(t => (t, 1))

    //定义有状态转化的函数	--1
    //三个参数（k格式,v格式Option,v状态State）
    val mapFunc = (word: String, one: Option[Int], state: State[Int]) => {
      //计算新的v（v当前值【可能不存在Option】+v之前的状态【可能不存在Option】）
      val sum = one.getOrElse(0) + state.getOption().getOrElse(0)
      //更新k的最新状态
      state.update(sum)
      //返回当前k-v
      val output = (word, sum)
      output
    }
    //调用mapWithState，传入定义好的状态转化函数（工厂方法State.function(定义的函数)）；
    //进一步传入了初始状态RDD（initialState(RDD)）
    val stateDstream = wordDstream.mapWithState(StateSpec.function(mapFunc).initialState(initRDD))

    stateDstream.print()
    ssc.start()
    ssc.awaitTermination()

  }
}
```



