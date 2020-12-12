## 1. 安装虚拟机
## 2. 配置网络

### 2.1 网络连接介绍
> 1.桥接模式：虚拟机和物理机连的是同一个网络，虚拟机和物理机是并列关系，地位是相当的。无论是虚拟系统还是真实系统，只要在同一个网段下，相互之间就能ping通。

> 2.NAT模式：物理机会充当一个“路由器”的角色，虚拟机要想上网，必须经过物理机，那物理机如果不能上网，虚拟机也就不能上网了。不需要进行任何其他的配置，只需要物理机能访问互联网即可。虚拟系统只能和设置虚拟机的主机之间双向访问，无法访问同一网段下其他真实主机。

> 3.仅主机模式：相当于拿一根网线直连了物理机和虚拟机。所有的虚拟系统是可以相互通信的，但虚拟系统和真实的网络是被隔离开。

### 2.2 网络验证
CentOS下查看ip地址：

    ip a
Windows下查看ip地址（运行cmd）：

    ipconfig

- 虚拟机 ping 网关
- 虚拟机 ping 物理主机
- 物理主机 ping 虚拟机
- 虚拟机 ping 百度（若ping不通，先在network-scripts文件夹下网络配置文件中，添加DNS1="网关地址"）

### 2.3 重启网卡
    service network restart
    systemctl restart network

### 2.4 关闭防火墙
> 分布式集群中，各个节点之间的通信会受到防火墙的阻碍；     
> 会将集群中各个节点的防火墙关闭；  
> 网络安全问题由专业的网络安全平台、软件在外围实现防火墙功能，统一管理。

停止  

    systemctl stop firewalld.service 

禁止开机启动  

    systemctl disable firewalld.service 

出现网络问题，先查看虚拟网卡配置文件

    vi /etc/sysconfig/network-scripts/ifcfg-ens33


## 3. 安装XShell、XFtp
## 4. Linux配置
### 4.1 切换用户 

    su root

### 4.2 配置时钟同步
（从多台节点上拿块数据，在内存中进行组合。计算机之间的通讯和数据的传输一般都是以时间为约定条件。导致取块数据时出现时间延迟。）
 
在线安装ntpdate

    yum install ntpdate 
    ntpdate cn.pool.ntp.org
    date

