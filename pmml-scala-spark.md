
[https://github.com/jpmml/jpmml-evaluator-spark](https://github.com/jpmml/jpmml-evaluator-spark)

```scala
package scala
import main.scala.IrisHelper
import org.apache.hadoop.shaded.org.eclipse.jetty.websocket.common.frames.DataFrame
import org.apache.spark.mllib.linalg.Vector
import org.jpmml.evaluator.spark.TransformerBuilder;
import java.util.stream.Collectors.toList

// https://blog.csdn.net/weixin_31897613/article/details/112224295

import scala.collection.JavaConversions
import java.util.Arrays
//import org.apache.spark
import org.apache.spark.SparkConf
import org.apache.spark.mllib.linalg.{Vector, Vectors}
import org.apache.spark.sql.SparkSession
import org.jpmml.evaluator.LoadingModelEvaluatorBuilder



object IrisHelper {
  case class InputRecord(
                          `Sepal.Length`:Double,
                          `Sepal.Width`:Double,
                          `Petal.Length`:Double,
                          `Petal.Width`:Double
                        )
}

object SpkJpmml {

  import IrisHelper._
  def main(args: Array[String]): Unit = {
    implicit val sparkSession = SparkSession
      .builder()
      .config(
        new SparkConf()
          .setAppName("GBDT4Iris")
          .setMaster("local")
      ).getOrCreate()

    // prepare the input data
    val inputRdd = sparkSession.sparkContext.makeRDD(Seq(
      InputRecord(5.1, 3.5, 1.4, 0.2),
      InputRecord(5.8, 3.1, 4.8, 1.8),
      InputRecord(4.9, 3, 1.4, 0.2)
    ))
    val inputData = sparkSession.createDataFrame(inputRdd)

    // load the pmml
    val pmml = getClass.getClassLoader.getResourceAsStream("GBDT.pmml")

    //create the evaluator
    val evaluator = new LoadingModelEvaluatorBuilder()
      .load(pmml)
      .build()

    val targetField  =evaluator.getTargetFields.toString
    println(targetField)

    val outputField  =evaluator.getOutputFields.toString
    println(outputField)
    //create the transformer //
    var pmmlTransformer = new TransformerBuilder(evaluator)
      .withTargetCols()
      .withOutputCols()
      .exploded(false) // This is it!!!
      .build()
      
    sparkSession.sql("set spark.sql.legacy.allowUntypedScalaUDF=true")
    var resultDs = pmmlTransformer.transform(inputData)//inputData
    resultDs.show
    
    resultDs = resultDs.select("pmml")
    resultDs.show
  }

}

```
![在这里插入图片描述](https://img-blog.csdnimg.cn/943e8cba304d4cd8966aaa5c7c90cf97.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAVmFsZXJpZUpK,size_20,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/60903e02a0d64761898fc48e05d49539.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAVmFsZXJpZUpK,size_17,color_FFFFFF,t_70,g_se,x_16)
