[hadoop官方使用手册](https://hadoop.apache.org/docs/r1.0.4/cn/cluster_setup.html#%E9%85%8D%E7%BD%AE)

# 免密登陆 
```powershell
(base) jj@jidafus-MBP ~ % /usr/libexec/java_home -V

(base) jj@jidafus-MBP ~ % ifconfig en0

            
(base) jj@jidafus-MBP ~ % nano ~/.zshenv
(base) jj@jidafus-MBP ~ % source ~/.zshenv

(base) jj@jidafus-MBP ~ % echo $JAVA_HOME
/Library/Internet Plug-Ins/JavaAppletPlugin.plugin/Contents/Home
(base) jj@jidafus-MBP ~ % java -version
java version "1.8.0_301"
Java(TM) SE Runtime Environment (build 1.8.0_301-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.301-b09, mixed mode)
```



```powershell
========== 我是分界线 ========
Last login: Wed Sep 15 11:35:12 from ::1
(base) jj@jidafus-MBP ~ % eval `ssh-agent -s`
Agent pid 11434
(base) jj@jidafus-MBP ~ % cd .ssh
(base) jj@jidafus-MBP .ssh % ls
authorized_keys	id_rsa.pub	jjkey.pub
id_rsa		jjkey		known_hosts
(base) jj@jidafus-MBP .ssh % cd ..
(base) jj@jidafus-MBP ~ % ssh-add 
Enter passphrase for /Users/jj/.ssh/id_rsa: 
Identity added: /Users/jj/.ssh/id_rsa (jj@jidafus-MBP)
(base) jj@jidafus-MBP ~ % ssh-add -K id_rsa
id_rsa: No such file or directory
(base) jj@jidafus-MBP ~ % ssh-add -K .ssh/id_rsa
Enter passphrase for .ssh/id_rsa: 
Identity added: .ssh/id_rsa (jj@jidafus-MBP)
(base) jj@jidafus-MBP ~ % ssh localhost
Last login: Wed Sep 15 11:48:46 2021

(base) jj@jidafus-MBP ~ % cd .ssh
(base) jj@jidafus-MBP .ssh % ls
authorized_keys	id_rsa.pub	jjkey.pub
id_rsa		jjkey		known_hosts

远程登陆及退出
Last login: Wed Sep 15 15:13:37 on ttys000
(base) jj@jidafus-MacBook-Pro ~ % ssh localhost
Last login: Wed Sep 15 15:13:52 2021
(base) jj@jidafus-MacBook-Pro ~ % exit
Connection to localhost closed.

将Mac上的authorized_keys上传到节点中：
base) jj@jidafus-MBP .ssh % scp authorized_keys worker0@192.168.43.158:/home/worker0/.ssh/
worker0@192.168.43.158's password: 
authorized_keys   
```

主机 jj 免密登陆 worker0 
![在这里插入图片描述](https://img-blog.csdnimg.cn/d15601b4b00e4704ad10c461b502df72.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)
 
Worker0 虚拟机免密登陆 主机 jj
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/0cfb7af82bbc4631904192f82dd4fb8b.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)

# zookeeper 
attention: 
/etc/hosts中，将127.0.1.1      hadoop0 注释掉，不然就会报错不能连接到其他zookeeper节点

```xml
admin.enableServer=false
clientPort=2181
dataDir=/Users/jj/zookeepers/zk1/data
dataLogDir=/Users/jj/zookeepers/zk1/log
server.1=master:2888:3888
server.2=hadoop0:2888:3888
server.3=hadoop1:2888:3888
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/8e3f4fba0ec447d7a155f5144c7bd564.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)


![在这里插入图片描述](https://img-blog.csdnimg.cn/48ecf131ac3b49f0a16be40276f1c359.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)

新增的从节点上，启动datanode和yarn的nodemanager

```powershell
hdfs --daemon start datanode
yarn --daemon start nodemanager
```

http://192.168.43.99:8099/cluster/nodes
![在这里插入图片描述](https://img-blog.csdnimg.cn/1a1a00035ee14b9a951a7aae7e3c432b.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)
注意因为在主节点的yarn-site.xml中，设置的yarn resoucemanager webui用的是外网ip：192.168.43.99:8099，输入localhost:8099的话是打不开网页的
![在这里插入图片描述](https://img-blog.csdnimg.cn/ed30e53e86564f62879eee2cc628dfb3.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)

第二台ubuntu (第三个datanode)：
从hadoop0传过来hadoop文件，用scp命令远程传输，或者在资源管理器上远程登录，用图形化界面复制传输。
```powershell
worker1@hadoop1:~/.ssh$ ssh localhost
worker1@localhost: Permission denied (publickey).

worker1@hadoop1:~/.ssh$ chmod 700 ~/.ssh
worker1@hadoop1:~/.ssh$ chmod 600 ~/.ssh/authorized_keys
worker1@hadoop1:~/.ssh$ cat id_rsa.pub >> authorized_keys
worker1@hadoop1:~/.ssh$ ssh localhost
Enter passphrase for key '/home/worker1/.ssh/id_rsa': 
worker1@localhost: Permission denied (publickey).
worker1@hadoop1:~/.ssh$ 
worker1@hadoop1:~/.ssh$ ssh-add
Could not open a connection to your authentication agent.
worker1@hadoop1:~/.ssh$ ssh-agent bash
worker1@hadoop1:~/.ssh$ vim authorized_keys
worker1@hadoop1:~/.ssh$ ssh-add
Enter passphrase for /home/worker1/.ssh/id_rsa: 
Identity added: /home/worker1/.ssh/id_rsa (worker1@hadoop1)
worker1@hadoop1:~/.ssh$ service sshd restart
==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ===
Authentication is required to restart 'ssh.service'.
Authenticating as: worker1,,, (worker1)
Password: 
==== AUTHENTICATION COMPLETE ===
worker1@hadoop1:~/.ssh$ ssh localhost
Last login: Sun Sep 19 00:23:03 2021 from 127.0.0.1
worker1@hadoop1:~$ exit
logout
Connection to localhost closed.
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/986853804339469f82ce7566394d5c20.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)
新建自己的存放数据的文件夹，注意路径不要直接用复制过来时hadoop0的路径
![在这里插入图片描述](https://img-blog.csdnimg.cn/743862856db64220aeb52797913b0911.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)

namenode主节点上此时运行着的datanode有自己(localhost)和hadoop0
namenode的/etc/hosts中有：
![在这里插入图片描述](https://img-blog.csdnimg.cn/70ae05c8ab674ff19ae19c5a25c45e91.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)
在namenode的workers中加上新节点hadoop1
![在这里插入图片描述](https://img-blog.csdnimg.cn/9ef3b56ab260488eb84809ba949ff427.png)
（可跳过这个）namenode检查是否处于安全模式
```powershell
(base) jj@jidafus-MBP ~ % hdfs dfsadmin -safemode get
2021-09-19 10:21:38,538 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Safe mode is OFF
```

namenode刷新
```powershell
(base) jj@jidafus-MBP ~ % hdfs dfsadmin -refreshNodes  
2021-09-19 10:22:13,865 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Refresh nodes successful
(base) jj@jidafus-MBP ~ % yarn rmadmin -refreshNodes
2021-09-19 10:10:21,626 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
2021-09-19 10:10:21,688 INFO client.DefaultNoHARMFailoverProxyProvider: Connecting to ResourceManager at /192.168.43.99:8033

```

在hadoop1上直接启动datanode和yarn

```powershell
jj@hadoop1:~/Desktop$ hdfs --daemon start datanode
jj@hadoop1:~/Desktop$ yarn --daemon start nodemanager
jj@hadoop1:~/Desktop$ jps
6628 Jps
6517 NodeManager
6389 DataNode
jj@hadoop1:~/Desktop$ hadoop fs -ls /
2021-09-19 10:57:22,212 WARN fs.FileSystem: "192.168.43.99:9000" is a deprecated filesystem name. Use "hdfs://192.168.43.99:9000/" instead.
Found 3 items
drwxr-xr-x   - jj supergroup          0 2021-09-19 10:52 /jjProj
drwxr-xr-x   - jj supergroup          0 2021-09-18 02:42 /mmProj

```
![请添加图片描述](https://img-blog.csdnimg.cn/41523de1ffdd42a18ba7f055a0c24c92.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/31ba8f52d9c24925b55c482a8c27424e.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)



关于ssh免密登陆
Some addition information might be useful
![在这里插入图片描述](https://img-blog.csdnimg.cn/7b0514f248b84987a502e3190a759d4e.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)
[主节点和从节点的hadoop-home路径不一致，不能群开，代码查看sbin/start-dfs.sh](https://www.edureka.co/community/54400/hadoop-error-slave-bash-line-hadoop-libexec-file-directory)


# hbase版本1.2.6，可用于hdfs3.3.1
1）hbase-site.xml
```xml
<configuration>
  <!-- hbase-site.xml -->
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
  
  <property>
    <name>hbase.tmp.dir</name>
    <value>/usr/local/Cellar/hbase-2.3.6/tmp</value>
  </property>
  <property>
          <name>hbase.master</name>
          <value>master:60000</value>
    </property>
    <property>
          <name>hbase.rootdir</name>
          <value>hdfs://master:9000/hbase</value><!--dfs://master:9000/hbase-->
      </property>
      
      <property>
          <name>hbase.unsafe.stream.capability.enforce</name>
          <value>false</value>
      </property>
      <!--<property>
          <name>zookeeper.znode.parent</name>
          <value>/hbase-unsecure</value>
      </property>-->
      <property>
              <name>hbase.zookeeper.property.dataDir</name>
              <value>/Users/jj/zookeepers/zk1/data</value>
          </property>
<property>
          <!-- 指定 zk 的地址，多个用“,”分割 -->
          <name>hbase.zookeeper.quorum</name>
          <value>master:2181,,hadoop0:2181,hadoop1:2181</value>
  </property>
  <property>
      <name>hbase.master.info.port</name>
      <value>16010</value>
  </property>
  <property>
      <name>hbase.regionserver.info.port</name>
      <value>16030</value>
    </property>
</configuration>

```

2）hbase-env.sh中的设置，使用外部（自己下载）的zookeeper管理集群
注意HBASE_CLASSPATH是$HADOOP_HOME
```xml
export HBASE_MANAGES_ZK=false
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_301.jdk/Contents/Home
export HBASE_HOME=/usr/local/Cellar/hbase-2.3.6
export HBASE_CLASSPATH=/usr/local/Cellar/hadoop/3.3.1/libexec/etc/hadoop
export HBASE_LOG_DIR=${HBASE_HOME}/logs
export HBASE_PID_DIR=${HBASE_HOME}/pids
```
3）修改regionservers文件，（master，hadoop0，hadoop1已在/etc/hosts中设置好，而且已确保可以用zookeeper管理）
![在这里插入图片描述](https://img-blog.csdnimg.cn/dbe66f21da654bb5b22227b8c93c1a33.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)

4）同时，我还将hadoop-home中的hdfs-site.xml和core-site.xml复制到hbase的conf目录下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/83aaa7f78d704858ad115ea7a59e2d36.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)
5）先启动了zookeeper和hadoop后
主节点master上start-hbase.sh，
从节点hadoop0, hadoop1 上 hbase-daemon.sh start regionserver
输入 hbase shell 命令，对hbase进行数据库操作
web url:  http://master:16010/master-status
![在这里插入图片描述](https://img-blog.csdnimg.cn/c5b79076b7e94f268a06a818041e41e8.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_17,color_FFFFFF,t_70,g_se,x_16)
![在这里插入图片描述](https://img-blog.csdnimg.cn/73a5365b4c45460d9c45902c335b8dda.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/c18f44fce9414e11a983c1144bd603ba.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/4beb90ef45da40a0827820de21b0bd4c.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/acb575b210ad4880b04021fe0f7b1116.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)

![](https://img-blog.csdnimg.cn/1cdfbbe04dd448d08207068243bbb64d.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)
有个小细节要记一下：在我MBP上一开始我设置了主机名为jj-laptop，hbase的从节点regionserver无法识别这个主机名，在hbase shell中会报错“jj-laptop error”，将主机名改为master，这样与zookeeper中的主机名以及/etc/hosts中一致即可。

![在这里插入图片描述](https://img-blog.csdnimg.cn/536367af7f2448858a13b64c5f54bfd4.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/ce8e1fa2d7f44d38b1d8966ebffea07c.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/cf33d2c76e27466291a2c05fcd5086a6.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/08dcacfd8ab44a129ccfea38cfbb49f5.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/4b9669dc7e584c2abd6e036973ee6308.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)

# spark
![在这里插入图片描述](https://img-blog.csdnimg.cn/404825f737f74040bad1018434c95e9b.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)

1）master主节点上：
start-master.sh
start-workers.sh
2）hadoop0, hadoop1上：
start-worker.sh spark://master:7077
![在这里插入图片描述](https://img-blog.csdnimg.cn/2aecafdaa7134a0d9c05642d8110fb74.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_16,color_FFFFFF,t_70,g_se,x_16)

![在这里插入图片描述](https://img-blog.csdnimg.cn/eb01fb57de8f429db6eb3be5fd0a1b1e.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA57qq5aSn56aP,size_20,color_FFFFFF,t_70,g_se,x_16)

# solved
### Hregionserver都可启动，但是web上不显示
solution：调整节点时间一致
查看时间 `date`      
<img width="824" alt="image" src="https://user-images.githubusercontent.com/43733497/159126513-26961d4c-9daf-4255-918e-365191ba0d62.png">
