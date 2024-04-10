+++
title = 'When more CPUs do not help your problem with CPUs'
date = 2024-04-10T14:47:56+01:00
+++

[This](https://discord.com/channels/566333122615181327/566334526540742656/1214860027668467772) is an interesting problem on Discourse where the symptoms belie the cause. Here, a very beefy Spark cluster is taking a long time process (admittedly) a large amount of data. However, it's the CPUs that are getting hammered. 

<img style="float: left; padding: 10px 10px 10px 13px" src="../cpu_exhaustion/spark_cpu.png"  width="400" title="Insanely high CPU usage" alt="Insanely high CPU usage" >

Insanely high CPU usage

The temptation at this point is to add more CPU resources but this won't help much.

When your Spark jobs that are not computationally intensive are using large amounts of CPU, there's an obvious suspect. Let's check time spent in Garbage Collection:

<img style="float: left; padding: 10px 10px 10px 13px" src="../cpu_exhaustion/spark_gc.png"  width="400" title="Insanely large GC Times" alt="Insanely large GC Times" >

Insanely large GC Times

Shuffle per worker seems modest but look at those GC Times. In a five hours job, nearly two hours is spent just garbage collecting. 

And this is something of a surprise to people new to Spark. Sure, it delivers on its promise to process more data than can fit in memory but if you want it to be performant, you need to give it as much memory as possible.  
