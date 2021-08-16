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

In our previous chapter, we started to see how we can work with Kafka from the command line and using a Java client. Now, we will expand on those first concepts and look at designing various solutions with Kafka.

## 3.1 Designing a Kafka project

While Kafka is being used in new companies and projects as they get started, that is not the case for all adopters. For those of us who have been in enterprise environments or worked with legacy systems (and anything over five years old is probably considered legacy these days), that is not a luxury we always have in reality.

We will work on the project for a company that is ready to shift from their current way of doing data and apply this new hammer named Kafka.

### 3.1.1 Taking over an existing data architecture

Using the topic of sensors as a uses-case, we will dig into a made-up example project. Our new fictional consulting company has just won a contract to help re-architect a plant that works on e-bikes and manages them remotely. Sensors were placed throughout the bike which continuously provide events about the condition and status of the internal equipment they are monitoring.

Besides this, our current data context includes traditional relational database systems that are large and clustered.

### 3.1.2 Kafka Connect

One of the best ways to start our journey is probably not a big-bang approach: all our data does not have to move into Kafka at once. If we use a database today and want to kick the tires on streaming data, one of the easiest on-ramps is to start with Kafka Connect. It can handle production loads, but it does not have to out of the gate. We will take one database table and start our new architecture while letting the existing applications run for the time being.

### 3.1.3 Connect Features

The purpose of Connect is to help move data into or out of Kafka without writing our own producers and consumers. Connect is a framework that is already part of Kafka that really can make it simple to use pieces that have previously been built to start your streaming journey. These pieces are called connectors, and they have been developed to work reliably with other data sources.

The easiest option to run and test Connect on your local machine is to run it in standalone mode.

In the folder where you installed Kafka, you should locate the following files: ``connect-standalone.properties`` and ``connect-file-source.properties`` under the config directory. Peeking inside the ``connect-standalone.properties`` file, you should see some configuration keys and values that should look familiar from some of the properties you used to make your own Java clients.

We are taking data from one data source and into Kafka, and we will treat data as being sourced from that file. Using the file, ``connect-file-source.properties``, included with your Kafka installation as an example template, let’s create our file called ``alert-source.properties``:

```properties
name=alert-source
connector.class=FileStreamSource #1
tasks.max=1                      #2
file=alert.txt                   #3
topic=alert-connect              #4
```

1) The class that we are delegating the work of interacting with our source file.
2) For standalone mode, 1 is a valid value to test our setup.
3) This is the file that will be monitored for changes.
4) The topic property is the name of the topic where this data will be sent.

This file defines the configuration that is needed to define the file name of interest, ``alert.txt``, and that we want the data to be sent to the specific topic named ``alert-connect``. With configuration and not code, we can get data into Kafka from any file! Since reading from a file is a common task, we can leverage the pre-built classes provided. In this case, the class is ``FileStreamSource``.

The file name of ``alert.txt`` will be monitored for changes for new messages. We have chosen 1 for the value of ``tasks.max`` since we only really need one task for our connector and are not worried about parallelism.

Now that we have done the needed configuration, we’ll need to start Connect and send in our configuration.

```shell
bin/connect-standalone.sh config/connect-standalone.properties alert-source.properties
```

Moving over to another terminal window, we will create a text file named ``alert.txt`` in the directory in which we started the Connect service. We can add a couple of lines of text to this file using a text editor.

Now we can use the console consumer command to verify that Connect is doing its job. Let’s open another terminal, and consume from the ``alert-connect`` topic.

```shell
bin/kafka-console-consumer.sh \
--bootstrap-server localhost:9092 \
--topic alert-connect --from-beginning
```

Before moving to another connector type, let’s quickly talk about the sink connector and how it can carry Kafka’s messages back out to another file. Since the destination (or sink) for this data will be another file, we are interested in looking at the file ``connect-file-sink.properties``.

Notice the small difference in the configuration as the new outcome is writing to a file rather than reading from a file as we did before. ``FileStreamSink`` is declared to define this new role as a sink. The topic ``alert-connect`` will be the source of our data in this scenario.

```properties
name=alert-sink                 
connector.class=FileStreamSink  #1
tasks.max=1                     #2
file=alert-sink.txt             #3
topics=alert-connect            #4
```

1) This is an available out of the box class to which we are delegating the work of  interacting with our file.
2) For standalone mode, 1 is a valid value to test our setup.
3) This is the destination file for any messages that make it to our Kafka topic
4) The topic property is the name of the topic that the data will come from.

```shell
bin/connect-standalone.sh config/connect-standalone.properties \
alert-source.properties alert-sink.properties
```

The end result should be data flowing from a file into Kafka and back out to a separate destination file.

### 3.1.4 Connect for our Invoices

Connect allows those with in-depth knowledge of creating custom connectors and share them with others to help those of us that may not be the experts in those systems.

To discover connectors that the community and vendors have developed, check out Confluent Hub confluent.io/hub - an app store for connectors.

To start using Connect in our manufacturing example, we will look at using an existing source connector that will stream table updates to a Kafka topic from a local database.

Again, our goal is not to change the entire data processing architecture at once. We are going to show how we would start bringing in updates from a database table-based application and develop our new application in parallel while letting the other system exist as-is. Our first step is to set up a database for our local examples. For ease of use and to get started quickly, we’ll be using SQLite.

For ease of development, we are going to be using connectors from Confluent for the SQLite example.

To create a database, we just run the following from the command line: ``sqlite3 kafkatest.db``. In this database, we will run the following to create the ``invoices`` table and insert some test data. As we design our table, it is helpful to think of how we will capture changes into Kafka. Most use-cases will not require us to capture the entire database but just changes after the initial load. A timestamp, sequence number, or ID could help us determine what data has changed and needs to be sent to Kafka.

```sql
CREATE TABLE invoices(
    id INT PRIMARY KEY NOT NULL,
    title TEXT NOT NULL,
    details CHAR(50),
    billedamt REAL,
    modified TIMESTAMP DEFAULT (STRFTIME('%s', 'now')) NOT NULL
);
INSERT INTO invoices (id,title,details,billedamt) VALUES (1, 'book', 'Franz Kafka', 500.00 );
```

By copying the pre-built ``etc/kafka-connect-jdbc/source-quickstart-sqlite.properties`` file to ``kafkatest-sqlite.properties`` and then after making slight changes to our database table name, we can see how additional inserts and updates to the rows will cause messages to be sent into Kafka.

```shell
bin/confluent local start
# OR
bin/connect-standalone etc/schema-registry/connect-avro-standalone.properties \
etc/kafka-connect-jdbc/kafkatest-sqlite.properties
```

## 3.2 Sensor Event Design

Since there are no existing connectors for our start-of-the-art sensors, we can directly interact with their event system through custom producers. This ability to hook into and write our producers to send data into Kafka is the context in which we will look at the following requirements.

### 3.2.1 Existing issues

#### DEALING WITH DATA SILOS

The shift from most traditional data thinking is to make the data available to everyone in its raw source. If you have access to the data as it comes in, you do not have to worry about the application API exposing it in their specific formats or after custom transformations have been done.

#### RECOVERABILITY

One of the excellent perks of a distributed system like Kafka is that failure is an expected condition, planned for and handled.

*Note*. Talks about same stuff. Nothing of practical use.

#### WHEN SHOULD DATA BE TRANSFORMED

One of the most common tasks for engineers that have worked with data for any amount of time is one in which data is taken from one location, transformed, and then loaded into a different system. Extract, transform, and load (ETL) is a common term for this work.

Just keep in mind that hardware, software, and logic can and will fail in distributed computing, so it’s always great to get data into Kafka first, which gives you the ability to replay data if any of those failures occur.

### 3.2.2 Why Kafka is a right fit

*Note*.
This has been talked about already.

### 3.2.3 Thought starters on our design

#### Past Kafka Version History Milestones

* 2.0.0 - ACLS with prefix support, host-name verification default for SSL
* 1.0.0 - Java 9 Support, JBOD disk failure improvements
* 0.11.0 - Admin API
* 0.10.2 - Improved client compatibility

The following questions are intended to make us think about how we want to process our data. These preferences will impact various parts of our design, but our main focus is just on figuring out the structure.

* **Is it okay to lose any messages in the system?** For example, is one missed event about a mortgage payment going to ruin your customer’s day and their trust in your business?
* **Does your data need to be grouped in any way?** Are the events correlated with other events that are coming in? For example, are we going to be taking in address changes? In that case, we might want to associate the various address changes with the customer whose address is changing.
* **Do you need data delivered in exact order?** What if a message gets delivered in an order other than when it occurred? For example, if you get an order canceled notice before the actual order.
* **Do you only want the last value of a specific item?** Or is the history of that item important? Do you care about the history of how your data has evolved?
* **How many consumers are we going to have?** Will they all be independent of each other, or will they need to maintain some sort of order when reading the messages?

### 3.2.4 User data requirements

### 3.2.5 High-Level Plan for applying our questions

Our new architecture will need to provide a couple of specific key features.

* We want to make sure that only users with access were able to perform actions against the sensors. 
* We should not lose messages as our audit would not be complete without all the events. 
* In this case, we do not need any grouping key. 
* Each event can be treated as independent. 
* The order does not matter inside our audit topic, as each message will have a timestamp in the data itself. Our primary concern is that all the data is there to process.

**As a side note, Kafka itself does allow messages to be sorted by time, and your message payload can include time as well.**

#### Audit Checklist

| Kafka Feature | Concern? |
| ------------- | ------------- |
| Message Loss  | X  |
| Grouping  |   |
| Ordering  |   |
| Last value only |   |
| Independent Consumer  | X  |

The alert trend of statuses requirement deals with the the alert of each process in the bike’s system, with a goal of spotting trends. So, it might be helpful to group this data using a key. We have not addressed the term **'key'** in-depth, **but it can be thought of as a way to group related events.**

#### Alert Trending Checklist

| Kafka Feature | Concern? |
| ------------- | ------------- |
| Message Loss  |   |
| Grouping  | X  |
| Ordering  |   |
| Last value only |   |
| Independent Consumer  | X  |

#### Alert Checklist

| Kafka Feature | Concern? |
| ------------- | ------------- |
| Message Loss  |   |
| Grouping  | X  |
| Ordering  |   |
| Last value only | X  |
| Independent Consumer  |  |

### 3.2.6 Reviewing our blueprint

One of the last things to think about is how we want to keep these groups of data organized. Logically, the groups of data can be thought of in the following manner:

* audit data 
* alert trend data 
* alert data

## 3.3 Data Format

If you look at the Kafka documentation, you may have noticed references to another serialization system called Apache Avro™. Avro provides schema definition support as well as storage of schemas in Avro files. Let’s take a closer look at why this format is commonly used in Kafka.

### 3.3.1 Schemas for Context

One of the significant gains of using Kafka is that the producers and consumers are not tied directly to each other. Further, Kafka does not care about the content of the data or do any validation by default. Kafka understands how to quickly move bytes and lets us have the freedom to put whatever data you need into the system.

One of the significant gains of using Kafka is that the producers and consumers are not tied directly to each other. Further, Kafka does not care about the content of the data or do any validation by default. Kafka understands how to quickly move bytes and lets us have the freedom to put whatever data you need into the system. However, there is likely a need for each process or application to understand the context of what that data means and what format is in use. By using a schema, we provide a way for our applications to understand the structure and intent of the data.

```json
{
  "type": "record",                     //1
  "name": "libraryCheckout",
  "namespace": "org.kafkainaction",
  "fields": [
    {
      "name": "materialName",
      "type": "string",
      "default": ""
    },
    {
      "name": "daysOverDue",            //2
      "type": "int",                    //3
      "default": 0                      //4
    },
    {
      "name": "checkoutDate",
      "type": "int",
      "logicalType": "date",
      "default": "-1"
    },
    {
      "name": "borrower",
      "type": {
        "type": "record",
        "name": "borrowerDetails",
        "fields": [
          {
            "name": "cardNumber",
            "type": "string",
            "default": "NONE"
          },
          {
            "name": "residentZipCode",
            "type": "string",
            "default": "NONE"
          }
        ]
      },
      "default": {}
    }
  ]
}
```

1) JSON-defined Avro schema
2) Direct mapping to a field name
3) Field with name, type, and default define
4) Default value provided

### 3.3.2 Usage of Avro

Now that we have discussed some of the advantages of using a schema, why would we look at Avro? First of all, Avro always is serialized with its schema. While not a schema itself, Avro supports schemas when reading and writing data and can apply rules to handle schemas that can change over time. Also, if you have ever seen JSON, it is pretty easy to understand Avro. Besides the data itself, the schema language itself is defined in JSON as well. The ease of readability does not have the same storage impacts of JSON, however. A binary representation is used for efficient storage. The interesting point is that Avro data is serialized with its schema. If the schema changes, you can still process data.

Clients are the ones who gain the benefit of dealing with Avro.

Let’s get started with how we will use Avro by adding it to our as ``pom.xml``.

```xml
<dependency>
    <groupId>org.apache.avro</groupId>
    <artifactId>avro</artifactId>
    <version>${avro.version}</version>
</dependency>
```

Since we are already modifying the pom file, let’s go ahead and include a plugin that will generate the Java source code for our schema definitions. As a side note, you can also generate the sources from a standalone java jar titled ``avro-tools`` if you do not want to use a Maven plugin.

```xml
<plugin>
    <groupId>org.apache.avro</groupId>
    <artifactId>avro-maven-plugin</artifactId>
    <version>${avro.version}</version>
    <executions>
        <execution>
            <phase>generate-sources</phase>
            <goals>
                <goal>schema</goal>
            </goals>
            <configuration>
                <sourceDirectory>${project.basedir}/src/main/resources/</sourceDirectory>
                <outputDirectory>${project.basedir}/src/main/java/</outputDirectory>
            </configuration>
        </execution>
    </executions>
</plugin>
```

Let’s start defining our schema by thinking about the data types we will be using, beginning with our alert status scenario. To start, we’ll create a new file named alert.avsc with a text editor.

Alert will be the name of the Java class we will interact with after the generation of source code from this file.

```json
{
  "namespace": "org.kafkainaction",   //1
  "type": "record",
  "name": "Alert",                    //2
  "fields": [                         //3
    {
      "name": "sensor_id",
      "type": "long",
      "doc": "The unique id that identifies the sensor"
    },
    {
      "name": "time",
      "type": "long",
      "doc": "Time the alert was generated as UTC milliseconds from the epoch"
    },
    {
      "name": "status",
      "type": {
        "type": "enum",
        "name": "AlertStatus",
        "symbols": [
          "Critical",
          "Major",
          "Minor",
          "Warning"
        ]
      },
      "doc": "The allowed values that our sensors will use to emit current status"
    }
  ]
}
```

1) Namespace determines the generated package
2) Alert will be the name of the created Java class
3) The fields defining data types and documentation notes

**One thing to note is that "doc" is not a required part of the definition.**

Now that we have the schema defined let’s run the Maven build to see what we are working with - ``mvn generate-sources`` or ``mvn install`` can generate the sources in our project. This should give us a couple of generated classes: ``org.kafkainaction.Alert.java`` and ``org.kafkainaction.AlertStatus.java`` we can now use in our examples.

While we could always create our own serializer for Avro, we already have an excellent example provided by Confluent. Access to those existing classes is accomplished by adding the kafka-avro-serializer dependency to our build.

```xml
<dependency>
    <groupId>io.confluent</groupId>
    <artifactId>kafka-avro-serializer</artifactId>
    <version>${confluent.version}</version>
</dependency>
```

This is needed to avoid having to create our own Avro serializer and deserializer for the keys and values of our events.

With the build setup and our Avro object ready to use, let’s take our example producer from the last chapter HelloWorldProducer, and slightly modify the class to use Avro.

Notice the use of ``io.confluent.kafka.serializers.KafkaAvroSerializer`` as the value of the property ``value.serializer``. This will handle the Alert object that we create and send to our new avrotest topic. Before, we could use a string serializer, but with Avro, we need to define a specific value serializer to tell the client how to deal with our data. The use of ``Alert``, rather than a string, shows how we can utilize types in our applications as long as we can serialize them. This example also makes use of the Schema Registry. We will cover more details of the Schema Registry in chapter 11, but at this point, it is helpful to know that the property ``schema.registry.url`` points to the URL of our registry. **This registry can have a versioned history of schemas and helps us manage schema evolution.**

