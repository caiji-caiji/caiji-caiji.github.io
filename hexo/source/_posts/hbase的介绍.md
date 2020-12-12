# hbase的介绍

### GFS------------HDFS分布式文件系统

### MAPREDUCE------------分布式并行计算       离线并行计算

###                                                   移动计算比移动数据更划算    数据在哪里计算就在哪里

### 解决离线问题

> mapreduce可以做到查询----------高延迟
>
> hbase----------实时

###  bigtable------------hbase   存储和数据访问

> 实时：-----ms
>
> 近实时：-----s
>
> 跳表结构：建多级索引
>
> hbase底层：布隆过滤器+跳表     每个索引范围的数据存储在不同的节点上
>
> 原始数据hdfs
>
> ​	索引存储放在hdfs
>
> ​	......
>
> ​	顶级索引：hdfs （各个节点都可以访问到）
>
> ​	需要一个能够记录顶级索引存储地点的叫做索引入口      关键----需要存储多份，数据需要同步
>
> ​	zookeeper存储    zookeeper就是存储索引入口
>
> hbase的原始数据存储在hdfs上



#  hbase的安装

###  hbase的安装：底层为java语言所写

1. jdk
2. zookeeper
3. hadoop

> （1）hbase版本
>
> ​		不选最新的版本   也不选太老的版本   选择1.2.6版本
>
> ​		注意hbase和别的组件的兼容性问题：
>
> ​		1.2版本要求   jdk>1.6；   hadoop的兼容性
>
> （2）hbase的架构：
>
> ​		主从架构             一主(Hmaster)多从(Hregionserver)的架构
>
> ​		规划    	3个节点
>
> 安装：
>
> ​		1）上传安装包
>
> ​		2）解压
>
> ​		3）配置环境变量

```shell
source /etc/profile	
hbase version 验证
```

> ​		4）修改配置文件

```shell
HBASE_HOME/conf
	1)hbase-env.sh
	  export JAVA_HOME=usr/java/jdk1.8.0_73
	2)export HBASE_MANAGES_ZK=false
```

> ​			 true:代表使用hbase自带的zookeeper    这个方案只适合单机模式  不适合分布式模式
> ​	 		 false:不使用hbase自带的   使用自己安装的zookeeper

```shell
	2)hbase-site.xml  #hbase的核心配置文件
	#需要添加zookeeper的地址
		<name>hbase.rootdir</name>
		<value>hdfs://master:8020/hbase</value>  #这里给的是命名空间
		<!--指定hbase是分布的-->
		<name>hbase.cluster.distributed</name> 
		<value>true</value> 
		<!--指定zk的地址，多用“,”分割-->
		<name>hbase.zookeeper.quorum</name> 
		<value>master,slave1,slave2,slave3</value>
```

```shell
	3)#regionsevers文件   从节点的主机名（slaves）
    vi regionslaves
    slave1
    slave2
    slave3
    #region:存储一定范围数据的单位
    #servers:服务
```

```shell
	4) vi backup-masters
	#这个是配置hmaster的备份节点
```

> 5）将hadoop的hdfs的hdfs-site.xml    core-site.xml放在hbase下的conf里

```shell
cp /usr/hadoop/hadoop-2.8.5/etc/hadoop/hdfs-site.xml
cp /usr/hadoop/hadoop-2.8.5/etc/hadoop/core-site.xml
```

> 6）将hbase的安装包发送到其他节点

```shell
scp -r hbase-1.2.6 slave1:$PWD
scp -r hbase-1.2.6 slave2:$PWD
scp -r hbase-1.2.6 slave3:$PWD
scp -r erc/profile slave1:/etc/
scp -r erc/profile slave2:/etc/
scp -r erc/profile slave3:/etc/
#奴隶机上分别执行   source
```

> 7）启动
>
> ​	1）启动hadoop  
>
> ​		1）启动zookeeper    每个节点单独启动
>
> ​			 检查状态
>
> ​		2）启动hdfs
>
> ​		3）启动yarn
>
> ​	2）启动hbase

```shell
start-hbase.sh
#这个命令在任意节点执行     在哪个节点执行就会在哪个节点启动一个Hmaster(活跃的)
```

