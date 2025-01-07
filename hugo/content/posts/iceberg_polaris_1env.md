+++
title = 'Iceberg and Polaris on a single laptop'
date = 2025-01-07T08:38:03Z
draft = false
+++

If you're developing Apache Iceberg code, you probably need a catalog and you're probably using Spark (other engines are available).
So, how do you easily do all this on a single developer laptop?


![Rotterdam docks by Bernd Dittrich](../docker_maven/bernd-dittrich-bUnsDLFRNWc-unsplash.jpg)

Thanks to wonders of Docker, we can have a whole ecosystem that can be easily spun up. 
Even better, with [Fabric8's `docker-maven-plugin`](https://dmp.fabric8.io/) you can start a cluster, run your tests and tear the cluster down all as part of your build.
It's free, [open source](https://github.com/fabric8io/docker-maven-plugin/tree/master) and integrates with the most popular buld tool in the Java ecosystem.