离线安装ntpdate  
ntpdate下载地址： [https://pkgs.org/](https://pkgs.org/)  

    rpm -ivh your-package 
    ntpdate cn.pool.ntp.org
    date

### 4.3 配置主机名
> 在网络中能够唯一标识主机，和ip地址一样；  
> 可以通过ip地址和网络主机名访问这台主机。

查看主机名：

    hostname

修改主机名：

    hostnamectl set-hostname 主机名

### 4.4 配置hosts列表
> hosts列表作用是让集群中的每台服务器彼此都知道对方的主机名和ip地址

    vi /etc/hosts

添加主机ip和主机名

    192.168.147.10 master

验证

    ping master

### 4.5 配置JAVA环境
创建JAVA目录

    mkdir /usr/java

复制JAVA安装包，解压，移动至java目录下

    tar -zxvf jdk-8u191-linux-x64.tar.gz
    mv /usr/yoseng/jdk1.8.0_121 /usr/java

进入系统配置文件，进行JAVA配置

    vi /etc/profile
    export JAVA_HOME=/usr/java/jdk1.8.0_121
    export PATH=$JAVA_HOME/bin:$PATH

生效配置

    source /etc/profile
    java -version

### 4.6 配置单机模式Hadoop环境
解压Hadoop安装包，并移动至新的文件夹中

	tar -zxvf hadoop-2.8.5.tar.gz
	mkdir hadoop
	mv hadoop-2.8.5  /usr/hadoop

配置环境变量

	vi /etc/profile
	export HADOOP_HOME=/usr/hadoop/hadoop-2.8.5
	export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin

使配置生效，进行验证

    source /etc/profile
    whereis hdfs

## 5. 单机模式的Hadoop安装
设置hadoop配置文件

    cd /usr/hadoop/hadoop-2.8.5/etc/hadoop
	vi hadoop-env.sh

找到下面这行代码：

	export JAVA_HOME=${JAVA_HOME}

将这行代码修改为

	export JAVA_HOME=/usr/java/jdk1.8.0_121

验证

    hadoop version

## 6. 伪分布式模式的Hadoop安装准备
### 6.1 克隆多台salve虚拟机
slave1、slave2、slave3  

### 6.2 修改slave的ip地址
修改完后使用Xshell登录

### 6.3 修改slave主机名
### 6.4 修改hosts文件
将slave的ip地址加入到各自的hosts文件中

### 6.5 配置SSH登录(先关闭防火墙)
使用rsa算法产生公钥和私钥（安装过程中，使用“Enter”键确定）

    ssh-keygen -t rsa

查看生成的私钥和公钥

	cd /root/.ssh/
	ls

在master上创建一个大家通用的公钥authorized_keys，修改authorized_keys权限，并将这个公钥发送给每个slave

	cat id_rsa.pub >> authorized_keys
	chmod 644 authorized_keys
	systemctl restart sshd.service
	scp /root/.ssh/authorized_keys slave1:/root/.ssh
	scp /root/.ssh/authorized_keys slave2:/root/.ssh
	scp /root/.ssh/authorized_keys slave3:/root/.ssh


> Linux文件权限  

- 1-3位数字代表文件所有者的权限
- 4-6位数字代表同组用户的权限
- 7-9位数字代表其他用户的权限
- 读取的权限等于4，用r表示
- 写入的权限等于2，用w表示
- 执行的权限等于1，用x表示

	444 r--r--r--  
	600 rw-------  
	644 rw-r--r--  
	666 rw-rw-rw-  
	700 rwx------  
	744 rwxr--r--  
	755 rwxr-xr-x  
	777 rwxrwxrwx  

ssh登录检验，不需要密码即可登录
路径从'~/.ssh '变成'~'，登出为exit

	ssh master
	ssh slave1
	exit

实现master到slave单向免登陆！！！

## 7. 伪分布式模式的Hadoop安装
### 7.1 配置hadoop-env.sh文件

添加Java安装路径

### 7.2 配置core-site.xml文件

	<configuration>
	    <!--指定文件系统的入口地址，可以为主机名或ip -->
	    <!--端口号默认为8020 -->
	    <property>
	        <name>fs.defaultFS</name>
	        <value>hdfs://master:8020</value>
	    </property>
	
	    <!--指定hadoop的临时工作存目录-->
	    <property>
	        <name>hadoop.tmp.dir</name>
	        <value>/usr/hadoop/tmp</value>
	    </property>
	</configuration>

### 7.3 配置yarn-env.sh文件
yarn负责管理hadoop集群的资源，修改文件中的Java路径

	export JAVA_HOME=/usr/java/jdk1.8.0_121

### 7.4 配置hdfs-site.xml文件

	<configuration>

	    <!--指定hdfs备份数量，小于等于从节点数目-->
	    <property>
	        <name>dfs.replication</name>
	        <value>3</value>
	    </property>
			
	  <!--  指定hdfs中namenode的存储位置-->
	  <!--  <property>-->
	  <!--      <name>dfs.namenode.name.dir</name>-->
	  <!--      <value>file:/usr/hadoop/dfs/name</value>-->
	  <!--  </property>-->
	
	  <!--  指定hdfs中datanode的存储位置-->
	  <!--  <property>-->
	  <!--      <name>dfs.datanode.data.dir</name>-->
	  <!--      <value>file:/usr/hadoop/dfs/data</value>-->
	  <!--</property>-->
	
	</configuration>

### 7.5 配置mapred-site.xml文件
生成文件

	cp mapred-site.xml.template mapred-site.xml

编辑mapred-site.xml文件

	<configuration>
	    <!--hadoop的MapReduce程序运行在YARN上-->
	    <!--默认值为local-->
	    <property>
	        <name>mapreduce.framework.name</name>
	        <value>yarn</value>
	    </property>
	</configuration>

### 7.6 配置yarn-site.xml文件


	<configuration>
	
	    <property>
	        <name>yarn.resourcemanager.hostname</name>
	        <value>master</value>
	    </property>
	        
	    <!--nomenodeManager获取数据的方式是shuffle-->
	    <property>
	        <name>yarn.nodemanager.aux-services</name>
	        <value>mapreduce_shuffle</value>
	    </property>  
	
	</configuration>

### 7.7 修改slaves文件
删除原有内容，替换奴隶主机名 

	slave1
	slave2
	slave3

### 7.8 将修改后的hadoop文件夹发送给三台slave机

	scp -r /usr/hadoop slave1:/usr/hadoop 
	scp -r /usr/hadoop slave2:/usr/hadoop 
	scp -r /usr/hadoop slave3:/usr/hadoop 

在三台slave机上验证hadoop
	
	hadoop version

## 8. 准备运行hadoop
### 8.1 格式化HDFS
进入hadoop下的bin文件夹，运行以下代码：

	hdfs namenode -format

> 注意：只需格式化一次！  
> 多次格式化会导致ID值不匹配，需要在格式化前删除NameNode，DataNode、日志等文件夹。

### 8.2 启动hadoop
进入sbin文件夹下，运行：

	start-dfs.sh
	start-yarn.sh

### 8.3 查看hadoop进程

	jps

### 通过web端访问hadoop

查看NameNode、DataNode：    
[http://192.168.233.4:50070](192.168.233.4:50070)  
查看SecondaryNameNode信息：  
[http://192.168.233.4:50090](192.168.233.4:50090)  
查看YARN界面：  
[http://192.168.233.4:8088](192.168.233.4:8088)