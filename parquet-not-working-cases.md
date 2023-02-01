# Setup

Gradle dependencies.

```
dependencies {

    implementation 'org.apache.parquet:parquet-avro:1.12.3'
    implementation 'org.apache.parquet:parquet-common:1.12.3'
    implementation 'org.apache.parquet:parquet-encoding:1.12.3'
    implementation 'org.apache.parquet:parquet-hadoop:1.12.3'
    implementation 'org.apache.parquet:parquet-column:1.12.3'
    implementation 'org.apache.parquet:parquet-cli:1.12.3'
    implementation 'org.apache.hadoop:hadoop-common:3.3.4'
    implementation 'org.apache.hadoop:hadoop-mapreduce-client-core:3.3.4'

}
```


# Why

I have found inconsistencies between how Avro and Parquet converts schemas, how values are serialized and deserialized, and how `parquet-cli` tool interacts with written files. I wanted to document those cases for both my own sanity and to raise awareness of these cases.

# What

Each case will be started by defining both Avro and Parquet schemas by hand. Then inspect how they are automatically converted using `AvroSchemaConverter` into one another and vice versa, use generic writting methods to serialise information and then deserialise it and lastly use `parquet-cli` to again read those files. I will start with simple cases and ramp up by adding complex types like lists and maps.

# Simple flat schema

## Hand written Avro Schema

```
{
	"type":"record",
	"name":"Out",
	"fields":[
		{
			"name":"MyInteger",
			"type":{"type":"int"}
		},
		{
			"name":"MyString",
			"type":{"type":"string"}
		}
	]
}
```

## Hand written Parquet Schema

```
message Out {
	required int32 MyInteger;
	required binary MyString (UTF8);
}
```

## `AvroSchemaConverter` conversion from Avro to Parquet

```
message Out {
  required int32 MyInteger;
  required binary MyString (STRING);
}
```

