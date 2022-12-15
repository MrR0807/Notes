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

## Vocabulary

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
