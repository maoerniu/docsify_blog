# RocketMQ 2Master2Slave集群搭建

#####集群架构

192.168.100.50 Maste1 broker_a nameserver

192.168.100.51 Master2 broker_b nameserver

192.168.100.52 Slave1 broker_a nameserver

192.168.100.53 Slave2 broker_b 

##### 准备工作

安装jdk1.8和mvn环境，安装unzip工具

##### 安装和配置

1、下载安装包，解压，进入解压目录，用mvn编译安装

``` shell
wget https://archive.apache.org/dist/rocketmq/4.6.0/rocketmq-all-4.6.0-source-release.zip
unzip rocketmq-all-4.6.0-source-release.zip
cd rocketmq-all-4.6.0-source-release/
mvn -Prelease-all -DskipTests clean install -U
```

在distribution/target目录下有一个rocketmq-4.6.0.tar.gz的文件，解压到安装目录

```shell
tar -zxf rocketmq-4.6.0.tar.gz -C /usr/local/include/
mv rocketmq-4.6.0 rocketmq
```

2、 创建目录

```shell
mkdir /usr/local/include/rocketmq/store
mkdir /usr/local/include/rocketmq/store/commitlog
mkdir /usr/local/include/rocketmq/store/consumequeue
mkdir /usr/local/include/rocketmq/store/index
```

3、添加配置文件

```shell
# 备份原默认文件
cd /usr/local/include/rocketmq/conf/2m-2s-async && rename properties properties_bak *properties
```

```shell
vim broker-a.properties
#所属集群
brokerClusterName=rocketmq-cluster
#broker名字，注意此处不同的配置文件填写的不一样 
brokerName=broker-a						# 此处各机器配置不同，50和52为a， 51和53为b
#0 表示 Master，>0 表示 Slave
brokerId=0  									# 50和51位0， 52和53为1
#nameServer地址，分号分割
namesrvAddr=192.168.100.50:9876;192.168.100.51:9876;192.168.100.52:9876
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数 
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭 
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭 
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=10911
#删除文件时间点，默认凌晨 4点
deleteWhen=04
#文件保留时间，默认 48 小时 
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整 
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/usr/local/include/rocketmq/store
#commitLog 存储路径
storePathCommitLog=/usr/local/include/rocketmq/store/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/usr/local/include/rocketmq/store/consumequeue
#消息索引存储路径
storePathIndex=/usr/local/include/rocketmq/store/index
#checkpoint 文件存储路径
storeCheckpoint=/usr/local/include/rocketmq/store/checkpoint
#abort 文件存储路径
abortFile=/usr/local/include/rocketmq/store/abort
#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000
#Broker 的角色
#- ASYNC_MASTER  异步复制Master
#- SYNC_MASTER  同步双写Master
#- SLAVE
brokerRole=ASYNC_MASTER							# 50和51为ASYNC_MASTER， 52和53为ASYNC_SLAVE
#刷盘方式
#- ASYNC_FLUSH  异步刷盘 
#- SYNC_FLUSH  同步刷盘 
flushDiskType=ASYNC_FLUSH
#checkTransactionMessageEnable=false
#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128
```

注：文件名需要修改

​		100.50 broker-a.properties

​		100.51 broker-b.properties

​		100.52 broker-a-s.properties

​		100.53 broker-b-s.properties

4、修改日志文件路径

```shell
mkdir -p /usr/local/include/rocketmq/logs
cd /usr/local/include/rocketmq/conf && sed -i 's#${user.home}#/usr/local/include/rocketmq#g' *.xml
```

5、修改启动参数，生产环境建议不修改

```shell
vim /usr/local/include/rocketmq/bin/runbroker.sh
JAVA_OPT="${JAVA_OPT} -server -Xms1g -Xmx1g -Xmn512m -XX:PermSize=128m -XX:MaxPermSize=320m"
vim /usr/local/include/rocketmq/bin/runserver.sh
JAVA_OPT="${JAVA_OPT} -server -Xms1g -Xmx1g -Xmn512m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"

```

##### 启动nameserver

启动3台nameserver

```shell
# 100.50
cd /usr/local/include/rocketmq/bin && nohup sh mqnamesrv >/dev/null 2>&1 &
# 100.51
cd /usr/local/include/rocketmq/bin && nohup sh mqnamesrv >/dev/null 2>&1 &
# 100.52
cd /usr/local/include/rocketmq/bin && nohup sh mqnamesrv >/dev/null 2>&1 &
```

##### 启动broker

```shell
# 机器启动
# 100.50
cd /usr/local/include/rocketmq/bin && nohup sh mqbroker -c /usr/local/include/rocketmq/conf/2m-2s-async/broker-a.properties >/dev/null 2>&1 &
# 100.51
cd /usr/local/include/rocketmq/bin && nohup sh mqbroker -c /usr/local/include/rocketmq/conf/2m-2s-async/broker-b.properties >/dev/null 2>&1 &
# 100.52
cd /usr/local/include/rocketmq/bin && nohup sh mqbroker -c /usr/local/include/rocketmq/conf/2m-2s-async/broker-a-s.properties >/dev/null 2>&1 &
# 100.53
cd /usr/local/include/rocketmq/bin && nohup sh mqbroker -c /usr/local/include/rocketmq/conf/2m-2s-async/broker-b-s.properties >/dev/null 2>&1 &
```

