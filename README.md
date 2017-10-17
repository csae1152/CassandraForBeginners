# CassandraForBeginners

Cassandra is a NoSQL database, which is optimized for high read and write throughput. Cassandra belongs to the wide column store families.

This makes Cassandra an optimal choice for storing your log output. In combination with Elasticsearch you have an efficient way to store and find information in your log files.

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

You can use netflix astyanax or the datastax driver for connecting with Java. 

Cassandra and Docker
====================

First hint: Use network hosting.

Make a decision how many nodes you will need.

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

What is a Snitch?

A snitch determines which data centers and racks are to be written to and read from. The job of a snitch is to simply determine relative host proximity. Hence, if a node has 3 options to copy the data, which host should it select? Which host should it prefer the data from?

If this is the kind of information the host would like to receive, it will call a particular snitch to see which host is relatively nearer. Snitches gather information about network topology. Depending on what type of snitch is being used, they will be aware about the network topology a person is using and they can efficiently route the requests.

For a single data center cluster, using the default simple snitch is sufficient. Therefore, a simple snitch is nothing but it is a rack unaware snitch. It does not know about the racks and data centers in a cluster. It does not have any information, so it will assume there are no racks and it will choose the nearest host in terms of the network bandwidth available. It wont consider whether it has to prefer a node from the same rack or same data center. But other replicas available are rack aware and there are different types of snitches.

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

Cassandra on Kubernetes - Seed Provider and Snitch
==================================================

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

1 datacenter
Data is hold only once.

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

Cassandra Query Language (CQL) v2.0
===================================

```
<selectStatement> ::= "SELECT" <whatToSelect>
                        "FROM" ( <name> "." )? <name>
                               ( "USING" "CONSISTENCY" <consistencylevel> )?
                               ( "WHERE" <selectWhereClause> )?
                               ( "LIMIT" <integer> )?
                    ;
<whatToSelect> ::= <term> ( "," <term> )*
                 | ("FIRST" <integer> )? "REVERSED"? <columnRange>
                 | "COUNT" "(" <countTarget> ")"
                 ;
<columnRange> ::= <term> ".." <term>
                | "*"
                ;
<countTarget> ::= "*"
                | "1"
                ;
<name> ::= <identifier>
         | <stringLiteral>
         | <integer>
         ;
<selectWhereClause> ::= <relation> ( "AND" <relation> )*
                      | <term> "IN" "(" <term> ( "," <term> )* ")"
                      ;
<relation> ::= <term> <relationOperator> <term>
             ;
<relationOperator> ::= "=" | "<" | ">" | "<=" | ">="
```

The SELECT expression determines which columns will appear in the results and can take a few different forms, as shown above. The simplest is a comma-separated list of column names. Note that column names in Cassandra can be specified with string literals or integers, in addition to identifiers.

It is also possible to specify a range of column names. The range notation consists of start and end column names, separated by two periods (..). The set of columns returned for a range is start and end inclusive. A single star (*) may be used as a range to request “all columns”.

When using a range, it is sometimes useful to limit the number of columns that can be returned as part of each row (since Cassandra is schemaless, it is not necessarily possible to determine ahead of time how many columns will be in the result set). To accomplish this, use the FIRST clause with an integer to specify an upper limit on the number of columns returned per row. The default limit is 10,000 columns.

The REVERSED option causes the sort order of the columns returned to be reversed. This affects the FIRST clause; when limiting the columns returned, the columns at the end of the range will be selected instead of the ones at the beginning of the range.

A SELECT expression may also be COUNT(*). In this case, the result will be only one value: the number of rows which matched the query.

It is worth noting that unlike the projection in a SQL SELECT, there is no guarantee that the results will contain all of the columns specified, because Cassandra is schemaless.

Because of this feature (schemaless) it's not necessary to restart Cassandra after DB changes.

That's one of the biggest advantages in a production environment.

Integrating Cassandra into a data pipeline with Kafka and Spark
===============================================================

First we have to decide wether we will use a lambda architecture or not..

Lambda architecture is a combination between batch- and realtime processing piplines.

For batch processing Hadoop and HDFS is still my prefered solution.

Storm would be the prefered for (near) realtime processing.

For the MyPredictiveFarm application we need a realtime pipline.

Cassandra vs. BigQuery
=====================

BigQuery -> Scanning 133GB of files in less than 19sec.

Let's try to achive this wonderfully result with Cassandra...

Configure Cassandra for working with DataStux-driver.

Advantages of Cassandra:

Read and write throughput both increase linearly as new machines are added, with no downtime or interruption to applications.

Dealing with database changes
=============================

An option would be using flyway for monitoring database changes.

Let's try and use MySQL as a NoSQL solution

We can use a MySQL farm as a key/value store.

Advantages of using MySQL instead of Cassandra

Is it better to use Cassandra with a SSD disk ?
===============================================

Cassandra is optimized for working with revolving disks.

Automatic Data Distribution

Relational databases and some NoSQL systems require manual, developer-driven methods for distributing data across the multiple machines of a database cluster. These techniques are commonly referred to by the term "sharding." Sharding is an old technique that has seen some success in the industry, but is beset by inherent design and operational challenges. In contrast to this legacy architecture, Cassandra automatically distributes and maintains data across a cluster, freeing developers and architects to direct their energies into value-creating application features.

