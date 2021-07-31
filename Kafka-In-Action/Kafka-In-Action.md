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

Apache Kafka is a distributed system at heart but it is also possible to install and run it on a single host. That gives us a starting point to dive into our sample use cases. As is often the case, the real questions start flowing once the hands hit the keyboard.

## 2.1 Producing and consuming a message

A message, also called a record, is the basic piece of data flowing through Kafka. Messages are how your data is represented in Kafka. **Each message has an optional key, a value, a timestamp, and optional metadata headers.** Here is a simple example of a message: 
* Message key: 1234567 
* Message value: "Alert: machine Failed" 
* Message timestamp: "2020-10-02T10:34:11.654Z"

![chapter-2-figure-1.PNG](pictures/chapter-2-figure-1.PNG)

## 2.2 What are brokers?

Brokers can be thought of as the server-side of Kafka. Before virtual machines and Kubernetes, you would likely have seen one physical server hosting one broker. Since almost all clusters will have more than one server (or node) we are going to have three Kafka servers running for most of our examples.

For our first example, we will be creating a topic and sending our first message to Kafka from the command-line. One thing to note is that Kafka was built with the command-line in mind. There is no GUI that we will be using, so we need to have a way to interact with the operating system’s command-line interface.

To send our first message, we will need a place to send it. To create a topic, we will run the ``kafka-topics.sh`` command in a shell window with the option. We will find this ``--create`` script in the installation directory of Kafka. For example, the path might look like this: ``~/kafka_2.13-2.7.1/bin``.

```shell
bin/kafka-topics.sh --bootstrap-server localhost:9092 --create \ 
--topic kinaction_helloworld --partitions 3 --replication-factor 3

Created topic kinaction_helloworld.
```

**We could have used any name, of course but a popular option is to follow general Unix/Linux naming conventions: including not using spaces.**

The ``partitions`` option determines how many parts we want the topic to be split into. For example, since we have three brokers, using three partitions will give us one partition per broker. For our test workloads, we might not need this many based on data needs alone. However, doing more than one at this stage will let us see how the system works in spreading data across partitions. The ``replication-factor`` also is set to three in this example. In essence, this says that for each partition, we are attempting to have three replicas. These copies will be a crucial part of our design to improve reliability and fault-tolerance.

The ``bootstrap-server`` option points to our local Kafka broker. This is why the broker should be running before invoking this script. For our work right now, the most important goal is to get a picture of the layout. We will dig into how to best estimate the numbers we would need in other use cases when we get into the broker details later.

We can also look at all existing topics that have been created and make sure that our new one is on the list. The ``--list`` option is what we can reach for to achieve this output.

```shell
bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
```

To get a feel for how our new topic looks, let’s run another command that will give us a little more insight into our cluster.

Note that it is not like a traditional single topic in other messaging systems: we have replicas and partitions! The number we see next to the labels for the Leader, Replicas, and Isr fields is the ``broker.id`` that corresponds to the value for the broker we set in our configuration files on our three brokers. Briefly looking at the output, we can see that our topic consists of three partitions: ``Partition 0``, ``Partition 1``, and ``Partition 2``. Each partition was replicated three times as we intended on topic creation.

```shell
bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic kinaction_helloworld

Topic:kinaction_helloworld  PartitionCount:3 ReplicationFactor:3         Configs:
Topic: kinaction_helloworld Partition: 0     Leader: 0 Replicas: 0,1,2   Isr: 0,1,2
Topic: kinaction_helloworld Partition: 1     Leader: 1 Replicas: 1,2,0   Isr: 1,2,0
Topic: kinaction_helloworld Partition: 2     Leader: 2 Replicas: 2,0,1   Isr: 2,0,1
```

Now, let’s zoom-in on ``Partition 0``. Partition 0 has its replica copy leader on broker 0. This partition also has replicas that exist on brokers 1 and 2. The last column, ``isr`` stands for in-sync replicas. In-sync replicas (ISR) show which brokers are current and not lagging behind the leader. Having a partition replica copy that is out of date or behind the leader is an issue that we will cover later. Still, it is critical to remember that replica health in a distributed system is something that we will want to keep an eye out for. Figure 2.2 shows a view if we look at the one broker with id 0.

![chapter-2-figure-2.PNG](pictures/chapter-2-figure-2.PNG)

*Note*. I'm running commands against a specific broker, which is 9092. There are two more. Hence, the book talks about interacting with one broker now.

For our topic, note how broker 0 holds the leader replica ``kinaction_helloworld`` for partition 0. It also holds replica copies for partitions 1 and 2 for which it is not the leader replica. In the case of its copy of partition 1, the data for this replica will be copied from broker 1.