```java
public class HelloWorldProducer {
    private static final Logger log = LoggerFactory.getLogger(HelloWorldProducer.class);
    
    public static void main(String[] args) {
        var producerProperties = new Properties();
        producerProperties.put("bootstrap.servers", "localhost:9092,localhost:9093,localhost:9094");
        producerProperties.put("key.serializer", "org.apache.kafka.common.serialization.LongSerializer");
        producerProperties.put("value.serializer", "io.confluent.kafka.serializers.KafkaAvroSerializer");
        producerProperties.put("schema.registry.url", "http://localhost:8081");
        try (Producer<Long, Alert> producer = new KafkaProducer<>(producerProperties)) {
            var alert = new Alert(12345L, Instant.now().toEpochMilli(), Critical);
            log.info("Alert -> {}", alert);
            var producerRecord = new ProducerRecord<Long, Alert>("avrotest", alert.getSensorId(), alert);
            producer.send(producerRecord);
        }
    }
}
```

Now that we have produced messages using Alert, the other changes would be on the consumption side of the messages. For a consumer to get the values produced to our new topic, it will have to use a value deserializer, in this case, ``KafkaAvroDeserializer``.

```java
public class HelloWorldConsumer {
    final static Logger log = LoggerFactory.getLogger(HelloWorldConsumer.class);
    private volatile boolean keepConsuming = true;
    
    public static void main(String[] args) {
        var props = new Properties();
        props.put("bootstrap.servers", "localhost:9092");
        props.put("group.id", "helloconsumer");
        props.put("enable.auto.commit", "true");
        props.put("auto.commit.interval.ms", "1000");
        props.put("key.deserializer", "org.apache.kafka.common.serialization.LongDeserializer");
        props.put("value.deserializer", "io.confluent.kafka.serializers.KafkaAvroDeserializer");
        props.put("schema.registry.url", "http://localhost:8081");
        var helloWorldConsumer = new HelloWorldConsumer();
        helloWorldConsumer.consume(props);
        Runtime.getRuntime().addShutdownHook(new Thread(helloWorldConsumer::shutdown));
    }
    private void consume(Properties props) {
        try (var consumer = new KafkaConsumer<Long, Alert>(props)) {
            consumer.subscribe(Collections.singletonList("avrotest"));
            while (keepConsuming) {
                ConsumerRecords<Long, Alert> records = consumer.poll(Duration.ofMillis(100));
                for (var record : records) {
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

## 3.4 Summary

* Designing a Kafka solution first involves understanding our data. These details include how we need to handle data loss, ordering of messages, and latency in our use-cases.
* The need to group data will determine whether we will key the messages in Kafka.
* Leveraging Avro schemas definitions can not only help us generate code but can also help us handle data changes in the future. These schemas can be used with our own custom Kafka clients.
* Kafka Connect provides existing connectors to write to and from various data sources. To implement a solution moving data into or out of Kafka, searching the existing source and sink connectors available can help us quickly get started with the Kafka ecosystem.

# Chapter 4. Producers: sourcing data

## 4.1 Introducing the Producer

*Note*. Nothing of interesting.

### 4.1.1 Key Producer Write Path

While it took relatively few lines of code to send a message as seen in Chapter 2, the Java client producer write process is doing various tasks behind the scenes.

Calling ``send()`` in our code puts some impressive machinery into motion. The Sender class provides the plumbing to work with the RecordAccumulator to send requests to the Kafka brokers from a producer. The sender’s job includes fetching metadata about the cluster itself. Since producers only write to the replica leader of the partition they are assigned, the metadata helps the producer find out which actual broker to write to since the producer record might only have been given a topic name without any other details. This is nice because the end-user of the producer does not have to make a separate call to get that information. **The end-user needs to have at least one running broker to connect to, and the Java client library can figure out the rest.**

Continuing with the write path flow, we can see that the record accumulator’s job is to accumulate the messages into batches. These batches are grouped by broker and partition. Why did I just mention the word batch? Isn’t that a bad word when discussing stream processing?

Collecting messages is really about improving throughput. If each message was sent one at a time, we would see significantly slower throughput. Another option for improving throughput is to compress messages. If compression for the messages is used, along with batching, the full batch is compressed. This likely would allow for a higher compression ratio than one message at a time.

**If one message in the batch fails during delivery, then the entire batch fails for that partition.**

Since this distributed system is designed to account for transient errors, like a network blip, the logic for retries is built-in already. However, **if the ordering of the messages is essential**, then besides setting the retries to a non-zero number, we will also need to set the ``max.in.flight.requests.per.connection`` value to 1 and set acks (the number of brokers that send acknowledgments back) to **all** to provide the best situation for making sure your producer’s messages arrive in the order you intend.

Another option to be aware of is using an idempotent producer. In effect, the desire is for a producer to only write a message once. The term idempotent is referring to how sending the same message multiple times will only result in the message being written once. These settings can also be set with the configuration property ``enable.idempotence=true``.

Despite being able to have more than one in-flight request, the safest method to keep the order of your messages (in addition to the exactly-once delivery) is to set it at 1.

**Another item to note is that the idempotent delivery is for the lifetime of a producer. If we have another producer sourcing the same data or even a restart of that producer code, we might see duplicate messages on your topic.**

## 4.2 Important Options

Recent versions of the producer have over 50 properties that we could choose to set. **One way to deal with all of the producer configuration key names is to use the constants provided in the java class ``ProducerConfig`` when developing producer code.**

### 4.2.1 Producer Options

**Important Producer Configuration**

| Key  | Purpose |
| ------------- | ------------- |
| acks  | Number of replica acknowledgments producer requires before success is considered  |
| bootstrap.servers  | One or many Kafka brokers to connect for startup  |
| value.serializer | The class that is being used for serialization of the value  |
| key.serializer  | The class that is being used for serialization of the key  |
| compression.type  | The compression type (if any) used to compress messages  |


The Kafka documentation has a helpful way of letting us know which options might have the most impact. Look for the IMPORTANCE label of High in the documentation listed at kafka.apache.org/documentation/\#producerconfigs.

### 4.2.2 Configuring the Broker list

By connecting to this one broker, the client is able to discover the metadata it needs, which includes data about other brokers in the cluster as well. It is a good idea to send a list rather than one in case one of the servers is down or under maintenance and unavailable.

### 4.2.3 How to go Fast (or Safer)

We can wait in our code for the result of a producer send request, but we also have the option of handling success or failure asynchronously with callbacks or ``Futures``. If we want to go faster and not wait for a reply, we can still handle the results at a later time with our own custom logic.

Another configuration property that will apply to our scenarios is the ``acks`` key, which stands for acknowledgments. **This controls how many acknowledgments the producer needs to receive from the partition leader’s followers before it returns a complete request.** The valid values for this property are **'all,' -1, 1, and 0**.

**Setting this value to 0 will probably get us the lowest latency, but at the cost of durability.** Guarantees are not made if any broker received the message, and retries are not attempted. As a sample use-case, say that we have a web tracking platform that collects the clicks on a page and sends these events to Kafka. In this situation, it might not be a big deal to lose a single link press event or hover event. If it is lost, there is no real business impact.

**Setting this value to 1 will involve the receiver of the message, the leader replica of the specific partition, sending confirmation back to the producer.** The producer client would wait for that acknowledgment. Setting the value to 1 will mean that, at least, the leader replica has received the message. **However, the followers might not have copied the message before a failure brings down the leader.** If that situation occurs before a copy was made, then the message would never appear on the replica followers for that partition.

**Values all or -1 are the strongest available option.** T**his value means that a partition leader replica waits on the entire list of its in-sync replicas (ISRs) to acknowledge it has received the message.** While this is better for durability, it is easy to see that it won’t the quickest due to the dependencies it has on other brokers. In many cases, it is worth paying the performance price in order to prevent data loss. **With many brokers in a cluster, it is important to be aware of the number of brokers the leader will have to wait on.** The broker that takes the longest to reply will be the determining factor for how long it takes for the producer to receive a success message.

### 4.2.4 Timestamps

Recent versions of the producer record will have a timestamp on the events you send. A user can either pass the time into the constructor as a Java type long when sending a ProducerRecord Java object or the current system time will be used. The actual time that is used in the message can stay this value or can be a broker timestamp that occurs when the message is logged. Setting the topic configuration ``message.timestamp.type`` to ``CreateTime`` will use the time set by the client, whereas setting it to ``LogAppendTime`` will use the broker time.

As always, timestamps can be tricky. **For example we might get a record with an earlier timestamp than that of a record before it. This can happen in cases where a failure occurred and a different message with a later timestamp was committed before the retry of the first record completed.** The data will be ordered in the log by offsets and not by timestamp. While reading timestamped data is often thought of as a consumer client concern, it is also a producer concern since the producer takes the first steps to ensure message ordering. As discussed earlier, this is also why ``max.in.flight.requests.per.connection`` is important to consider whether you want to allow retries or many inflight requests at a time. If a retry happens and other requests succeeded on their first attempt, earlier messages might be added after those later ones.

### 4.2.5 Adding compression to our messages

One of the topics that we discussed briefly above, when talking about message batches, was compression. Google Snappy, GNU gzip, lz4, zstd (developed at FacebookTM), and none are all options that can be used and are set with the configuration key ``compression.type``.

Compression is done at the batch level. Size and rate of the data should be considered when deciding whether or not to use compression. **Small messages being compressed do not necessarily make sense in all cases. In addition, if you have low traffic, compression might not provide significant benefits.**

### 4.2.6 Custom Serializer

```java
public class Alert implements Serializable {
	private final int alertId;
	private String stageId;
	private final String alertLevel;
	private final String alertMessage;
	
	public Alert(int alertId, String stageId, String alertLevel, String alertMessage) {
		this.alertId = alertId;
		this.stageId = stageId;
		this.alertLevel = alertLevel;
		this.alertMessage = alertMessage;
	}
	public int getAlertId() {
		return alertId;
	}
	public String getStageId() {
		return stageId;
	}
	public void setStageId(String stageId) {
		this.stageId = stageId;
	}
	public String getAlertLevel() {
		return alertLevel;
	}
	public String getAlertMessage() {
		return alertMessage;
	} 
}
```

```java
public class AlertKeySerde implements Serializer<Alert>, Deserializer<Alert> {
  public byte[] serialize(String topic, Alert key) {
    if (key == null) {
      return null;
    }
    return key.getStageId().getBytes(StandardCharsets.UTF_8);
  }
  public Alert deserialize(String topic, byte[] value) {
    //We will leave this part for later
    return null;
  }
  //... 
}
```

This example is straightforward since the focus is on the technique of using a Serde. Other options of Serdes that are often used are JSON and Avro implementations.

Serde - it means that the serializer and deserializer are both handled by the same implementation of that interface.

### 4.2.7 Producer Interceptor

One of the other options when using the producer is creating producer interceptors. They were introduced in KIP-42 whose main goal was to help support measurement and monitoring. 

Kafka also reports various internal metrics using Oracle JMXTM that can be used if monitoring is the main concern.

**If you do create an interceptor, remember to set the producer config interceptor.classes.**

### 4.2.8 Our own partition code

So far, in our examples of writing to Kafka, the data has been directed to a topic, and no additional metadata has been provided from the client. Since the topics are made up of partitions that sit on the brokers, Kafka has provided a default way to decide where to send a message to a specific partition. **The default for a message with no key (which has been used in the examples so far) was a round-robin assignment strategy prior to Apache Kafka version 2.4. Versions after 2.4 changed no key assignments to use a sticky partition strategy in which messages were batched to the same partition until the batch was full, and a new partition was randomly assigned.** If a key is present, then the key is used to create a hashed value. That hash is then used to determine a specific partition. **However, sometimes we have some specific ways we want our data to be partitioned. One way to take control of this is to write our own custom partitioner class.**

The client has the ability to control what partition it writes, too, by configuring a custom partitioner. This can be one way to load balance the data over the partitions. This might come into the picture if we have specific keys that we want to be treated in a special way. Let’s look at an example for our sensor use-case.

Some sensors' information might be more important than others, i.e., they might be on the critical path of our e-bike that could cause downtime if not addressed. Let’s say we have four levels of alerts: Critical, Major, Minor, and Warning. We could create a partitioner that would place the different levels in different partitions. Our consumer clients could always make sure to read the critical alerts before processing the others and have their own service-level agreements (SLA) based on the alert level.

```java
public class AlertLevelPartitioner implements Partitioner {
  
