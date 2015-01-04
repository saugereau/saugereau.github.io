---
layout: post
title: How to store tweet to cassandra
category: Architecture
tags: twitter-api flume kafka storm cassandra
summary: Blablabla
---

I wanted to do some tries to make a use case for a JUG session about cassandra, so I thinks about few things

- How to have a large volume of data
- How to get data this data and heavily insert into Cassandra
- How to do this with the minimum of code and using existing tools


## First step : The datas

How can I find a source that provides a constant and high volume of data for which I can later do a little calculation :

- logs : it was proportional dto the underlying system, ant it can be difficult to generate this activity during a demo
- twitter : it can be interesting for a next content analysis, like tweet's similarity

## Next step : How to get these data 

### [Flume](http://flume.apache.org/)

Flume is initially a service for efficiently collecting, aggregating, and moving large amounts of log data.
 
- Flume define an [experimental source type for twitter](http://flume.apache.org/FlumeUserGuide.html#twitter-1-firehose-source-experimental), for use it you need to [create OAuth tokens](https://dev.twitter.com/oauth/overview/application-owner-access-tokens).
- [flume-ng-cassandra-sink 1.0.0](https://github.com/btoddb/flume-ng-cassandra-sink) is a cassandra sink for flume

The Data Flow Model

<figure>
  <img src="/blog/assets/images/heavy-writing-to-cassandra/flume-model.png" />
  <figcaption>Flume Data flow model</figcaption>
</figure> 

A little vocabulary :

- A **source** consumes events delivered to it by an external
- When a **source** receives an event, it stores it into one or more **channels**. The **channel** is a passive store that keeps the event until it’s consumed by a Flume **sink**
- The **sink** removes the event from the channel and puts it into an external repository like HDFS (via Flume HDFS sink) or forwards it to the Flume source of the next Flume agent (next hop) in the flow
- An **agent** is a (JVM) process that hosts the components through which events flow from an external source to the next destination.

<figure>
  <img src="/blog/assets/images/heavy-writing-to-cassandra/flume-model-sink-to-source.png" />
  <figcaption>Flume Sink forwards event to the Flume source of the next Flume agent</figcaption>
</figure> 



Drawbacks 

- Twitter source is experimental
- flume-ng-cassandra-sink is no more maintained and has some old dependancies and it use old cassandra model version :

    - In the documentation, the keyspace and table creation are the old way (with column family instead of table)
    - The flume-version in pom.xml is 1.1.0, it was a beta version released more than two years ago
    - The sink is defined for write on a cassandra's defined table structure, so we have to do some update to change that
    - The cassandra client api used is Hector 1.1.2, it has been released in Nov 2012 and build upon cassandra-thrift and cassandra-all 1.1.0 (probably matching Cassandra 1.1) 
    - The cassandra 1.x and Cassandra 2.x model are not compatible, there is heavy legacy ... Since there is a specific [java client developed by dataStax](http://www.datastax.com/documentation/developer/java-driver/2.0/java-driver/whatsNew2.html)

### [Kafka](http://kafka.apache.org/)

Kafka is a distributed, partitioned, replicated commit log service. It provides the functionality of a messaging system, but with a unique design.

<figure>
  <img src="/blog/assets/images/heavy-writing-to-cassandra/kafka-model.png" />
  <figcaption>Kafka model</figcaption>
</figure>

A little vocabulary :

- A **producer** is a process that publish messages to a Kafka topic
- A **consumer** is a process that subscribe to topics and process the feed of published messages
- A **Kafka cluster** is comprised of one or more servers, each of which is called a broker.
- A **topic** is a category or feed name to which messages are published. For each topic, the Kafka cluster maintains a partitioned log with one or more partition.
- The **partitions** of the log are distributed over the servers in the Kafka cluster (Kafka uses ZooKeeper to manage partition) with each server handling data and requests for a share of the partitions. Each partition is replicated across a configurable number of servers for fault tolerance

In this architecture, we need one producer that publish to kafka queue the tweet, and one or more consumer that store the data to Cassandra

Drawbacks
- Need some installation : Zookeeper (we can use the one embedded with kafka), Kafka
- I've already use kafka and there is no big challenge


### [Storm](https://storm.apache.org/)

<figure>
  <img src="/blog/assets/images/heavy-writing-to-cassandra/storm-cluster.png" />
  <figcaption>Storm cluster</figcaption>
</figure>

A little vocabulary :

- A **spout** is a source of streams. For example, a spout may read tuples off of a Kestrel queue and emit them as a stream
- A **bolt** consumes any number of input streams, does some processing, and possibly emits new streams
- Networks of **spouts** and **bolts** are packaged into a **topology** which is the top-level abstraction that you submit to Storm clusters for execution
- Each worker node (**bolt** and **spout**) runs a daemon called the **Supervisor**. The supervisor listens for work assigned to its machine and starts and stops worker processes as necessary based on what Nimbus has assigned to it
- Zookeper supervise the coordination between Nimbus and the Supervisors

Advantages
- [a storm-starter project](https://github.com/apache/storm/tree/master/examples/storm-starter) 
- a local mode for test who doesn't care Zookeeper's installation
- the storm starter project already contains a TwitterSampleSpout
- we can introduce some business logic in the bolts (words tokenization, etc ...)

Drawbacks 
- we can't have parallelized twitter spout (this will probably cause duplicate rows in cassandra) 

## Implementing the solution


