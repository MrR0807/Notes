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

I have found inconsistencies between how Avro and Parquet converts schemas, how values are serialized and deserialized, and how parquet cli tool interacts with written files. I wanted to document those cases for both my own sanity and to raise awareness of these cases.

# What

Each case will be started by defining both Avro and Parquet schemas by hand. They inspecting how they are automatically converted using `AvroSchemaConverter` into one another, use generic writting methods to serialise information and then deserialise it and lastly use `parquet-cli` to again read those files. I will start with simple cases and ramp up by adding complex types like lists and maps.

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

The only difference is that instead of `UTF8` it is `STRING`. 


## `AvroSchemaConverter` conversion from Parquet to Avro

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




















