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
