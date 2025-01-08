+++
title = 'Iceberg and Polaris on a single laptop'
date = 2025-01-07T08:38:03Z
draft = false
+++

If you're developing Apache Iceberg code, you probably need a catalog and you're probably using Spark (other engines are available).
So, how do you easily do all this on a single developer laptop?


![Rotterdam docks by Bernd Dittrich](../docker_maven/bernd-dittrich-bUnsDLFRNWc-unsplash.jpg)

Thanks to wonders of Docker, we can have a whole ecosystem that we can easily spin up.
Even better, with [Fabric8's `docker-maven-plugin`](https://dmp.fabric8.io/) you can start a cluster, run your tests and tear the cluster down all as part of your build requiring no manual intervention.
It's free, [open source](https://github.com/fabric8io/docker-maven-plugin/tree/master) and integrates with the most popular buld tool in the Java ecosystem.

## Choosing a Catalog for your Test Suite

You can, of course, run Iceberg and Spark in a single JVM using the local disk for tests _most of the time_.
If you want to keep it really simple, you would use the Hadoop catalog backed by the local filesystem. 
(Note that the term 'Hadoop' is a bit of a misnomer here. 
"While the name of the catalog is the “Hadoop catalog” it works on any file system (or things that look like file systems like cloud object stores)" [1])
However, "concurrent writes with a Hadoop catalog are not safe with a local FS or S3" backing it, according to the [official documentation](https://iceberg.apache.org/docs/latest/java-api-quickstart/#using-a-hive-catalog).
So, if you want to test behaviour during concurrent writes, Hadoop is not a good choice.

Another choice might be Hive Metastore since it comes bundled with Spark anyway. But once again, the moment you try to play with concurrent access, you hit [this](https://github.com/apache/iceberg/issues/7847) error. The default Hive Metastore is backed by an in-memory [Apche Derby](https://db.apache.org/derby/) database which is normally great for testing but it "doesn't support the interfaces (or concurrent access) required for Iceberg to work".

You could then use a more production-ready DB like Postgres. But now we're starting to need external processes. This is not necessarily a major problem but Hive Metastore was built for Hive not Iceberg.

> For example, if you change the table scheme directly in hive, it does not change the schema in your iceberg table.  Same with setting table properties." 
>
> [Daniel Weeks](https://apache-iceberg.slack.com/archives/C025PH0G1D4/p1711079937677039?thread_ts=1711078621.965309&cid=C025PH0G1D4) 

So, why use Hive?


## A More Realistic Catalog for your Tests

If you want an environment that is more production-like, you'll need a more industrial grade catalog.
Since we like free and open source solutions, we chose [Apache Polaris](https://polaris.apache.org/).
However, if you want to run Polaris in the same JVM to keep things simpler, you might be disappointed. 
Because of the transitive dependencies of Polaris and Spark being incompatible, you'll find yourself in Classpath Hell.
That's when firing up Polaris in Docker becomes a great solution.


[1] Apache Iceberg: The Definitive Guide
