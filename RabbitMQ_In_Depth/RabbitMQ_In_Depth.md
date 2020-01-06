# Part 1. RabbitMQ and application architecture

## Foundational RabbitMQ

### RabbitMQ’s features and benefits

* Open source
* Platform and vendor neutral
* Lightweight
* Client libraries for most modern languages
* Flexibility in controlling messaging trade-offs
* Plugins for higher-latency environments - Because not all network topologies and architectures are the same, RabbitMQ provides for messaging in low-latency environments and plugins for higher-latency environments, such as the internet. This allows for RabbitMQ to be clustered on the same local network and share federated messages across multiple data centers.
* Layers of security

# How to speak Rabbit: the AMQ Protocol

## AMQP as an RPC transport

As an AMQP broker, RabbitMQ speaks a strict dialect for communication, utilizing a **remote procedure call (RPC)** pattern in nearly every aspect of communication with the core product.

### Kicking off the conversation

When you’re communicating with someone new in a foreign country, it’s inevitable that one of you will kick off the conversation with a greeting, something that lets you and the other person know if you’re both capable of speaking the same language. When speaking AMQP, this greeting is the protocol header, and it’s sent by the client to the server. This greeting shouldn’t be considered a request, however, as unlike the rest of the conversation that will take place, it’s not a command. RabbitMQ starts the command/response sequence by replying to the greeting with a Connection.Start command, and the client responds to the RPC request with Connection.StartOk response frame (figure 2.1).
```
Client       Server
  | ----1---->  |
  |             |
  | <----2----  |
  |             |
  | ----3---->  |
  |             |

1. Protocol header;
2. Connection.Start;
3. Connection.StartOk;

```
### Tuning in to the right channel

Similar in concept to channels on a two-way radio, the AMQP specification defines channels for communicating with RabbitMQ. Two-way radios transmit information to each other using the airwaves as the connection between them. In AMQP, channels use the negotiated AMQP connection as the conduit for transmitting information to each other, and like channels on a two-way radio, they isolate their transmissions from other conversations that are happening. **A single AMQP connection can have multiple channels, allowing multiple conversations between a client and server to take place. In technical terms, this is called multiplexing.**

## AMQP’s RPC frame structure

Very similar in concept to object-oriented programming in languages such as C++, Java, and Python, AMQP uses classes and methods, referred to as AMQP commands, to create a common language between clients and servers. The classes in AMQP define a scope of functionality, and each class contains methods that perform different tasks.
![AMQP_Class_Function](AMQP_Class_Function.PNG).

### AMQP frame components

When commands are sent to and from RabbitMQ, all of the arguments required to execute them are encapsulated in **data structures called frames** that encode the data for transmission.

As figure 2.3 illustrates, a low-level AMQP frame is composed of five distinct components:
* Frame type
* Channel number
* Frame size in bytes
* Frame payload
* End-byte marker (ASCII value 206)

![AMQP_Frame_Anatomy](AMQP_Frame_Anatomy.PNG).

A low-level AMQP frame starts off with three fields, referred to as a **frame header** when combined:
* First field is a single byte indicating the frame type
* Second field specifies the channel the frame is for
* Third field carries the byte size of the frame payload

### Types of frames

The AMQP specification defines five types of frames:
* **Protocol header frame** is only used once, when connecting to RabbitMQ.
* **Method frame** carries with it the RPC request or response that’s being sent to or received from RabbitMQ.
* **Content header frame** contains the size and properties for a message.
* **Body frames** contain the content of messages.
* **Heartbeat frame** is sent to and from RabbitMQ as a check to ensure that both sides of the connection are available and working properly.

***NOTE***. Oftentimes developers in single-threaded or asynchronous development environments will want to increase the timeout to some large value. To turn off - set heartbeat interval to 0. 

### Marshaling messages into frames

