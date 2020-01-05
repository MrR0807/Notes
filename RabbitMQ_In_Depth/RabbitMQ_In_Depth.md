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

TODO

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






















































