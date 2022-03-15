# hadoop-cluster-notes
Mac: hadoop-cluster(3.3.1)+zookeeper+hbase(1.2.6)+spark(3.1.2)

Here are some tips: 

`maven-spark.md` records how to use maven to pack a spark application and submit it to the spark-cluster in yarn-client mode.

`hdfs-neo4j.md` shows how to store graph-data in neo4j-database with data loaded from HDFS directly. (Neo4j is a database designed for storing graph-structural data)

`pmml-scala-spark.md` contains the codes that using spark to run a GBDT-model which was pretrained in tensorflow and was saved in `pmml` format.

Note that more codes to operate HDFS&Hbase in Scala is available on this site https://github.com/valerieJJ/RecSys
