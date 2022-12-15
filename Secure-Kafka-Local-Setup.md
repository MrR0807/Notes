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

```java
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
```

And in this case, when `CallbackHandler` is invoked, and it is of our type `TestCallback`, I just set the username and password to specific values. Why there is a loop will be addressed in follow sections.

##### Summary

We have established several things from main test class called JaasPlaysJazz:
* When application starts it initialises `LoginContext` which underneath searches for `AppConfigurationEntry` entries (there can be multiple authentication `LoginModules`) for speficied name. In this case `Jaashands` which was defined in JAAS file;
* `CallbackHandler` sole purpose is to provide `LoginModule` with credentials. Depending on application, it can do it numerious ways. In this example I’ve just hardcode those credentials as if they were provided by the user;
* `loginContext.login()` call starts the authentication logic which is covered in next section;
* After successful `loginContext.login()`, I can access Subject which contains Principal entries.

#### `LoginModule`

Implementations of `LoginModules` provide the core of JAAS authentication. Though `LoginContext` provides the client interface for JAAS authentication, `LoginContext` acts as a controller, delegating the majority of the authentication work and decisions to the list of `LoginModules` configured.

The job of a `LoginModule` is simple: use credentials to verify a Subject's identity and then associate the appropriate `Principals` and credentials with that `Subject`.

##### LoginModule Life-Cycle

Below is an activity diagram of a `javax.security.auth.spi.LoginModule` implementations lifecycle:

First, the `LoginModule` implementation's default constructor is used to create a new instance of the `LoginModule`. Because the default, no argument constructor is used, another method is used to pass in the objects the `LoginModule` will use. This is done with the `initialize()` method, which takes the `Subject` to be authenticated, the `Callbackhandler` to use to gather credentials, a `Map` used as a session shared by all the `LoginModules` in use, and a `Map` of configuration options specified by the `Configuration`.

When the LoginContext executes a LoginModule's login() method, the LoginModule does whatever is needed to authenticate the Subject. This is the first phase of the two-phase process. If the login attempt succeeds, true to returned; if it failed, a javax.security.auth.LoginException, or one of it's sub-classes, is thrown; if the LoginModule should be ignored [for what reason?], false is returned. Each LoginModule stores whether is succeeded or not as private state, accessible by the other methods during the authorization lifecycle.

Notice that Principals and Credentials are not yet added to the Subject in the login() method.

Once all the LoginModules required to be successful have succeeded, the LoginContext controller calls the commit() method on each LoginModule. In the commit() method, the LoginModule will access the private success state. If authentication succeeded, the commit() method adds Principals and Credentials to the Subject and does any cleanup needed; if unsuccessful, just clean-up is done.

If login wasn’t successful, the LoginModule's abort() method is called instead of commit(). Execution of the abort() method signals that the LoginModules should cleanup any state kept, and assure that no Principals or Credentials are added to the Subject.

The full TestLoginModule class:

```java
public class TestLoginModule implements LoginModule {

	private Subject subject;
	private CallbackHandler callbackHandler;

	// the authentication status
	private boolean succeeded = false;
	private boolean commitSucceeded = false;

	// username and password
	private String username;
	private String password;

	private TestPrincipal principal;


	@Override
	public void initialize(Subject subject, CallbackHandler callbackHandler, Map<String, ?> sharedState, 
						   Map<String, ?> options) {

		this.subject = subject;
		this.callbackHandler = callbackHandler;
	}

	@Override
	public boolean login() throws LoginException {

		if (callbackHandler == null) {
			throw new LoginException("No CallbackHandler available to garner authentication 
										information from the user");
		}

		final var testCallback = new TestCallback();
		var callbacks = new Callback[]{testCallback};

		try {
			callbackHandler.handle(callbacks);
			this.username = testCallback.getUsername();
			this.password = testCallback.getPassword();
		} catch (IOException | UnsupportedCallbackException e) {
			throw new LoginException("Error during login");
		}


		//verify credentials
		if (this.username.equals("john") && this.password.equals("jonhspassword")) {
			this.succeeded = true;
			System.out.println("Authenticated successfully");
			return true;
		} else {
			this.username = null;
			this.password = null;
			this.succeeded = false;
			throw new FailedLoginException("Incorrect credentials");
		}
	}

	@Override
	public boolean commit() {
		if (succeeded == false) {
			return false;
		} else {
			principal = new TestPrincipal(username);
			if (!subject.getPrincipals().contains(principal)) {
				subject.getPrincipals().add(principal);
				System.out.println("Principal added");
			}

			username = null;
			password = null;
			commitSucceeded = true;
			return true;
		}
	}

	@Override
	public boolean abort() throws LoginException {
		if (succeeded == false) {
			return false;
		} else if (succeeded == true && commitSucceeded == false) {
			this.succeeded = false;
			this.username = null;
			this.password = null;
			this.principal = null;
		} else {
			logout();
		}

		System.out.println("Abort");

		return true;
	}

	@Override
	public boolean logout() {
		subject.getPrincipals().remove(principal);
		this.succeeded = false;
		this.commitSucceeded = false;
		this.username = null;
		this.password = null;
		this.principal = null;

		System.out.println("Logout");

		return true;
	}
}
```

#### Connecting `TestLoginModule` with `JaasPlaysJazz`

In this section I will go step by step and show how everything interacts.

First step in `JaasPlaysJazz`.

```java
final var loginContext = new LoginContext("Jaashands", new CallbackHandler() {})
```

As already established, `LoginContext` searches for all possible implementations of `LoginModule` with name `Jaashands`. Now regarding `new CallbackHandler()`, this handler is passed to `TestLoginModule` `initialize` method: 
```java
public void initialize(Subject subject, CallbackHandler callbackHandler, Map<String, ?> sharedState, 
						   Map<String, ?> options) {

		this.subject = subject;
		this.callbackHandler = callbackHandler;
	}
```

When in `JaasPlaysJazz` `loginContext.login();` is called, the full `LoginModule` lifecycle starts. Firstly, the `login()` method is called:

```java
public boolean login() throws LoginException {

		if (callbackHandler == null) {
			throw new LoginException("No CallbackHandler available to garner authentication 
										information from the user");
		}

		final var testCallback = new TestCallback();
		var callbacks = new Callback[]{testCallback};

		try {
			callbackHandler.handle(callbacks);
			this.username = testCallback.getUsername();
			this.password = testCallback.getPassword();
		} catch (IOException | UnsupportedCallbackException e) {
			throw new LoginException("Error during login");
		}


		//verify credentials
		if (this.username.equals("john") && this.password.equals("jonhspassword")) {
			this.succeeded = true;
			System.out.println("Authenticated successfully");
			return true;
		} else {
			this.username = null;
			this.password = null;
			this.succeeded = false;
			throw new FailedLoginException("Incorrect credentials");
		}
	}
```

In this method, I initialise the data carrier `TestCallback`, then I call `callbackHandler` which was initialised in `initialize` method. `CallbackHandler` is defined in `JaasPlaysJazz`. I could essentially exchange these lines:

```java
callbackHandler.handle(callbacks);
this.username = testCallback.getUsername();
this.password = testCallback.getPassword();
```

```java
for (Callback callback : callbacks) {
	if (callback instanceof TestCallback tc) {
		tc.setUsername("john");
		tc.setPassword("jonhspassword");
	} else {
		throw new UnsupportedCallbackException(callback, "Unrecognized Callback");
	}
}
this.username = testCallback.getUsername();
this.password = testCallback.getPassword();
```

Once the `username` and `password` is provided by `callbackHandler`, I can authenticate whether the user is correct. In this section, if this was a web application, I could search for username in the database and validate password against the entry in database. However, in this example I’m just hardcoding validation with `if (this.username.equals("john") && this.password.equals("jonhspassword"))`.

Next step is `commit()` which gets executed as of two-phase process already explained. The key thing to address is that only if previous step was successful, `commit()` should be executed:

```java
public boolean commit() {
	if (succeeded == false) {
		return false;
	} else {
		principal = new TestPrincipal(username);
		if (!subject.getPrincipals().contains(principal)) {
			subject.getPrincipals().add(principal);
			System.out.println("Principal added");
		}

		username = null;
		password = null;
		commitSucceeded = true;
		return true;
	}
}
```

`TestPrincipal` is a simple data class:

```java
public record TestPrincipal(String name) implements Principal {

   @Override   
   public String getName() {
      return name;
   }
}
```

The remaining actions is to add `Principal` to `Subject`. What you add in this method to `Subject`, is later on accessible in `JaasPlaysJazz` class:

```java
final var subject = loginContext.getSubject();
final var principals = subject.getPrincipals();
System.out.println("Size of principals: " + principals.size());
```

The two remaining methods were explained and there is no need to deep dive into them.

#### Running the example

As stated, there are several ways how to provide LoginModule information. The main and recommended way is via JAAS config file. Those config files have to be defined via VM variable when launching application as so:

```shell
-Djava.security.auth.login.config=security/test_jaas.conf
```

Or `java.security.auth.login.config` can be set programmatically:

```java
System.setProperty("java.security.auth.login.config", "security/test_jaas.conf");
```

Or as already defined, via initialisation of `Configuration`:

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

## Example Two

In this example I’ll skip explaining concepts which were already discussed or showcasing the same code. If there are no examples of the code, that means it is completely the same as from previous example.

### JAAS config file

```
JaashandsExampleTwo {
  lt.test.testjaas.TestLoginModule required debug="yesplease";
};
```

As you can see, I’ve added optional `ModuleOptions` to showcase how they are provided to `LoginModule`.

### Main test class

