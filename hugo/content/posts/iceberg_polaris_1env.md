+++
title = 'Cut Costs, Build Faster'
date = 2025-01-07T08:38:03Z
draft = false
+++

It's common to develop against external systems that we share with other devs. 
But wouldn't it be so much better if we could just develop on our laptops with our favourite IDE in an isolated space? 
Well, we can!

In this example, we're going to test the functionality of [Apache Iceberg](https://iceberg.apache.org/).
Note, we 're not going to test its performance.
For that we need too much data to fit on a laptop.

Now, if you're developing an Apache Iceberg app, you probably need a catalog and you're probably using Spark (other engines are available).
How do we get all this working on our dev machine?

![Rotterdam docks by Bernd Dittrich](../docker_maven/bernd-dittrich-bUnsDLFRNWc-unsplash.jpg)

## Why do you care?
At this point, some might say "well, I'm fine developing while connected to the cloud." 
But this doesn't scale.
If you're a lone dev, fine, but for the rest of us we need [Developer Productivity Engineering](https://trishagee.com/presentations/developer-productivity-engineering-whats-in-it-for-me/), "the next big thing in software development".
The idea is to build faster and better. 
This obviously cuts costs.

What we want is a development environment and a full cluster all on one laptop. We don't need any expensive cloud environments, expensive infrastructure guys and the whole thing is easier to manage because everything is replicated and isolated to everybody's machine. Say goodbye to changing development data while other devs are using it!

## Tools of the trade
Thanks to wonders of Docker, we can have a whole ecosystem that we can easily spin up.
Even better, with [Fabric8's `docker-maven-plugin`](https://dmp.fabric8.io/) you can start a cluster, run your tests and tear the cluster down all as part of your build requiring no manual intervention.
It's free, [open source](https://github.com/fabric8io/docker-maven-plugin/tree/master) and integrates with the most popular build tool in the Java ecosystem (Maven).

## Choosing a Catalog for your Test Suite
You can, of course, run Iceberg and Spark in a single JVM using the local disk for tests _most of the time_.
If you want to keep it really simple, you would use the Hadoop catalog backed by the local filesystem. 

[Note that the term 'Hadoop' is a bit of a misnomer here. 
"While the name of the catalog is the _Hadoop catalog_ it works on any file system (or things that look like file systems like cloud object stores)" - Apache Iceberg: The Definitive Guide ]

However, "concurrent writes with a Hadoop catalog are not safe with a local FS or S3" backing it, according to the [official documentation](https://iceberg.apache.org/docs/latest/java-api-quickstart/#using-a-hive-catalog).
So, if you want to test behaviour during concurrent writes, Hadoop is not a good choice.

Another choice might be Hive Metastore since it comes bundled with Spark anyway. But once again, the moment you try to play with concurrent access, you hit [this](https://github.com/apache/iceberg/issues/7847) error. The default Hive Metastore is backed by an in-memory [Apache Derby](https://db.apache.org/derby/) database which is normally great for testing but it "doesn't support the interfaces (or concurrent access) required for Iceberg to work".

You could then use a more production-ready DB like Postgres. But now we're starting to need external processes. This is not necessarily a major problem but Hive Metastore was built for Hive not Iceberg.

> For example, if you change the table scheme directly in hive, it does not change the schema in your iceberg table.  Same with setting table properties." 
>
> [Daniel Weeks](https://apache-iceberg.slack.com/archives/C025PH0G1D4/p1711079937677039?thread_ts=1711078621.965309&cid=C025PH0G1D4) 

So, why use Hive?


## A More Realistic Catalog for your Tests
If you want an environment that is more production-like, you'll need a more industrial grade catalog.
Since we like free and open source solutions, we chose [Apache Polaris](https://polaris.apache.org/) as REST catalogs are becoming ever more popular.
However, if you want to run Polaris in the same JVM to keep things simpler, you might be disappointed. 
Because of the transitive dependencies of Polaris and Spark being incompatible, you'll find yourself in Classpath Hell.
That's when firing up Polaris in Docker becomes a great solution.


## Bring it altogether

Right, so we have our Behaviour Driven Development tests that fire up a Spark instance configured to use the [Iceberg extensions](https://github.com/PhillHenry/IcebergPlayground/blob/f9b2b57d84b3f12df0d8a302ec13b0d5687cc7b8/src/test/scala/uk/co/odinconsultants/SparkForTesting.scala#L38) that uses Polaris as a catalog.
That Polaris catalog is in its own [Docker container](https://github.com/PhillHenry/IcebergPlayground/blob/main/Dockerfile) that [Maven](https://github.com/PhillHenry/IcebergPlayground/blob/f9b2b57d84b3f12df0d8a302ec13b0d5687cc7b8/pom.xml#L219) started.
After the tests pass, Maven kills it.
The output is [automatically generated documentation](https://iceberg.thebigdata.space/) that covers various Iceberg corner cases.

And that's it. No need for expensive infrastructure and a DevOps team. It's all available in one [GitHub repo](https://github.com/PhillHenry/IcebergPlayground/tree/main).

Enjoy!