**When publishing a message to RabbitMQ, the method, header, and body frames are used.** The **first frame sent is the method frame carrying the command** and the parameters required to execute it, such as the exchange and routing key. **Following the method frame are the content frames: a content header and body.** The content header frame contains the message properties along with the body size. AMQP has a maximum frame size, and if the body of your message exceeds that size, the content will be split into multiple body frames.

As figure 2.4 illustrates, when sending a message to RabbitMQ, a Basic.Publish command is sent in the method frame, and that’s followed by a content header frame with the message’s properties, such as the message’s content type and the time when the message was sent. These properties are encapsulated in a data structure defined in the AMQP specification as Basic.Properties . Finally, the content of the message is marshaled into the appropriate number of body frames.

***NOTE***. Although the default frame size is 131 KB, client libraries can negotiate a larger or smaller maximum frame size during the connection process, up to a 32-bit value for the number of bytes in a frame.

![Single_Message_Composed_Of_Frames](Single_Message_Composed_Of_Frames.png)

The content in the method frame and content header frame is binary packed data and is not human-readable. The message content carried inside the body frame isn’t packed or encoded and may be anything from plain text to binary image data.

### The anatomy of a method frame

Method frames carry with them the class and method your RPC request is going to make as well as the arguments that are being passed along for processing.

![Anatomy_Of_Method_Frame](Anatomy_Of_Method_Frame.png)

***NOTE*** In fact, the AMQP specification goes as far as to say that success, as a general rule, is silent, whereas errors should be as noisy and intrusive as possible. But if you’re using the mandatory flag when publishing your messages, your application should be listening for a Basic.Return command sent from RabbitMQ. If RabbitMQ isn’t able to meet the requirements set by the mandatory flag, it will send a Basic.Return command to your client on the same channel.

### The content header frame

The header frame also carries attributes about your message that describe the message to both the RabbitMQ server and to any application that may receive it. These attributes, as values in a **Basic.Properties** table, may contain data that describes the **content of your message or they may be completely blank.** Most client libraries will prepopulate a minimal set of fields, such as the content type and the delivery mode.

![Anatomy_Of_Method_Frame](Anatomy_Of_Method_Frame.png)

### The body frame

The body frame for a message is agnostic to the type of data being transferred, and it may contain either binary or text data.

![Anatomy_Of_Body_Frame](Anatomy_Of_Body_Frame.png)

## Putting the protocol to use

There are a few configuration-related steps you must take care of before you can publish messages into a queue. **At a minimum, you must set up both an exchange and a queue, and then bind them together.**

### Declaring an exchange

**Exchanges are created using the Exchange.Declare command**, which has arguments that define the name of the exchange, its type, and other metadata that may be used for message processing. Once the command has been sent and RabbitMQ has created the exchange, an Exchange.DeclareOk method frame is sent in response (figure 2.8). **If, for whatever reason, the command should fail, RabbitMQ will close the channel that the Exchange.Declare command was sent on by sending a *Channel.Close* command.** This response will include a numeric reply code and text value indicating why the Exchange.Declare failed and the channel was closed.

![Declare_Exchange](Declare_Exchange.png)

### Declaring a queue

Once the exchange has been created, it’s time to create a queue by sending a Queue.Declare command to RabbitMQ. Like the Exchange.Declare command, there’s a simple communication sequence that takes place (figure 2.9), and should the Queue.Declare command fail, the channel will be closed.

![Declare_Queue](Declare_Queue.png)

**When declaring a queue, there’s no harm in issuing the same Queue.Declare command more than once.**

### Binding a queue to an exchange

Once the exchange and queue have been created, it’s time to bind them together. Like with Queue.Declare , the command to bind a queue to an exchange, Queue.Bind, can only specify one queue at a time.

![Queue_Bind](Queue_Bind.png)

### Publishing a message to RabbitMQ

As you previously learned, when publishing messages to RabbitMQ, multiple frames encapsulate the message data that’s sent to the server. Before the actual message content ever reaches RabbitMQ, the client application sends a Basic.Publish method frame, a content header frame, and at least one body frame (figure 2.11).

