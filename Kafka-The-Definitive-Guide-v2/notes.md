# Chapter 1. Meet Kafka

## Publish/Subscribe Messaging

Publish/subscribe messaging is a pattern that is characterized by the sender (publisher) of a piece of data (message) not specifically directing it to a receiver.

### How It Starts

Many use cases for publish/subscribe start out the same way: with a simple message queue or interprocess communication channel. For example, you create an application that needs to send monitoring information somewhere, so you write in a direct connection from your application to an app that displays your metrics on a dashboard, and push metrics over that connection.

![chapter-1-figure-1.png](pictures/chapter-1-figure-1.png)

This is a simple solution to a simple problem that works when you are getting started with monitoring. Before long, you decide you would like to analyze your metrics over a longer term, and that doesn’t work well in the dashboard. You start a new service that can receive metrics, store them, and analyze them. In order to support this, you modify your application to write metrics to both systems. By now you have three more applications that are generating metrics, and they all make the same connections to these two services.

![chapter-1-figure-2.png](pictures/chapter-1-figure-2.png)

The technical debt built up here is obvious, so you decide to pay some of it back. You set up a single application that receives metrics from all the applications out there, and provide a server to query those metrics for any system that needs them.

![chapter-1-figure-3.png](pictures/chapter-1-figure-3.png)

### Individual Queue Systems

At the same time that you have been waging this war with metrics, one of your coworkers has been doing similar work with log messages. Another has been working on tracking user behavior on the frontend website and providing that information to developers who are working on machine learning, as well as creating some reports for management. You have all followed a similar path.

![chapter-1-figure-4.png](pictures/chapter-1-figure-4.png)

There is a lot of duplication. Your company is maintaining multiple systems for queuing data, all of which have their own individual bugs and limitations.

## Enter Kafka

Apache Kafka is a publish/subscribe messaging system designed to solve this problem. It is often described as a “distributed commit log” or more recently as a “distributing streaming platform.”

### Messages and Batches

The unit of data within Kafka is called a **message**. If you are approaching Kafka from a database background, you can think of this as similar to a row or a record. **A message is simply an array of bytes as far as Kafka is concerned, so the data contained within it does not have a specific format or meaning to Kafka.**

A message can have an optional piece of metadata, which is referred to as a **key**. The key is also a byte array and, as with the message, has no specific meaning to Kafka. Keys are used when messages are to be written to partitions in a more controlled manner. **The simplest such scheme is to generate a consistent hash of the key, and then select the partition number for that message by taking the result of the hash modulo the total number of partitions in the topic.** This assures that messages with the same key are always written to the same partition.

For efficiency, messages are written into Kafka in batches. **A batch is just a collection of messages, all of which are being produced to the same topic and partition.** An individual roundtrip across the network for each message would result in excessive overhead, and collecting messages together into a batch reduces this.

**Batches are also typically compressed, providing more efficient data transfer and storage at the cost of some processing power.**

### Schemas

Talks about Avro.

### Topics and Partitions

Messages in Kafka are categorized into **topics**. **The closest analogies for a topic are a database table or a folder in a filesystem.** Topics are additionally broken down into a number of **partitions**. **Going back to the “commit log” description, a partition is a single log.** Messages are written to it in an append only fashion, and are read in order from beginning to end.

**Note that as a topic typically has multiple partitions, there is no guarantee of message time-ordering across the entire topic, just within a single partition.**

Partitions are also the way that Kafka provides redundancy and scalability. Each partition can be hosted on a different server, which means that a single topic can be scaled horizontally across multiple servers to provide performance far beyond the ability of a single server. Additionally, partitions can be replicated, such that different servers will store a copy of the same partition in case one server fails.

### Producers and Consumers

The consumer subscribes to one or more topics and reads the messages in the order in which they were produced. The consumer keeps track of which messages it has already consumed by keeping track of the offset of messages. **The offset — an integer value that continually increases — is another piece of metadata that Kafka adds to each message as it is produced.**

Consumers work as part of a consumer group, which is one or more consumers that work together to consume a topic. The group assures that each partition is only consumed by one member. In Figure 1-6, there are three consumers in a single group consuming a topic. Two of the consumers are working from one partition each, while the third consumer is working from two partitions. The mapping of a consumer to a partition is often called ownership of the partition by the consumer. In this way, consumers can horizontally scale to consume topics with a large number of messages. **Additionally, if a single consumer fails, the remaining members of the group will rebalance the partitions being consumed to take over for the missing member.**

