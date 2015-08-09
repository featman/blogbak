title: 基于Hadoop平台的局域网和广域网环境下的处理性能比较  
date: 2015-1-11 14:01:01
categories: hadoop 
tags: [hadoop,n2n,vpn] 

---

## 测试目标
检验在局域网环境下的hadoop平台与广域网环境下通过VPN组建的hadoop平台在处理性能上的差异。
实际上检测网络通信情况，通过hadoop的terasort示例工具来做排序的压力测试，通过对比排序所用时间来确定差异。

<!--more-->
## 测试环境

hadoop-1.0.0
三台vmware下的ubuntu12.04，一台为Master，两台为Slave。

## 实验步骤
### 测试局域网环境下的计算性能
a.首先搭建hadoop平台

b.搭建好hadoop平台后，通过master启动hadoop

c.本人hadoop装在/usr/hadoop下，首先先通过teragen工具产生一定的随机数据

d.在/usr/hadoop下运行bin/hadoop jar hadoop-examples-1.0.0.jar

teragen 10000000 input
(即产生1G的文本数据，每行有100个字节，所以总共有一千万行的随机数据，产生的数据存入到hdfs用户目录的input下)
(附：有时候输入命令，无反应。

出现如下错误：

　　org.apache.hadoop.dfs.SafeModeException: Cannot delete ..., Name node is in safe mode

　　在分布式文件系统启动的时候，开始的时候会有安全模式，当分布式文件系统处于安全模式的情况下，文件系统中的内容不允许修改也不允许删除，直到安全模式结束。安全模式主要是为了系统启动的时候检查各个DataNode上数据块的有效性，同时根据策略必要的复制或者删除部分数据块。运行期通过命令也可以进入安全模式。在实践过程中，系统启动的时候去修改和删除文件也会有安全模式不允许修改的出错提示，只需要等待一会儿即可。

　　解决方案是：关闭安全模式
　　
　　
```
hadoop dfsadmin -safemode leave
```
e.运行结束后，然后运行bin/hadoop jar hadoop-examples-1.0.0.jar terasort input output
(这条命令是让hadoop执行文本数据的排序，从input读，最终产生的文件也在hdfs的output文件夹下，会生成一个单一的文件。)

f.同时还可用/bin/hadoop jar hadoop-examples-1.0.0.jar teravalidate output validate 来执行检验排序的正确与否。
#### 结果
（1）通过6次重复实验，可根据e运行的结果如下：

第一次：5分3秒
第二次：4分34秒
第三次：3分44秒
第四次：4分20秒
第五次：4分20秒
第六次：3分59秒

（2）同时观察到两个Slave节点在map和reduce的过程中，cpu和内存充分被利用，cpu占用率大概在60%-75%之间，内存占用百分之90以上。而当总的运算进度已达到百分之70多的时候，有一个节点的cpu和内存已经空闲，说明他自己的运算工作已经完成，另一个节点在总进度达到100%时cpu和内存变为空闲。另外，在hadoop刚刚开始运算的时候，Master瞬间cpu占用比高，之后运算空闲。这也验证了Hadoop平台的主从架构。
### 测试广域网环境下的计算性能
广域网测试延迟巨大，处理速度差异巨大。
## 结论
分布式的处理架构，对网络带宽的要求苛刻。一般内网中组建的hadoop平台，网络带宽应该在100M。如果公网能找到这样的环境，应该也是可以的。但是这种环境搭建成本太高，所以hadoop平台在广域网范围下的应用失败。
（据说，处理1TB的数据，需要在网络中传输3TB的数据。）

## 附录

```
hadoop@Master:/usr/hadoop$ bin/hadoop jar hadoop-examples-1.0.0.jar teragen 10000000 in4
Warning: $HADOOP_HOME is deprecated.


Generating 10000000 using 2 maps with step of 5000000
15/01/09 18:35:30 INFO mapred.JobClient: Running job: job_201501101813_0020
15/01/09 18:35:31 INFO mapred.JobClient:  map 0% reduce 0%
15/01/09 18:35:47 INFO mapred.JobClient:  map 17% reduce 0%
15/01/09 18:35:50 INFO mapred.JobClient:  map 40% reduce 0%
15/01/09 18:35:53 INFO mapred.JobClient:  map 46% reduce 0%
15/01/09 18:35:54 INFO mapred.JobClient:  map 56% reduce 0%
15/01/09 18:35:57 INFO mapred.JobClient:  map 61% reduce 0%
15/01/09 18:35:59 INFO mapred.JobClient:  map 75% reduce 0%
15/01/09 18:36:00 INFO mapred.JobClient:  map 84% reduce 0%
15/01/09 18:36:03 INFO mapred.JobClient:  map 95% reduce 0%
15/01/09 18:36:12 INFO mapred.JobClient:  map 100% reduce 0%
15/01/09 18:36:17 INFO mapred.JobClient: Job complete: job_201501101813_0020
15/01/09 18:36:17 INFO mapred.JobClient: Counters: 19
15/01/09 18:36:17 INFO mapred.JobClient:   Job Counters
15/01/09 18:36:17 INFO mapred.JobClient:     SLOTS_MILLIS_MAPS=55752
15/01/09 18:36:17 INFO mapred.JobClient:     Total time spent by all reduces waiting after reserving slots (ms)=0
15/01/09 18:36:17 INFO mapred.JobClient:     Total time spent by all maps waiting after reserving slots (ms)=0
15/01/09 18:36:17 INFO mapred.JobClient:     Launched map tasks=2
15/01/09 18:36:17 INFO mapred.JobClient:     SLOTS_MILLIS_REDUCES=0
15/01/09 18:36:17 INFO mapred.JobClient:   File Input Format Counters
15/01/09 18:36:17 INFO mapred.JobClient:     Bytes Read=0
15/01/09 18:36:17 INFO mapred.JobClient:   File Output Format Counters
15/01/09 18:36:17 INFO mapred.JobClient:     Bytes Written=1000000000
15/01/09 18:36:17 INFO mapred.JobClient:   FileSystemCounters
15/01/09 18:36:17 INFO mapred.JobClient:     HDFS_BYTES_READ=167
15/01/09 18:36:17 INFO mapred.JobClient:     FILE_BYTES_WRITTEN=42624
15/01/09 18:36:17 INFO mapred.JobClient:     HDFS_BYTES_WRITTEN=1000000000
15/01/09 18:36:17 INFO mapred.JobClient:   Map-Reduce Framework
15/01/09 18:36:17 INFO mapred.JobClient:     Map input records=10000000
15/01/09 18:36:17 INFO mapred.JobClient:     Physical memory (bytes) snapshot=77443072
15/01/09 18:36:17 INFO mapred.JobClient:     Spilled Records=0
15/01/09 18:36:17 INFO mapred.JobClient:     CPU time spent (ms)=14870
15/01/09 18:36:17 INFO mapred.JobClient:     Total committed heap usage (bytes)=31850496
15/01/09 18:36:17 INFO mapred.JobClient:     Virtual memory (bytes) snapshot=694943744
15/01/09 18:36:17 INFO mapred.JobClient:     Map input bytes=10000000
15/01/09 18:36:17 INFO mapred.JobClient:     Map output records=10000000
15/01/09 18:36:17 INFO mapred.JobClient:     SPLIT_RAW_BYTES=167
----------------------------------------------------------------------------------------------------------------------------------------------------------------------
hadoop@Master:/usr/hadoop$ bin/hadoop jar hadoop-examples-1.0.0.jar terasort in4                                                                              out4
Warning: $HADOOP_HOME is deprecated.


15/01/09 18:36:49 INFO terasort.TeraSort: starting
15/01/09 18:36:50 INFO mapred.FileInputFormat: Total input paths to process : 2
15/01/09 18:36:50 INFO util.NativeCodeLoader: Loaded the native-hadoop library
15/01/09 18:36:50 INFO zlib.ZlibFactory: Successfully loaded & initialized native-zlib library
15/01/09 18:36:50 INFO compress.CodecPool: Got brand-new compressor
Making 1 from 100000 records
Step size is 100000.0
15/01/09 18:36:50 INFO mapred.FileInputFormat: Total input paths to process : 2
15/01/09 18:36:51 INFO mapred.JobClient: Running job: job_201501101813_0021
15/01/09 18:36:52 INFO mapred.JobClient:  map 0% reduce 0%
15/01/09 18:37:09 INFO mapred.JobClient:  map 10% reduce 0%
15/01/09 18:37:11 INFO mapred.JobClient:  map 21% reduce 0%
15/01/09 18:37:12 INFO mapred.JobClient:  map 22% reduce 0%
15/01/09 18:37:13 INFO mapred.JobClient:  map 24% reduce 0%
15/01/09 18:37:17 INFO mapred.JobClient:  map 25% reduce 0%
15/01/09 18:37:23 INFO mapred.JobClient:  map 30% reduce 0%
15/01/09 18:37:26 INFO mapred.JobClient:  map 35% reduce 0%
15/01/09 18:37:28 INFO mapred.JobClient:  map 36% reduce 0%
15/01/09 18:37:33 INFO mapred.JobClient:  map 37% reduce 2%
15/01/09 18:37:39 INFO mapred.JobClient:  map 37% reduce 4%
15/01/09 18:37:42 INFO mapred.JobClient:  map 37% reduce 6%
15/01/09 18:37:47 INFO mapred.JobClient:  map 37% reduce 8%
15/01/09 18:37:48 INFO mapred.JobClient:  map 37% reduce 10%
15/01/09 18:37:49 INFO mapred.JobClient:  map 44% reduce 10%
15/01/09 18:37:52 INFO mapred.JobClient:  map 47% reduce 10%
15/01/09 18:37:59 INFO mapred.JobClient:  map 49% reduce 10%
15/01/09 18:38:01 INFO mapred.JobClient:  map 56% reduce 10%
15/01/09 18:38:03 INFO mapred.JobClient:  map 60% reduce 10%
15/01/09 18:38:06 INFO mapred.JobClient:  map 61% reduce 12%
15/01/09 18:38:10 INFO mapred.JobClient:  map 62% reduce 12%
15/01/09 18:38:16 INFO mapred.JobClient:  map 66% reduce 14%
15/01/09 18:38:20 INFO mapred.JobClient:  map 68% reduce 16%
15/01/09 18:38:21 INFO mapred.JobClient:  map 69% reduce 16%
15/01/09 18:38:22 INFO mapred.JobClient:  map 69% reduce 18%
15/01/09 18:38:29 INFO mapred.JobClient:  map 73% reduce 20%
15/01/09 18:38:32 INFO mapred.JobClient:  map 83% reduce 20%
15/01/09 18:38:35 INFO mapred.JobClient:  map 87% reduce 20%
15/01/09 18:38:47 INFO mapred.JobClient:  map 93% reduce 25%
15/01/09 18:38:51 INFO mapred.JobClient:  map 95% reduce 25%
15/01/09 18:38:54 INFO mapred.JobClient:  map 98% reduce 25%
15/01/09 18:38:57 INFO mapred.JobClient:  map 99% reduce 27%
15/01/09 18:39:00 INFO mapred.JobClient:  map 100% reduce 29%
15/01/09 18:39:03 INFO mapred.JobClient:  map 100% reduce 31%
15/01/09 18:39:12 INFO mapred.JobClient:  map 100% reduce 33%
15/01/09 18:39:30 INFO mapred.JobClient:  map 100% reduce 67%
15/01/09 18:39:33 INFO mapred.JobClient:  map 100% reduce 69%
15/01/09 18:39:36 INFO mapred.JobClient:  map 100% reduce 71%
15/01/09 18:39:39 INFO mapred.JobClient:  map 100% reduce 73%
15/01/09 18:39:42 INFO mapred.JobClient:  map 100% reduce 75%
15/01/09 18:39:46 INFO mapred.JobClient:  map 100% reduce 77%
15/01/09 18:39:49 INFO mapred.JobClient:  map 100% reduce 78%
15/01/09 18:39:52 INFO mapred.JobClient:  map 100% reduce 79%
15/01/09 18:39:55 INFO mapred.JobClient:  map 100% reduce 81%
15/01/09 18:39:58 INFO mapred.JobClient:  map 100% reduce 83%
15/01/09 18:40:01 INFO mapred.JobClient:  map 100% reduce 85%
15/01/09 18:40:04 INFO mapred.JobClient:  map 100% reduce 86%
15/01/09 18:40:07 INFO mapred.JobClient:  map 100% reduce 88%
15/01/09 18:40:10 INFO mapred.JobClient:  map 100% reduce 90%
15/01/09 18:40:13 INFO mapred.JobClient:  map 100% reduce 92%
15/01/09 18:40:16 INFO mapred.JobClient:  map 100% reduce 94%
15/01/09 18:40:19 INFO mapred.JobClient:  map 100% reduce 96%
15/01/09 18:40:22 INFO mapred.JobClient:  map 100% reduce 98%
15/01/09 18:40:28 INFO mapred.JobClient:  map 100% reduce 100%
15/01/09 18:40:39 INFO mapred.JobClient: Job complete: job_201501101813_0021
15/01/09 18:40:40 INFO mapred.JobClient: Counters: 31
15/01/09 18:40:40 INFO mapred.JobClient:   Job Counters
15/01/09 18:40:40 INFO mapred.JobClient:     Launched reduce tasks=1
15/01/09 18:40:40 INFO mapred.JobClient:     SLOTS_MILLIS_MAPS=432306
15/01/09 18:40:40 INFO mapred.JobClient:     Total time spent by all reduces waiting after reserving slots (ms)=0
15/01/09 18:40:40 INFO mapred.JobClient:     Total time spent by all maps waiting after reserving slots (ms)=0
15/01/09 18:40:40 INFO mapred.JobClient:     Rack-local map tasks=2
15/01/09 18:40:40 INFO mapred.JobClient:     Launched map tasks=18
15/01/09 18:40:40 INFO mapred.JobClient:     Data-local map tasks=16
15/01/09 18:40:40 INFO mapred.JobClient:     SLOTS_MILLIS_REDUCES=193553
15/01/09 18:40:40 INFO mapred.JobClient:   File Input Format Counters
15/01/09 18:40:40 INFO mapred.JobClient:     Bytes Read=1000057358
15/01/09 18:40:40 INFO mapred.JobClient:   File Output Format Counters
15/01/09 18:40:40 INFO mapred.JobClient:     Bytes Written=1000000000
15/01/09 18:40:40 INFO mapred.JobClient:   FileSystemCounters
15/01/09 18:40:40 INFO mapred.JobClient:     FILE_BYTES_READ=2382257412
15/01/09 18:40:40 INFO mapred.JobClient:     HDFS_BYTES_READ=1000059054
15/01/09 18:40:40 INFO mapred.JobClient:     FILE_BYTES_WRITTEN=3402633323
15/01/09 18:40:40 INFO mapred.JobClient:     HDFS_BYTES_WRITTEN=1000000000
15/01/09 18:40:40 INFO mapred.JobClient:   Map-Reduce Framework
15/01/09 18:40:40 INFO mapred.JobClient:     Map output materialized bytes=1020000096
15/01/09 18:40:40 INFO mapred.JobClient:     Map input records=10000000
15/01/09 18:40:40 INFO mapred.JobClient:     Reduce shuffle bytes=1020000096
15/01/09 18:40:40 INFO mapred.JobClient:     Spilled Records=33355441
15/01/09 18:40:40 INFO mapred.JobClient:     Map output bytes=1000000000
15/01/09 18:40:40 INFO mapred.JobClient:     Total committed heap usage (bytes)=2367090688
15/01/09 18:40:40 INFO mapred.JobClient:     CPU time spent (ms)=124990
15/01/09 18:40:40 INFO mapred.JobClient:     Map input bytes=1000000000
15/01/09 18:40:40 INFO mapred.JobClient:     SPLIT_RAW_BYTES=1696
15/01/09 18:40:40 INFO mapred.JobClient:     Combine input records=0
15/01/09 18:40:40 INFO mapred.JobClient:     Reduce input records=10000000
15/01/09 18:40:40 INFO mapred.JobClient:     Reduce input groups=10000000
15/01/09 18:40:40 INFO mapred.JobClient:     Combine output records=0
15/01/09 18:40:40 INFO mapred.JobClient:     Physical memory (bytes) snapshot=2481545216
15/01/09 18:40:40 INFO mapred.JobClient:     Reduce output records=10000000
15/01/09 18:40:40 INFO mapred.JobClient:     Virtual memory (bytes) snapshot=5898289152
15/01/09 18:40:40 INFO mapred.JobClient:     Map output records=10000000
15/01/09 18:40:40 INFO terasort.TeraSort: done
```
