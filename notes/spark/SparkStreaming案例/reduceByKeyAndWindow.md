# 滑动窗口应用reduceByKeyAndWindow

每隔10秒钟，统计最近60秒钟的搜索词的搜索频次，并打印出**排名最靠前的3个搜索词以及出现次数**

```scala
package com.yber.hotcenter.test

import org.apache.spark.rdd.RDD
import org.apache.spark.streaming.dstream.DStream
import org.apache.spark.streaming.{Seconds, StreamingContext}
import org.apache.spark.{SparkConf, SparkContext}

object SparkStreaming_top3 {
  def main(args: Array[String]): Unit = {
    val conf: SparkConf = new SparkConf().setAppName("SparkStreaming_top3").setMaster("local[2]")
    val sc = new SparkContext(conf)
    val ssc = new StreamingContext(sc,Seconds(10))

    ssc.checkpoint("/Users/zhangyanbo/apps/temp_date")



    val socketDStream = ssc.socketTextStream("localhost",7777)

    val wordsRDD: DStream[String] = socketDStream.map(logs => {
      logs.split(" ")(1)
    })

    val pairWordsDStream = wordsRDD.map(t => (t,1))

    val wordCountsDStream: DStream[(String, Int)] = pairWordsDStream.reduceByKeyAndWindow(_+_,_-_,Seconds(30),Seconds(10))

    val finalDStream: DStream[(String, Int)] = wordCountsDStream.transform(wordCountRDD => {
      val v1: RDD[(Int, String)] = wordCountRDD.map(t => (t._2, t._1))
      val v2: RDD[(Int, String)] = v1.sortByKey(false)
      val v3: RDD[(String, Int)] = v2.map(t => (t._2, t._1))
      val topRDD: Array[(String, Int)] = v3.take(3)

      for (value <- topRDD) {
        println("---" + value + "---")
      }
      wordCountRDD
    })

    finalDStream.print

    ssc.start()
    ssc.awaitTermination()



  }

}

```