The **Basic.Publish** method frame carries with it the exchange name and routing key for the message. When evaluating this data, RabbitMQ will try to match the exchange name in the Basic.Publish frame against its database of configured exchanges.

![Publishing_A_Message](Publishing_A_Message.png)

***NOTE*** By default, if you’re publishing messages with an exchange that doesn’t exist in RabbitMQ’s configuration, it will silently drop the messages.

When RabbitMQ finds a match to the exchange name in the Basic.Properties method frame, it evaluates the bindings in the exchange, looking to match queues with the routing key.

### Consuming messages from RabbitMQ

To consume messages from a queue in RabbitMQ, a consumer application subscribes to the queue in RabbitMQ by issuing a Basic.Consume command. Like the other synchronous commands, the server will respond with Basic.ConsumeOk to let the client know it’s going to open the floodgates. 

At RabbitMQ’s discretion, the consumer will start receiving messages of **Basic.Deliver methods** and their content header and body frame counterparts.

![Consuming_Messages](Consuming_Messages.png)

If a consumer wants to stop receiving messages, it can issue a **Basic.Cancel** command. It’s worth noting that this command is issued asynchronously while RabbitMQ may still be sending messages, so a consumer can still receive any number of messages RabbitMQ has preallocated for it prior to receiving a Basic.CancelOk response frame.

### Writing a message publisher in Java
**In book it is written in Python**

```
private static final String RABBIT_URI = "amqp://guest:guest@localhost:5672";
private static final String EXCHANGE_NAME = "chapter2-example";
private static final String QUEUE_NAME = "example";
private static final String ROUTING_KEY = "example-routing-key";

public static void main(String[] args) throws NoSuchAlgorithmException, KeyManagementException, URISyntaxException {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setUri(RABBIT_URI);
    try (Connection connection = factory.newConnection();
         Channel channel = connection.createChannel()) {

        channel.exchangeDeclare(EXCHANGE_NAME, "direct");
        channel.queueDeclare(QUEUE_NAME, true, false, false, Map.of());
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, ROUTING_KEY);

        String message = "Hello World!";
        var i = 0;
        while (i < 50) {
            channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY, null, message.getBytes());
            System.out.println(" [x] Sent '" + message + "'");
            i++;
        }

    } catch (TimeoutException | IOException e) {
        e.printStackTrace();
    }
}
```

This creates explicit Exchange named "chapter2-example".

![RabbitMQ_Specific_Exchange](RabbitMQ_Specific_Exchange.PNG)
![RabbitMQ_Specific_Exchange_2](RabbitMQ_Specific_Exchange_2.PNG)

It is possible to use default Exchange.

#### Default Exchange

The default exchange is a direct exchange with no name (empty string) pre-declared by the broker. It has one special property that makes it very useful for simple applications: every queue that is created is automatically bound to it with a routing key which is the same as the queue name.

For example, when you declare a queue with the name of "search-indexing-online", the AMQP 0-9-1 broker will bind it to the default exchange using "search-indexing-online" as the routing key (in this context sometimes referred to as the binding key). Therefore, a message published to the default exchange with the routing key "search-indexing-online" will be routed to the queue "search-indexing-online".

```
private static final String QUEUE_NAME = "example";

public static void main(String[] args) {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    try (Connection connection = factory.newConnection();
         Channel channel = connection.createChannel()) {

        channel.queueDeclare(QUEUE_NAME, true, false, false, Map.of());
        String message = "Hello World!";
        var i = 0;
        while (i < 50) {
            //QUEUE_NAME instead of ROUTING_KEY, because Default Exchange expects routing_key = queue_name
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
            System.out.println(" [x] Sent '" + message + "'");
            i++;
        }

    } catch (TimeoutException | IOException e) {
        e.printStackTrace();
    }
}
```

### Getting messages from RabbitMQ



