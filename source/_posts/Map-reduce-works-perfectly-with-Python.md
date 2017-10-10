---
title: 用Python在Hadoop上跑MapReduce
date: 2017-09-29 16:15:51
tags:
    - Hadoop
    - Python
    - Docker
    - MapReduce
categories:
    - Big Data
---

# 本文目的
这篇文章主要会给大家介绍一下如何将Python和Hadoop结合起来工作。有接触过MapReduce的朋友都知道，Hadoop的运行环境主要是Java，一般介绍Hadoop和MapReduce的教程和书籍也都是基于Java的。因为我个人对Java并不太感冒，一直以来钟情于Python的简洁实用理念，同时又对MapReduce有兴趣，因此萌生了Python的MapReduce结合的想法。本文也是我经过Google学习他人教程，以及自己实际练习得出来的一些心得，在此分享给各位。

<!--more-->

---
# 环境搭建
首先，你需要有个Hadoop的运行环境，还有Python运行环境。本文主要目的不在分享安装环境，因此有从零开始的朋友，可以先去百度或者Google上搜一下相关教程。下面分享几个相关的教程：
- [Hadoop: Setting up a Single Node Cluster](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html)
- [Hadoop安装教程_单机/伪分布式配置_Hadoop2.6.0/Ubuntu14.04](http://www.powerxing.com/install-hadoop/)
- [使用Docker在本地搭建Hadoop分布式集群](http://www.tashan10.com/yong-dockerda-jian-hadoopwei-fen-bu-shi-ji-qun/)

---
# MapReduce in Python
下面我就来看Python里如何实现mapper和reducer。
## mapper.py
mapper要做的工作就是从`stdin`里读取数据，然后分割成`<key, value>`的pair。这里以最基础的word count为例，`key`就是指文章中拆出来的词，`value`就是指每个词的个数。mapper是不会将相同的词的个数进行统计加和的，那是reducer的工作，因此mapper的输出就是由很多行`<key> 1`组成，下面会看到程序运行实际的结果。

`mapper.py`
```python
#!/usr/bin/env python3

import sys

# 从stdin中获取输出信息
for line in sys.stdin:
    words = line.strip().split(" ")
    for word in words:

        # output format: word'\t'1
        print("{0}\t{1}".format(word, 1))
```
---

## reducer.py
上面提到了，mapper只进行词汇分割，计数为1的工作，那么reducer就是用来将相同词汇的出现次数进行统计加和的工作。同mapper一样，reducer也是从`stdin`中获取输入，然后将结果输出到`stdout`。

`reducer.py`
```python
#!/usr/bin/env python3

import sys

last_word = None
total_count = 0

for line in sys.stdin:
    word, count = line.strip().split("\t")
    count = int(count)
    
    # 如果当前的word不等于上一个，说明开始新的了
    if last_word and last_word != word:
        # 这里输出结果，相当于把结果写进stdout
        print("{0}\t{1}".format(last_word, total_count))
        total_count = 0
    last_word = word
    total_count += count

# 不要忘记最后一个word的输出
if last_word:
    print("{0}\t{1}".format(last_word, total_count))
```
---

# 本地测试
我们先来本地测试一下正确性。
```console
YuanMBP:src Vergil$ echo "东 南 西 北 中 发 白" | demo/mapper.py 
东   1
南   1
西   1
北   1
中   1
发   1
白   1
```

再来看一下reducer：
```console
YuanMBP:src Vergil$ echo "东 南 西 北 东 中 东 发 东 白" | demo/mapper.py | sort | demo/reducer.py 
东   4
中   1
北   1
南   1
发   1
白   1
西   1
```

请注意，这里我在运行mapper和reducer之间加入了一个sort，这是必须的，了解map-reduce工作原理的朋友应该都明白这里为什么有一个sort，如果不加的话，我们的东风杠就识别不了啦。我们在写mapper和reducer的时候是不需要关注它排序的问题，因为Hadoop中的map-reduce会自动进行排序。
```console
YuanMBP:src Vergil$ echo "东 南 西 北 东 中 东 发 东 白" | demo/mapper.py | demo/reducer.py 
东   1
南   1
西   1
北   1
东   1
中   1
东   1
发   1
东   1
白   1
```

---
# 在Hadoop上跑程序

## 准备测试数据

我的运行环境是在Docker上搭建的，首先我们需要先把用来测试的文章放到HDFS里
```console
root@hadoop-master:~/src/demo# hdfs dfs -put words1.txt /input
root@hadoop-master:~/src/demo# hdfs dfs -ls /input
Found 1 item
-rw-r--r--   2 root supergroup        127 2017-09-30 03:57 /input/words1.txt
root@hadoop-master:~/src/demo#
```

`words1.txt`
```
Let me write down something trivial
something that is not important
something that looks like bullshit
yes that is what I want
```
我就随便写了几句放在`words1.txt`里，看看运行结果是否正确。

---
## 运行MapReduce

用过Java版Hadoop的朋友，应该还有印象如何编译运行吧，其实就和运行Java程序的过程很像。但这里用Python来执行，就稍微有些不太一样了。首先我们需要用到一个`hadoop-streaming-2.x.x.jar`这样的一个工具，这里xx代表版本号。它的具体解释可以参考Hadoop官方给的[Document](http://hadoop.apache.org/docs/r1.2.1/streaming.html#Hadoop+Streaming)，我这里就做个简单的介绍。**Hadoop Streaming**是Hadoop提供的一个工具，可以让你以任意的可执行程序或脚本，来创建和运行MapReduce，这里官网给了一个简单的例子：
```console
$HADOOP_HOME/bin/hadoop  jar $HADOOP_HOME/hadoop-streaming.jar \
    -input myInputDirs \
    -output myOutputDir \
    -mapper /bin/cat \
    -reducer /bin/wc
```

---
现在，我们来用Hadoop Streaming来运行自己的程序。
```console
root@hadoop-master:~/src/demo# hadoop jar ../hadoop-streaming-2.7.2.jar \
> -input /input \
> -output /output \
> -mapper mapper.py \
> -reducer reducer.py \
> -file mapper.py \
> -file reducer.py
```

---
这里有两点需要注意：
- 后面的两个`-file`是必须要加的，否则程序无法顺利运行；
- `mapper.py`和`reducer.py`要提前记得赋予它们可执行的属性。

接下来，我们来验收一下程序的结果：
```console
root@hadoop-master:~/src/demo# hdfs dfs -ls /output
Found 2 items
-rw-r--r--   2 root supergroup          0 2017-09-30 04:31 /output/_SUCCESS
-rw-r--r--   2 root supergroup        128 2017-09-30 04:31 /output/part-00000
root@hadoop-master:~/src/demo# hdfs dfs -cat /output/part*
I   1
Let 1
bullshit    1
down    1
important   1
is  2
like    1
looks   1
me  1
not 1
something   3
that    3
trivial 1
want    1
what    1
write   1
yes 1
root@hadoop-master:~/src/demo#
```

---
试一下多文件看看有没有问题，我将words1.txt复制出一模一样的两份，也就是现在有三份相同的输入文件，再来跑一遍试试：
```console
root@hadoop-master:~/src/demo# hdfs dfs -rm -r /output
17/09/30 04:48:37 INFO fs.TrashPolicyDefault: Namenode trash configuration: Deletion interval = 0 minutes, Emptier interval = 0 minutes.
Deleted /output
root@hadoop-master:~/src/demo# ./run_script.sh /input /output mapper.py reducer.py

Running python in Hadoop by hadoop streaming...
```
这里我自己做了`run_script.sh`这样子一个shell script，用来缩短执行命令的长度，不然每次都要输入那么长真的好麻烦……
```console
root@hadoop-master:~/src/demo# hdfs dfs -cat /output/part*
I   3
Let 3
bullshit    3
down    3
important   3
is  6
like    3
looks   3
me  3
not 3
something   9
that    9
trivial 3
want    3
what    3
write   3
yes 3
```

看上去没有任何问题。

---
# 后记
更多的关于Hadoop Streaming的内容，还希望大家去官网文档中查阅。例如像分配map和reduce的数量，设置partitioner，这样的参数都可以通过Hadoop Streaming来调整，还是很有意思的。

在下一篇关于MapReduce的文章中我会介绍一个相比较于word count复杂一点的例子，依然是用Python和Hadoop的结合。

---
# Related Links
- [Writing an hadoop mapreduce program in python](http://www.michael-noll.com/tutorials/writing-an-hadoop-mapreduce-program-in-python/)
- [Hadoop Streaming Documentation](http://hadoop.apache.org/docs/r1.2.1/streaming.html#Hadoop+Streaming)