```java
final var loginContext = new LoginContext("JaashandsExampleTwo", callbacks -> {
   for (Callback callback : callbacks) {
      // prompt the user for a username      
      if (callback instanceof NameCallback nameCallback) {
         System.out.print(nameCallback.getPrompt());
         nameCallback.setName((new BufferedReader(new InputStreamReader(System.in))).readLine());
      }
      // prompt the user for sensitive information      
      else if (callback instanceof PasswordCallback passwordCallback) {
         System.out.print(passwordCallback.getPrompt());
         passwordCallback.setPassword((new BufferedReader(new InputStreamReader(System.in))).readLine().toCharArray());
      } else {
         throw new UnsupportedCallbackException(callback, "Unrecognized Callback");
      }
   }
});
```

The `CallbackHandler` now contains two different `Callback`. Both of them are from Java library:
* `javax.security.auth.callback.NameCallback`;
* `javax.security.auth.callback.PasswordCallback`.

The classes are farely trivial (they were cleaned up from javadoc and unnecessary properties):

```java
public class NameCallback implements Callback {

    private String prompt;
    private String inputName;

    public NameCallback(String prompt) {
        if (prompt == null || prompt.isEmpty())
            throw new IllegalArgumentException();
        this.prompt = prompt;
    }

    public String getPrompt() {
        return prompt;
    }

    public void setName(String name) {
        this.inputName = name;
    }

    public String getName() {
        return inputName;
    }
}
```

```java
public class PasswordCallback implements Callback {

    private String prompt;
    private boolean echoOn;
    private char[] inputPassword;

    public PasswordCallback(String prompt, boolean echoOn) {
        if (prompt == null || prompt.isEmpty())
            throw new IllegalArgumentException();

        this.prompt = prompt;
        this.echoOn = echoOn;
    }

    public String getPrompt() {
        return prompt;
    }

    public boolean isEchoOn() {
        return echoOn;
    }

    public void setPassword(char[] password) {
        this.inputPassword = (password == null ? null : password.clone());
    }

    public char[] getPassword() {
        return (inputPassword == null ? null : inputPassword.clone());
    }

    public void clearPassword() {
        if (inputPassword != null) {
            for (int i = 0; i < inputPassword.length; i++)
                inputPassword[i] = ' ';
        }
    }
}
```

They are essentially the same as what `TestCallback`, but just split into two classes. Also, instead of hardcoding these values, we’re asking for user’s input via `new BufferedReader(new InputStreamReader(System.in))).readLine()`. When lauching the application you will be prompted with two inputs.

### LoginModule

There are two difference from previous example:
* Parsing `debug="yesplease"` from JAAS config;
* Using `NameCallback` and `PasswordCallback` in `login()`.

```java
@Override
public void initialize(Subject subject, CallbackHandler callbackHandler, Map<String, ?> sharedState, 
                                Map<String, ?> options) {

   this.subject = subject;
   this.callbackHandler = callbackHandler;
   this.debug = Optional.ofNullable(options.get("debug"))
         .map(shouldDebug -> shouldDebug.equals("yesplease"))
         .orElse(false);
}
```

```java
@Override
public boolean login() throws LoginException {

   final var callbacks = new Callback[2];
   callbacks[0] = new NameCallback("username: ");
   callbacks[1] = new PasswordCallback("password: ", false);

   try {
      callbackHandler.handle(callbacks);
      this.username = ((NameCallback)callbacks[0]).getName();
      //This is not a secure way of storing password      
      this.password = new String(((PasswordCallback) callbacks[1]).getPassword());
   } catch (IOException | UnsupportedCallbackException e) {
      throw new LoginException("Error during login");
   }

   if (debug) {
      System.out.println("I'm helping");
   }
   ...
}
```

You can later on use `debug` flag however you see fit.

## Example Three

As previously stated, `LoginModule` is **plugable**. Plugable means that the same `LoginModule` implementation can potentially be re-used in different applications, and added to an application without having to recompile code. For example, a `LoginModule` that authenticates users for in Windows Domains or Active Directory, could be provided by a third party for use in other applications.

`LoginModule` are also **stackable**. Stackable means that you can use more than one `LoginModule` because your application may have more than one identity management system. For example, your application could be an employee records system that needs to authenticate with the HR system to access insurance records and the payroll system to access salary records. Each of these two systems could require a user to login, contributing `Principals` to the `Subject`.

In this example, we’ll explore the stackable property by using two `LoginModules`.

### JAAS config file

```
JaashandsExampleThree {
  lt.test.testjaas.TestLoginModule required debug="yesplease";
  lt.test.testjaas.TestLoginModuleTwo required;
};
```

In this case, both `TestLoginModule` and `TestLoginModuleTwo` will be invoked during authentication process.

### Main test class

This is exactly the same as from Example one, just the LoginContext name is different: `new LoginContext("JaashandsExampleThree", new CallbackHandler() {}`.

#### `TestLoginModuleTwo`

```java
public class TestLoginModuleTwo implements LoginModule {

	@Override
	public void initialize(Subject subject, CallbackHandler callbackHandler, Map<String, ?> sharedState,
						   Map<String, ?> options) {
	}

	@Override
	public boolean login() throws LoginException {
		System.out.println("Login in TestLoginModuleTwo");
		return true;
	}

	@Override
	public boolean commit() throws LoginException {
		System.out.println("Commit in TestLoginModuleTwo");
		return true;
	}

	@Override
	public boolean abort() throws LoginException {
		System.out.println("Abort in TestLoginModuleTwo");
		return true;
	}

	@Override
	public boolean logout() throws LoginException {
		System.out.println("Logout in TestLoginModuleTwo");
		return true;
	}
}
```

As you can see it just returns `true` for all methods and logs which method was called in this class. Same logging approached was used in  `TestLoginModule`.

Running the application yields this:

```shell
Login in TestLoginModule
Login in TestLoginModuleTwo
Commit in TestLoginModule
Commit in TestLoginModuleTwo
```

This has demonstrated the stackable property of `LoginModule`. Both modules were invoked. And only after they both succeeded, the commit phase started.

#### `Configuration Flag`

As stated previously there are four different JAAS config flags, e.g. `required`. Each of them can influence the flow of authentication, for example `sufficient` flag states: 

> The LoginModule is not required to succeed. If it does succeed, control immediately returns to the application (authentication does not proceed down the LoginModule list). If it fails, authentication continues down the LoginModule list.

More on flags in [Javadoc](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/javax/security/auth/login/Configuration.html). Changing the JAAS config file to:

```
JaashandsExampleThree {
  lt.test.testjaas.TestLoginModuleTwo sufficient;
  lt.test.testjaas.TestLoginModule required debug="yesplease";
};
```

And running the application will yield (as expected):

```
Login in TestLoginModuleTwo
Commit in TestLoginModuleTwo
```

Because it’s only required that `TestLoginModuleTwo` succeed. If it does, the others are not invoked.

## Authorization

