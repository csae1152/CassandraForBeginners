# CassandraForBeginners

Cassandra is a NoSQL database, which is optimized for high read and write througput. Cassandra belongs to the wide column families.

Like relational databases Cassandra has no ACID functionality. But objects to be stored are written atomar as so called columns.

This behaviour makes it the first choice for example logging entries. (all called methods)

Cassandra can scale in theory infinite. It works as Peer system, adding new nodes on demand.

You can access Cassandra within Java with Hector or Astyanax or a CQL shell.

Replication and consistency
===========================

On creating a new cassandra keyspace you have to choose a new strategy (SimpleStrategy or NetworkTopologyStrategy) as well as a replication factor.

A replication factor 1 means 1 row will be written to 1 node. That means the data record is created only one time.

If you choose replication factor 2 that means you replicate a given data record. Which is in case better :-)

Middleware
==========

Cassandra can be used in middleware either as a service or as a LogHandler. For accessing those services you can use astyanax from netflix.

Cassandra Do's and Dont's
=========================

You should use wide rows, but please don't make these rows smaller than 50 to 100 MB.

A column family should use a maximum of 10000 entries.

You should have a maximum of 500 column families in one keyspace.

Distribution of nodes
=====================

On a low level you could divide cassandra nodes in racks and dataCenter. Both are possibilities to describe geographical relationship between nodes.

Cassandra and logging - a love story 
====================================

One possible scenario you can use cassandra for is logging.









