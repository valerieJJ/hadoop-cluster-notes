# hadoop-cluster-notes
Mac: hadoop-cluster(3.3.1)+zookeeper+hbase(1.2.6)+spark(3.1.2)

Here are some tips: 

`Hbase-HA.md` notes the process of setting up a highly available hbase-cluster with zookeeper. 

`maven-spark.md` records how to use maven to pack a spark application and submit it to the spark-cluster in yarn-client mode.

`hdfs-neo4j.md` shows a demo to store graph-data in neo4j-database with data loaded from HDFS directly. (Neo4j is a database designed for storing graph-structural data)

`pmml-scala-spark.md` contains the codes that using spark to run a GBDT-model which was pretrained in tensorflow and was saved in `pmml` format.


Note that more codes to operate HDFS&Hbase in Scala is available on this site https://github.com/valerieJJ/RecSys


<img width="1331" alt="image" src="https://user-images.githubusercontent.com/43733497/159135016-59b4a35e-0c67-4323-8fe5-cb0ca19ecefb.png">
