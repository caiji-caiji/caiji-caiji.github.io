# 第七章 Flume

## 1. Flume产生背景

### 1.1 问题的产生 

HDFS、MapReduce、HBase 数据都是老师给你的

你在工作中需要代码处理一个业务，老板只会提需求，你要解决 首要是知道数据类型 数据长什么样子

订单数据、用户数据、商品数据都是存储在mysql中，

效率高，是因为select*from goods where name like %s%

某个商品男的看到多,还是女的看的多，数据库里面没有！

也就是说数据库会存储数据，但有些业务也没有数据！

**所以我们要收集数据！**
### 1.2 收集数据
数据来源：

- 文件
- 数据库
- 爬虫-只针对公共数据，就是提供共享大家使用
- 合作、购买（微信 + 京东）流量吸引、数据共享，**导流**

*问题1：微信里面为什么没有淘宝？*

遇到的问题：

一个人看一个商品的次数   
一个人购买一个商品的次数  
哪个多？  
看商品的数据（**冷数据**）数据量大的时候，不会放入数据库中,因为需要计算

> 冷数据：数据一般不会变，历史数据，日志数据

格式种类

- csv comma
- tsv table
- json
- xml
- text
- 行式
- 列式
- 压缩

### 1.3 解决思路
**最终结论：必须要解决数据格式和存储地不统一  
希望：一个组件能解决所有问题**

