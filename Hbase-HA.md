# Hbase高可用的玩法

（注：此处backup-master我只设置了一个节点作为备用master。

也可以设置多个backup-master，则master挂掉后，zookeeper会从这些backup-master中选举一个master。
但要注意前提是zk集群节点数量>=3。

zk成功选举Leader必须要备节点过半,2n和2n-1（n>1）的容错数都是 n-1 

根据Leader选举“过半法则”，`可用节点数量>总节点数量/2`，其实偶数个节点也是可以的：

- 3节点zookeeper，一个主节点挂了，另外两个备节点过半，顺利选出Leader对外提供集群服务，容错数为1）
- 5节点zookeeper，两个主节点挂了，另外三个备节点过半，对外提供集群服务，容错数为2
- 6节点zookeeper，两个主节点挂了，另外四个备节点过半，对外提供集群服务，容错数为2，


主节点master上

```bash
start-hbase.sh
```
开启第二个从节点hadoop0:

```bash
hbase-daemon.sh start regionserver
```

这时候还不是高可用，因为集群中只有一个HMaster
Backup master = 0
这时如果主节点master挂掉了，Hbase不可用

**高可用模式：**
在hadoop0上也开启Hmaster：
```bash
hbase-daemon.sh start master
```
这样集群中就有了两个HMaster（状态不同，hadoop0是备用的backup master）

主节点master上查看http://master:16010/   显示为Master，
从节点hadoop0上也打开http://local:16010/ 显示为Backup Master
![在这里插入图片描述](https://img-blog.csdnimg.cn/1d27d20f08bc41d28f8e046445fcf9ad.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAVmFsZXJpZUpK,size_20,color_FFFFFF,t_70,g_se,x_16)
主节点master挂掉后，从节点hadoop0变为工作状态的Hmaster：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2cacb0aa0fe74ebb84b462492fe946ad.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAVmFsZXJpZUpK,size_20,color_FFFFFF,t_70,g_se,x_16)