> 验证：
>
> master:16010
>
> 单独启动Hmaster:

```shell
hbase-damon.sh start master
#backup-masters：集群启动的
```

> 在hbase中可以启动多个master   只有一个是active的   剩下的全部是backup的。

### hbase的存储和架构

#### hbase的特点：

> ​	面向列的nosql数据库
>
> ​	nosql-------- no   sql  不支持标准sql       insert   delete   update   select(不支持)
>
> ​	not only sql         不仅仅支持sql      phoenix基于hbase    对外提供标准sql的 
>
> ​	nosql数据库：redis, moungdb
>
> > 特点：
> >
> > 1）不支持复杂事务（只支持行级事务）
> >
> > 2）不支持多表join      想要做多表join使用hive
> >
> > 3）hbase数据库中数据存储格式：byte[]     可以存储半结构化   或非结构化
> >
> > 4）hbase主要用于存储半结构化（快速查询，  近实时）和结构化数据（html   xml）
> >
> > 5）hbase无模式
>
> > > mysql------------写模式：写数据的时候进行校验
> > >
> > > hive---------------读模式的数据仓库：读数据的时候进行校验
> > >
> > > hbase无严格模式数据库
> > >
> > > hbase中的数据最终存储在hdfs上    上层的数据计算------mapreduce
>
> > hbase中存储的表的特点：
> >
> > 1.大    底层存储在hdfs上    理论上存储数据无上限   千亿级别    分布式存储的
> >
> > 2.面向列存储  
> >
> > ​		mysql/oracle/sql  server       面向行存储的    一行的数据肯定存储在同一个文件中
> >
> > ​		对于关系型数据库：做的最多的查询select  lie from table
> >
> > ​		hbase中面向列存储的    一列（列簇）存储一个文件
> >
> > > 优点：		
> > >
> > > 1）提升查询性能
> > >
> > > 2）减少io
> > >
> > > 3）同一列数据类型一般相同   压缩比更大
> >
> > 3.稀疏
> >
> > ​		mysql中有大量的null值存在存储的时候占用存储空间
> >
> > > ​		    1）存储的结构上：hbase中null值不占用存储空间
> > >
> > > ​			2）列对应关系上：
> > >
> > > ​				mysql中没有的列也要补全   哪怕用null
> > >
> > > ​				hbase：有的字段就插入数据   没有此列的就不用管
>
> hbase中的相关概念
>
> > 行键：rowkey：一行数据的标志   类似于mysql数据库中的id
> >
> > 不同行的rowkey肯定不同   行标志   相同的rowkey肯定是同一行
> >
> > >  1）在hbase中行键不宜过大   通常10-100byte     16byte最好，涉及存储问题
> > >
> > >  ​		每个列簇的物理文件中都会存储一个rowkey
> > >
> > >  2）hbase中存储的时候默认会按照rowkey的字典进行升序排序
> > >
> > >  ​	跟底层的存储有关
> >
> > 在hbase中对数据查询3种方式：
> >
> > > 1）全表扫描   效率低
> > >
> > > 2）通过rowkey  范围查询    指定查询某一个rowkey范围内的数据
> > >
> > > 3）查询当rowkey的数据
> >
> > 列簇：一个或多个列
> >
> > > 通常情况下一个表中的列簇个数不要超过3个
> > >
> > > 列簇就是hbase存储的物理切分单位  面向列簇的   实际存储的时候会（每个列簇存储一个文件）
> > >
> > > 什么样的列划分到一个列簇中？具有相同io特性    通常情况下会一起访问的列
> > >
> > > 具有相同io特性的会放在同一个列簇中，如果一个表中字段特别多，io特性也特别多   将不同的io特性强制分到一个列簇中，最好列簇中不超过3个   实际生产中1个
> > >
> > > 列簇越多，存储的文件个数越多      增加扫描成本   建表的时候执行
> >
> > 列：数据插入的时候  指定的列名   列的值
> >
> > > 没有严格的表结构的
> >
> > 时间戳：
> >
> > > 在数据进行插入的时候   每插入一条数据就会自动生成一个时间戳   时间戳的作用记录数据的版本
> > >
> > > hbase中可以存储多个版本的数据   多个版本之间靠时间戳记录的
> > >
> > > 时间戳默认就是系统时间    时间戳  也可以进行手动指定
> >
> > 单元格：cell
> >
> > > 某一行数据的某一列的某一个值
> > >
> > > 在hbase中想定位一个单元格需要哪些：    行键+列簇+时间戳
>
> hbase的架构和存储：
>
> > 寻址路径------zookeeper中
> >
> > 主从
> >
> > 主：hmaster
> >
> > 从：hregionserver
> >
> > 用于管理region的
> >
> > region：hbase在行方向上的逻辑切分概念
> >
> > 一个hbase中的表在最开始的时候只有一个region   面向行的
> >
> > 随着数据增加   region会进行分裂    分裂的时候依然会按照行方向进行分裂   取中间
> >
> > 主要目的：便于查询   建索引
> >
> > region分裂文件大小：
> >
> > > 0.9中   256M
> > >
> > > 1.x中   10G
> >
> > 存储：
>