	public int partition(final String topic, final Object objectKey, final byte[] keyBytes,
                             final Object value, final byte[] valueBytes, final Cluster cluster) {
    
          final List<PartitionInfo> partitionMetaList = cluster.availablePartitionsForTopic(topic);
          final int criticalPartition = 0;
          final String key = ((Alert) objectKey).getAlertLevel(); //1
          return key.contains("CRITICAL") ? criticalPartition : Math.abs(key.hashCode()) % partitionMetaList.size(); //2
    }
//... 
}
```

1) We are casting the value to a String to check the value.
2) Critical alerts should end up on partition 0. Other alerts locations will be based on the key’s hashcode.

By implementing the Partitioner interface, we can use the partition method to send back the specific partition we would have our producer write to.

```java
Properties props = new Properties();
//...
props.put("partitioner.class", "org.kafkainaction.partitioner.AlertLevelPartitioner");
```

## 4.3 Generating data for our requirements

Let’s start with the audit checklist that we designed in Chapter 3 for use with Kafka in an e-bike factory. One requirement was that there was no need to correlate (or group together) any events. Another requirement was to make sure we don’t lose any messages.

```java
Properties producerProperties = new Properties();
producerProperties.put("bootstrap.servers", "localhost:9092,localhost:9093,localhost:9094");
producerProperties.put("acks", "all");          //1
producerProperties.put("retries", "3");         //2
producerProperties.put("max.in.flight.requests.per.connection", "1");
```

1) We are using ``acks=all`` to get the strongest guarantee we can.
2) We are letting the client do retries for us in cases of failure, so we don’t have to implement our own failure logic.

Since we do not have to correlate (group) any events together, we are not using a key for these messages. However, there is an essential piece of code that we do want to change in order to wait for the result before moving on to other codes. This means waiting for the response to complete synchronously. **The ``get`` method is how we are waiting for the result to come back before moving on in the code.**

```java
RecordMetadata result = producer.send(producerRecord).get();
log.info("offset = {}, topic = {}, timestamp = {}}, result.offset(), result.topic(), result.timestamp());
```

Waiting on the response directly in a synchronous way ensures that the code is handling each record’s results as they come back before another message is sent. The focus is on delivering the messages without loss over speed.

Another goal of our design for the factory was to capture the alert trend status of our stages and track their alerts over time. Since we care about the information for each stage (and not all sensors at a time), it might be helpful to think of how we are going to group these events. **In this case, since each stage id will be unique, it makes sense that we can use that id as a key.**

```java
public class AlertTrendingProducer {
  
    private static final Logger log = LoggerFactory.getLogger(AlertTrendingProducer.class);
  
    public static void main(String[] args) throws InterruptedException, ExecutionException {
    	Properties producerProperties = new Properties();
    	producerProperties.put("bootstrap.servers", "localhost:9092,localhost:9093,localhost:9094");
    	producerProperties.put("key.serializer", "org.kafkainaction.serde.AlertKeySerde");                                            //1
    	producerProperties.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
    
    try (Producer<Alert, String> producer = new KafkaProducer<>(producerProperties)) {
        Alert alert = new Alert(0, "Stage 0", "CRITICAL", "Stage 0 stopped");
    	ProducerRecord<Alert, String> producerRecord = new ProducerRecord<>("kinaction_alerttrend", alert, alert.getAlertMessage());  //2
    	RecordMetadata result = producer.send(producerRecord).get();
    	log.info("offset = {}, topic = {}, timestamp = {}", result.offset(), result.topic(), result.timestamp());
    } 
  }
}
```

1) We need to tell our producer client how to serialize our custom Alert object into a key.
2) Instead of null for the 2nd parameter, we put the actual object we wish to used to help populate the key.

If we use the default hashing of the key partition assigner, the same key should produce the same partition assignment, and nothing will need to be changed. In other words, the same stage ids (the keys) will be grouped together just by using the correct key. We will keep an eye on the distribution of the size of the partitions to note if they become uneven in the future, but we will go with this for the time being.

Our last requirement was to have any alerts quickly processed to let operators know about any critical outages. We do want to group by the stage id in this case as well. One reason is that we can tell if that sensor was failed or recovered (any state change) by looking at only the last event for that stage id. We do not care about the history of the status checks, only the current scenario. In this case, we also want to partition our alerts.

The intention is for us to have the data available in a specific partition so those that process the data would have access to the critical alerts specifically and could go after other alerts (in other partitions) when those were handled.

```java
public class AlertProducer {
  
  public static void main(String[] args) {
    Properties producerProperties = new Properties();
    producerProperties.put("bootstrap.servers", "localhost:9092,localhost:9093");
    producerProperties.put("key.serializer", "org.kafkainaction.serde.AlertKeySerde");                                          //1
    producerProperties.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
    producerProperties.put("partitioner.class", "org.kafkainaction.partitioner.AlertLevelPartitioner");                         //2
    try (Producer<Alert, String> producer = new KafkaProducer<>(producerProperties)) {
      Alert alert = new Alert(1, "Stage 1", "CRITICAL", "Stage 1 stopped");
      ProducerRecord<Alert, String> producerRecord = new ProducerRecord<>("kinaction_alert", alert, alert.getAlertMessage()); //3
      producer.send(producerRecord, new AlertCallback());
    }
  } 
}
```

1) We are reusing our custom Alert key serializer.
2) We are using the property ``partitioner.class`` to set our custom partitioner class we created above.
3) This is the first time we have used a callback to handle the completion or failure of an asynchronous send call.

One addition we see above is how we are adding a callback to run on completion. While we said that we are 100% concerned with message failures from time to time due to the frequency o events, we want to make sure that we do not see a high failure rate that could be a hint at our application-related errors.

```java
public class AlertCallback implements Callback {
  
  private static final Logger log = LoggerFactory.getLogger(AlertCallback.class);
  
  public void onCompletion(RecordMetadata metadata, Exception exception) {
    if (exception != null) {
      log.error("Error sending message:", exception);
    } else {
      log.info("Message sent: offset = {}, topic = {}, timestamp = {}", metadata.offset(), metadata.topic(), metadata.timestamp());
    }
  }
}
```

While we will focus on small sample examples in most of our material, I think that it is helpful to look at how a producer is used in a real project as well. As mentioned earlier, Apache Flume can be used alongside Kafka to provide various data features. When Kafka is used as a sink, Flume places data into Kafka.

Flume Sing Configuration:

```properties
a1.sinks.k1.kafka.topic = helloworld
a1.sinks.k1.kafka.bootstrap.servers = localhost:9092
a1.sinks.k1.kafka.producer.acks = 1
a1.sinks.k1.kafka.producer.linger.ms = 1
a1.sinks.k1.kafka.producer.compression.type = snappy
```

### 4.3.1 Client and Broker Versions

One important thing to note is that Kafka broker and client versions do not always have to match. If you are running a broker that is at Kafka version 1.0 and the Java producer client you are using is at 0.10 (using these versions as an example since they were the first to handle this mismatch!), the broker will handle this upgrade in the message version. **However, because you can, it does not mean you should do it in all cases.**

# Chapter 5. Consumers: unlocking data

## 5.1 Introducing the Consumer

Why is it important to know that the consumer is subscribing to topics (pulling messages) and not being pushed to instead? The power of processing control shifts to the consumer in this situation.

**Consumers themselves control the rate of consumption.**

```java
public class WebClickConsumer {

    final static Logger log = LoggerFactory.getLogger(WebClickConsumer.class);
    private volatile boolean keepConsuming = true;

    public static void main(String[] args) {
        Properties props = new Properties();

        props.put("bootstrap.servers", "localhost:9092,localhost:9093,,localhost:9094");
        props.put("group.id", "webconsumer");                                                               //1
        props.put("enable.auto.commit", "true");
        props.put("auto.commit.interval.ms", "1000");
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");          //2
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");

        WebClickConsumer webClickConsumer = new WebClickConsumer();
        webClickConsumer.consume(props);
        Runtime.getRuntime().addShutdownHook(new Thread(webClickConsumer::shutdown));
    }

    private void consume(Properties props) {

        try (KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props)) {                         //3
            consumer.subscribe(Arrays.asList("webclicks"));                                                 //4

            while (keepConsuming) {                                                                         //5
                ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(500));
                for (ConsumerRecord<String, String> record : records) {
                    log.info("[Consumer Record] offset = {}, key = {}, value = {}", record.offset(), record.key(), record.value());
                    log.info("value = {}", Double.parseDouble(record.value()) * 1.543);
                }
            }
        }
    }

    private void shutdown() {
        keepConsuming = false;
    }
}
```

1) ``group.id`` defined and will be discussed with consumer groups. 
2) Deserializers for the key and values defined.
3) Properties are passed into the KafkaConsumer constructor. 
4) Subscribe to one topic: ``webclicks``.
5) The logic polls in an infinite loop for records that will come from the topic.

LinkedIn’s use-case of user activity events. Let’s say that we have a specific formula that uses the time a user spends on the page as well as the number of interactions they had on the page that is sent as a value to a topic to project future click rates with a new promotion. Imagine we run the consumer and process all of the messages on the topic and are happy with our application of the formula (in this case multiplying by a magic number).

After generating a value for every message in the topic we find out that our modeling formula wasn’t correct! So what should we do now? Attempt to recalculate the data we have from our end-results (assuming the correction would be harder than in this example) and then apply a new formula? This is where we could use our knowledge of consumer behavior in Kafka to rather replay the messages we already processed.

**By having that raw data retained, we do not have to worry about trying to recreate that original data.** Developer mistakes, application logic mistakes, and even dependent application failures have a chance to be corrected because the data is not removed from our topics once it is consumed.

## 5.2 Important Consumer Options

You will notice a couple of properties that are related to the ones that were important for the producer clients as well. For one, ee always need to know the brokers we can attempt to connect to on the startup of the client. One minor gotcha is to make sure that you are using the deserializers for the key and values that match the serializers you produced the message with.

| Key  | Purpose |
| ------------- | ------------- |
| bootstrap.servers  | One or many Kafka brokers to connect on startup  |
| value.deserializer  | The class that is being used for deserialization of the value  |
| key.deserializer  | The class that is being used for deserialization of the key  |
| group.id  | A name that will be used to join a consumer group  |
| client.id  | An id used for being able to identify a client (will use in chapter 10)  |
| heartbeat.interval.ms  | Interval for consumer’s pings to the group coordinator  |

### 5.2.1 Understanding Tracking Offsets

One of the items that we have only talked about in passing so far is the concept of offsets. Offsets used as an index position in the log that the consumer sends to the broker to let it know what messages it wants to consume and from where. If you think back to our console consumer example, we used the flag . This sets the consumer’s --from-beginning configuration parameter auto.offset.reset to earliest behind the scenes. With that configuration, you should see all the records on that topic for the partitions you are connected to-even if they were sent before you started up the console consumer.

**If you didn’t add that option, the default is latest.**

In this case, you will not see any messages from the producer unless you send them after you start the consumer.

Consumers usually read from the consumer’s partition leader replica. Follower replicas are used in the case of failure, after a new replica assumes leadership, but are not actively serving consumer fetch requests.

As a side note, **if you do need to fetch from a follower replica due to an issue like network latency concerns - like having a cluster that stretches across data-centers, KIP-392 introduced this ability in version 2.4.0.**

Partitions play a very important role in how we can process messages. While the topic is a logical name for what your consumers are interested in, they will read from the leader replicas of the partitions that they are assigned to. But how do consumers figure out what partition to connect to? And not just what partition, but where the leader exists for that partition? For each group of consumers, a specific broker will take on the role of being a group coordinator. The consumer client will talk to this coordinator in order to get a partition assignment along with other details it needs in order to consume messages.

**To be clear, in the instance where there are more partitions than consumers, consumers will handle more than one partition if needed.**

Since the number of partitions determines the amount of parallel consumers you can have, some might ask why you don’t always choose a large number like having 500 partitions. This quest for higher throughput is not free. This is why you will need to choose what best matches the shape of your data flow. **One key consideration is that many partitions might increase end-to-end latency.** If milliseconds count in your application, you might not be able to wait until a partition is replicated between brokers. **This replication is key to having in-sync replicas and is done before a message is available to be delivered to a consumer. You would also need to make sure that you watch the memory usage of your consumers. If you do not have a 1-to-1 mapping of partitions to a consumers, each consumer’s memory requirements may increase as it is assigned more partitions.**

We have addressed the consumer running in an infinite loop, but what if you do want to stop a consumer, what is the correct way?

The proper way includes calling a close method on the consumer. This proper shutdown method allows the group coordinator to receive notification about membership of the group being changed due to the client leaving rather than determining this from timeouts or errors from that consumer as it disappears.

By calling the public method shutdown, a different class can flip the boolean and stop our consumer from polling for new records.

```java
public class StopConsumer implements Runnable {

    private final KafkaConsumer<String, String> consumer;
    private final AtomicBoolean stopping = new AtomicBoolean(false);

    ...

    public StopConsumer(KafkaConsumer<String, String> consumer) {
        this.consumer = consumer;
    }

    public void run() {
        try {
            consumer.subscribe(Arrays.asList("webclicks"));
            while (!stopping.get()) {
                ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
                ...
            }
        } catch (WakeupException e) {
            if (!stopping.get()) throw e;
        } finally {
            consumer.close();
        }
    }
    public void shutdown() {
        stopping.set(true);
        consumer.wakeup();
    }
}
```

If your consumer process does not shutdown correctly, Kafka should still be informed of a down consumer by using a heartbeat. If no heartbeat occurs within a specific time, the coordinator will consider the client to be gone and reassign that client’s partitions to the remaining active consumers.

## 5.3 Consumer Groups

Probably the most important reason is that scaling is impacted by either adding more or less consumers to a group. Consumers that are not part of the same group do not share the same coordination of offset knowledge.

```java
Properties props = new Properties();
props.put("group.id", "testgroup");
```

An example of a group that would be named testgroup. If you instead had made up a new group.id (like a random GUID) it means that you are going to be starting as a new consumer with no stored offsets and with no other consumers in your group

**If you join an existing group (or one that had offsets stored already), your consumer will be able to share work with others or even be able to resume where it left off reading from any previous runs.**

It is often the case that you will have many consumers reading from the same topic. An important detail to decide on if you need a new group id is whether your consumers are working as part of one application or as separate logic flows. Why is this important? Think of two use cases for data that came from a human resources system. One team is wondering about the number of hires from specific states, and the other team is more interested in the data for the impact on travel budgets for interviews. So would anyone on the first team care about what the other team was doing or would either of them want to consume only a portion of the messages? Likely not! So how can we keep this separation? The answer is by making each application have its own specific group.id. Each consumer that uses the same group.id as another consumer will be considered to be working together to consume the partitions and offsets of the topic as one logical application.

*Note.*

**Can multiple Kafka consumers read same message from the partition?**

It depends on Group ID. Suppose you have a topic with 12 partitions. If you have 2 Kafka consumers with the same Group Id, they will both read 6 partitions, meaning they will read different set of partitions = different set of messages. If you have 4 Kafka cosnumers with the same Group Id, each of them will all read three different partitions etc.

But when you set different Group Id, the situation changes. If you have two Kafka consumers with different Group Id they will read all 12 partitions without any interference between each other. Meaning both consumers will read the exact same set of messages independently. If you have four Kafka consumers with different Group Id they will all read all partitions etc.

## 5.4 The Need for Offsets

While we are going through our usage patterns so far, we have not talked too much about how we keep track of what each client has read. Let’s briefly talk about how some message brokers handle messages in other systems. In some systems, consumers do not track what they have read, they pull the message, and then it will not exist on a queue anymore after it has been acknowledged. This works well for a single message that needs to have exactly one application process it. Some systems will use topics in order to publish the message to all those that are subscribers. And often, future subscribers will have missed this message entirely since they were not actively part of that receiver list when the event happened.

In systems where the message would be consumed and not available for more than one consumer, this is needed for separate applications to each get a copy.

![chapter-5-figure-5.PNG](pictures/chapter-5-figure-5.PNG)

You can imagine the copies grow as an event becomes a popular source of information. Rather than have entire copies of the queue (besides those for replication/failover), Kafka can serve multiple applications from the same partition leader replica.

In addition, since many applications could be reading the same topic, it is important that the offsets and partitions are tied-back and specific to a certain consumer group. The **key coordinates to let your consumer clients work together is a unique blend of the following: group, topic, and partition number.**

### 5.4.1 Group Coordinator

**There is usually one broker that takes over the important job of working with offset commits for a specific consumer group.** The offset commit can be thought of the last message consumed by a specific member of that consumer group. These duties are handled by the GroupCoordinator which helps with being an offset manager.

For example, a consumer that is part of the marketing consumer group and is assigned partition 0 would be ready to read offset 3 next.

Figure 5.7 shows a scenario where the same partitions of interest exist on 3 separate brokers for 2 different consumer groups, marketing and ad sales. The consumers in each group will get their own copy of the data from the partitions on each broker. They do not work together unless they are part of the same group.

![chapter-5-figure-7.PNG](pictures/chapter-5-figure-7.PNG)

One of the neat things about being a part of a consumer group is that when a consumer fails or leaves a group, the partitions that it was reading are re-assigned. An existing consumer will take the place of reading a partition that was once being read by the consumer that dropped out of the group.

One way a consumer can drop out of a group membership is by failing to send a heartbeat to the GroupCoordinator. This heartbeat is the way that the consumer communicates with the coordinator to let it know it is still replying in a timely fashion and working away.

One way for this is to stop the consumer client by either termination of the process or failure due to a fatal exception. If the client isn’t running, it will not be sending messages back to the group coordinator. **Another common issue is the time that your client code could be using to process the last batch of messages (long running code that takes seconds or minutes) before another poll loop.**

*Note*.

**Make sure that heartbeat and how long it takes to process message does not conflict.**

Consumer groups also help when new consumer clients are being added. Take an example of one consumer client reading from a topic made up of 2 partitions. Adding a second consumer client should result in each consumer processing one partition. **If more consumers are present, then those consumers without assignments will be sitting idle.**

### 5.4.2 Partition Assignment Strategy

The property is what determines which ``partition.assignment.strategy`` partitions are assigned to each consumer.

There are three different strategies:
* Range - This assigner uses a single topic to find the number of partitions (ordered by number) and then divides by the number of consumers. If the division is not even, then the first consumers (using alphabetical order) will get the remaining partitions. Note, if you have a topic and partition numbers that often cause partitions to be assigned unevenly, you might see some consumers taking a larger number of partitions over time. Make sure that you see a spread of partitions that your consumers can handle and consider switching the assignment strategy if some consumer clients used up all their resources while others are fine. Figure 5.9 shows how the first of 3 clients will grab 3 out of 8 total partitions and will end up with more partitions than the last client. 
* Round Robin - This strategy is most easily shown by a consumer group that all subscribe to all the same topics. The partitions will be uniformly distributed in that the largest difference between assignments should be one partition. Figure 5.9 shows an example of 3 clients part of the same consumer group assigned in a round-robin fashion for one topic made of 8 partitions. Like dealing cards around a table, the first consumer gets the first partition, the second consumer the second, and so on until the partitions run out. 
* Sticky - This is a strategy that was added in version 0.11.0. The driver is to not only try and distribute topic partitions as evenly as possible but also to have partitions stay with their existing consumer clients when possible. Allowing partition assignments to stay with the same consumers could help speed up a rebalance which could occur during any consumer group changes. Allowing certain consumers to stick to their existing partition assignment rather than make potentially every consumer change assignments is the goal.

![chapter-5-figure-9.PNG](pictures/chapter-5-figure-9.PNG)

You can also create your own strategy. ``AbstractPartitionAssignor`` is an abstract class that does some of the work to help implement your own logic.

### 5.4.3 Manual Partition Assignment

If the assignment of partitions is important to your applications, then you will want to look at how to specify specific partitions in your client code. To implement manual assignment, you would call ``assign`` where you had previously called ``subscribe``.

``TopicPartition`` objects are used to help your code tell Kafka what specific partitions you are interested in for a topic. Passing the TopicPartition objects created to the ``assign`` method takes the place of allowing a consumer to be at the discretion of a group coordinator.

```java
consumer.assign(List.of(new TopicPartition(TOPIC_NAME, 1), new TopicPartition(TOPIC_NAME, 2)));