**When we reference a partition leader in the image, we are referring to a replica leader. It is important to remember that a partition can consist of one or more replicas, but only one replica will be a leader. A leader’s role involves being updated by external clients, whereas non-leaders will take updates only from their leader.**

Now once we have created our topic, and verified that it exists, we can start sending real messages! Those who have worked with Kafka before might ask why we took the above step to create the topic before sending a message. **There is a configuration to enable or disable the auto-creation of topics. However, it is usually best to control the creation of topics as a specific action** as we do not want new topics to randomly show up if one mistypes a topic name once or twice or be recreated due to producer retries.

To send a message, we will start-up a terminal tab to run a producer that will run as a console application and take user-input. The below command will start an interactive program that will take over that shell (to exit CTRL-C).

```shell
bin/kafka-console-producer.sh --bootstrap-server localhost:9092 \
--topic kinaction_helloworld
```

Notice that we reference the topic that we want to interact with and a bootstrap-server parameter. This parameter can be just one (or a list) of our current brokers in our cluster. By supplying this information, the cluster can obtain the metadata it needs to work with the topic.

**Note to make sure that we are using the same topic parameter for both commands, otherwise we won’t see anything.**

```shell
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic kinaction_helloworld --from-beginning
```

You should see your message.

As we send more messages and confirm the delivery to the consumer application, we can terminate the process and leave off the ``--from-beginning`` option when we restart it. Notice that we didn’t see all of the previously sent messages. Only those messages produced since the consumer console was started show up.

## 2.3 Tour of Kafka

![chapter-2-table-1.PNG](pictures/chapter-2-table-1.PNG)

A Producer is a tool for sending messages to Kafka topics. A producer is also used to send messages inside Kafka itself. For example, if we are reading data from a specific topic and wanted to send it to a different topic, we would also use a producer.

In contrast, a Consumer is a tool for retrieving messages from Kafka. In the same vein as producers, if we are talking about getting data out of Kafka, we are looking at consumers being involved directly or indirectly.

### 2.3.2 Topics Overview

Topics are where most users will start to think about the logic of what messages should go where. **Topics consist of units called partitions. In other words, one or many partitions can make up a single topic.** As far as what is actually implemented on the computer’s disk, partitions are what Kafka will be working with for the most part. **A single partition replica only exists on one broker and will not be split between brokers.**

![chapter-2-figure-7.PNG](pictures/chapter-2-figure-7.PNG)

Think back to our first example of the kinaction_helloworld topic. If you are looking at reliability and want three copies of the data, the topic itself is not one entity (or single file) that is copied; instead, it is the various partitions that are replicated three times each.

One of the most important concepts to understand at this point is the idea that one of the partition copies (replicas) will be what is referred to as a 'leader'. For example, if you have a topic made up of three partitions and a replication factor of three, every partition will have a leader replica elected.

Producers and consumers will only read or write from the leader replica of each partition it is assigned during scenarios where there are no exceptions or failures.

But how does your producer or consumer know which partition replica is the leader?

### 2.3.3 The What And Why Of Zookeeper

One of the oldest sources of added complexity in the Kafka ecosystem might be that it uses ZooKeeper. Apache ZooKeeper is a distributed store zookeeper.apache.org/ that is used to provide discovery, configuration, and synchronization services in a highly available way.

While Kafka provides fault-tolerance and resilience, someone has to provide coordination, and ZooKeeper enables that piece of the overall system.

As you have seen already, our cluster for Kafka includes more than one broker (server). To act as one correct application, these brokers need to not only communicate with each other; they also need to reach an agreement. Agreeing on whom the replica leader of a partition is, is one example of the practical application of ZooKeeper within the Kafka ecosystem.

### 2.3.4 Kafka’s High-level Architecture

One of Kafka’s keys is its usage of the ``pagecache`` of the operating system. 

By avoiding caching in the JVM heap, the brokers can help prevent some of the issues that large heaps can have: i.e. long or frequent garbage collection pauses. Another design consideration was the access pattern of data. While new messages flood in, it is likely that the latest messages would be of most interest to many consumers which could then be served from this cache.

As mentioned before, Kafka uses its own protocol. Using an existing protocol like AMQP was noted by the creators as having too large a part in the impacts on the actual implementation.

### 2.3.5 The Commit Log

One of the core concepts to help you master Kafka’s foundation is to understand the **commit log**. The concept is simple but powerful. This becomes clearer as you understand the significance of this design choice. To clarify, the log we are talking about is not the same as the log use case that involved aggregating the output from loggers from an application process, such as LOGGER.error messages in Java.

![chapter-2-figure-10.PNG](pictures/chapter-2-figure-10.PNG)