![chapter-1-figure-6.png](pictures/chapter-1-figure-6.png)

### Brokers and Clusters

Kafka brokers are designed to operate as part of a cluster. Within a cluster of brokers, **one broker will also function as the cluster controller (elected automatically from the live members of the cluster).** The controller is responsible for administrative operations, including assigning partitions to brokers and monitoring for broker failures. A partition is owned by a single broker in the cluster, and that broker is called the leader of the partition. A partition may be assigned to multiple brokers, which will result in the partition being replicated.

A key feature of Apache Kafka is that of retention, which is the durable storage of messages for some period of time. Kafka brokers are configured with a default retention setting for topics, either retaining messages for some period of time (e.g., 7 days) or until the topic reaches a certain size in bytes (e.g., 1 GB). Once these limits are reached, messages are expired and deleted. In this way, the retention configuration defines a minimum amount of data available at any time. Individual topics can also be configured with their own retention settings so that messages are stored for only as long as they are useful.

### Multiple Clusters

As Kafka deployments grow, it is often advantageous to have multiple clusters. There are several reasons why this can be useful: 
* Segregation of types of data 
* Isolation for security requirements 
* Multiple datacenters (disaster recovery)

The Kafka project includes a tool called MirrorMaker, used for replicating data to other clusters. At its core, MirrorMaker is simply a Kafka consumer and producer, linked together with a queue. Messages are consumed from one Kafka cluster and produced for another.

## Why Kafka?

Already covered in Kafka in Action.

### Multiple Producers

### Multiple Consumers

### Disk-Based Retention

Not only can Kafka handle multiple consumers, but durable message retention means that consumers do not always need to work in real time. Messages are committed to disk, and will be stored with configurable retention rules.

### Scalable

### High Performance

## The Data Ecosystem

### Use Cases

#### Activity tracking

#### Messaging

#### Metrics and logging

#### Commit log

Since Kafka is based on the concept of a commit log, database changes can be published to Kafka and applications can easily monitor this stream to receive live updates as they happen. This changelog stream can also be used for replicating database updates to a remote system, or for consolidating changes from multiple applications into a single database view.

#### Stream processing

## Kafka’s Origin

### LinkedIn’s Problem

### The Birth of Kafka

### Open Source

### Commercial Engagement

### The Name

## Getting Started with Kafka

# Chapter 2. Installing Kafka

## Setup of Zookeper and Kafka.Broker Configuration

There are numerous configuration options for Kafka that control all aspects of setup and tuning. **The majority of the options can be left to the default settings though, as they deal with tuning aspects of the Kafka broker that will not be applicable until you have a specific use case that requires adjusting these settings.**

### General Broker

There are several broker configuration parameters that should be reviewed when deploying Kafka for any environment other than a standalone broker on a single server. **These parameters deal with the basic configuration of the broker, and most of them must be changed to run properly in a cluster with other brokers.**

#### broker.id

Every Kafka broker must have an integer identifier, which is set using the ``broker.id`` configuration. By default, this integer is set to 0, but it can be any value. It is essential that the integer must be unique for each broker within a single Kafka cluster.

It is highly recommended to set this value to something intrinsic to the host so that when performing maintenance it is not onerous to map broker ID numbers to hosts. For example, if your hostnames contain a unique number (such as host1.example.com, host2.example.com, etc.), then 1 and 2 would be good choices for the broker.id values respectively.

#### listeners

Older versions of Kafka used a simple ``port`` configuration. This can be still be used as a backup for simple configurations but is a **deprecated config**. The example configuration file starts Kafka with a listener on TCP port 9092.

The new ``listeners`` config is a comma separated list of URIs that we listen on with the listener names.

A listener is defined as ``<protocol>:<hostname>:<port>``. An example of legal listener config: ``PLAINTEXT://localhost:9092,SSL://:9091``. Specifying the hostname as ``0.0.0.0`` will bind to all interfaces. Leaving the hostname empty will bind it to the default interface.

**Don't start Kafka with ports lower than 1024.**

#### zookeeper.connect

The location of the Zookeeper used for storing the broker metadata is set using the ``zookeeper.connect`` configuration parameter. The example configuration uses a Zookeeper running on port 2181 on the local host, which is specified as ``localhost:2181``.

The format for this parameter is a semicolon-separated list of ``hostname:port/path`` strings. 

*Self-Note*. ``path`` part is not interesting at this moment.

#### log.dirs