```
public static final String QUEUE_NAME = "example";

public static void main(String[] args) throws IOException, TimeoutException, NoSuchAlgorithmException, KeyManagementException, URISyntaxException {
    String url = "amqp://guest:guest@localhost:5672";
    ConnectionFactory factory = new ConnectionFactory();
    factory.setUri(url);

    Connection connection = factory.newConnection();
    Channel channel = connection.createChannel();

    DeliverCallback deliverCallback = (consumerTag, message) -> consumeMessage(channel, message);

    boolean autoAck = false;
    channel.basicConsume(QUEUE_NAME, autoAck, deliverCallback, consumerTag -> {});
}

private static void consumeMessage(Channel channel, Delivery message) throws IOException {
    Envelope envelope = message.getEnvelope();
    AMQP.BasicProperties properties = message.getProperties();
    String routingKey = envelope.getRoutingKey();
    String contentType = properties.getContentType();
    long deliveryTag = envelope.getDeliveryTag();

    String s = new String(message.getBody());
    System.out.println(String.format("Body: %s, Routing Key: %s, Content type: %s, Delivery Tag: %d", s, routingKey, contentType, deliveryTag));
    channel.basicAck(deliveryTag, false);
}

```

# Chapter 3. An in-depth tour of message properties (Basic.Properties)

## Using properties properly

The message properties contained in the header frame are a predefined set of values specified by the Basic.Properties data structure (figure 3.2).

![Basic_Properties](Basic_Properties.png)

## Creating an explicit message contract with content-type

Like in the various standardized HTTP specifications, content-type conveys the MIME type of the message body. If your application is sending a JSON-serialized data value, set the content-type property to application/json.

## Reducing message size with gzip and content-encoding

Messages sent over AMQP aren’t compressed by default. This can be problematic with overly verbose markup such as XML, or even with large messages using less markup-heavy formats like JSON or YAML. Your publishers can compress messages prior to publishing them and decompress them upon receipt, similarly to how web pages can be compressed on the server with gzip and the browser can decompress them on the fly prior to rendering.

***NOTE***. Some AMQP clients automatically set the content-encoding value to UTF-8, but this is incorrect behavior. The AMQP specification states that content-encoding is for storing the MIME content encoding.

## Referencing messages with message-id and correlation-id

In the AMQP specification, **message-id** and **correlation-id** are specified **“for application use”** and have no formally defined behavior.

### Message-id

Some message types, such as a login event, aren’t likely to need a unique message ID associated with them, but it’s easy to imagine types of messages that would, such as sales orders or support requests. The message-id property enables the message to carry data in the header that uniquely identifies it as it flows through the various components in a loosely coupled system.

### Correlation-id

Although there’s no formal definition for the correlation-id in the AMQP specification, one use is to indicate that the message is a response to another message by having it carry the message-id of the related message. Another option is to use it to carry a transaction ID or other similar data that the message is referencing.

## Born-on dating: the timestamp property

One of the more useful fields in Basic.Properties is the timestamp property. Timestamp is specified as **“for application use.”**

The timestamp is sent as a Unix epoch or integer-based timestamp indicating the number of seconds since midnight on January 1, 1970. For example, February 2,2002, at midnight would be represented as the integer value 1329696000. **Unfortunately there’s no time zone context for the timestamp, so it’s advisable to use UTC.**

## Automatically expiring messages

The expiration property tells RabbitMQ when it should discard a message if it hasn’t been consumed. In addition, the specification of the expiration property is a bit odd; **it’s specified “for implementation use, no formal behavior,” meaning RabbitMQ can implement its use however it sees fit. One final oddity is that it’s specified as a short string, allowing for up to 255 characters, whereas the other property that represents a unit of time, timestamp, is an integer value.**

