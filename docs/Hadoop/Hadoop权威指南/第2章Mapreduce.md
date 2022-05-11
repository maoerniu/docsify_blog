# 第2章Mapreduce

Hadoop 将 MapReduce 的输入数据划分成等长的小数据块，称为输入分片
最佳的输入分片的大小应该与HDFS的一个块的大小相同，默认是128M