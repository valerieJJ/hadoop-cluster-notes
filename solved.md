

# 启动顺序
1) zookeepers
```powershell
jj@JJ:~$ zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /home/jj/Downloads/zookeepers/zookeeper00/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
jj@JJ:~/Downloads/zookeepers$ zkServer.sh start zookeeper01/conf/zoo.cfg
ZooKeeper JMX enabled by default
Using config: zookeeper01/conf/zoo.cfg
Starting zookeeper ... STARTED
jj@JJ:~/Downloads/zookeepers$ zkServer.sh start zookeeper02/conf/zoo.cfg
ZooKeeper JMX enabled by default
Using config: zookeeper02/conf/zoo.cfg
Starting zookeeper ... STARTED
```
2) hdfs
```powershell
jj@JJ:~/Downloads/zookeepers$ start-dfs.sh
Starting namenodes on [localhost]
Starting datanodes
Starting secondary namenodes [JJ]
```
3) spark master , workers
```powershell
jj@JJ:~/Downloads/zookeepers$ start-master.sh
starting org.apache.spark.deploy.master.Master, logging to /home/jj/Downloads/spark-3.1.2-bin-hadoop3.2/logs/spark-jj-org.apache.spark.deploy.master.Master-1-JJ.out
jj@JJ:~/Downloads/zookeepers$ start-workers.sh
localhost: starting org.apache.spark.deploy.worker.Worker, logging to /home/jj/Downloads/spark-3.1.2-bin-hadoop3.2/logs/spark-jj-org.apache.spark.deploy.worker.Worker-1-JJ.out
```
4) yarn
```powershell
jj@JJ:~/Downloads/zookeepers$ start-yarn.sh
Starting resourcemanager
Starting nodemanagers
```
5) hbase
```powershell
jj@JJ:~/Downloads/zookeepers$ start-hbase.sh
/home/jj/Downloads/hadoop-3.1.2/libexec/hadoop-functions.sh:
```
6) Web:
- hadoop: http://localhost:8088/cluster
- hbase master: http://localhost:16010/master-status
- namenode: http://localhost:9870/dfshealth.html#tab-overview
- spark master: http://localhost:8001/ (在spark的sbin/start-master.sh中修改了SPARK_MASTER_WEBUI_PORT=8001，否则该端口默认为8080)

# Hbase 伪分布式
## /etc/profile
```powershell
# hbase
export HBASE_HOME=/home/jj/Downloads/hbase-2.2.2
export PATH=$PATH:$HBASE_HOME/bin:/$HBASE_HOME/sbin
```

## hbase-env.sh
```powershell
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HBASE_HOME=/home/jj/Downloads/hbase-2.2.2
#export HBASE_CLASSPATH=/home/jj/Downloads/hbase-2.2.2/lib
export HBASE_CLASSPATH=/home/jj/Downloads/hbase-2.2.2/lib/ruby/*:/home/jj/Downloads/hbase-2.2.2/lib/*
export HBASE_LOG_DIR=${HBASE_HOME}/logs
export HBASE_PID_DIR=${HBASE_HOME}/pids
# 是否使用外部zl，true表示使用自带zk
export HBASE_MANAGES_ZK=false
```