**Because of the ambiguity in the specification, the expiration value is likely to have different implications when using different message brokers or even different versions of the same message broker.** To auto-expire messages in RabbitMQ using the expiration property, it must contain a Unix epoch or integer-based timestamp, but stored as a string. **Instead of storing an ISO-8601 formatted timestamp such as "2002-02-20T00:00:00-00" , you must set the string value to the equivalent value of "1329696000".**
When using the expiration property, if a message is published to the server with an expiration timestamp that has already passed, the message will not be routed to any queues, but instead will be discarded.

## Balancing speed with safety using delivery-mode

The delivery-mode property is a byte field that indicates to the message broker that you’d like to persist the message to disk prior to it being delivered to any awaiting consumers. The delivery-
mode property has two possible values: 1 for a non-persisted message and 2 for a persisted message.

## Validating message origin with app-id and user-id

### app-id

The app-id property is defined in the AMQP specification as a “short-string,” allowing for up to 255 UTF-8 characters. If your application has an API-centric design with versioning, you could use the app-id to convey the specific API and version that were used to generate the message. As a method of enforcing a contract between publisher and consumer, examining the app-id prior to processing allows the application to discard the message if it’s from an unknown or unsupported source. 

Another possible use for app-id is in gathering statistical data.

### user-id

RabbitMQ checks every message published with a value in the user-id property against the RabbitMQ user publishing the message, and if the two values don’t match, the message is rejected. For example, if your application is authenticating with RabbitMQ as the user “www”, and the user-id property is set to “linus”, the message will be rejected.

## Getting specific with the message type property

Type property as the “message type name,” saying that it’s for application use and has no formal behavior.

## Using reply-to for dynamic workflows

The reply-to property has no formally defined behavior and is also specified for application use.

## Custom properties using the headers property

The **headers** property is a key/value table that allows for arbitrary, user-defined keys and values. Keys can be ASCII or Unicode strings that have a maximum length of 255 characters. Values can be any valid AMQP value type.

![Headers_Property](Headers_Property.png)

## The priority property

**It’s defined as an integer with possible values of 0 through 9 to be used for message prioritization in queues.** As specified, if a message with a priority of 9 is published, and subsequently a message with a priority of 0 is published, a newly connected consumer would receive the message with the priority of 0 before the message with a priority of 9. Interestingly, RabbitMQ implements the priority field as an unsigned byte, so priorities could be anywhere from 0 to 255, but the priority should be limited to 0 through 9 to maintain interoperability with the specification.

## A property you can’t use: cluster-id/reserved

## Summary

![Properties_Summary_Table](Properties_Summary_Table.png)

# Chapter 4. Performance trade-offs in publishing

## Balancing delivery speed with guaranteed delivery

![Performance_trade-offs](Performance_trade-offs.PNG)

In RabbitMQ, each mechanism designed to create delivery guarantees will come with some impact on performance. Only by performing your own performance benchmarks can you determine the acceptable trade-off of performance versus guaranteed delivery. The following questions can help find the right balance between high performance and message safety:
* How important is it that messages are guaranteed to be enqueued when published?
* Should a message be returned to a publisher if it can’t be routed?
* If a message can’t be routed, should it be sent somewhere else where it can later be reconciled?
* Is it okay if messages are lost when a RabbitMQ server crashes?
* Should RabbitMQ confirm that it has performed all requested routing and persistence tasks to a publisher when it processes a new message?
* Should a publisher be able to batch message deliveries and then receive confirmation from RabbitMQ that all requested routing and persistence tasks have been applied to all of the messages in the batch?
* If you’re batching the publishing of messages that require confirmation of routing and persistence, is there a need for true atomic commits to the destination queues for a message?
* Are there acceptable trade-offs in reliable delivery that your publishers can use to achieve higher performance and message throughput?
* What other aspects of message publishing will impact message throughput and performance?

### What to expect with no guarantees

No guarantees works great for applications like performance statistics gathering, where data is collected every minute. Even if you lose some data, it is not that crucial.

### RabbitMQ won’t accept non-routable messages with mandatory set