while (keepConsuming) {
    var records = consumer.poll(Duration.ofMillis(100));
    for (ConsumerRecord<String, String> record : records) {
        log.info("offset = {}, key = {}, value = {}", record.offset(), record.key(),record.value());
        log.info("value = {}", Integer.getInteger(record.value()) * 1.543);
    }
}
```

Due to this assignment decision, it is important to note that each consumer acts on its own and does not coordinate with other consumers in a group. Sharing a group id with another consumer does not give this consumer any benefits like we have seen in examples when a consumer fails. In addition, if you add a partition that your consumer should know about, the consumer client will have to assign itself to see those messages from the new partition, your client won’t learn automatically about and consume any new partitions. **Make sure that you know the responsibilities of managing your own consumers when you reach for this feature.**

## 5.5 Auto or Manual Commit of Offsets

One option is to use is ``enable.auto.commit=true`` which is the default for consumer clients.

Offsets are committed on your behalf on a recurring interval: ``auto.commit.interval.ms``. One of the nicest parts of this option is that you do not make any other calls to commit the offsets that you have consumed. At-least-once delivery is related to automatic commits since Kafka brokers will resend messages again if they were never not automatically acknowledged due to a consumer client failure. But what sort of trouble could we get into? If we are processing messages that we got from our last poll, say in a separate application thread, the automatic commit offset could be marked as being read even if you do not actually have everything done with those specific offsets. What if we had a message fail in our processing that we would need to retry? With our next poll, we could be getting the next set of offsets after what was already committed as being consumed. It is possible and easy to lose messages that look like they have been consumed despite not being processed by your consumer logic.

When looking at what you commit, notice that timing might not be perfect. If you do not call a commit method on a consumer with metadata specifically noting your specific offset to commit, you might have some undefined behavior based on the timing of polls, expired timers, or even your own threading logic. **If you need to be sure to commit a record at a specific time as you process it, or a specific offset in general, you should make sure that you send the offset metadata into the commit method.**

Let’s explore this topic more and talk about using manual commits enabled by: ``enable.auto.commit=false``. At-least-once delivery guarantees can be achieved with this pattern.

As you get a message, let’s stay that you poll a message at offset 100. During processing, the consumer stops because of an error. Since the code never actually committed offset 100, the next time a consumer of that same group starts reading from that partition, it will get the message at offset 100 again. So by delivering the message twice, the client was able to complete the task without missing the message. **On the flip side, you did get the message twice!** If for some reason your processing actually worked and you achieved a successful write, your code will have to handle the fact that you might have duplicates.

Let’s look at some of the code that we would use to manually commit our offsets. **As we did with a producer when we sent a message, we can also commit offsets in a synchronous manner or asynchronous.**

```java
while (keepConsuming) {
    var records = consumer.poll(Duration.ofMillis(100));
    for (var record : records) {
        log.info("offset = {}, key = {}, value = {}",record.offset(), record.key(), record.value());
    }
    consumer.commitSync();
}
```

Also, note that the commit is taking place outside of the loop that goes through all of the returned records. Since no parameters are passed in, the call to commitSync will not commit each record one by one, as the looping logic processed each record. **Rather, the entire set of record offsets returned from the last will be committed.**

CommitAsync is the path to manually commit without blocking your next iteration. One of the options that you can use in this method call is the ability to send in a callback.

```java
while (keepConsuming) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
    for (ConsumerRecord<String, String> record : records) {
        log.info("[Consumer Record] offset = {}, key = {}, value = {}", record.offset(), record.key(), record.value()); 
    }
    consumer.commitAsync(callback);
}
```

To implement your own callback you need to use the interface: ``OffsetCommitCallback``.

```java
public static void commitOffset(long offset,
                                int partition,
                                String topic,
                                KafkaConsumer<String, String> consumer) {
    OffsetAndMetadata offsetMeta = new OffsetAndMetadata(offset + 1, "");
    Map<TopicPartition, OffsetAndMetadata> offsetMap = new HashMap<>();
    offsetMap.put(new TopicPartition(topic, partition), offsetMeta);
    consumer.commitAsync(offsetMap, (map, e) -> {
        if (e != null) {
            for (TopicPartition key : map.keySet()) {
                log.info("Commit failed: topic {}, partition {}, offset {}", key.topic(),
                        key.partition(), map.get(key).offset());
            }
        } else {
            for (TopicPartition key : map.keySet()) {
                log.info("OK: topic {}, partition {}, offset {}", key.topic(), key.partition(),
                        map.get(key).offset());
            }
        }
    });
}
```

Shows how to create an asynchronous commit with a callback by implementing the OffsetCommitCallback interface. This instance allows us to have log messages to determine our success or failure even though our code is not waiting for a response before moving on to the next instruction.

Why would you want to choose synchronous or asynchronous commit patterns? For one, you need to keep in mind that your latency will be higher if you are waiting for a blocking call. This time factor might be worth it if your requirements include needs for data consistency. What does consistency mean in this instance? It really is a question of how you need to process your data. Do you need to wait to process your messages in order before committing the offsets that have been processed even in the case of failures? If a failure occurred before a commit, does your logic handle reprocessing messages that your application might have seen before but did not mark as committed to Kafka?

## 5.6 Reading From a Compacted Topic

Consumers should be aware if they are reading from a compacted topic. Chapter 7 will go further into how these topics work to, in effect, **update records that have the same key value. In short, the partition log is compacted by Kafka on a background process and records with the same key can be removed except for the last one.** If you do not need a history of messages, but rather just the last value, you might wonder how this concept works with an immutable log that only adds records to the end. At this point, the biggest gotcha for consumers that might cause an error is that when reading records, consumers might still get multiple entries for a single key! How is this possible? Since compaction runs on the log files that are on-disk, compaction might not see every message that exists in memory during cleanup. Clients will need to be able to handle this case where there is not only one value per key. Please have logic in place to handle duplicate keys and, if needed, ignore all but the last value.

## 5.7 Reading for a Specific Offset

While there is no lookup of a message by a key option in Kafka, it is possible to seek to a specific offset. Thinking about our log of messages being an ever-increasing array with each one message having an index, we have a couple of options:
* starting from the beginning;
* going to the end;
* starting at a given offset;
* finding offsets based on times.

### 5.7.1 Start at the beginning

The important configuration to note for this behavior is to set to earliest . Another technique ``auto.offset.reset`` that can be used is to run the same logic but use a different group id.

```java
Properties props = new Properties();
props.put("group.id", UUID.randomUUID().toString());
props.put("auto.offset.reset", "earliest");
```

Also, the Java client consumer has the following method available: ``seekToBeginning``. This is yet another way to achieve the same result as the code.

### 5.7.2 Going to the end

Using a UUID isn’t necessary except for testing when you want to make sure that you don’t find a consumer offset from before and will instead default to the latest offset Kafka has for your subscriptions.

```java
Properties props = new Properties();
props.put("group.id", UUID.randomUUID().toString());
props.put("auto.offset.reset", "latest");
```

The Java client consumer also has the following method available: ``seekToEnd``. This is another way to achieve the same result as the configuration.

### 5.7.3 Seek to an Offset

Kafka also gives you the ability to find any offset and go directly to it. The seek action sets a specific offset that will be used by the next poll on the consumer. **The topic and partition are needed as the coordinates for giving your consumer the information it needs to set that offset.**

Shows an example of seeking to the offset number 5000 for a single topic and partition combination. A ``TopicPartition`` object is used to provide the topic name and partition of interest which is partition one in this example. The ``seek`` method takes the topic information to set up the provided offset as the starting place of the next poll of the consumer.

```java
long offset = 5000L;
TopicPartition partitionOne = new TopicPartition(topicName, 1);
consumer.seek(topicPartition, offset);
```

### 5.7.4 Offsets For Times

One of the trickier offset search methods is ``offsetsForTimes``. This method allows you to send a map of topics and partitions as well as a timestamp for each in order to get a map back of the offset and timestamp for those topics and partitions given. This can be used in situations where a logical offset is not known, but a timestamp is known. For example, if you have an exception that was logged related to an event, you might be able to use a consumer to determine the data that was processed around your specific timestamp.

Shows, we have the ability to retrieve the offset and timestamps per a topic/partition when we map each to a timestamp. After we get our map of metadata returned from the ``offsetsForTimes`` call, we then can seek directly to the offset we are interested in by seeking to the offset returned for each respective key.

One thing to be aware of is that the offset returned is the first message with a timestamp that makes your criteria. However, due to the producer resending messages on failures or variations in when timestamps are added (by consumers, perhaps), this timestamp is not an official order.

## 5.8 Reading Concerns

One thing that we have talked about is that the consumer is in the driver’s seat about when data comes to your application. However, let’s picture a common scenario. We had a minor outage but bought back all of our consumers with new code changes. Due to the amount of messages we missed, we suddenly see our first poll bring back a huge amount of data. Is there a way that we could have controlled this being dumped into our consumer application memory? ``fetch.max.bytes`` is one property that will help us avoid too many surprises. The value is meant to set the max amount of bytes per request.

However, the consumer might make parallel fetch calls at the same time from different partitions and impact the true total on your client.

## 5.9 Retrieving data for our factory requirements

Let’s try to use this information we gathered about how consumers work to see if we can start working on our own solutions designed in chapter 3 for use with Kafka in our e-bike factory: but from the consumer client perspective in this chapter.

One requirement was that there was no need to correlate (or group together) any events across the individual events. This means that there are no concerns on the order or need to read from specific partitions, any consumer reading any partition should be good. Also, another one of the requirements was to not lose any messages. One safe way to make sure that our logic is executed for each audit event is to manually commit the offset per record after it is consumed. To control the commit as part of the code: ``enable.auto.commit`` will be set to false.

An example of leveraging a synchronous commit after each record is processed for the Audit feature. Details of the next offset to consume in relation to the topic and partition of the offset that was just consumed is sent as a part of each loop through the records. Note that it might seem odd to add 1 to the current offset, but this offset sent to your broker is supposed to be the offset of the next message that will be consumed. The method ``commitSync`` is called and passed the offSet map containing the offset of the record that was just processed.

```java
    props.put("enable.auto.commit", "false");
    
    try (KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props)) {
        consumer.subscribe(List.of("kinaction_audit"));
        while (keepConsuming) {
            var records = consumer.poll(Duration.ofMillis(100));
            for (ConsumerRecord<String, String> record : records) {
                log.info("offset = {}, key = {}, value = {}",
                        record.offset(), record.key(), record.value());
                OffsetAndMetadata offsetMeta = new OffsetAndMetadata(record.offset() + 1, "");      //1
                Map<TopicPartition, OffsetAndMetadata> offsetMap = new HashMap<>();
                offsetMap.put(new TopicPartition("audit", record.partition()), offsetMeta);         //2
                consumer.commitSync(offsetMap);                                                     //3
            }
        }
    }
