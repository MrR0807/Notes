# Questions to answer

* `crc` files (see crc files section). Should we filter them out when uploading to S3?
* Why everything is firstly converted to Avro and only then to Parquet? Why not convert directly to Parquet schema?
* Create an example with using parquet-cli `Schemas` class;
* 


# Parquet CLI

## Parquet file inspection

There is an IntelliJ plugin which allows to inspect parquet files' schema and data by drag and drop. The plugin is called `Avro and Parquet Viewer`. For CLI experience use `parquet-cli` - not `parquet-tools` which is deprecated. You won't be able to install it on Mac via brew. However, there are old stackoverflow answers like this [one](https://stackoverflow.com/questions/36140264/inspect-parquet-from-command-line) having example with deprecated `parquet-tools`.

## AvroJson

Maybe helpful in order to parse Json into Avro and then to Parquet?

`Schemas` also in `https://github.com/apache/parquet-mr/`.

**Answer**: It uses the same kite-sdk to infer Avro schema.

# json-avro-converter

There is this library: https://github.com/allegro/json-avro-converter, which can convert Json to Avro, but https://issues.apache.org/jira/browse/CARBONDATA-2627 remove this dependency and provides example without this depedency.




# crc files

[Disabling crc file generation](https://groups.google.com/a/cloudera.org/g/cdk-dev/c/JR45MsLeyTE) points to Hadoop The Definitive Guide, 4th Edition chapter of Data Integrity. TODO - READ and summarize.

> Basically, they are used to make sure data hasn't been corrupted and in some cases replace corrupt copies with good ones. The overhead is fairly minimal for the utility you get, so I don't think it's a good idea to add an option to turn it off. The main concern over a bunch of tiny files is MR performance, but these are not used when calculating splits.

However, somebody has a problem which might be a problem to me:

> I have completely agreed with you that .crc file is good for data integrity and it is not adding any overhead on NN. Still, there are few cases where we need to avoid .crc file, for e.g. in my case I have mounted S3 on S3FS and saving data from rdd to mounting point. It is creating lots of .crc file on S3 which we don't require, to overcome this we need to write an extra utility to filter out all the .crc file which degrade our performance. The interesting observation is that there is a .crc file for `_SUCCESS` file too. and that .crc files is 8 bytes of size while the `_SUCCESS` file is 0 byte. If we are having 1000 million part files than we are using extra `1000M*12` bytes.

# Parquet file anatomy

# Parquet file anatomy via Java implementation


# From JSON to Parquet

## Dates/Timestamps

What format should we expect: https://stackoverflow.com/questions/10286204/what-is-the-right-json-date-format.

# Apache Avro

https://avro.apache.org/docs/1.11.1/getting-started-java/


# CDC with Parquet to S3

SQL database -> Maxwell or Debezium -> Kafka -> Transformer App -> S3

* SQL Datatabase
  * How many databases are supporting this?
  * If we exchange database from say MySQL to PostgreSQL, will the output from (Maxwell or Debezium) be the same?
  * Can the format change with upgrade?
  * For example, [MySQL has 3 types of Binary Logging Formats](https://dev.mysql.com/doc/refman/8.0/en/binary-log-formats.html): statement-based, row-based (default) and mixed logging. Amazon's RDS [recommends using mixed](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_LogAccess.MySQL.BinaryFormat.html). Does it even matter?
* CDC
  * How does Maxwell ensure exactly once? There is Github [issue where the resolution](https://github.com/zendesk/maxwell/issues/785) is unclear (was it implemented?);
  * What happens when Maxwell instance restarts? It should maintain somewhere what it managed to send to Kafka. Is it write ahead log? Say Maxwell writes into ahead log that it read X change and sent it to Kafka. What if Kafka does not respond/is dead. Will it retry? What if both die? Will it retry with new Maxwell instance?
  * Should we use encoding when sending to Kafka from Maxwell (Avro, Protobuf etc)?
  * Procedure how Maxwell will be introduce into new MySQL instances and already existing ones?
* Kafka
  * If we have several partitions and several consumers, how will we ensure order of statements?
  * 


* General observatios about the whole flow
  * A very rigid and clear process of creating new tables/new schemas/new tenants/new microservices/extracting existing capabilities into microservices. This will affect almost all Mambu eventually.
  * Current Banking Engine has a luxury that most likely its database tables can be firstly extracted and have a baseline on which bin logs can be applied. What about databases that won't have a clear way to get baseline?
  * Say two tables are co-dependant in CBE. Entries are written one after another (transactional). Say that one of the tables is extracted into a microservices and has its own lifecycle. Say we have CDC from both of them. There is no way to ensure that this behaviour will be kept. Will we ever be required to maintain that order (like streaming with payments)?
  * We have to ensure database tables evolution without braking our whole flow. Checks of braking changes have to be done before application is started (does not matter if CBE or new microservices). Avro schemas for each table which participates in data extraction? Which is validated against database before starting? These validations have to happen both locally (so people can test locally) and in pipeline. If we don't ensure the validity of schema in upstream, there is little to do in say Flink/Custom Transformer App. The application will detect a change and will do what? Refuse to processes it will the amount of messages in Kafka grow? Or will it just ignore and introduce a breaking change in 




## SQL Database

## Maxwell

## Debezium

## Kafka

## Transformer App

Ingest data -> Transform -> Push to Sink

### Ingest Data

#### Kafka

### Transform

Once data is in the application there are numerious ways how one can map incoming data into Parquet file. To write into Parquet file, there are three things requered:
* Parquet file schema;
* Data to parquet file;
* `ParquetWriter`

#### Construct Parquet File Schema

Parquet format is defined in both Parquet Documentation and in Github. Neither is a good place for a beginner. There are bits and pieces around the internet which try to explain the format, but it is not nearly enough. On this particular topic - in a different section.

##### Define Parquet Schema explicitly

```java
MessageType schema = MessageTypeParser.parseMessageType("""
			message OutputEntity {
				required INT64 timestamp;
				required binary mappedContent (UTF8);
			}""");
```

##### Infer Parquet Schema from JSON

There are several libraries which does this, and initially I've relied on [kite-sdk](https://github.com/kite-sdk/kite). However, because it depends on older Parquet dependencies, there were incompatibility issues, which could not be solved without ditching the library. The next logic step was to inspect how Parquet itself solves this via `parquet-cli`. There is command which, according to documentation, "Create a Parquet file from a data file". The class can be found [here](https://github.com/apache/parquet-mr/blob/master/parquet-cli/src/main/java/org/apache/parquet/cli/commands/ConvertCommand.java). Here are the main snippets: 

```java

# ConvertCommand

public int run() throws IOException {
...
  Schema schema;
  if (avroSchemaFile != null) {
    schema = Schemas.fromAvsc(open(avroSchemaFile));
  } else {
    schema = getAvroSchema(source);
  }
...
}

# BaseCommand class

protected Schema getAvroSchema(String source) throws IOException {
    Formats.Format format;
    try (SeekableInput in = openSeekable(source)) {
      format = Formats.detectFormat((InputStream) in);
      in.seek(0);

      switch (format) {
        case PARQUET:
          return Schemas.fromParquet(getConf(), qualifiedURI(source));
        case AVRO:
          return Schemas.fromAvro(open(source));
        case TEXT:
          if (source.endsWith("avsc")) {
            return Schemas.fromAvsc(open(source));
          } else if (source.endsWith("json")) {
            return Schemas.fromJSON("json", open(source));
          }
        default:
      }

      throw new IllegalArgumentException(String.format(
          "Could not determine file format of %s.", source));
    }
  }

# Schemas class

public static Schema fromJSON(String name, InputStream in) throws IOException {
  return AvroJson.inferSchema(in, name, 20);
}
```

To my surpirse, Parquet cli which is in source of parquet format Java implementation firstly converts to Avro schema, and then uses `AvroParquetWriter`. This is very weird. Wouldn't it make more sense to convert directly to Parquet Schema and write using ParquetWriter? Why the extra hop?

Anyway, by adding `parquet-cli` depedency, it is not possible to infer *Avro* schema from *JSON*:

```java
String json = """
		{
			"id": 1,
			"string": "hello world"
		}""";

final var byteArrayInputStream = new ByteArrayInputStream(json.getBytes(StandardCharsets.UTF_8));
Schema avroSchema = Schemas.fromJSON("thisisname", byteArrayInputStream);
```

##### Infer Parquet Schema from Java Object

There is yet another way to infer Parquet schema and that is using Avro library (org.apache.avro):

```
final var schemaString = ReflectData.get().getSchema(<Class instance>).toString();
final var schema = new Schema.Parser().parse(schemaString);
```

##### Any more? 

I'm sure if I'd spent more time I would find even more different ways to infer Parquet Schema. The problem that there is no straight path to Parquet schema using Java implementation, but only going via already defined encodings (e.g. Protobuf, Avro etc).

#### Data to parquet file

Building Parquet schema has many ways, providing data into Parquet file is no different.

##### Example Parquet Writer

Parquet format Java implementation developers decided not to create a simple, production ready Parquet writer or reader and everything should go through other encodings (e.g. Protobuf, Avro etc.). Well, at least from first glance. However, they've created some example implementations of `ParquerWriter` in [example package](https://github.com/apache/parquet-mr/tree/master/parquet-hadoop/src/main/java/org/apache/parquet/hadoop/example). It is hard to know whether these implementations should be used in production code or not (if I don't want to jump through Avro hoops), but here's the example:


```java
public class Example {

  public static void main(String[] args) throws IOException {

    MessageType schema = MessageTypeParser.parseMessageType("""
      message Pair {
	required binary left (UTF8);
	required binary right (UTF8);
      }""");

    final var simpleGroup = new SimpleGroup(schema);

    simpleGroup
	.append("left", "L")
        .append("right", "R");

    final var out = new ByteArrayOutputStream();
    final var writer = ExampleParquetWriter
	.builder(new ParquetBufferedWriter(new BufferedOutputStream(out)))
	.withType(schema)
	.build();

    writer.write(simpleGroup);
    writer.close();
  }
}
```

After close, the data is flushed to `ByteArrayOutputStream` and can be read or outputed into a file/S3.

##### Implementing your own ParquetWriter

##### 


##### What is `org.apache.parquet.io.OutputFile` and `ParquetBufferedWriter`

There are two possible outputs for `ParquetWriter` - `org.apache.hadoop.fs.Path`





### Push to Sink

#### Amazon S3

Depending on the size of the data you are uploading, Amazon S3 offers the following options:
* Upload an object in a single operation using the AWS SDKs, REST API, or AWS CLI — With a single PUT operation, you can upload a single object up **to 5 GB in size**.
* Upload an object in parts using the AWS SDKs, REST API, or AWS CLI — Using the multipart upload API, you can upload a single large object, up **to 5 TB in size**.

Because I'm not going to upload a file bigger than 5GB in size, hence I will not utilise multipart upload API.

##### Testing Locally

Firstly, use LocalStack as a substition to S3.

```yaml
version: '3'
services:
  localstack:
    container_name: localstack
    image: localstack/localstack
    ports:
      - 4566:4566
      - 4510-4559:4510-4559
    volumes:
      - "localstack-data:/var/lib/localstack"
      - "/var/run/docker.sock:/var/run/docker.sock"

  s3manager:
    container_name: s3manager
    image: cloudlena/s3manager
    ports:
      - 8080:8080
    environment:
      - ACCESS_KEY_ID=NONE
      - SECRET_ACCESS_KEY=NONE
      - ENDPOINT=localstack:4566
      - USE_SSL=false
      - SKIP_SSL_VERIFICATION=true
      - LIST_RECURSIVE=true
    depends_on:
      - localstack

volumes:
  localstack-data:
    driver: local
```

To use Amazon's SDK with LocalStack it is important to set `.withPathStyleAccessEnabled(true)`.

The simple S3 custom client looks like so:

```java
public class AmazonS3Sink {

	private static final AmazonS3 s3Client = S3Client();

	public static void putObject(String bucketName, String key, InputStream input, ObjectMetadata metadata) {

		try {
			s3Client.putObject(bucketName, key, input, metadata);
		} catch (AmazonServiceException e) {
			// The call was transmitted successfully, but Amazon S3 couldn't process
			// it, so it returned an error response.
			e.printStackTrace();
		} catch (SdkClientException e) {
			// Amazon S3 couldn't be contacted for a response, or the client
			// couldn't parse the response from Amazon S3.
			e.printStackTrace();
		}
	}

	public static void createBucket(String bucketName) {

		try {
			if (!s3Client.doesBucketExistV2(bucketName)) {
				// Because the CreateBucketRequest object doesn't specify a region, the
				// bucket is created in the region specified in the client.
				s3Client.createBucket(new CreateBucketRequest(bucketName));

				// Verify that the bucket was created by retrieving it and checking its location.
				String bucketLocation = s3Client.getBucketLocation(new GetBucketLocationRequest(bucketName));
				System.out.println("Bucket location: " + bucketLocation);
			}
		} catch (AmazonServiceException e) {
			// The call was transmitted successfully, but Amazon S3 couldn't process
			// it and returned an error response.
			e.printStackTrace();
		} catch (SdkClientException e) {
			// Amazon S3 couldn't be contacted for a response, or the client
			// couldn't parse the response from Amazon S3.
			e.printStackTrace();
		}
	}

	private static AmazonS3 S3Client() {

		return AmazonS3ClientBuilder
				.standard()
				.withEndpointConfiguration(new AwsClientBuilder.EndpointConfiguration("http://localhost:4566", Regions.US_EAST_1.getName()))
				.withPathStyleAccessEnabled(true)
				.build();
	}
}
```




##### Testing in Amazon






















# Sources
* Hadoop The Definitive Guide, 4th Edition
* [Parquet Types](https://parquet.apache.org/docs/file-format/types/)
* [Parquet Format Github](https://github.com/apache/parquet-format)
* [Parquet Example Package](https://github.com/apache/parquet-mr/tree/master/parquet-hadoop/src/main/java/org/apache/parquet/hadoop/example)
* [Disabling crc file generation](https://groups.google.com/a/cloudera.org/g/cdk-dev/c/JR45MsLeyTE)
* [Amazon S3 AWS SDK Upload Objects](https://docs.aws.amazon.com/AmazonS3/latest/userguide/upload-objects.html)
* [LocalStack](https://localstack.cloud/)