Kafka persists all messages to disk, and these log segments are stored in the directory specified in the ``log.dir`` configuration. For multiple directories, the config ``log.dirs`` is preferrable.

If this value is not set, it will default back to ``log.dir``. ``log.dirs`` is a comma-separated list of paths on the local system.

#### num.recovery.threads.per.data.dir

Kafka uses a configurable pool of threads for handling log segments. Currently, this thread pool is used: 
* When starting normally, to open each partition’s log segments 
* When starting after a failure, to check and truncate each partition’s log segments 
* When shutting down, to cleanly close log segments

As these threads are only used during startup and shutdown, it is reasonable to set a larger number of threads in order to parallelize operations. Specifically, when recovering from an unclean shutdown, this can mean the difference of several hours when restarting a broker with a large number of partitions!

#### auto.create.topics.enable

The default Kafka configuration specifies that the broker should automatically create a topic under the following circumstances: 
* When a producer starts writing messages to the topic 
* When a consumer starts reading messages from the topic 
* When any client requests metadata for the topic

**Set to false.**

#### auto.leader.rebalance.enable

In order to ensure a Kafka cluster doesn’t become unbalanced by having all topic leadership on one broker, this config can be specified to ensure leadership is balances as much as possible. It enables a background thread that checks the distribution of partitions at regular intervals (this interval in configurable via ``leader.imbal ance.check.interval.seconds``). If leadership imbalance exceeds another config ``leader.imbalance.per.broker.percentage`` then a rebalance of preferred leaders for partitions is started.

#### delete.topic.enable

Disabling topic deletion can be set by setting this flag to false.

### Topic Defaults

**The defaults in the server configuration should be set to baseline values that are appropriate for the majority of the topics in the cluster.**

#### num.partitions

The ``num.partitions`` parameter determines how many partitions a new topic is created with, primarily when automatic topic creation is enabled (which is the default setting). **This parameter defaults to one partition. Keep in mind that the number of partitions for a topic can only be increased, never decreased.**

Many users will have the partition count for a topic be equal to, or a multiple of, the number of brokers in the cluster.

##### How to Choose the Number of Partitions

There are several factors to consider when choosing the number of partitions: 
* What is the throughput you expect to achieve for the topic? For example, do you expect to write 100 KB per second or 1 GB per second? 
* What is the maximum throughput you expect to achieve when consuming from a single partition? A partition will always be consumed completely by a single consumer (as even when not using consumer groups, the consumer must read all messages in the partition). If you know that your slower consumer writes the data to a database and this database never handles more than 50 MB per second from each thread writing to it, then you know you are limited to 50 MB/sec throughput when consuming from a partition. 
* You can go through the same exercise to estimate the maximum throughput per producer for a single partition, but since producers are typically much faster than consumers, it is usually safe to skip this. 
* **If you are sending messages to partitions based on keys, adding partitions later can be very challenging, so calculate throughput based on your expected future usage, not the current usage.** 
* Consider the number of partitions you will place on each broker and available diskspace and network bandwidth per broker. 
* **Avoid overestimating, as each partition uses memory and other resources on the broker and will increase the time for metadata updates and leadership transfers.** 
* Will you be mirroring data? You may need to consider the throughput of your mirroring configuration as well. Large partitions can become a bottleneck in many mirroring configurations. 
* **If you are using cloud services, do you have IOPS limitiations on your VMs or disks?** There may be hard caps on the number of IOPS allowed depending on your cloud service and VM configuration that will cause you to hit quotas. Having too many partitions can have the side-effect of increasing the amount of IOPS due to the parallelism involved.

**If you have some estimate regarding the target throughput of the topic and the expected throughput of the consumers, you can divide the target throughput by the expected consumer throughput and derive the number of partitions this way.** So if I want to be able to write and read 1 GB/sec from a topic, and I know each consumer can only process 50 MB/s, then I know I need at least 20 partitions. This way, I can have 20 consumers reading from the topic and achieve 1 GB/sec.

Starting small and expanding as needed is easier than starting too large.

#### default.replication.factor

**If auto-topic creation is enabled, this configuration sets what the replication factor should be for new topics.**

It is highly recommended to set the replication factor to at least 1 above ``min.insync.replicas`` setting. For more fault resistant settings if you have large enough clusters and enough hardware, setting your replication factor to 2 above the ``min.insync.replicas`` (abbrevated as RF++) can be preferrable. RF++ will allow easier maintenance and prevent outages. **The reasoning behind this recommendation is to allow for one planned outage within the replica set and one unplanned outage to occur simultaneously.** For a typical cluster, this would mean you’d have a minumum of 3 replicas of every partition.