```

1) Adding 1 to the current offset determines the next offset that will be read.
2) The offset map allows for a topic and partition key to be related to a specific offset.
3) ``commitSync`` called to commit the offsets processed.

Another goal of the design for our e-bike factory was to capture our stage alert status and track the stage alert trend over time. Even though we know our records have a key that is the stage id, there is no need to consume a group at a time or worry about the order. Since message loss isn’t a huge concern, allowing auto commit of messages is good enough in this situation.

```java
props.put("enable.auto.commit", "true");
props.put("key.deserializer", "org.kafkainaction.serde.AlertKeySerde");
props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
KafkaConsumer<Alert, String> consumer = new KafkaConsumer<Alert, String>(props);
consumer.subscribe(Arrays.asList("kinaction_alerttrend"));
while (true) {
    ConsumerRecords<Alert, String> records = consumer.poll(Duration.ofMillis(100));
    for (ConsumerRecord<Alert, String> record : records) {
        log.info("offset = {}, key = {}, value = {}", record.offset(), record.key().getStageId(), record.value());
    }
}
```

The last requirement was to have any alerts quickly processed to let operators know about any critical issues. Since the producer used a custom, we will ``Partitioner`` assign a consumer directly to that same partition to alert on critical issues. Since a delay in case of other alerts is not desirable, the commit will be for each offset in an asynchronous manner.

*Note*.

Trade-offs: latency vs. data consistency

* If you have to ensure the data consistency, choose ``commitSync()`` because it will make sure that, before doing any further actions, you will know whether the offset commit is successful or failed. But because it is sync and blocking, you will spend more time on waiting for the commit to be finished, which leads to high latency.
* If you are ok of certain data inconsistency and want to have low latency, choose ``commitAsync()`` because it will not wait to be finished. Instead, it will just send out the commit request and handle the response from Kafka (success or failure) later, and meanwhile, your code will continue executing.

Consumer client logic focused on critical alerts assigning themselves to the specific topic and partition that would have been used when producing alerts when the custom partitioner class ``AlertLevelPartitioner`` was used. In this case, it was partition number 0. The consumer assigns itself the partition rather than subscribing to the topic. For each record that comes back from the consumer poll, an asynchronous commit will be used with a callback. A commit of the next offset to consume will be sent to the broker and should not block the consumer from processing the next record. The asynchronous commit method is sent the topic and partition mapping to offset information as well as a callback to run. This ability to handle any issues with a callback at a later time helps make sure that other records can be processed quickly.

```java
props.put("enable.auto.commit", "false");
KafkaConsumer<Alert, String> consumer = new KafkaConsumer<Alert, String>(props);
TopicPartition partitionZero = new TopicPartition(kinaction_alert", 0);
consumer.assign(Arrays.asList(partitionZero));

while (true) {
    ConsumerRecords<Alert, String> records = consumer.poll(Duration.ofMillis(100));
    for (ConsumerRecord<Alert, String> record : records) {
        log.info("offset = {}, key = {}, value = {}",
                record.offset(), record.key().getStageId(), record.value());
        commitOffset(record.offset(), record.partition(), topicName, consumer);
    }
}
...
public static void commitOffset(long offset,int part, String topic,
                                KafkaConsumer<Alert, String> consumer) {
    OffsetAndMetadata offsetMeta = new OffsetAndMetadata(offset + 1, "");
    Map<TopicPartition, OffsetAndMetadata> offsetMap = new HashMap<TopicPartition, OffsetAndMetadata>();
    offsetMap.put(new TopicPartition(topic, part), offsetMeta);
    OffsetCommitCallback callback = new OffsetCommitCallback() {
    ...
    };
    consumer.commitAsync(offsetMap, callback);
}
```

## 5.10 Summary

* Consumer clients provide developers a way to get data out of Kafka. As with producer clients, consumer clients have a large number of configuration options available for clients to set rather than custom coding being needed.
* Consumer groups allow more than one client to work as a group to process records. By grouping together, clients can process data in parallel. Kafka coordinates changes on the groups behalf such as clients joining or dropping from the group dynamically.
* Offsets represent the position of a record in regards to a commit log that exists on a broker. By using offsets, consumers can control where they want to start reading data. This offset can even be a previous offset that it has already seen (ability to replay records again).
* Consumers can read data in a synchronous or an asynchronous manner. If asynchronous methods are used, the consumer can leverage code in callbacks to run logic once data is received.

# Chapter 6. Brokers

## 6.1 Introducing the Broker

Being a broker in Kafka means being a part of a cluster of machines - brokers work together with other brokers to form the core of the system.

Rack awareness - knowing which physical server rack a machine is hosted on. Kafka has a rack awareness feature that will make replicas for a partition exist physically on separate racks. This is important since if all of the servers that make up our cluster are on the same rack, and the entire rack is offline due to a power or network issue, it would be the same as if we had lost our entire data center.

When seting up our own Kafka cluster, it is important to know that we have another cluster to be aware of: ZooKeeper.

## 6.2 Role of ZooKeeper

At this point, ZooKeeper is a key part of how the brokers work and is a requirement for Kafka to run.

Since ZooKeeper needs to have a quorum (a minimum number) in order to elect leaders and reach a decision, this cluster is indeed important for our brokers. ZooKeeper itself holds information on the brokers that are in the cluster. This list should be a current membership list and can change over time. How do brokers join this list? When a broker starts, it will have a unique id that will be listed on ZooKeeper. This id can either be created with the configuration ``broker.id`` or ``broker.id.generation.enable=true``. Nodes can leave the group as well.

ZooKeeper also helps Kafka make decisions - like electing a new controller. **The controller is a single broker in a cluster that takes on additional responsibilities** and will be covered in more depth later in this chapter. Any broker could become the controller, but only one, and it’s important that all of the brokers know who the controller is. ZooKeeper helps the brokers by coordinating this assignment and notification.

With all of this interaction with the brokers, it is very important that we have ZooKeeper running when starting our brokers. **The health of the ZooKeeper cluster will impact the health of our Kafka brokers.**

Certain frameworks versions you use may also still provide a means of connecting our client application with our Zookeeper cluster due to legacy needs. **Avoid direct dependencies on ZooKeeper.**

Using the Kafka tool which is located in the folder ``zookeeper-shell.sh`` bin of our Kafka installation, we can connect to a ZooKeeper host in our cluster and look at how data is stored. Looking at the path ``/brokers/topics`` we will see a list of the topics that we have created. At this point, we should have the ``kinaction_helloworld`` topic in the list. We might also see the topic ``__consumer_offsets`` which is created once we use a consumer client with our cluster. That topic is one that we did not need to create, it is a private topic used internally by Kafka. This topic stores the committed offsets for each topic and partition by consumer group id. Knowing about these internal topics helps to avoid surprises when we see topics in our cluster that we didn’t create.

```shell
bin/zookeeper-shell.sh localhost:2181 #Connect to local ZooKeeper instance
ls /brokers/topics                    #List all the topics with ls command
# OR
#Using the kafka-topics command connect to ZooKeeper as well to provide the same information
bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

Note that we can also use a different Kafka tool, ``kafka-topics.sh`` to see the list of topics as well and get the same results! Both commands connect to ZooKeeper for their data but do so with a different command interface!

It is important when reading anything about ZooKeeper and Kafka to note that KIP-500: Replace ZooKeeper with a Self-Managed Metadata Quorum is currently in progress at the time of publication.

Even when ZooKeeper is no longer helping to power Kafka, we might need to work with clusters that have not migrated yet.

## 6.3 The Job of a Kafka broker

Being a Kafka broker means being able to coordinate with the other brokers as well as talking to ZooKeeper. In testing, or working with proof-of-concept clusters, we might have only one broker node. However, in production, we will almost always have multiple brokers.

## 6.4 Settings at the Broker Level

Recall the setup steps to create our first brokers from Appendix A, we modified a file titled ``server.properties``. This file is a common way to pass a specific configuration to a broker instance and was passed as a command-line argument to the broker startup shell script. As with the producer and consumer configuration, look for the IMPORTANCE label of High in the documentation listed at ``kafka.apache.org/documentation/#brokerconfigs``. The ``broker.id`` configuration property should always be set to an integer value that will be unique in the cluster.

One of the clearest and most important examples of a default would be the property: ``default.replication.factor``. This configuration property controls the number of replicas that will be used when a topic is created without the option itself set. So let’s say that we want 3 copies of the data as a default. However, let’s say that we have other topics whose data is less important in a recovery situation. To make this more concrete, let’s think of a website that handles traffic for an online bakery. The order information from customers is considered by the owners to be important enough to have multiple copies. They probably wouldn’t want to mess up someone’s wedding cake details! However, the section of their website that handles chat history might not be worth attempting to recover in a failure. Kafka allows us to do both reliable and not-as-reliable structures for our topics.

Let’s do an example of what happens when we have only one copy of our data and the broker it is on goes down. Make sure that our local test Kafka is cluster running with three nodes and create a topic with a ``replication.factor`` of 1.

```shell
bin/kafka-topics.sh --create \
--bootstrap-server localhost:9092 \
--replication-factor 1 \
--partitions 1 \
--topic kinaction_one_replica

bin/kafka-topics.sh --describe --bootstrap-server localhost:9092 \
--topic kinaction_one_replica

Topic: one-replica PartitionCount: 1 ReplicationFactor: 1 Configs:
Topic: kinaction_one_replica Partition: 0 Leader: 2 Replicas: 2 Isr: 2
```

We see that there is only one value in the fields: Partition, Leader, Replicas, and In-Sync Replicas (ISR) and that the broker will be the same id value. This means that the entire topic depends on that one broker being up and working. If we terminated the broker with id 2 in this example and then tried to consume a message from that topic, we would get a message like the following: ``1 partitions have leader brokers without a matching listener``. Since there were no replica copies for the partition of the topic, there is no easy way to keep producing or consuming to that topic without recovery of that broker.

### 6.4.1 Kafka’s Other Logs: Application Logs

As with most applications, Kafka provides logs for letting us know what is going on inside of the application.

When we start a broker, the application log directory is in the Kafka base installation directory under the folder ``logs/``. We can change this location by editing the ``config/log4j.properties`` file and the value for ``kafka.logs.dir``.

### 6.4.2 Server Log

**The server log, ``server.log`` is where we would look if there is a startup error or an exception that terminates the broker.** It seems to be the most natural place to look first for any issues. Look or use the grep command for the heading ``KafkaConfig`` values. This is helpful since many errors and unexpected behaviors can be traced back to configuration issues on startup.

**Also, it is often a good idea to have these logs on a different disk altogether than the one that stores our message logs.**

Something else to mention in regards to these logs is that they are located on each broker. **They are not aggregated by default into one location.**

## 6.5 Controllers Duties

As we discussed in Chapter 2, each partition will have a single leader replica. A leader replica will reside on a single broker at any given time. A broker can host the leader replica of multiple partitions, and any broker in a cluster can host leader replicas, but only one broker in the cluster will act as the controller. The role of the controller is to manage the state of partitions and replicas for the entire cluster. The controller also will perform other administrative actions like partition reassignment, creating topics, deleting topics, and creating partitions.

*Note.*

**There is only one controller. Even if cluster consists of 100 brokers - only one controller.**

The controller leverages ZooKeeper to detect restarts and failures in the cluster as a whole. If a broker is restarted, the controller will send the leader information as well as the In-Sync Replica (ISR) information for the broker re-joining the cluster. For a broker failure, the controller will select a new leader and update the ISR. These values will be persisted to ZooKeeper. In order to maintain broker coordination, it also sends the new leader and ISR changes to all other brokers in the cluster. **While a broker is serving as the controller, it can still perform its traditional broker role as well.**

When the controller broker fails, the cluster can recover due to having the data in ZooKeeper, however there is a higher operational cost to restarting the controller, as this kicks off the controller election process. **So, when we consider a rolling upgrade of a cluster, shutting down and restarting one broker at a time, it is best to do the controller last.**

To figure out which broker is the current controller, we can use the zookeeper-shell script to look up the id of the broker.

```shell
bin/zookeeper-shell.sh localhost:2181
get /controller
```

There will also be a controller log file with the name: ``controller.log`` that serves as an application log on broker 0 in this case. This will be important when we are looking at broker actions and failures. The ``state-change.log`` is also useful as it records the decisions that the controller logic completed.

## 6.6 Partition Replica Leaders and their role

As a quick refresher, **topics are made up of partitions. Partitions can have replicas for fault tolerance. Also, partitions are written on the disks of the Kafka brokers. One of the replicas of the partition will have the job of being the leader. The leader is in charge of handling the writes for that partition from external producer clients. Since this leader is the only one with the newly written data, it also has the job of being the source of data for the replica followers. The In-Sync Replicas (ISR) list is maintained by the leader.** Thus, it knows which replicas are caught up and have seen all the current messages. The replicas will act as consumers of the leader partition and will fetch the messages. Similar to how external consumers have offset metadata about which messages have been read, metadata is also present with replication offset checkpointing to track which records have been copied to the followers.

Three brokers with ISR = [3,2,1] means that the 3rd broker is a leader and the remaining ones are followers.6.7 In-Sync Replicas (ISRs) Defined

## 6.7 In-Sync Replicas (ISRs) Defined

One of the details to note with Kafka is that **replicas do not heal themselves by default. If you lose a broker on which one of your copies of a partition existed, Kafka does not currently create a new copy. So an important item to look at when monitoring the health of our system might be how many of our ISRs are indeed matching our desired number.**

**It is also important to note that if a replica starts to get too far behind in copying messages from the leader, it might be removed from the list of in-sync replicas (ISRs). The leader will notice if a follower is taking too long and will drop it from its list of followers.**

## 6.8 Unclean Leader Election

An important tradeoff to consider is the importance of uptime vs the need to keep our data from being lost. What if we have no in-sync-replicas but lost our lead replica due to a failure? **When is true, the controller will select a leader ``unclean.leader.election.enable`` for a partition even if it is not up-to-date, and the system will keep running.** The problem with this is that data could be lost since none of the replicas had all of the data at the time of the leader’s failure. While data loss might not be okay for payment transactions, it might be okay for a web analytics engine to miss a hover event on a couple of pages. **The ``unclean.leader.election.enable`` setting is cluster-wide, but can be overridden at the topic level.**

## 6.9 Metrics from Kafka

Kafka, written for the Java virtual machine (JVM), uses Java Management Extensions (JMX) to enable us to peek into the Kafka process.

We’ll use Prometheus to extract and store the Kafka metrics data. Then we’ll send that data to Grafana to produce helpful graphical views.

For the Kafka exporter, let’s work with a tool available at github.com/danielqsj/kafka_exporter. I prefer the simplicity of this tool since we can just run it and give it one (or a list) of Kafka servers to watch and it might work well for your use-cases as well. Notice that we will get many client and broker specific metrics since there are quite a few options that we might want to monitor. However, this is not a complete list of the metrics available to us.

As noted, the Kafka exporter we are using does not expose every JMX metric. To get more JMX metrics, we can set the ``JMX_PORT`` variable when starting our Kafka processes.

```shell
JMX_PORT=9990 bin/kafka-server-start.sh config/server0.properties
```

If we already have a broker running and do not have this port exposed, we will need to restart the broker to affect this change.

### 6.9.1 Cluster Maintenance

Another decision to think about is whether you want one giant cluster for your entire enterprise or one per product team. Some organizations, like LinkedIn where Kafka was created, have 60 brokers (if not more) as part of one production cluster (and multiple other clusters as well). Other teams prefer to keep smaller clusters that power their smaller product suites. There are many ways to handle a production cluster and both methods listed above can work.

### 6.9.2 Adding a Broker

To add a Kafka broker to our cluster, we just start-up a new Kafka broker with a unique id. That is pretty much it! But, there is something to be aware of in this situation. The new broker will not be assigned any partitions. Any topic partitions that were created before this new broker was added would still exist on the brokers that existed at the time of their creation. Topic partitions are not moved automatically. If we are okay with the new broker only handling new topics, then we don’t need to do anything else. However, if we do want to change the existing layout, we can use the tool: ``kafka-reassign-partitions``. This script is located in the bin directory of our Kafka installation.

### 6.9.3 Upgrading your Cluster

One technique that can be used to avoid downtime for our Kafka applications is the rolling restart.

One very important broker configuration property is ``controlled.shutdown.enable``. Setting this to true will enable the transfer of partition leadership before a broker shuts down. Another setting to be aware of is ``min.insync.replicas``. Since Kafka does not automatically create a new partition replica on a different broker, we could be short one partition from our in-sync-replica total during broker downtime. If we start with 3 replicas and remove a broker that hosts one of those replicas, a minimum replica number of 3 will result in clients not being able to continue until the broker rejoins and syncs up.

**As a reminder, another good practice is to identify which broker is serving as the controller in our cluster and let it be the last server upgraded.**

### 6.9.4 Upgrading your clients

Clients can usually be upgraded after all of the Kafka brokers in a cluster have already been upgraded.

There is also the option to use newer clients without worrying about the broker version.

### 6.9.5 Backups

Kafka does not have a backup strategy like one would use for a database. We don’t take a snapshot or disk backup per se. Since Kafka logs exist on disk, why not just copy the entire partition directories? While nothing is stopping us from doing this, one concern is making a copy of the data directories. To do an accurate copy without missing data, we would probably have to stop the broker before performing the copy. If we wanted a complete copy of a topic with more than one partition, we could be talking about multiple brokers. Rather than performing manual copies and coordinating across brokers, one of preferred option is for a cluster to be backed by a second cluster. Between the two clusters, events are then replicated between topics. One of the earliest tools that you might have seen in production settings is called MirrorMaker. The basic architecture of MirrorMaker logically sits between two separate clusters.

While MirrorMaker is still a popular option, a newer version of this tool called MirrorMaker 2.0 was released with Kafka version 2.4.0. In the directory of the Kafka install directory, we bin will find the shell script named ``kafka-mirror-maker`` as well as the new MirrorMaker 2.0 script named ``connect-mirror-maker``.

**Where the first version of MirrorMaker used consumer and producer clients, version 2.0 leverages the Kafka Connect framework instead.** The framework is reflected in the script name itself connect-mirror-maker. Another difference from the previous version of the tool, is that a configuration file holds the information needed for replication rather than command line arguments.

## 6.10 A Note on Stateful Systems

One of the ongoing trends for many technology companies is the drive to move applications and infrastructure to the cloud. Kafka is an application that definitely works with stateful data stores.

There are some great resources including Confluent’s site on using the Kubernetes Confluent Operator API (www.confluent.io/confluent-operator/) as well as Docker images available to do what you need. Another interesting option is Strimzi ( github.com/strimzi/strimzi-kafka-operator) if you are looking at running your cluster on Kubernetes or OpenShift.

The purpose of the StatefulSet is to manage the Kafka pods and help guarantee ordering and a persistent identity for each pod. **So if the pod that hosts a broker (the JVM process) with id 0 fails, a new pod will be created with that identity (not a random id) and will be able to attach to the same persistent storage volume as before.** Since these volumes hold the messages of the Kafka partitions, the data is maintained. This statefulness helps overcome the sometimes short lives of containers. Each ZooKeeper node would also be in its own pod and part of its own StatefulSet.

## 6.11 Exercise

Since it can be hard to apply some of our new learning in a hands-on manner and since this chapter is heavier on commands than code, it might be helpful to have a quick exercise to explore a different way to discover the metric under-replicated partitions. Besides using something like a Grafana dashboard to see this data, what command-line options can we use to discover this information?

Let’s say that we want to confirm the health of one of our topics named ``kinaction_replica_test`` that we created with each partition having 3 replicas. We would want to make sure we have 3 brokers listed in the ISR list in case there is ever a broker failure. What command should we run to look at that topic and see the current status?

```shell
$ bin/kafka-topics.sh --describe --bootstrap-server localhost:9092 \
--topic kinaction_replica_test
Topic:kinaction_replica_test PartitionCount:1 ReplicationFactor:3 Configs:
Topic: kinaction_replica_test Partition: 0 Leader: 0 Replicas: 1,0,2 Isr: 0,2
```

Notice that the ReplicationFactor is 3 and the Replicas list shows 3 broker ids as well. However, the ISR list only shows 2 values when it should show 3!

While we noticed the issue by looking at the details of the command output, we could have also used the ``--under-replicated-partitions`` flag to see any issues quickly!

```shell
bin/kafka-topics.sh --bootstrap-server localhost:9092 \
--describe --under-replicated-partitions
Topic: kinaction_replica_test Partition: 0 Leader: 0 Replicas: 1,0,2 Isr: 0,2
```

We can run this command to display issues across topics and to quickly find issues on our cluster.

## 6.12 Summary

* Brokers are the centerpiece of Kafka and provide the logic with which external clients interface with our applications. More than one broker makes up a cluster to help provide not only scale but also reliability.
* ZooKeeper (or the Kafka Raft Metadata mode) is leveraged in order to provide agreement in a distributed cluster. One example is to elect a new controller between multiple available brokers.
* Configuration can be set at the broker level to help manage your cluster. This can help set reasonable defaults that your clients can override for specific options.
* Replicas allow for a number of copies of data to span across a cluster. This helps in the event a broker fails and can not be reached. In-Sync Replicas (ISRs) are those that are current and up-to-date with the leader’s data and can take over leadership for a partition without data loss.
* JMX can be used to help produce graphs to visually monitor a cluster or alert on potential issues.
* Mirror Maker (now at version 2.0) is an option to provide a backup strategy across different clusters for those looking to make sure their data is highly available.

# Chapter 7. Topics and partitions

## 7.1 Topics

To quickly refresh our memory, it is important to know that a topic is more of a logical concept rather than a physical structure. It does not usually exist on only one broker. Most applications consuming Kafka data will view that data as being in a single topic: no other details are needed for them to subscribe. However, behind the topic name are one or more partitions which actually hold the data. The logs that are written to the broker filesystems that make up a topic are the result of Kafka writing the data in the cluster.

Let’s say that our company is selling reservations for a training class using a web-based application that sends events of user actions into our Kafka cluster. Our overall application process could generate a number of events. For example, there would be an event for the initial search on location, one for the specific training being selected by the customer, and a third for payment being received. Should the producing applications send all of this data to a single topic or several topics? Should we decide that each message is a specific type of event and should remain separated in different topics? There are tradeoffs with each approach and some things to consider that will help us determine the best approach to take in each situation.

We see topic design as a two-step process:
* The first being for the events I have, **do they belong in 1 topic or more than one.** 
* Second, **for each topic, what is the number of partitions I should use?**

While we can set a default number to be used in topic creation, in most cases we should consider the usage of the topic and the data it will hold. We should have a reason to pick a specific number of partitions and Jun Rao has a fantastic article on the Confluent blog about this every subject titled: "How to choose the number of topics/partitions in a Kafka cluster?"

Let’s take a look at a checklist of items to think about in general and in this scenario.

### Data correctness

**First off, data correctness is at the top of most data concerns in real-world designs. In regards to topics, this would involve making sure that events that must be ordered end up in the same partition (and thus the same topic).**

While events can likely be placed in an order based on an event timestamp by your consumers, it will be more trouble (and error-prone) to handle cross-topic event coordination than it is worth.

If we are using keyed messages and need them in order, we will very much care about partitions and any future changes to those partitions. With our three example events above, it might be helpful **to place the events with a message key including the customer id for the actual 'booked' and 'paid' events in three separate topics.** These events are customer-specific and it would be helpful to make sure that payment of a booking occurs for that specific customer. The search events themselves, however, may not be of interest or need to be ordered for a specific customer if our analytics team is looking for the most popular searched cities rather than customer information.

### Volume of messages of interest per consumer

Next we should consider the volume of messages of interest per consumer. For the training system above, let’s look at the number of events when we consider the topic placement. The search events themselves would far outnumber the other events. Let’s say that a training location near a large city gets 50,000 searches a day but only has room for 100 students. Traffic on most days produces 50,000 search events and less than 100 actual 'booked' training events. Would the Billing team have an application that would want to subscribe to a generic event topic in which it used or cared about less than 1% of the total messages? Most of the consumer’s time would be, in effect, filtering out the mass of events to process only a select few.

### How much data you will be trying to process

Another point to consider is how much data we will be processing. Will the number of messages require multiple consumers to be running in order to process within the time constraints required by our applications? If so, we have to be aware of how the number of consumers in a group is limited by the partitions in our topic. It is easier at this point to create more partitions than you think you might require. Having more capacity for consumers to grow, will allow us to increase in volume without having to deal with re-partitioning data.

However, more partitions require more open file handles for example. Also, having more brokers to migrate, in case of a broker failure, could be a headache in the future.

**Another thing to consider when deciding on the number of partitions for a topic, is that reducing that number is not currently supported.** There may be ways to do it, but it is definitely not advised.

### 7.1.1 Topic Creation Options

We need to make sure we dig into the basic parameters: ``bootstrap-server,`` ``topic``, ``partitions``, and ``replication-factor``.

Defaults might be a good catch-all to make sure topics are created with certain levels of failover (like a minimum of 3 replicas) **but our preference is to be intentional.**

Another important decision to make at creation time is if you ever want to delete a topic. Kafka requires us to configure the option ``delete.topic.enable``. If this is set to 'true' we will be able to successfully delete the topic and it will be removed.

As we think of a name to represent our data, there are a couple of rules that might help us avoid random problems later:
* **Avoid trying to use spaces.** 
* **Avoid creating topics with two underscores.** Kafka managed internal topic names starting with two underscores like __schemas.

**Another option that is good to take care of at the broker level is to set ``auto.create.topics.enable`` to false.** Doing this makes sure that we are creating topics on purpose and not by a producer sending a message to a topic name that was mistyped and never actually existed (before a message was attempted).

*Note.*

If you need to alter topic which has messages already, then it's easier just to create a new topic and migrate old consumers/producers to it, rather than trying to change existing.

### 7.1.2 Replication Factors

**For practical purposes, we should plan on having less than or equal to the number of replicas than the total number of brokers.** In fact, attempting to create a topic with the number of replicas being greater than the total number of brokers will result in an error.

## 7.2 Partitions

From a consumer standpoint, each partition is an immutable log of messages. It should only grow and append messages to our data store.

### 7.2.1 Partition Location

One thing that might be helpful is to look at how the data is stored on our brokers. To start, let’s peek at the configuration value for log.dirs. Under that directory, we should be able to see a subfolder that is named with a topic name and a partition number. If we pick one of those folders and look inside, we will see a couple of different files with the extensions:
* ``index``
* ``log``
* ``timeindex``

```shell
/tmp/kafka-logs-0/kinaction_topic-1
|- 0000000000000000000.index
|- 0000000000000000000.log
|- 0000000000000000000.timeindex
\- leader-epoch-checkpoint
```

Sharp-eyed readers might see the files named leader-epoch-checkpoint and maybe even those with a .snapshot extension (not shown). In general, leader-epoch-checkpoint is related to leader failures and replicas.

**The files with the log extension are where our data payload will be stored.**

Kafka is built for speed, it uses the index file to store a mapping between the logical message offset and a physical position inside the index file. Having the index for Kafka to find a message quickly and then directly read from a position in the log file makes it easy to see how Kafka can still serve consumers looking for specific offsets quickly. The timeindex file works similarly, however, using a timestamp rather than an offset value.

One thing that we have not talked about before is that partitions are made up of segments. In essence, this means that on a physical disk, a partition is not one single file, but rather split into segments. Segments are a key part of helping messages be removed based on retention policies.

An active segment will be the file to which new messages are currently being written.

Older segments will be managed by Kafka in various ways that the active segment will not be: this includes being managed for retention based on the size of the messages or time configuration or eligible for topic compaction.

### 7.2.2 Viewing Segments

Let’s try to take a peek at a log file to see the messages we have produced to the topic so far. If we open it in a text editor, we will not see those messages in a human-readable format. Kafka includes a script, in the ``/bin`` directory, called ``kafka-dump-log.sh`` which we can leverage to look at those log segments.

```shell
bin/kafka-dump-log.sh --print-data-log --files /tmp/kafka-logs-0/kinaction_test-1/00000000000000000000.log
```

Assuming the command is successful, we should see a list of messages with offsets as well as other related metadata like compression codecs used.

![chapter-7-figure-7.png](pictures/chapter-7-figure-7.png)

## 7.3 More Topic and Partition Maintenance

### 7.3.1 Removing a Topic

As previously noted, shouldn't delete topics, but rather migrate off. To delete a topic:

```shell
 bin/kafka-topics.sh --bootstrap-server localhost:9092 --delete --topic kinaction_topic
```

### 7.3.2 Reelect Leaders

As a reminder from our partition discussion earlier, each partition has a leader that handles all client-related data needs. The other replicas are only followers that do not talk to clients. When topics are first created, the cluster tries to spread these leaders evenly among the brokers to split some of the leaders' workload evenly.

The 'preferred' leader will not change for the topic as it will always be considered the replica that is first in the ISR list even across broker failures. The script ``kafka-leader-election.sh`` can be used in this scenario to help bring leadership back to replicas.

```shell
 bin/kafka-leader-election.sh --bootstrap-server localhost:9092    \
  --election-type preferred --partition 1 \
  --topic kinaction_test
```

### 7.3.3 EmbeddedKafkaCluster

With all of the options we have changed with partitions, it might be nice to test them as well. What if we could spin up a Kafka cluster without having a real production-ready cluster handy? Kafka Streams provides an integration utility class called ```EmbeddedKafkaCluster``` that serves as middle ground between mock objects and a full-blown cluster. This class provides an in-memory Kafka cluster. While built with Kafka Streams in mind, we can leverage it in our testing of Kafka clients. An example below is setup like the tests found in Kafka Streams in Action by William P. Bejeck Jr. That title and his following book, Event Streaming with Kafka Streams and ksqlDB, show more in-depth testing examples that we recommend checking out including his suggestion of using Testcontainers.

```java
@ClassRule
public static final EmbeddedKafkaCluster embeddedKafkaCluster = new EmbeddedKafkaCluster(BROKER_NUMBER);
private Properties producerConfig;
private Properties consumerConfig;

@Before
public void setUpBeforeClass() throws Exception {
    embeddedKafkaCluster.createTopic(TOPIC, PARTITION_NUMBER, REPLICATION_NUMBER);
    producerConfig = TestUtils.producerConfig(embeddedKafkaCluster.bootstrapServers(),
            AlertKeySerde.class,
            StringSerializer.class);
    consumerConfig = TestUtils.consumerConfig(embeddedKafkaCluster.bootstrapServers(),
            AlertKeySerde.class,
            StringDeserializer.class);
}

@Test
public void testAlertPartitioner() throws InterruptedException {
    AlertProducer alertProducer =  new AlertProducer();
    try {
        alertProducer.sendMessage(producerConfig);
    } catch (Exception ex) {
        fail("Made producer call with EmbeddedKafkaCluster should not throw exception" <linearrow /> + ex.getMessage());
    }
    AlertConsumer alertConsumer = new AlertConsumer();
    ConsumerRecords<Alert, String> records = alertConsumer.getAlertMessages(consumerConfig);
    TopicPartition partition = new TopicPartition(TOPIC, 0);
    List<ConsumerRecord<Alert, String>> results = records.records(partition);
    assertEquals(0, results.get(0).partition());
}
```

Since this cluster is temporary, another key point is to make sure that the producer and consumer clients know how to point to this in-memory cluster. To discover those endpoints, the method ``bootstrapServers()`` can be used to provide the configuration needed to the clients.

### 7.3.4 Kafka Testcontainers

If you find that you are having to create and tear-down your infrastructure, one option that could be used (especially for integration testing) is to use Testcontainers ( www.testcontainers.org/modules/kafka/).

## 7.4 Topic Compaction

To make it clear, topic compaction is different from the retention of existing segment log files when compared to a regular topic. With compaction, the goal is not to expire messages but rather to make sure that the latest value for a key exists and not maintain any previous state. As just referenced, **compaction depends on a key being part of the messages and not being null.** Otherwise, the whole concept fails if there is no 'key' to update a value for in practice. ``cleanup.policy=compact`` is the topic level configuration option we use to create a compacted topic. This is different from the default configuration value that was set to 'delete' before our override.

One of the easiest comparisons for how a compacted topic presents data can be seen in how a database table updates an existing field rather than appending more data. Let’s say that we want to keep a current membership status for an online membership. A user can only be in one state at a time, either a 'Basic' or 'Gold' membership. At first, the user enrolled for the 'Basic' plan but over time ungraded to the 'Gold' plan for more features. While this is still an event that Kafka stores, in our case we only want the most recent membership level for a specific customer (our key).

![chapter-7-figure-8.png](pictures/chapter-7-figure-8.png)

After compaction is done on the example topic, the latest Customer 0 update is all that exists in the topic. A message with offset 2 replaced the old value of 'Basic' (message offset 0) for Customer 0 with 'Gold'. Customer 1 has a current value of 'Basic' since the latest key specific offset of 100 updated the previous offset 1 'Gold' state. Since Customer 2 only had one event, that event carried over to the compacted topic without any changes. **Another interesting thing to consider is that compaction can appear to cause missing offsets.** In effect, this is the end result of messages with the same key replacing an earlier message.

### 7.4.1 Compaction Cleaning

When a topic is marked for compaction, a single log can be seen in a couple of different states: clean or dirty. Clean is the term for the messages that have been through compaction before. Duplicate values for each key should have been reduced to just one value. The dirty messages are those that have not yet been through compaction. Multiple values might still exist for this message for a specific key until all messages are cleaned.

### 7.4.2 Compaction Record Removal

Even though we have the notion of an immutable stream of events, there is a time when updating a value for a customer might include a removal of that value. Following our subscriber level scenario above, what if the customer decides to remove their account? **By sending an event with the customer key and a null message value, Kafka will keep this last message for a certain time period. The message is considered a 'tombstone'.**

## 7.5 Summary

* Topics are a logical concept rather than a physical structure. To understand the behavior of the topic, a consumer of that topic needs to know about the number of partitions, and the replication factors in play as well.
* The replication factor determines how many copies (of each partition) you want to have in case of failures.
* Partitions makeup topics and are the basic unit for parallel processing of data inside a topic. Log-file segments are written in partition directories and are managed by the broker.
* Reelecting leaders and deleting topics are maintenance that need to be completed with thought about how clients could be impacted by these changes.
* Integration testing can be used to test certain parition logic.
* Topic compaction is a way to provide a view of the latest value of a specific record. Remember, topic compaction is different from the expiration process of data.

# Chapter 8. Kafka storage

So far we have thought of our data as moving into and out of Kafka for brief periods of time. Another decision to analyze is where our data should live longer term. When you think of databases like MySQL or MongoDB, you may not always think about if or how that data is expiring. Rather, you know that the data is likely going to exist for the majority of your application’s entire lifetime. In comparison, Kafka storage logically sits somewhere between the long-term storage solutions of a database and the transient storage of a message broker.

Let’s look at a couple of options for storing and moving data in our environment.

## 8.1 How Long to Store Data

**Currently, the default retention for data in Kafka topics is seven days but this can be easily configured by time or data size.** However, can Kafka hold data itself for a period of years? One real-world example is how the New York Times uses Kafka.

The content in their cluster is in a single partition that was less than 100GB at the time of writing. If you recall from our discussion in Chapter 7 about partitions you know that all of this data will exist on a single broker drive (as will any replica copies exist on their own drives) as partitions are not split between brokers. Since storage is considered to be relatively cheap and the capacity of modern hard drives is way beyond hundreds of gigabytes, most companies would not have any size issues with keeping that data around.

How do we configure retention for brokers? The main considerations are the size of the logs and the length of time the data exists.

| Key  | Purpose |
| ------------- | ------------- |
| log.retention.bytes  | The largest size in bytes threshold for deleting  |
| log.retention.ms  | The length in milliseconds a log will be maintained before being deleted  |
| log.retention.minutes  | Length before deletion in minutes. log.retention.ms is used if both are set  |
| log.retention.hours  | Length before deletion in hours. log.retention.ms and log.retention.minutes would be used before this value if either of those were set  |

**How do we disable log retention limits and allow them to stay 'forever'? By setting both log.retention.bytes and log.retention.ms to -1 we can effectively turn off data deletion.**

Another thing to consider is how we may get similar retention for the latest values by using keyed events with a compacted topic. While data can still be removed during compaction cleaning, the most recent keyed messages will always be in the log.

## 8.2 Data Movement

### 8.2.1 Keeping the original event

One thing that I would like to note is my preference for event formats inside of Kafka. While open for debate and your use-case requirements, **my preference is to store messages in the original format in a topic.** Why keep the original message and not format it right away before placing it into a topic? For one, having the original message makes it easier to go back and start over if you messed up your transform logic. Instead of having to try to figure out how to fix our mistake on the altered data, you can always just go back to the original data and start again.

### 8.2.2 Moving away from a batch mindset

Nothing of interest.

## 8.3 Tools

Data movement is a key to many systems, Kafka included. While you can stay inside the open-source Kafka and Confluent offerings like Connect, which was discussed in chapter 3, there are other tools that might fit your infrastructure or are already available in your tool suite.

### 8.3.1 Apache Flume

Flume can provide an easier path for getting data into a cluster and relies more on configuration than custom code.

Information on Flume. Not interested.

### 8.3.2 Debezium

Debezium (debezium.io) describes itself as a distributed platform that helps turn databases into event streams. In other words, updates to our database can be treated as events! If you have a database background (or not!) you may have heard of the term change data capture (CDC). As the name implies, **the data changes can be tracked and used to react to that change.** At the time of writing this chapter, MySQL, MongoDB, and PostgresSQL servers were supported with more systems slated to be added.

### 8.3.3 Secor

Secor (github.com/pinterest/secor) is an interesting project that has been around since 2014 from Pinterest that aims to help persist Kafka log data to a variety of storage options including S3 and Google Cloud Storage.

## 8.4 Bringing data back into Kafka

Say data lived out its normal lifespan in Kafka and was archived in S3. When a new application logic change required that older data be reprocessed, we did not have to create a client to read from both S3 and Kafka. Rather, using a tool like Kafka Connect, we were able to load that data from S3 back into Kafka.

### 8.4.1 Tiered Storage

A newer option from Confluent Platform version 6.0.0 on, is called Tiered Storage.

In this model, local storage is still the broker itself and remote storage is introduced for data that is older (and stored in a remote location) and controlled by time configuration (``confluent.tier.local.hotset.ms``).

## 8.5 Architectures with Kafka

Let’s take a peek at a couple of architectures that could be powered by Kafka (and to be fair other streaming platforms). This will help us get a different perspective on how we might design systems for our customers.

### 8.5.1 Lambda Architecture

Not interested.

### 8.5.2 Kappa Architecture

Not interested.

## 8.6 Multiple Cluster setups

In this section, we are going to talk about scaling by adding clusters rather than by adding brokers alone.

### 8.6.1 Scaling by adding Clusters

Usually, the first things to scale would be the resources inside your existing cluster. The number of brokers is usually the first option and makes a straight-forward path to growth. However, Netflix’s MultiCluster strategy is a captivating take on how to scale Kafka clusters.

Instead of using the broker number as the way to scale the cluster, they found they could scale by adding clusters themselves! They found that **it might make sense for event producers to talk to one Kafka cluster and have those events moved to other Kafka clusters for consumption.** In this setup, consumers would be reading messages from a Kafka cluster that is entirely different from the ingestion cluster.

As per their scenario, the amount of data going into the cluster was not a problem. **However, once they had multiple consumers pulling that data out at the same time, they saw a specific bottleneck for their network bandwidth.** Even with 1 or 10 Gigabit Ethernet (GbE) data center networks, the rate of data and the number of bytes out can add up quickly.

![chapter-8-figure-7.png](pictures/chapter-8-figure-7.png)

Imagine a scenario (we’ll assume generous network data amounts for ease of understanding) where we have producers that are sending about a gigabyte (GB) of data per second to one cluster. If we have six different consumers, we would be sending 6 copies of that 1 GB of data for around 7 GB per second (again let’s not worry about exact numbers). We do not have much more room on a 10 GbE network.

### 8.6.2 Data over Two Regions

Another important item to consider is the distance we have between our application’s consumers and the data in our cluster. Data that is local to our applications will take much less time to load than would data pulled from the other side of the country if we only had a single cluster as our source. Let’s look at one of the most common cluster setups that we have worked with: the Active-Active region setup.

In an Active-Active model each data-center has its own cluster. End-users of the clusters usually connect to the data center cluster that is closest to them unless there is an outage or failure scenario. The clusters have data replicated between them for both clusters to use no matter where the data originated using MirrorMaker or something similar like Confluent Replicator. A service-level agreement (SLA) might also drive us to pursue this option.

## 8.7 Cloud and Container-Based Storage Options

With AWS being a popular cloud deployment option, let’s focus on a specific example even though these points would be similar for persistent disk solutions on other cloud providers.

### 8.7.1 Amazon Elastic Block Store

In regards to AWS, some users might be tempted to use local instance storage only when planning out their cluster. Since Kafka was built to handle broker failures, the cluster more than likely would be able to recover on newly created broker instances as long as the cluster was otherwise healthy. This solution is straightforward for the most part, you start a new broker node and the cluster will work on gathering data for its partitions from the other brokers. This usually cheaper storage option might be worth the impact of the network traffic increase and any time delays for retrieving data vs other options.

However, Elastic Block Store (EBS) could be a safer option for our use-case. In case you are not familiar with the EBS service, it provides persistent block storage that helps data remain even in the case of a compute instance failure. Besides the fact that you can reassign the EBS volume to a new instance, the broker would only have to recover from messages that were missed during downtime vs the entire data partitions having to be replicated to that node. Amazon also offers instances that are optimized for writing Kafka logs: ie that are built for sequential I/O that is used in writing log files.

### 8.7.2 Kubernetes Clusters

Dealing with a containerized environment, we might run into similar challenges as we would in the cloud. If we hit a poorly configured memory limit on our broker, we might find ourselves on an entirely new node without our data unless we have the data persisted correctly! Unless we are in a sandbox environment in which we can lose data, **persistent volume claims will be needed by our brokers to make sure that our data will survive any restarts, failures, or moves.**

Kafka applications will likely use the StatefulSet API in order to maintain the identity of each broker across failures or pod moves. This static identity also helps us claim the same persistent volumes that were used before our pod was down.

## 8.8 Summary

* Data retention should be driven by business needs. Decisions to weigh include the cost of storage and the growth rate of that data over time.
* Size and time are the basic parameters for defining how long data is retained on disk.
* Long-term storage of data outside of Kafka, like in S3, is an option for data that might need to be retained for long periods. Data can be reintroduced by producing the data into a cluster at a later time if needed.
* The ability of Kafka to handle data quickly and also replay data can enable architectures such as the Lambda and Kappa architectures.
* Cloud and containers workloads often involve short-lived broker instances. Data that needs to be persisted requires a plan for making sure newly created or recovered instances can utilize that data across instances.

# Chapter 9. Management: tools and logging

The best way to keep a cluster moving along is to understand the data that is flowing through it, and to monitor that activity at runtime.

## 9.1 Administration clients

So far, we have performed most of our cluster management activities with the command line tools that come with Kafka. However, there are some helpful options we can use to branch out from these provided scripts.

### 9.1.1 Administration in code with AdminClient

One useful tool to look at is the AdminClient. Although the Kafka shell scripts are great to have at hand for quick access or one-off tasks, there are situations, such as automation, where the Java AdminClient really shines. The AdminClient is in the same kafka-clients.jar that we used for the Producer and Consumer clients. It can be pulled into a Maven project (see the pom.xml from Chapter 2) or it can be found in the ``share/`` or ``libs/`` directory of the Kafka installation.

Let’s look at how we could execute a command we have completed before, to create a new topic, but this time with the AdminClient.

```shell
bin/kafka-topics.sh --bootstrap-server localhost:9092 \
  --create --topic kinaction_selfserviceTopic --partitions 2 \
  --replication-factor 2
```

Though this command-line example works fine, we don’t want to be called every time someone needs a new topic in our dev environment. Instead, we’ll create a self-service portal that other devlopers can use to create new topics on our development cluster.

In this example, we could add logic to make sure that naming conventions for topics fit a certain pattern if we had such a business requirement. This could be a way to maintain more control over our cluster rather than users working from the command-line tools. To start, we need to create a ``NewTopic`` class.

Once we have this information, we can leverage the ``AdminClient`` object to complete the work. The ``AdminClient`` takes a ``Properties`` object that contains the same properties we’ve used with other clients, like ``bootstrap.servers`` and ``client.id``. Note that the class ``AdminClientConfig`` holds constants for configuration values such as ``BOOTSTRAP_SERVERS_CONFIG`` as a helper for those names. Then we’ll call the method createTopics on the client.

```java
NewTopic requestedTopic = new NewTopic("kinaction_selfserviceTopic", 2,2);
AdminClient client = AdminClient.create(props);
CreateTopicsResult topicResult = client.createTopics(Collections.singleton(requestedTopic));

// topicResult.values().get("kinaction_selfserviceTopic").get();
```

At this time there is no synchronous API, but we can make a synchronous call by using the ``get()`` function.

Because this API is still evolving, the following list of tasks that can be accomplished with the ``AdminClient``:

* alter config 
* create/delete/list ACLS 
* create partitions 
* create/delete/list topics 
* describe/list consumer groups 
* describe the cluster

### 9.1.2 Kafkacat

Kafkacat (github.com/edenhill/kafkacat) is a very handy tool to have on your workstation, especially when connecting remotely to your clusters. At this time, it focuses on being a producer and consumer client that can also give you metadata about your cluster. If you ever wanted to quickly work with a topic and didn’t have the entire Kafka tools downloaded to your current machine, this executable will help you avoid the need to have those shell or bat scripts!

### 9.1.3 Confluent REST Proxy API

This proxy is a separate application that would likely be hosted on its own server for production usage.

![chapter-9-figure-2.png](pictures/chapter-9-figure-2.png)

To test out how to leverage the REST proxy, let’s start it up. For this to work, we will need to already have ZooKeeper and Kafka instances running before we start the proxy:

```shell
bin/kafka-rest-start.sh etc/kafka-rest/kafka-rest.properties
```

Since we’re already familiar with listing topics, let’s look at how that can be done with the REST proxy, using a command like curl to hit an HTTP endpoint.

```shell
curl -X GET -H "Accept: application/vnd.kafka.v2+json" localhost:8082/topics
// Output:
["__confluent.support.metrics","_confluent-metrics","_schemas","kinaction_alert"]
```

## 9.2 Running Kafka as a systemd Service

One decision we need to make, concerning running Kafka, is how to perform starts and restarts of our brokers. Those who are used to managing servers as a Linux-based service with a tool like Puppet (puppet.com/), may be familiar with installing service unit files and would likely leverage that knowledge to create running instances. For those not familiar with systemd, it initializes and maintains components throughout the system. One common way to define Zookeeper and Kafka are as unit files that are used by systemd.

An example service unit file that would start a ZooKeeper service when the server starts. It will also restart after an abnormal exit. In practice, this means something like a kill -9 command against the PID of this process would trigger a restart of the process.

```shell
[Unit]
After=network.target
[Service]
Type=simple
ExecStart=/usr/kafka/bin/zookeeper-server-start.sh
 <linearrow /> /usr/kafka/config/zookeeper.properties #1
ExecStop=/usr/kafka/bin/zookeeper-server-stop.sh      #2
Restart=on-abnormal                                   #3
[Install]
WantedBy=multi-user.target
```

1) ExecStart captures the start command that will be run and should be similar to what we manually ran so far to start ZooKeeper
2) ExecStop captures how to property shut down the ZooKeeper instance
3) The Restart condition would run ExecStart if an error condition caused the process to fail

