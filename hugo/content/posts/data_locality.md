+++
title = 'The Death of Data Locality?'
date = 2024-01-29T16:39:27Z
draft = true
+++

![Joshua Sortino <](../data_locality.jpg)


Data locality is where the computation and the storage are on the same node. This means we don't need to move huge data sets around the network. But it's a pattern that has fallen out of fashion in recent years.

With a lot of cloud offerings, we lose the data locality that made [Hadoop](https://github.com/apache/hadoop/blob/b2fac14828b69c761858dd7cb9ab17313c28b161/hadoop-hdfs-project/hadoop-hdfs-client/src/main/java/org/apache/hadoop/hdfs/shortcircuit/ShortCircuitReplica.java#L277) such a great framework on which to run Spark some 10 years ago. The cloud providers counter this with a "just rent more nodes" argument. But if you have full control over your infrastructure (say you have an on-prem cluster) throwing away data locality is a huge waste.

Just to recap, data locality gives you doubleplusgood efficiency. Not only does the network not take a hit sending huge amoungs of data from storage to compute nodes, but we retain OS treats like caching. 

What? The OS has built in caching? Have you ever grepped a large directory and then noticed that executing the same command a second time is orders of magnitude faster than the first time? That's because modern operating systems [leave pages in memory](https://www.linuxatemyram.com/) unless there is a reason to dispose of them. Most of the time, there is no point in putting some caching layer on the same machine as where the database lives for simple queries - a strange anti-pattern I've seen in the wild.

Of course, none of this is not available over the network.

Another advantage of having the data locally is that apps can employ a pattern called "[memory mapping](https://javaagile.blogspot.com/2023/11/memories-are-made-of-these.html)". The idea is that as far as the app is concerned, a file is just a location in memory. You read it just like you would a sequence of bytes in RAM. Hadoop takes advantage of this [here](https://github.com/apache/hadoop/blob/40467519399c0e51beb25d6e557c55859382e8cf/hadoop-hdfs-project/hadoop-hdfs/src/main/java/org/apache/hadoop/hdfs/server/datanode/fsdataset/impl/MemoryMappableBlockLoader.java#L79).

Why is memory mapping useful? Well, you don't even need to make [kernel calls](https://javaagile.blogspot.com/2012/07/why-system-calls-are-slow.html) so there is no context switching and certainly no copying data. [Here](https://github.com/PhillHenry/JavaPlayground/blob/bed20b52d89930e83a279480ef8eb4e5b9171441/src/main/java/uk/co/odinconsultants/memory/MemoryMapMain.java) is an example of how to do this in Java. You can prove to yourself that there are no kernel calls by running:

`sudo strace -p $(jstack $(jps | grep MemoryMapMain | awk '{print $1}')  | grep ^\"main | perl -pe s/\].*\//g | perl -pe s/.*\\[//g)`

Note there are kernel calls in setting up the memory mapping but after that, there is nothing as we read the entire file.

So, why have many architects largely abandoned data locality? It's generally a matter of economics as the people at MinIO point out [here](https://min.io/solutions/hdfs-migration). If your data is not homogenous, you might be paying for, say, 32 CPUs on a node that's just being used for storage. An example might be that you have a cluster with 10 years of data but you mainly use that last two years. If the data for the first eight years is living on expensive hardware and rarely accessed, that could be a waste of money.

So, should you use data locality today? The answer, as ever, is "it depends".
