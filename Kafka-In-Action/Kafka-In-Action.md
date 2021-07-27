# Chapter 1. Introduction to Kafka

Kafka is changing the standards for data platforms. It is leading the way to move from extract, transform, load (ETL) and batch workflows, in which work was often held and processed in bulk at one various pre-defined time of day, to near real-time data feed. Batch processing, which was once the standard workhorse of enterprise data processing, might not be something to turn back to after seeing the powerful feature set that Kafka provides.

## 1.1 What is Kafka?

The Apache Kafka® site (kafka.apache.org/intro) defines it as a distributed streaming platform that has three main capabilities: 
* Reading and writing records like a message queue. 
* Store records with fault-tolerance. 
* Process streams as they occur.

![chapter-1-figure-1-2.png](pictures/chapter-1-figure-1-2.png)

So what does this have to do with Kafka exactly? Kafka includes clients to interface with the other systems. One such client type is called a Producer that can send multiple data streams to the Kafka brokers. On other side sits Consumers - clients that can read data from the brokers and process it.

Data does not only have to be limited to a single destination. The Producers and Consumers are completely decoupled, allowing each client to work independently.

As do other messaging platforms, Kafka acts, in reductionist terms, like a middle man to data coming into the system (from producers) and out of the system (consumers). The loose coupling can be achieved by allowing this separation between the producer and end-user of the message. The producer can send whatever messages it wants and have no clue about if anyone is subscribed.

Further, Kafka also has various ways that it can deliver messages to fit your business case. **Kafka message delivery can take at least the following three delivery methods**: 
* At least once semantics - A message will be resent if needed until it is acknowledged. 
* At most once semantics - A message will only be sent once and not resent on failure.
* Exactly once semantics - A message will only be seen once by the consumer of the message.

### At least Once semantics

Kafka’s default guarantee is "at least once" semantics. In this case, Kafka can be configured to allow a producer of messages to send the same message more than once and have it written to the brokers. When a message has not received a guarantee that it was written to the broker, the producer can send the message again to try again. **For those cases where you can’t miss a message, say that someone has paid an invoice. This guarantee might take some filtering on the consumer end but is one of the safest delivery methods.**

### At most Once semantics

"At most once" semantics is when a producer of messages might send a message once and never retries. In the event of a failure, the producer moves on and never attempts to send again. Why would someone ever be okay with losing a message? If a popular website is tracking page views for visitors, it might be okay with missing a few page view events out of the millions they do process per day. Keeping the system performing and not waiting on acknowledgments might outweigh any cost of lost data.

### Exactly Once semantics

"Exactly once" semantics (EOS), added to the Kafka feature set in version 0.11.0, generated a lot of mixed discussion with its release. On the one hand, exactly-once semantics are ideal for a lot of use-cases. This guarantee makes logic to remove duplicate messages a thing of the past. Most developers appreciate sending one message in and receiving that same message on the consuming side as well. Another discussion that followed the release was a debate on if exactly once was even possible. While this goes into deeper computer science - it is helpful to be aware of how Kafka defines their EOS feature 4. In the context of a Kafka system, **if a producer sends a message more than once, it would still be delivered only once to the end consumer.** EOS has touchpoints at all Kafka layers, from producers, topics, brokers, and consumers, and will be tackled as we move along our discussion later in this book.

## 1.2 Kafka Usage

With many traditional companies facing challenges of becoming more and more technical and software-driven, one of the questions is how they will be prepared for the future. Kafka is noted for being a high-performance message-delivery workhorse that features replication and fault-tolerance as a default.

### 1.2.1 Kafka for the Developer

*Note*. I'm making a summary of what is written.

* Growing demand of Kafka knowledge.
* Decoupling applications.
* Learn Kafka before learning applications that use Kafka under the covers (Kafka Streams, ksqlDB, Apache Spark).

### 1.2.2 Explaining Kafka to your manager

One of Kafka’s most important features is the ability to take volumes of data and make it available for use by various business units. A data backbone that would make information coming into the enterprise available to all the multiple areas would allow flexibility and openness on a company-wide scale. Nothing is prescribed, but it is a potential outcome. Most executives will also know that more data than ever is flooding in, and they want insights as fast as possible. Rather than pay for data to molder on disk, the value can be derived from most of it as it arrives. Kafka is one way to move away from a daily batch job that limited how quickly that data could be turned into value. Fast Data seems to be a newer term that hints that real value focuses on something different from the promises of Big Data alone.

## 1.3 Kafka Myths

### 1.3.1 Kafka only works with Hadoop

As mentioned, Kafka is a powerful tool that is often used in various situations. However, it seemed to appear on radars when used in the Hadoop ecosystem and might have first appeared as a tool as part of a Cloudera or Hortonworks suite. It isn’t uncommon to hear the myth that Kafka only works on Hadoop.

One other fundamental myth that often appears is that Kafka requires the Hadoop Filesystem - HDFS. That is not the case.

