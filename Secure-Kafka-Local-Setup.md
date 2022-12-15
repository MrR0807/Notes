This documentation is aimed at helping developers to test integration with Secure Kafka locally. Furtheremore, it tries to explain key concepts without overloading with complex information. Some explanations might lack depth, but I’ve tried to balance between overarching theory and need-to-know basis.

Sections:
* Vocabulary
* Kafka Docker images compatibility
* Plaintext Kafka 
* Java Authentication and Authorization Service (JAAS) explained 
* SASL PLAIN Kafka
* SASL SCRAM-SHA-512 Kafka
* TLS in layman's terms
* SSL Kafka - Without CA
* SSL Kafka - With CA
* SASL SSL Kafka

# Vocabulary

* `KAFKA_LISTENERS` vs `KAFKA_ADVERTISED_LISTENERS`. `KAFKA_LISTENERS` is a comma-separated list of listeners and the host/IP and port to which Kafka binds to for listening. `KAFKA_ADVERTISED_LISTENERS` is a comma-separated list of listeners with their host/IP and port. This is the metadata that’s passed back to clients. More information in Confluent Blog post - [Kafka Listeners – Explained](https://www.confluent.io/blog/kafka-listeners-explained/).
* **Simple Authentication and Security Layer (SASL)** is a framework for providing authentication using different mechanisms in connection-oriented protocols. Kafka brokers support the following SASL mechanisms:
  * **GSSAPI:** Kerberos authentication.
  * **PLAIN:** User name/password authentication.
  * **SCRAM-SHA-256 and SCRAM-SHA-512:** User name/password authentication.
  * **OAUTHBEARER:** Authentication using OAuth bearer tokens.
* **Transport Layer Security (TLS)**, commonly referred to by the name of its predecessor, Secure Sockets Layer (SSL) supports encryption as well as client and server authentication.
* **Mutual Transport Layer Security (mTLS)** is a process that establishes an encrypted TLS connection in which both parties use X.509 digital certificates to authenticate each other.
* **Kafka security protocols**. Kafka brokers are configured with listeners on one or more endpoints and accept client connections on these listeners. Each listener can be configured with its own security settings. **Kafka supports four security protocols using two standard technologies, SSL and SASL**. Each Kafka security protocol combines a transport layer (PLAINTEXT or SSL) with an optional authentication layer (SASL):
  * **PLAINTEXT:** PLAINTEXT transport layer with no authentication.
  * **SSL:** SSL transport layer with optional SSL client authentication (which is done using mTLS, more on that can be found in ).
  * **SASL_PLAINTEXT:** PLAINTEXT transport layer with SASL client authentication. Some SASL mechanisms also support server authentication. Does not support encryption and hence is suitable only for use within private networks.
  * **SASL_SSL:** SSL transport layer with SASL authentication.
* **Keystore** is used to store private key and identity certificates that a specific program should present to both parties (server or client) for verification.
* **TrustStore** is used to store certificates, optionally signed by Certified Authorities (CA), that verify the certificate presented by the server in an SSL connection.
* **A certificate** contains an identity (say, a server name) and a public key, which is purported to belong to the designated entity (that named server). Certificates can additionally be signed by CA.
* **Certificate Authority** is another piece of public key infrastructure (PKI). It is an entity that stores, signs, and issues digital certificates. The CA makes sure that the public key is really owned by the named entity, and then issues (i.e. signs) the certificate; the CA also has its own public/private key pair. That way, users that see the certificate and know the CA public key can verify the signature on the certificate, thus gain confidence in the certificate contents.

# Kafka Docker images compatibility

Say you have code which utilised [Kafka library](https://mvnrepository.com/artifact/org.apache.kafka/kafka-clients) `org.apache.kafka:kafka-clients` to communicate with Kafka. If you’d look into maven repository versions, you’ll see that it contains versions like `3.3.1` or `3.3.0`.

However, if you visit [Confluent’s Dockerhub Kafka images'](https://hub.docker.com/r/confluentinc/cp-kafka/) versions, you won’t find equivalents.

The [chart from Confluent](https://docs.confluent.io/platform/current/installation/versions-interoperability.html#cp-and-apache-ak-compatibility) helps to map Apache Kafka versions with Docker image versions.

# Plaintext Kafka

Firstly, we have to validate that current setup without SSL or/and SASL works. To validate, use bellow configuration.

```yaml

version: '3'
services:

  zookeeper:
    image: zookeeper:3.6
    ports:
      - "2181:2181"

  kafka:
    image: confluentinc/cp-kafka:6.0.9
    ports:
      - "29092:29092"
      - "9092:9092"
    depends_on:
      - zookeeper
    environment:
      KAFKA_LISTENERS: PLAINTEXT://kafka:9092,PLAINTEXT_HOST://0.0.0.0:29092
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092,PLAINTEXT_HOST://localhost:29092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'false'
      KAFKA_DELETE_TOPIC_ENABLE: 'true'
      KAFKA_BROKER_ID: 0
      # Otherwise you'll get Number of alive brokers '1' does not meet the required replication factor '3' for the offsets topic
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

Next, create a simple KafkaAdmin, KafkaProducer, KafkaConsumer and integrating test class.

## Common Kafka Properties Settings

```java
public enum TestCommonKafkaProperties {

	BOOTSTRAP_SERVERS_CONFIG("localhost:29092");

	private final String value;

	TestCommonKafkaProperties(String value) {
		this.value = value;
	}

	public String value() {
		return value;
	}
}
```

## Admin

```java
public class Admin {

   private final AdminClient adminClient;

   public Admin() {
      	adminClient = AdminClient.create(Map.of(
				CommonClientConfigs.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS_CONFIG.value()
		));
   }

   public void deleteTopic(String topic) throws ExecutionException, InterruptedException {
      adminClient.deleteTopics(List.of(topic)).all().get();
   }

   public void createTopic(String topic) throws ExecutionException, InterruptedException {
      try {
         adminClient.createTopics(List.of(new NewTopic(topic, 3, (short) 1))).all().get();
      } catch (Exception e) {
         if (e.getCause() instanceof TopicExistsException) {
            return;
         }
         throw e;
      }
   }
}
```

## Producer

```java
public class PlainProducer {

   private final KafkaProducer<String, String> kafkaProducer;
   private final String topic;

   public PlainProducer(Map<String, Object> properties, String topic) {

      this.kafkaProducer = new KafkaProducer<>(properties);
      this.topic = topic;
   }

   public void produce(String message) {

      try {
         kafkaProducer.send(new ProducerRecord<>(topic, message)).get();
      } catch (Exception e) {
         e.printStackTrace();
      }
   }
}
```

## Consumer


```java
public class PlainConsumer {

   private final KafkaConsumer<String, String> consumer;

   public PlainConsumer(Map<String, Object> properties, String topic) {
      this.consumer = new KafkaConsumer<>(properties);
      consumer.subscribe(List.of(topic));
   }

   public void consume() {

      try {
         while (true) {
            final var records = consumer.poll(Duration.ofMillis(100));
            for (final var record : records) {
               System.out.println("*".repeat(10));
               System.out.printf("topic = %s, partition = %d, offset = %d, customer = %s, message = %s\n",
                     record.topic(), record.partition(), record.offset(),
                     record.key(), record.value());
               System.out.println("*".repeat(10));
            }
            consumer.commitSync();
         }
      } catch (WakeupException e) {
         //nothing      
      } finally {
         consumer.close(Duration.ofMillis(500));
         System.out.println("-".repeat(10));
         System.out.println("Closing consumer");
         System.out.println("-".repeat(10));
      }
   }

   public void stop() {
      consumer.wakeup();
   }
}
```

## Integrating Test Class

```java
public class TestMe {

	private static final ExecutorService executor = Executors.newSingleThreadExecutor();

	public static void main(String[] args) throws ExecutionException, InterruptedException {

		final var topic = "hello.world";

		final var admin = new Admin();
		admin.createTopic(topic);

		Map<String, Object> producerConfiguration = Map.of(
				CommonClientConfigs.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS_CONFIG.value(),
				ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer",
				ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer"
		);

		Map<String, Object> consumerConfiguration = Map.of(
				CommonClientConfigs.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS_CONFIG.value(),
				ConsumerConfig.GROUP_ID_CONFIG, "groupsecurity",
				ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer",
				ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer"
		);

		final var plainConsumer = new PlainConsumer(consumerConfiguration, topic);
		executor.submit(plainConsumer::consume);

		TimeUnit.SECONDS.sleep(5);

		final var plainProducer = new PlainProducer(producerConfiguration, topic);
		plainProducer.produce("hello");

		TimeUnit.SECONDS.sleep(5);
		plainConsumer.stop();
		TimeUnit.SECONDS.sleep(5);
		System.out.println("Deleting topic");
		admin.deleteTopic(topic);
		TimeUnit.SECONDS.sleep(2);

		executor.shutdownNow();
		executor.awaitTermination(1, TimeUnit.SECONDS);
	}
}
```

```shell
docker-compose up -d
```

Print logs of kafka container:

```shell
docker logs <kafka-container>
```

You should see:

```shell
started (kafka.server.KafkaServer)
```

```
**********
topic = hello.world, partition = 1, offset = 0, customer = null, message = hello
**********
----------
Closing consumer
----------
Deleting topic
```





























