The only difference is that instead of `UTF8` it is `STRING`. From [parquet-format Github](https://github.com/apache/parquet-format/blob/master/LogicalTypes.md#string) definition, they are compatible:

> `STRING` corresponds to `UTF8` ConvertedType.


## `AvroSchemaConverter` conversion from Parquet to Avro

Exactly the same as hand written.

```
{
	"type":"record",
	"name":"Out",
	"fields":[
		{"name":"MyInteger","type":"int"},
		{"name":"MyString","type":"string"}
]}
```

## Full Code

```java
public class TestOne {

	private static final AvroSchemaConverter AVRO_SCHEMA_CONVERTER = new AvroSchemaConverter(new Configuration());

	public static void main(String[] args) throws IOException {

		final var avroSchemaString = """
				{
					"type":"record",
					"name":"Out",
					"fields":[
						{
							"name":"MyInteger",
							"type":{"type":"int"}
						},
						{
							"name":"MyString",
							"type":{"type":"string"}
						}
					]
				}""";
		final var avroSchema = new Schema.Parser().parse(avroSchemaString);

		final var parquetSchemaString = """
				message Out {
					required int32 MyInteger;
					required binary MyString (UTF8);
				}""";
		final var parquetSchema = MessageTypeParser.parseMessageType(parquetSchemaString);

		final var avroSchemaFromParquet = AVRO_SCHEMA_CONVERTER.convert(parquetSchema);
		/**
		 * {
		 * 	"type":"record",
		 * 	"name":"Out",
		 * 	"fields":[
		 * 		{"name":"MyInteger","type":"int"},
		 * 		{"name":"MyString","type":"string"}
		 * 	]}
		 */

		System.out.println(avroSchemaFromParquet);

		final var parquetSchemaFromAvro = AVRO_SCHEMA_CONVERTER.convert(avroSchema);
		/**
		 * message Out {
		 *   required int32 MyInteger;
		 *   required binary MyString (STRING);
		 * }
		 */
		System.out.println(parquetSchemaFromAvro);

		writeUsingExampleParquetWriter(parquetSchema);
		writeUsingAvroParquetWriter(avroSchema);

		readParquetFromFile("test.parquet");
		readParquetFromFile("avrotest.parquet");
	}

	private static void writeUsingExampleParquetWriter(MessageType parquetSchema) throws IOException {
		final var parquetWriter = buildWriter(parquetSchema);
		final SimpleGroup parquetRecord = createParquetGenericRecord(parquetSchema);
		parquetWriter.write(parquetRecord);
		parquetWriter.close();
	}

	private static ParquetWriter<Group> buildWriter(MessageType parquetSchema) {

		try {
			return ExampleParquetWriter.<Group>builder(new Path("test.parquet"))
					.withType(parquetSchema)
					.build();
		} catch (IOException e) {
			throw new RuntimeException(e);
		}
	}

	private static SimpleGroup createParquetGenericRecord(MessageType parquetSchema) {
		final var parquetRecord = new SimpleGroup(parquetSchema);
		parquetRecord.append("MyInteger", 1)
				.append("MyString", "string");
		return parquetRecord;
	}

	private static void writeUsingAvroParquetWriter(Schema avroSchema) throws IOException {
		final var avroParquetWriter = buildAvroParquetWriter(avroSchema);
		GenericRecord avroParquetRecord = createAvroGenericRecord(avroSchema);
		avroParquetWriter.write(avroParquetRecord);
		avroParquetWriter.close();
	}

	private static ParquetWriter<GenericRecord> buildAvroParquetWriter(Schema parquetSchema) {

		try {
			return AvroParquetWriter.<GenericRecord>builder(new Path("avrotest.parquet"))
					.withSchema(parquetSchema)
					.build();
		} catch (IOException e) {
			throw new RuntimeException(e);
		}
	}

	private static GenericRecord createAvroGenericRecord(Schema avroSchema) {
		GenericRecord avroParquetRecord = new GenericData.Record(avroSchema);
		avroParquetRecord.put("MyInteger", 1);
		avroParquetRecord.put("MyString", "string");
		return avroParquetRecord;
	}

	private static void readParquetFromFile(String fileName) throws IOException {
		ParquetReader<Group> reader = new ParquetReader<>(new Path(fileName), new GroupReadSupport());

		Group result = reader.read();
		final var myInteger = result.getInteger("MyInteger", 0);
		final var myString = result.getString("MyString", 0);
		System.out.println(myInteger);
		System.out.println(myString);
	}
}
```

## Reading with `ParquetReader`

Files are read with `ParquetReader` outputting:

```shell
1
string
1
string
```

## Reading with `parquet-cli`

```shell
parquet cat test.parquet    
{"MyInteger": 1, "MyString": "string"}
parquet cat avrotest.parquet
{"MyInteger": 1, "MyString": "string"}
```

# Simple schema with array

Parquet schema is similar to Protobuf, but not entirely. While they are very similar with primitive types, with nested types differences start to show. If I wanted to add an array of integers in Protobuf, I could define schema like:

```
message Out {
  repeated int32 integers = 1
}
```

While in Parquet it should be defined as per [documentation](https://github.com/apache/parquet-format/blob/master/LogicalTypes.md#lists): 

```
message Out {
  required/optional group integers (LIST) {
    repeated group list {
      required/optional int32 element;
    }
  }
}
```

However, there are nuances with arrays, Parquet and Avro.


## Hand written Avro Schema

```
{
	"type":"record",
	"name":"Out",
	"fields":[
		{
			"name":"Integers",
			"type":{"type":"array", "items": "int"}
		}
	]
}
```

## Hand written Parquet Schema

I'm going to write two schemas and in examples we'll see why.

Per documentation:

```
message Out {
  required group Integers (LIST) {
    repeated group list {
      required int32 element;
    }
  }
}
```

Deviates from documentation:

```
message Out {
  required group Integers (LIST) {
    repeated int32 array;
  }
}
```


## `AvroSchemaConverter` conversion from Avro to Parquet

As you can see, from Avro schema to Parquet creates a second schema from "Hand written Parquet Schema". Why?

```
message Out {
  required group Integers (LIST) {
    repeated int32 array;
  }
}
```

Also, notice that name of "repeated" type is array. This is important in further steps.


## `AvroSchemaConverter` conversion from Parquet to Avro

Converting first schema from "Hand written Parquet Schema":

```
{
  "type": "record",
  "name": "Out",
  "fields": [
    {
      "name": "Integers",
      "type": {
        "type": "array",
        "items": {
          "type": "record",
          "name": "list",
          "fields": [
            {
              "name": "element",
              "type": "int"
            }
          ]
        }
      }
    }
  ]
}
```

This is nothing like hand written Avro example.


Converting second schema from "Hand written Parquet Schema":

```
{
  "type": "record",
  "name": "Out",
  "fields": [
    {
      "name": "Integers",
      "type": {
        "type": "array",
        "items": "int"
      }
    }
  ]
}
```

As expected, it maps to exactly.

### More misalignments 

Avro has additional schema infering from Java objects functionality. Example:

```java
public static class Out {
	private List<Integer> Integers;

	public Out(List<Integer> integers) {
		Integers = integers;
	}

	public List<Integer> getIntegers() {
		return Integers;
	}
}
	
public static void main(String[] args) throws IOException {

	final var schemaString = ReflectData.get().getSchema(Out.class).toString();
	final var schema = new Schema.Parser().parse(schemaString);

	System.out.println(schema);
}
```

Running main prints:

```
{
  "type": "record",
  "name": "Out",
  "namespace": "com.test.fromjsontoparquet.TestTwo",
  "fields": [
    {
      "name": "Integers",
      "type": {
        "type": "array",
        "items": "int",
        "java-class": "java.util.List"
      }
    }
  ]
}
```

Again, this is very much the same as written by hand Avro schema and when writting Parquet schema (second example). However, this goes against what is documented in the Parquet format documentation. Why?

Another case - use `parquet-cli` command `convert`, which converts JSON file into parquet and inspect it's schema:

```shell

$ echo '{ "Integers": [1,2] }' > commandtest.json
$ parquet convert commandtest.json -o commandtest.parquet
$ parquet cat commandtest.parquet
> {"Integers": [1, 2]}
$ parquet schema commandtest.parquet
> 
{
  "type" : "record",
  "name" : "json",
  "fields" : [ {
    "name" : "Integers",
    "type" : {
      "type" : "array",
      "items" : "int"
    },
    "doc" : "Type inferred from '[1,2]'"
  } ]
}
```

Again, this corresponds to non-documented Parquet schema. Why? Why `parquet-cli` tool generates schemas, which do not comply with their own standards?

Lastly, in `org.apache.parquet.example.Paper` there is Dremel paper example built using Parquet Java classes:

```java
  public static final MessageType schema =
      new MessageType("Document",
          new PrimitiveType(REQUIRED, INT64, "DocId"),
          new GroupType(OPTIONAL, "Links",
              new PrimitiveType(REPEATED, INT64, "Backward"),
              new PrimitiveType(REPEATED, INT64, "Forward")
              ),
          new GroupType(REPEATED, "Name",
              new GroupType(REPEATED, "Language",
                  new PrimitiveType(REQUIRED, BINARY, "Code"),
                  new PrimitiveType(OPTIONAL, BINARY, "Country")),
              new PrimitiveType(OPTIONAL, BINARY, "Url")));
```

Which corresponds to Protobuf schema defined in [Dremel paper](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/36632.pdf):

```
message Document {
 required int64 DocId;
 optional group Links {
   repeated int64 Backward;
   repeated int64 Forward; 
 }
 repeated group Name {
   optional string Url; 
   repeated group Language {
     required string Code;
     optional string Country; 
   }
 }
}
 ```
 
 Taking inspiration, I can define similarly this test's schema:
 
 ```java
 final var parquetSchema =
     new MessageType("Out",
         new GroupType(Type.Repetition.REQUIRED, "Integers",
	         new PrimitiveType(Type.Repetition.REPEATED, PrimitiveType.PrimitiveTypeName.INT32, "array")
	)
     );
 ```
 
 Printing this schema:
 
 ```
 message Out {
  required group Integers {
    repeated int32 array;
  }
}
```

The main difference is that it has missing "(LIST)" type hint after "Integers". This shcema does not even work with `AvroSchemaConverter`, due to:

```java
java.lang.UnsupportedOperationException: REPEATED not supported outside LIST or MAP. Type: repeated int32 array
	at org.apache.parquet.avro.AvroSchemaConverter.convertFields(AvroSchemaConverter.java:292)
	at org.apache.parquet.avro.AvroSchemaConverter.convertField(AvroSchemaConverter.java:440)
	at org.apache.parquet.avro.AvroSchemaConverter.convertFields(AvroSchemaConverter.java:290)
	at org.apache.parquet.avro.AvroSchemaConverter.convert(AvroSchemaConverter.java:279)
```

## Full Code

```java
public class TestTwo {

	private static final AvroSchemaConverter AVRO_SCHEMA_CONVERTER = new AvroSchemaConverter(new Configuration());

	public static void main(String[] args) throws IOException {

		final var avroSchemaString = """
				{
					"type":"record",
					"name":"Out",
					"fields":[
						{
							"name":"Integers",
							"type":{"type":"array", "items": "int"}
						}
					]
				}""";
		final var avroSchema = new Schema.Parser().parse(avroSchemaString);

		final var parquetSchemaString = """
				message Out {
				  required group Integers (LIST) {
				    repeated int32 array;
				  }
				}""";
		final var parquetSchema = MessageTypeParser.parseMessageType(parquetSchemaString);
		System.out.println(parquetSchema);

		final var avroSchemaFromParquet = AVRO_SCHEMA_CONVERTER.convert(parquetSchema);
		System.out.println(avroSchemaFromParquet);

		final var parquetSchemaFromAvro = AVRO_SCHEMA_CONVERTER.convert(avroSchema);
		System.out.println(parquetSchemaFromAvro);

		writeUsingExampleParquetWriter(parquetSchema);
		writeUsingAvroParquetWriter(avroSchema);

		readParquetFromFile("test.parquet");
		readParquetFromFile("avrotest.parquet");
	}

	private static void writeUsingExampleParquetWriter(MessageType parquetSchema) throws IOException {
		final var parquetWriter = buildWriter(parquetSchema);
		final SimpleGroup parquetRecord = createParquetGenericRecord(parquetSchema);
		parquetWriter.write(parquetRecord);
		parquetWriter.close();
	}

	private static ParquetWriter<Group> buildWriter(MessageType parquetSchema) {

		try {
			return ExampleParquetWriter.<Group>builder(new Path("test.parquet"))
					.withType(parquetSchema)
					.build();
		} catch (IOException e) {
			throw new RuntimeException(e);
		}
	}

	private static SimpleGroup createParquetGenericRecord(MessageType parquetSchema) {
		final var parquetRecord = new SimpleGroup(parquetSchema);
		parquetRecord.addGroup("Integers")
				.append("array", 1)
				.append("array", 2);

		return parquetRecord;
	}

	private static void writeUsingAvroParquetWriter(Schema avroSchema) throws IOException {
		final var avroParquetWriter = buildAvroParquetWriter(avroSchema);
		GenericRecord avroParquetRecord = createAvroGenericRecord(avroSchema);
		avroParquetWriter.write(avroParquetRecord);
		avroParquetWriter.close();
	}

	private static GenericRecord createAvroGenericRecord(Schema avroSchema) {
		GenericRecord avroParquetRecord = new GenericData.Record(avroSchema);
		avroParquetRecord.put("Integers", List.of(1, 2));
		return avroParquetRecord;
	}

	private static ParquetWriter<GenericRecord> buildAvroParquetWriter(Schema parquetSchema) {

		try {
			return AvroParquetWriter.<GenericRecord>builder(new Path("avrotest.parquet"))
					.withSchema(parquetSchema)
					.build();
		} catch (IOException e) {
			throw new RuntimeException(e);
		}
	}

	private static void readParquetFromFile(String fileName) throws IOException {
		ParquetReader<Group> reader = new ParquetReader<>(new Path(fileName), new GroupReadSupport());

		Group result = reader.read();
		final var group = result.getGroup("Integers", 0);
		final var i1 = group.getInteger("array", 0);
		final var i2 = group.getInteger("array", 1);
		System.out.println(i1);
		System.out.println(i2);
	}
}
```


## Reading with `ParquetReader`

Files are read with `ParquetReader` outputting:

```shell
1
2
1
2
```

## Reading with `parquet-cli`

```shell
$ parquet cat test.parquet
{"Integers": [1, 2]}
$ parquet cat avrotest.parquet
{"Integers": [1, 2]}
```

## Bonus twist

Remember in "`AvroSchemaConverter` conversion from Avro to Parquet" section I've wrote: "Also, notice that name of "repeated" type is array. This is important in further steps". What happens if the name is not array? For example:

```java
final var parquetSchemaString = """
		message Out {
		  required group Integers (LIST) {
			repeated int32 whatever;
		  }
		}""";
```

And I've modified the `TestTwo` class by extracting Avro generations:

```java
public class TestTwo {

	private static final AvroSchemaConverter AVRO_SCHEMA_CONVERTER = new AvroSchemaConverter(new Configuration());

	public static void main(String[] args) throws IOException {

		final var parquetSchemaString = """
				message Out {
				  required group Integers (LIST) {
				    repeated int32 whatever;
				  }
				}""";
		final var parquetSchema = MessageTypeParser.parseMessageType(parquetSchemaString);
		System.out.println(parquetSchema);

		final var avroSchemaFromParquet = AVRO_SCHEMA_CONVERTER.convert(parquetSchema);
		System.out.println(avroSchemaFromParquet);

		writeUsingExampleParquetWriter(parquetSchema);

		readParquetFromFile("test.parquet");
	}

	private static void writeUsingExampleParquetWriter(MessageType parquetSchema) throws IOException {
		final var parquetWriter = buildWriter(parquetSchema);
		final SimpleGroup parquetRecord = createParquetGenericRecord(parquetSchema);
		parquetWriter.write(parquetRecord);
		parquetWriter.close();
	}

	private static ParquetWriter<Group> buildWriter(MessageType parquetSchema) {

		try {
			return ExampleParquetWriter.<Group>builder(new Path("test.parquet"))
					.withType(parquetSchema)
					.build();
		} catch (IOException e) {
			throw new RuntimeException(e);
		}
	}

	private static SimpleGroup createParquetGenericRecord(MessageType parquetSchema) {
		final var parquetRecord = new SimpleGroup(parquetSchema);
		parquetRecord.addGroup("Integers")
				.append("whatever", 1)
				.append("whatever", 2);

		return parquetRecord;
	}

	private static void readParquetFromFile(String fileName) throws IOException {
		ParquetReader<Group> reader = new ParquetReader<>(new Path(fileName), new GroupReadSupport());

		Group result = reader.read();
		final var group = result.getGroup("Integers", 0);
		final var i1 = group.getInteger("whatever", 0);
		final var i2 = group.getInteger("whatever", 1);
		System.out.println(i1);
		System.out.println(i2);
	}
}
```


Running `main` is successful and ParquetReader successfully reads the outcome. However, running `parquet-cli`:

```shell
$ parquet cat test.parquet
{"Integers": null}
```

`Integers` are null.

## Conclusion














# Simple schema with XXXXXXXXXXX

## Hand written Avro Schema

## Hand written Parquet Schema

## `AvroSchemaConverter` conversion from Avro to Parquet

## `AvroSchemaConverter` conversion from Parquet to Avro

## Full Code

## Reading with `ParquetReader`

## Reading with `parquet-cli`



# Simple schema with YYYYYYYYYYYYYYY

## Hand written Avro Schema

## Hand written Parquet Schema

## `AvroSchemaConverter` conversion from Avro to Parquet

## `AvroSchemaConverter` conversion from Parquet to Avro

## Full Code

## Reading with `ParquetReader`

## Reading with `parquet-cli`
















