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





















