#### log.retention.ms

The most common configuration for how long Kafka will retain messages is by time. The default is specified in the configuration file using the ``log.retention.hours`` parameter, and it is set to 168 hours, or one week. However, there are two other parameters allowed, ``log.retention.minutes`` and ``log.retention.ms``. All three of these control the same goal (the amount of time after which messages may be deleted) but the **recommended parameter to use is ``log.retention.ms``, as the smaller unit size will take precedence if more than one is specified.** This will make sure that the value set for ``log.retention.ms`` is always the one used.

#### log.retention.bytes

Another way to expire messages is based on the total number of bytes of messages retained. This value is set using the ``log.retention.bytes`` parameter, and it is **applied per-partition**. This means that if you have a topic with 8 partitions, and ``log.retention.bytes`` is set to 1 GB, **the amount of data retained for the topic will be 8 GB at most.** Note that all retention is performed for individual partitions, not the topic. This means that **should the number of partitions for a topic be expanded, the retention will also increase if ``log.retention.bytes`` is used.** Setting the value to -1 will allow for infinite retention.

#### log.segment.bytes

The log-retention settings previously mentioned operate on log segments, not individual messages. As messages are produced to the Kafka broker, they are appended to the current log segment for the partition. Once the log segment has reached the size specified by the ``log.segment.bytes`` parameter, which **defaults to 1 GB**, the log segment is closed and a new one is opened. Once a log segment has been closed, it can be considered for expiration. **A smaller log-segment size means that files must be closed and allocated more often, which reduces the overall efficiency of disk writes.**

Adjusting the size of the log segments can be important if topics have a low produce rate. For example, **if a topic receives only 100 megabytes per day of messages, and ``log.segment.bytes`` is set to the default, it will take 10 days to fill one segment. As messages cannot be expired until the log segment is closed, if log.retention.ms is set to 604800000 (1 week), there will actually be up to 17 days of messages retained until the closed log segment expires.** This is because once the log segment is closed with the current 10 days of messages, that log segment must be retained for 7 days before it expires based on the time policy (as the segment cannot be removed until the last message in the segment can be expired).

#### log.segment.ms

Another way to control when log segments are closed is by using the ``log.segment.ms`` parameter, which specifies the amount of time after which a log segment should be closed. As with the ``log.retention.bytes`` and ``log.retention.ms`` parameters, ``log.segment.bytes`` and ``log.segment.ms`` are **not mutually exclusive properties**. Kafka will close a log segment either when the size limit is reached or when the time limit is reached, whichever comes first. **By default, there is no setting for ``log.seg ment.ms``, which results in only closing log segments by size.**

#### min.insync.replicas

When configuring your cluster for data durability, **setting ``min.insync.replicas`` to 2 ensures that at least two replicas are caught up and “in sync” to the producer. This is used in tandem with setting the producer config to ack “all” requests. This will ensure that at least two replicas (leader and one other) acknowledge a write in order for it to be successful.** This can prevent data loss in scenarios where the leader acks a write then suffers a failure and leadership is transferred to a replica that does not have a successful write.

#### message.max.bytes

The Kafka broker limits the maximum size of a message that can be produced, configured by the ``message.max.bytes`` parameter, which defaults to 1000000, or 1 MB. A producer that tries to send a message larger than this will receive an error back from the broker, and the message will not be accepted. As with all byte sizes specified on the broker, **this configuration deals with compressed message size, which means that producers can send messages that are much larger than this value uncompressed**, provided they compress to under the configured ``message.max.bytes`` size.

##### Coordinating Message Size Configurations

**The message size configured on the Kafka broker must be coordinated with the fetch.message.max.bytes configuration on consumer clients. If this value is smaller than message.max.bytes, then consumers that encounter larger messages will fail to fetch those messages, resulting in a situation where the consumer gets stuck and cannot proceed. The same rule applies to the replica.fetch.max.bytes configuration on the brokers when configured in a cluster.**

## Hardware Selection

To low level for now.

## Kafka in the Cloud

Not interested.

## Kafka Clusters

A single Kafka server works well for local development work, or for a proof-of- concept system, but there are significant benefits to having multiple brokers configured as a cluster. 

**The biggest benefit is the ability to scale the load across multiple servers. A close second is using replication to guard against data loss due to single system failures.**

### How Many Brokers?

