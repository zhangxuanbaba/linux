## 单机搭建Zookeeper集群并用dubbo访问（伪集群 三个节点）

#### Zookeeper简介
##### (一) Zookeeper基础知识、体系结构、数据模型
1. zookeeper是一个类似hdfs的树形文件结构，zookeeper可以用来保证数据在(zk)集 群之间的数据的事务性一致。
2. zookeeper有watch事件，是一次性触发的，当watch监视的数据发生变化时，通 知设置了该watch的client，即watcher。
3. zookeeper有三个角色: Learder，Follower，Observer4 zookeeper应用场景。
统一命名服务(Name Service)	
配置管理(Configuration Management)	
集群管理(Group Membership)	
共享锁(Locks)	
队列管理	

##### (二) Zookeeper配置(搭建zookeeper服务器集群)

1. 结构:一共三个节点 (zk服务器集群规模不小于3个节点),要求服务器之间系统时间保持一致。

###### 创建文件存放目录

		mkdir -p /usr/local/software

###### 进入到文件存放目录

		cd /usr/local/software/

###### 下载Zookeeper-3.4.5

		 wget http://archive.apache.org/dist/zookeeper/zookeeper-3.4.5/zookeeper-3.4.5.tar.gz

###### 创建zokeeper node节点安装位置

		mkdir -p /usr/local/zookeeper-3.4.5

###### 把文件解压到/usr/local/zookeeper-3.4.5

		tar -zxvf /usr/local/software/zookeeper-3.4.5.tar.gz -C /usr/local/zookeeper-3.4.5/

###### 进入到zookeeper节点总目录

		cd /usr/local/zookeeper-3.4.5

###### 对zookeeper文件名进行更改 更改为node1(其中一个子节点)

		mv /usr/local/zookeeper-3.4.5/zookeeper-3.4.5/ /usr/local/zookeeper-3.4.5/node1

###### 更改后文件显示如下

		[root@localhost zookeeper-3.4.5]# ls -l
		total 4
		drwxr-xr-x 10 501 games 4096 Nov  5  2012 node1

###### 进入到其中一个node节点

		[root@localhost zookeeper-3.4.5]# cd /usr/local/zookeeper-3.4.5/node1/

###### 在node1节点下创建data目录 data目录是用来存放myid文件的 在其后的zoo.cfg中会用到(后面有介绍)

		[root@localhost node1]# mkdir -p /usr/local/zookeeper-3.4.5/node1/data

###### 在node1节点下创建logs目录 logs目录用来存放zookeeper文件输出

		mkdir -p /usr/local/zookeeper-3.4.5/node1/logs

###### 在data下创建myid文件 myid为了标识zookeeper的节点

		[root@localhost node1]# vim /usr/local/zookeeper-3.4.5/node1/data/myid

###### 服务器标识配置: myid (内容为服务器标识 : 1)

		[root@localhost node1]# vim /usr/local/zookeeper-3.4.5/node1/data/myid
		1 (在myid文件中 添加一个字”1”)

###### 进入到zookeeper node1中的conf文件夹

		cd /usr/local/zookeeper-3.4.5/node1/conf/

###### copy zookeeper node1的配置 然后对其进行修改

		[root@localhost conf]# cp /usr/local/zookeeper-3.4.5/node1/conf/zoo_sample.cfg /usr/local/
		zookeeper-3.4.5/node1/conf/zoo.cfg

###### 修改zoo.cfg文件

		[root@localhost conf]# sudo vim /usr/local/zookeeper-3.4.5/node1/conf/zoo.cfg


