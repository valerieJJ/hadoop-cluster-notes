# Hbase spark Scala读写

1）

```scala
package main.jjhbase

import org.apache.hadoop.hbase.io.ImmutableBytesWritable
import org.apache.hadoop.hbase.{HBaseConfiguration, TableName}
import org.apache.hadoop.hbase.client.{ConnectionFactory, Scan, Put,Result}
import org.apache.hadoop.hbase.mapreduce.TableInputFormat
import org.apache.hadoop.hbase.util.Bytes
import org.apache.hadoop.hbase._
import org.apache.spark._

object sparkHbase {
    val spconf = new SparkConf().setAppName("test-hbase").setMaster("local[*]")
    val sc = new SparkContext(spconf)

  val hbconf = HBaseConfiguration.create()
  hbconf.set("hbase.zookeeper.property.clientPort", "2181")
  hbconf.set("hbase.zookeeper.quorum", "master,hadoop0")
  hbconf.set(TableInputFormat.INPUT_TABLE, "cora")

  val connection = ConnectionFactory.createConnection(hbconf);
  val admin = connection.getAdmin();


  def scanDataFromHTable(tableName:String,columnFamily: String, column: String) = {
//    val TabName = TableName.valueOf("cora")
    val table = connection.getTable(TableName.valueOf(tableName))
    val scan = new Scan()

    scan.addFamily(columnFamily.getBytes()) //添加列簇名称
    val scanner = table.getScanner(scan) //从table中抓取数据来scan
    var result = scanner.next()
    //数据不为空时输出数据
    while (result != null) {
      println(s"rowkey:${Bytes.toString(result.getRow)},列簇:${columnFamily}:${column},value:${Bytes.toString(result.getValue(Bytes.toBytes(columnFamily), Bytes.toBytes(column)))}")
      result = scanner.next()
    }
    //通过scan取完数据后，记得要关闭ResultScanner，否则RegionServer可能会出现问题(对应的Server资源无法释放)
    scanner.close()
  }

  def main(args: Array[String]): Unit = {
    val tables = admin.listTables()
    tables.foreach(println)

//    scanDataFromHTable(tableName = "cora", columnFamily = "author", column = "name")

    // read from hbase 
    val hbaseRDD = sc.newAPIHadoopRDD(hbconf,
                                        classOf[TableInputFormat],
                                        classOf[ImmutableBytesWritable],
                                        classOf[Result]
                                      )
    hbaseRDD.foreach{ case (_, result) =>
      val key = Bytes.toString(result.getRow)
      val a_name = Bytes.toString(result.getValue("author".getBytes,"name".getBytes))
      val a_age = Bytes.toString(result.getValue("author".getBytes,"age".getBytes))
      val a_institution = Bytes.toString(result.getValue("author".getBytes,"institution".getBytes))
      val p_title = Bytes.toString(result.getValue("paper".getBytes,"title".getBytes))
      val p_conference = Bytes.toString(result.getValue("paper".getBytes,"conference".getBytes))
      println("*** data row-authorId:"+ key
                  +", name:" + a_name
                  + ", age:" + a_age
                  + ", institution:" + a_institution
              + "; paper-title:" + p_title
              + ", paper-conference:" + p_conference
            )
    }

  // write to hbase
    val table = connection.getTable(TableName.valueOf("cora"))
    val p = new Put(Bytes.toBytes("004"));
    p.add(Bytes.toBytes("author"), Bytes.toBytes("name"), Bytes.toBytes("Peter"));
    p.add(Bytes.toBytes("author"), Bytes.toBytes("age"), Bytes.toBytes("30"));
    p.add(Bytes.toBytes("author"), Bytes.toBytes("institution"), Bytes.toBytes("Yale"));
    p.add(Bytes.toBytes("paper"), Bytes.toBytes("title"), Bytes.toBytes("paper4"));
    p.add(Bytes.toBytes("paper"), Bytes.toBytes("conference"), Bytes.toBytes("IJCAI"));
    table.put(p)
    println("new date added!")


    admin.close()
    
  }

}

```
![在这里插入图片描述](https://img-blog.csdnimg.cn/188280e4c0e848d9b64d5fa93fcf6b95.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAVmFsZXJpZUpK,size_20,color_FFFFFF,t_70,g_se,x_16)


2） 
```scala
package main.jjhbase

import org.apache.hadoop.hbase.io.ImmutableBytesWritable
import org.apache.hadoop.hbase.{HBaseConfiguration, TableName}
import org.apache.hadoop.hbase.client.{ConnectionFactory, Scan, Put,Result}
import org.apache.hadoop.hbase.mapreduce.TableInputFormat
import org.apache.hadoop.hbase.util.Bytes
import org.apache.hadoop.hbase._
import org.apache.spark._

object sparkHbase {
    val spconf = new SparkConf().setAppName("test-hbase").setMaster("local[*]")
    val sc = new SparkContext(spconf)

  val hbconf = HBaseConfiguration.create()
  hbconf.set("hbase.zookeeper.property.clientPort", "2181")
  hbconf.set("hbase.zookeeper.quorum", "master,hadoop0")
  hbconf.set(TableInputFormat.INPUT_TABLE, "cora")

  val connection = ConnectionFactory.createConnection(hbconf);
  val admin = connection.getAdmin();


  def scanDataFromHTable(tableName:String,columnFamily: String, column: String) = {
//    val TabName = TableName.valueOf("cora")
    val table = connection.getTable(TableName.valueOf(tableName))
    val scan = new Scan()

    scan.addFamily(columnFamily.getBytes()) //添加列簇名称
    val scanner = table.getScanner(scan) //从table中抓取数据来scan
    var result = scanner.next()
    //数据不为空时输出数据
    while (result != null) {
      println(s"rowkey:${Bytes.toString(result.getRow)},列簇:${columnFamily}:${column},value:${Bytes.toString(result.getValue(Bytes.toBytes(columnFamily), Bytes.toBytes(column)))}")
      result = scanner.next()
    }
    //通过scan取完数据后，记得要关闭ResultScanner，否则RegionServer可能会出现问题(对应的Server资源无法释放)
    scanner.close()
  }

  def main(args: Array[String]): Unit = {
    val tables = admin.listTables()
    tables.foreach(println)
//  法一
//    scanDataFromHTable(tableName = "cora", columnFamily = "author", column = "name")

//  法二
    val hbaseRDD = sc.newAPIHadoopRDD(hbconf,
                                        classOf[TableInputFormat],
                                        classOf[ImmutableBytesWritable],
                                        classOf[Result]
                                      )

    hbaseRDD.foreach{ case (_, result) =>
      val key = Bytes.toString(result.getRow)
      val a_name = Bytes.toString(result.getValue("author".getBytes,"name".getBytes))
      val a_age = Bytes.toString(result.getValue("author".getBytes,"age".getBytes))
      val a_institution = Bytes.toString(result.getValue("author".getBytes,"institution".getBytes))
      val p_title = Bytes.toString(result.getValue("paper".getBytes,"title".getBytes))
      val p_conference = Bytes.toString(result.getValue("paper".getBytes,"conference".getBytes))
      println("***data row-authorId:"+ key
                  +", name:" + a_name
                  + ", age:" + a_age
                  + ", institution:" + a_institution
              + "; paper-title:" + p_title
              + ", paper-conference:" + p_conference
            )
    }

    admin.close()


    /*
    * java
    * import org.apache.hadoop.hbase.util.Bytes
      val list: Nothing = new Nothing(FilterList.Operator.MUST_PASS_ONE)
      val filter1: Nothing = new Nothing(cf, column, CompareOperator.EQUAL, Bytes.toBytes("my value"))
      list.add(filter1)
      val filter2: Nothing = new Nothing(cf, column, CompareOperator.EQUAL, Bytes.toBytes("my other value"))
      list.add(filter2)
    * */

    /*
    * scala
    * FilterList list = new FilterList(FilterList.Operator.MUST_PASS_ONE);
      SingleColumnValueFilter filter1 = new SingleColumnValueFilter(
        cf,
        column,
        CompareOperator.EQUAL,
        Bytes.toBytes("my value")
        );
      list.add(filter1);
      SingleColumnValueFilter filter2 = new SingleColumnValueFilter(
        cf,
        column,
        CompareOperator.EQUAL,
        Bytes.toBytes("my other value")
        );
      list.add(filter2);
      scan.setFilter(list);
    *
    * */
  }

}

```