## hbase-2.2.2/conf/hbase-site.xml
```powershell
<configuration>
	<!--独立运行模式,持久化到hdfs实例而非本地文件系统，
	rootdir只想hdfs中实例的目录
	<property>
		<name>hbase.rootdir</name>
		<value>hdfs://localhost:9000</value>
	</property>-->

	<!-- false是单机模式，true是分布式模式  -->
	<property>
		<name>hbase.cluster.distributed</name>
		<value>true</value>
	</property>	
	
	<property>
		<name>hbase.master</name>
		<value>localhost:60000</value>
	</property>
	<property>
		<name>hbase.rootdir</name>
		<value>hdfs://localhost:9000/hbase</value>
	</property>

	<property>
		<name>zookeeper.znode.parent</name>
		<value>/hbase-unsecure</value>
	</property>
	<property>
		<name>hbase.unsafe.stream.capability.enforce</name>
		<value>false</value>
	</property>

	<property>
                <name>hbase.tmp.dir</name>
                <value>/home/jj/Downloads/hbase-2.2.2/tmp</value>
        </property>

	<property>
		<name>hbase.zookeeper.property.dataDir</name>
		<value>/home/jj/Downloads/zookeepers/zookeeper00/dataDir</value>
	</property>
	<property>
        	<name>hbase.zookeeper.property.clientPort</name>
        	<value>2181</value>
	</property>

	<property>
       		<name>hbase.regionserver.info.port</name>
        	<value>16030</value>
	</property>
	<property>
		<name>hbase.master.info.port</name>
		<value>16010</value>
	</property>
	<property>
		<name>hbase.zookeeper.quorum</name>
		<value>localhost</value>
	</property>
</configuration>
```
## hdfs-site.xml
![在这里插入图片描述](https://img-blog.csdnimg.cn/0d940abc19864fe692e525b84c761f31.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDc3MjIwOA==,size_16,color_FFFFFF,t_70)./start-hbase.sh
./stop-hbase.sh
![在这里插入图片描述](https://img-blog.csdnimg.cn/dc94d96d4e194ce8b94b7c111aeb941a.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDc3MjIwOA==,size_16,color_FFFFFF,t_70)HMaster，HRegionServer启动！
http://localhost:16010/master-status
![在这里插入图片描述](https://img-blog.csdnimg.cn/cb59f48505b54fe48a89d7404940e87f.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDc3MjIwOA==,size_16,color_FFFFFF,t_70)


[hbase报错集合](https://blog.csdn.net/pycrossover/article/details/102627807#commentBox)

# hadoop&spark 伪分布式 
## solved:  master, workers未启动
conf/spark-env.sh中，master配置出问题了，改为本地ip地址
(shell command: ifconfig查询本地ip)
```powershell
export SPARK_MASTER_IP=172.16.158.143
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/26cf23208dcb4e6c8a58884ca808a40b.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDc3MjIwOA==,size_16,color_FFFFFF,t_70)
在/etc/profile中添加环境变量![在这里插入图片描述](https://img-blog.csdnimg.cn/398873d8e80644ff832355ce7abd1842.png)
this works： [Spark伪分布式启动只有jps没有Master和Worker](https://www.pianshen.com/article/69331311917/)

```powershell
jj@JJ:~/Downloads/spark-3.1.2-bin-hadoop3.2/sbin$ start-dfs.sh
Starting namenodes on [localhost]
Starting datanodes
Starting secondary namenodes [JJ]
jj@JJ:~/Downloads/spark-3.1.2-bin-hadoop3.2/sbin$ start-all.sh
WARNING: Attempting to start all Apache Hadoop daemons as jj in 10 seconds.
WARNING: This is not a recommended production deployment configuration.
WARNING: Use CTRL-C to abort.
Starting namenodes on [localhost]
localhost: namenode is running as process 212555.  Stop it first.
pdsh@JJ: localhost: ssh exited with exit code 1
Starting datanodes
localhost: datanode is running as process 212750.  Stop it first.
pdsh@JJ: localhost: ssh exited with exit code 1
Starting secondary namenodes [JJ]
JJ: secondarynamenode is running as process 212998.  Stop it first.
pdsh@JJ: JJ: ssh exited with exit code 1
Starting resourcemanager
Starting nodemanagers


# start master
jj@JJ:~/Downloads/spark-3.1.2-bin-hadoop3.2/sbin$ start-master.sh
starting org.apache.spark.deploy.master.Master, logging to /home/jj/Downloads/spark-3.1.2-bin-hadoop3.2/logs/spark-jj-org.apache.spark.deploy.master.Master-1-JJ.out
# start workers
jj@JJ:~/Downloads/spark-3.1.2-bin-hadoop3.2/sbin$ start-workers.sh
localhost: starting org.apache.spark.deploy.worker.Worker, logging to /home/jj/Downloads/spark-3.1.2-bin-hadoop3.2/logs/spark-jj-org.apache.spark.deploy.worker.Worker-1-JJ.out
#jps
jj@JJ:~/Downloads/spark-3.1.2-bin-hadoop3.2/sbin$ jps
214466 Master
212998 SecondaryNameNode
156709 QuorumPeerMain
199467 SparkSubmit
212555 NameNode
156618 QuorumPeerMain
46859 org.eclipse.equinox.launcher_1.1.0.v20100507.jar
212750 DataNode
214675 Worker
156535 QuorumPeerMain
110841 JournalNode
214201 NodeManager
213849 ResourceManager
214812 Jps
jj@JJ:~/Downloads/spark-3.1.2-bin-hadoop3.2/sbin$
```
[spark伪分布式及全分布式部署](https://blog.csdn.net/buyu95/article/details/93743825)
## solved: master, workers正常启动，但是localhost:8080无法访问
8080端口被占用，改一个master的WebUi端口就好
参考 [spark伪分布式搭建及spark页面8080端口访问出错的问题](https://blog.csdn.net/qq_31706571/article/details/78925278)
将 start-master.sh中的 SPARK_MASTER_WEBUI_PORT=8080改为其他端口
```powershell
vim sbin/start-master.sh
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2906d5fcf0b243d28e2800b608f378ce.png)

## solved:  Unable to load native-hadoop library
WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable

solution:
在Hadoop的配置文件core-site.xml中可以设置是否使用本地库：（Hadoop默认的配置为启用本地库）
```powershell
<property>
	<name>hadoop.native.lib</name>
	<value>false</value>
	<description>Should native hadoop libraries.if present, be used.</description>
</property>
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/97f149145760488c8c52062b1e1a372d.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDc3MjIwOA==,size_16,color_FFFFFF,t_70)


# Zookeeper伪分布式 
## 安装 & 配置
官网下载，注意选择文件名含要含有'bin'：***apache-zookeeper-3.5.9-bin.tar.gz***

1. 解压，将其移至我的新建文件夹/home/jj/Downloads/zookeepers，
修改文件名为“zookeeper00”，作为第一个节点

2. 编辑第一个节点myid=1的配置zoo.cfg
```powershell
mkdir zookeeper00/dataDir # 作为zoo1节点的数据存放处
mkdir zookeeper00/dataLogDir  # 作为zoo1节点的日志数据存放处
```
将conf文件夹下的zoo_sample.cfg改名zoo.cfg,
编辑zoo.cfg：

> tickTime=2000
initLimit=10
syncLimit=5
> 
> dataDir=/home/jj/Downloads/zookeepers/zookeeper00/dataDir
> dataLogDir=/home/jj/Downloads/zookeepers/zookeeper00/dataLogDir
> 
> clientPort=2181
> server.1=localhost:2888:3888#注意不要有空格！
> server.2=localhost:2889:3889
> server.3=localhost:2890:3890

3. 在该节点的dataDIr文件夹中，手动添加myid文件，写入该节点id（0~N）
在zookeeper00/dataDir下，vim myid
输入1，保存退出
![在这里插入图片描述](https://img-blog.csdnimg.cn/491e62dffd8e42d6a6d2a60719207c19.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDc3MjIwOA==,size_16,color_FFFFFF,t_70)
4. 同样方法添加其他节点
```powershell
cp zookeeper00 zookeeper01 -rf # node2, myid=2
cp zookeeper00 zookeeper02 -rf # node3, myid=3
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/c382bcd026cd4ec4b9fff5b65fa8408a.png#pic_center)

分别修改/添加他们的cfg和myid
zookeeper01/conf/zoo.cfg中：

> tickTime=2000
initLimit=10
syncLimit=5
> 
> dataDir=/home/jj/Downloads/zookeepers/zookeeper01/dataDir# 修改存放位置
> dataLogDir=/home/jj/Downloads/zookeepers/zookeeper01/dataLogDir# 修改存放位置
> 
> clientPort=2182#必须修改端口
> server.1=localhost:2888:3888#server是全局定义的，不要修改(注意不要有空格！)
> server.2=localhost:2889:3889
> server.3=localhost:2890:3890
>
修改zookeeper01/dataDir/myid，设为2

 修改zookeeper02/conf/zoo.cfg 
> tickTime=2000
> initLimit=10> 
> syncLimit=5
> 
> dataDir=/home/jj/Downloads/zookeepers/zookeeper02/dataDir# 修改存放位置
> dataLogDir=/home/jj/Downloads/zookeepers/zookeeper02/dataLogDir# 修改存放位置
> 
> clientPort=2183#必须修改端口
> server.1=localhost:2888:3888#server是全局定义的，不要修改(注意不要有空格！)
> server.2=localhost:2889:3889
> server.3=localhost:2890:3890

修改zookeeper02/dataDir/myid，设为3

5. 添加环境变量

```powershell
sudo vim /etc/profile
export ZK_HOME=/home/jj/Downloads/zookeepers/zookeeper00
export PATH=$PATH:$ZK_HOME/bin
source /etc/profile
```
就可以直接使用zkServer.sh命令了

6.  运行

```powershell
# zookeeper00 start
jj@JJ:~/Downloads/zookeepers$ zkServer.sh start zookeeper00/conf/zoo.cfg
ZooKeeper JMX enabled by default
Using config: zookeeper00/conf/zoo.cfg
Starting zookeeper ... STARTED
# zookeeper00 status - follower
jj@JJ:~/Downloads/zookeepers$ zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /home/jj/Downloads/zookeepers/zookeeper00/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost. Client SSL: false.
Mode: follower

# zookeeper01 start
jj@JJ:~/Downloads/zookeepers$ zkServer.sh start zookeeper01/conf/zoo.cfg
ZooKeeper JMX enabled by default
Using config: zookeeper01/conf/zoo.cfg
Starting zookeeper ... already running as process 25453.
# zookeeper01 status - 选举为leader
jj@JJ:~/Downloads/zookeepers$ zkServer.sh status zookeeper01/conf/zoo.cfg
ZooKeeper JMX enabled by default
Using config: zookeeper01/conf/zoo.cfg
Client port found: 2182. Client address: localhost. Client SSL: false.
Mode: leader

# zookeeper02 start
jj@JJ:~/Downloads/zookeepers$ zkServer.sh start zookeeper02/conf/zoo.cfg
ZooKeeper JMX enabled by default
Using config: zookeeper02/conf/zoo.cfg
Starting zookeeper ... already running as process 25769.
# zookeeper01 status - follower
jj@JJ:~/Downloads/zookeepers$ zkServer.sh status zookeeper02/conf/zoo.cfg
ZooKeeper JMX enabled by default
Using config: zookeeper02/conf/zoo.cfg
Client port found: 2183. Client address: localhost. Client SSL: false.
Mode: follower

# jps
jj@JJ:~/Downloads/zookeepers$ jps
10240 SecondaryNameNode
10851 NodeManager
10500 ResourceManager
29076 QuorumPeerMain
9958 DataNode
29544 Jps
25769 QuorumPeerMain
9787 NameNode
25453 QuorumPeerMain
jj@JJ:~/Downloads/zookeepers$ 
```
7. 查看端口及进程
clientPort 2181 -- PID 29076
clientPort 2182 -- PID 25453
clientPort 2182 -- PID 25769
![在这里插入图片描述](https://img-blog.csdnimg.cn/bd5399b4e3724fa5a4d5be583cb46fdf.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDc3MjIwOA==,size_16,color_FFFFFF,t_70)

通过zkCli.sh登录server
```powershell
jj@JJ:~/Downloads/zookeepers/zookeeper00/bin$ ./zkCli.sh -server localhost:2181
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/1d5fb2a854294054ab13c99e16589f3e.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDc3MjIwOA==,size_16,color_FFFFFF,t_70)


## * solved: zkServer.sh fail to start
zookeeper的配置文件出现异常，查看配置，发现在第二个服务端口号后多了几个空格（可能是直接复制配置文件的时候），导致zookeeper在解析端口号的时候出现异
常，
1. 先检查日志
![在这里插入图片描述](https://img-blog.csdnimg.cn/4160789a558e498db1f6dce506905a34.png)
报错invalid config
2. 检查zoo.cfg：端口后面多了个空格！！！删掉后，problem solved
![在这里插入图片描述](https://img-blog.csdnimg.cn/28c56249069d4b588b436eb09d54533b.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDc3MjIwOA==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/d34dc88efac545439f04638aca303c4e.png)



# HDFS WebUI
## namenode: localhost:9000, webui报错
![在这里插入图片描述](https://img-blog.csdnimg.cn/6289dbc251d64480a18074ae4874faf9.png) 
solution:
发现是单节点的hadoop的问题，正确为**http://localhost:8088/cluster**
[Hadoop Ports Clarification](https://stackoverflow.com/questions/19938453/hadoop-ports-clarification)
![在这里插入图片描述](https://img-blog.csdnimg.cn/60d5c3b36077486ebbf39733e34a259b.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDc3MjIwOA==,size_16,color_FFFFFF,t_70)
##  http://localhost:50070/ web UI doesn't work
The later version of Hadoop i.e. 3.x.x; the 50070 has changed to 9870. So, type in the browser

**localhost:9870**
![在这里插入图片描述](https://img-blog.csdnimg.cn/b3acaaac5a7a431aadc6bf2438fac115.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDc3MjIwOA==,size_16,color_FFFFFF,t_70)



# pdsh默认设置为rcmd=ssh
1.检查您的pdsh默认rcmd rsh，查看您的pdsh默认rcmd是什么。
2.将pdsh的默认rcmd修改为ssh

```powershell
pdsh -q -w localhost
export PDSH_RCMD_TYPE=ssh
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/a92d8133476740d0bba70e1c4086a72e.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDc3MjIwOA==,size_16,color_FFFFFF,t_70)

直接在profile里面改了省事儿
![在这里插入图片描述](https://img-blog.csdnimg.cn/9c2160442b08454b90d664575b65405c.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDc3MjIwOA==,size_16,color_FFFFFF,t_70)



# 伪分布式 - 配置
vim hadoop-env.sh
![在这里插入图片描述](https://img-blog.csdnimg.cn/f63c80b1534c4cdab6dacd27d3b11d95.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDc3MjIwOA==,size_16,color_FFFFFF,t_70)


vim hdfs-site.xml: 
![在这里插入图片描述](https://img-blog.csdnimg.cn/ec5570de447942b295fff5d0bb45e2d7.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDc3MjIwOA==,size_16,color_FFFFFF,t_70)
vim core-site.xml: 
![在这里插入图片描述](https://img-blog.csdnimg.cn/a329212baae44add9923d2fb77cae601.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDc3MjIwOA==,size_16,color_FFFFFF,t_70)

```powershell
jj@JJ:~$ hadoop version
Hadoop 3.1.2
Source code repository https://github.com/apache/hadoop.git -r 1019dde65bcf12e05ef48ac71e84550d589e5d9a
Compiled by sunilg on 2019-01-29T01:39Z
Compiled with protoc 2.5.0
From source with checksum 64b8bdd4ca6e77cce75a93eb09ab2a9
This command was run using /home/jj/Downloads/hadoop-3.1.2/share/hadoop/common/hadoop-common-3.1.2.jar
# 创建hdfs临时文件存放的目录(core-site.xml，hdfs-site.xml中的配置)
jj@JJ:~$ mkdir -p /home/jj/HadoopFiles/tmp/data
jj@JJ:~$ mkdir -p /home/jj/HadoopFiles/tmp/name
# namenode格式化
jj@JJ:~$ hadoop namenode -format
WARNING: Use of this script to execute namenode is deprecated.
WARNING: Attempting to execute replacement "hdfs namenode" instead.

2021-08-18 22:02:35,045 INFO namenode.NameNode: STARTUP_MSG: 
/************************************************************
STARTUP_MSG: Starting NameNode
STARTUP_MSG:   host = JJ/127.0.1.1
STARTUP_MSG:   args = [-format]
STARTUP_MSG:   version = 3.1.2
STARTUP_MSG:   classpath = /home/jj/Downloads/hadoop-3.1.2/etc/hadoop:/home/jj
。。。。。。。

jj@JJ:~$ start-dfs.sh
Starting namenodes on [localhost]
Starting datanodes
Starting secondary namenodes [JJ]

jj@JJ:~$ start-yarn.sh
Starting resourcemanager
Starting nodemanagers

jj@JJ:~$ jps
74708 SecondaryNameNode
75380 NodeManager
75032 ResourceManager
74459 DataNode
74266 NameNode
75582 Jps

```

![在这里插入图片描述](https://img-blog.csdnimg.cn/65b580f63f5540f98f548ad62aae4930.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/08990e555c1b4aa889c25f3dc38c6821.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/a5a072326e3c4111ab46c6dee719a55f.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDc3MjIwOA==,size_16,color_FFFFFF,t_70)


# sudo vim /etc/profile

```powershell
# miniconda3
export PATH=$PATH:/home/jj/miniconda3/bin
# java
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

# spark
export SPARK_HOME=/home/jj/Downloads/spark-3.1.2-bin-hadoop3.2
export PATH=$PATH:$SPARK_HOME/bin
export PYSPARK_PYTHON=/usr/bin/python
export PATH=$PATH:$SPARK_HOME/sbin

# hadoop 
export HADOOP_HOME=/home/jj/Downloads/hadoop-3.1.2
export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:
# export PATH=${JAVA_HOME}/bin:${HADOOP_HOME}/bin:$PATH

```


# * Spark Standalone
## start-master.sh
```powershell
jj@JJ:~/Desktop$ start-master.sh
starting org.apache.spark.deploy.master.Master, logging to /home/jj/Downloads/spark-3.1.2-bin-hadoop3.2/logs/spark-jj-org.apache.spark.deploy.master.Master-1-JJ.out
```
重复运行start-master.sh，给出进程id：
![在这里插入图片描述](https://img-blog.csdnimg.cn/24479e6bcbad48d78b1c1f2b48a2a407.png)

url地址栏： JJ:8080
可以对应的找到占用的端口，然后打开网页：
![在这里插入图片描述](https://img-blog.csdnimg.cn/25a5a080739d48d3a7de42e86235a66e.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDc3MjIwOA==,size_16,color_FFFFFF,t_70)

```powershell
jj@JJ:~/Desktop$ netstat -nap | grep processId
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/942cf297cbe44d46a7358241cf7649cf.png)
## start-worker.sh spark://localhost:port

```powershell
jj@JJ:~/Desktop$ start-master.sh
org.apache.spark.deploy.master.Master running as process 32696.  Stop it first.

jj@JJ:~/Desktop$ start-worker.sh
Usage: ./sbin/start-worker.sh <master> [options]
21/08/18 13:33:16 INFO SignalUtils: Registering signal handler for TERM
21/08/18 13:33:16 INFO SignalUtils: Registering signal handler for HUP
21/08/18 13:33:16 INFO SignalUtils: Registering signal handler for INT
21/08/18 13:33:16 WARN Utils: Your hostname, JJ resolves to a loopback address: 127.0.1.1; using 172.16.158.143 instead (on interface eno1)
21/08/18 13:33:16 WARN Utils: Set SPARK_LOCAL_IP if you need to bind to another address

Master must be a URL of the form spark://hostname:port

Options:
  -c CORES, --cores CORES  Number of cores to use
  -m MEM, --memory MEM     Amount of memory to use (e.g. 1000M, 2G)
  -d DIR, --work-dir DIR   Directory to run apps in (default: SPARK_HOME/work)
  -i HOST, --ip IP         Hostname to listen on (deprecated, please use --host or -h)
  -h HOST, --host HOST     Hostname to listen on
  -p PORT, --port PORT     Port to listen on (default: random)
  --webui-port PORT        Port for web UI (default: 8081)
  --properties-file FILE   Path to a custom Spark properties file.
                           Default is conf/spark-defaults.conf.
jj@JJ:~/Desktop$ start-worker.sh spark://JJ:7077
starting org.apache.spark.deploy.worker.Worker, logging to /home/jj/Downloads/spark-3.1.2-bin-hadoop3.2/logs/spark-jj-org.apache.spark.deploy.worker.Worker-1-JJ.out

```

刷新网页jj:8080，Alive Worker加1
![在这里插入图片描述](https://img-blog.csdnimg.cn/111674c6e258495cbaded168d8cec427.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDc3MjIwOA==,size_16,color_FFFFFF,t_70)

关掉7077这个worker，后，换个worker (spark://JJ:8888 )试试：

```powershell
jj@JJ:~/Desktop$ start-worker.sh spark://JJ:7077
starting org.apache.spark.deploy.worker.Worker, logging to /home/jj/Downloads/spark-3.1.2-bin-hadoop3.2/logs/spark-jj-org.apache.spark.deploy.worker.Worker-1-JJ.out
jj@JJ:~/Desktop$ start-worker.sh spark://JJ:8888
org.apache.spark.deploy.worker.Worker running as process 36526.  Stop it first.
jj@JJ:~/Desktop$ stop-worker.sh
stopping org.apache.spark.deploy.worker.Worker

# 然后重新开一个worker：spark://JJ:8888
jj@JJ:~/Desktop$ start-worker.sh spark://JJ:8888
starting org.apache.spark.deploy.worker.Worker, logging to /home/jj/Downloads/spark-3.1.2-bin-hadoop3.2/logs/spark-jj-org.apache.spark.deploy.worker.Worker-1-JJ.out
# 拿到进行id
jj@JJ:~/Desktop$ start-worker.sh spark://JJ:8888
org.apache.spark.deploy.worker.Worker running as process 36929.  Stop it first.
# 用进程id查占用的端口
jj@JJ:~/Desktop$ netstat -nap | grep 36929
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp6       0      0 :::8081                 :::*                    LISTEN      36929/java          
tcp6       0      0 172.16.158.143:42685    :::*                    LISTEN      36929/java          
unix  2      [ ]         STREAM     CONNECTED     531252   36929/java           
unix  2      [ ]         STREAM     CONNECTED     531284   36929/java 

```

url地址栏输入jj:8081，
![在这里插入图片描述](https://img-blog.csdnimg.cn/f7f4b48fc0a74c198ec9911e4102115a.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDc3MjIwOA==,size_16,color_FFFFFF,t_70)
url地址栏再打开jj:8080，显示这个worker状态为DEAD
![在这里插入图片描述](https://img-blog.csdnimg.cn/48bfa700a64f46cda9030820f72904f5.png)
把这个jj:8888worker停掉，再开7077worker，打开jj:8080网页发现：
![在这里插入图片描述](https://img-blog.csdnimg.cn/d85ec6ce7774450c9056ec15a77fd169.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDc3MjIwOA==,size_16,color_FFFFFF,t_70)
<font color=#AAgre size=4>**大概明白了，莫非standalone就是只能有一个worker是ALIVE，而且这个worker的url是master默认的，spark://:JJ:7077**

## stop-master.sh & stop-worker.sh
# * pyspark.SparkContext读取文件
```python
class pyspark.SparkContext (
   master = None,
   appName = None, 
   sparkHome = None, 
   pyFiles = None, 
   environment = None, 
   batchSize = 0, 
   serializer = PickleSerializer(), 
   conf = None, 
   gateway = None, 
   jsc = None, 
   profiler_cls = <class 'pyspark.profiler.BasicProfiler'>
)

# init:
from pyspark import SparkContext
sc = SparkContext("local", "First App")
# dara = sc.textFile("hdfs://...")
dara = sc.textFile("home/jj/...")
```