The log used in Kafka is not just a detail that is hidden in other systems that might use something similar (like a write-ahead-log for a database). It is front-and-center, and its users will use offsets to know where they are in that log.

What makes the commit log special is its append-only nature in which events are always added to the end. In most traditional systems, linear read and writes usually perform better than random operations that would require spinning hard-drive disks. The persistence as a log itself for storage is a major part of what separates Kafka from other message brokers. **Reading a message does not remove it from the system or exclude it from other consumers.**

One common question then becomes, how long can I retain data in Kafka? In various companies today, it is not rare to see that after the Kafka commit logs' data hits a configurable size or time retention period, the data is often moved into a permanent store like S3 or HDFS. However, it is a matter of how much disk space you need and your processing workflow. The New York Times has a single partition that holds less than 100GB.

## 2.4 Various Source Code Packages And What They Do?

Kafka is often mentioned in the titles of various APIs. There are also certain components that are described as stand-alone products. We are going to look at some of these to see what options we have.

### 2.4.1 Kafka Stream

Kafka Streams has grabbed a lot of attention compared to core Kafka itself. This API is found in the Kafka source code project’s directory ``streams`` and is mostly written in Java. One of the sweet spots for Kafka Streams is that no separate processing cluster is needed. It is meant to be a lightweight library to use in your application.

The more you move throughout this book, you will understand the foundations of how the Kafka Streams API uses the existing core of Kafka to do some exciting and powerful work.

### 2.4.2 Kafka Connect

Kafka Connect is found in the core Kafka folder and is also mostly connect written in Java. **This framework was created to make integrations with other systems easier.** In many ways, it can be thought to help replace other tools such as an Apache incubator project Gobblin and Apache Flume. If one is familiar with Flume, some of the terms used will likely seem familiar. **Source connectors are used to import data from a source into Kafka.** For example, if we want to move data from MySQL tables to Kafka’s topics, we would use a Connect source to produce those messages into Kafka. On the other hand, **sink connectors** are used to export data from Kafka into a different system. For example, if we wanted messages in some topic to be maintained longer-term, we would use a sink connector to consume those messages from the topic and place them somewhere like HDFS or S3.

**Kafka Connect is an excellent choice for making quick and simple data pipelines that tie together common systems.**

### 2.4.3 AdminClient Package

With version 0.11.0, Kafka introduced the AdminClient API. Before this API, scripts and other programs that wanted to perform specific administrative actions would either have to run shell scripts (which Kafka provides) or invoke internal classes often used by those shell scripts.

### 2.4.4 ksqlDB

In late 2017, a developer preview was released by Confluent of a new SQL engine for Kafka that was called KSQL before being renamed to ksqlDB. This allowed developers and data analysts who have used mostly SQL for data analysis to leverage streams by using the interface they have known for years.

While the interface for data engineers will be a familiar SQL-like grammar, the idea that queries will be continuously running and updating is where use cases like dashboards on service outages would likely replace applications that once used point-in-time SELECT statements.

## 2.5 What clients can I use for my language of choice?

Since using a client is the most likely way you will interact with Kafka in your applications, let’s look at using the Java client.

```java
public class HelloWorldProducer {

  public static void main(String[] args) {

    Properties producerProperties = new Properties();   //<1>
    producerProperties.put("bootstrap.servers", "localhost:9092,localhost:9093,localhost:9094");   //<2>

    producerProperties.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");    //<3>
    producerProperties.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

    try (Producer<String, String> producer = new KafkaProducer<>(producerProperties)) { //<4>

      ProducerRecord<String, String> producerRecord = new ProducerRecord<>("kinaction_helloworld", null, "hello world again!");   //<5>

      producer.send(producerRecord);    //<6>
    }
  }
}
```

1) The producer takes a map of name/value items to configure its various options.
2) This property can take a list of Kafka brokers.
3) The message’s key and value have to be told what format they will be serializing.
4) This creates a producer instance. Producers are thread-safe! Producers implement Closable interface, and will be closed automatically by Java runtume.
5) This is what represents our message.
6) Sending the record to the Kafka broker!

The above code is a simple producer. The first step to create a producer involved setting up configuration properties. The properties are set in a way that anyone who has used a map will be comfortable using. The ``bootstrap.servers`` parameter is one essential config item, and its purpose may not be apparent at first glance. This list is a list of your Kafka brokers. **A best practice is to include more than one server to ensure that if one server in the list had crashed or was in maintenance, your producer would still have something alive to talk to on start-up. This list does not have to be every server you have, though, as after it connects, it will be able to find out information about the rest of the cluster’s brokers and not depend on that list.** The ``key.serializer`` and ``value.serializer`` are also something to take note of when developing. We need to provide a class that will serialize the data as it moves into Kafka. **Keys and values do not have to use the same serializer.**