### 1.3.2 Kafka is the same as other message brokers

Another big myth is that Kafka is just another message broker. Direct comparisons of the features of various tools such as RabbitMQ by Pivotal or IBM MQSeries to Kafka often have asterisks (or fine print) attached and are not always fair to the best use cases of each.

In general, some of the most exciting and different features are the following that we will dig into below: 
* The ability to replay messages by default 
* Parallel processing of data

Kafka was designed to have multiple consumers. **What that means is one application reading a message off of the message brokers doesn’t remove it from other applications that might want to consume it as well.** One effect of this is that a consumer who has already seen that message can again choose to read that (and other messages).

Kafka provides a way for Consumers to seek specific points and read messages (with a few constraints) again by just seeking an earlier position on the topic.

## 1.4 Kafka in the Real World

### 1.4.1 Early Examples

Some users' first experience with Kafka (as was mine) was using it as a messaging tool. Personally, after years of using other tools like IBM WebSphere MQ (formerly MQ Series), Kafka (which was around version 0.8.3 at the time) seemed simple to use to get messages from point A to point B. It forgoes using popular protocols and standards like the Extensible Messaging and Presence Protocol (XMPP), Java Message Service (JMS) API (now part of Jakarta EE), or the **OASIS® Advanced Message Queuing Protocol (AMQP) in favor of a custom TCP binary protocol.**

As an end-user developing with a Kafka client, most of the details are in the configuration, and the logic becomes relatively straightforward, ie. I want to place a message on this topic.

Having a durable channel for sending messages is also why Kafka is used. Oftentimes, memory storage of data in RAM only will not be enough to protect your data; if that server dies, the messages are not persisted across a reboot. **High availability and persistent storage are built into Kafka from the start.**

Kafka enables robust applications to be built and helps handle the expected failures that distributed applications are bound to run into at some point.

The throughput of small messages can sometimes overwhelm a system since the processing of each method takes time and overhead. Kafka uses batching of messages for sending data as well as writing data. Writing to the end of a log helps as well rather than random access to the filesystem.

### 1.4.2 Later Examples

Microservices used to talk to each other with APIs like REST, but can now leverage Kafka to communicate between asynchronous services with events.

Kafka has placed itself as a fundamental piece for allowing developers to get data quickly. While Kafka Streams is now a likely default for many when starting work, Kafka had already established itself as a successful solution by the time the Streams API was released in 2016. **The Streams API can be thought of as a layer that sits on top of producers and consumers.** This abstraction layer is a client library that is providing a higher-level view of working with your data as an unbounded stream.

In the Kafka 0.11 release, exactly-once semantics was introduced. We will cover what that means in practice later on once we get a more solid foundation. However, users running end-to-end workloads on top of Kafka with the Streams API may benefit from hardened delivery guarantees. Imagine banks using Kafka to debit or credit your account. With the "at least once" semantics, you would have to ensure that debit was not completed twice.

The number of devices for the Internet of Things will only increase with time. With all of those devices sending messages, sometimes in bursts whenever they get wifi or cellular connection, something needs to be able to handle that data effectively. As you may have gathered, massive quantities of data are one of the critical areas where Kafka shines.

### 1.4.3 When Kafka might not be the right fit

What if you only need a once-monthly or even once yearly summary of aggregate data? Suppose you don’t need an on-demand view, quick answer, or even the ability to reprocess data. In that case, you might not need Kafka running throughout the entire year for that task alone (notably if that amount of data is manageable to process at once as a batch). As always, your mileage may vary: different users have different thresholds on what is a large batch.

If your main access pattern for data is a mostly random lookup of data, Kafka might not be your best option. Linear read and writes is where Kafka shines and will keep your data moving as quickly as possible. Even if you have heard of Kafka having index files, they are not really what you would compare to a relational database having fields and primary keys that indexes are built.

Similarly, if you need the exact ordering of messages in Kafka for the entire topic, you will have to look at how practical your workload is in that situation. To avoid any unordered messages, care should have to be taken to ensure that only one producer request thread is the max simultaneously and that there is only one partition in the topic. **There are various workarounds, but if you have vast amounts of data that depend on strict ordering, there are potential gotchas that might come into play once you notice that your consumption would be limited to one consumer per group at a time.**

One of the other practical items that come to mind is that large messages are an exciting challenge. **The default message size limit is about 1 MB.** With larger messages, you start to see memory pressure increase. In other words, the lower number of messages you can store in page cache could become a concern. So if you are planning on sending huge archives around, you might want to look at if there is a better way to manage those messages.

## 1.5 Online resources to get started

The community around Kafka has been one of the best (in my opinion) in making documentation available. Kafka has been a part of Apache (graduating from the Incubator in 2012) and keeps the current documentation at the project website at kafka.apache.org.

Another great resource for more information is Confluent® (www.confluent.io/resources). Confluent was founded by original Kafka’s creators and is actively influencing the future direction of the work.

# Chapter 2. Getting to know Kafka

