---
layout: post
title: How to integrate PIG with Cassandra (without DSE)
category: Tools
tags: Pig Cassandra
summary: Some configuration tips for use Pig with Cassandra
---

Currently I used latest Cassandra release : 2.1.1

Download of latest [pig release](http://pig.apache.org/releases.html)
Or use packaged version with [apache bigtop](http://bigtop.apache.org/) 

Create symbolic link (I extract my pig tar in /usr/local/etc)
ln -s /usr/local/etc/pig-0.13.0/bin/pig /usr/bin/pig

Add some export in .bashrc or .profile

```bash
export JAVA_HOME=/usr/lib/jvm/java-7-oracle
export PIG_HOME=/usr/local/etc/pig-0.13.0
export PIG_INITIAL_ADDRESS=myipadress
export PIG_RPC_PORT=9160 (port defined in cassandra.yaml for rpc)
export PIG_PARTITIONER=org.apache.cassandra.dht.Murmur3Partitioner (the partioner of your cassandra.yaml)
```

I try [example from datastax DSE 3.1](http://www.datastax.com/docs/datastax_enterprise3.1/solutions/about_pig) to check that my initial configuration is good


Run Pig in local mode to check Grunt console

```bash
pig -x local
grunt> moretestvalues= LOAD 'cql://cql3ks/moredata/' USING CqlStorage;
2015-03-08 09:42:06,096 [main] ERROR org.apache.pig.tools.grunt.Grunt - ERROR 1070: Could not resolve CqlStorage using imports: [, java.lang., org.apache.pig.builtin., org.apache.pig.impl.builtin.]
Details at logfile: /home/jabberwock/pig_1425804114514.log
```

This seems to be a dependency issue ? So I will find the jar needed containing the needed class :

```bash
unzip -l /usr/share/cassandra/apache-cassandra.jar | grep CqlS
    1074  2014-10-24 10:31   org/apache/cassandra/hadoop/pig/CqlStorage$1.class
    27495  2014-10-24 10:31   org/apache/cassandra/hadoop/pig/CqlStorage.class
```

I get it !

Add the cassandra jar to additional jars for run pig

```bash
pig -Dpig.additional.jars=/usr/share/cassandra/apache-cassandra.jar -x local

grunt> moretestvalues= LOAD 'cql://cql3ks/moredata/' USING CqlStorage;
2015-03-08 09:42:06,096 [main] ERROR org.apache.pig.tools.grunt.Grunt - ERROR 1070: Could not resolve CqlStorage using imports: [, java.lang., org.apache.pig.builtin., org.apache.pig.impl.builtin.]
Details at logfile: /home/jabberwock/pig_1425804114514.log
```

With defining the CqlStorage it works better but it does not work yet completely
 
```bash
grunt> define CqlStorage org.apache.cassandra.hadoop.pig.CqlStorage();
grunt> moretestvalues= LOAD 'cql://cql3ks/moredata/' USING CqlStorage;
2015-03-08 09:43:18,506 [main] ERROR org.apache.pig.tools.grunt.Grunt - ERROR 2998: Unhandled internal error. org/apache/thrift/TBase
Details at logfile: /home/jabberwock/pig_1425804114514.log
```

It seems that we need all dependancies, so finally I did something more simple :

```bash
pig -Dpig.additional.jars=/usr/share/cassandra/*.jar:/usr/share/cassandra/lib/*.jar -x local
```

So I can, continue with our example :

```bash
grunt> moretestvalues= LOAD 'cql://cql3ks/moredata/' USING CqlStorage;
grunt> insertformat= FOREACH moretestvalues GENERATE TOTUPLE(TOTUPLE('a',x)),TOTUPLE(y);
grunt> STORE insertformat INTO
       'cql://cql3ks/test?output_query=UPDATE+cql3ks.test+set+b+%3D+%3F'
       USING CqlStorage;

Failed!

Failed Jobs:
JobId	Alias	Feature	Message	Outputs
N/A	insertformat,moretestvalues	MAP_ONLY	Message: org.apache.pig.backend.executionengine.ExecException: ERROR 2118: org.apache.cassandra.exceptions.ConfigurationException: Unable to find inputformat class 'org.apache.cassandra.hadoop.cql3.CqlPagingInputFormat'
	at org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.PigInputFormat.getSplits(PigInputFormat.java:275)
	at org.apache.hadoop.mapred.JobClient.writeNewSplits(JobClient.java:962)
......
Caused by: java.io.IOException: org.apache.cassandra.exceptions.ConfigurationException: Unable to find inputformat class 'org.apache.cassandra.hadoop.cql3.CqlPagingInputFormat'
	at org.apache.cassandra.hadoop.pig.AbstractCassandraStorage.getInputFormat(AbstractCassandraStorage.java:259)
	at org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.PigInputFormat.getSplits(PigInputFormat.java:260)
	... 20 more
```

```bash
unzip -l apache-cassandra-2.1.1.jar | grep Cql
     1891  2014-10-24 10:31   org/apache/cassandra/cql/CqlLexer$DFA15.class
     1207  2014-10-24 10:31   org/apache/cassandra/cql/CqlLexer$DFA7.class
    52924  2014-10-24 10:31   org/apache/cassandra/cql/CqlLexer.class
      464  2014-10-24 10:31   org/apache/cassandra/cql/CqlParser$comparatorType_return.class
    83745  2014-10-24 10:31   org/apache/cassandra/cql/CqlParser.class
     1236  2014-10-24 10:31   org/apache/cassandra/cql3/CqlLexer$DFA11.class
     2472  2014-10-24 10:31   org/apache/cassandra/cql3/CqlLexer$DFA19.class
    69106  2014-10-24 10:31   org/apache/cassandra/cql3/CqlLexer.class
      705  2014-10-24 10:31   org/apache/cassandra/cql3/CqlParser$1.class
      449  2014-10-24 10:31   org/apache/cassandra/cql3/CqlParser$username_return.class
   256830  2014-10-24 10:31   org/apache/cassandra/cql3/CqlParser.class
     4210  2014-10-24 10:31   org/apache/cassandra/hadoop/cql3/CqlBulkOutputFormat.class
     1831  2014-10-24 10:31   org/apache/cassandra/hadoop/cql3/CqlBulkRecordWriter$1.class
     1974  2014-10-24 10:31   org/apache/cassandra/hadoop/cql3/CqlBulkRecordWriter$ExternalClient.class
     6694  2014-10-24 10:31   org/apache/cassandra/hadoop/cql3/CqlBulkRecordWriter.class
    22378  2014-10-24 10:31   org/apache/cassandra/hadoop/cql3/CqlConfigHelper.class
     1323  2014-10-24 10:31   org/apache/cassandra/hadoop/cql3/CqlInputFormat$1.class
     2635  2014-10-24 10:31   org/apache/cassandra/hadoop/cql3/CqlInputFormat.class
```

CqlPagingInputFormat doesn't exist anymore in Cassandra 2.1 it was replaced by CqlInputFormat (see release's change of 2.0.10), see also [Jira ticket](https://issues.apache.org/jira/browse/CASSANDRA-6454)

In this [DSE 4.5 reference to PIG](http://www.datastax.com/documentation/datastax_enterprise/4.5/datastax_enterprise/ana/anaPigExRel.html) there is some information :
"Save the relation to the Cassandra simple_table1 table. In DataStax Enterprise 4.5.2 and later, use **USING CqlNativeStorage instead of USING CqlStorage**."

If we check the [release's note of the DSE 4.5.2](http://www.datastax.com/documentation/datastax_enterprise/4.5/datastax_enterprise/RNdse45.html), we can see that it is built on the Cassandra 2.0.10.
So in my case (with cassandra 2.1.x) I must use CqlNativeStorage.

Again :

```bash
grunt> define CqlStorage org.apache.cassandra.hadoop.pig.CqlNativeStorage();
grunt> moretestvalues= LOAD 'cql://cql3ks/moredata/' USING CqlNativeStorage;
grunt> insertformat= FOREACH moretestvalues GENERATE TOTUPLE(TOTUPLE('a',x)),TOTUPLE(y);
grunt> STORE insertformat INTO 'cql://cql3ks/test?output_query=UPDATE+cql3ks.test+set+b+%3D+%3F' USING CqlNativeStorage;
       
2015-03-08 15:21:42,950 [Thread-3] WARN  org.apache.hadoop.mapred.FileOutputCommitter - Output path is null in cleanup
2015-03-08 15:21:42,950 [Thread-3] WARN  org.apache.hadoop.mapred.LocalJobRunner - job_local_0001
java.lang.NoClassDefFoundError: com/codahale/metrics/Metric
	at com.datastax.driver.core.Cluster$Manager.<init>(Cluster.java:1120)
	at com.datastax.driver.core.Cluster$Manager.<init>(Cluster.java:1064)
	at com.datastax.driver.core.Cluster.<init>(Cluster.java:113)
	at com.datastax.driver.core.Cluster.<init>(Cluster.java:100)
	at com.datastax.driver.core.Cluster.buildFrom(Cluster.java:169)
	at com.datastax.driver.core.Cluster$Builder.build(Cluster.java:1029)
	at org.apache.cassandra.hadoop.cql3.CqlConfigHelper.getInputCluster(CqlConfigHelper.java:313)
	at org.apache.cassandra.hadoop.cql3.CqlRecordReader.initialize(CqlRecordReader.java:129)
	at org.apache.pig.backend.hadoop.executionengine.mapReduceLayer.PigRecordReader.initialize(PigRecordReader.java:178)
	at org.apache.hadoop.mapred.MapTask$NewTrackingRecordReader.initialize(MapTask.java:522)
	at org.apache.hadoop.mapred.MapTask.runNewMapper(MapTask.java:763)
	at org.apache.hadoop.mapred.MapTask.run(MapTask.java:370)
```

Just add metrics-core-3.0.2 to additional jars and it works !!