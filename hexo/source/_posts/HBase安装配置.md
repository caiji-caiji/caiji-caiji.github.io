# HBase安装配置

### 1.环境变量配置

> 将安装包上传至CentOS中，解压安装包到hadoop目录下

    tar -zxvf hbase-1.3.6-bin.tar.gz -C /usr/hadoop

> 在hadoop目录下创建zookeeper文件夹

	mkdir zookeeper


> 配置环境变量，

	vi /etc/profile

> 在末尾添加以下代码，保存退出

	export HBASE_HOME=/usr/hadoop/hbase-1.3.6
	export PATH=$HBASE_HOME/bin:$PATH

> 生效配置

	source /etc/profile


### 2.配置HBase

> 进入hbase-1.3.6/conf 文件夹中，配置hbase-env.sh文件，修改Java路径，并去掉注释

	export JAVA_HOME=/usr/java/jdk1.8.0_121/

> 去掉注释

	export HBASE_MANAGES_ZK=true

> 保存退出，并打开hbase-site.xml文件，修改该文件

	<configuration>

		<!-- 将HBase数据保存在HDFS目录中 -->
  		<property>
   			<name>hbase.rootdir</name>
    		<value>hdfs://master:8020/hbase</value>
  		</property>
		
		<!-- HBase是否是分布式环境 -->
	    <property>
		    <name>hbase.cluster.distributed</name> 
		    <value>true</value> 
	    </property>

		<!-- 配置zookeeper地址，4个节点全部启用zookeeper -->
	    <property>
		    <name>hbase.zookeeper.quorum</name> 
		    <value>master,slave1,slave2,slave3</value>
	    </property> 

		<!-- zookeeper数据目录  -->
    	<property>
		    <name>hbase.zookeeper.property.dataDir</name>
		    <value>/usr/hadoop/zookeeper</value>
	    </property>

		<!-- 设置Region的冗余度  -->
		<property>
                <name>dfs.replication</name>
                <value>2</value>
        </property>

		<!-- HBase端口号，默认为16010  -->
	    <property>
		    <name>hbase.master.info.port</name>
		   	<value>16011</value>
	    </property>

    </configuration>

> 修改配置文件regionservers，将里面内容修改为：

	master
	slave1
	slave2
	slave3

### 3.同步HBase配置文件

>同步master结点的HBase配置文件，至slave1、slave2、slave3

	scp -r /usr/hadoop/hbase-1.3.6 slave1:/usr/hadoop 
	scp -r /usr/hadoop/hbase-1.3.6 slave1:/usr/hadoop 
	scp -r /usr/hadoop/hbase-1.3.6 slave1:/usr/hadoop 


> 分别配置slave1、slave2、slave3的环境变量

	vi /etc/profile

> 在末尾添加以下代码，保存退出

	export HBASE_HOME=/usr/hadoop/hbase-1.3.6
	export PATH=$HBASE_HOME/bin:$PATH

> 生效配置

	source /etc/profile

> 3个slave结点配置步骤同上，并进行验证

	hbase version

### 4.启动HBase

> 运行HBase启动命令

	start-hbase.sh 

> 输入jps进行查看

	[root@master bin]# jps

	9760 ResourceManager
	9447 NameNode
	13208 HMaster
	13337 HRegionServer
	12202 HQuorumPeer
	9612 SecondaryNameNode
	14414 Jps


> 进入HBase web管理页面

	http://192.168.147.10:16011/

### tips：如出现以下情况，导致web管理页面无法打开


	pids/hbase-root-master.pid: 没有那个文件或目录


### tips 1 确定hdfs与hbase配置文件是否相同
> hbase-site.xml下的hbase.rootdir下面的value值,必须要和hadoop配置文件core-site.xml下的fs.defaultFS下的value值，ip和端口相同！

### tips 2 修改hbase的pid文件保存路径

>  打开conf目录下的hbase-env.sh，找到以下代码

	# export HBASE_PID_DIR=/var/hadoop/pids

> 修改为自己的路径
	
	export HBASE_PID_DIR=/usr/hadoop/hbase-1.3.6/pids


### 5.进入HBase Shell

	hbase shell