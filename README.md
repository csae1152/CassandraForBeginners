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












