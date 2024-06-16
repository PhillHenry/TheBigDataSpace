+++
title = 'First look at the Unity Catalog'
date = 2024-06-16T10:30:33+01:00
+++

Databrick's Unity Catalog has now been open sourced [GitHub](https://github.com/unitycatalog/unitycatalog). But there's not a huge amount of code there - I counted a mere 133 Java files which were neither test nor examples.

![Claudio Schwarz Unity Through Diversity](../unity_through_diversity2.jpg)

This is not too surprising since it's little more than a REST API over a ([H2](https://github.com/h2database/h2database)) database that stores metadata. It is just a catalog after all.

What is a little more surprising is that "MANAGED table creation is not supported yet." [GitHub](https://github.com/unitycatalog/unitycatalog/blob/873131f57ca6f8e43e0bb707e6a4c1c59facd488/server/src/main/java/io/unitycatalog/server/persist/TableRepository.java#L110) Managed tables are those that Unity, well, manages. That is, the whole life cycle of the table is under its purview. Contrast this with EXTERNAL tables that live elsewhere, who knows where - it's not important. 

Remember that Unity is a catalog not a data store. This confusing the map with the territory [Wikipedia](https://en.wikipedia.org/wiki/Map%E2%80%93territory_relation) is common when coming across metastores. This is perhaps due to the Apache Hive project that is somewhat moribund these days and Hive's metastore which is very much alive. But think of a metastore like web URLs where the locale of the machine hosting the website is irrelevant. Only its domain name is what we care about.

With Databricks recent [acquisition](https://www.databricks.com/blog/databricks-tabular) of Tabular.io (the defacto commercial force behind Apache Iceberg) it will be interesting to see how Iceberg integrates with it, if at all. Iceberg does [not](https://iceberg.apache.org/docs/latest/reliability/) (as demonstrated [here](http://iceberg.thebigdata.space/IcebergCRUD.html)) store its table information in a metastore (unlike [Delta](http://deltalake.thebigdata.space/Crud.html)). All its information is contained in the metadata files. Whether Databricks will encourage Iceberg to be tied to the metastore remains to be seen.
