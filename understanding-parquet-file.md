# Parquet file anatomy

To start of, I'm going to quote some of the descriptions of Parquet.

From Parquet [oficial documentation](https://parquet.apache.org/):

> Apache Parquet is an open source, column-oriented data file format designed for efficient data storage and retrieval. It provides efficient data compression and encoding schemes with enhanced performance to handle complex data in bulk.

Take from [Wiki](https://en.wikipedia.org/wiki/Apache_Parquet):

> Apache Parquet is a free and open-source column-oriented data storage format in the Apache Hadoop ecosystem. It is similar to RCFile and ORC, the other columnar-storage file formats in Hadoop, and is compatible with most of the data processing frameworks around Hadoop. It provides efficient data compression and encoding schemes with enhanced performance to handle complex data in bulk.

From book [Hadoop: The Definitive Guide](https://www.oreilly.com/library/view/hadoop-the-definitive/9780596521974/)

> Apache Parquet is a columnar storage format that can efficiently store nested data.

When I've started delving into this topic, none of these descriptions really rang with me. What is not explicitly emphasized that Parquet has built on top of previous solutions, which are blended together. I think each component should be addressed independently before trying to understand aggregate - Parquet.

These are components that I'm going to address:
* MapReduce
* File's metadata importance
* Columnar data layout
* Nested columnar data layout (Google's Dremel)
* Encoding (e.g. Avro, Thrift)

## MapReduce

TODO.

Google BigQuery book why Parquet was created.

## File's metadata importance

## Columnar data layout

## Nested columnar data layout

Data structure can be represented in two forms:
* Flat
* Nested

The best example of flat structure can be SQL database entries or CSV file rows. For example SQL table of clients could look:

| Id  | First Name | Last Name |
|-----|------------|-----------|
| 1   | John       | Johnson   |
| 2   | Adam       | Stevenson |
| 3   | Eve        | Stevenson |

An entry of one row could be represented in JSON format like so:

```json
{
  "id": 1,
  "firstName": "John",
  "lastName": "Johnson"
}
```

However, flat structures are not always best represantion of data as stated in Google's Dremel document: 

> The data used in web and scientific computing is often nonrelational. Hence, a flexible data model is essential in these domains. Data structures used in programming languages, messages exchanged by distributed systems, structured documents, etc. lend themselves naturally to a **nested** representation. <...> A **nested data model underlies most of structured data processing** at Google and reportedly at other major web companies.

A nested type, for example, in SQL databases can be represented via relationships: one-to-many, many-to-many etc. This is represented in SQL by duplicating the parent data next to the child. For example, say we have additional table, which represents sales to particular client. The client table will be represented as previous table, while the sales transactions could look like so:

| Sales Id | Client Id | Product | Amount |
|----------|-----------|------------|--|
| 1 | 1 | Apple | 0.60 |
| 2 | 1 | Banana | 1.00 |
| 3 | 3 | Apple | 0.60 |

Via join I can create a nested structure:

```sql
SELECT * FROM clients AS c 
INNER JOIN  sales AS s
ON c.id = s.client_id;
```

Result:

| Id  | First Name | Last Name | Sales Id | Client Id | Product | Amount |
|-----|-----------|-----------|----------|-----------|---------|--------|
| 1   | John      | Johnson          | 1        | 1         | Apple   | 0.60   |
| 1   | John      | Johnson          | 2        | 1         | Banana  | 1.00   |
| 3   | Eve       | Stevenson          | 3        | 3         | Apple   | 0.60   |

As stated, nested type in flat structure in this case has been represented as repetition of parent entity. In JSON format, this can be represented easier:

```json
[
  {
    "id": 1,
    "firstName": "John",
    "lastName": "Johnson"
    "sales": [
      {
        "id": 1,
        "product": "Apple",
        "amount": 0.60
      },
      {
        "id": 2,
        "product": "Banana",
        "amount": 1.00
      }
    ]
  },
  {
    "id": 3,
    "firstName": "Eve",
    "lastName": "Stevenson"
    "sales": [
      {
        "id": 3,
        "product": "Apple",
        "amount": 0.60
      }
    ]
  }
]
```

Nested types before Google's Dremel in columnar formats were not solved or at least as stated in the paper:

> Column stores have been adopted for analyzing relational data [1] but to the **best of our knowledge have not been extended to nested data models.**

Furthermore, trying to adapt their data representation to existing flat columnar structures by "<...> normalizing and recombining such data at web scale is [was] usually **prohibitive**". Thus they needed a new solution - Dremel.

At this point, it is important to explicitly emphasize that Dremel's nested structure came to existance to solve Google's (and other web companies) natural data structure representation need, which was nested, in columnar databases instead of trying to apply flattening strategies and then recombination in current columnar data representation solution space.

The creation of nested columnar structure was so successful, that opensource projects like Parquet were born. Later (after 4 years from Dremel publication), Google publicised another paper proving that [Storing and Querying Tree-Structured Records in Dremel](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/43119.pdf) are performant and scalable.

### Dremel's Nested Structure

In this section, I will explore Dremel's nested structure and via several examples, showcase Definition and Repetition concepts and how they help represent nested structures.

**Note!** A lot of information in this section will be copy pasted from this great Twitter [blog post on this very topic](https://blog.twitter.com/engineering/en_us/a/2013/dremel-made-simple-with-parquet). I'm copying just to have all information in one place without needing to jump between pages and also adding additional examples for better clarity.

#### The model

To store in a columnar format we first need to describe the data structures using a schema. This is done using a model similar to **Protocol buffers**. This model is minimalistic in that it represents nesting using groups of fields and repetition using repeated fields. There is no need for any other complex types like Maps, List or Sets as they all can be mapped to a combination of repeated fields and groups.

The root of the schema is a group of fields called a message. Each field has three attributes: a repetition, a type and a name. The type of a field is either a group or a primitive type (e.g., int, float, boolean, string) and the repetition can be one of the three following cases:
* required: exactly one occurrence
* optional: 0 or 1 occurrence
* repeated: 0 or more occurrences

For example, here’s a schema one might use for an address book:

```
message AddressBook {
  required string owner;
  repeated string ownerPhoneNumbers;
  repeated group contacts {
    required string name;
    optional string phoneNumber;
  }
}
```

Lists (or Sets) can be represented by a repeating field:


| Schema: List of Strings                                   | Data: ["a", "b", "c", ...]                                   |
|-----------------------------------------------------------|--------------------------------------------------------------|
| message ExampleList { <br/>repeated string list;<br/> } | {<br/> list: "a", <br/> list: "b", <br/> list: "c",<br/> ... <br/> } |

A Map is equivalent to a repeating field containing groups of key-value pairs where the key is required:


| Schema: Map of strings to strings                                                                             | Data: {"AL" -> "Alabama", ... }                                                                                   |
|---------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------|
| message ExampleMap {<br/>repeated group map {<br/>required string key;<br/>optional string value;<br/>}<br/>} | {<br/>map: {<br/>key: "AL",<br/>value: "Alabama"<br/>},<br/>map: {<br/>key: "AK",<br/>value: "Alaska"<br/>}<br/>} |








## Encoding








**BIG TODO**

## Conclusion

So the nested columnar data is represented in Protobuf schema, the metada is represented in Thrift encoding, columnar data applies compactions algos.



# Parquet file anatomy via Java implementation

**BIG TODO**

# CDC with Parquet to S3

SQL database -> Maxwell/Debezium -> Kafka -> Transformer App -> S3

## Questions

* SQL Datatabase:
  * If we exchange database from say MySQL to PostgreSQL, will the output from (Maxwell or Debezium) be the same?
  * Can the format change with upgrade of database?
  * For example, [MySQL has 3 types of Binary Logging Formats](https://dev.mysql.com/doc/refman/8.0/en/binary-log-formats.html): statement-based, row-based (default) and mixed logging. Amazon's RDS [recommends using mixed](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_LogAccess.MySQL.BinaryFormat.html). Does it even matter?
* CDC:
  * How does Maxwell ensure exactly once? There is Github [issue where the resolution](https://github.com/zendesk/maxwell/issues/785) is unclear (was it implemented?);
  * What happens when Maxwell instance restarts? It should maintain somewhere what it managed to send to Kafka. Is it write ahead log? Say Maxwell writes into ahead log that it read X change and sent it to Kafka. What if Kafka does not respond/is dead. Will it retry? What if both die? Will it retry with new Maxwell instance?
  * Should we use encoding when sending to Kafka from Maxwell (Avro, Protobuf etc)?
  * Procedure how Maxwell will be introduce into new MySQL instances and already existing ones?
* Kafka
  * If we have several partitions and several consumers, how will we ensure order of statements?
  * We have to identify which tables have no need for order then they can have more partitions, while tables where orders matters have to have only one partition (CDC throughput for such tables).
* Transformation App:
  * Use standard encodings (Avro, Protobuf etc)?
  * If we decide to stick with JSON, then we need to decide how will we deserialize and serialize that data. Should we write our own Parquet Schema infer logic from JSON? (Infering parquet schema from JSON is not a good solution, because that means schema can change without us noticing).
  * Buffering? Should we try to buffer according to file size or just flush on time bases? This might create widely different file sizes.
  * If we decide to buffer and flush on size, then we'll have to investigate how each data type/compression algorith affects the size. When Stream of data is moving, there is no way of knowing for sure what size Parquet file will be. This is due to several reasons: 1) Parquet itself is an encoding format not only column oriented data structure; 2) Due to being column oriented structure it can perform different kinds of store optimisation like [Run-length encoding](https://en.wikipedia.org/wiki/Run-length_encoding) or [Dictionary Encoding](https://github.com/apache/parquet-format/blob/master/Encodings.md#dictionary-encoding-plain_dictionary--2-and-rle_dictionary--8); 3) Lastly, we can apply compression like Snappy.
  * How much does it cost in terms of efficiency to create `ParquetWriter` for each stream? Should they be reusable?
  * Next to parquet files, we have to provide simple CRUD operations that happened in SQL format/txt files. This will allow clients to transition from database backup more easily and will help us in future features (archiving). However, this will add double size pressure (presumambly even bigger than Parquet files) on sink.
  * Recovery in the application? Say application has processed X amount of data and crashes. The new instance will just reprocesses the same data and it should be fine. However, what if the X amount of data is a big number? Should we have some kind of checkpoints like Flink? How much overhead does it create? Where will be store this data? In Kubernetes persistance storage of 3rd party like S3?
  * Recovery part two. Say application managed to flush Parquet file, but not sent offset to Kafka, how will we validate that we shouldn't duplicate data?
  * Should we deal somehow with possible duplication? Create hashcodes of each statements and check whether such statements were already processed in X time window (say we have moving 5 minutes time window).

* General observatios about the whole flow:
  * A very rigid and clear process of creating new tables/new schemas/new tenants/new microservices/extracting existing capabilities into microservices. This will affect almost all organisation eventually.
  * Current Banking Engine has a luxury that most likely its database tables can be firstly extracted and have a baseline on which bin logs can be applied. What about databases that won't have a clear way to get baseline?
  * Say two tables are co-dependant in CBE. Entries are written one after another (transactional). Say that one of the tables is extracted into a microservices and has its own lifecycle. Say we have CDC from both of them. There is no way to ensure that this behaviour will be kept. Will we ever be required to maintain that order (like streaming with payments)?
  * We have to ensure database tables evolution without braking our whole flow. Checks of braking changes have to be done before application is started (does not matter if CBE or new microservices). Avro schemas for each table which participates in data extraction? Which is validated against database before starting? These validations have to happen both locally (so people can test locally) and in pipeline. If we don't ensure the validity of schema in upstream, there is little to do in say Flink/Custom Transformer App. The application will detect a change and will do what? Refuse to processes it will the amount of messages in Kafka grow? Or will it just ignore and introduce a breaking change for clients?


# Full CDC flow

## SQL Database

### MySQL

Todo

### PostgreSQL

Todo

## Maxwell

Todo

## Debezium

Todo

## Kafka

Todo. The mechanism is clear, it just a matter of selecting and writing the correct way to handle incoming records (most likely use kafka-sdk library), whether parallelize each received batch of records per X worker threads, each working with a separate Parquet file etc wherever possible.

## Transformer App

Ingest data -> Transform -> Push to Sink

### Ingest Data

#### Kafka

### Transform

Once data is in the application there are numerious ways how one can map incoming data into Parquet file. To write into Parquet file, there are three things requered:
* Parquet file schema;
* Data to parquet file in read form;
* `ParquetWriter`

#### Construct Parquet File Schema

Parquet format is defined in both Parquet Documentation and in parquet-format Github repository. Neither is a good place for a beginner. There are bits and pieces around the internet which try to explain the format, but it is not nearly enough. On this particular topic - in a different section.

##### Define Parquet Schema explicitly

The simplest and most straightforward approach is to construct Parquet schema by hand. This might get more complicated if nested structures are introduced due to Definition and Repetition properties, but nevertheless, doable.

```java
MessageType schema = MessageTypeParser.parseMessageType("""
			message OutputEntity {
				required INT64 timestamp;
				required binary mappedContent (UTF8);
			}""");
```

##### Infer Avro Schema from JSON

There are several libraries which does this, and initially I've relied on [kite-sdk](https://github.com/kite-sdk/kite). However, because it depends on older Parquet dependencies, there were incompatibility issues, which could not be solved without ditching the library. The next logical step was to inspect how Parquet itself solves this via `parquet-cli`. There is command which, according to documentation, "Creates a Parquet file from a data file". The class can be found [here](https://github.com/apache/parquet-mr/blob/master/parquet-cli/src/main/java/org/apache/parquet/cli/commands/ConvertCommand.java). Here are the main snippets: 

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

To my surpirse, Parquet cli, which is in source of Parquet format Java implementation, firstly converts to Avro schema, and then uses `AvroParquetWriter`. This is very weird. Wouldn't it make more sense to convert directly to Parquet Schema and write using ParquetWriter? Why the extra hop?

Anyway, by adding `parquet-cli` depedency, it is possible to infer *Avro* schema from *JSON*:

```java
String json = """
		{
			"id": 1,
			"string": "hello world"
		}""";

final var byteArrayInputStream = new ByteArrayInputStream(json.getBytes(StandardCharsets.UTF_8));
Schema avroSchema = Schemas.fromJSON("thisisname", byteArrayInputStream);
```

##### Infer Avro Schema from Java Object via reflection

There is yet another way to infer Parquet schema and that is using Avro library (org.apache.avro):

```
final var schemaString = ReflectData.get().getSchema(<Class instance>).toString();
final var schema = new Schema.Parser().parse(schemaString);
```

Again, once Avro schema is defined, we can use `AvroParquetWriter`.

##### Any more? 

I'm sure if I'd spent more time I would find even more different ways to infer Parquet Schema. The problem that there is no straight path to Parquet schema using Java implementation, but only going via already defined encodings (e.g. Protobuf, Avro etc).

#### Data to parquet file

Building Parquet schema has many ways, building data into `ParquetWriter` understandable format is no different. How the data is built depends directly on type of writer, but at the same time not really. Let me show you want I mean.

##### Building data manually

One of the easiest ways to build data which can be writter by `ParquetWriter` is by building it manually. For example, to build a Parquet record, I can use a Java object from `org.apache.parquet.example`:

```java
MessageType schema = MessageTypeParser.parseMessageType("""
	message OutputEntity {
		required INT64 timestamp;
		required binary mappedContent (UTF8);
	}""");

final var simpleGroup = new SimpleGroup(schema);

simpleGroup
		.append("timestamp", Instant.now().toEpochMilli())
		.append("mappedContent", "This is content");
```

By creating `SimpleGroup` and appending data manually, I have created a record which can be written using `ExampleParquetWriter` from ` org.apache.parquet.hadoop.example`. Or, say I'd like to use `AvroParquetWriter`, then I'd have to create Avro record manually:

```java
// Schema is in Avro format, not Parquet
final var avroSchema = """
	{
		"type": "record",
		"name": "OutputEntity",
		"fields": [
		    {"name": "timestamp", "type": "long"},
		    {"name": "mappedContent", "type": ["string"]}
		 ]
	}""";

final var schema = new Schema.Parser().parse(avroSchema);

GenericRecord user = new GenericData.Record(schema);
user.put("timestamp", Instant.now().toEpochMilli());
user.put("mappedContent", "This is content");
```

Similar things can be done with other encodings.

##### Via Java Objects

Instead of manually defining each field and then mapping a value to it, I can create a Java objects which will automatically map the values into Parquet file from object's instances values. 

```java
MessageType schema = MessageTypeParser.parseMessageType("""
	message OutputEntity {
		required INT64 timestamp;
		required binary mappedContent (UTF8);
	}""");


final var outputEntity = new OutputEntity(schema, Instant.now().toEpochMilli(), "This is yet to be");
```

```java
public static class OutputEntity extends SimpleGroup {
	public OutputEntity(GroupType schema, long timestamp, String mappedContent) {
		super(schema);
		add("timestamp", timestamp);
		add("mappedContent", mappedContent);
	}
}
```

Similar thing can be done with [Avro's serializing](https://avro.apache.org/docs/1.11.1/getting-started-java/#serializing).

##### Via Avro's reflection utilities

As stated previously, even `parquet-cli` firstly maps to Avro schema and then uses `AvroParquetWriter` to write data into Parquet files. Building on this weird pratices, I can use Avro's utilities like inspecting data via reflection and mapping it:

```java
final var schemaString = ReflectData.get().getSchema(lt.test.simplecdc.model.OutputEntity.class).toString();
final var schema = new Schema.Parser().parse(schemaString);

final var record = new OutputEntity(Instant.now().toEpochMilli(), "Content");
```

And then when creating `AvroParquetWriter`:

```java
final var writer = AvroParquetWriter.<OutputEntity>builder(new Path("hello123.parquet"))
		.withSchema(schema)
		.withDataModel(ReflectData.get()) //This has to be defined to inspect OutputEntity instance properties
		.build();
```

This is a feature of `AvroParquetWriter` and not possible with `ExampleParquetWriter`.

##### Extending Avro's `IndexedRecord`

If my custom class extends Avro's `IndexedRecord` then reflection is not required anymore.

```java
public static class OutputEntity implements IndexedRecord {

	private String mappedContent;
	private long timestamp;

	public OutputEntity(long timestamp, String mappedContent) {
		this.mappedContent = mappedContent;
		this.timestamp = timestamp;
	}

	public long getTimestamp() {
		return timestamp;
	}

	public String getMappedContent() {
		return mappedContent;
	}

	@Override
	public void put(int i, Object v) {
		switch (i) {
			case 0 -> this.mappedContent = (String) v;
			case 1 -> this.timestamp = (Long) v;
			default -> throw new RuntimeException("");
		}
	}

	@Override
	public Object get(int i) {
		return switch (i) {
			case 0 -> this.mappedContent;
			case 1 -> this.timestamp;
			default -> throw new RuntimeException("");
		};
	}

	@Override
	public Schema getSchema() {
		return null;
	}
}
```


**NOTE!**. Because this relies on strict order, it is best to define Avro schema by hand and maintaine dependency between field order in the Avro schema and order of `get` method in Java class.


##### From JSON to Avro `GenericRecord`

Another tricket that Avro library has is constructing a `GenericRecord`, which I've used in section "Building data manually", from JSON. Here's the code:

```java
final var avroSchema = """
	{
		"type": "record",
		 "name": "OutputEntity",
		 "fields": [
			 {"name": "timestamp", "type": "long"},
			 {"name": "mappedContent", "type": "string"}
		 ]
	}""";

final var schema = new Schema.Parser().parse(avroSchema);

final var jsonDecoder = DecoderFactory.get().jsonDecoder(schema, new DataInputStream(new ByteArrayInputStream(JSON.getBytes())));
final var reader = new GenericDatumReader<GenericRecord>(schema);
//Just to show that this is GenericRecord
final GenericRecord jsonRecord = reader.read(null, jsonDecoder);
```

Then, `AvroParquetWriter` can be used.

##### Any more?

I'm sure if I'd spent more time I would find even more different ways to build record for `ParquetWriter`.


#### `ParquetWriter`

In parquet-mr Github repo, there are four implementations of `ParquetWriter` provided out of the box:
* `AvroParquetWriter`;
* `ExampleParquetWriter`;
* `ProtoParquetWriter`;
* `ThriftParquetWriter`.

The biggest problem I think with trying to implement your own `ParquetWriter` is lack of good documentation and clear guidance how all classes interact.

##### `ExampleParquetWriter`

Parquet format Java implementation developers decided not to create a simple, production ready Parquet writer or reader. Everything it seems should go through other encodings (e.g. Protobuf, Avro etc.). However, they've created some example implementations of `ParquerWriter` in [example package](https://github.com/apache/parquet-mr/tree/master/parquet-hadoop/src/main/java/org/apache/parquet/hadoop/example). It is hard to know whether these implementations should be used in production code or not (if I don't want to jump through Avro hoops), but here's the example of using it:


```java
public class Example {

  public static void main(String[] args) throws IOException {

    MessageType schema = MessageTypeParser.parseMessageType("""
	message OutputEntity {
		required INT64 timestamp;
		required binary mappedContent (UTF8);
	}""");

    final var simpleGroup = new SimpleGroup(schema);

    simpleGroup
		.append("timestamp", Instant.now().toEpochMilli())
		.append("mappedContent", "This is content");

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

##### Implementing your own `ParquetWriter`

**BIG TODO**

##### Using `AvroParquetWriter`

`AvroParquetWriter` seems to be go to writer when used in examples or even parquet's Java implementation own code. Building `AvroParquetWriter` is as straightfoward as `ExampleParquetWriter`:

```java
public static void main(String[] args) throws IOException {

	final var avroSchema = """
		{
			"type": "record",
			 "name": "OutputEntity",
			 "fields": [
				 {"name": "timestamp", "type": "long"},
				 {"name": "mappedContent", "type": ["string"]}
			 ]
		}""";

	final var schema = new Schema.Parser().parse(avroSchema);

	GenericRecord user = new GenericData.Record(schema);
	user.put("timestamp", Instant.now().toEpochMilli());
	user.put("mappedContent", "This is content");

	final var writer = AvroParquetWriter.builder(new Path("hello.parquet"))
			.withSchema(schema)
			.build();

	writer.write(user);
	writer.close();
}
```


##### What is `org.apache.parquet.io.OutputFile` and `ParquetBufferedWriter`

Maybe you've noticed that in `ExampleParquetWriter` I've provided to `builder()` method `ParquetBufferedWriter` instance while with `AvroParquetWriter` - `org.apache.hadoop.fs.Path`. 

Looking into `ParquetWriter` source code, there are two ways how to obtain builder instance:

```java
protected Builder(Path path) {
  this.path = path;
}

protected Builder(OutputFile path) {
  this.file = path;
}
```

When `Path` is provided, the `writer.close()` method will create a file and place `ParquetWriter` content to it. While with `OutputFile`, the content will be placed into. However, the lack of documentation about `OutputFile` makes it hard to understand the full scope of this class and how it should be implemented. Taken from source code (doesn't even have a clear explanation what this interface should represent):

```java
public interface OutputFile {

  PositionOutputStream create(long blockSizeHint) throws IOException;

  PositionOutputStream createOrOverwrite(long blockSizeHint) throws IOException;

  boolean supportsBlockSize();

  long defaultBlockSize();

  default String getPath() {
    return null;
  }
}
```

**TODO investigate** how it is used and other examples like https://github.com/apache/flink/blob/master/flink-formats/flink-parquet/src/main/java/org/apache/flink/formats/parquet/StreamOutputFile.java and org.apache.parquet.hadoop.util.HadoopOutputFile.


### Push to Sink

At this stage it is time to write data into sink.

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














# Stuff without a place

## Parquet CLI

### Parquet file inspection

There is an IntelliJ plugin which allows to inspect parquet files' schema and data by drag and drop. The plugin is called `Avro and Parquet Viewer`. For CLI experience use `parquet-cli` - not `parquet-tools` which is deprecated. You won't be able to install it on Mac via brew. However, there are old stackoverflow answers like this [one](https://stackoverflow.com/questions/36140264/inspect-parquet-from-command-line) having example with deprecated `parquet-tools`.

### crc files

[Disabling crc file generation](https://groups.google.com/a/cloudera.org/g/cdk-dev/c/JR45MsLeyTE) points to Hadoop The Definitive Guide, 4th Edition chapter of Data Integrity. TODO - READ and summarize.

> Basically, they are used to make sure data hasn't been corrupted and in some cases replace corrupt copies with good ones. The overhead is fairly minimal for the utility you get, so I don't think it's a good idea to add an option to turn it off. The main concern over a bunch of tiny files is MR performance, but these are not used when calculating splits.

However, somebody has a problem which might be a problem to me:

> I have completely agreed with you that .crc file is good for data integrity and it is not adding any overhead on NN. Still, there are few cases where we need to avoid .crc file, for e.g. in my case I have mounted S3 on S3FS and saving data from rdd to mounting point. It is creating lots of .crc file on S3 which we don't require, to overcome this we need to write an extra utility to filter out all the .crc file which degrade our performance. The interesting observation is that there is a .crc file for `_SUCCESS` file too. and that .crc files is 8 bytes of size while the `_SUCCESS` file is 0 byte. If we are having 1000 million part files than we are using extra `1000M*12` bytes.





# TODO

* [parquet-json some library INVESTIGATE](https://github.com/getyourguide/parquet-json)
* https://hackolade.com/help/Parquetschema.html
* https://dzone.com/articles/understanding-how-parquet
* https://liam-blog.ml/2020/05/31/details-you-need-to-know-about-Apache-Parquet/
* https://github.com/apache/parquet-format/blob/master/LogicalTypes.md
* http://www.benstopford.com/2015/02/14/log-structured-merge-trees/
* https://github.com/apache/parquet-format/blob/master/Encodings.md#dictionary-encoding-plain_dictionary--2-and-rle_dictionary--8
* https://towardsdatascience.com/demystifying-the-parquet-file-format-13adb0206705
* https://blog.acolyer.org/2018/09/26/the-design-and-implementation-of-modern-column-oriented-database-systems/
* https://cloud.google.com/blog/products/bigquery/inside-capacitor-bigquerys-next-generation-columnar-storage-format
* https://storage.googleapis.com/pub-tools-public-publication-data/pdf/43119.pdf


# Sources
* Hadoop The Definitive Guide, 4th Edition
* [Parquet Types](https://parquet.apache.org/docs/file-format/types/)
* [Parquet Format Github](https://github.com/apache/parquet-format)
* [Parquet Example Package](https://github.com/apache/parquet-mr/tree/master/parquet-hadoop/src/main/java/org/apache/parquet/hadoop/example)
* [Disabling crc file generation](https://groups.google.com/a/cloudera.org/g/cdk-dev/c/JR45MsLeyTE)
* [Amazon S3 AWS SDK Upload Objects](https://docs.aws.amazon.com/AmazonS3/latest/userguide/upload-objects.html)
* [LocalStack](https://localstack.cloud/)
* [Apache Avro Documentation](https://avro.apache.org/docs/1.11.1/getting-started-java/)
* [Dremel made simple with Parquet](https://blog.twitter.com/engineering/en_us/a/2013/dremel-made-simple-with-parquet)
* [Dremel: Interactive Analysis of Web-Scale Datasets](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/36632.pdf)
* [Column-oriented Database Systems](http://www.cs.umd.edu/~abadi/papers/columnstore-tutorial.pdf)
* [Storing and Querying Tree-Structured Records in Dremel](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/43119.pdf)








