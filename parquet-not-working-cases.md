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


# Simple Objects

Let's start with simple objects. In this test case I will also use `AvroSchemaConverter` to convert from Parquet to Avro and from Avro to Parquet in order to see, whether there are discrepancies between hand created and automatically created schemas.

Hand created avro schema:

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


Equivalent hand created Parquet schema:

```
message Out {
	required int32 MyInteger;
	required binary MyString (UTF8);
}
```



