This section is not required, because since Java 17, JAAS authorization mechanism is marked for removal: [JEP 411: Deprecate the Security Manager for Removal](https://openjdk.org/jeps/411).

### The SecurityManager and AccessController

As stated in JEP 411: Deprecate the Security Manager for Removal removes `SecurityManager` and doesn’t really talk about JAAS. How it’s connected then? Well, the pre-JAAS security model employed the concept of a security manager, a singleton of `java.lang.SecurityManager` through which all the types of permission were expressed and checked. This class is still used in current versions of Java as the preferred entry point for security checks, however, most of the methods just delegate to `AccessController` which was delivered as part of rewrite of `SecurityManager` by introducing JAAS. 

Having this in mind, we can find in the JEP that:

> java.security.{AccessController, AccessControlContext, AccessControlException, DomainCombiner} — The primary APIs for the access controller, which is the default implementation to which the Security Manager delegates permission checks. These APIs do not have value without the Security Manager, since certain operations will not work without both a policy implementation and access-control context support in the VM.

Indeed those are removed, hence no point in exploring them.

### Ok, but why did they exist in the first place?

JAAS was created during Java applet times. It was paramount to have a security framework which would control access to local file system at very granual level. One of requirements was to deny access to the local file system to, for example, prevent applets from installing spyware or adware on your machine, or installing viruses. This in turn changed the whole concept of on who’s behalf Java code is executing. Hence a lot of operations were required to be check before executing.

I’m not going to deep dive into how it works, because there are examples in [Oracle documetation](https://docs.oracle.com/en/java/javase/17/security/jaas-authorization.html#GUID-69241059-CCD0-49F6-838F-DDC752F9F19F), but just to showcase the principal:

```java
 package chp02;

import java.io.File;
import java.io.IOException;

public class Chp02aMain {
	public static void main(String[] args) throws IOException {
		File file = new File("build/conf/cheese.txt");
		try {
			file.canWrite();
			System.out.println("We can write to cheese.txt");
		} catch (SecurityException e) {
			System.out.println("We can NOT write to cheese.txt");
		}
	}
}
```

`file.canWrite()`:

```java
public boolean canWrite() {
    @SuppressWarnings("removal")
    SecurityManager security = System.getSecurityManager();
    if (security != null) {
        security.checkWrite(path);
    }
    if (isInvalid()) {
        return false;
    }
    return fs.checkAccess(this, FileSystem.ACCESS_WRITE);
}
```

As you can see, Java has a check with SecurityManager. And if’d search JDK, such checks are everywhere.

## Kafka

In SASL PLAIN Kafka and SASL SCRAM-SHA-512 Kafka there will be two different `kafka_server_jaas.conf` used. For plain:

```
KafkaServer {
  org.apache.kafka.common.security.plain.PlainLoginModule required
  username="admin"
  password="admin-secret"
  user_admin="admin-secret";
};
```

And for SCRAM:

```
KafkaServer {
  org.apache.kafka.common.security.scram.ScramLoginModule required
  username="admin"
  password="admin-secret"
  user_admin="admin-secret";
};
```

Looking at these we can conclude, where to look for `LoginModule` implementations and see that all parameters after `required` are options that will be provided as `Map` key/value pairs just like with example two `debug="yesplease"`. In next section I’ll just inspect how `PlainLoginModule` works.

### PlainLoginModule

```java
public class PlainLoginModule implements LoginModule {

    private static final String USERNAME_CONFIG = "username";
    private static final String PASSWORD_CONFIG = "password";

    static {
        PlainSaslServerProvider.initialize();
    }

    @Override
    public void initialize(Subject subject, CallbackHandler callbackHandler, Map<String, ?> sharedState, 
                           Map<String, ?> options) {
        String username = (String) options.get(USERNAME_CONFIG);
        if (username != null)
            subject.getPublicCredentials().add(username);
        String password = (String) options.get(PASSWORD_CONFIG);
        if (password != null)
            subject.getPrivateCredentials().add(password);
    }

    @Override
    public boolean login() {
        return true;
    }

    @Override
    public boolean logout() {
        return true;
    }

    @Override
    public boolean commit() {
        return true;
    }

    @Override
    public boolean abort() {
        return false;
    }
}
```

From this we can conclude, that during the first authentication phase, `Subject` is populated with credentials which are taken from JAAS config file under `username` and `password`. The remaining phases are not used. Looking into `CallbackHandler`:

```java
public class PlainServerCallbackHandler implements AuthenticateCallbackHandler {

    private static final String JAAS_USER_PREFIX = "user_";
    private List<AppConfigurationEntry> jaasConfigEntries;

    @Override
    public void configure(Map<String, ?> configs, String mechanism, List<AppConfigurationEntry> jaasConfigEntries) {
        this.jaasConfigEntries = jaasConfigEntries;
    }

    @Override
    public void handle(Callback[] callbacks) throws IOException, UnsupportedCallbackException {
        String username = null;
        for (Callback callback: callbacks) {
            if (callback instanceof NameCallback)
                username = ((NameCallback) callback).getDefaultName();
            else if (callback instanceof PlainAuthenticateCallback) {
                PlainAuthenticateCallback plainCallback = (PlainAuthenticateCallback) callback;
                boolean authenticated = authenticate(username, plainCallback.password());
                plainCallback.authenticated(authenticated);
            } else
                throw new UnsupportedCallbackException(callback);
        }
    }

    protected boolean authenticate(String username, char[] password) throws IOException {
        if (username == null)
            return false;
        else {
            String expectedPassword = JaasContext.configEntryOption(jaasConfigEntries,
                    JAAS_USER_PREFIX + username,
                    PlainLoginModule.class.getName());
            return expectedPassword != null && Utils.isEqualConstantTime(password, expectedPassword.toCharArray());
        }
    }
}
```

`JaasContext`:

```java
public static String configEntryOption(List<AppConfigurationEntry> configurationEntries, String key, String loginModuleName) {
    for (AppConfigurationEntry entry : configurationEntries) {
        if (loginModuleName != null && !loginModuleName.equals(entry.getLoginModuleName()))
            continue;
        Object val = entry.getOptions().get(key);
        if (val != null)
            return (String) val;
    }
    return null;
}
```

The authentication process is very simple, which is defined in `authenticate` method. It loops through a `List<AppConfigurationEntry>` and searches for `AppConfigurationEntry` where `LoginModule` is defined as `PlainLoginModule`. Once found, extracts the value of key `JAAS_USER_PREFIX + username`, which in this case is `user_admin="admin-secret"` and compares with password value.

However `PlainLoginModule` is not like our previous examples, because Kafka is utilising even more Java’s security objects like `java.security.Provider` and `javax.security.sasl.SaslServer`. Nevertheless, we can still find bits of familiar code in, for example, `org.apache.kafka.common.security.plain.internalsPlainSaslServer`:

```java
...
NameCallback nameCallback = new NameCallback("username", username);
PlainAuthenticateCallback authenticateCallback = new PlainAuthenticateCallback(password.toCharArray());
try {
    callbackHandler.handle(new Callback[]{nameCallback, authenticateCallback});
} catch (Throwable e) {
    throw new SaslAuthenticationException("Authentication failed: credentials for user could not be verified", e);
}
...
```

# SASL PLAIN Kafka

Kafka protocol supports authentication using SASL and has built-in support for several commonly used SASL mechanisms. SASL can be combined with TLS as the transport layer to provide a secure channel with authentication and encryption. SASL authentication is performed through a sequence of server challenges and client-responses where the SASL mechanism defines the sequence and wire format of challenges and responses. Kafka brokers support the following SASL mechanisms out-of-the box with customizable callbacks to integrate with existing security infrastructure.
* **GSSAPI**: Kerberos authentication is supported using SASL/GSSAPI and can be used to integrate with Kerberos servers like Active Directory or OpenLDAP.
* **PLAIN**: User name/password authentication that is typically used with a custom server-side callback to verify passwords from an external password store.
* **SCRAM-SHA-256 and SCRAM-SHA-512**: User name/password authentication available out-of-the-box with Kafka without the need for additional password stores.
* **OAUTHBEARER**: Authentication using OAuth bearer tokens that is typically used with custom callbacks to acquire and validate tokens granted by standard OAuth servers.

Kafka uses Java Authentication and Authorization Service (JAAS) for configuration of SASL. The configuration option `sasl.jaas.config` contains a single JAAS configuration entry that specifies a login module and its options.

JAAS configuration may also be specified in configuration files using the Java system property `java.security.auth.login.config`.

## Docker Compose Yaml

When it comes to docker-compose.yml there are [nuances with Kafka Docker Image](https://github.com/confluentinc/cp-docker-images/blob/fec6d0a8635cea1dd860e610ac19bd3ece8ad9f4/debian/kafka/include/etc/confluent/docker/configure). In SASL case, I have to define `KAFKA_OPTS` `java.security.auth.login.config` path otherwise container will not start. The path points to jaas config file.

Also, disabling SASL between Zookeeper and Kafka as this is not the goal of documentation and just complicates the setup. Furthermore, defining inter broker communication as PLAIN, due to ease of setup.

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
      KAFKA_LISTENERS: SASL_PLAINTEXT://0.0.0.0:29092, PLAINTEXT://kafka:9092
      KAFKA_ADVERTISED_LISTENERS: SASL_PLAINTEXT://localhost:29092, PLAINTEXT://kafka:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: SASL_PLAINTEXT:SASL_PLAINTEXT, PLAINTEXT:PLAINTEXT
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'false'
      KAFKA_DELETE_TOPIC_ENABLE: 'true'
      KAFKA_BROKER_ID: 0
      # Otherwise you'll get Number of alive brokers '1' does not meet the required replication factor '3' for the offsets topic
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1

      # Security settings
      ZOOKEEPER_SASL_ENABLED: "false"
      # inter.broker.listener.name must be a listener name defined in advertised.listeners
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_SASL_ENABLED_MECHANISMS: PLAIN

      # To add VM options to Kafka
      KAFKA_OPTS: "-Djava.security.auth.login.config=/etc/kafka/secrets/kafka_server_jaas.conf"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./security:/etc/kafka/secrets
```

In order to mount `kafka_server_jaas.conf` to Kafka container, create:

```shell
mkdir security &&
touch security/kafka_server_jaas.conf
```

The content of `kafka_server_jaas.conf`:

```
KafkaServer {
  org.apache.kafka.common.security.plain.PlainLoginModule required
  username="admin"
  password="admin-secret"
  user_admin="admin-secret";
};
Client{};
```

Each KafkaServer/Broker uses the `KafkaServer` section in the JAAS file to provide SASL configuration options for the broker, including any SASL client connections made by the broker for inter-broker communications.

`Client` section is for authenticating a SASL connection with ZooKeeper, and also to allow brokers to set a SASL ACL on ZooKeeper nodes. `Client` is empty, because in this section communication with Zookeeper is `PLAINTEXT`.

An alternative content of `kafka_server_jaas.conf` where user `john` instead of `admin`:

```
KafkaServer {
  org.apache.kafka.common.security.plain.PlainLoginModule required
  username="john"
  password="thisisjohnsecret"
  user_john="thisisjohnsecret";
};
Client{};
```

Start docker-compose:

```
docker-compose up -d
docker logs <kafka-container>
```

Console output from Kafka container:
```
SASL is enabled.
===> Running preflight checks ...
...
...
started (kafka.server.KafkaServer)
```

## Common Kafka Properties Settings

```java
public enum TestCommonKafkaProperties {

	BOOTSTRAP_SERVERS_CONFIG("localhost:29092"),
	SECURITY_PROTOCOL_CONFIG("SASL_PLAINTEXT"),
	SASL_MECHANISM("PLAIN"),
	SASL_JAAS_CONFIG("org.apache.kafka.common.security.plain.PlainLoginModule required username=\"admin\" password=\"admin-secret\";");
    //For user jonh
	//SASL_JAAS_CONFIG("org.apache.kafka.common.security.plain.PlainLoginModule required username=\"john\" password=\"thisisjohnsecret\";");
	
	

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
adminClient = AdminClient.create(Map.of(
		CommonClientConfigs.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS_CONFIG.value(),
		// Security configs
		CommonClientConfigs.SECURITY_PROTOCOL_CONFIG, SECURITY_PROTOCOL_CONFIG.value(),
		SaslConfigs.SASL_MECHANISM, SASL_MECHANISM.value(),
		SaslConfigs.SASL_JAAS_CONFIG, SASL_JAAS_CONFIG.value()
));
```


## Producer

```java
Map<String, Object> producerConfiguration = Map.of(
		CommonClientConfigs.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS_CONFIG.value(),
		ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer",
		ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer",
		// Security configs
		CommonClientConfigs.SECURITY_PROTOCOL_CONFIG, SECURITY_PROTOCOL_CONFIG.value(),
		SaslConfigs.SASL_MECHANISM, SASL_MECHANISM.value(),
		SaslConfigs.SASL_JAAS_CONFIG, SASL_JAAS_CONFIG.value()
);
```

## Consumer

```java
Map<String, Object> consumerConfiguration = Map.of(
		CommonClientConfigs.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS_CONFIG.value(),
		ConsumerConfig.GROUP_ID_CONFIG, "groupsecurity",
		ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer",
		ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer",
		// Security configs
		CommonClientConfigs.SECURITY_PROTOCOL_CONFIG, SECURITY_PROTOCOL_CONFIG.value(),
		SaslConfigs.SASL_MECHANISM, SASL_MECHANISM.value(),
		SaslConfigs.SASL_JAAS_CONFIG, SASL_JAAS_CONFIG.value()
);
```

Running TestMe class yields:

```
**********
topic = hello.world, partition = 1, offset = 0, customer = null, message = hello
**********
----------
Closing consumer
----------
Deleting topic
```

Changing SASL_JAAS_CONFIG content in TestCommonKafkaProperties would fail Authentication:

```
SaslAuthenticationException: Authentication failed: Invalid username or password
```

# SASL SCRAM-SHA-512 Kafka

## Docker Compose Yaml

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
      KAFKA_LISTENERS: SASL_PLAINTEXT://0.0.0.0:29092, PLAINTEXT://kafka:9092
      KAFKA_ADVERTISED_LISTENERS: SASL_PLAINTEXT://localhost:29092, PLAINTEXT://kafka:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: SASL_PLAINTEXT:SASL_PLAINTEXT, PLAINTEXT:PLAINTEXT
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'false'
      KAFKA_DELETE_TOPIC_ENABLE: 'true'
      KAFKA_BROKER_ID: 0
      # Otherwise you'll get Number of alive brokers '1' does not meet the required replication factor '3' for the offsets topic
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1

      # Security settings
      ZOOKEEPER_SASL_ENABLED: "false"
      # inter.broker.listener.name must be a listener name defined in advertised.listeners
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_SASL_ENABLED_MECHANISMS: SCRAM-SHA-512

      # To add VM options to Kafka
      KAFKA_OPTS: "-Djava.security.auth.login.config=/etc/kafka/secrets/kafka_server_jaas.conf"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./security:/etc/kafka/secrets
```

As with SASL PLAIN Kafka, in order to mount kafka_server_jaas.conf to Kafka container, create:

```
mkdir security &&
touch security/kafka_server_jaas.conf
```

The content of `kafka_server_jaas.conf`:

```
KafkaServer {
  org.apache.kafka.common.security.scram.ScramLoginModule required
  username="admin"
  password="admin-secret"
  user_admin="admin-secret";
};
Client{};
```

Differently from SASL PLAIN Kafka, instead of `plain.PlainLoginModule` use `scram.ScramLoginModule`.

Each KafkaServer/Broker uses the `KafkaServer` section in the JAAS file to provide SASL configuration options for the broker, including any SASL client connections made by the broker for inter-broker communications.

`Client` section is for authenticating a SASL connection with ZooKeeper, and also to allow brokers to set a SASL ACL on ZooKeeper nodes. `Client` is empty, because in this section communication with Zookeeper is `PLAINTEXT`.

Start docker-compose:

```shell
docker-compose up -d
docker logs <kafka-container>
```

Console output from Kafka container:

```shell
SASL is enabled.
===> Running preflight checks ...
...
sasl.enabled.mechanisms = [SCRAM-SHA-512]
...
started (kafka.server.KafkaServer)
```

Because SCRAM implementation in Kafka stores SCRAM credentials in ZooKeeper, we must create SCRAM credentials in ZooKeeper.

Loggin to docker container:

```shell
docker exec -it <kafka-container> /bin/bash
```

According to recommended way (below command line) adding SCRAM admin credentials to Zookeeper via Kafka should be done like so:

```shell
kafka-configs --bootstrap-server localhost:9092 --alter --add-config 'SCRAM-SHA-512=[password=admin-secret]' --entity-type users --entity-name admin
```

However, recommended way is not working with 6.0.9 and you will get:

```shell
Only quota configs can be added for 'users' using --bootstrap-server. Unexpected config names: Set(SCRAM-SHA-512)
```

Gotta use deprecated way via Zookeeper:

```shell
kafka-configs --zookeeper zookeeper:2181 --alter --add-config 'SCRAM-SHA-512=[password=admin-secret]' --entity-type users --entity-name admin
```

There might be exceptions, but the last line is the most important:

```shell
Warning: --zookeeper is deprecated and will be removed in a future version of Kafka.
Use --bootstrap-server instead to specify a broker to connect to.
WARN SASL configuration failed: javax.security.auth.login.LoginException: No JAAS configuration section named 'Client' was found in specified JAAS configuration file: '/etc/kafka/secrets/kafka_server_jaas.conf'. Will continue connection to Zookeeper server without SASL authentication, if Zookeeper server allows it. (org.apache.zookeeper.ClientCnxn)
ERROR [ZooKeeperClient] Auth failed. (kafka.zookeeper.ZooKeeperClient)
Completed updating config for entity: user-principal 'admin'.
```

### Common Kafka Properties Settings

```java
public enum TestCommonKafkaProperties {

	BOOTSTRAP_SERVERS_CONFIG("localhost:29092"),
	SECURITY_PROTOCOL_CONFIG("SASL_PLAINTEXT"),
	SASL_MECHANISM("SCRAM-SHA-512"),
	SASL_JAAS_CONFIG("org.apache.kafka.common.security.scram.ScramLoginModule required username=\"admin\" password=\"admin-secret\";");

	private final String value;

	TestCommonKafkaProperties(String value) {
		this.value = value;
	}

	public String value() {
		return value;
	}
}
```

### Admin

```java
adminClient = AdminClient.create(Map.of(
		CommonClientConfigs.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS_CONFIG.value(),
		CommonClientConfigs.SECURITY_PROTOCOL_CONFIG, SECURITY_PROTOCOL_CONFIG.value(),
		SaslConfigs.SASL_MECHANISM, SASL_MECHANISM.value(),
		SaslConfigs.SASL_JAAS_CONFIG, SASL_JAAS_CONFIG.value()
));
```

### Producer

```java
Map<String, Object> producerConfiguration = Map.of(
		CommonClientConfigs.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS_CONFIG.value(),
		ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer",
		ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer",
		// Security configs
		CommonClientConfigs.SECURITY_PROTOCOL_CONFIG, SECURITY_PROTOCOL_CONFIG.value(),
		SaslConfigs.SASL_MECHANISM, SASL_MECHANISM.value(),
		SaslConfigs.SASL_JAAS_CONFIG, SASL_JAAS_CONFIG.value()
);
```

### Consumer

```java
Map<String, Object> consumerConfiguration = Map.of(
		CommonClientConfigs.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS_CONFIG.value(),
		ConsumerConfig.GROUP_ID_CONFIG, "groupsecurity",
		ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer",
		ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer",
		// Security configs
		CommonClientConfigs.SECURITY_PROTOCOL_CONFIG, SECURITY_PROTOCOL_CONFIG.value(),
		SaslConfigs.SASL_MECHANISM, SASL_MECHANISM.value(),
		SaslConfigs.SASL_JAAS_CONFIG, SASL_JAAS_CONFIG.value()
);
```

Running TestMe class yields:

```
**********
topic = hello.world, partition = 1, offset = 0, customer = null, message = hello
**********
----------
Closing consumer
----------
Deleting topic
```

# TLS in layman's terms

There are numerous sources explaining TLS in great detail. Most of those sources are fairly technical and deep dive into details. In this section I wanted to explain the need for TLS in a simplistic manner. 

TLS solves an old problem of secret communication. One of the earlier and simpler (by no means first) cipher was called Caesar’s cipher. It works by replacing each letter in the message by a letter some fixed number of positions down the alphabet. Say we have a message: “All your base are belong to us”. If we shift alphabet by two, then the message becomes: “Cnn aqwt dcug ctg dgnqpi vq wu” (Caesar cipher online version). In this case, the cipher key is “shift alphabet by 2”. Furthermore, this cipher is called symmetrical. In other words, you use the same key for both encryption and decryption.

Now let’s imagine that this cipher is uncrackable. And let’s say you want to send an encrypted letter to someone in another city. How will you share your secret key with the letter recipient so no one else intercepts it?

Until mid-1970 all cipher systems used symmetric key algorithms and the problem of sharing cipher key was persistent and very exploitable. Enter asymmetric cryptography (public key cryptography).

The key application of asymmetric cryptography is that the message that a sender encrypts using the recipient's public key can only be decrypted by recipient's paired private key. This solves the problem of sharing symmetrical key. Continuing the example of sending a letter encrypted with Caesar’s cipher - we can encrypt the original message with symmetric key, and then encrypt the symmetric key using asymmetric cryptography. This way, only the recipient will be able to decrypt the message, by firstly decrypting symmetric key using paired private key and then decrypt original message using symmetric key.

A natural question arises - why not just use asymmetric cryptography all the time? Why use both symmetric and asymmetric cryptography? The answer is pretty simple - [speed](https://crypto.stackexchange.com/a/30778):

> With symmetric ciphers, encryption and decryption speed can be several gigabytes per seconds on a common PC core; see these [benchmarks](http://bench.cr.yp.to/results-stream.html).

> With RSA encryption (asymmetric ciphers), on comparable hardware, we are talking tens of thousands encryptions per second, and only few hundreds of decryption per seconds, for common key sizes, and small messages (like 1 bit to 250 bytes, way enough for a session keys and authenticators); see these [benchmarks](http://bench.cr.yp.to/results-encrypt.html).

> Pure asymmetric encryption would often be like 3 to 5 decimal orders of magnitude slower than symmetric encryption is.

## Pen and Paper RSA (asymmetric cipher) example

Previously I have showed you a very simple symmetrical cipher called Caesar’s cipher. Now, I’d like to show you asymmetric cipher - RSA.

RSA (Rivest–Shamir–Adleman) is a public-key algorithm that is widely used for secure data transmission. Because asymmetric cryptography is based on numbers theory, it is harder to understand how it works compared to symmetrical. Nevertheless let’s have a very simplified example. [Source of the example](https://www.youtube.com/watch?v=4zahvcJ9glg).

Our message will contain one letter: B. RSA requires an encryption pair of numbers: for this example private key is (11, 14) and public key is (5, 14). Notice the second number is the same in both pairs of numbers. The public pair of numbers is the lock we hand over to the public. Whoever wants to send us a message should use these numbers in below steps.

Encryption steps:
* Convert message containing B to number. B → 2 (alphabetical).
* Raise 2 to the power of 5 (the first number of the public key pair) and take the mod using the second number 14. The encrypted message is 4.

```math
2^5(mod14) = 32(mod14) = 4
```

Decryption steps:
* Raise 4 to the power of 11 (first number from private pair) and take the mod using the second number 14.

```math
4^11(mod14) = 4194304(mod14) = 2
```

The decrypted message is 2 -> B.

### How pairs are generated

Naturally, the next question - how private and public key pairs are generated? Here is the sequence of  actions that are required to take to find in this example used private and public key pairs (as noted earlier, asymmetric algorithms are based on number theory hence the required steps are more advanced in nature):
* Prime numbers are selected. In real world scenario, selected prime numbers are very long. However in this example, we’ll use one of firstly encountered prime numbers - 2 and 7.
* Product of 2 and 7 = 14. This number is the module in both public and private keys.
* Apply the phi function (also known as Euler's totient function) to the module part. Phi function result is calculated like so:
  * List all numbers between 1 and 14 - 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14.
  * Cross out all common factors (can divide exactly without leaving any remainder) of 2 in the list. The remaining list - 1, 3, 5, 7, 9, 11, 13.
  * Cross out all common factors of 7 in the list. The remaining list - 1, 3, 5, 9, 11, 13. These numbers are called co-prime with number 14.
  * The answer to phi function is 6 (the number of co-prime numbers listed previously - 1, 3, 5, 9, 11, 13).
* The public key (let’s call it e for encryption) has to adhere to two properties:
  * 1 < e < phi (which in our example is 6). In this case it is 2, 3, 4, 5.
  * e has to be a co-prime with phi (from step 3) and 14 (from step 2). The only common co-prime number of 6 and 14 is 5. You can use this online co-prime calculator to recheck. From this we have our public key pair: [5, 14]. Reminder, 14 was calculated from step 2.
* The private key (let’s call it d for decryption) is calculated from formula: 

```math
de(modphi(N)) = 1
```

We have e = 5 (from step 4), then we have phi (N) =  6 (from step 3), hence the function looks like so:

```math
5d(mod6) = 1
```

By just setting d incrementally, we can see how the results change:
* when d = 1, then 5 x 1 mod 6 = 5
* when d = 2, then 5 x 2 mod 6 = 4
* when d = 3, then 5 x 3 mod 6 = 3
* when d = 4, then 5 x 4 mod 6 = 2
* when d = 5, then 5 x 5 mod 6 = 1 (this could be a first part of the private key)
….
* when d = 11, then 5 x 11 mod 6 = 1 (this is our key from example)

# SSL Kafka - Without CA

There are several nuances when it comes to setting up SSL. Firstly, there are two tools for preparing Certificates (refer to Vocabulary and TLS in layman's terms for basic explanation of security concepts):
* openssl - open source tool, which among other things, enables users to create public/private keys, certificates, signed certificates etc.
* keytool - a certificate management utility included with Java. It allows to create keystore and truststore amongst other things.

Furthermore, there are four ways how SSL/TLS can be utilised:
* When a client imports exact certificate of the server to its trust store.
* When a client imports certificate authority’s (CA) certificate into its trust store, which was used to sign servers certificate.
* When both client and server exchange their respected certificates (mTLS). That means that in their respected truststores they should have each others certificates.
* Lastly, when both client and server exchange their respected certificates (mTLS), which are signed by the same CA. That way each can have the same trust store with only CA certificate in it.

In this section I will cover first scenario. Also, as previously, internal communication will be left with PLAINTEXT.

## Docker Compose Yaml

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
      KAFKA_LISTENERS: SSL://0.0.0.0:29092, PLAINTEXT://kafka:9092
      KAFKA_ADVERTISED_LISTENERS: SSL://localhost:29092, PLAINTEXT://kafka:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: SSL:SSL, PLAINTEXT:PLAINTEXT
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'false'
      KAFKA_DELETE_TOPIC_ENABLE: 'true'
      KAFKA_BROKER_ID: 0
      # Otherwise you'll get Number of alive brokers '1' does not meet the required replication 
      # factor '3' for the offsets topic
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1

      # Security settings
      KAFKA_SSL_KEYSTORE_FILENAME: kafka-broker-identity.jks
      KAFKA_SSL_KEYSTORE_CREDENTIALS: password.txt
      KAFKA_SSL_KEY_CREDENTIALS: key-password.txt
      # Turns off hostname verification
      KAFKA_SSL_ENDPOINT_IDENTIFICATION_ALGORITHM: " "
      # inter.broker.listener.name must be a listener name defined in advertised.listeners
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_SSL_CLIENT_AUTH: none
      KAFKA_SSL_KEYSTORE_TYPE: PKCS12

      # To add VM options to Kafka
      KAFKA_OPTS: "-Djavax.net.debug=ssl,handshake"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./security:/etc/kafka/secrets
```

There are several difference from SASL PLAIN Kafka or SASL SCRAM-SHA-512 Kafka:
* As in SASL PLAIN Kafka, Kafka docker image has [several hardcoded values](https://github.com/confluentinc/cp-docker-images/blob/fec6d0a8635cea1dd860e610ac19bd3ece8ad9f4/debian/kafka/include/etc/confluent/docker/configure). One of them is `KAFKA_SSL_KEYSTORE_FILENAME`. This is the location of Kafka’s keystore.
* `KAFKA_SSL_KEYSTORE_CREDENTIALS` self explanatory. What is not self explanatory is that password values have to be defined in file.
* `KAFKA_SSL_KEY_CREDENTIALS` sets KeyStore’s private key password. Same as `KAFKA_SSL_KEYSTORE_CREDENTIALS`. 
* `KAFKA_SSL_ENDPOINT_IDENTIFICATION_ALGORITHM` is blank, because for a local setup it’s tedious process to set hostname verification.
* `KAFKA_SSL_CLIENT_AUTH` defines whether mTLS is enabled or not. In other words, whether our application should send its certificate to Kafka. In this case we will not.
* `KAFKA_SSL_KEYSTORE_TYPE` defines public key standards. There are many different standards, but current and latest is PKCS12. [List of PKCS](https://en.wikipedia.org/wiki/PKCS).
* `KAFKA_OPTS: "-Djavax.net.debug=ssl,handshake"` JVM flags allows to debug SSL connection easily. With this enabled, we can inspect SSL handshakes and their status in Kafka container.
* `volume` mounts specifically to `/etc/kafka/secrets`. Again, this is from hardcoded [Kafka docker image configuration file](https://github.com/confluentinc/cp-docker-images/blob/fec6d0a8635cea1dd860e610ac19bd3ece8ad9f4/debian/kafka/include/etc/confluent/docker/configure).  For example, KeyStore has to be placed in `KAFKA_SSL_KEYSTORE_LOCATION="/etc/kafka/secrets/$KAFKA_SSL_KEYSTORE_FILENAME"`.

## Generating Certificates, Key Pairs, Keystore and Truststore

### Using OpenSSL tool

#### Generate Kafka’s key pair and certificate

Old way, which is quite common in StackOverflow answers, is using `openssl genrsa`, however in OpenSSL documentation it’s recommended to use `genpkey`: “The use of the `genpkey` program is encouraged over the algorithm specific utilities".

To generate a private/public key pair run:

```shell
openssl genpkey -out fd.key -algorithm RSA -pkeyopt rsa_keygen_bits:4096 -pass pass:secret
```

* `genpkey` - command generates a private/public key.
* `-out` - output the key to the specified file.
* `-algorithm` - OpenSSL supports RSA, DSA, ECDSA, and EdDSA key algorithms (asymmetrical), but not all of them are useful in practice. For example, DSA is obsolete and EdDSA is not yet widely supported. That leaves us with RSA and ECDSA algorithms to use in our certificates.
* `-pkeyopt rsa_keygen_bits:2048` - The default key sizes might not be secure, which is why you should always explicitly configure key size. For example, the default for RSA keys used to be 512 bits, which is insecure. If you used a 512-bit key on your server today, an intruder could take your certificate and use brute force to recover your private key, after which she could impersonate your web site. Today, 2,048-bit RSA keys are considered secure. 
* `-pass` - Using a passphrase with a key is optional, but strongly recommended. Protected keys can be safely stored, transported, and backed up. On the other hand, such keys are inconvenient, because they can’t be used without their pass phrases.

Create a self signed certificate using previously generated private/public key:

```shell
openssl req -new -x509 -days 365 -key fd.key -out kafka.crt -subj '/CN=test/OU=Unknown/O=Unknown/L=Unknown/ST=Unknown/C=US' -passin pass:secret -passout pass:secret
```

* `req` - command primarily creates and processes certificate requests in PKCS#10 format. It can additionally create self signed certificates for use as root CAs for example.
* `-new` - this option generates a new certificate request.
* `-x509` - because we’re using this option, this option outputs a self signed certificate instead of a certificate request (certificate request will be used in SSL Kafka - With CA).
* `-days` - how many days certificate is valid.
* `-key` - this specifies the file to read the private key from.
* `-out` - this specifies output file for certificate.
* `-subj` - replaces subject field of input request with specified data and outputs modified request. Otherwise you’d be prompted by command line to enter each value separately.
* `-passin` - the input file password source. In this case it’s private/public key password from previous step -pass pass:secret. 
* `-passout` the output file password source. In this case it will be applied to certificate.

Or instead of two commands it is possible to do it one single command. I will explain only the difference of commands between this step and previous one:

```shell
openssl req -new -x509 -newkey rsa:4096 -days 365 -keyout fd.key -out kafka.crt -subj '/CN=test/OU=Unknown/O=Unknown/L=Unknown/ST=Unknown/C=US' -passin pass:secret -passout pass:secret
```

* `-newkey` rsa:4096 - because I am generating both private/public key and certificate in one go, I have to define the key algorithm and key size.  
* `-keyout` - again, because key was not generated with a separate step, this gives the filename to write the newly created private key to.

#### Generate Kafka’s Java Keystore

Private keys and certificates can be stored in a variety of formats, for example:
* ASCII
* PKCS7
* PKCS8
* PKCS12

With JEP 229: Create PKCS12 Keystores by Default, all Java versions above 9, uses PKCS12 format by default. Prior to Java 9, PKCS12 are compatible, hence I need to convert the key and certificates in PEM format to PKCS12:

```shell
openssl pkcs12 -export -out keyStore.p12 -inkey fd.key -in kafka.crt -passin pass:secret -passout pass:secret
```

* `pkcs12` - command allows PKCS#12 files (sometimes referred to as PFX files) to be created and parsed.
* `-export` - this option specifies that a PKCS#12 file will be created rather than parsed.
* `-out` - this specifies filename to write the PKCS#12 file to.
* `-inkey` - file to read private key from.
* `-in` - the filename to read certificates.

Now I have a private key and a certificate in correct format. Time to create a Java KeyStore. OpenSSL is not a tool for that, because it’s a Java specific file, hence I have to use keytool.

```shell
keytool -importkeystore -srckeystore keystore.p12 -srcstorepass secret -srcstoretype pkcs12 -destkeystore kafka-broker-identity.jks -deststorepass secret -destkeypass secret -deststoretype pkcs12  
```

* `-importkeystore` - command to import a single entry or all entries from a source keystore to a destination keystore.
* `-srckeystore` - source of the keystore.
* `-srcstorepass` - source keystore password.
* `-srcstoretype` - type (as previously outlined there are many formats a private key and certificate can be stored). In this case it is PKCS12.
* `-destkeystore` - destination of keystore.
* `-deststorepass` - destination keystore password.
* `-destkeypass` - password for the key in the keystore file. This might be confusing, but in our case, there are two passwords in action - truststore’s password and password of an entry of private key it contains. It’s best to make them [both the same](https://docs.oracle.com/en/java/javase/13/docs/specs/man/keytool.html#commands-for-importing-contents-from-another-keystore:~:text=For%20example%2C%20most%20third%2Dparty%20tools%20require%20storepass%20and%20keypass%20in%20a%20PKCS%20%2312%20keystore%20to%20be%20the%20same.%20To%20create%20a%20PKCS%2312%20keystore%20for%20these%20tools%2C%20always%20specify%20a%20%2Ddestkeypass%20that%20is%20the%20same%20as%20%2Ddeststorepass.).
* `-deststoretype` - type, again is PKCS12 as explained previously.

We can inspect newly created keystore:

```shell
keytool -keystore kafka-broker-identity.jks -storepass secret -list 
```

You should see:

```shell
Keystore type: PKCS12
Keystore provider: SUN

Your keystore contains 1 entry

<date>, PrivateKeyEntry, 
Certificate fingerprint (SHA-256): <key hashcode>
```

#### Generate Kafka Client’s Java Truststore

Now it’s time to create client’s truststore. In this section’s case, it has to contain exact certificate copy of Kafka broker (server). 

```shell
keytool -importcert -file kafka.crt -alias kafkaCer -keystore client-truststore.jks -storepass truststorepass -noprompt 
```

* `-importcert` - this command is used for reading the certificate from `-file` file and storing it in the `-keystore` destination. The most confusing part about this that we’re creating truststore with `-keystore` command. However, looking from keytool’s perspective, it does not matter how we call the file - whether it’s a truststore or keystore. What matters is the content inside and how the file is used in Java applications - whether we load that file to `KeyManager` or `TrustManager` instance.
* `-file` - input file. In this case it has to be certificate.
* `-alias` - alias name for entry in TrustStore.
* `-keystore` - keystore/truststore output file.
* `-storepass` - keystore/truststore password.
* `-noprompt` - if omitted it will prompt a questions “Trust this certificate?” and require additional command line input.

### Using Keytool

As noted in the beginning of the chapter, there are two tools for generating certificates. In this section, certificates will be generated using Java `keytool`. The flow is a bit different, but the end goal is the same.

#### Generate Kafka’s Java Keystore

Because keytool is predominately used for interactions with Java's Keystore and Truststore, generating certificate is from the other side - firstly Keystore is generated and then it is possible to extract certificate from it.

```shell
keytool -genkeypair -keystore kafka-broker-identity.jks -dname "CN=test, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown" -storepass secret -keypass secret -keyalg RSA -keysize 4096 -alias server -validity 3650 -deststoretype pkcs12
```

* `-genkeypair` - command generates a private/public key. Wraps the public key into an X.509 self-signed certificate, which is stored inside new Keystore.
* `-keystore` - if you omit this part, the Keystore will be generated in user's home directory under name .keystore. Otherwise it will be generated where defined. 
* `-dname` - replaces subject field of input request with specified data and outputs modified request. Otherwise you’d be prompted by command line to enter each value separately (similar to openssl `-subj`).
* `-storepass` - the password of the keystore.
* `-keypass` - password used to protect the private key of the generated key pair.
* `-keyalg` - value specifies the algorithm to be used to generate the key pair. This is similar to openssl -algorithm. Java supports DSA, RSA, EC, RSASSA-PSS, EdDSA, Ed25519, Ed448 as per [documentation](https://docs.oracle.com/en/java/javase/17/docs/specs/man/keytool.html#:~:text=appropriate%20level%20of%20security%20strength%20as%20follows).
* `-keysize` - value specifies the size of each key to be generated. This is similar to openssl `-pkeyopt rsa_keygen_bits:2048`. Omitting this value will use defaults.
* `-alias` - alias name for entry in Keystore.
* `-validity` - how many days certificate is valid. Similar to openssl `-days` flag.
* `-deststoretype` - as previously stated private keys and certificates can be stored in a variety of formats. PKCS12 is latest standard.

#### Export Kafka’s certificate

As stated, the process is backwards. Once Keystore is generated, certificate can be extracted from it.

```shell
keytool -exportcert -file kafka.crt -alias server -keystore kafka-broker-identity.jks -storepass secret -rfc
```

* `-exportcert` - command reads a certificate from the Keystore that is associated with `-alias` and stores it in the `-file`. When a file is not specified, the certificate is output to stdout
* `-file` - as stated above, stores certificate to this defined destination.
* `-alias` - as stated above, selects key pair associated with alias.
* `-keystore` - source of the key pair.
* `-storepass` - the password of the keystore
* `-rfc` - certificates are often stored using the printable encoding format defined by the Internet RFC 1421 standard, instead of their binary encoding. This certificate format, also known as Base64 encoding, makes it easy to export certificates to other applications by email or through some other mechanism. If the value is omitted, the flow will not be affected, but simply, the certificate will be stored in binary format.

#### Generate Kafka Client’s Java Truststore

The command below is exactly the same as in “Using OpenSSL tool” section, thus I won’t explain options.

```shell
keytool -importcert -file kafka.crt -alias kafkaCer -keystore client-truststore.jks -storepass truststorepass -noprompt
```

### Prepare generated KeyStore for Docker Compose

Remember that Kafka expects specific files in specific locations:
* `"/etc/kafka/secrets/$KAFKA_SSL_KEYSTORE_FILENAME"`
* `"/etc/kafka/secrets/$KAFKA_SSL_KEYSTORE_CREDENTIALS"`
* `"/etc/kafka/secrets/$KAFKA_SSL_KEY_CREDENTIALS"`

This means that three files have to exists, and all of them have to be mounted into `/etc/kafka/secrets/`.

```yaml
volumes:
  - /var/run/docker.sock:/var/run/docker.sock
  - ./security:/etc/kafka/secrets
```

Several actions have to be done:

```shell
mkdir security &&
mv kafka-broker-identity.jks security &&
echo "secret" > security/password.txt &&
echo "secret" > security/key-password.txt
```

```shell
docker-compose up -d
```

“secret” value to both `security/password.txt` and `security/key-password.txt` are set using previous command with options `-deststorepass secret -destkeypass secret`. 

In Kafka logs you should see:

```shell
===> Configuring ...
SSL is enabled.
===> Running preflight checks ...
...
...
started (kafka.server.KafkaServer)
```

## Common Kafka Properties Settings

```java
public enum TestCommonKafkaProperties {

   BOOTSTRAP_SERVERS_CONFIG("localhost:29092"),
   SECURITY_PROTOCOL_CONFIG("SSL"),
   SSL_TRUSTSTORE_LOCATION_CONFIG(<Absolute Path to client truststore>),
   SSL_TRUSTSTORE_PASSWORD_CONFIG("truststorepass"),
   SSL_ENDPOINT_IDENTIFICATION_ALGORITHM_CONFIG(" ");

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
adminClient = AdminClient.create(Map.of(
      CommonClientConfigs.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS_CONFIG.value(),
      CommonClientConfigs.SECURITY_PROTOCOL_CONFIG, SECURITY_PROTOCOL_CONFIG.value(),
      SslConfigs.SSL_TRUSTSTORE_LOCATION_CONFIG, SSL_TRUSTSTORE_LOCATION_CONFIG.value(),
      SslConfigs.SSL_TRUSTSTORE_PASSWORD_CONFIG, SSL_TRUSTSTORE_PASSWORD_CONFIG.value(),
      SslConfigs.SSL_ENDPOINT_IDENTIFICATION_ALGORITHM_CONFIG, SSL_ENDPOINT_IDENTIFICATION_ALGORITHM_CONFIG.value()
));
```

## Producer

```java
Map<String, Object> producerConfiguration = Map.of(
      CommonClientConfigs.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS_CONFIG.value(),
      ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer",
      ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringSerializer",
      // Security configs      
      CommonClientConfigs.SECURITY_PROTOCOL_CONFIG, SECURITY_PROTOCOL_CONFIG.value(),
      SslConfigs.SSL_TRUSTSTORE_LOCATION_CONFIG, SSL_TRUSTSTORE_LOCATION_CONFIG.value(),
      SslConfigs.SSL_TRUSTSTORE_PASSWORD_CONFIG, SSL_TRUSTSTORE_PASSWORD_CONFIG.value(),
      SslConfigs.SSL_ENDPOINT_IDENTIFICATION_ALGORITHM_CONFIG, SSL_ENDPOINT_IDENTIFICATION_ALGORITHM_CONFIG.value()
);
```

## Consumer

```java
Map<String, Object> consumerConfiguration = Map.of(
      CommonClientConfigs.BOOTSTRAP_SERVERS_CONFIG, BOOTSTRAP_SERVERS_CONFIG.value(),
      ConsumerConfig.GROUP_ID_CONFIG, "groupsecurity",
      ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer",
      ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer",
      // Security configs      
      CommonClientConfigs.SECURITY_PROTOCOL_CONFIG, SECURITY_PROTOCOL_CONFIG.value(),
      SslConfigs.SSL_TRUSTSTORE_LOCATION_CONFIG, SSL_TRUSTSTORE_LOCATION_CONFIG.value(),
      SslConfigs.SSL_TRUSTSTORE_PASSWORD_CONFIG, SSL_TRUSTSTORE_PASSWORD_CONFIG.value(),
      SslConfigs.SSL_ENDPOINT_IDENTIFICATION_ALGORITHM_CONFIG, SSL_ENDPOINT_IDENTIFICATION_ALGORITHM_CONFIG.value()
);
```

```shell
**********
topic = hello.world, partition = 1, offset = 0, customer = null, message = hello
**********
----------
Closing consumer
----------
Deleting topic
```

## Debugging SSL

As previously mentioned in Docker Compose file there is an environment variable `KAFKA_OPTS: "-Djavax.net.debug=ssl,handshake"` which enables JVM to print out logs related to SSL communication. The same command can be used in (for example IntelliJ) when running test clients. A snippet of printed communication:

```shell
 javax.net.ssl|DEBUG|F0|kafka-admin-client-thread | adminclient-1|2022-09-02 10:57:41.330 EEST|ClientHello.java:652|Produced ClientHello handshake message (
"ClientHello": {
  "client version"      : "TLSv1.2",
  "random"              : "32FD552BFB58787472EEACE349FB290108C9D38FBFC61393B27241E2D7A74112",
  "session id"          : "961BA0FDCD1DD54C22229A8C01741485D46BEA0D8B747252B2AB6A2070F7B55D",
  "cipher suites"       : "[TLS_AES_256_GCM_SHA384(0x1302), TLS_AES_128_GCM_SHA256(0x1301), TLS_CHACHA20_POLY1305_SHA256(0x1303), TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384(0xC02C), TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256(0xC02B), TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256(0xCCA9), TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384(0xC030), TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256(0xCCA8), TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256(0xC02F), TLS_DHE_RSA_WITH_AES_256_GCM_SHA384(0x009F), TLS_DHE_RSA_WITH_CHACHA20_POLY1305_SHA256(0xCCAA), TLS_DHE_DSS_WITH_AES_256_GCM_SHA384(0x00A3), TLS_DHE_RSA_WITH_AES_128_GCM_SHA256(0x009E), TLS_DHE_DSS_WITH_AES_128_GCM_SHA256(0x00A2), TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384(0xC024), TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384(0xC028), TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256(0xC023), TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256(0xC027), TLS_DHE_RSA_WITH_AES_256_CBC_SHA256(0x006B), TLS_DHE_DSS_WITH_AES_256_CBC_SHA256(0x006A), TLS_DHE_RSA_WITH_AES_128_CBC_SHA256(0x0067), TLS_DHE_DSS_WITH_AES_128_CBC_SHA256(0x0040), TLS_ECDH_ECDSA_WITH_AES_256_GCM_SHA384(0xC02E), TLS_ECDH_RSA_WITH_AES_256_GCM_SHA384(0xC032), TLS_ECDH_ECDSA_WITH_AES_128_GCM_SHA256(0xC02D), TLS_ECDH_RSA_WITH_AES_128_GCM_SHA256(0xC031), TLS_ECDH_ECDSA_WITH_AES_256_CBC_SHA384(0xC026), TLS_ECDH_RSA_WITH_AES_256_CBC_SHA384(0xC02A), TLS_ECDH_ECDSA_WITH_AES_128_CBC_SHA256(0xC025), TLS_ECDH_RSA_WITH_AES_128_CBC_SHA256(0xC029), TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA(0xC00A), TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA(0xC014), TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA(0xC009), TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA(0xC013), TLS_DHE_RSA_WITH_AES_256_CBC_SHA(0x0039), TLS_DHE_DSS_WITH_AES_256_CBC_SHA(0x0038), TLS_DHE_RSA_WITH_AES_128_CBC_SHA(0x0033), TLS_DHE_DSS_WITH_AES_128_CBC_SHA(0x0032), TLS_ECDH_ECDSA_WITH_AES_256_CBC_SHA(0xC005), TLS_ECDH_RSA_WITH_AES_256_CBC_SHA(0xC00F), TLS_ECDH_ECDSA_WITH_AES_128_CBC_SHA(0xC004), TLS_ECDH_RSA_WITH_AES_128_CBC_SHA(0xC00E), TLS_RSA_WITH_AES_256_GCM_SHA384(0x009D), TLS_RSA_WITH_AES_128_GCM_SHA256(0x009C), TLS_RSA_WITH_AES_256_CBC_SHA256(0x003D), TLS_RSA_WITH_AES_128_CBC_SHA256(0x003C), TLS_RSA_WITH_AES_256_CBC_SHA(0x0035), TLS_RSA_WITH_AES_128_CBC_SHA(0x002F), TLS_EMPTY_RENEGOTIATION_INFO_SCSV(0x00FF)]",
  "compression methods" : "00",
  "extensions"          : [
    "status_request (5)": {
      "certificate status type": ocsp
      "OCSP status request": {
        "responder_id": <empty>
        "request extensions": {
          <empty>
        }
      }
    },
    "supported_groups (10)": {
    ...
...
javax.net.ssl|DEBUG|F0|kafka-admin-client-thread | adminclient-1|2022-09-02 10:57:41.706 EEST|ServerHello.java:888|Consuming ServerHello handshake message
...
javax.net.ssl|DEBUG|F0|kafka-admin-client-thread | adminclient-1|2022-09-02 10:57:41.733 EEST|EncryptedExtensions.java:171|Consuming EncryptedExtensions handshake message
...
javax.net.ssl|DEBUG|F0|kafka-admin-client-thread | adminclient-1|2022-09-02 10:57:41.739 EEST|CertificateMessage.java:1172|Consuming server Certificate handshake message
...
javax.net.ssl|DEBUG|F0|kafka-admin-client-thread | adminclient-1|2022-09-02 10:57:41.761 EEST|CertificateVerify.java:1166|Consuming CertificateVerify handshake message
...
javax.net.ssl|DEBUG|F0|kafka-admin-client-thread | adminclient-1|2022-09-02 10:57:41.764 EEST|Finished.java:917|Consuming server Finished handshake message
...
javax.net.ssl|DEBUG|F0|kafka-admin-client-thread | adminclient-1|2022-09-02 10:57:41.809 EEST|NewSessionTicket.java:567|Consuming NewSessionTicket message
```

# SSL Kafka - With CA

Instead of having explicit certificates of each Kafka instance, we can trust Certificate Authority. For example, we could have Company’s CA, which would sign all internally used certificates. This way we could have only one Company’s CA certificate in client’s Truststore. The con of this approach is that you lose granular control of which applications are whitelisted as permissions are provided to any application who has certificate signed by CA.

## Docker Compose Yaml

Docker compose is exactly the same as in previous chapter - SSL Kafka - Without CA.

## Generating Certificates, Key Pairs, Keystore and Truststore

This section will be a bit different from SSL Kafka - Without CA, because CA’s key pair and certificate will have to be generated, then its private key will be used to sign Kafka broker’s certificate and lastly, instead of using Kafka broker’s certificate in client’s truststore, we’ll use CA’s certificate.

### Using OpenSSL tool

#### Generate Certificate Authority

This is completely the same as generating self signed certificate in SSL Kafka - Without CA.

```shell
openssl req -new -x509 -newkey rsa:4096 -days 365 -keyout CA.key -out CA.crt -subj '/CN=CA-test/OU=Unknown/O=Unknown/L=Unknown/ST=Unknown/C=US' -passin pass:secret -passout pass:secret 
```

#### Generate Kafka’s key pair and certificate request

Everything is mostly the same as in SSL Kafka - Without CA, only difference - instead of creating a self-signed certificate, a certificate request has to be created. Again, as in SSL Kafka - Without CA, there are two ways to create key pair and certificate request. The below command is the same:

```shell
openssl genpkey -out fd.key -algorithm RSA -pkeyopt rsa_keygen_bits:4096 -pass pass:secret
```

However, next command will be slightly different. It will create a certificate request instead of self-signed certificate, by omitting `-x509` flag:

```shell
openssl req -new -days 365 -key fd.key -out kafka-request.crt -subj '/CN=test/OU=Unknown/O=Unknown/L=Unknown/ST=Unknown/C=US' -passin pass:secret -passout pass:secret
```

Instead of two commands, this can be done with just one:

```shell
openssl req -new -newkey rsa:4096 -days 365 -keyout fd.key -out kafka-request.crt -subj '/CN=test/OU=Unknown/O=Unknown/L=Unknown/ST=Unknown/C=US' -passin pass:secret -passout pass:secret
```

Again, same flag `-x509` is omitted to create certificate request. Everything else is the same as in SSL Kafka - Without CA.

#### Sign Kafka’s certificate request with CA

```shell
openssl x509 -req -CA CA.crt -CAkey CA.key -days 365 -in kafka-request.crt -out kafka-signed.crt -CAcreateserial -passin pass:secret
```

* `x509` - x509 command is a multi purpose certificate utility. It can be used to display certificate information, convert certificates to various forms, edit certificate settings and most importantly for this exercise, sign certificate requests.
* `-req` - by default a certificate is expected on input. With this option a certificate request is expected instead.
* `-CA` - specifies the CA certificate to be used for signing - that is its issuer name is set to the subject name of the CA and it is digitally signed using the CA's private key.
* `-CAkey` - sets the CA private key to sign a certificate with. If this option is not specified then it is assumed that the CA private key is present in the CA certificate file.
* `-days` - specifies the number of days to make a certificate valid for. The default is 30 days.
* `-in` - specifies the input filename to read a certificate from. In this case it is a certificate request.
* `-out` - specifies the output filename to write to. In this case output is a signed certificate.
* `-CAcreateserial` - with this option and the `-CA` option the CA serial number file is created if it does not exist. A random number is generated, used for the certificate, and saved into the serial number file.  A certificate serial number is a required field and has to contain an identifier for each Certificate generated by a Certificate Issuer. More information can be found [here](https://ldapwiki.com/wiki/Certificate%20Serial%20Number).
* `-passin` - the input file password source. In this case it’s private/public key password from previous step `-pass pass:secret`.

#### Generate Kafka’s Java Keystore

The command is pretty much the same as in SSL Kafka - Without CA. The only difference is that instead of self signed Kafka’s broker certificate - CA signed Kafka’s broker certificate will be used:

```shell
openssl pkcs12 -export -out keyStore.p12 -inkey fd.key -in kafka-signed.crt -passin pass:secret -passout pass:secret
```

Again, same command from SSL Kafka - Without CA:

```shell
keytool -importkeystore -srckeystore keystore.p12 -srcstorepass secret -srcstoretype pkcs12 -destkeystore kafka-broker-identity.jks -deststorepass secret -destkeypass secret -deststoretype pkcs12  
```

#### Generate Kafka Client’s Java Truststore

Instead of using Kafka’s broker certificate, CA’s certificate will be added to client’s truststore:

```shell
keytool -v -importcert -file CA.crt -keystore client-truststore.jks -storepass truststorepass -noprompt
```

### Using Keytool

In this scenario using Java’s keytool is not as straightforward as using openssl, but nevertheless - possible.

#### Generate Certificate Authority

Generating CA’s certificate is similar to generating self-signed certificate using keytool from SSL Kafka - Without CA. However, two additional flags were added:

```shell
keytool -genkeypair -dname "CN=Root-CA,OU=Certificate Authority,O=Unknown,C=UN" -keystore CA.jks -storepass secret -keypass secret -keyalg RSA -keysize 4096 -alias root-ca -validity 3650 -deststoretype pkcs12 -ext KeyUsage=digitalSignature,keyCertSign -ext BasicConstraints=ca:true,PathLen:3
```

* `-ext KeyUsage=digitalSignature,keyCertSign`
* `-ext BasicConstraints=ca:true,PathLen:3`

According to [X.509 standard](https://www.rfc-editor.org/rfc/rfc5280), `KeyUsage` values, or so called extensions, define what can be done with the key contained in the certificate. This is a full list of possible operations: `digitalSignature`, `nonRepudiation`, `keyEncipherment`, `dataEncipherment`, `keyAgreement`, `keyCertSign`, `cRLSign`, `encipherOnly`, `decipherOnly`. I will not explain each bit separately, but only those which are used in this exercise. 

`BasicConstraints` identifies if the subject of certificates is a CA who is allowed to issue child certificates. `BasicConstraints` can only contain two values:
* `ca` - defines whether certificate is a CA certificate or end entity certificate. `ca` is of type `boolean` and by default is false;
* `pathLenConstraint` - defines how many CAs are allowed in the chain below current CA certificate. The bigger the chain, the more CAs are possible, the less granular certificate is. `pathLenConstraint` is an optional parameter, which is of type `INTEGER (0..MAX)`.

For a certificate that can be used to sign other certificates, the information is some what duplicated, because `BasicConstraints` should contain `ca:true` and `KeyUsage` must contain `keyCertSign`. As per documentation: “The keyCertSign bit is asserted when the subject public key is used for verifying a signature on public key certificates.”

The remaining `digitalSignature` bit, “is asserted when the subject public key is used with a digital signature mechanism to support security services other than certificate signing `[keyCertSign]`”.

#### Generate Kafka’s key pair and certificate request

The below command is the same as in SSL Kafka - Without CA.

```shell
keytool -genkeypair -keystore kafka-broker-identity.jks -dname "CN=test, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown" -storepass secret -keypass secret -keyalg RSA -keysize 4096 -alias server -validity 3650 -deststoretype pkcs12
```

However, the command just create a Keystore with self signed certificate. Thus next step is to create a certificate request from named Keystore, sign it with CA and then put it back into Keystore.

Creating certificate request from existing Keystore command:

```shell
keytool -certreq -file kafka-request.csr -keystore kafka-broker-identity.jks -alias server -keypass secret -storepass secret -storetype pkcs12 
```

* `-certreq` - this command generates certificate request.
* `-file` - this is output file. In this case this will create a certificate request.
* `-keystore` - from which Keystore certificate entry will be taken.
* `-alias` - alias name of the entry to process. From previous step it was `-alias server`, hence this is pointing to the same entry.
* `-keypass` - password used to protect the private key of the generated key pair.
* `-storepass` - the password of the Keystore.
* `-storetype` - in previous step it was defined that the Keystore is defined as PKCS12 format. In this command just repeating this information.

#### Sign Kafka’s certificate request with CA

```shell
keytool -gencert -infile kafka-request.csr -outfile kafka-signed.cer -keystore CA.jks -storepass secret -alias root-ca -ext KeyUsage=digitalSignature,dataEncipherment,keyEncipherment,keyAgreement -ext ExtendedKeyUsage=serverAuth,clientAuth
```

* `-gencert` - command generates a certificate as a response to a certificate request file. The command reads the request signs it by using the alias's private key, and outputs the X.509 certificate into -outfile.
* `-infile` - input file name. In this case it is certificate request.
* `-outfile` - singed X.509 certificate.
* `-keystore` - from which Keystore certificate entry will be used for signing.
* `-storepass` - the password of the Keystore.
* `-alias` - alias name of certificate which will be used for signing.
* `-ext KeyUsage=digitalSignature,dataEncipherment,keyEncipherment,keyAgreement` - differently from CA certificate/key pair generation, this indicates that the certificate will be used for TLS communication. More on each can be read in [4.2.1.3 Key Usage](https://www.rfc-editor.org/rfc/rfc3280#section-4.2.1.3).
* `-ext ExtendedKeyUsage=serverAuth,clientAuth` - as per [documentation](https://www.rfc-editor.org/rfc/rfc3280#section-4.2.1.13) - “This extension indicates one or more purposes for which the certified public key may be used, in addition to or in place of the basic purposes indicated in the key usage extension”. In this case it is stated that key can be used for both server authentication and client authentication.

#### Generate Kafka’s Java Keystore

The identity keystore of the Kafka broker still has the unsigned certificate. Next step is to replace it with the signed one. The keytool has a strange limitation/design. It won't allow you to directly import the signed certificate, and it will give you an error - `java.lang.Exception: Failed to establish chain from reply` - if you try it. The certificate of the Certificate Authority must be present within the `kafka-broker-identity.jks`. Firstly, extract CA certificate:

```shell
keytool -exportcert -file CA.pem -alias root-ca -keystore CA.jks -storepass secret -rfc
```

You can list current certificates (only one should be present):

```shell
keytool -list -keystore kafka-broker-identity.jks -storepass secret
```

Add CA certificate, then signed certificate and delete CA certificate from keystore:

```shell
keytool -v -importcert -file CA.pem -alias root-ca -keystore kafka-broker-identity.jks -storepass secret -noprompt &&
keytool -v -importcert -file kafka-signed.cer -alias server -keystore kafka-broker-identity.jks -storepass secret -noprompt &&
keytool -v -delete -alias root-ca -keystore kafka-broker-identity.jks -storepass secret
```

You can list current certificates (only one should be present with different certificate fingerprint, which indicates that the certificate was overridden):

```shell
keytool -list -keystore kafka-broker-identity.jks -storepass secret
```

#### Generate Kafka Client’s Java Truststore

The command is exactly the same as in SSL Kafka - Without CA, “Generate Kafka Client’s Java Truststore”, just instead of Kafka broker certificate - CA certificate is added.

```shell
keytool -v -importcert -file CA.pem -alias root-ca -keystore client-truststore.jks -storepass truststorepass -noprompt
```

### Prepare generated KeyStore for Docker Compose

Everything else is exactly the same as in SSL Kafka - Without CA.

## Common Kafka Properties Settings

The same as in SSL Kafka - Without CA.

# SASL SSL Kafka























































