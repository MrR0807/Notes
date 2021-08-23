# Kafka Producer Properties

## High
key.serializer
value.serializer
acks
bootstrap.servers
buffer.memory
compression.type
retries

## Medium
batch.size
client.dns.lookup
client.id
connections.max.idle.ms
delivery.timeout.ms
linger.ms
max.block.ms
max.request.size
partitioner.class
receive.buffer.bytes
request.timeout.ms
send.buffer.bytes
socket.connection.setup.timeout.max.ms
socket.connection.setup.timeout.ms

## Low
enable.idempotence
interceptor.classes
max.in.flight.requests.per.connection
metadata.max.age.ms
metadata.max.idle.ms
metric.reporters
metrics.num.samples
metrics.recording.level
metrics.sample.window.ms
reconnect.backoff.max.ms
reconnect.backoff.ms
retry.backoff.ms
transaction.timeout.ms
transactional.id

------------------------------------------------------------------------------------------------------------------

## High Security

ssl.key.password
ssl.keystore.certificate.chain
ssl.keystore.key
ssl.keystore.location
ssl.keystore.password
ssl.truststore.certificates
ssl.truststore.location
ssl.truststore.password

## Medium Security

sasl.client.callback.handler.class
sasl.jaas.config
sasl.kerberos.service.name
sasl.login.callback.handler.class
sasl.login.class
sasl.mechanism
security.protocol
ssl.enabled.protocols
ssl.keystore.type
ssl.protocol
ssl.provider
ssl.truststore.type

## Low Security
sasl.kerberos.kinit.cmd
sasl.kerberos.min.time.before.relogin
sasl.kerberos.ticket.renew.jitter
sasl.kerberos.ticket.renew.window.factor
sasl.login.refresh.buffer.seconds
sasl.login.refresh.min.period.seconds
sasl.login.refresh.window.factor
sasl.login.refresh.window.jitter
security.providers
ssl.cipher.suites
ssl.endpoint.identification.algorithm
ssl.engine.factory.class
ssl.keymanager.algorithm
ssl.secure.random.implementation
ssl.trustmanager.algorithm


# Kafka Consumer Properties

## High
key.deserializer
value.deserializer
bootstrap.servers
fetch.min.bytes
group.id
heartbeat.interval.ms
max.partition.fetch.bytes
session.timeout.ms

## Medium
allow.auto.create.topics
auto.offset.reset
client.dns.lookup
connections.max.idle.ms
default.api.timeout.ms
enable.auto.commit
exclude.internal.topics
fetch.max.bytes
group.instance.id
isolation.level
max.poll.interval.ms
max.poll.records
partition.assignment.strategy
receive.buffer.bytes
request.timeout.ms
send.buffer.bytes
socket.connection.setup.timeout.max.ms
socket.connection.setup.timeout.ms

## Low
auto.commit.interval.ms
check.crcs
client.id
client.rack
fetch.max.wait.ms
interceptor.classes
metadata.max.age.ms
metric.reporters
metrics.num.samples
metrics.recording.level
metrics.sample.window.ms
reconnect.backoff.max.ms
reconnect.backoff.ms
retry.backoff.ms

------------------------------------------------------------------------------------------------------------------

## High Security
ssl.key.password
ssl.keystore.certificate.chain
ssl.keystore.key
ssl.keystore.location
ssl.keystore.password
ssl.truststore.certificates
ssl.truststore.location
ssl.truststore.password

## Medium Security
sasl.client.callback.handler.class
sasl.jaas.config
sasl.kerberos.service.name
sasl.login.callback.handler.class
sasl.login.class
sasl.mechanism
security.protocol
ssl.enabled.protocols
ssl.keystore.type
ssl.protocol
ssl.provider
ssl.truststore.type

## Low Security
sasl.kerberos.kinit.cmd
sasl.kerberos.min.time.before.relogin
sasl.kerberos.ticket.renew.jitter
sasl.kerberos.ticket.renew.window.factor
sasl.login.refresh.buffer.seconds
sasl.login.refresh.min.period.seconds
sasl.login.refresh.window.factor
sasl.login.refresh.window.jitter
security.providers
ssl.cipher.suites
ssl.endpoint.identification.algorithm
ssl.engine.factory.class
ssl.keymanager.algorithm
ssl.secure.random.implementation
ssl.trustmanager.algorithm