#### Hbase  shell操作

> 在hbase中弱化库的概念       namespace
>
> help
>
> help  命令
>
> 表：
>
> > 创建表
> >
> > 列簇是建表时指定的     列名是插入时指定的
> >
> > ```shell
> > create 't1',{NAME => 'f1'},{NAME => 'f2'},{NAME => 'f3'}
> > create 't1','f1','f2','f3'
> > ```
> >
> > > t1:表名         f1,f2,f3:column  family （列簇）
> > >
> > > list查看所有表信息
> > >
> > > descripbe 表名     查看表的描述信息    一个{}表示一个列簇     列簇中显示的信息皆可在建表时修改
> > >
> > > > TTL：生命周期    forever  永久
> > > >
> > > > VERSION：保存的版本   默认1
> >
> > 增
> >
> > > 在表中插入数据
> > >
> > > ``` shell
> > > put 't1','r1','c1','value'
> > > ```
> > >
> > > t1:表名
> > >
> > > r1:行键    rowkey  行键相同的会认为同一行
> > >
> > > c1:列名   这里的列名需要指定列簇   列名随意   'f1:c1'
> > >
> > > value:列的值
> > >
> > > ##### 写完后去hdfs查看，并没有看到数据  关闭hbase之后重启发现有数据证明写入数据的时候先写到缓存中再进行hbase关闭的时候缓存中的数据会flush
> > >
> > > > data: 存储数据目录
> > > >
> > > > default:namespace名称
> > > >
> > > > t1:表名
> > > >
> > > > 一长串数字与字母：region的编号
> > > >
> > > > f1:列簇
> > > >
> > > > 一长串数字与字母：列簇存储的文件----------Hfile    每一个Hfile文件对应一个列簇
> >
> > 删
> >
> > > 清空表数据
> > >
> > > ```shell
> > > truncate 't1'   #先禁用  再清空   又启用
> > > ```
> > >
> > > 删除表
> > >
> > > ``` shell    
> > > drop 't1'   #先禁用表再删除表
> > > disable 't1'
> > > drop 't1'
> > > ```
> > >
> > > delete
> > >
> > > ```\
> > > delete 't1','r1',
> > > ```
> >
> > 改
> >
> > > 存在列簇的情况下
> > >
> > > ```shell
> > > alter 't1',NAME=>'f1',VERSION=>3    #修改列簇的信息
> > > ```
> > >
> > > 不存在列簇f1的情况下
> > >
> > > ```shell
> > > alter 't1',NAME=>'f1',VERSION=>3     #添加一个f1的列簇
> > > ```
> >
> > 查
> >
> > > 全表扫描的命令
> > >
> > > ```shell
> > > scan 't1'       #t1为表名
> > > scan 't1',{STARTROW=>'r1',ENDROW=>'r2'}    #查询r1到r2行之间的所有行
> > > scan 't1',{COLUMNS=>'f1:c1'}
> > > ```
> > >
> > > get查看
> > >
> > > ```shell
> > > get 't1','r1',{TIMERANGE=>[]}     #timerange 时间戳
> > > ```
> > >
> > > 



