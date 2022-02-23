[官方手册 spark core](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.SparkContext.addFile.html)
[官方手册 Apache - Spark,Hadoop,Kafka,Hbase](https://docs.microsoft.com/zh-cn/azure/hdinsight/spark/apache-spark-create-standalone-application)
```scala
package main.scala
import org.apache.spark
import org.apache.spark.{SparkConf, SparkContext, SparkFiles}
//import org.apache.spark.ml.regression.{LinearRegression, LinearRegressionModel}
//import org.apache.spark.ml.linalg.{Vector, Vectors}
//import org.apache.spark.sql._
//import org.apache.spark.sql.SparkSession
//import org.apache.hadoop

import scala.io.Source
//Exception in thread "main" java.lang.NoSuchMethodError: scala.util.matching.Regex
object jjj {
  def main(args: Array[String]): Unit = {

//    val spark = SparkSession
//      .builder()
//      .appName("WordCount")
//      .master("local[*]")
//      .enableHiveSupport()
//      .getOrCreate()

    val conf = new SparkConf()
    conf.setAppName("WordCount")
    conf.setMaster("local[*]")
    val sc = new SparkContext(conf)
//    val jjfile = sc.textFile("/jjProj/words.txt") //hdfs://192.168.43.99:9000
    val files = "hdfs://jidafus-MBP:9000/jjProj/words.txt"
    sc.addFile(files)
    val path = SparkFiles.get("words.txt")
    println(path)
    val source = Source.fromFile(path)
    val lineIterator = source.getLines
    val lines =lineIterator.toArray
    println("\n\n============= 1 =============")
    println(lines.mkString(","))
    println("============= 2 =============")
    val textFileRDD = sc.textFile("hdfs://jidafus-MBP:9000/jjProj/words.txt")
    val wordRDD = textFileRDD.flatMap(line => line.split(" "))
    println("============= 3 =============")
    val pairWordRDD = wordRDD.map(word => (word, 1))
    println(pairWordRDD)
    println("============= 4 =============")
    val wordCountRDD = pairWordRDD.reduceByKey((a, b) => a + b)
    println("\n\n")
    wordCountRDD.saveAsTextFile("hdfs://jidafus-MBP:9000/jjProj/wordcount3")
  }
}


```
![在这里插入图片描述](https://img-blog.csdnimg.cn/6a34713cf97f4c03bba7c5fa80f6006c.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/e222a19cbd9047f3a6e7d441b3b4ae34.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/629e91336f104f0ca689bf3bc49875d6.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)


# maven打包，spark-submit
![在这里插入图片描述](https://img-blog.csdnimg.cn/92474f94a1134b1fb3f3722f26e48d42.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2ca570fe8a4b4b299e1eacc888fe883b.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)


删除已存在的result.txt
![在这里插入图片描述](https://img-blog.csdnimg.cn/dce7c75a0a9742ff8ba7c1f57394c84d.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)
提交jar包

1）spark集群的standalone模式提交
```bash
(base) jj@JJ-laptop ~ % spark-submit --master spark://master:7077 /Users/jj/IdeaProjects/untitled/JJspark/packages/myJJspark_jar.jar
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/1257926509e44e82afdc652370355c97.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)

2）提交至yarn-client
![在这里插入图片描述](https://img-blog.csdnimg.cn/bef597c9a4ca47b0aba8d9225628b6a0.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAVmFsZXJpZUpK,size_20,color_FFFFFF,t_70,g_se,x_16)


```bash
spark-submit --master yarn --deploy-mode client --class main.scala.jjj /Users/jj/IdeaProjects/untitled/JJspark/packages/thisisitonyarn.jar

```
![在这里插入图片描述](https://img-blog.csdnimg.cn/adacdd0785d440efad23920a477ee186.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAVmFsZXJpZUpK,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/878ab25bc3b84b17b7a4dae71699b8ff.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAVmFsZXJpZUpK,size_20,color_FFFFFF,t_70,g_se,x_16)

# SparkStream

```bash
nc -lk 9999
```

先打开9999端口监听，向9999端口输入数据
![在这里插入图片描述](https://img-blog.csdnimg.cn/645ef33f676d48709b7b9f0cee28026e.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAVmFsZXJpZUpK,size_10,color_FFFFFF,t_70,g_se,x_16)
然后运行程序
1）在IDEA中本地运行测试

```scala
package main.streams

import org.apache.spark.SparkConf
import org.apache.spark.streaming.{Seconds, StreamingContext}

object exmStreaming {

  def main(args: Array[String]): Unit = {
    val conf = new SparkConf()
    conf.setAppName("test-socketstream")
    conf.setMaster("local[*]")
    val ssc = new StreamingContext(conf,Seconds(10))
    // Create a DStream that will connect to hostname:port, like localhost:9999
    val stream = ssc.socketTextStream("master", 9999) // 开启端口 nc -lk 9999
    val words = stream.flatMap(_.split(" ")).map((_, 1)).reduceByKey(_ + _).print()
    words.print()
    ssc.start()
    ssc.awaitTermination()
  }

}

```
代码结构
![在这里插入图片描述](https://img-blog.csdnimg.cn/5b0de9252e4341dba895ebfb984eec1c.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAVmFsZXJpZUpK,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/6cf776df62bf4420b967b365ee6d9ebe.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAVmFsZXJpZUpK,size_20,color_FFFFFF,t_70,g_se,x_16)

2）注释掉“conf.setMaster("local[*]")”，Build Artifacts -> clean -> build，打包提交至yarn

![在这里插入图片描述](https://img-blog.csdnimg.cn/7da9d3a20f3747f28e190660d4a02286.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAVmFsZXJpZUpK,size_20,color_FFFFFF,t_70,g_se,x_16)

```bash
(base) jj@master ~ % spark-submit --master yarn  --deploy-mode client --class main.streams.exmStreaming /Users/jj/IdeaProjects/untitled/mvnProj/out/artifacts/teststreamonyarn/teststream—onyarn.jar
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/99ae44e2640f444ebbe819f76eb158b4.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAVmFsZXJpZUpK,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/0bc938bcde714f2cbe4ac924fc2ad7d0.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAVmFsZXJpZUpK,size_20,color_FFFFFF,t_70,g_se,x_16)

master:4040查看jobs
![在这里插入图片描述](https://img-blog.csdnimg.cn/ec9fe732277544ada21f22235d70fc12.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAVmFsZXJpZUpK,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/66f8d3dd0b944f3ca518ab4f74ad9610.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAVmFsZXJpZUpK,size_20,color_FFFFFF,t_70,g_se,x_16)

# 记一个maven打包报错：找不到主类
解决：检查Build Artifacts 中的设置，
![在这里插入图片描述](https://img-blog.csdnimg.cn/d57c09a90b9849eda950ab81fd5da84b.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAVmFsZXJpZUpK,size_20,color_FFFFFF,t_70,g_se,x_16)
问题出在 **Module --> Dependencies --> Module SDK
这里的Module SDK，默认是Project JDK1.8，改为自己安装的jdk8('1.8.0_301')就好了**
![在这里插入图片描述](https://img-blog.csdnimg.cn/850e69378e7f4c969e11b26293c7f17c.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAVmFsZXJpZUpK,size_20,color_FFFFFF,t_70,g_se,x_16)
下面这样是对的
![在这里插入图片描述](https://img-blog.csdnimg.cn/e541c06c52ee44c8946bdb0191e00640.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAVmFsZXJpZUpK,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/1fe66f3433794ed48c9cce63ea98bdac.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAVmFsZXJpZUpK,size_20,color_FFFFFF,t_70,g_se,x_16)
src包也设置为Source Folders根目录了，没毛病
![在这里插入图片描述](https://img-blog.csdnimg.cn/9d3a6352d5734bc4aab0ffd709a09d65.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAVmFsZXJpZUpK,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/92f69fe19f62445a91ccfc1d02b06c8e.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAVmFsZXJpZUpK,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/92ea35503a044fefadeea5242d3b279d.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAVmFsZXJpZUpK,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/59bb241b46604adea9077015063bcea3.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAVmFsZXJpZUpK,size_20,color_FFFFFF,t_70,g_se,x_16)