注: 也可把之前的zoo.cfg文件全部删除 然后把下面的代码直接copy到zoo.cfg
![](http://47.93.176.227:8888//group1/M00/8A/05/L12w41tlQvGAbXfoAAgpiltiLn8815.jpg)


		# 服务器之间或客户端与服务器之间维持心跳的时间间隔(基本事件单元，以毫秒为单位)，也就是每隔 tickTime时间就会发送一个心跳。
		tickTime=2000
		# 这个配置项是用来配置 Zookeeper 接受客户端初始化连接时最长能忍受多少个心跳时间间隔数，当已经超过 10 个心跳的时间(也就是 tickTime)长度后 Zookeeper 服务器还没有收到客户端的返回信息，那么表明这个客户端连接失败。总的时间长度就是10*2000=20 秒。
		initLimit=10
		# 这个配置项标识 Leader 与 Follower之间发送消息，请求和应答时间长度，最长不能超过多少个 tickTime的时间长度，总的时间长度就是 5*2000=10 秒
		syncLimit=5
		# 存储内存中数据库快照的位置，顾名思义就是 Zookeeper 保存数据的目录，默认情况下，Zookeeper 将写数据的日志文件也保存在这个目录里。
		dataDir=/usr/local/zookeeper-3.4.5/node1/data
		# 存储内存中数据库快照的位置 顾名思义就是日志文件存放目录
		dataLogDir=/usr/local/zookeeper-3.4.5/node1/logs
		# 这个端口就是客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求
		clientPort=3210
		# 注: 以下ip为内网ip即可
		# server.A = B:C:D
		# A表示这个是第几号服务器
		# B 是这个服务器的ip地址
		# C 表示的是这个服务器与集群中的Leader服务器交换信息的端口
		# D 表示的是万一集群中的Leader服务器挂了，需要一个端口来重新进行选举，选出一个新的Leader
		server.1=ip:3211:3212
		server.2=ip:3221:3222
		server.3=ip:3231:3232
		
ok, 目前为至, 其中一个节点已经搭建好了 

##### 注意: 现在不能启动该节点, 会报错, 目前该节点还不能找到集群中的其他两个节点

###### 把node1节点复制一份到node2 并修改node2的相关配置

		[root@localhost conf]# cp -r /usr/local/zookeeper-3.4.5/node1/ /usr/local/zookeeper-3.4.5/node2

###### 到node2节点路径下

		[root@localhost zookeeper-3.4.5]# cd /usr/local/zookeeper-3.4.5/node2/

###### 修改data文件中的myid 把当前myid为1的值 修改为2

		[root@localhost node2]# sudo vim /usr/local/zookeeper-3.4.5/node2/data/myid
		2

###### 修改zookeeper中node2的zoo.cfg配置

		[root@localhost node2]# sudo vim /usr/local/zookeeper-3.4.5/node2/conf/zoo.cfg

		# 注意把之前dataDir中的node1 换成了node2
		dataDir=/usr/local/zookeeper-3.4.5/node2/data
		# 注意把之前dataLogDir中的node1 换成了node2
		dataLogDir=/usr/local/zookeeper-3.4.5/node2/logs
		# 把端口号从3210换为了3220
		clientPort=3220
		
ok, 到此zookeepr第二个节点搭建完毕

##### 把node1节点复制一份到node3 并修改node3的相关配置

		[root@localhost node2]# cp -r /usr/local/zookeeper-3.4.5/node1/ /usr/local/zookeeper-3.4.5/node3

###### 到node3节点路径下

		[root@localhost zookeeper-3.4.5]# cd /usr/local/zookeeper-3.4.5/node3/

###### 修改data文件中的myid 把当前myid为1的值 修改为3

		[root@han node3]# sudo vim /usr/local/zookeeper-3.4.5/node3/data/myid
		3

###### 修改zookeeper中node3的zoo.cfg配置

		[root@han node3]# sudo vim /usr/local/zookeeper-3.4.5/node3/conf/zoo.cfg

		# 注意把之前dataDir中的node1 换成了node3
		dataDir=/usr/local/zookeeper-3.4.5/node3/data
		# 注意把之前dataLogDir中的node1 换成了node3
		dataLogDir=/usr/local/zookeeper-3.4.5/node3/logs
		# 把端口号从3210换为了3230
		clientPort=3230

ok, 到此zookeepr第三个节点搭建完毕

------------


启动zookeeper集群
注意: 这里3台机器都要进行启动

		/usr/local/zookeeper-3.4.5/node1/bin/zkServer.sh  start
		/usr/local/zookeeper-3.4.5/node2/bin/zkServer.sh  start
		/usr/local/zookeeper-3.4.5/node3/bin/zkServer.sh  start

停止zookeeper集群:

		/usr/local/zookeeper-3.4.5/node1/bin/zkServer.sh  stop
		/usr/local/zookeeper-3.4.5/node2/bin/zkServer.sh  stop
		/usr/local/zookeeper-3.4.5/node3/bin/zkServer.sh  stop

查看zookeeper集群状态：
注意: 查看三个zookeeper节点的status时,一个leader和俩个follower(如下图)

		/usr/local/zookeeper-3.4.5/node1/bin/zkServer.sh  status
		/usr/local/zookeeper-3.4.5/node2/bin/zkServer.sh  status
		/usr/local/zookeeper-3.4.5/node3/bin/zkServer.sh  status