**The mandatory flag is an argument that’s passed along with the Basic.Publish** RPC command and tells RabbitMQ that if a message isn’t routable, it should send the message back to the publisher via a Basic.Return. **The mandatory flag can be thought of as turning on fault detection mode; it will only cause RabbitMQ to notify you of failures, not successes.**

To publish a message with the mandatory flag, you simply pass in the argument after passing in the exchange, routing key, message, and properties.

```
private static final String RABBIT_URI = "amqp://guest:guest@localhost:5672";
private static final String EXCHANGE_NAME = "chapter2-example";
private static final String QUEUE_NAME = "example";
private static final String ROUTING_KEY = "example-routing-key";

public static void main(String[] args) throws NoSuchAlgorithmException, KeyManagementException, URISyntaxException {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setUri(RABBIT_URI);
    try (Connection connection = factory.newConnection();
         Channel channel = connection.createChannel()) {

        channel.exchangeDeclare(EXCHANGE_NAME, "direct");
        channel.queueDeclare(QUEUE_NAME, true, false, false, Map.of());
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, ROUTING_KEY);
        channel.addReturnListener(Send::handleReturn);

        String message = "Hello World!";
        var i = 0;
        while (i < 3) {
            //Set true to mandatory field
            channel.basicPublish(EXCHANGE_NAME, "BAD-ROUTING-KEY", true, null, message.getBytes());
            System.out.println(" [x] Sent '" + message + "'");
            i++;
        }

        //To wait for error messages
        TimeUnit.SECONDS.sleep(5);
    } catch (TimeoutException | IOException | InterruptedException e) {
        e.printStackTrace();
    }
}

private static void handleReturn(int replyCode, String replyText, String exchange, String routingKey, AMQP.BasicProperties properties, byte[] body) {
    System.out.println("-".repeat(10) + "Message Failed" + "-".repeat(10));
    System.out.println(replyCode);
    System.out.println(replyText);
    System.out.println(exchange);
    System.out.println(routingKey);
    System.out.println(new String(body));
}

```

This code will generate:
```
----------Message Failed----------
312
NO_ROUTE
chapter2-example
BAD-ROUTING-KEY
Hello World!
```

The Basic.Return call is an asynchronous call from RabbitMQ, and it may happen at any time after the message is published. If the code isn’t set up to listen for this call, it will fall on deaf ears, and collectd will never know that the message wasn’t published correctly.

**NOTE** The **immediate flag** directs a broker to issue a Basic.Return if the message can’t be immediately routed to its destination. **This flag is deprecated as of RabbitMQ 2.9 and will raise an exception and close the channel if used.**

### Publisher Confirms as a lightweight alternative to transactions

The **Publisher Confirms** feature in RabbitMQ is an enhancement to the AMQP specification and is **only supported by client libraries that support RabbitMQ-specific extensions**. **Storing messages on disk** is an important step in preventing message loss, but **doesn’t assure the publisher that a message was delivered.** Prior to publishing any messages, a message publisher must issue a Confirm.Select RPC request to RabbitMQ and wait for a Confirm.SelectOk response to know that delivery confirmations are enabled. At that point, for each message that a publisher sends to RabbitMQ, the server will respond with an acknowledgement response (Basic.Ack) or a negative acknowledgement response (Basic.Nack) (figure 4.4).

![Publisher_Confirms](Publisher_Confirms.PNG)

A Basic.Ack request is sent to a publisher when a message that it has published has been directly consumed by consumer applications on all queues it was routed to, or when the message was enqueued and persisted if requested. Publisher Confirms don’t work in conjunction with transactions and is considered a lightweight and more performant alternative to the AMQP TX process.

### Using alternate exchanges for unroutable messages

Alternate exchanges are another extension to the AMQ model, created by the RabbitMQ team as a way to handle unroutable messages.

