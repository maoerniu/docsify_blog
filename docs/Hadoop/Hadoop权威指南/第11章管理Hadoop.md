# 第11章 管理Hadoop

## 11.1 HDFS
### 11.1.1 永久性数据结构
编辑日志：在概念上是单个实体，在磁盘上是多个文件，每个文件名称由前缀 edits 及后缀组成，后缀指出该文件包含的事务ID

fsimage：每个 fsimage 文件都是文件系统元数据的一个完整的永久性检查点。
如果 namenode 发生故障，最近的 fsimage 文件将被载入到内存以重构元数据的最近状态
每个 fsimage 文件包含文件系统中的所有目录和文件 inode 的序列化信息

编辑日志会无限增长
解决方案是运行辅助 namenode

在大型集群中，辅助namenode 需要运行在一台专用的机器上

### 11.1.2 安全模式
查看 namenode 是否处于安全模式
```shell
% hdfs dfsadmin -safemode get
Safe mode is ON
```
临时退出安全模式
```shell
% hdfs dfsadmin -safemode wait
```

进入安全模式
```shell
% hdfs dfsadmin -safemode enter
Safe mode is ON
```

离开安全模式
```shell
% hdfs dfsadmin -safemode leave
Safe mode is OFF
```

### 11.1.3 日志审计
对日志进行审计是 log4j 在 INFO级别实现的
在默认配置下，此项特性并未启用，可以通过在文件 hadoop-env-sh 中增加以下这行命令，启动日志审计特性
`export HDFS_AUDIT_LOGGER="INFO, REAALUDIT"`

### 11.1.4 工具
1. dfsadmin 工具
可以查找 HDFS 状态信息，也可以在 HDFS 上执行管理操作
2. 文件系统检查 fsck 工具
fsck 工具检查 HDFS 中文件的健康状况
**查找一个文件的数据块**
```shell
% hdfs fsck /user/tom/part-00007 -files -blocks -racks
```
- -files 显示包括文件名称、大小、块数量和健康状况
- -blocks 选项描述文件中各个块的信息，每个块一行
- -racks 选项显示各个块的机架为止和datanode的地址

3. datanode 块扫描器
