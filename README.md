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

Middleware (Astyanax)
=====================

Cassandra can be used in middleware either as a service or as a LogHandler. For accessing those services you can use astyanax from netflix.

Cassandra Do's and Dont's
=========================

You should use wide rows, but please don't make these rows smaller than 50 to 100 MB.

A column family should use a maximum of about 10000 entries.

You should have a maximum of 500 column families in one keyspace. 

Distribution of nodes
=====================

On a low level you could divide cassandra nodes in racks and dataCenter. Both are possibilities to describe geographical relationship between nodes.

Cassandra and logging - a love story 
====================================

One possible scenario you can use cassandra for is logging.

Example
=======

CREATE TABLE person (
    domain text PRIMARY KEY,
    created_at timestamp,
    names set<text>,
    person_counters map<text, float>,
    email map<text, text>
) WITH comment='person';

Smarter Snitches and Strategies
===============================

Types of Snitches

Simple Snitch- It has the strategy of placing the copy of the row on the next available node walking clockwise through the nodes.

Rack Inferring Snitch- It tries to place copies of rows of different racks in the data center. It will know about the rack and data center and will try to place copies in different racks and data centers. From the IP address, it can determine the data center address and the rack. So the IP address will have to be configured  in such a way that the second unit of IP address will be used to identify the data center. The third unit identifies the rack.

Property file snitch- In rack inferring, it will read the IP address but in case the address is not configured in that way, there is an option of defining this information in a property file. So how do you define this information in a property file?

Cassandra has another Snitch called PropertyFileSnitch which maintains
much more information about nodes within the ring. PropertyFileSnitch
maintains a mapping of node, datacenter, and rack so that we can determine,
for any node, what data center itisin, and whatrack within that datacenter it
isin. Thisinformation isstatically defined in cassandra-topology.properties.

There is also a Strategy thatis made to use the information from a
PropertyFileSnitch called NetworkTopologyStrategy (NTS). The NTS
algorithm isimplemented asfollows:
GetDatacentersfrom strategy options: {DC0:1,DC1:1}
For each data center entry Getreplication factor
Get a list of all endpointsfor this datacenter from the snitch
Create a ringIterator from the datacenter endpointslist and Collect
endpointsto write to – only select an endpointfrom the listfor any given
rack once (distribute acrossracks)

If replication factor has not been met, continue to collect endpointsfrom
the list, allowing racksthat already contain an endpointin the write list
If our replication factor is not equal to our list of endpoints, throw an error
because there are not enough nodesin the data center to meetthe replication
factor

There is a lot of importantstuff going on here (see the presentation slidesfor
more in depth coverage of whatis going on internally), butto keep it brief, the
key difference isthatinstead of iterating over an entire set of nodesin the ring,
NTS creates an iterator for EACH datacenter and places writes discretely for
each. The resultisthatNTS basically breaks each datacenter into it's own
logicalring when it places writes.

Data placement
==============

Cassandra is not“fixed” in the way thatit places data around the ring. It uses
two components, Snitches and Strategies, to determine which nodes will
receive copies of data. 

Snitches define proximity of nodes within the ring.
Strategies use the information Snitches provide them about node proximity
along with an implemented algorithm to collect nodesthat willreceive writes.

Mirrored Offset Tokens
======================

If we choose even tokensfor each data center, in our example token range of 0-
100, we would end up with tokens 0 and 50 for each of our nodes. We can not
assign the exactsame token to more than one node though,so we must offset
tokensthat are in conflict. For the first data center assign 0 and 50, for the
second data center assign 1 and 51, for the third data center, 2 and 52, etc.

Introducing FiloDB. Distributed. Versioned. Columnar.
=====================================================

Distributed – FiloDB is designed from the beginning to run on best-of-breed distributed, scale-out storage platforms such as Apache Cassandra. Queries run in parallel in Apache Spark for scale-out ad-hoc analysis.
Columnar – FiloDB brings breakthrough performance levels for analytical queries by using a columnar storage layout with different space-saving techniques like dictionary compression. The performance is comparable to Parquet, and one to two orders of magnitude faster than Spark on Cassandra 2.x for analytical queries. For the POC performance comparison, please see the cassandra-gdelt repo.

Versioned – At the same time, row-level, column-level operations and built in versioning gives FiloDB far more flexibility than can be achieved using file-based technologies like Parquet alone.

Examples and tests
==================

2 datacenter (N1, N2 + N3, N4), N1 and N3 are seed nodes - replication 2:
Data is hold twice through datacenter.

2 datacenter (N1, N2 + N3, N4), N1 and N3 are seed nodes - replication 1

1 datacenter (N1 + N2), only N1 is seed node - replication 2

Mirrored Offset Tokens
======================

If we choose even tokensfor each data center, in our example token range of 0-
100, we would end up with tokens 0 and 50 for each of our nodes. We can not
assign the exactsame token to more than one node though,so we must offset
tokensthat are in conflict. 

For the first data center assign 0 and 50, for the
second data center assign 1 and 51, for the third data center, 2 and 52, etc.

Building a logging framework with Cassandra
===========================================

1. Cassandra is great for storing and analysing log files.
2. Let's connect Java with our Cassandra store:
    We use Astyanax for connecting.

Why to use Cassandra:
=====================

Cassandra can be integrated with Hadoop, Hive and Apache Spark for batch processing.

In nowadays software application it becomes more and more important to analyse huge amount of log-data. To understand log-data has become critical to find, understand and predict bugs.

Cassandra is a good candidate for real time analytics, however there might be scenarios where you might have to perform batch processing on the stored data. Cassandra can be easily integrated with Hadoop and Hive to achieve this. Also, on-demand in-memory analytics can be done through Apache Spark integration.

New Features in Cassandra 0.7
=============================

Online Schema Changes 

Prior to Cassandra 0.7 adding and removing column families and keyspaces required you to first distribute an updated configuration file to each of your nodes and then execute a rolling restart of your cluster.  That was not too bad, but it was manual and required human intervention, so it was possible to make mistakes.

Cassandra 0.7 solves this problem by exposing the ability to create and drop column families and keyspaces from its client API.  Using the same methods there is limited support for updating existing column families and keyspaces (e.g., increasing the replication factor for a particular keyspace).















