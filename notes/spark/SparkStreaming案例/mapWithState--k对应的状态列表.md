# 有状态运算符mapWithState

mapWithState它会按照时间线在每一个批次间隔返回之前的发生改变的或者新的key的状态，不发生变化的不返回。

- 有状态的计算key对应的value集合

```scala
package com.yber.hotcenter.main

import org.apache.spark.broadcast.Broadcast
import org.apache.spark.streaming._
import org.apache.spark.streaming.dstream.DStream
import org.apache.spark.util.LongAccumulator
import org.apache.spark.{SparkConf, SparkContext}

import scala.collection.immutable.Set

object HotMajorSocket {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf()
      .setAppName("HotMajorSocket")
      .setMaster("local[3]")

    val sc = new SparkContext(conf)
    val ssc = new StreamingContext(sc,Seconds(5))

    val SocketDstream = ssc.socketTextStream("localhost",7777)

    ssc.checkpoint(".")
    //设置一个Set集合，用来存放当天不重复的热搜标题
//    var todayTitle = scala.collection.mutable.Set[String]("hello")
    //将这个热搜标题集合广播出去
//    val todaytitle: Broadcast[Set[String]] = sc.broadcast(todayTitle.toSet)

    val newTitleNumber: LongAccumulator = sc.longAccumulator("newTitleNumber")

    val  titleAndHot1: DStream[String] = SocketDstream
      .flatMap(_.split(" "))

    val titleAndHot2: DStream[(String, Int)] = titleAndHot1.map(t => {
      val value = t.split("\\|")
      (value(0),value(1).toInt)
    }).cache()


    val stateFunc = (key:String,value:Option[Int],state:State[Set[Int]]) =>{
      val previous: Set[Int] = state.getOption().getOrElse(Set.empty[Int])
      val newValues =
        if(value.isDefined) {
          previous.+(value.get)
        }else{
          previous
        }
      val output = (key,newValues)
      state.update(newValues)
      output
    }


    val stateDstream= titleAndHot2.mapWithState(StateSpec.function(stateFunc))

    stateDstream.print()

    ssc.start()
    ssc.awaitTermination()
  }
}

```