It is sometimes desirable to let clients handle messages that an exchange was unable to route (i.e. either because there were no bound queues or no matching bindings). Typical examples of this are:
* detecting when clients accidentally or maliciously publish messages that cannot be routed
* "or else" routing semantics where some messages are handled specially and the rest by a generic handler

#### Configuration Using a Policy

**This is the recommended way of of defining alternate exchanges.**

To specify an AE using policy, add the key 'alternate-exchange' to a policy definition and make sure that the policy matches the exchange(s) that need the AE defined. For example:
```
rabbitmqctl set_policy AE "^my-direct$" '{"alternate-exchange":"my-ae"}'
```

#### Configuration Using Client-provided Arguments

**This way of defining an alternate exchange is discouraged.**

```
Map<String, Object> args = new HashMap<String, Object>();
args.put("alternate-exchange", "my-ae");
channel.exchangeDeclare("my-direct", "direct", false, false, args);
channel.exchangeDeclare("my-ae", "fanout");
channel.queueDeclare("routed");
channel.queueBind("routed", "my-direct", "key1");
channel.queueDeclare("unrouted");
channel.queueBind("unrouted", "my-ae", "");
```

#### How Alternate Exchanges Work

Whenever an exchange with a configured AE cannot route a message to any queue, it publishes the message to the specified AE instead. If that AE does not exist then a warning is logged. If an AE cannot route a message, it in turn publishes the message to its AE, if it has one configured. This process continues until either the message is successfully routed, the end of the chain of AEs is reached, or an AE is encountered which has already attempted to route the message.

For example if we publish a message to 'my-direct' with a routing key of 'key1' then that message is routed to the 'routed' queue, in accordance with the standard AMQP behaviour. However, when publishing a message to 'my-direct' with a routing key of 'key2', rather than being discarded the message is routed via our configured AE to the 'unrouted' queue.

The behaviour of an AE purely pertains to routing. **If a message is routed via an AE it still counts as routed for the purpose of the 'mandatory' flag, and the message is otherwise unchanged (does NOT return via Basic.Return).**

Example will **NOT print "Message Failed"**:
```
private static final String RABBIT_URI = "amqp://guest:guest@localhost:5672";

public static void main(String[] args) throws NoSuchAlgorithmException, KeyManagementException, URISyntaxException {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setUri(RABBIT_URI);
    try (Connection connection = factory.newConnection();
         Channel channel = connection.createChannel()) {

        channel.exchangeDeclare("my-direct", "direct", false, false, Map.of("alternate-exchange", "my-ae"));
        channel.exchangeDeclare("my-ae", "fanout");
        channel.queueDeclare("routed", true, false, false, Map.of());
        channel.queueBind("routed", "my-direct", "key1");
        channel.queueDeclare("unrouted", true, false, false, Map.of());
        channel.queueBind("unrouted", "my-ae", "");
        channel.addReturnListener(Send::handleReturn);

        String message = "Hello World!";
        var i = 0;
        while (i < 3) {
            //Set true to mandatory field and send to different key than binded
            channel.basicPublish("my-direct", "key2", true, null, message.getBytes());
            System.out.println(" [x] Sent '" + message + "'");
            i++;
        }
        TimeUnit.SECONDS.sleep(5); //To wait for error messages
    } catch (TimeoutException | IOException | InterruptedException e) {
        e.printStackTrace();
    }
}

private static void handleReturn(int replyCode, String replyText, String exchange, String routingKey, AMQP.BasicProperties properties, byte[] body) {
    System.out.println("-".repeat(10) + "Message Failed" + "-".repeat(10));
    System.out.println(replyCode);
    System.out.println(replyText);
    System.out.println(exchange);
    System.out.println(routingKey);
    System.out.println(new String(body));
}
```

### Batch processing with transactions

Before there were delivery confirmations, the only way you could be sure a message was delivered was through transactions. The AMQP transaction, or **TX**, class **provides a mechanism by which messages can be published to RabbitMQ in batches and then committed to a queue or rolled back**.

