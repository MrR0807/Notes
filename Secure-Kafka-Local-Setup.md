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

# Java Authentication and Authorization Service (JAAS) explained

In sections SASL PLAIN Kafka and SASL SCRAM-SHA-512 Kafka I’ll be using kafka_server_jaas.conf files. To better understand why they are needed and what is happening behind the scenes, in this section, I’ll explain JAAS framework with practical examples. I’ll balance between overloading with theoretical information and practical usage. Furthermore, this chapter’s emphasis is on Authentication rather than Authorization. For more detailed explanation, there is a great book - [The Java Authentication and Authorization Service (JaaS) in Action](https://leanpub.com/jaas) as well as Oracle’s official documentation - [JAAS Authentication](https://docs.oracle.com/en/java/javase/17/security/jaas-authentication.html#GUID-0C6EB04B-D203-4688-A3E2-A7D442334623) and [JAAS Authorization](https://docs.oracle.com/en/java/javase/17/security/jaas-authorization.html#GUID-69241059-CCD0-49F6-838F-DDC752F9F19F).

## What is JAAS

JAAS can be used for two purposes:
* for **authentication** of users, to reliably and securely determine who is currently executing Java code, regardless of whether the code is running as an application, an applet, a bean, or a servlet;
* for **authorization** of users to ensure they have the access control rights (permissions) required to do the actions performed.

As such, JAAS is not concerned with other aspects of the Java security framework, such as encryption, digital signatures, or secure connections.

JAAS is a standard part of the Java security framework since version 1.4 and was available as an optional package in J2SE 1.3. Like most Java APIs, **JAAS is exceptionally extensible**. Most of the sub-systems in the framework allow substitution of default implementations so that almost any situation can be handled. For example, an application that once stored user ids and passwords in the database can be changed to use Windows OS credentials. But because of extensibility, complexity creeps in.

## Authetincation

There are several classes/interfaces involved in the authentication process:
* javax.security.auth.login.LoginContext
* javax.security.auth.login.Configuration
* javax.security.auth.spi.LoginModule
* java.security.Principal
* java.security.auth.Subject

In most cases, a `Subject` can be thought of as a "user," but put more broadly, a `Subject` is any entity that Java code is being executed on behalf of. That is, a `Subject` need not always be a person who's, for example, logged into a system with their username and password.

When JAAS authenticates a `Subject` it first verifies the user's claims of identity by checking their credentials. If these credentials are successfully verified, the authentication framework associates the credentials, as needed, with the `Subject`, and then adds `Principals` to a `Subject`. The `Principals` represent any sort of identity the `Subject` has in the system, whether that identity is an "individual identity," such as an employee number, or a "group identity," such as belonging to a certain user group.

To perform the above functions, the `LoginContext` must be configured to use plug-in implementations of `LoginModules`, usually provided by you.

Lastly, `Configuration` object is responsible for specifying which `LoginModules` should be used for a particular application, and in what order the `LoginModules` should be invoked.

Everything will be more clear once you read examples.

### More on `Principals`

JAAS doesn’t directly associate a user’s identities with a `Subject`. Instead, each `Subject` holds onto any number of `Principals`. In the simplest sense, a `Principal` is an identity. Thus, a `Subject` can be thought of as a container for all of `Subject's` identities, similar to how your wallet contains all of your id cards: driver's license, social security, insurance card, or pet store club card.

In addition to `Principals` representing different identities of a `Subject`, they can also represent different roles the `Subject` is authorized to perform. For example, there could be `Principals` called User Administrator, System Administrator. As with identities, rather than directly associate each role’s abilities, or permissions, with a `Subject`, JAAS associates the role's abilities with `Principals`. For example `Subject`, then, would have `Principals` that represent both of these Administrator roles. 

## Example One

### JAAS config file

```
Jaashands {
  lt.test.testjaas.TestLoginModule required;
};
```

The full class name defined here points to `TestLoginModule`. If this is invalid, then when launching application you’ll get `No LoginModule found for <incorrectly defined module full class name>`.

`Jaashands` is just a name. It can be anything.

The file structure has to adhere to `javax.security.auth.login.Configuration` expected pattern as defined in [Javadocs](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/javax/security/auth/login/Configuration.html). The standard is:

```
Name {
      ModuleClass  Flag    ModuleOptions;
      ModuleClass  Flag    ModuleOptions;
      ModuleClass  Flag    ModuleOptions;
};
Name {
      ModuleClass  Flag    ModuleOptions;
      ModuleClass  Flag    ModuleOptions;
};
other {
      ModuleClass  Flag    ModuleOptions;
      ModuleClass  Flag    ModuleOptions;
};
```

Where `Flag` value can be one of the following:
* Required;
* Requisite;
* Sufficient;
* Optional.

While `ModuleOptions` is a space separated list of `LoginModule` - specific values which are passed directly to the underlying `LoginModules`. More information can be found in named [Javadocs](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/javax/security/auth/login/Configuration.html).

### Main test class

```java
public class JaasPlaysJazz {

	public static void main(String[] args) throws LoginException {

		final var loginContext = new LoginContext("Jaashands", new CallbackHandler() {
			@Override
			public void handle(Callback[] callbacks) throws IOException, UnsupportedCallbackException {
				for (Callback callback : callbacks) {
					if (callback instanceof TestCallback tc) {
						tc.setUsername("john");
						tc.setPassword("jonhspassword");
					} else {
						throw new UnsupportedCallbackException(callback, "Unrecognized Callback");
					}
				}
			}
		});

		loginContext.login();
		final var subject = loginContext.getSubject();
		final var principals = subject.getPrincipals();
		System.out.println("Size of principals: " + principals.size());
	}
}
```

I have not used Lambdas here, just to showcase the full method signature `handle(Callback[] callbacks)`.

The first part is initialization of `LoginContext`:

```java
final var loginContext = new LoginContext("Jaashands", new CallbackHandler() {}
```

Underneath, `LoginContext` is just looking for all possible configurations with name `Jaashands`. The configuration for our `LoginModule` was described in JAAS config file previously (remember the file’s content started with `Jaashands {...}`. To make it even more apparent, we can create such entry programmatically (then JAAS config file is not required):

```java
final var config = new Configuration() {

   @Override   
   public AppConfigurationEntry[] getAppConfigurationEntry(String name) {
      return new AppConfigurationEntry[]{
            new AppConfigurationEntry(TestLoginModule.class.getName(),
                  AppConfigurationEntry.LoginModuleControlFlag.REQUIRED,
                  Map.of())
      };
   }
};
Configuration.setConfiguration(config);
```

The `AppConfigurationEntry` defines exactly the same things as was defined in the config file:
* Full path to custom `LoginModule`;
* Flag `REQUIRED`;
* And `ModuleOptions` which in our current case is just an empty map.

The only thing missing is searching by provided `name`. No matter what `name` will be provided, we will be returning the same `AppConfigurationEntry`. This can be fixed easily, by creating a `HashMap`, which would contain more than one entry, but the point is simple - JAAS config file defines  `AppConfigurationEntry` which are later on used by `LoginContext` to find required `LoginModule`. This pattern makes JAAS “pluggable”. Anyone can write `LoginModule` and define via `LoginContext` when to use it.

#### CallbackHandler

The `CallbackHandler` is given the responsibility of gathering credentials for the `Subject` during authentication. A `CallbackHandler` is always associated with a `LoginContext` instance, and passed to the `LoginModules` that `LoginContext` controls. For example, a console-centric `CallbackHandler` may use "Username" and "Password" prompts to gather those two credentials; a Web application might promt a signin window for username and password.

**In summary, all a `CallbackHandler` does is provide `LoginModules` with credentials when asked by the `LoginModules`.**

The `CallbackHandler` interface contains just one method `handle(Callback[] callbacks)`. The `Callback` interface itself has no methods. It’s just a marker interface. 

In this example case, I have created a special `Callback` which doesn’t really do anything, but is a simple data carrier class (in next example, I’ll use Java’s special callbacks):

```java
public class TestCallback implements Callback {

   private String username;
   private String password;

   public TestCallback() {
   }

   public String getUsername() {
      return username;
   }

   public void setUsername(String username) {
      this.username = username;
   }

   public String getPassword() {
      return password;
   }

   public void setPassword(String password) {
      this.password = password;
   }
}
```












