More on how to use systemd Services.

## 9.3 Logging

The logs addressed in this section are not the events and messages from Kafka servers, but the output of the operation of Kafka itself. And we can not forget about Zookeeper either!

### 9.3.1 Kafka application logs

While we might be used to one log file for an entire application, Kafka has multiple log files that we might be interested in or need for troubleshooting. Due to multiple files, we might have to look at modifying different log4j appenders to maintain the necessary views of our operations.

By default, the server logs will continue being added to the directory as new logs are produced. No logs are removed - and this might be the preferred behavior if these files are needed for auditing or troubleshooting. However, if we do want to control the number and size, **the easiest way is to update the file ``log4j.properties`` before we start the broker server.**

```properties
log4j.appender.kafkaAppender.MaxFileSize=500KB              #1
log4j.appender.kafkaAppender.MaxBackupIndex=10              #2
```

1) This settings helps define the size of the file in order to determine when to create a new log file.
2) This value helps set the number of older files to keep. This might be helpful if you want more than just the current log if needed for troubleshooting.

Note that modifying the kafkaAppender changes only how the server.log file is treated. If we want to apply different file sizes and backup file numbers for various Kafka-related files, we can use the appender to log file name table to know which appenders to update.

Note that the appender name in the left column will be the logging key which will affect how the log files on the right will be stored on the brokers

