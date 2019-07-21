[TOC]

## Hadoop的核心就是HDFS和MapReduce

## HDFS

（Hadoop Distributed File System，Hadoop分布式文件系统），它是一个高度容错性的系统，适合部署在廉价的机器上。HDFS能提供高吞吐量的数据访问，适合那些有着超大数据集（large data set）的应用程序。

### HDFS的设计特点

1、大数据文件，非常适合上T级别的大文件或者一堆大数据文件的存储，如果文件只有几个G甚至更小就没啥意思了。
2、文件分块存储，HDFS会将一个完整的大文件平均分块存储到不同计算器上，它的意义在于读取文件时可以同时从多个主机取不同区块的文件，多主机读取比单主机读取效率要高得多得都。
3、流式数据访问，一次写入多次读写，这种模式跟传统文件不同，它不支持动态改变文件内容，而是要求让文件一次写入就不做变化，要变化也只能在文件末添加内容。
4、廉价硬件，HDFS可以应用在普通PC机上，这种机制能够让给一些公司用几十台廉价的计算机就可以撑起一个大数据集群。
5、硬件故障，HDFS认为所有计算机都可能会出问题，为了防止某个主机失效读取不到该主机的块文件，它将同一个文件块副本分配到其它某几个主机上，如果其中一台主机失效，可以迅速找另一块副本取文件。

### HDFS的关键元素

Block：将一个文件进行分块，通常是64M。
NameNode：保存整个文件系统的目录信息、文件信息及分块信息，这是由唯一一台主机专门保存，当然这台主机如果出错，NameNode就失效了。在Hadoop2.*开始支持activity-standy模式----如果主NameNode失效，启动备用主机运行NameNode。
DataNode：分布在廉价的计算机上，用于存储Block块文件。



![img](https://upload-images.jianshu.io/upload_images/1938547-fa8570a75f6f0b87.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/664/format/webp)

## MapReduce

通俗说MapReduce是一套从海量·源数据提取分析元素最后返回结果集的编程模型，将文件分布式存储到硬盘是第一步，而从海量数据中提取分析我们需要的内容就是MapReduce做的事了。

MapReduce的基本原理就是：将大的数据分析分成小块逐个分析，最后再将提取出来的数据汇总分析，最终获得我们想要的内容。当然怎么分块分析，怎么做Reduce操作非常复杂，Hadoop已经提供了数据分析的实现，我们只需要编写简单的需求命令即可达成我们想要的数据。

## hadoop框架

Hadoop使用主/从（Master/Slave）架构，主要角色有NameNode，DataNode，secondary NameNode，JobTracker，TaskTracker组成。

其中NameNode，secondary NameNode，JobTracker运行在Master节点上，DataNode和TaskTracker运行在Slave节点上。

### 1，NameNode

NameNode是HDFS的守护程序，负责记录文件是如何分割成数据块的，以及这些数据块被存储到哪些数据节点上。它的功能是对内存及I/O进行集中管理。

### 2，DataNode

集群中每个从服务器都运行一个DataNode后台程序，后台程序负责把HDFS数据块读写到本地文件系统。需要读写数据时，由NameNode告诉客户端去哪个DataNode进行具体的读写操作。

### 3，Secondary NameNode

Secondary NameNode是一个用来监控HDFS状态的辅助后台程序，如果NameNode发生问题，可以使用Secondary NameNode作为备用的NameNode。

### 4，JobTracker

JobTracker后台程序用来连接应用程序与Hadoop，用户应用提交到集群后，由JobTracker决定哪个文件处理哪个task执行，一旦某个task失败，JobTracker会自动开启这个task。

### 5，TaskTracker

TaskTracker负责存储数据的DataNode相结合，位于从节点，负责各自的task。