The created producer is thread-safe and takes in the configuration properties as an argument in the constructor we used. With this producer, we can now send messages. The ProducerRecord will contain the actual input that we wish to send. In our examples, "kinaction_helloworld" is the name of the topic we wish to send. The next fields are the message key, followed by the message value. We will discuss keys more in Chapter 4, but it is enough to know that it can indeed be a null value and this makes our current example less complicated.

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>2.7.1</version>
</dependency>
```

Now that we have created a new message, let’s use our Java client to create a consumer that can see the message.

```java
public class HelloWorldConsumer {

  final static Logger log = LoggerFactory.getLogger(HelloWorldConsumer.class);

  private volatile boolean keepConsuming = true;

  public static void main(String[] args) {
    Properties props = new Properties();  //<1>
    props.put("bootstrap.servers", "localhost:9092,localhost:9093,localhost:9094");
    props.put("group.id", "helloconsumer");
    props.put("enable.auto.commit", "true");
    props.put("auto.commit.interval.ms", "1000");
    props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
    props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

    HelloWorldConsumer helloWorldConsumer = new HelloWorldConsumer();
    helloWorldConsumer.consume(props);
    Runtime.getRuntime().addShutdownHook(new Thread(helloWorldConsumer::shutdown));
  }

  private void consume(Properties props) {
    try (KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props)) {
      consumer.subscribe(Collections.singletonList("kinaction_helloworld"));  //<2>

      while (keepConsuming) {
        ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));  //<3>
        for (ConsumerRecord<String, String> record : records) {   //<4>
          log.info("[Consumer Record] offset = {}, key = {}, value = {}", record.offset(), record.key(), record.value());
        }
      }
    }
  }

  private void shutdown() {
    keepConsuming = false;
  }
}
```

1) Properties are set the same way as producers.
2) The consumer needs to tell Kafka what topics it is interested in.
3) We want to keep polling for new messages as they come in.
4) We are printing out each record that we consume to the console to see the result.

**However, unlike the producer, the Java consumer client is not thread-safe.**

Our code is responsible for ensuring that any access is synchronized: one simple option is having only one consumer per Java thread. Also, whereas we told the producer where to send the message, we now have the consumer subscribe to the topics it wants. **A subscribe command can subscribe to more than one topic at a time.**

One of the most important sections to note is the ``poll`` call on the consumer. This is what is actively trying to bring messages to our application. **No messages, one message, or many messages could all come back with a single poll, so it is important to note that our logic should account for more than one result with each poll call.**

## 2.6 Stream Processing and Terminology

![chapter-2-figure-14.PNG](pictures/chapter-2-figure-14.PNG)

Kafka has many moving parts that depend on data coming into and out of its core to provide value to its users. Producers send data into Kafka, which works as a distributed system for reliability and scale with logs as the basis for storage. Once data is inside the Kafka ecosystem, consumers will help users utilize that data in their other applications and use cases.

Our brokers make up the cluster and coordinate with a ZooKeeper cluster to maintain metadata. Since Kafka stores data on disk, the ability to replay data in case of an application failure is also part of our context and feature set.

### 2.6.1 Streaming process

Stream processing seems to have various definitions throughout various projects. The core principle of streaming data is that data will keep arriving and will not end.

### 2.6.2 What exactly once means in our context

The easiest way to maintain exactly-once is to stay within Kafka’s walls (and topics). Having a closed system that can be completed as a transaction is why using the Streams API is one of the easiest paths to exactly once. Various Kafka Connect connectors also support ``exactly-once`` and are great examples of bringing data out of Kafka since it won’t always be the final endpoint for all data in every scenario.

## 2.7 Summary

* Messages represent your data in Kafka. Kafka’s cluster of brokers handle this data and interact with outside systems and clients.
* Kafka’s use of a commit log is an implementation detail. However, this detail helps in understanding the system overall. Messages appended to the end of a log frame the context of how data is stored and can be used again. By being able to start at the beginning of the log, applications can reprocess data in a specific order to fulfill different use-cases. 
* Producers are clients that help move data into the Kafka ecosystem. Populating existing information from other data sources, like a database, into Kafka can help expose data that was once siloed into a system that can provide a data interface for other applications. 
* Consumer clients retrieve messages from Kafka. Many consumers can read the same data at the same time. The ability for separate consumers to start reading at various positions also shows the flexibility of consumption possible from Kafka topics. 
* Continuously flowing data between destinations with Kafka can help us redesign systems that used to be limited to batch or time-delayed workflows.

# Chapter 3. Design a Kafka project