| Appender Name  | Log File Name |
| ------------- | ------------- |
| kafkaAppender  | server.log  |
| stateChangeAppender  | state-change.log  |
| requestAppender  | kafka-request.log  |
| cleanerAppender  | log-cleaner.log  |
| controllerAppender  | controller.log  |
| authorizerAppender  | kafka-authorizer.log  |

Changes to log4j.properties require the broker to be restarted, so it is best to determine our logging requirements before starting our brokers for the first time if possible.

As we look at managing our logs, it is important to consider where the logs are being sent. **Feeding error logs from Kafka into the same Kafka instance is something to avoid.**

### 9.3.2 Zookeeper logs

Not interested.

## 9.4 Firewall

Depending on network configuration, we might need to serve clients that exist inside the network or those out of the network where the Kafka brokers are set up.

Kafka brokers can listen on multiple ports. For example, the default for a plain text port is 9092. An SSL port can also be set up on that same host at 9093. Both of these ports might need to be open depending on how clients are connecting to our brokers.

In addition, Zookeeper ports include 2181 for client connections. Port 2888 is used by follower Zookeeper nodes to connect to the leader Zookeeper node and port 3888 is also used between Zookeeper nodes to communicate.

### 9.4.1 Advertised listeners

One error when connecting that often appears like a firewall issue is when using the listeners and advertised.listeners properties. Clients will have to use the correct hostname to connect if given - so it will need to be reachable however the rules are setup.

Let’s imagine we are connecting to a broker and can get a connection when the client starts up, but not when it attempts to consume messages? How could we have this behavior that appears inconsistent? Remember that whenever a client starts up, it connects to any broker to get metadata about which broker to connect to. **The first initial connection from the client uses the information that is located in the Kafka ``listeners`` configuration. What it gives back to the client to connect to next is the data in Kafka ``advertised.listeners``. This makes it so the client might be connecting to a different host to do its work.**

``inter.broker.listener.name`` is also an important setting to look at that will determine how the brokers connect across the cluster to each other. If the brokers cannot reach each other, replicas will fail and the cluster will not be in a good state, to say the least!

**For an excellent explanation of advertised listeners check out the article by Robin Moffatt titled "Kafka Listeners – Explained".**

## 9.5 Metrics

### 9.5.1 JMX Console

It is possible to use a GUI to explore the exposed metrics and get an idea of what is available. VisualVM (visualvm.github.io/), is one example.

**When installing VisualVM, be sure to go through the additional step of installing the MBeans browser. We can do this by going to the menu bar Tools → Plugins.**

As noted in Chapter 6, we must have a JMX_PORT defined for each broker we want to connect to. **This can be done with an environment variable in the terminal like so: ``export JMX_PORT=1099``**. It is interesting to note that if you have ZooKeeper running on the same server as your broker, its configuration might rely on this same property. **Make sure that you scope it correctly to be separate for each broker as well as each ZooKeeper node.**

KAFKA_JMX_OPTS is also another option to look at for connecting remotely to Kafka brokers. Make sure to note the correct port and hostname. The following example uses port 1099 and the localhost as the hostname.

```shell
KAFKA_JMX_OPTS="-Djava.rmi.server.hostname=127.0.0.1        #1
  -Dcom.sun.management.jmxremote.local.only=false           #2
  -Dcom.sun.management.jmxremote.rmi.port=1099              #3
  -Dcom.sun.management.jmxremote.authenticate=false         #4
  -Dcom.sun.management.jmxremote.ssl=false"                 #4
```

1) Setting the hostname for the RMI server to be localhost
2) Allowing remote connections
3) The port we are exposing for JMX
4) Turning off authenticate and SSL checks

### 9.5.2 Broker alerts

Because Kafka brokers are built as part of a cluster, some problems might be hard to spot without keeping an eye on the specific data points. Let’s look at some metrics that might mean action is needed to help restore a cluster.

Confluent Platform suggestions alerting on the following top three values to start with. I also included ``OfflinePartitionsCount`` as that is a metric I find useful and recommend.

``UnderMinIsrPartitionCount`` is located at ``kafka.server:type=ReplicaManager,name=UnderMinIsrPartitionCount``. **Any value greater than zero is a cause of concern.** This total is the number of partitions less than the number required for the minimum ISR.

``UnderReplicatedPartitions`` is located at ``kafka.server:type=ReplicaManager,name=UnderReplicatedPartitions``. **This is again the number of partitions and this metric being greater than 0 is an issue.** We are not getting the number of replica copies that we expect and we can also handle fewer failure scenarios.

``UnderMinIsr`` is located at ``kafka.cluster:type=Partition,topic={topic},name=UnderMinIsr,partition={partition}``. This is again related to the amount of ISRs being less than the minimum number. **For example, if we have three replicas and we need acknowledgment from all before our produce request is satisfied, we will not have any messages written to the topic partition.**

``OfflinePartitionsCount`` is located at ``kafka.controller:type=KafkaController,name=OfflinePartitionsCount``. Because clients read and write to a leader partition on a broker - there always needs to be a leader for clients to be successful. If the number of offline partitions is greater than 0, the ability to read and write to that partition is compromised.

### 9.5.3 Client alerts

``record-error-rate`` is located at ``kafka.producer:type=producer-metrics,client-id={YOUR_ID_HERE},name=record-error-rate``. The ideal value of this attribute is zero. This is an average for the number of messages that failed per second for producer clients

``records-lag-max`` is located at ``kafka.consumer:type=consumer-fetch-manager-metrics,client-id={YOUR_ID_HERE},name=records-lag-max``. This value is the maximum number of records for a given partition in which the consumer is behind the producer. If we notice an increasing value over time, we might consider alerting on this.

### 9.5.4 Burrow

Although we can usually rely on Kafka being fast for event delivery, there can be times when a specific consumer client gets behind in pulling current messages. Even if a client does not stop or fail on its own, sometimes it might just have more messages to process than its normal workload. **Subtracting the highest offset for a partition and the offset that the consumer client is currently processing is called consumer lag.**

Burrow (github.com/linkedin/Burrow) is a tool that provides consumer lag checking and seems to have been developed with a thoughtful mindset toward its subject. I was introduced to this tool in the hallways of a Kafka Summmit conference and was excited right away to learn more. I say thoughtful because Burrow’s code shows the marks of experienced operators that have worked with Kafka enough to be able to automate more meaning behind this metric. One such issue that is addressed is that the lag metric itself would be driven by and impacted by the consumer client that was doing its own reporting.

## 9.6 Tracing with Interceptors

What if we wanted to trace a single message through the system?

Let’s talk about a simple but straightforward model that might work for our requirements. Let’s say that we have a producer in which each event has a unique ID. Because each message is important, we do not want to miss any of these events. With one client, the business logic runs as normal and consumes the messages from the topic. It makes sense to log to a database or flat file the ID of the event that was processed. A separate consumer, let’s call it an auditing consumer in this instance, would fetch data from the same topic and make sure that there were no IDs missing from the processed entries of the first application. Though this process can work well, it does require adding logic to our application, and so might not be the best choice.

**In practice, the interceptor that we define is a way to add logic to the producer, consumer, or both by hooking into the normal flow of our clients, intercepting the record, and adding our custom data before it moves along its normal path.**

### 9.6.1 Producer interceptor

The order that we list the classes is important as that is the order in which logic will run. The first interceptor gets the record from the producer client. If the interceptor modified that record, note that it might not be the exact same as the first interceptor received.

Let’s start with looking at the Java interface ``ProducerInterceptor``.

Implementing the interface ``ProducerInterceptor`` allows us to hook into the producer’s interceptor life-cycle. The logic in the ``onSend`` method will be called by the send from the normal producer client we have used so far. For our example, we are going to add a header called 'traceId'.

