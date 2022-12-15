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



























