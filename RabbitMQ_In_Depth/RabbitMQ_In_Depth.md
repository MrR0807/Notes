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
* Do you need to ensure that all messages are received, or can they be discarded?
* Can you receive messages and then acknowledge or reject them as a batch operation?
* If not, can you use transactions to improve performance by automatically batching your individual operations?
* Do you really need transactional commit and rollback functionality in your consumers?
* Does your consumer need exclusive access to the messages in the queues it’s consuming from?
* What should happen when your consumer encounters an error? Should the message be discarded? Requeued? Dead-lettered?

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

**When a message is published into a queue that’s set up as an HA queue, it’s sent to each server in the cluster that’s responsible for the HA queue (figure 4.7). Once a message is consumed from any node in the cluster, all copies of the message will be immediately removed from the other nodes.**

![HA_Queues](HA_Queues.png)

HA queues **can span every server in a cluster, or only individual nodes.** To specify individual nodes, instead of passing in an argument of **"x-ha-policy": "all"**, pass in an **"x-ha-policy": "nodes"**  and then another argument, x-ha-nodes containing a list of the nodes the queue should be configured on.

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

**TCP Backpressure** - a mechanism which RabbitMQ utilizes and stops accepting low-level data on the TCP socket. This method works well to protect RabbitMQ from being overwhelmed by a single publisher.

Internally, RabbitMQ uses the notion of credits to manage when it’s going to pushback against a publisher. When a new connection is made, the connection is allotted a predetermined amount of credits it can use. Then, as each RPC command is received by RabbitMQ, a credit is decremented. Once the RPC request has been internally processed, the connection is given the credit back. A connection’s credit balance is evaluated by RabbitMQ to determine if it should read from a connection’s socket. If a connection is out of credits, it’s simply skipped until it has enough credits.

**Connection.Blocked** and **Connection.Unblocked** are asynchronous methods that can be sent at any time to notify the client when RabbitMQ has blocked the publishing client and when that block has been removed.

### Checking the connection status with Java listener

```
ConnectionFactory factory = new ConnectionFactory();
Connection connection = factory.newConnection();
connection.addBlockedListener(new BlockedListener() {
    public void handleBlocked(String reason) throws IOException {
        // Connection is now blocked
    }

    public void handleUnblocked() throws IOException {
        // Connection is now unblocked
    }
});
```

Or you can check RabbitMQ Management page, next to Connection.

![Blocked_Connection](Blocked_Connection.png)

# Chapter 5. Don’t get messages; consume them

## Basic.Get vs. Basic.Consume

RabbitMQ implements two different AMQP RPC commands for retrieving messages from a queue: Basic.Get and Basic.Consume. In the simplest terms, **Basic.Get is a polling model, whereas Basic.Consume is a push model**.

### Basic.Get

When your application uses a Basic.Get request to retrieve messages, it must send a new request each time it wants to receive a message, even if there are multiple messages in the queue.

In simple message velocity tests, using **Basic.Consume is at least twice as fast as using Basic.Get.** The most obvious reason for the speed difference is that **with Basic.Get**, each message delivered carries with it the overhead of the **synchronous communication** with RabbitMQ.

### Basic.Consume

In contrast, by consuming messages with the Basic.Consume RPC command, you’re registering your application with RabbitMQ and telling it to send messages asynchronously to your consumer as they become available.

![Basic_Consume](Basic_Consume.PNG)

## Performance-tuning consumers

As figure 5.5 points out, there are several options that can be used to speed message delivery from RabbitMQ to your application.

![Consumer_Tuning_Perfomance](Consumer_Tuning_Perfomance.PNG)

### Using no-ack mode for faster throughput