```java
class AlertProducerMetricsInterceptor implements ProducerInterceptor<Alert, String> {

	final static Logger log = LoggerFactory.getLogger(AlertProducerMetricsInterceptor.class);

	public ProducerRecord<Alert, String> onSend(ProducerRecord<Alert, String> record) {
		Headers headers = record.headers();
		String traceId = UUID.randomUUID().toString();
		headers.add("traceId", traceId.getBytes());
		log.info("Created traceId: {}", traceId);
		return record;
	}
	
	//onAcknowledgement will be called when a record is acknowledged or an error occurs
	public void onAcknowledgement(RecordMetadata metadata, Exception exception) {
		if (exception != null) {
			log.info("producer send exception " + exception.getMessage());
		} else {
			log.info("ack'ed topic = {}, partition = {}, offset = {}",
					metadata.topic(), metadata.partition(), metadata.offset());
		} }
	// rest of the code omitted
}
```

We also have to modify our existing AlertProducer class to register the new interceptor. The property name interceptor.classes will need to be added to the producer configuration.

```java
Properties props = new Properties();
...
props.put("interceptor.classes","org.kafkainaction.producer.AlertProducerMetricsInterceptor");
Producer<Alert, String> producer = new KafkaProducer<>(props);
```

### 9.6.2 Consumer Interceptor

```java
public class AlertConsumerMetricsInterceptor implements ConsumerInterceptor<Alert, String> {
	
	public ConsumerRecords<Alert, String> onConsume(ConsumerRecords<Alert, String> records) {
		if (records.isEmpty()) {
			return records;
		} else {
			for (ConsumerRecord<Alert, String> record : records) {
				Headers headers = record.headers();
				for (Header header : headers) {
					if ("traceId".equals(header.key())) {
						log.info("TraceId is: " + new String(header.value()));
					} 
				}
			} 
		}
		return records;
	}
}
```

```java
Properties props = new Properties();
...
props.put("group.id", "alertinterceptor");
props.put("interceptor.classes", "org.kafkainaction.consumer.AlertConsumerMetricsInterceptor");
```

### 9.6.3 Overriding Clients

If we control the source code for clients that other developers will use, we can also subclass an existing client or create our own that implements the Kafka Producer/Consumer interfaces. The Brave (github.com/openzipkin/brave) project has an example of one such client that adds tracing data at the time of writing. For those not familiar with Brave, it is a library meant to help add instrumentation for distributed tracing. It has the ability to send this data to something like a Zipkin (zipkin.io/) server, which can handle the collection and search of this data. Take a peek at the TracingConsumer class (``github.com/openzipkin/brave/blob/master/instrumentation/kafka-clients/src/main/java/brave/kafka/clients/TracingConsumer.java``) for a real-world example of tracing with Kafka.

Developers wanting to consume messages with the custom logic will use an instance of CustomConsumer which includes a reference to a regular consumer client named kafkaConsumer in this example. The custom logic functionality will be added to provide our needed behavior while still interacting with the traditional client. This code is an example of yet another way to provide tracing for Kafka messages.

```java
final class CustomConsumer<K, V> implements Consumer<K, V> {
...
  final Consumer<K, V> kafkaConsumer;
  @Override
  public ConsumerRecords<K, V> poll(final Duration timeout) {
    //Custom logic here
    return kafkaConsumer.poll(timeout);  // Normal Kafka consumer used as normal
  }
... }
```

## 9.7 General monitoring tools

### 9.7.1 CMAK / Kafka Manager

Cluster Manager for Apache Kafka (CMAK) (``github.com/yahoo/CMAK``), once known as Kafka Manager, is an interesting project that focuses on managing Kafka as well as being a UI for various administrative activities. One key feature is the ability to manage multiple clusters. When we look at the commands that we have run so far, we have been looking at one cluster. These commands can be harder to scale the more clusters we are running at once.

### 9.7.2 Cruise Control

Some of the most interesting features are how Cruise Control watches our cluster and can generate suggestions on rebalances based on workloads.

### 9.7.3 Confluent Control Center

Confluent Control Center (docs.confluent.io/current/control-center/index.html) is another web-based tool that can help us monitor and manage our clusters.

### 9.7.4 General monitoring needs

A couple of core operating system monitoring needs appear in order to provide a solid foundation for Kafka to run on.

File handles are something to be aware of. This number will include open file handles that Kafka has on our system. There will be a filehandle for each log file on the broker. In other words, the number can add up quickly. When installing Kafka, it is wise to adjust the file descriptors limits. **While entering a large number more than 100,000 is not unheard of, we still might want an alert when that number is almost gone.**

We will want to make sure that we do not have unnecessary swapping of RAM. Kafka likes to be fast! Swapping can affect performance of the brokers, and the less that occurs, the better.

## 9.8 Summary

* Besides the shell scripts that are packaged with Kafka, an administration client also exists to provide API access to important tasks such as creating a topic.
* Tools such as Kafkacat and the Confluent REST Proxy API also allow ways for developers to interact with the cluster.
* While Kafka leverages a log for client data at its core, there are still various logs specific to the operation of the broker that should be maintained. Managing these logs (and ZooKeeper logs) should be addressed to provide details for troubleshooting or auditing when needed.
* Understanding advertised listeners can help determine behavior that at first appears inconsistent for client connections. Listeners alone might not determine how clients connect.
* Kafka uses JMX for metrics. You can see metrics from clients (producers and consumers) as well as from the brokers.
* Producer and consumer interceptors can be used to implement crosscutting concerns. One such example would be adding tracing ids for message delivery monitoring.

# Chapter 10. Protecting Kafka

## 10.1 Security Basics

### 10.1.1 Encryption with SSL

So far all of our brokers have supported plain text. In effect, there has been no authentication or encryption over the network. Knowing these facts, it might make more sense when reviewing one of the broker server configuration values. If you look at the current ``server.properties`` file, you will find an entry like the following: ``listeners=PLAINTEXT:localhost//:9092``. That listener is in effect providing a mapping of a protocol to a specific port on the broker. Since the brokers support multiple ports, this will allow us to keep the PLAINTEXT port up and running as we test adding SSL, or other protocols, on a new port.

Setting up SSL between the brokers in our cluster and our clients is one place to start. No extra servers or directories are needed. No client coding changes should be required as the changes will be configuration driven.

### 10.1.2 SSL Between Brokers and Clients

Let’s walk through the process and see what we are going to need to accomplish in order to get our cluster updated with this SSL/TLS:
* Create a key per broker
* Create a certificate per broker
* Use a certificate authority (CA) to sign each broker’s certificate
* Add the CA to the client truststore
* Sign each cluster certificate and import that back into its keystore with the CA cert
* Make sure that clients and the broker have their properties updated to use their new truststores

The term broker0 is included in some file names to suggest that this is only for one specific broker - not one that is meant for every broker.

```shell
keytool -genkey -noprompt \
  -alias localhost \
  -dname "CN=kia.manning.com, OU=TEST, O=MASQUERADE, L=Portland, S=Or, C=US" \
  -keystore kafka.broker0.keystore.jks \
  -keyalg RSA \
  -storepass masquerade \
  -keypass masquerade \
  -validity 999
```

Since we have a key that in a way identifies our broker, we need something to signal that we don’t just have any certificate issued by a random user. One way to verify our certificates is by signing them with a CA.

In our examples, we are going to be our own CA to avoid any need of verifying our identity to a third-party. Our next step is to create our own CA.

```shell
openssl req -new -x509 -keyout cakey.crt -out ca.crt \
  -days 999 -subj '/CN=localhost/OU=TEST/O=MASQUERADE/L=Portland/S=Or/C=US' \
  -passin pass:masquerade -passout pass:masquerade
```

Now that we have generated our CA, we will use it to sign our certificates for our brokers that we have already made.

```shell
#Adding our CA to the trusted broker0 truststore
keytool -keystore kafka.broker0.truststore.jks \
-alias CA -import -file ca.crt  \
-keypass masquerade -storepass masquerade

#We extract the key from the broker0 keystore we created earlier
keytool -keystore kafka.broker0.keystore.jks \
-alias localhost -certreq -file cert-file \
-storepass masquerade -noprompt

#We use the CA to sign this broker0 key
openssl x509 -req -CA ca.crt -CAkey cakey.crt \
-in cert-file -out signed.crt -days 999 \
-CAcreateserial -passin pass:masquerade

#We import the CA certificate into our broker0 keystore
keytool -keystore kafka.broker0.keystore.jks \
-alias CA -import -file ca.crt \
-storepass masquerade -noprompt

#We import the signed certificate into the broker broker0 keystore as well
keytool -keystore kafka.broker0.keystore.jks \
-alias localhost -import -file signed.crt \
-storepass masquerade -noprompt

#Adding our CA to the client truststore
keytool -keystore kafka.client.truststore.jks \
-alias CA -import -file ca.crt  \
-keypass masquerade -storepass masquerade
```

As part of our changes, we need to update the ``server.properties`` configuration file on each broker as well (broker0 shown only):

```properties
listeners=PLAINTEXT://localhost:9092,SSL://localhost:9093
ssl.truststore.location=/var/ssl/private/kafka.broker0.truststore.jks
ssl.truststore.password=masquerade
ssl.keystore.location=/var/ssl/private/kafka.broker0.keystore.jks
ssl.keystore.password=masquerade
ssl.key.password=masquerade
```

Changes are also needed for our clients:

```properties
security.protocol=SSL
ssl.truststore.location=/var/private/ssl/client.truststore.jks
ssl.truststore.password=masquerade
```

One of the simplest ways to check our SSL setup is to quickly use the console producers and consumer clients to connect to a topic on our SSL port 9093:

```shell
bin/kafka-console-producer.sh --bootstrap-server localhost:9093 \
  --topic kinaction_test_ssl \
  --producer.config custom-ssl.properties
bin/kafka-console-consumer.sh --bootstrap-server localhost:9093 \
  --topic kinaction_test_ssl \
  --consumer.config custom-ssl.properties
```

### 10.1.3 SSL Between Brokers

Another detail to research is since we also have our brokers talking to each other, we might want to determine if we need to use SSL for those interactions. ``security.inter.broker.protocol = SSL`` should be used in the server properties if we do not want to continue using plaintext for communication between brokers as well as a potential port change.

## 10.2 Simple Authentication and Security Layer (SASL)

### 10.2.1 Kerberos

Read when required.

### 10.2.2 HTTP Basic Auth

Read when required.

## 10.3 Authorization in Kafka

### 10.3.1 Access Control Lists

As we quickly reviewed, authorization is the process that controls what a user can do. One way to do that is with Access Control Lists (ACLs).

Kafka designed their authorizer to be pluggable to allow users to make their own logic if desired. Kafka does have a class SimpleAclAuthorizer that we will use in our example. With this authorizer, ACLs are stored in ZooKeeper. Brokers asynchronously get this information and cache this metadata in order to make subsequent processing quicker.

```properties
authorizer.class.name=kafka.security.auth.SimpleAclAuthorizer
super.users=User:Franz
```

The Kafka Authorizer CLI tool is included with the other Kafka scripts in our install. ``kafka-acls.sh`` allows us to add, delete, or list current ACLs. An important item to note is that once we configure an Authorizer, ACLs will need to be set or only those considered super users will have access.

Let’s figure out how to make only Team Clueful have access to produce and consume from their own topic: clueful_secrets.

For brevity, we will have 2 users in our example team, Franz and Hemingway.

```shell
bin/kafka-acls.sh --authorizer-properties \
  --bootstrap-server localhost:9092 --add \
  --allow-principal User:Franz --allow-principal User:Hemingway \
  --operation Read --operation Write --topic clueful_secrets
```

### 10.3.2 Role-based access control

Role-based access control (RBAC) is an option that the Confluent Platform supports. RBAC is a way to control access based on roles. Users are then assigned to their role according to their needs such as a job duty. Instead of granting each and every user permissions, with RBAC, you manage the privileges assigned to predefined roles.

RBAC can be used as a layer on top of ACLs that we discussed earlier. One of the biggest reasons to consider both RBAC and ACLS is if you need more granular access control in certain areas. While a user might be able to access a resource like a topic due to RBAC, you could still use an ACL to deny access to specific members of that group.

## 10.4 ZooKeeper

We will need to set the value zookeeper.set.acl to true per broker:

```properties
zookeeper.set.acl=true
```

Without this setting (or set to false), ACLs would not be created. Note that these ACLs are ZooKeeper specific, not the ACLs we talked about earlier which applied to Kafka resources only.

### 10.4.1 Kerberos Setup

Read when required.

## 10.5 Quotas

Let’s say that users of our web application start to notice that they don’t have any issues with trying over and over to request data. While this is often a good thing for end-users who want to be able to use a service as much as they want without their progress being limited, the cluster might need some protection from users who might use that to their advantage. In our instance, since we made it so the data was locked down to members of our team only, some users have thought of a way to try to prevent others from using the system. In effect, they are trying to use a distributed denial-of-service (DDoS) attack against our system.

We can use quotas to prevent this behavior. One detail to know is that quotas are defined on a per-broker basis. **The cluster does not look across each broker to calculate a total, so a per-broker definition is needed.**

Also, by default, quotas are unlimited.

To set our own custom quotas, we need to know how to identify the "who" to limit and the limit we want to set. Whether or not we have security impacts what options we have for defining who we are limiting. Without security, we are able to use the ``client.id`` property. With security enabled, we can also add the user, and thus, any user and ``client.id`` combinations as well.

There are a couple of types of quotas that we can look at defining for our clients: 
* network bandwidth;
* request rate quotas.

### 10.5.1 Network Bandwidth Quota

**Network bandwidth is measured by the number of bytes per second.**

Each user in our competition uses a client id that is specific to their team for any producer or consumer requests from their clients.

We are going to limit the clients using the client id ``clueful`` by setting a ``producer_byte_rate`` and a ``consumer_byte_rate``.

```shell
bin/kafka-configs.sh  --bootstrap-server localhost:9092 --alter \
  --add-config 'producer_byte_rate=1048576,consumer_byte_rate=5242880' \
  --entity-type clients --entity-name clueful
```

The ``add-config`` parameter is used to set both the producer and consumer rate. The ``entity-name`` applies the rule to our specific clients clueful.

As is often the case, we might need to list our current quotas as well as delete them if they are no longer needed. All of these commands can be completed by sending different arguments to the ``kafka-configs.sh`` script.

```shell
bin/kafka-configs.sh  --bootstrap-server localhost:9092 \
  --describe \
  --entity-type clients --entity-name clueful
bin/kafka-configs.sh  --bootstrap-server localhost:9092 --alter \
  --delete-config 'producer_byte_rate,consumer_byte_rate' \
  --entity-type clients --entity-name clueful
```

As we start to add quotas we might end up with more than one quota applied to a client. We will need to be aware of the precedence in which various quotas are applied. While it might seem like the most restrictive (lowest bytes allowed) of the quotas would be applied, that is not always the case. The following is the order in which quotas are applied with the highest precedence listed at the top
* User and client.id provided quotas
* User quotas
* Client.id quotas

**For example, if a user named Franz had a user quota limit of 10 MB and a client.id limit of 1 MB, the consumer he used would be allowed 10 MB per second due to the User-defined quota having higher precedence.**

### 10.5.2 Request Rate Quotas

The other quota to examine is on request rate. Why the need for a second quota? While a DDoS attack is often thought of as a network issue, clients making lots of connections could still overwhelm the broker by making CPU-intensive requests (anything related to SSL or message compressed/decompression are good examples). Consumer clients that poll continuously with a setting of ``fetch.max.wait.ms=0`` are also a concern that can be addressed with this quota.

The simplest way to think about this quota is that it represents the total percentage of CPU that a group of related clients is allowed to use.

To set this quota, we use the same entity types and ``add-config`` options as we did with our other quotas. The biggest difference is setting the configuration for ``request_percentage``. A formula that we can use is the number of I/O threads (num.io.threads) + the number of network threads (num.network.threads) * 100% and was listed in the KIP-124 - Request rate quotas wiki.

## 10.6 Data at Rest

Another thing to consider is whether you need to encrypt the data that Kafka writes to disk. By default, Kafka does not encrypt the events it adds to the log.

## 10.7 Summary

* PLAINTEXT, while fine for prototypes, needs to be evaluated before production usage. SSL can help protect your data between clients and brokers and even between brokers.
* Kerberos can be used to provide a principal identity and might allow you to leverage existing Kerberos environments that exist in an infrastructure already.
* Access control lists (ACLs) help define which users have specific operations granted. Role-based access control (RBAC) is also an option that the Confluent Platform supports.
* Quotas can be used with network bandwidth and request rate limits to protect the available resources of a cluster. These quotas can be changed and fine-tuned to allow for normal workloads and peak demand over time.

# Chapter 11. Schema registry