The transactional mechanism provides a method by which a publisher can be **notified of the successful delivery of a message to a queue on the RabbitMQ broker.** To begin a transaction, the publisher sends a TX.Select RPC request to RabbitMQ, and RabbitMQ will respond with a TX.SelectOk response. Once the transaction has been opened, the publisher may send one or more messages to RabbitMQ (figure 4.6).

![Publisher_Transactions](Publisher_Transactions.PNG)

When RabbitMQ is unable to route a message due to an error, such as a non-existent exchange, it will return the message with a Basic.Return response prior to sending a TX.CommitOk response. Publishers wishing to abort a transaction should send a TX.Rollback RPC request and wait for a TX.RollbackOk response from the broker prior to continuing.

***NOTE*** Unfortunately for those looking for true atomicity, RabbitMQ only implements atomic transactions when every command issued affects a single queue. If more than one queue is impacted by any of the commands in the transaction, the commit won’t be atomic.

**Consider using Publisher Confirms as a lightweight alternative - it's faster and can provide both positive and negative confirmation.**

### Surviving node failures with HA queues

As you look to strengthen the contract between publishers and RabbitMQ to guarantee message delivery, don’t overlook the important role that **highly available queues (HA queues)** can play in mission-critical messaging architectures. HA queues - **not part of the AMQP specification** - is a feature that allows queues to have redundant copies across multiple servers.

HA queues require a clustered RabbitMQ environment and can be set up in one of two ways: 
* using AMQP 
* using the web-based management interface

# Code example TODO:

**When a message is published into a queue that’s set up as an HA queue, it’s sent to each server in the cluster that’s responsible for the HA queue (figure 4.7). Once a message is consumed from any node in the cluster, all copies of the message will be immediately removed from the other nodes.**

![HA_Queues](HA_Queues.png)

HA queues **can span every server in a cluster, or only individual nodes.** To specify individual nodes, instead of passing in an argument of **"x-ha-policy": "all"**, pass in an **"x-ha-policy": "nodes"**  and then another argument, x-ha-nodes containing a list of the nodes the queue should be configured on.

# CODE EXAMPLE TODO:

***NOTE*** Even if you don’t have node1 , node2 , or node3 defined, RabbitMQ will allow you to define the queue,

**HA queues have a single primary server node, and all the other nodes are secondary. Should the primary node fail, one of the secondary nodes will take over the role of primary node.** Should a secondary node be lost in an HA queue configuration, the other nodes would continue to operate as they were, sharing the state of operations that take place across all configured nodes. **When a lost node is added back**, or a new node is added to the cluster, it won’t contain any messages that are already in the queue across the existing nodes. Instead, **it will receive all new messages and only be in sync once all the previously published messages are consumed.**

### HA queues with transactions

If you’re using transactions or delivery confirmations, RabbitMQ won’t send a successful response until the message has been confirmed to be in all active nodes in the HA queue definition. This can create a delay in responding to your publishing application.

### Persisting messages to disk via delivery-mode 2

If the RabbitMQ broker dies prior to consuming the messages, they’ll be lost forever unless you tell RabbitMQ when publishing the message that you want the messages persisted to disk.

Delivery-mode is one of the message properties specified as part of AMQP’s Basic.Properties definition. If deliver-mode is set to:
* 1 - does not need to store message to disk (default)
* 2 - all messages are stored to disk. If the RabbitMQ broker is restarted the message will still be in the queue once RabbitMQ is running again.

***NOTE*** In **addition to delivery-mode of 2**, for messages to truly survive a restart of a RabbitMQ broker, your **queues must be declared as durable** when they’re created.

It’s important to correctly size your hardware needs when using persisted messages. An undersized server that’s tasked with a heavy write workload can bring a whole RabbitMQ server to a crawl. **Although message persistence is one of the most important ways to guarantee that your messages will ultimately be delivered, it’s also one of the most costly.**

## When RabbitMQ pushes back












