Cassandra has an internal component called a partitioner, which determines how data is distributed across the nodes that make up a database cluster. In short, a partitioner is a hashing mechanism that takes a table row’s primary key, computes a numerical token for it, and then assigns it to one of the nodes in a cluster in a way that is predictable and consistent.

While the partitioner is a configurable property of a Cassandra cluster, the default partitioner is one that randomizes data across a cluster and ensures an even distribution of all data. Cassandra also automatically maintains the balance of data across a cluster even when existing nodes are removed or new nodes are added to a system.



package com.marxmart.persistence;
import com.datastax.driver.core.Cluster;
import com.datastax.driver.core.Host;
import com.datastax.driver.core.Metadata;
import com.datastax.driver.core.Session;
import static java.lang.System.out;
/**
 * Class used for connecting to Cassandra database.
 */
public class CassandraConnector
{
   /** Cassandra Cluster. */
   private Cluster cluster;
   /** Cassandra Session. */
   private Session session;
   /**
    * Connect to Cassandra Cluster specified by provided node IP
    * address and port number.
    *
    * @param node Cluster node IP address.
    * @param port Port of cluster host.
    */
   public void connect(final String node, final int port)
   {
      this.cluster = Cluster.builder().addContactPoint(node).withPort(port).build();
      final Metadata metadata = cluster.getMetadata();
      out.printf("Connected to cluster: %s\n", metadata.getClusterName());
      for (final Host host : metadata.getAllHosts())
      {
         out.printf("Datacenter: %s; Host: %s; Rack: %s\n",
            host.getDatacenter(), host.getAddress(), host.getRack());
      }
      session = cluster.connect();
   }
   /**
    * Provide my Session.
    *
    * @return My session.
    */
   public Session getSession()
   {
      return this.session;
   }
   /** Close cluster. */
   public void close()
   {
      cluster.close();
   }
}

Data distribution and replication

In Cassandra, data distribution and replication go together. Data is organized by table and identified by a primary key, which determines which node the data is stored on. Replicas are copies of rows. When data is first written, it is also referred to as a replica.

Factors influencing replication include:

1. Virtual nodes: assigns data ownership to physical machines.
2. Partitioner: partitions the data across the cluster.
3. Replication strategy: determines the replicas for each row of data.
4. Snitch: defines the topology information that the replication strategy uses to place replicas.

FAULT TOLERANT

Data is automatically replicated to multiple nodes for fault-tolerance. Replication across multiple data centers is supported. Failed nodes can be replaced with no downtime.

Unit Testing
============

The most simple way to test code in Cassandra is probably by writing a unit test. Cassandra uses JUnit as a testing framework and test cases can be found in the test/unit directory. Ideally you’d be able to create a unit test for your implementation that would exclusively cover the class you created (the unit under test). Unfortunately this is not always possible and Cassandra doesn’t have a very mock friendly code base. Often you’ll find yourself in a situation where you have to make use of an embedded Cassandra instance that you’ll be able to interact with in your test. If you want to make use of CQL in your test, you can simply extend CQLTester and use some of the convenient helper methods such as in the following example.

The CAP-Theorem & Tunable Consistency

Possibly the best-known peculiarity of distributed systems is the CAP-Theorem by Dr. E. A. Brewer. It states that of the three attributes, Consistency, Availability and Partition Tolerance, any such system can only fulfill two at a time (take this with a grain of salt).

Without going into too many details, RDBMS have for years focused on the first two, consistency and availability, allowing for such great things as transactions. The whole NoSQL movement (from a 10,000ft view) is essentially about choosing partition tolerance instead of (strong) consistency. This has led to the popular belief that NoSQL databases are completely unsuitable for applications requiring just that. And while this might be true for some, it isn’t for Cassandra.

One of Cassandra’s stand-out features is called “Tunable Consistency”. This means that the programmer can decide if performance or accuracy is more important, on a per-query level. For write-requests, Cassandra will either replicate to any available (replication) node, a quorum of nodes or to all nodes, even providing options how to deal with multi-datacenter setups.

For read-requests, you can instruct Cassandra to either wait for any available node (which might return stale data), a quorum of machines (thereby reducing the probability to get stale data) or to wait for every node, which will always return the latest data and provide us with our long-sought, strong consistency.

About snapshots 
===============

Cassandra backs up data by taking a snapshot of all on-disk data files (SSTable files) stored in the data directory. You can take a snapshot of all keyspaces, a single keyspace, or a single table while the system is online.

Using a parallel ssh tool (such as pssh), you can snapshot an entire cluster. This provides an eventually consistent backup. Although no one node is guaranteed to be consistent with its replica nodes at the time a snapshot is taken, a restored snapshot resumes consistency using Cassandra's built-in consistency mechanisms.

After a system-wide snapshot is performed, you can enable incremental backups on each node to backup data that has changed since the last snapshot: each time a memtable is flushed to disk and an SSTable is created, a hard link is copied into a /backups subdirectory of the data directory (provided JNA is enabled). Compacted SSTables will not create hard links in /backups because snapshot_before_compaction creates a new set of hardlinks before every compaction that can be used to recreate any SSTables compacted.



