[http://hadoop.apache.org/](http://hadoop.apache.org/ "Hadoop官网")  
[http://flume.apache.org/](http://flume.apache.org/ "Flume官网")  
主页上没有，是cloudera捐献的

### 1.4 现存问题

大数据处理流程：

- **数据采集**
- 数据ETL
- 数据存储
- 数据计算、分析
- 数据可视化

数据采集难点：  

- 数据源多种多样
- 数据量大，变化快，**流数据**
- 避免重复数据
- 保证数据的质量
- 数据采集的性能

命名：

- flume OG（original generation）版本1.0以前
- flume NG（next generation）1.0之后

## 2. Flume介绍

优点：可靠性、横向性

一般性步骤：  

1. Flume数据采集  
2. MapReduce清洗，计算
3. 存入HBase
4. Hive统计、分析
5. 存入Hive表
6. Sqoop导出
7. MySQL
8. Web可视化

### 2.1 Flume简介

- Event：是Flume数据传输的基本单元，以事件的形式将数据从源头送到最终的目的
- Client：将一个原始log包装成为Events并发送到一个或多个实体
- Agent：将Events从一个节点传输到另外一个节点或最终目的地

其中Agent包含Source，Channel，Sink

- Source：用于对接数据源，接受Event或收集到的Data包装成Event
- Channel：包含Event驱动和轮询两种类型。
> source必须至少和一个channel关联    
> 可以和任何数量的sourc、sink工作  
> 用来缓存进来的Event，将source和sink连接起来，在内存中运行

- sink：存储Event到最终的目的地终端如HDFS、HBase
> 类似JDBC数据缓存池，一条一条没有一次插入性能高

### 2.2 Flume的架构

#### 2.2.1 单一Agent采集数据

![](http://flume.apache.org/_images/DevGuide_image00.png)

#### 2.2.2 多Agent串联采集数据

![](http://flume.apache.org/_images/UserGuide_image03.png)

#### 2.2.3 多Agent合并串联采集数据

![](http://flume.apache.org/_images/UserGuide_image02.png)

#### 2.2.4 多Agent合并串联采集数据

![](http://flume.apache.org/_images/UserGuide_image01.png)


## 3. Flume的安装配置
### 3.1 上传安装包至CentOS下

>解压安装包到hadoop目录下

	tar -zxvf apache-flume-1.9.0-bin.tar.gz -C /usr/hadoop

### 3.2 配置环境变量

	vi /etc/profile

在末尾添加以下代码，保存退出

	export FLUME_HOME=/usr/hadoop/apache-flume-1.9.0-bin
	export PATH=$FLUME_HOME/bin:$PATH

> 生效配置

	source /etc/profile

### 3.3 验证环境

	flume-ng version

> 出现以下结果配置正确

	Flume 1.9.0
	Source code repository: https://git-wip-us.apache.org/repos/asf/flume.git
	Revision: d4fcab4f501d41597bc616921329a4339f73585e
	Compiled by fszabo on Mon Dec 17 20:45:25 CET 2018
	From source with checksum 35db629a3bda49d23e9b3690c80737f9

### 3.4 配置Flume文件

规则：  
1. 指定Agent的名称以及指定Agent的各个组件的名称  
2. 指定Source  
3. 指定Channel  
4. 指定Sink  
5. 指定Source、Channel、Sink之间的关系  


[https://centos.pkgs.org/](https://centos.pkgs.org/)  

>下载 telnet 安装包并进行安装    

	rpm -ivh your-package  

> 在Flume下创建和修改配置文件

	mkdir agent
	vi netcat-logger.properties

> 配置Agent名称、Source、Channel、Sink的名称

	# 配置Agent名称、Source、Channel、Sink的名称
	a1.sources=r1
	a1.channels=c1
	a1.sinks=k1
	
	# 配置Channel组件属性c1
	a1.channels.c1.type=memory
	a1.channels.c1.capacity=1000
	
	# 配置Source组件属性r1
	a1.sources.r1.type=netcat
	a1.sources.r1.bind=localhost
	a1.sources.r1.port=8888
	
	# 配置Sink组件属性k1
	a1.sinks.k1.type=logger
	
	#连接关系
	a1.sources.r1.channels=c1
	a1.sinks.k1.channel=c1

> 启动Agent去采集数据

	-c conf:指定flume自身配置文件所在目录
	-n a1：指定agent的名字
	-f agent/netcat-logger.properties：指定采集规则目录
	-D：Java配置参数

>输入以下命令：

	flume-ng agent -c conf -n a1 -f ../agent/netcat-logger.properties -Dflume.root.logger=INFO,console

![TIM截图20191130005444.png](https://i.loli.net/2019/11/30/EvLnzGOMsKStR6Q.png)

>使用telnet命令，输入“hello”

![TIM截图20191130005604.png](https://i.loli.net/2019/11/30/8NtlnIWJ97cQUoi.png)

>生成event

![TIM截图20191130005651.png](https://i.loli.net/2019/11/30/mhtDPAX6sQfebiw.png)



## 4. Flume组件

### 4.1 **Source**

Flume中常用的Source有NetCat，Avro，Exec，Spooling Directory，Taildir，也可以根据业务场景的需要自定义Source,具体介绍如下。

 

#### **4.1.1 NetCat Source**

NetCat Source可以使用TCP和UDP两种协议方式，使用方法基本相同，通过监听指定的IP和端口来传输数据，它会将监听到的每一行数据转化成一个Event写入到Channel中。（必须参数以@标示，下类同）

channels@ –

type@ – 类型指定为：netcat

bind@ – 绑定机器名或IP地址

port@ – 端口号

max-line-length

 

| Property Name   | Default        | Description                           |
| --------------- | -------------- | ------------------------------------- |
| channels@       | –              |                                       |
| type@           | –              | 类型指定为：netcat                    |
| bind@           | –              | 绑定机器名或IP地址                    |
| port@           | –              | 端口号                                |
| max-line-length | 512            | 一行的最大字节数                      |
| ack-every-event | true           | 对成功接受的Event返回OK               |
| selector.type   | replicating    | 选择器类型replicating or multiplexing |
| selector.*      | 选择器相关参数 |                                       |
| interceptors    | –              | 拦截器列表，多个以空格分隔            |
| interceptors.*  | 拦截器相关参数 |                                       |

 

#### **4.1.2 Avro Source**

不同主机上的Agent通过网络传输数据可使用的Source，一般是接受Avro client的数据，或和是上一级Agent的Avro Sink成对存在。

| Property Name    | Default | Description                                                |
| ---------------- | ------- | ---------------------------------------------------------- |
| channels@        | –       |                                                            |
| type@            | –       | 类型指定为：avro                                           |
| bind@            | –       | 监听的主机名或IP地址                                       |
| port@            | –       | 端口号                                                     |
| threads          | –       | 传输可使用的最大线程数                                     |
| selector.type    |         |                                                            |
| selector.*       |         |                                                            |
| interceptors     | –       | 拦截器列表                                                 |
| interceptors.*   |         |                                                            |
| compression-type | none    | 可设置为“none”  或 “deflate”. 压缩类型需要和AvroSource匹配 |

#### 4.1.3 **Exec Source**

Exec source通过执行给定的Unix命令的传输结果数据，如cat，tail -F等，实时性比较高，但是一旦Agent进程出现问题，可能会导致数据的丢失。

| Property Name   | Default     | Description                           |
| --------------- | ----------- | ------------------------------------- |
| channels@       | –           |                                       |
| type@           | –           | 类型指定为：exec                      |
| command@        | –           | 需要去执行的命令                      |
| shell           | –           | 运行命令的shell脚本文件               |
| restartThrottle | 10000       | 尝试重启的超时时间                    |
| restart         | false       | 如果命令执行失败，是否重启            |
| logStdErr       | false       | 是否记录错误日志                      |
| batchSize       | 20          | 批次写入channel的最大日志数量         |
| batchTimeout    | 3000        | 批次写入数据的最大等待时间（毫秒）    |
| selector.type   | replicating | 选择器类型replicating or multiplexing |
| selector.*      |             | 选择器其他参数                        |
| interceptors    | –           | 拦截器列表，多个空格分隔              |
| interceptors.*  |             |                                       |

#### **4.1.4 Spooling Directory Source**

通过监控一个文件夹将新增文件内容转换成Event传输数据，特点是不会丢失数据，使用Spooling Directory Source需要注意的两点是:

1.不能对被监控的文件夹下的新增的文件做出任何更改

2.新增到监控文件夹的文件名称必须是唯一的。由于是对整个新增文件的监控，Spooling Directory Source的实时性相对较低，不过可以采用对文件高粒度分割达到近似实时。

| Property Name     | Default     | Description                                                  |
| ----------------- | ----------- | ------------------------------------------------------------ |
| channels@         | –           |                                                              |
| type@             | –           | 类型指定：spooldir.                                          |
| spoolDir@         | –           | 被监控的文件夹目录                                           |
| fileSuffix        | .COMPLETED  | 完成数据传输的文件后缀标志                                   |
| deletePolicy      | never       | 删除已经完成数据传输的文件时间：never or immediate           |
| fileHeader        | false       | 是否在header中添加文件的完整路径信息                         |
| fileHeaderKey     | file        | 如果header中添加文件的完整路径信息时key的名称                |
| basenameHeader    | false       | 是否在header中添加文件的基本名称信息                         |
| basenameHeaderKey | basename    | 如果header中添加文件的基本名称信息时key的名称                |
| includePattern    | ^.*$        | 使用正则来匹配新增文件需要被传输数据的文件                   |
| ignorePattern     | ^$          | 使用正则来忽略新增的文件                                     |
| trackerDir        | .flumespool | 存储元数据信息目录                                           |
| consumeOrder      | oldest      | 文件消费顺序：oldest, youngest and random.                   |
| maxBackoff        | 4000        | 如果channel容量不足，尝试写入的超时时间，如果仍然不能写入，则会抛出ChannelException |
| batchSize         | 100         | 批次处理粒度                                                 |
| inputCharset      | UTF-8       | 输入码表格式                                                 |
| decodeErrorPolicy | FAIL        | 遇到不可解码字符后的处理方式：FAIL，REPLACE，IGNORE          |
| selector.type     | replicating | 选择器类型：replicating or multiplexing                      |
| selector.*        |             | 选择器其他参数                                               |
| interceptors      | –           | 拦截器列表，空格分隔                                         |
| interceptors.*    |             |                                                              |

#### 4.1.5 **Taildir Source**

可以实时的监控指定一个或多个文件中的新增内容，由于该方式将数据的偏移量保存在一个指定的json文件中，即使在Agent挂掉或被kill也不会有数据的丢失，需要注意的是，该Source不能在Windows上使用。

| Property Name    | Default                        | Description                                         |
| ---------------- | ------------------------------ | --------------------------------------------------- |
| channels@        | –                              |                                                     |
| type@            | –                              | 指定类型：TAILDIR.                                  |
| filegroups@      | –                              | 文件组的名称，多个空格分隔                          |
| filegroups.@     | –                              | 被监控文件的绝对路径                                |
| positionFile     | ~/.flume/taildir_position.json | 存储数据偏移量路径                                  |
| headers..        | –                              | Header key的名称                                    |
| byteOffsetHeader | false                          | 是否添加字节偏移量到key为‘byteoffset’值中           |
| skipToEnd        | false                          | 当偏移量不能写入到文件时是否跳到文件结尾            |
| idleTimeout      | 120000                         | 关闭没有新增内容的文件超时时间（毫秒）              |
| writePosInterval | 3000                           | 在positionfile 写入每一个文件lastposition的时间间隔 |
| batchSize        | 100                            | 批次处理行数                                        |
| fileHeader       | false                          | 是否添加header存储文件绝对路径                      |
| fileHeaderKey    | file                           | fileHeader启用时，使用的key                         |

 

### **4.2 Channels**

官网提供的Channel有多种类型可供选择，这里介绍Memory Channel和File Channel。

#### **4.2.1 Memory Channel**

Memory Channel是使用内存来存储Event，使用内存的意味着数据传输速率会很快，但是当Agent挂掉后，存储在Channel中的数据将会丢失。

| Property Name                | Default         | Description                                            |
| ---------------------------- | --------------- | ------------------------------------------------------ |
| type@                        | –               | 类型指定为：memory                                     |
| capacity                     | 100             | 存储在channel中的最大容量                              |
| transactionCapacity          | 100             | 从一个source中去或者给一个sink，每个事务中最大的事件数 |
| keep-alive                   | 3               | 对于添加或者删除一个事件的超时的秒钟                   |
| byteCapacityBufferPercentage | 20              | 定义缓存百分比                                         |
| byteCapacity                 | see description | Channel中允许存储的最大字节总数                        |

#### **4.2.2 File Channel**

File Channel使用磁盘来存储Event，速率相对于Memory Channel较慢，但数据不会丢失。

| Property Name        | Default                          | Description                                         |
| -------------------- | -------------------------------- | --------------------------------------------------- |
| type@                | –                                | 类型指定：file.                                     |
| checkpointDir        | ~/.flume/file-channel/checkpoint | checkpoint目录                                      |
| useDualCheckpoints   | false                            | 备份checkpoint，为True，backupCheckpointDir必须设置 |
| backupCheckpointDir  | –                                | 备份checkpoint目录                                  |
| dataDirs             | ~/.flume/file-channel/data       | 数据存储所在的目录设置                              |
| transactionCapacity  | 10000                            | Event存储最大值                                     |
| checkpointInterval   | 30000                            | checkpoint间隔时间                                  |
| maxFileSize          | 2146435071                       | 单一日志最大设置字节数                              |
| minimumRequiredSpace | 524288000                        | 最小的请求闲置空间（以字节为单位）                  |
| capacity             | 1000000                          | Channel最大容量                                     |
| keep-alive           | 3                                | 一个存放操作的等待时间值（秒）                      |
| use-log-replay-v1    | false                            | Expert: 使用老的回复逻辑                            |
| use-fast-replay      | false                            | Expert: 回复不需要队列                              |
| checkpointOnClose    | true                             |                                                     |

### **4.3 Sinks**

Flume常用Sinks有Log Sink，HDFS Sink，Avro Sink，Kafka Sink，当然也可以自定义Sink。

#### 4.3.1 Logger Sink

Logger Sink以INFO 级别的日志记录到log日志中，这种方式通常用于测试。

| Property Name | Default | Description      |
| ------------- | ------- | ---------------- |
| channel@      | –       |                  |
| type＠        | –       | 类型指定：logger |

####  4.3.2 HDFS Sink

Sink数据到HDFS，目前支持text 和 sequence files两种文件格式，支持压缩，并可以对数据进行分区，分桶存储。

| Name                   | Default      | Description                                                  |
| ---------------------- | ------------ | ------------------------------------------------------------ |
| channel@               | –            |                                                              |
| type@                  | –            | 指定类型：hdfs                                               |
| hdfs.path@             | –            | HDFS的路径  hdfs://namenode/flume/webdata/                   |
| hdfs.filePrefix        | FlumeData    | 保存数据文件的前缀名                                         |
| hdfs.fileSuffix        | –            | 保存数据文件的后缀名                                         |
| hdfs.inUsePrefix       | –            | 临时写入的文件前缀名                                         |
| hdfs.inUseSuffix       | .tmp         | 临时写入的文件后缀名                                         |
| hdfs.rollInterval      | 30           | 间隔多长将临时文件滚动成最终目标文件，单位：秒，  如果设置成0，则表示不根据时间来滚动文件 |
| hdfs.rollSize          | 1024         | 当临时文件达到多少（单位：bytes）时，滚动成目标文件， 如果设置成0，则表示不根据临时文件大小来滚动文件 |
| hdfs.rollCount         | 10           | 当 events 数据达到该数量时候，将临时文件滚动成目标文件，如果设置成0，则表示不根据events数据来滚动文件 |
| hdfs.idleTimeout       | 0            | 当目前被打开的临时文件在该参数指定的时间（秒）内，  没有任何数据写入，则将该临时文件关闭并重命名成目标文件 |
| hdfs.batchSize         | 100          | 每个批次刷新到 HDFS 上的 events 数量                         |
| hdfs.codeC             | –            | 文件压缩格式，包括：gzip, bzip2, lzo, lzop, snappy           |
| hdfs.fileType          | SequenceFile | 文件格式，包括：SequenceFile, DataStream,CompressedStre， 当使用DataStream时候，文件不会被压缩，不需要设置hdfs.codeC;  当使用CompressedStream时候，必须设置一个正确的hdfs.codeC值； |
| hdfs.maxOpenFiles      | 5000         | 最大允许打开的HDFS文件数，当打开的文件数达到该值，最早打开的文件将会被关闭 |
| hdfs.minBlockReplicas  | –            | HDFS副本数，写入 HDFS 文件块的最小副本数。  该参数会影响文件的滚动配置，一般将该参数配置成1，才可以按照配置正确滚动文件 |
| hdfs.writeFormat       | Writable     | 写 sequence 文件的格式。包含：Text,  Writable（默认）        |
| hdfs.callTimeout       | 10000        | 执行HDFS操作的超时时间（单位：毫秒）                         |
| hdfs.threadsPoolSize   | 10           | hdfs sink 启动的操作HDFS的线程数                             |
| hdfs.rollTimerPoolSize | 1            | hdfs sink 启动的根据时间滚动文件的线程数                     |
| hdfs.kerberosPrincipal | –            | HDFS安全认证kerberos配置                                     |
| hdfs.kerberosKeytab    | –            | HDFS安全认证kerberos配置                                     |
| hdfs.proxyUser         |              | 代理用户                                                     |
| hdfs.round             | false        | 是否启用时间上的”舍弃”                                       |
| hdfs.roundValue        | 1            | 时间上进行“舍弃”的值                                         |
| hdfs.roundUnit         | second       | 时间上进行”舍弃”的单位，包含：second,minute,hour             |
| hdfs.timeZone          | Local Time   | 时区。                                                       |
| hdfs.useLocalTimeStamp | false        | 是否使用当地时间                                             |
| hdfs.closeTries 0      | Number       | hdfs sink 关闭文件的尝试次数；如果设置为1，当一次关闭文件失败后，hdfs sink将不会再次尝试关闭文件， 这个未关闭的文件将会一直留在那，并且是打开状态； 设置为0，当一次关闭失败后，hdfs sink会继续尝试下一次关闭，直到成功 |
| hdfs.retryInterval     | 180          | hdfs sink 尝试关闭文件的时间间隔， 如果设置为0，表示不尝试，相当于于将hdfs.closeTries设置成1 |
| serializer             | TEXT         | 序列化类型                                                   |
| serializer.*           |              |                                                              |

####  4.3.3 **Avro Sink**

| Property Name         | Default      | Description                                    |
| --------------------- | ------------ | ---------------------------------------------- |
| channel@              | –            |                                                |
| type@                 | –            | 指定类型：avro.                                |
| hostname@             | –            | 主机名或IP                                     |
| port@                 | –            | 端口号                                         |
| batch-size            | 100          | 批次处理Event数                                |
| connect-timeout 20000 | 连接超时时间 |                                                |
| request-timeout       | 20000        | 请求超时时间                                   |
| compression-type      | none         | 压缩类型，“none” or “deflate”.                 |
| compression-level     | 6            | 压缩级别，0表示不压缩，1-9数字越大，压缩比越高 |
| ssl                   | false        | 使用ssl加密                                    |

#### **4.3.4 Kafka Sink**

传输数据到Kafka中，需要注意的是Flume版本和Kafka版本的兼容性

| Property Name           | Default             | Description                                     |
| ----------------------- | ------------------- | ----------------------------------------------- |
| type                    | –                   | 指定类型：org.apache.flume.sink.kafka.KafkaSink |
| kafka.bootstrap.servers | –                   | kafka服务地址                                   |
| kafka.topic             | default-flume-topic | kafka Topic                                     |
| flumeBatchSize          | 100                 | 批次写入kafka Event数                           |

 

## 5. 使用Flume监控文件夹

### 5.1 生成测试文件

在master节点上进行操作

```
mkdir yoseng-log
cd yoseng-log
vi 1.log
```

随意添加内容至1.log文件中

```
cp 1.log 2.log
```

### 5.2 配置文件

在agent文件夹中创建配置文件spooldir-hdfs.properties

```
#agent名， source、channel、sink的名称
agent1.sources = source1
agent1.channels = channel1
agent1.sinks = sink1

#配置source
agent1.sources.source1.type = spooldir
agent1.sources.source1.spoolDir = /usr/yoseng-log
agent1.sources.source1.fileHeader=false

#配置拦截器
agent1.sources.source1.interceptors=i1
agent1.sources.source1.interceptors.i1.type = host
agent1.sources.source1.interceptors.i1.hostHeader = hostname

# 配置sink
agent1.sinks.sink1.type = hdfs
agent1.sinks.sink1.hdfs.path =hdfs://master:8020/flume-log/%y-%m-%d/%H-%M
agent1.sinks.sink1.hdfs.filePrefix = events
#最大同时打开文件的数量
agent1.sinks.sink1.hdfs.maxOpenFiles = 5000
#批次传输的个数
agent1.sinks.sink1.hdfs.batchSize= 100
agent1.sinks.sink1.hdfs.fileType = DataStream
agent1.sinks.sink1.hdfs.writeFormat =Text
#HDFS上的文件达到128M时生成一个文件
agent1.sinks.sink1.hdfs.rollSize = 102400
agent1.sinks.sink1.hdfs.rollCount = 1000000
#HDFS上的文件达到60秒生成一个文件
agent1.sinks.sink1.hdfs.rollInterval = 60
agent1.sinks.sink1.hdfs.useLocalTimeStamp = true

#配置channel
agent1.channels.channel1.type = memory
agent1.channels.channel1.keep-alive=120
agent1.channels.channel1.capacity = 10000
agent1.channels.channel1.transactionCapacity = 100

#组装source、channel、sink
agent1.sources.source1.channels = channel1
agent1.sinks.sink1.channel = channel1
```
### 5.3 运行Flume

复制hadoop配置文件到flume的conf中

```
cp core-site.xml hdfs-site.xml /usr/hadoop/apache-flume-1.9.0-bin/conf
```

进入flume/bin文件夹中

```
flume-ng agent -c conf -n agent1 -f ../agent/spooldir-hdfs.properties
```
可选参数：让控制台显示数据
```
-Dflume.root.logger=INFO,console
```

进入yoseng-log，文件改名为

```
1.log.COMPLETED  2.log.COMPLETED
```

进入web界面查看hdfs目录，查看生成的event的文件

http://192.168.147.10:50070/explorer.html#/flume-log

或通过代码查看
```
hadoop fs -ls /flume-log/
hdfs dfs -ls /flume-log
```

### 5.4 总结

1. spooldir是用来监控文件夹的，文件夹多出一个文件会被监测到
2. 当文件名后缀为 '.COPLETED ' flume不会采集
3. 文件内容发生变化，不会采集



