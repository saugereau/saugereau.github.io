---
layout: post
title: Cassandra row count tips
category: Coding Tips
tags: cassandra nosql devcenter
summary: How to get the row count in a cassandra table, by distinguishing these values from cluster or node
---

## Before anything, some clarification about the tools and the mode of connection they use.

When you use [DevCenter](http://www.datastax.com/what-we-offer/products-services/devcenter), you define a connection with one or more server.
Thus, when you do an CQL instruction, this one is done on the cluster with the default consistency level ( the consistency level defaults to ONE for all write and read operations),
and so on the closest replica respond.

When you use Cqlsh, in the same way your CQL instruction are done on the cluster (and not on the node on which you execute the cqlsh script).

A simple example :

- Create a cluster of two nodes (update your cassandra.yaml), we call them nodeA and nodeB
- Start cqlsh on nodeA
- Create a keyspace (with a replication factor of 2), thus all your node have all datas

```
CREATE KEYSPACE mybeautifulkeyspace WITH REPLICATION = { 'class' : 'SimpleStrategy', 'replication_factor' : 2};
```

- CREATE a simple table on this keyspace

```
USE mybeautifulkeyspace;
CREATE TABLE mysimpletable (id int, label text, PRIMARY KEY(id));
```

- Insert a row in this table

```
INSERT INTO mysimpletable (id, label) values (1,'un libelle');
```

- you can start cqlsh on nodeB, and do a select :

```
USE mybeautifulkeyspace;
SELECT * FROM mysimpletable;
```

- your row is correctly returned
- the tricky things come here, stop cassandra on nodeB
- insert a new line on nodeA and stop cassandra on nodeA
- restart cassandra on nodeB, and connect via cqlsh to the keyspace mybeautifulkeyspace, the select return only one rows.

```
 id | label
----+------------
  1 | un libelle
```

- restart cassandra on nodeA
- on nodeB, always with cqlsh do the select, it could take before the cassandra on nodeB could answer (after restarting), but after few select on nodeB the second line appears too

```
 id | label
----+--------------
  1 |   un libelle
  2 | un libelle 2
```

- Ok, this is great, cqlsh get correctly datas from nodeA, but if I stop again my nodeA, what happen ?
The select let appear the second line on nodeB, because the node do a read repair.


## Row count

### On the cluster :

```
select count(*) from mysimpletable
```

However, there is a default limit of 10,000 applied to this statement which will truncate the result for larger tables. The limit can be increased :

```
select count(*) from mysimpletable limit 1000000
```

### On a specific node :

Or you could use nodetool, but it's only estimated value (this information disappeared in Cassandra 2.1)

```
>nodetool cfstats mybeautifulkeyspace

Keyspace: mybeautifulkeyspace
        Read Count: 0
        Read Latency: NaN ms.
        Write Count: 0
        Write Latency: NaN ms.
        Pending Tasks: 0
                Table: mysimpletable
                SSTable count: 1
                Space used (live), bytes: 4734
                Space used (total), bytes: 4734
                SSTable Compression Ratio: 0.5972222222222222
                Number of keys (estimate): 128
                Memtable cell count: 0
                Memtable data size, bytes: 0
                Memtable switch count: 0
                Local read count: 0
                Local read latency: 0,000 ms
                Local write count: 0
                Local write latency: 0,000 ms
                Pending tasks: 0
                Bloom filter false positives: 0
                Bloom filter false ratio: 0,00000
                Bloom filter space used, bytes: 16
                Compacted partition minimum bytes: 61
                Compacted partition maximum bytes: 86
                Compacted partition mean bytes: 79
                Average live cells per slice (last five minutes): 0.0
                Average tombstones per slice (last five minutes): 0.0

```

For my two rows the estimated value is 128 ...

Or other option is to use the sstablekeys that allows to get the keys from sstable file (generaly in /var/lib/cassandra/data but
it could be specified in cassandra.yaml).

An example of usage on this bash script :

```
#!/bin/bash

#prend en parametre le nom de la table
if [ -z "$1" ]
  then echo "fournir le nom d'une table en parametre"
  exit 1
fi

nodetool flush
#recuperation de tout les fichiers data correspondant
for file in `ls /data/cassandra/data/mybeautifulkeyspace/$1/*-$1-*-Data.db`
do
  echo "operating $file"
  sstablekeys $file >> output.txt
done
nbLigne=$(sort output.txt | uniq | wc -l)
rm output.txt
echo "nombre de ligne $nbLigne"
```

## What about the deleted rows ?

Deletes in Cassandra rely on Tombstones to support the Eventual Consistency model. More information [here](http://wiki.apache.org/cassandra/DistributedDeletes)
Tombstones are markers that can exist at different levels of the data model and let the cluster know that a delete was recored on a replica, and when it happened.
Tombstones then play a role in keeping deleted data hidden and help with freeing space used by deleted columns on disk.
The CQL count query doesn't count them,

Instead the compaction process reconciles the data in multiple SSTables on disk. The row fragments from each SSTable are collated and columns with the same name reconciled using the process we’ve already seen. The result of the compaction is a single SSTable that contains the same “truth” as the input files, but may be considerably smaller due to reconciling overwrites and deletions.
So it was possible that the count by sstable count these deleted rows until the compaction is done.