In Java client, noAck as been change to autoAck. [Github issue](https://github.com/rabbitmq/rabbitmq-dotnet-client/issues/255).

Consuming messages with no_ack=True is the fastest way to have RabbitMQ deliver messages to your consumer, but it’s also the least reliable way to send messages.

Not recommended.

### Controlling consumer prefetching via quality of service settings

The AMQP specification calls for channels to have a quality of service (QoS) setting where a consumer can ask for a prespecified number of messages to be received prior to the consumer acknowledging receipt of the messages. The QoS setting allows RabbitMQ to more efficiently send messages by specifying how many messages to preallocate for the consumer.

**Unlike a consumer with acknowledgments disabled (no_ack=True)**, if your consumer application crashes before it can acknowledge the messages, all the prefetched messages will be returned to the queue when the socket closes.

As part of this RPC request, you can specify whether the QoS setting is for the channel it’s sent on or all channels open on the connection.

More information can be found here: https://www.rabbitmq.com/consumer-prefetch.html

#### ACKNOWLEDGING MULTIPLE MESSAGES AT ONCE

One of the nice things about using the QoS setting is that you don’t need to acknowledge each message received with a Basic.Ack RPC response. Instead, the Basic.Ack RPC response has an attribute named multiple, and when it’s set to True it lets RabbitMQ know that your application would like to acknowledge all previous unacknowledged messages.

```
public static void main(String[] args) throws IOException, TimeoutException, NoSuchAlgorithmException, KeyManagementException, URISyntaxException {
    String url = "amqp://guest:guest@localhost:5672";
    ConnectionFactory factory = new ConnectionFactory();
    factory.setUri(url);

    Connection connection = factory.newConnection();
    Channel channel = connection.createChannel();
    final AtomicInteger window = new AtomicInteger(0);
    final int prefetchCount = 5;

    DeliverCallback deliverCallback = (consumerTag, message) -> {
        consumeMessage(message);
        window.getAndIncrement();
        if (window.get() == prefetchCount) {
            System.out.println("-".repeat(10) + "Window Limit Reached" + "-".repeat(10));
            //Sleep for 10 seconds to see the nack messages before ther are acknowledge in RabbitMQ management page
            sleep();
            channel.basicAck(message.getEnvelope().getDeliveryTag(), true);
            window.getAndSet(0);
        }

    };
    channel.basicQos(prefetchCount);

    boolean autoAck = false;
    channel.basicConsume(QUEUE_NAME, autoAck, deliverCallback, consumerTag -> {
    });
}
```

**Acknowledging multiple messages at the same time allows you to minimize the network communications required to process your messages, improving message throughput.** Should you successfully process some messages and your application dies prior to acknowledging them, all the unacknowledged messages will return to the queue to be processed by another consumer process.

### Using transactions with consumers

Like when publishing messages into RabbitMQ, transactions allows your consumer applications to commit and roll back batches of operations. **Transactions (AMQP TX class) can have a negative impact on message throughput with one exception. If you aren’t using QoS settings, you may actually see a slight performance improvement when using transactions to batch your message acknowledgments (figure 5.9).**

![Perfomance_Improvement_Batch_Transactions](Perfomance_Improvement_Batch_Transactions.PNG)

## Rejecting messages

Acknowledging messages is a great way to ensure that RabbitMQ knows the consumer has received and processed a message before it discards it, but what happens when a problem is encountered, either with the message or while processing the message? In these scenarios, **RabbitMQ provides two mechanisms for kicking a message back to the broker: Basic.Reject and Basic.Nack**.

### Basic.Reject

Basic.Reject is an AMQP-specified RPC response to a delivered message that informs the broker that the message couldn’t be processed. When a consumer rejects a message, you can instruct RabbitMQ to either discard the message or to requeue the message with the **requeue flag. When the requeue flag is enabled, RabbitMQ will put the message back into the queue to be processed again.**

I often use this feature in writing consumer applications that communicate with other services, such as databases or remote APIs. Instead of writing logic in my consumer for retrying on failure due to a remote exception, such as a disconnected database cursor or failure to contact a remote API, I simply catch the exception and reject the message with requeue set to True.

Also, you can implement "two-strikes and you're out" policy via checking property [isRedeliver()](https://javadoc.io/static/com.rabbitmq/amqp-client/5.7.3/com/rabbitmq/client/Envelope.html#isRedeliver--). 

In code example, check if message was redelivered. If yes, reject and set requeue:false.
```
DeliverCallback deliverCallback = (consumerTag, message) -> {
    consumeMessage(message);
    if (message.getEnvelope().isRedeliver()) {
        channel.basicReject(message.getEnvelope().getDeliveryTag(), false);
    } else {
        channel.basicReject(message.getEnvelope().getDeliveryTag(), true);
    }
};

boolean autoAck = false;
channel.basicConsume(QUEUE_NAME, autoAck, deliverCallback, consumerTag -> {});
```

### Basic.Nack

**Basic.Reject allows for a single message to be rejected**. Basic.Nack method implements the same behavior as the Basic.Reject response method but it adds the missing multiple argument to complement the Basic.Ack multiple behavior.

***WARNING*** This is not part of AMQP specification.


### Dead letter exchanges

RabbitMQ’s dead-letter exchange (DLX) feature is an extension to the AMQP specification and is an optional behavior that can be tied to rejecting a delivered message.

For example, one type of consumer application I’ve written takes XML-based messages and turns them into PDF files using a standard markup language called XSL:FO. By combining the XSL:FO document and the XML from the message, I was able to use Apache’s FOP application to generate a PDF file and subsequently file it electronically. The process worked pretty well, but every now and then it would fail. By using a dead-letter exchange on the queue, I was able to inspect the failing XML documents and manually run them against the XSL:FO document to troubleshoot the failures. Without the dead-letter exchange, I would have had to add code to my consumer that wrote out the XML document to some place where I could then manually process it via the command line. Instead, I was able to interactively run my consumer by pointing it at a different queue.

The only thing that makes an exchange a dead-letter exchange is the declared use of the exchange for rejected messages when creating a queue. Upon rejecting a message that isn’t requeued, RabbitMQ will route the message to the exchanged specified in the **queue’s x-dead-letter-exchange argument (figure 5.11).**

![Dead_Letter_Exchange](Dead_Letter_Exchange.PNG)

Specifying a dead-letter exchange when declaring a queue is fairly trivial. Simply pass in the exchange name as the dead_letter_exchange argument when creating the rabbitpy Queue object or as the x-dead-letter-exchange argument when issuing the Queue.Declare RPC request.

***NOTE***. If routing key is not used, **exchange has to be of type fanout.**

```
channel.exchangeDeclare("dead-exchange", "fanout");
channel.queueDeclare("dead-queue", false, false, false, Map.of());
channel.queueBind("dead-queue", "dead-exchange", "");

channel.exchangeDeclare(EXCHANGE_NAME, "direct");
channel.queueDeclare(QUEUE_NAME, false, false, false, Map.of("x-dead-letter-exchange", "dead-exchange"));
channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, ROUTING_KEY);
```
If routing key is used, **exchange can be of type direct.**
```
channel.exchangeDeclare("dead-exchange", "direct");
channel.queueDeclare("dead-queue", false, false, false, Map.of());
channel.queueBind("dead-queue", "dead-exchange", "some-routing-key");

channel.exchangeDeclare(EXCHANGE_NAME, "direct");
channel.queueDeclare(QUEUE_NAME, false, false, false, Map.of("x-dead-letter-exchange", "dead-exchange",
        "x-dead-letter-routing-key", "some-routing-key"));
channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, ROUTING_KEY);
```

## Controlling queues

When defining a queue, there are multiple settings that determine a queue’s behavior. Queues can do the following, and more:
* Auto-delete themselves
* Allow only one consumer to consume from them
* Automatically expire messages
* Keep a limited number of messages
* Push old messages off the stack

### Temporary queues

#### AUTOMATICALLY DELETING QUEUES

RabbitMQ provides for queues that will delete themselves once they’ve been used and are no longer needed. It’s important to note that any number of consumers can consume from an automatically deleting queue; the queue will only delete itself when there are no more consumers listening to it.

One use case is a chat style application where each queue represents a user’s inbound chat buffer. If a user’s connection is severed, it’s not unreasonable for such an application to expect that the queue and any unread messages should be deleted.

#### ALLOWING ONLY A SINGLE CONSUMER

Without the exclusive setting enabled on a queue, RabbitMQ allows for very promiscuous consumer behavior. It sets no restrictions on the number of consumers that can connect to a queue and consume from it.

**A queue that’s declared as exclusive may only be consumed by the same connection and channel that it was declared on**, unlike queues that are declared with **auto_delete** set to True, which can have any number of consumers from any number of connections. An exclusive queue will also automatically be deleted when the channel that the queue was created on is closed, which is similar to how a queue that has auto-delete set will be removed once there are no more consumers subscribed to it.

#### AUTOMATICALLY EXPIRING QUEUES

While we’re on the subject of queues that are automatically deleted, RabbitMQ allows for an optional argument when declaring a queue that will tell RabbitMQ to delete the queue if it has gone unused for some length of time.

Creating an automatically expiring queue is as simple as declaring a queue with an x-expires argument with the queue’s time to live (TTL) specified in milliseconds.

```
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-message-ttl", 60000);
channel.queueDeclare("myqueue", false, false, false, args);
```

There are some strict rules around automatically expiring queues:
* The queue will only expire if it has no consumers
* The queue will only expire if there has been no Basic.Get request for the TTL duration
* As with any other queue, the settings and arguments declared with an x-expires argument can’t be redeclared or changed
* RabbitMQ makes no guarantees about how promptly it will remove the queue post expiration

[More on this topic](https://www.rabbitmq.com/ttl.html)

### Permanent queues

#### QUEUE DURABILITY

When declaring a queue that should persist across server restarts, the durable flag should be set to True. Often queue durability is confused with message persistence. As we discussed in the previous chapter, messages are stored on disk when a message is published with the delivery-mode property set to 2. The durable flag, in contrast, instructs RabbitMQ that you want the queue to be configured until a Queue.Delete request is called.

**Durability of a queue does not make messages that are routed to that queue durable. If broker is taken down and then brought back up, durable queue will be re-declared during broker startup, however, only persistent messages will be recovered.**

#### AUTO-EXPIRATION OF MESSAGES IN A QUEUE

With non-mission-critical messages, sometimes it’s better to have them automatically go away if they hang around too long without being consumed.

```
byte[] messageBodyBytes = "Hello, world!".getBytes();
AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder()
                                   .expiration("60000")
                                   .build();
channel.basicPublish("my-exchange", "routing-key", properties, messageBodyBytes);
```

#### MAXIMUM LENGTH QUEUES

As of RabbitMQ 3.1.0, queues may be declared with a maximum size. If you set the x-max-length argument on a queue, once it reaches the maximum size, RabbitMQ will drop messages from the front of the queue as new messages are added.

```
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-max-length", 10);
channel.queueDeclare("myqueue", false, false, false, args);
```

# Chapter 6. Message patterns via exchange routing

Four basic types of exchanges:
* Direct exchange
* Fanout exchange
* Topic exchange
* Headers exchange

## Simple message routing using the direct exchange

The direct exchange is useful when you’re going to deliver a message with a specific target, or a set of targets. **Any queue that’s bound to an exchange with the same routing key that’s being used to publish a message will receive the message.**

![Direct_Exchange](Direct_Exchange.PNG)

This architecture is good for computationally complex processes such as image or video processing, leveraging remote RPC workers. If this application were running in the cloud, for example, the application publishing the request could live on small-scale virtual machines, and the image processing worker could make use of larger hardware

## Broadcasting messages via the fanout exchange

Where a direct exchange allows for queues to receive targeted messages, a fanout exchange doesn’t discriminate. All messages published through a fanout exchange are delivered to all queues in the fanout exchange. **This provides significant performance advantages** because RabbitMQ doesn’t need to evaluate the routing keys when delivering messages, but the lack of selectivity means all applications consuming from queues bound to a fanout exchange should be able to consume messages delivered through it.

**When binding a queue and publishing message in fanout mode, routing key is not taken into consideration.**

Fanout exchanges provide a great way to allow every consumer access to the fire hose of data. This can be a double-edged sword, however, because consumers can’t be selective about the messages they receive.

## Selectively routing messages with the topic exchange

Like direct exchanges, topic exchanges will route messages to any queue bound with a matching routing key. But by using a period-delimited format, queues may bind to routing keys using wildcard-based pattern matching.

**An asterisk (\*) will match all characters up to a period in the routing key, and the pound character (#) will match all characters that follow, including any subsequent periods.**

**There can be as many words in the routing key as you like, up to the limit of 255 bytes.**

A topic exchange is excellent for routing a message to queues so that single-purpose consumers can perform different actions with it.

**Using a topic exchange with namespaced routing keys is a good choice for futureproofing your applications. Even if the pattern matching in routing is overkill for your needs at the start, a topic exchange (used with the right queue bindings) can emulate the behavior of both direct and fanout exchanges.** To emulate the direct exchange behavior, bind queues with the full routing key instead of using pattern matching. Fanout exchange behavior is even easier to emulate, as queues bound with # as the routing key will receive all messages published to a topic exchange. With such flexibility, it’s easy to see why the topic exchange can be a powerful tool in your messagingbased architecture.

## Selective routing with the headers exchange

The fourth built-in exchange type is the headers exchange. It allows for arbitrary routing in RabbitMQ by using the headers table in the message properties. Queues that are bound to the headers exchange use the Queue.Bind arguments parameter to pass in an array of key/value pairs to route on and an x-match argument. **The x-match argument is a string value that’s set to any or all. If the value is any, messages will be routed if any of the headers table values match any of the binding values. If the value of x-match is all, all values passed in as Queue.Bind arguments must be matched.**

Producer:

```
public static void main(String[] args) throws NoSuchAlgorithmException, KeyManagementException, URISyntaxException {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setUri(RABBIT_URI);
    try (Connection connection = factory.newConnection();
         Channel channel = connection.createChannel()) {

        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.HEADERS);

        var i = 0;
        while (i < 5) {
            //Set true to mandatory field
            String message = "Hello World! Count: " + i;
            AMQP.BasicProperties props = new AMQP.BasicProperties.Builder()
                    .headers(Map.of("x-match", "all", "source", "profile", "object", "image"))
                    .build();
            channel.basicPublish(EXCHANGE_NAME, "", props, message.getBytes());
            i++;
        }
        System.out.println("Sent message count: " + i);
    } catch (TimeoutException | IOException e) {
        e.printStackTrace();
    }
}
```

Consumer:

```
private static final String EXCHANGE_NAME = "my-exchange";
public static final String QUEUE_NAME = "my-queue";

public static void main(String[] args) throws IOException, TimeoutException, NoSuchAlgorithmException, KeyManagementException, URISyntaxException {
    String url = "amqp://guest:guest@localhost:5672";
    ConnectionFactory factory = new ConnectionFactory();
    factory.setUri(url);

    Connection connection = factory.newConnection();
    Channel channel = connection.createChannel();

    channel.queueDeclare(QUEUE_NAME, false, false, false, Map.of());
    channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "", Map.of("x-match", "all", "source", "profile", "object", "image"));

    DeliverCallback deliverCallback = Receive::consumeMessage;

    boolean autoAck = true;
    channel.basicConsume(QUEUE_NAME, autoAck, deliverCallback, consumerTag -> {});
}
```

Conventional wisdom is that the headers exchange is significantly slower than the other exchange types due to the additional computational complexity. But in benchmarking for this chapter, I found that there was no significant difference between any of the built-in exchanges with regard to performance when using the same quantity of values in the headers property.

## Going meta: exchange-to-exchange routing

The mechanism for exchange-toexchange binding is very similar to queue binding, but instead of binding a queue to an exchange, you bind an exchange to another exchange using the Exchange.Bind RPC method.

**When using exchange-to-exchange binding, the routing logic that’s applied to a bound exchange is the same as it would be if the bound object were a queue.**

## Routing messages with the consistent-hashing exchange

The consistent-hashing exchange, a plugin that’s distributed with RabbitMQ, distributes data among the queues that are bound to it. It can be used to load-balance the queues that receive messages published into it. You can use it to distribute messages to queues on different physical servers in a cluster or to queues with single consumers, providing the potential for faster throughput than if RabbitMQ were distributing messages to multiple consumers in a single queue. When using databases or other systems that can directly integrate with RabbitMQ as a consumer, the consistent-hashing exchange can provide a way to shard out data without having to write middleware.

The consistent-hashing exchange uses a consistent-hashing algorithm to pick which queue will receive which message, with all queues being potential destinations. Instead of queues being bound with a routing key or header values, they’re bound with an integerbased weight that’s used as part of the algorithm for determining message delivery. Consistent-hashing algorithms are commonly used in clients for network-based caching systems.

**The consistent-hashing exchange doesn’t roundrobin the messages**, but rather deterministically routes messages based upon a hash value of the routing key or a message properties header-type value.

## Summary

![Exchange_Types_Summary](Exchange_Types_Summary.PNG)

# Managing RabbitMQ in the data center or the cloud

# Chapter 7. Scaling RabbitMQ with clusters

## About clusters

A RabbitMQ cluster creates a seamless view of RabbitMQ across two or more servers. In a RabbitMQ cluster, runtime state containing exchanges, queues, bindings, users, virtual hosts, and policies are available to every node. Because of this shared runtime state, every node in a cluster can bind, publish, or delete an exchange that was created when connected to the first node (figure 7.1).

![Cross_Node_Publishing](Cross_Node_Publishing.PNG)

Despite the advantages of using RabbitMQ’s built-in clustering, it’s important to recognize the limitations and downsides of RabbitMQ clustering. **First, clusters are designed for low-latency environments. You should never create RabbitMQ clusters across a WAN or internet connection. State synchronization and cross-node message delivery demand low-latency communication that can only be achieved on a LAN.** You can run RabbitMQ in cloud environments such as Amazon EC2, but not across availability zones. To synchronize RabbitMQ messages in high-latency environments, you’ll want to look at the Shovel and Federation tools.

Another important issue to consider with RabbitMQ clusters is cluster size. The work and overhead of maintaining the shared state of a cluster is directly proportionate to the number of nodes in the cluster.

### Clusters and the management UI

The Overview page of the management UI contains top-level information about a RabbitMQ cluster and its nodes (figure 7.2).

![Rabbit_UI_Nodes](Rabbit_UI_Nodes.PNG)

### Cluster node types

There are two types of nodes:
* **Disk Node.** Disk nodes store the runtime state of a cluster to both RAM and disk.
* **RAM Node.** RAM nodes only store the runtime state information in an in-memory database.

#### THE STATS NODE

The stats node only works in conjunction with disk nodes. The stats node is responsible for gathering all of the statistical and state data from each node in a cluster. **Only one node in a cluster can be the stats node at any given time.**

A good strategy for larger cluster setups is to have a dedicated management node that’s your primary disk node and the stats node, and to have at least one more disk node to provide failover capabilities (figure 7.3).

![Stats_Node](Stats_Node.PNG)

In a cluster topology setup with two disk nodes, if the primary node fails, the stats node designation will be transferred to the secondary disk node. Should the primary disk node come back up, it will not regain the stats node designation unless the secondary disk node with the stats node designation stops or leaves the cluster.

The stats node plays an important part in managing your RabbitMQ clusters. Without the rabbitmq-management plugin and a stats node, it can be difficult to get clusterwide visibility of performance, connections, queue depths, and operational issues.

### Clusters and queue behavior

#### HIGHLY AVAILABLE QUEUES

In large clusters, you should consider just how many nodes your queue should use prior to declaring it. Because HA queues have a copy of each message on each node, you should ask yourself if you need more than two or three nodes to ensure that you don’t lose any messages.

## Cluster setup

### Adding nodes to the cluster

There are two ways to add nodes to a cluster with RabbitMQ. The first involves editing the rabbitmq.config configuration file and defining each node in a cluster. This method is preferred if you’re using an automated configuration management tool.

Alternatively, you can add and remove nodes from a cluster in an ad hoc manner by using the rabbitmqctl command-line tool.

#### ERLANG COOKIES

To secure multi-node communication, Erlang and the RabbitMQ process have a **shared secret file called a cookie.**

The Erlang cookie file for RabbitMQ is contained in the RabbitMQ data directory. On \*NIX platforms, the file is usually at /var/lib/rabbitmq/.erlang.cookie, though this can vary by distribution and package. The cookie file contains a short string and should be the same on every node in a cluster. If the cookie file isn’t the same on each node in the cluster, the nodes won’t be able to communicate with each other.

The cookie file will be created the first time you run RabbitMQ on any given server, or if the file is missing. **When setting up a cluster, you should ensure that RabbitMQ isn’t running and you overwrite the cookie file with the shared cookie file prior to starting RabbitMQ again.**

#### CREATING AD HOC CLUSTERS

To do so, you must first tell RabbitMQ on the secondary node to stop using rabbitmqctl. You won’t be stopping the RabbitMQ server process itself, but using rabbitmq to instruct RabbitMQ to halt internal processes in Erlang that allow it to process connections. Run the following command in the terminal:
```
rabbitmqctl stop_app
```

You should see output similar to the following:
```
Stopping node rabbit@secondary ...
...done.
```
Now that the process has stopped, you need to erase the state in this RabbitMQ node, making it forget any runtime configuration data or state that it has. To do this, you’ll instruct it to reset its internal database:
```
rabbitmqctl reset
```
You should see a response similar to this:
```
Resetting node rabbit@secondary ...
...done.
```
Now you can join it to the primary node and form the cluster:
```
rabbitmqctl join_cluster rabbit@primary
```
This should return with the following output:
```
Clustering node rabbit@secondary with rabbit@primary ...
...done.
```
Finally, start the server again using the following command:
```
rabbitmqctl start_app
```
You should see the output that follows:
```
Starting node rabbit@secondary ...
...done.
```
Congratulations! You now have a running RabbitMQ cluster with two nodes.

# TODO

# Chapter 8. Cross-cluster message distribution

Federation plugin provides two different ways to get messages from one cluster to another:
* By using a **federated exchange**, messages published to an exchange in another RabbitMQ server or cluster are automatically routed to bound exchanges and queues on the downstream host. 
* Alternatively, if your needs are more specific, **federated queues** provide a way to target the messages in a single queue instead of an exchange. Federated queues, on the other hand, allow for downstream nodes to act as consumers of shared queues on upstream nodes, **providing the ability to roundrobin messages across multiple downstream nodes.**

### Federated exchanges

When the federated server is set up, all you have to do is create policies that apply to the exchanges you need the messages from. If the upstream RabbitMQ server has an exchange called events that the login, article, and comment messages are published into, your downstream RabbitMQ server should create a federation policy matching that exchange name.

Once RabbitMQ is publishing messages from the upstream server to the downstream queue, you don’t have to worry about what will happen if the internet connectivity is severed between the two. When connectivity is restored, RabbitMQ will dutifully reconnect to the main RabbitMQ cluster and start locally queuing all of the messages that were published by the website while the connection was down.

### Federated queues

This is especially useful for messaging workloads where a particular queue may have heavy spikes of publishing activity and much slower or throttled consuming. When using a federated queue, message publishers use the upstream node or cluster, and messages are distributed to the same-named queue across all downstream nodes.

Like the upstream queue, downstream queues can exist on a single node or as part of an HA queue in a cluster. **The federation plugin ensures that downstream queues will only receive messages when the queues have consumers available for processing messages.
By checking the consumer count for each queue and only binding to the upstream node when consumers are present, it prevents idle messages from collecting in consumerless queues.**

## Connecting upstream

Federation configuration has **two parts: the upstream configuration and a federation policy.** 
First, the downstream node is configured with the information required for the node to make an AMQP connection to the upstream node. Then policies are created that apply upstream connections and configuration options to downstream exchanges or queues. A single RabbitMQ server can have many federation upstreams and many federation policies.

### Defining federation upstreams

When installed, the federation management plugin adds two new Admin tabs to the RabbitMQ management interface: Federation Status and Federation Upstreams.

When launching RabbitMQ in docker for the first time, **federation plugin will not be enabled.** You won't see "Federation Status" and "Federation Upstream"

![RabbitMQ_With_Federation_Disabled](RabbitMQ_With_Federation_Disabled.PNG)

To enable federation, run:
```
rabbitmq-plugins enable rabbitmq_federation
```
Output will be something like this:
```
Enabling plugins on node rabbit@rabbit_three:
rabbitmq_federation
The following plugins have been configured:
  rabbitmq_federation
  rabbitmq_management
  rabbitmq_management_agent
  rabbitmq_web_dispatch
Applying plugin configuration to rabbit@rabbit_three...
The following plugins have been enabled:
  rabbitmq_federation
```
And then enable federation plugin management:
```
rabbitmq-plugins enable rabbitmq_federation_management
```
Output:
```
Enabling plugins on node rabbit@rabbit_three:
rabbitmq_federation_management
The following plugins have been configured:
  rabbitmq_federation
  rabbitmq_federation_management
  rabbitmq_management
  rabbitmq_management_agent
  rabbitmq_web_dispatch
Applying plugin configuration to rabbit@rabbit_three...
The following plugins have been enabled:
  rabbitmq_federation_management
```

Then Federation tabs will appear.

![RabbitMQ_With_Federation_Enabled.PNG](RabbitMQ_With_Federation_Enabled)

The Federation Upstreams tab is the first place you’ll go to start the configuration process. Only the name and AMQP URI for the connection are required. In a production environment, you’ll likely want to configure the other options as well.

**The full specification for URI can be found [here](https://www.rabbitmq.com/uri-spec.html).**

The connection won’t be used until a policy is created that references the upstream. When a policy is applied using the upstream, the federation plugin will connect to the upstream node. Should it be disconnected due to a routing error or some other network event, the default behavior is to try to reconnect once per second.

![Federation_Upstream_Input](Federation_Upstream_Input.PNG)

### Defining a policy

Federation configuration is managed using RabbitMQ’s policy system, which provides a flexible way to dynamically configure the rules that tell the federation plugin what to do. When you create a policy, you first specify a policy name and a pattern. The pattern can either evaluate for direct string matching or it can specify a regular expression (regex) pattern to match against RabbitMQ objects. The pattern can be compared against exchanges, queues, or both exchanges and queues. Policies can also specify a priority that’s used to determine which policy should be applied to queues or exchanges that match multiple policies. When a queue or exchange is matched by multiple policies, the policy with the highest priority value wins.

For a first example, you’ll create a policy named federation-test that will do string-equality checking against an exchange named test (figure 8.17). To tell the federation plugin that you want to federate the exchange from the cluster-a upstream, enter a key of federation-upstream with a value of cluster-a in the definition table. Once you’ve entered that information, click the Add Policy button to add it to the system.

![Federation_Policy](Federation_Policy.PNG)

With the policy added, you’ll need to add the test exchange to both nodes. You can use the Exchanges tab of each management interface to add the exchange. To prevent the cluster-b node from trying to federate a non-existent exchange on the cluster-a node, declare the exchange on the cluster-a node first. You can use any of the built-in exchange types, but I recommend using a topic exchange for flexibility in experimenting. Whichever type you select, you should **be consistent and use the same exchange type for the test exchange on both cluster-a and cluster-b.**

**Example**

In downstream RabbitMQ add "Federation Upstream". ***Name*** has to be equal to the name of the RabbitMQ.

![Add_Federation_Upstream_Example](Add_Federation_Upstream_Example.PNG)

Add policy. ***federation-upstream*** has to be equal to upstream RabbitMQ.

![Add_Federation_Policy](Add_Federation_Policy.PNG)

Add exchange to upstream RabbitMQ.Add exchange to downstream RabbitMQ.

![Upstream_Exchanges](Upstream_Exchanges.PNG)
![Downstream_Exchanges](Downstream_Exchanges.PNG)

To help identify messages that were distributed via federation, the federation plugin adds an **x-received-from** field to the headers table in the message properties. The value of the field is a key/value table that includes the upstream uri, exchange, cluster-name, and a flag indicating if the message was redelivered.

### Leveraging upstream sets

In addition to defining individual upstream nodes, the federation plugin provides the ability to group multiple nodes together for use in a policy.

#### PROVIDING REDUNDANCY

For example, imagine your upstream node is part of a cluster. You could create an upstream set that defines each node in the upstream cluster, allowing the downstream node to connect to every node in the cluster, ensuring that should any one node go down, messages published into the upstream cluster won’t be missed downstream (figure 8.21).

#### CREATING AN UPSTREAM SET

TODO


#### Bidirectional federated exchanges

The examples in this chapter have thus far covered distributing messages from an upstream exchange to a downstream exchange, but federated exchanges can be set up to be bidirectional.

In a bidirectional setup, messages can be published into either node, and using the default configuration, they’ll only be routed once on each node. This setting can be tweaked by the max-hops setting in the upstream configuration. The default value of 1 for max-hops prevents message loops where messages received from an upstream node are cyclically sent back to the same node.

![Bidirectional_Federation](Bidirectional_Federation.PNG)

It’s important to recognize that like any graph structure, the more nodes you add, the more complex things become.

### Federation for cluster upgrades

One of the more difficult operational concerns with managing a RabbitMQ cluster is handling upgrades in a production environment where downtime is undesirable.

If the cluster is large enough, you can move all traffic off a node, remove it from the cluster, and upgrade it. Then you could take another node offline, remove it from the cluster, upgrade it, and add it to a new cluster consisting of the node that was removed first.

Alternatively, provided that you have the resources to set up a mirror of the cluster setup on a new version of the cluster, federation can provide a seamless way to migrate your messaging traffic from one cluster to another.
When using federation as a means to upgrade RabbitMQ, you start by rolling out the new cluster, creating the same runtime configuration on the new cluster, including virtual hosts, users, exchanges, and queues. Once you’ve set up and configured the new cluster, add the federation configuration, including the upstreams and policies to wildcard-match on all exchanges.

As you migrate the consumers off a queue, you should unbind the queue on the old cluster, but don’t delete it. Instead, you can create a temporary policy on the new cluster to federate that queue, moving the messages from the old cluster to the new one. It’s advisable to automate this process as much as possible, because you want to minimize the chance of duplicate messages being added to the new cluster’s queues due to using both the federated exchange and the federated queue.

Once you’ve finished moving all of the consumers off, and you’ve unbound the queues on the old cluster, you can migrate the publishers. When all of your publishers have been moved, you should be fully migrated to the upgraded RabbitMQ cluster. Of course, you may want to keep the federation going for a while to ensure that no rogue publishers are connecting to the old cluster when they’re not expected to do so.
