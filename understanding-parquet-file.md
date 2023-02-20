# Parquet file anatomy

When I've started delving into Parquet, I quickly found out that there is a huge gap of concise and in depth overview of what Parquet really is, why it exists and what kind of problems it solves. To my disappointment, official Parquet documentation is quite shallow and lacks any depth. There aren't any books solely on this particular topic as well. This is why I've decided to try to compile different insights, knowledge and my own observations into this document.

Let's start our journey with quotes, which describe Parquet.

From Parquet [oficial documentation](https://parquet.apache.org/):

> Apache Parquet is an open source, column-oriented data file format designed for efficient data storage and retrieval. It provides efficient data compression and encoding schemes with enhanced performance to handle complex data in bulk.

Take from [Wiki](https://en.wikipedia.org/wiki/Apache_Parquet):

> Apache Parquet is a free and open-source column-oriented data storage format in the Apache Hadoop ecosystem. It is similar to RCFile and ORC, the other columnar-storage file formats in Hadoop, and is compatible with most of the data processing frameworks around Hadoop. It provides efficient data compression and encoding schemes with enhanced performance to handle complex data in bulk.

From book [Hadoop: The Definitive Guide](https://www.oreilly.com/library/view/hadoop-the-definitive/9780596521974/)

> Apache Parquet is a columnar storage format that can efficiently store nested data.

What is not explicitly emphasized that Parquet is built on top of several solutions, which are blended together into what's know as Parquet. I think each component should be addressed independently before trying to understand aggregate.

Components that I'm going to address:
* MapReduce
* File's metadata importance
* Columnar data layout
* Nested columnar data layout (Google's Dremel)
* Encoding (e.g. Avro, Thrift)

## MapReduce

TODO.

Google BigQuery book why Parquet was created.

## File's metadata importance

In this section will use the idea from Designing Data-Intensive Applications book[1] to portrait metadata usefulness. In named book, author explores metadata concept by introducing a simple, bash database.  


Instead of `bash` scripts and thought practices, I'll build a simple database in Java and explore simplified metadata and indexes. Also, I will not implement compaction, because Parquet does not formulate such practices. However, those practices could be implemented in downstream specialised databases for which Parquet files acts as an input.




One of key components in Parquet is metadata. Metadata allows for quick seek of information.....

log-structured storage engines, because it is very familiar to Parquet compared to something like page-oriented storage engines such as B-trees.

There are two families of storage engines: log-structured storage engines, and page-oriented storage engines such as B-trees[1]. In upcoming sections I'll build a simple log-structured storage to emphasize metadata.

### Simple database

On the most fundamental level, a database needs to do two things: when you give it some data, it should store the data, and when you ask it again later, it should give the data back to you[1].

Let's start with simple case:

```java
public class SimpleDatabase {

	private static final Path DATABASE_PATH = Path.of("database.txt");
	private static final String ENTRY_TEMPLATE = "index:%d{%s}";
	private static final Pattern ENTRY_PATTERN_MATCHER = Pattern.compile("""
			index:(\\d+)\\{"name":"([\\w\\s]+)",\\s"age":(\\d+),\\s"salary":(\\d+)""");

	public static void main(String[] args) throws IOException {

		final var simpleDatabase = new SimpleDatabase();
		simpleDatabase.writeToDatabase(1, """
				"name":"John", "age":26, "salary":1000""");
		simpleDatabase.writeToDatabase(2, """
				"name":"John", "age":27, "salary":2000""");
		simpleDatabase.writeToDatabase(3, """
				"name":"John", "age":28, "salary":3000""");
		simpleDatabase.writeToDatabase(4, """
				"name":"Marry", "age":26, "salary":1000""");
		simpleDatabase.writeToDatabase(5, """
				"name":"Marry", "age":27, "salary":2000""");


		System.out.println("-".repeat(10));
		System.out.println(simpleDatabase.readAllFromDatabase());
		System.out.println("-".repeat(10));


		System.out.println("*".repeat(10));
		System.out.println(simpleDatabase.readBy(3));
		System.out.println("*".repeat(10));
	}

	private void writeToDatabase(long index, String data) throws IOException {

		final var entry = ENTRY_TEMPLATE.formatted(index, data);
		Files.write(DATABASE_PATH, List.of(entry), StandardOpenOption.APPEND, StandardOpenOption.CREATE);
	}

	private List<String> readAllFromDatabase() throws IOException {
		return Files.readAllLines(DATABASE_PATH);
	}

	private Optional<Entry> readBy(long indexToFind) throws IOException {

		try (var lines = Files.lines(DATABASE_PATH)) {
			return lines
					.map(SimpleDatabase::fromLine)
					.filter(entry -> entry.index == indexToFind)
					.findFirst();
		}
	}

	private static Entry fromLine(String line) {

		final var matcher = ENTRY_PATTERN_MATCHER.matcher(line);

		if (matcher.find()) {

			final var index = matcher.group(1);
			final var name = matcher.group(2);
			final var age = matcher.group(3);
			final var salary = matcher.group(4);

			return new Entry(Long.parseLong(index), name, Integer.parseInt(age), Integer.parseInt(salary));
		}

		throw new RuntimeException("Database is corrupted");
	}

	private record Entry(long index, String name, int age, int salary) {
	}
}
```

Running `main` should print:

```
----------
[index:1{"name":"John", "age":26, "salary":1000}, index:2{"name":"John", "age":27, "salary":2000}, index:3{"name":"John", "age":28, "salary":3000}, index:4{"name":"Marry", "age":26, "salary":1000}, index:5{"name":"Marry", "age":27, "salary":2000}]
----------
**********
Optional[Entry[index=3, name=John, age=28, salary=3000]]
**********
```

And there should be a new file - `database.txt` with content:

```
index:1{"name":"John", "age":26, "salary":1000}
index:2{"name":"John", "age":27, "salary":2000}
index:3{"name":"John", "age":28, "salary":3000}
index:4{"name":"Marry", "age":26, "salary":1000}
index:5{"name":"Marry", "age":27, "salary":2000}
```

This implementation creates a key-value store. The underlying storage format is very simple: a text file where each line contains a key-value pair, separated by a comma (roughly like a CSV file, ignoring escaping issues).  Every call to `writeToDatabase` appends to the end of the file, so if you update a key several times, the old versions of the value are not overwritten — you need to look at the last occurrence of a key in a file to find the latest value[1]. This is pretty much how Parquet represents its data as well as in, it just appends data to the end of the file without trying to update by some key or executing any other complicated data manipulation like compaction[2] or Apache Iceberg's position delete files[3].

The `writeToDatabase` has pretty good performance for something that is so simple, because appending to a file is generally very efficient[1]. Similarly to what `writeToDatabase` does, many databases internally use a log, which is an append-only data file (e.g. Write Ahead Log). Real databases have more issues to deal with (such as concurrency control, reclaiming disk space so that the log doesn’t grow forever, and handling errors and partially written records), but the basic principle is the same[1].

On the other hand, `readAllFromDatabase` and `readBy` has a terrible performance if you have a large number of records in your database:
* `readAllFromDatabase` actually loads all of the data to memory. This means that big files (e.g. terabyte size) just won't fit. 
* `readBy` is smarter, because it is streaming information without having to load it to memory, unfortunately it does this sequentially, essentially doing full table scan talking in SQL terms. In algorithmic terms, the cost of a lookup is `O(n)`: if you double the number of records n in your database, a lookup takes twice as long[1].

Let's generate some amount of data for this database and try small searching experiment:

```java
public static void main(String[] args) throws IOException {

	final var simpleDatabase = new SimpleDatabase();

	for (int i = 0; i < (Integer.MAX_VALUE / 1000); i++) {

		simpleDatabase.writeToDatabase(i, """
			"name":"John", "age":26, "salary":2147483646""");
	}
}
```

For me, this has generated a file "weighting" around 130 MB.

Let's try searching the the first and the last entries:

```java
public static void main(String[] args) throws IOException {

	final var simpleDatabase = new SimpleDatabase();

	final var now = Instant.now();
	System.out.println(simpleDatabase.readBy(2147482));
	final var after = Instant.now();

	System.out.println(Duration.between(now, after).toMillis());
}
```

Searching for the last entry (with `index:2147482`) takes around `1000 - 1500 ms`. While searching for the first entry it takes about `30 - 40 ms`.

### Index

In order to efficiently find the value for a particular key in the database, we need a different data structure: an index. The general idea behind them is to keep some additional metadata on the side, which acts as a signpost and helps you to locate the data you want. If you want to search the same data in several different ways, you may need several different indexes on different parts of the data[1].

An index is an additional structure that is derived from the primary data. Many databases allow you to add and remove indexes, and this doesn’t affect the contents of the database; it only affects the performance of queries. Maintaining additional structures incurs overhead, especially on writes. For writes, it’s hard to beat the performance of simply appending to a file, because that’s the simplest possible write operation. Any kind of index usually slows down writes, because the index also needs to be updated every time data is written[1].

#### Hash Indexes

Let’s start with indexes for key-value data. This is not the only kind of data you can index, but it’s very common, and it’s a useful building block for more complex indexes[1].

Key-value stores are quite similar to the dictionary type that you can find in most programming languages, and which is usually implemented as a hash map (hash table)[1].

Let’s say our data storage consists only of appending to a file, as in the preceding example. Then the simplest possible indexing strategy is this: keep an in-memory hash map where every key is mapped to a byte offset in the data file - the location at which the value can be found. Whenever you append a new key-value pair to the file, you also update the hash map to reflect the offset of the data you just wrote (this works both for inserting new keys and for updating existing keys). When you want to look up a value, use the hash map to find the offset in the data file, seek to that location, and read the value [1].


#### Implementation

In order to know offset position during our writes and then read from given offset positions, I will have to rewrite Java database from ground up. This is because `Files` abstractions lack any such controls and deemed them too low level. There are a couple of options for rewrite:
* `RandomAccessFile`[4].
* `SeekableByteChannel`[5].

It is clearly stated in stackoverflow post that `java.nio` with `FileChannel` is faster by about >250% compared with `FileInputStream/FileOuputStream`[6], however, the difference between `RandomAccessFile` and `SeekableByteChannel` is not conclusive or well documented. I have found several instances, which claim that `SeekableByteChannel` is faster[7], but this is yet to be tested in another time. Anyway, I have chose to use `SeekableByteChannel`.

```java
public class DatabaseInternals implements AutoCloseable {

	private static final int DEFAULT_BUFFER_SIZE = 8192;
	private static final int END_OF_THE_LINE_HEX = 0x0A;
	private static final int COLON_HEX = 0x3a;
	private static final int CURLY_BRACELETS_HEX = 0x7b;
	private final int lastBlockLength;
	private final long totalBlockCount;
	private final long fileSize;
	private final SeekableByteChannel seekableByteChannel;

	private static final Pattern ENTRY_PATTERN_MATCHER = Pattern.compile("""
			index:(\\d+)\\{"name":"([\\w\\s]+)",\\s"age":(\\d+),\\s"salary":(\\d+)""");

	public DatabaseInternals(Path file) {

		try {
			this.seekableByteChannel = Files.newByteChannel(file, StandardOpenOption.READ, StandardOpenOption.WRITE, StandardOpenOption.CREATE);
			final var size = seekableByteChannel.size();

			int lastBlockLength = (int) size % DEFAULT_BUFFER_SIZE;
			long totalBlockCount;

			if (lastBlockLength > 0) {
				totalBlockCount = size / DEFAULT_BUFFER_SIZE + 1;
			} else {
				totalBlockCount = size / DEFAULT_BUFFER_SIZE;
				if (size > 0) {
					lastBlockLength = DEFAULT_BUFFER_SIZE;
				}
			}

			this.lastBlockLength = lastBlockLength;
			this.totalBlockCount = totalBlockCount;
			this.fileSize = size;
		} catch (IOException e) {
			throw new RuntimeException(e);
		}
	}

	public List<Entry> readBlock(long offset) throws IOException {

		final var buffer = ByteBuffer.allocate(DEFAULT_BUFFER_SIZE);
		seekableByteChannel.position(offset);
		seekableByteChannel.read(buffer);
		buffer.flip();
		final var entries = readRecord(buffer);
		buffer.clear();
		return entries;
	}

	public long readLastIndex() throws IOException {

		if (seekableByteChannel.size() == 0) {
			return 0;
		}

		final var buffer = ByteBuffer.allocate(DEFAULT_BUFFER_SIZE);
		if (fileSize > DEFAULT_BUFFER_SIZE) {
			seekableByteChannel.position(DEFAULT_BUFFER_SIZE * (totalBlockCount - 2));
			seekableByteChannel.read(buffer);
		} else {
			seekableByteChannel.position(0);
			seekableByteChannel.read(buffer);
		}

		buffer.flip();
		final var lastRecordPosition = readLastRecordPosition(buffer);
		buffer.position( (int) lastRecordPosition.startOffset);
		return readLastRecordIndex(buffer);
	}

	private static EntryPosition readLastRecordPosition(ByteBuffer buffer) {
		int lineStart = buffer.position() - 1;
		int lineEnd = 0;

		while (buffer.hasRemaining()) {

			final var x = buffer.get();
			if (END_OF_THE_LINE_HEX == x) {
				lineEnd = buffer.position();
				byte[] s = new byte[lineEnd - lineStart];
				final var slice = buffer.slice(lineStart, lineEnd - lineStart);
				slice.get(s);
			}

			if (buffer.position() == lineEnd + 1) {
				lineStart = buffer.position() - 1;
			}
		}

		return new EntryPosition(lineStart, lineEnd);
	}

	private static long readLastRecordIndex(ByteBuffer buffer) {
		int startOfIndex = 0;
		int endOfIndex = 0;

		while (buffer.hasRemaining()) {
			final var b = buffer.get();

			if (b == COLON_HEX) {
				startOfIndex = buffer.position();
			}

			if (b == CURLY_BRACELETS_HEX) {
				endOfIndex = buffer.position() - 1;
				break;
			}
		}

		final var slice = buffer.slice(startOfIndex, endOfIndex - startOfIndex);
		byte[] indexValue = new byte[endOfIndex - startOfIndex];
		slice.get(indexValue);
		return Long.parseLong(new String(indexValue, StandardCharsets.UTF_8));
	}

	private static List<Entry> readRecord(ByteBuffer buffer) {
		int lineStart = buffer.position() - 1;
		int lineEnd = 0;
		final var entries = new ArrayList<Entry>();

		while (buffer.hasRemaining()) {

			final var x = buffer.get();
			if (END_OF_THE_LINE_HEX == x) {
				lineEnd = buffer.position();
				byte[] lineBytes = new byte[lineEnd - lineStart];
				final var slice = buffer.slice(lineStart, lineEnd - lineStart);
				slice.get(lineBytes);
				final var entry = fromLine(new String(lineBytes, StandardCharsets.UTF_8));
				entries.add(entry);
			}

			if (buffer.position() == lineEnd + 1) {
				lineStart = buffer.position() - 1;
			}
		}
		return entries;
	}

	private static Entry fromLine(String line) {

		final var matcher = ENTRY_PATTERN_MATCHER.matcher(line);

		if (matcher.find()) {

			final var index = matcher.group(1);
			final var name = matcher.group(2);
			final var age = matcher.group(3);
			final var salary = matcher.group(4);

			return new Entry(Long.parseLong(index), name, Integer.parseInt(age), Integer.parseInt(salary));
		}

		throw new RuntimeException("Database is corrupted");
	}

	private record Entry(long index, String name, int age, int salary) {

	}
	public record EntryPosition(long startOffset, long endOffset) {

	}

	private static final String ENTRY_TEMPLATE = "index:%d{%s}\n";

	public EntryPosition write(long index, String data) throws IOException {

		final var entry = ENTRY_TEMPLATE.formatted(index, data);
		long startOffset = seekableByteChannel.position();
		seekableByteChannel.write(ByteBuffer.wrap(entry.getBytes()));
		return new EntryPosition(startOffset, seekableByteChannel.position());
	}


	@Override
	public void close() throws Exception {

		seekableByteChannel.close();
	}
}
```

I will not explain the implementation details and if you want to read more about `java.nio.channels` usage there are great blogs[8][9][10][11] and a book[12]. Also, this implementation is not super optimised and readable, but I might improve it over time. For now, it is a good starting point.

Let's generate same data, but this time with metadata:

```java
public static void main(String[] args) throws Exception {

	try (var simpleDatabase = new SimpleDatabase(new DatabaseInternals(DATABASE_PATH))) {

		for (int i = 0; i < (Integer.MAX_VALUE / 1000); i++) {

			final var write = simpleDatabase.databaseInternals.write(++simpleDatabase.lastIndex, """
				"name":"John", "age":26, "salary":2147483646""");
			simpleDatabase.indexOffsetMap.put(simpleDatabase.lastIndex, write.startOffset());
		}

		serializeMetadata(simpleDatabase.indexOffsetMap);
	}
}

private static void serializeMetadata(Map<Long, Long> metadata) throws IOException {

	try (final var out = new ObjectOutputStream(new FileOutputStream(METADATA_FILE_NAME))) {
		out.writeObject(metadata);
	}
}
```

Again, the `database.txt` file is the same size (around 130MB) and has the same content, however, there is a new file - `metadata.ser` which weights around 60MB. If you think that is a lot for a metadata file, well, we had a database in production, where indexes were 7x the size of the table. True story. Anyway, we will shrink it down in upcoming iterations.

```java
public class SimpleDatabase implements AutoCloseable {

	private static final Path DATABASE_PATH = Path.of("database.txt");
	private static final String METADATA_FILE_NAME = "metadata.ser";
	private final Map<Long, Long> indexOffsetMap;

	private long lastIndex;
	private final DatabaseInternals databaseInternals;

	public SimpleDatabase(DatabaseInternals databaseInternals) throws IOException {

		this.databaseInternals = databaseInternals;
		this.lastIndex =  databaseInternals.readLastIndex();
		this.indexOffsetMap = readMetadata();
	}

	private static Map<Long, Long> readMetadata() {
		try (final var in = new ObjectInputStream(new FileInputStream(METADATA_FILE_NAME))){
			return (Map<Long, Long>) in.readObject();
		} catch (IOException | ClassNotFoundException e) {
			// Do nothing;
		}
		return new HashMap<>();
	}

	public static void main(String[] args) throws Exception {

		try (var simpleDatabase = new SimpleDatabase(new DatabaseInternals(DATABASE_PATH))) {

			final var now = Instant.now();
			final var offset = simpleDatabase.indexOffsetMap.get(2147483L);
			simpleDatabase.databaseInternals.readBlock(offset);
			final var after = Instant.now();
			System.out.println("Reading data: " + Duration.between(now, after).toMillis());
		}
	}

	private static void serializeMetadata(Map<Long, Long> metadata) throws IOException {

		try (final var out = new ObjectOutputStream(new FileOutputStream(METADATA_FILE_NAME))) {
			out.writeObject(metadata);
		}
	}

	@Override
	public void close() throws Exception {
		databaseInternals.close();
	}
}
```

If I read the last entry `2147483` or the first, the speed is pretty much constant - data is fetched between `10 - 20 ms`. This is possible, because file is no longer being traversed from start to finish. The database only needs to fetch offset according to index and seek file to the exact position. Blazing fast. The downside, as already stated in "Encoding" chapter, Java serialization and desrialization framework is notoriously slow. Let's time how long it takes for in-memory map to get deserialized along with look up of last entry:

```java
public SimpleDatabase(DatabaseInternals databaseInternals) throws IOException {

	this.databaseInternals = databaseInternals;
	this.lastIndex =  databaseInternals.readLastIndex();
	final var now = Instant.now();
	this.indexOffsetMap = readMetadata();
	final var after = Instant.now();
	System.out.println("Reading metadata: " + Duration.between(now, after).toMillis());
}
```

Running several times, for me, it shows that preparing in-memory metadata hashmap takes about `14000 ms`. There are several ways to improve this:
* Because indexes are ordered we no longer need to keep an index of all the keys in memory, but only every couple of hundreds for example and read more data than required. This will shrink the `metadata.ser` however will slightly increase read part.
* Use a more advanced encoding framework, e.g. Avro.

#### Serializing and deserializing metadata with Avro

Avro, as we've found out (in "Encoding" chapter), is one of the fastest encoding frameworks. Let's try to leverage it and see whether it improves our Simple Database startup.

Let's firstly rewrite all data so we have offset metadata in Avro format. Full class:

```java
public class SimpleDatabaseWithAvro implements AutoCloseable {

	private static final Path DATABASE_PATH = Path.of("database.txt");
	private static final String METADATA_FILE_NAME = "metadata.avro";
	private static final Path METADATA_PATH = Path.of(METADATA_FILE_NAME);
	private final Map<Utf8, Long> avroIndexOffsetMap;

	private long lastIndex;
	private final DatabaseInternals databaseInternals;

	private static final Schema SCHEMA = new Schema.Parser().parse("""
			{
			  "type": "map",
			  "values": "long",
			  "default": {}
			}""");

	public SimpleDatabaseWithAvro(DatabaseInternals databaseInternals) throws IOException {

		this.databaseInternals = databaseInternals;
		this.lastIndex =  databaseInternals.readLastIndex();
		final var now = Instant.now();
		this.avroIndexOffsetMap = readMetadata();
		final var after = Instant.now();
		System.out.println("Reading metadata: " + Duration.between(now, after).toMillis());
	}

	private static Map<Utf8, Long> readMetadata() throws IOException {
		if (METADATA_PATH.toFile().exists()) {

			final var reader = new GenericDatumReader<Map>(SCHEMA);
			try (final var fileReader = new DataFileReader<>(Path.of("metadata.avro").toFile(), reader)) {
				return (Map<Utf8, Long>) fileReader.next();
			}
		} else {

			return new HashMap<>();
		}
	}

	public static void main(String[] args) throws Exception {

		final var indexOffsetMap = new HashMap<Long, Long>();

		try (var simpleDatabase = new SimpleDatabaseWithAvro(new DatabaseInternals(DATABASE_PATH))) {

			for (int i = 0; i < (Integer.MAX_VALUE / 1000); i++) {

				final var write = simpleDatabase.databaseInternals.write(++simpleDatabase.lastIndex, """
					"name":"John", "age":26, "salary":2147483646""");
				indexOffsetMap.put(simpleDatabase.lastIndex, write.startOffset());
			}

			serializeMetadata(indexOffsetMap);
		}
	}

	private static void serializeMetadata(Map<Long, Long> metadata) throws IOException {

		final var metadataWriter = new GenericDatumWriter<Map>(SCHEMA);

		try (final var metadataFileWriter = new DataFileWriter<>(metadataWriter)) {
			metadataFileWriter.create(SCHEMA, METADATA_PATH.toFile());
			metadataFileWriter.append(metadata);
		}
	}

	@Override
	public void close() throws Exception {
		databaseInternals.close();
	}
}
```

**NOTE!** You might have noticed that I write data as `Map<Long, Long>` but read as `Map<Utf8, Long>`. Well, Avro, according to its documentation assumes that all map keys are to be strings.

And reading part. I'm just adding different main, the class is absolutely the same:


```java
public static void main(String[] args) throws Exception {

	try (var simpleDatabase = new SimpleDatabaseWithAvro(new DatabaseInternals(DATABASE_PATH))) {

		final var now = Instant.now();
		final var offset = simpleDatabase.avroIndexOffsetMap.get(new Utf8("2147483"));
		final var entries = simpleDatabase.databaseInternals.readBlock(offset);
		System.out.println(entries);
		final var after = Instant.now();
		System.out.println("Reading data: " + Duration.between(now, after).toMillis());
	}
}
```

Running this several times returns me that metadata is read in `800 - 1500 ms`. This is a 10-18x improvement. Remember from "Columnar data layout" chapter, shrinking the size of metadata file is not to primarily save space, but to optimise disk transfer part. Oh, and `metadata.avro` file is about 25 MB.

#### Ranges of indexes








### Resources

1. [Designing Data-Intensive Applications](https://www.amazon.com/Designing-Data-Intensive-Applications-Reliable-Maintainable/dp/1449373321)
2. [Topic Compaction](https://developer.confluent.io/learn-kafka/architecture/compaction/)
3. [Apache Iceberg:Position Delete Files](https://iceberg.apache.org/spec/#position-delete-files)
4. [Java Documentation. RandomAccessFile](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/io/RandomAccessFile.html)
5. [Java Documentation. SeekableByteChannel](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/nio/channels/SeekableByteChannel.html)
6. [Java NIO FileChannel versus FileOutputstream performance / usefulness](https://stackoverflow.com/questions/1605332/java-nio-filechannel-versus-fileoutputstream-performance-usefulness)
7. https://mechanical-sympathy.blogspot.com/2011/12/java-sequential-io-performance.html
8. https://www.happycoders.eu/java/filechannel-memory-mapped-io-locks/
9. https://blogs.oracle.com/javamagazine/post/java-nio-nio2-buffers-channels-async-future-callback
10. https://docs.oracle.com/javase/tutorial/essential/io/file.html
11. [Java Documentation. Package java.nio.channels] https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/nio/channels/package-summary.html
12. [Java NIO](https://www.oreilly.com/library/view/java-nio/0596002882/)







## Row, Columnar, Hybrid data layouts

Say we have a familiar, traditional database layout (or CSV file for that matter) with entries like so:

| Id  | Name  | Age | Salary |
|-----|-------|-----|--------|
| 1   | John  | 26  | 1000   |
| 2   | Adam  | 41  | 2000   |
| 3   | Eve   | 29  | 2500   |
| 4   | Maria | 55  | 1500   |
| 5   | Chris | 67  | 3000   |
| 6   | Emma  | 80  | 3500   |
| 7   | Ava   | 18  | 10000  |
| 8   | Liam  | 19  | 10000  |
| 9   | Lucas | 37  | 1000   |
| 10  | Peter | 61  | 7500   |

This representation of information is provided in 2D. Before data gets written to physical disk it goes through several stages[2]:
* Linearization - from 2D data to a 1D sequence of values (e.g. `1,John,26,1000,2,Adam,41,2000,3,Eve,29,2500,4,Maria,55,1500...`).
* Serialization - from a 1D sequence of values to bytes on virtual pages (e.g. `00110001010010100110111101101000011011100011001000110110...`).
* Devirtualization - from virtual pages to physical pages.
* Materialization - from physical pages to storage devices.

The order of the data does not matter in theory, it can be `John,26,1,1000,Adam,41,2,2000...` or `John,Adam,Even,1,2,3...`,  as long as we can find it efficiently and rebuild it back via what is called tuple reconstruction or recombination.

### File Systems

To further proceed with row, columnar, hybrid analysis we firstly have to start at the beginning - File System. Thus a little a bit of theory. Thus a little a bit of theory.

The main purpose of computers is to create, manipulate, store, and retrieve data. A file system provides the machinery to support these tasks. At the highest level a file system is a way to organize, store, retrieve, and manage information on a permanent storage medium such as a disk[7].

When discussing file systems there are many terms for referring to certain concepts, and so it is necessary to define how we will refer to the specific concepts that make up a file system[7]. I will not list all of them, but only few which are required for this document:
* Disk - A permanent storage medium of a certain size. A disk also has a sector or block size, which is the minimum unit that the disk can read or write. The block size of most modern hard disks is 512 bytes.
* Block - **The smallest readable/writable unit by a disk or file system.** Everything a file system does is composed of operations done on blocks. A file system block is always the same size as or larger (in integer multiples) than the disk block size.
* Partition - A subset of all the blocks on a disk. A disk can have several partitions.
* Volume - The name we give to a collection of blocks on some storage medium (i.e., a disk). That is, a volume may be all of the blocks on a single disk, some portion of the total number of blocks on a disk, or it may even span multiple disks and be all the blocks on several disks. The term “volume” is used to refer to a disk or partition that has been initialized with a file system.
* I-node - The place where a file system stores all the necessary metadata about a file. Examples of information stored in i-nodes are the last access time of the file, the type, the creator, a version number, and a reference to the directory that contains the file. However, the most important information stored in an i-node is the connection to the data in the file (i.e., where it is on disk).
* File - The primary functionality that all file systems must provide is a way to store a named piece of data and to later retrieve that data using the name given to it. A file is where a program stores data permanently. In its simplest form a file stores a single piece of information. 

#### Retrieving file's data

At this point we have several key concepts cleared up. Files are streams of data. Files' metadata is stored in i-node, which among other things, *knows* where the file is kept on the disk. Disk is made of blocks. Blocks are the **smallest unit that the disk can read or write**.

Say our blocks are made of 1024 bytes. Image a request to read from position 4096 of a file. We need to find the fourth block of the file because the file position, 4096, divided by the file system block size, is 4. The i-node contains a list of blocks that make up the file and it can tell us the disk address of the fourth block of the file. Then the file system must ask the disk to read that block. Finally, having retrieved the data, the file system can pass the data back to the user[7].

We simplified this example quite a bit, but the basic idea is always the same. Given a request for data at some position in a file, the file system must translate that logical position to a physical disk location, request that block from the disk, and then pass the data back to the user[7].

When a request is made to read (or write) data that is not on a file system block boundary, the file system must round down the file position to the beginning of a block. Then when the file system copies data to/from the block, it must add in the offset from the start of the block of the original position. For example, if we used the file offset 4732 instead of 4096, **we would still need to read the fourth block of the file.** But after getting the fourth block, we would use the data at byte offset 636 (4732 - 4096) within the fourth block[7].

When a request for I/O spans multiple blocks (such as a read for 8192 bytes), the file system must find the location for many blocks. If the file system has done a good job, **the blocks will be contiguous on disk. Requests for contiguous blocks on disk improve the efficiency of doing I/O to disk. The fastest thing a disk drive can do is to read or write large contiguous regions of disk blocks, and so file systems always strive to arrange file data as contiguously as possible.**

In short, there are two key concepts to remember:
* Even if we need partial data from a given block, we still need to read all block's data.
* Reading/writing in sequence (contiguous) improves the efficiency of doing I/O from/to disk.

Remember when I said that the order of the data does not matter, well it does, but it heavily depends on how the file systems are utilised.

### Columnar vs Row layout

Let's get back to our example and take two extremes how data could be stored. 

As already stated, data from 2D can be linearized as `1,John,26,1000,2,Adam,41,2000,3,Eve,29,2500,4,Maria,55,1500...`. This is called row oriented layout and common row oriented databases are PostgreSQL or MySQL. 
The other extreme is to linearize each row vertically: `1,2,3,4,5,6,7,8,9,10,John,Adam,Eve,Maria`. This is called column oriented layout and common column oriented databases are Google's BigQuery or Amazon's Redshift.

#### Row oriented layout

Row-oriented database management systems store data in records or rows. Their layout is quite close to the tabular data representation, where every row has the same set of fields. This approach works well for cases where several fields constitute the record (name, birth date, and a phone number) uniquely identified by the key (in this example, a monotonically incremented number). All fields representing a single user record are often read together. When creating records (for example, when the user fills out a registration form), we write them together as well. At the same time, each field can be modified individually. This is great for cases when we’d like to access an entire user record, but makes queries accessing individual fields of multiple user records (for example, queries fetching only the phone numbers) more expensive, since data for the other fields will be paged in as well [9].

##### Practical examples

Say each disk block can contain 4 values (int, string, etc.). Our table data would be stored on a disk in a row oriented database in order row by row like this:

| Block 1           | Block 2           | Block 3           | ... | Block 10            |
|-------------------|-------------------|-------------------|-----|---------------------|
| 1, John, 26, 1000 | 2, Adam, 41, 2000 | 3, Eve, 29, 2500  |     | 10, Peter, 61, 7500 |

This allows the database write a row quickly because, all that needs to be done to write to it is to tack on another row to the end of the data (disk block 11):

| Block 1           | Block 2           | Block 3           | ... | Block 10            | Block 11             |
|-------------------|-------------------|-------------------|-----|---------------------|----------------------|
| 1, John, 26, 1000 | 2, Adam, 41, 2000 | 3, Eve, 29, 2500  |     | 10, Peter, 61, 7500 | 11, Monica, 55, 4500 |

```sql
UPDATE TABLE
SET Age = 62, Salary = 8000
WHERE Id = 10
```

If I need to update both age and salary for a particular person by id, all the data is in one block. That means my update operation will be extremely efficient.

Same goes for selecting particular person and accessing all data:

```sql
SELECT * 
FROM TABLE
WHERE Id = 10
```

However, what happens when I need to calculate an average of all salaries?

```sql
SELECT AVG(Salary)
FROM TABLE
```

It will require the database to read in significantly more data, as both the needed attributes and the surrounding attributes stored in the same blocks need to be read. Not to mention going over all blocks of data.

Lastly, let’s assume a Disk can only hold enough bytes of data for three blocks to be stored on each disk. In a row oriented database the table above would be stored as:

| Disk 1                 | Disk 2                 | Disk 3                 |
|------------------------|------------------------|------------------------|
| Block1, Block2, Block3 | Block4, Block5, Block6 | Block7, Block8, Block9 |

To get the salary average, the database would need to look through all three disks, which might not even be co-located adding additional network latency.

#### Column oriented layout

Column-oriented database management systems partition data vertically (i.e., by column) instead of storing it in rows. Here, values for the same column are stored contiguously on disk (as opposed to storing rows contiguously as in the previous example). For example, if we store historical stock market prices, price quotes are stored together. Storing values for different columns in separate files or file segments allows efficient queries by column, since they can be read in one pass rather than consuming entire rows and discarding data for columns that weren’t queried (like example with salary average).
Column-oriented stores are a good fit for **analytical workloads** that compute aggregates, such as finding trends, computing average values, etc. Processing complex aggregates can be used in cases when logical records have multiple fields, but some of them (in this case, price quotes) have different importance and are often consumed together[9].

##### Practical examples

Again, the same conditions stand - each disk block can store 4 values. Our table data would be stored on a disk in a column oriented database in order column by column like this:

| Block 1           | Block 2           | Block 3           | ... | Block 10            |
|-------------------|-------------------|-------------------|-----|---------------------|
| 1, 2, 3, 4 | 5, 6, 7, 8 | 9, 10, John, Adam  |     | 10000, 10000, 1000, 7500 |

Writing to a column store would require to navigate to each block holding last entries and plugging data into them. This is obviously less efficient than row layout.

```sql
UPDATE TABLE
SET Age = 62, Salary = 8000
WHERE Id = 10
```

Would require to traverse each block, find entry's correct values to update. Again, inefficient.

```sql
SELECT * 
FROM TABLE
WHERE Id = 10
```

Selecting all parameters for particular id is, again, same problem.

However, what happens when I need to calculate an average of all salaries?

```sql
SELECT AVG(Salary)
FROM TABLE
```

It is going to be blazing fast, because data will be contiguous without needing to read all parameters and all blocks like it was with row layout.

Similar thing when data is distributed through several disks. It will only need to access one, instead of traversing all of them.

#### Columnar vs Row layout simple conclusion

To decide whether to use a columnor a row-oriented store, you need to understand your **access patterns**. 

If data is stored on magnetic disk, then if a query needs to access only a single record (i.e., all or some of the attributes of a single row of a table), a column-store will have to seek several times (to all columns/files of the table referenced in the query) to read just this single record. However, if a query needs to access many records, then large swaths of entire columns can be read, amortizing the seeks to the different columns. In a conventional row-store, in contrast, if a query needs to access a single record, only one seek is needed as the whole record is stored contiguously, and
the overhead of reading all the attributes of the record (rather than just the relevant attributes requested by the current query) will be negligible relative to the seek time. However, as more and more records are accessed, the transfer time begins to dominate the seek time, and a column-oriented approach begins to perform better than a row-oriented approach. For this reason, column-stores are typically used in analytic applications, with queries that scan a large fraction of individual tables and compute aggregates or other statistics over them[1].

In other words, columnar databases are best for OLAP loads, while row databases for OLTP[9]:

| Property             | Transaction processing systems (OLTP)              | Analytic Systems (OLAP)                   |
|----------------------|----------------------------------------------------|-------------------------------------------|
| Main read pattern    | Small number of records per query, fetched by key  | Aggregate over large number of records    |
| Main write pattern   | Random-access, low-latency writes from user input  | Bulk import (ETL) or event stream         |
| Primarily used by    | End user/customer, via web application             | Internal analyst, for decision support    |
| What data represents | Latest state of data (current point in time)       | History of events that happened over time |
| Dataset size         | Gigabytes to terabytes                             | Terabytes to petabytes                    |

### Columnar layout advance

In previouse sections I've tried to visualise the problem space and simplisticly explain what is the difference between row and column storages. In this section I'd like to take a deeper dive into columnar databases optimisations and how advancing technology popularised columnar databases[1].

#### History

The roots of column-oriented database systems can be traced to the 1970s, when transposed files first appeared. TOD (Time Oriented Database) was a system based on transposed files and designed for medical record management. One of the earliest systems that resembled modern column-stores was Cantor. It featured compression techniques for integers that included zero suppression, delta encoding, RLE (run length encoding), and delta RLE – all these are commonly employed by modern column-stores[1].

Research on transposed files was followed by investigations of vertical partitioning as a technique for table attribute clustering. At the time, row-stores were the standard architecture for relational database systems. A typical implementation for storing records inside a page was a slotted-page approach. This storage model is known as the N-ary Storage Model or NSM. In 1985, Copeland and Khoshafian proposed an alternative to NSM, the Decomposition Storage Model or DSM – a predecessor to columnstores.  For many, that work marked the first comprehensive comparison of row- and column-stores. For the next 20 years, the terms DSM and NSM were more commonly used instead of row- or column-oriented storage[1].

An analysis (based on technology available at the time) showed that DSM could speed up certain scans over NSM when only a few columns were projected, at the expense of extra storage space. Since DSM slowed down scans that projected more than a few columns, the authors focused on the advantages of DSM pertaining to its simplicity and flexibility as a storage format. They speculated that physical design decisions would be simpler for DSM-based stores (since there were no index-creation decisions to make) and query execution engines would be easier to build for DSM. The original DSM paper did not examine any compression techniques nor did it evaluate any benefits of column orientation for relational operators other than scans[1].

Although the research efforts around DSM pointed out several advantages of column over row storage, it was not until much later, in the 2000s, that technology and application trends paved the ground for the case of column-stores for data warehousing and analytical tasks[1].

At its core, the basic design of a relational database management system has remained to date very close to systems developed in the 1980s. The hardware landscape, however, has changed dramatically. In 1980, a Digital VAX 11/780 had a 1 MIPS CPU with 1KB of cache memory, 8 MB maximum main memory, disk drives with 1.2 MB/second transfer rate and 80MB capacity, and carried a $250K price tag. In 2010, servers typically had 5,000 to 10,000 times faster CPUs, larger cache and RAM sizes, and larger disk capacities. Disk transfer times for hard drives improved about 100 times and average disk-head seek times are 10 times faster (30msec vs. 3msec). **The differences in these trends (10,000x vs. 100x vs. 10x) have had a significant impact on the performance of database workloads**[1].

The imbalance between disk capacity growth and the performance improvement of disk transfer and disk seek times can be viewed through two metrics: (a) the transfer bandwidth per available byte (assuming the entire disk is used), which has been reduced over the years by two orders of magnitude, and (b) the ratio of sequential access speed over random access speed, which has increased one order of magnitude. **These two metrics clearly show that DBMSs need to not only avoid random disk I/Os whenever possible, but, most importantly, preserve disk bandwidth**[1].

As random access throughout the memory hierarchy became increasingly expensive, query processing techniques began to increasingly rely on sequential access patterns, to the point that most DBMS architectures are built around the premise that completely **sequential access should be done whenever possible**. However, as database sizes increased, scanning through large amounts of data became slower and slower. A bandwidth-saving solution was clearly needed, yet most database vendors did not view DSM as viable replacement to NSM, due to limitations identified in early DSM implementations[22] where DSM was superior to NSM only when queries access very few columns. In order for a column-based (DSM) storage scheme to outperform row-based (NSM) storage, **it needed to have a fast mechanism for reconstructing tuples** (since the rest of the DBMS would still operate on rows) and it also needed to be able to **amortize the cost of disk seeks when accessing multiple columns on disk**. **Faster CPUs would eventually enable the former and larger memories (for buffering purposes) would allow the latter**[1].

Although modern column-stores gained popularity for being efficient on processing disk-based data, in the 1990s, column-stores were mostly widely used in main-memory systems[1].

Around the 1996 one of the first commercial columnstore systems, SybaseIQ emerged, demonstrating the benefits that compressed, column-oriented storage could provide in many kinds of analytical applications. Although it has seen some commercial success over the years, it failed to capture the mindshare of other database vendors or the academic community, possibly due to a combinations of reasons, e.g., because it was too early to the market, hardware advances that later favored column-storage (and triggered database architecture innovations) such as large main memories, SIMD instructions, etc. where not available at the time, and possibly because it lacked some of the architectural innovations that later proved to be crucial for the performance advantages of column-stores, such as (extreme) late materialization, direct operation on compressed data throughout query plans, etc. Sybase IQ did offer some early variations of those features, e.g., compressing columns separately, or performing joins only on compressed data, avoiding stitching of tuples as early as loading data from disk, etc. but still it did not offer an execution engine which was designed from scratch with both columnar storage and columnar execution in mind[1].

By the 2000s column-stores saw a great deal of renewed academic and industrial interest. Incredibly inexpensive drives and CPUs had made it possible to collect, store, and analyze vast quantities of data. New, internet-scale user-facing applications led to the collection of unprecedented volumes of data for analysis and the creation of multiterabyte and even petabyte-scale data warehouses. To cope with challenging performance requirements, architects of new database systems revisited the benefits of column-oriented storage, this time combining several techniques around column-based data, such as read -only optimizations, fast multi-column access, disk/CPU efficiency, and lightweight compression. The (re)birth of column-stores was marked by the introduction of two pioneering modern column-store systems, C-Store and VectorWise. These systems provided numerous innovations over the state-of-the-art at the time, such as column-specific compression schemes, operating over compressed data, C-store projections, vectorized processing and various optimizations targeted at both modern processors and modern storage media[1].

Through the end of the 2000s there was an explosion of new columnoriented DBMS products.

Lastly, with popularization of solid state storages (SSDs) column-oriented storage has shown to never be worse than row storage, and in some cases where selective predicates were used, it outperformed row storage for any projectivity; if selectivity is high, then column-stores can minimize the amount of intermediate results they create which otherwise represents a significant overhead.

**In summary**:
* Column-oriented database systems can be traced to the 1970s.
* Column-oriented database systems lacked optimizations in early days which were advanced over the years.
* Column-oriented database systems were not good fit in the early days due to hardware limitations.
* Column-oriented database systems were not seriously considered, because the amount of data was perfectly manageable by row-oriented databases.
* With ever grow data size, sequential data access became more and more important (remember conclusion from File Systems section). 


#### Optimizations

As stated in previous section, simply storing data in columns wasn't/isn’t sufficient to get the full performance out of column-based stores. There are a number of techniques that have been developed over the years that also make a big impact. The figure below shows an unoptimised column store performing worse than a row store on a simplified TPC-H benchmark. But by the time you add in a number of the optimisations we’re about to discuss, it ends up about 5x faster than the row store [10].

![columnar-vs-row](https://github.com/MrR0807/Notes/blob/master/column-vs-row.jpeg)

**One of the most important factors in achieving good performance is preserving I/O bandwidth (by e.g. using sequential access wherever possible and avoiding random accesses). Thus even when we look at techniques such as compression, the main motivation is that moving compressed data uses less bandwidth (improving performance), not that the reduced sizes save on storage costs.**

You should also ask yourself, don't all these optimizations require more CPU cycles? Yes, but as previously discussed, **changes in hardware (CPU faster 10,000x vs. Disk transfer speed 100x vs. disk-head seek 10x) created a space where there is abundance of untapped CPU cycles compared to other aspects of reading/writing data. This means that now we have more CPU cycles to spare in decompressing compressed data fast** which is preferable to transferring uncompressed and thus bigger data at slow speeds (in terms of waisted CPU cycles) through the memory hierarchy[1].

Are These Column-store Specific Features and optimizations? Some of the features and concepts described above can be applied with some variations to row-store systems as well. In fact, most of these design features have been inspired by earlier research in row-store systems and over the years several notable efforts both in academia and industry tried to achieve similar effects for individual features with add-on designs in traditional row stores, i.e., designs that would not disturb the fundamental rowstore architecture significantly[1].

Figure 4.6 summarizes a list of features and design principles that altogether define modern column-stores along with pointers to similar but isolated features that have appeared in the past in the context of row-stores[1].

![columnar-vs-row-optimizations](https://github.com/MrR0807/Notes/blob/master/columnar-vs-row-optimizations.jpeg)

**In next few sections I will provide some columnar optimizations techniques which are also utilised in Parquet format.**

##### Run-Length Encoding (RLE)

> Run-length encoding (RLE) compresses runs of the same value in a column to a compact singular representation. Thus, it is well-suited for columns that are sorted or that have reasonable-sized runs of the same value. These runs are replaced with triples: (value, start position, runLength) where each element of the triple is typically given a fixed number of bits. For example, if the first 42 elements of a column contain the value ‘M’, then these 42 elements can be replaced with the triple: (‘M’, 1, 42) [1].

> Let’s say we have a column with 10,000,000 values, but all the values are 0. To store this information, we just need 2 numbers: 0 and 10,000,000 —the value and the number of times it repeated [6].

##### Dictionary Encoding with Bit-Packing

> Let’s say we have a column that contains country names, some of which are very long. If we wanted to store “The Democratic Republic of Congo,” we would need a string column that can handle at least 32 characters. Dictionary encoding replaces each value in our column with a small integer and stores the mapping in our data page’s metadata. When on disk, our encoded values are bit-packed to take up the least amount of space possible, but when we read the data we can still convert our column back to its original values[6].

##### Compression

> Intuitively, data stored in columns is more compressible than data stored in rows <...>. For example, assume a database table containing information about customers (name, phone number, e-mail address, snail-mail address, etc.). Storing all data together in the form of rows, means that each data page contains information on names, phone numbers, addresses, etc. and we have to compress all this information together. On the other hand, storing data in columns allows all of the names to be stored together, all of the phone numbers together, etc. Certainly phone numbers are more similar to each other than to other fields like e-mail addresses or names. This has two positive side-eects that strengthen the use of compression in column-stores; first, compression algorithms may be able to compress more data with the same common patterns as more data of the same type fit in a single page when storing data of just one attribute, and second, more similar data implies that in general the data structures, codes, etc. used for compression will be smaller and thus this leads to better compression [1].

### Hybrid layout

Let's go back to Parquet. Which layout does it utilise? The answer is both - otherwise known as hybrid layout.

**Note!** There are multiple different implementations of hybrid layout currently, but I'm going to cover only one of them which Parquet is based upon.

Hybrid layout was first suggested in academic paper called "Weaving Relations for Cache Performance"[3]. Within the paper, the first hybrid layout is described - PAX or Partition Attributes Across. According to the paper, PAX is:

> a new layout for data records that combines the best of the two worlds and exhibits performance superior to both placement schemes by eliminating unnecessary accesses to main memory. For a given relation, PAX stores the same data on each page as NSM. Within each page, however, PAX groups all the values of a particular attribute together on a minipage.

A visualisation of hybrid layout using previous example:

| Block 1           | Block 2           | Block 3           | ... | Block 10            |
|-------------------|-------------------|-------------------|-----|---------------------|
| 1, 2, 3, 4 | John, Adam, Eve, Maria | 26, 41, 29, 55  | ...   | 10000, 10000, 1000, 7500 |

The order of data is diveded into **row groups**. In this instance, one row group is comprised of 4 elements of each row's column. Just to further clarify I'll provide full 1D layout for each discussed layout.

I'll repeat the full table so scrolling is not necessary:

| Id  | Name  | Age | Salary |
|-----|-------|-----|--------|
| 1   | John  | 26  | 1000   |
| 2   | Adam  | 41  | 2000   |
| 3   | Eve   | 29  | 2500   |
| 4   | Maria | 55  | 1500   |
| 5   | Chris | 67  | 3000   |
| 6   | Emma  | 80  | 3500   |
| 7   | Ava   | 18  | 10000  |
| 8   | Liam  | 19  | 10000  |
| 9   | Lucas | 37  | 1000   |
| 10  | Peter | 61  | 7500   |

**Row Layout**

```
1,John,26,1000,2,Adam,41,2000,3,Eve,29,2500,4,Maria,55,1500,5,Chris,67,3000,6,Emma,80,3500,7,Ava,18,10000,8,Liam,19,10000,9,Lucas,37,1000,10,Peter,61,7500
```

**Column Layout**

```
1,2,3,4,5,6,7,8,9,10,John,Adam,Eve,Maria,Chris,Emma,Ava,Liam,Lucas,Peter,26,41,29,55,67,80,18,19,37,61,1000,2000,2500,1500,3000,3500,10000,10000,1000,7500
```

**Hybrid Layout**

With row groups of 4.

```
1,2,3,4,John,Adam,Eve,Maria,26,41,29,55,1000,2000,2500,1500,5,6,7,8,Chris,Emma,Ava,Liam,67,80,18,19,3000,3500,10000,10000,9,10,Lucas,Peter,37,61,1000,7500
```

Such layout as described in the paper turned out be performant[3]: 

> We evaluated PAX against NSM and DSM using (a) predicate selection queries on numeric data and (b) a variety of queries on TPC-H datasets on top of the Shore storage manager. We vary query parameters including selectivity, projectivity, number of predicates, distance between the projected attribute and the attribute in the predicate, and degree of the relation. The experimental results show that, when compared to NSM, PAX (a) incurs 50-75% fewer second-level cache misses due to data accesses when executing a main-memory workload, (b) executes range selection queries and updates in 17-25% less elapsed time, and (c) executes TPC-H queries involving I/O 11-42% faster than NSM on the platform we studied. When compared to DSM, PAX executes queries faster and its execution time remains stable as more attributes are involved in the query, while DSM’s execution time increases due to the high record reconstruction cost.

**Note!**. Projection (projectivity) means choosing which columns (or expressions) the query shall return. Selection (selectivity) means which rows are to be returned. If the query is: `sql select name, salary from TABLE where age>30;` then `name` and `salary` are **projection** part while `where age >30` is the **selection** part.

Below find some of the performance comparison between PAX, NSM and DSM layouts[3].

![PAX-NSM-DSM-1](https://github.com/MrR0807/Notes/blob/master/PAX-NSM-DSM-1.png)

![PAX-NSM-DSM-2](https://github.com/MrR0807/Notes/blob/master/PAX-NSM-DSM-2.png)

![PAX-NSM-DSM-3](https://github.com/MrR0807/Notes/blob/master/PAX-NSM-DSM-3.png)

![PAX-NSM-DSM-4](https://github.com/MrR0807/Notes/blob/master/PAX-NSM-DSM-4.png)

I will not go into details how the performance improvements were achieved, because you can read yourself in the paper, but in summary[5]:

> PAX was able to achieve the CPU efficiency of column-stores while maintaining the disk I/O properties of row-stores. For those without detailed knowledge of column-stores, this might seem strange: the way most column-stores pitch their products is by accentuating the disk I/O efficiency advantage (you only need to read in from disk exactly what attributes are accessed by a particular query). Why would a column-store want equivalent disk access patterns as a row-store? Well, it turns out column-stores have an oft-overlooked significant CPU efficiency as well. The aspect of CPU efficiency that the PAX paper examined was cache hit ratio and memory bandwidth requirements. It turns out that having column data stored sequentially within a block allows cache lines to contain data from just one column. Since most DBMS operators only operate on one or two columns at a time, the cache is filled with relevant data for that operation, thereby reducing CPU inefficiency due to cache misses. Furthermore, only relevant columns for any particular operation need to shipped from memory.

Lastly, I'd like to just add on top what is outlined in the quote - remember "Columnar layout optimizations" section. **Because data in row group is layed out in columnar fashion, columnar optimizations can be applied e.g. Run-Length Encoding (RLE)**.

### References

1. [The Design and Implementation of Modern Column-Oriented Database Systems](https://stratos.seas.harvard.edu/files/stratos/files/columnstoresfntdbs.pdf)
2. [Database Systems: Data Layouts (Part 1)](https://www.youtube.com/watch?v=bkwtWfFcwq0)
3. [Weaving Relations for Cache Performance](https://www.vldb.org/conf/2001/P169.pdf)
4. [Row-Store / Column-Store / Hybrid-Store](https://db.in.tum.de/teaching/ws1718/seminarHauptspeicherdbs/paper/sterjo.pdf?lang=de)
5. [A tour through hybrid column/row-oriented DBMS schemes](http://dbmsmusings.blogspot.com/2009/09/tour-through-hybrid-columnrow-oriented.html)
6. [Demystifying the Parquet File Format](https://towardsdatascience.com/demystifying-the-parquet-file-format-13adb0206705)
7. [Practical File System Design](http://www.nobius.org/dbg/practical-file-system-design.pdf)
8. [Row vs Column Oriented Databases](https://dataschool.com/data-modeling-101/row-vs-column-oriented-databases/)
9. [Designing Data-Intensive Applications](https://www.amazon.com/Designing-Data-Intensive-Applications-Reliable-Maintainable/dp/1449373321)
10. [The design and implementation of modern column-oriented database systems](https://blog.acolyer.org/2018/09/26/the-design-and-implementation-of-modern-column-oriented-database-systems/)


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

However, flat structures are not always best represantion of data as stated in Google's Dremel document[2]: 

> The data used in web and scientific computing is often nonrelational. Hence, a flexible data model is essential in these domains. Data structures used in programming languages, messages exchanged by distributed systems, structured documents, etc. lend themselves naturally to a **nested** representation. <...> A **nested data model underlies most of structured data processing** at Google and reportedly at other major web companies.

A nested type, for example, in SQL databases can be represented via relationships: one-to-many, many-to-many etc. This is represented in SQL by duplicating the parent data next to the child. For example, say we have additional table, which represents sales to a particular client. The client table will be represented as previous table, while the sales transactions could look like so:

| Sales Id | Client Id | Product | Amount |
|----------|-----------|------------|--|
| 1 | 1 | Apple | 0.60 |
| 2 | 1 | Banana | 1.00 |
| 3 | 3 | Apple | 0.60 |

Via join I can create a nested structure:

```sql
SELECT * FROM clients AS c 
INNER JOIN sales AS s
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

The creation of nested columnar structure was so successful, that opensource projects like Parquet used it. Later (after 4 years from Dremel publication), Google publicised another paper proving that nested structures were performant and scalable[3].

### Dremel's Nested Structure

In this section, I will explore Dremel's nested structure and via several examples, showcase core Definition and Repetition concepts, which allow for nested structures to be represented in columnar data.

**Note!** A lot of information in this section will be copy pasted from great Twitter blog post called "Dremel made simple with Parquet"[4]. I'm copying just to have all information in one place without needing to jump between pages. Also adding additional examples for better clarity.

#### The schema

To store in a nested format we first need to describe the data structures using a schema. This is done using a model similar to **Protocol buffers (Protobuf)**.

**Sidenote!** I'm guessing Parquet schema is similar to Google's Protobuf's schema, because they (Parquet developers) did not want to deviate from Google's Dremel paper, which naturally represented the nested structure schema in their used encoding format which was/is Protobuf. Nevertheless, as we'll see, there are differences.

This model [Protobuf] is minimalistic in that it represents nesting using groups of fields and repetition using repeated fields. There is no need for any other complex types like Maps, List or Sets as they all can be mapped to a combination of repeated fields and groups.

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

#### Definition levels

To support nested records we need to store the level for which the field is null. This is what the definition level is for: from 0 at the root of the schema up to the maximum level for this column. When a field is defined then all its parents are defined too, but when it is null we need to record the level at which it started being null to be able to reconstruct the record.

In a flat schema, an optional field is encoded on a single bit using 0 for null and 1 for defined. In a nested schema, we use an additional value for each level of nesting (as shown in the example), finally if a field is required it does not need a definition level.

For example, consider the simple nested schema below:

```
message ExampleDefinitionLevel {
  optional group a {
    optional group b {
      optional string c;
    }
  }
}
```

It contains one column: a.b.c where all fields are optional and can be null. When c is defined, then necessarily a and b are defined too, but when c is null, we need to save the level of the null value. There are 3 nested optional fields so the maximum definition level is 3.

Here is the definition level for each of the following cases:


| Value                 | Definition Level     |
|-----------------------|----------------------|
| a: null               | 0                    |
| a: { b: null }        | 1                    |
| a: { b: { c: null }}  | 2                    |
| a: { b: { c: "foo" }} | 3 (actually defined) |

Making definition levels small is important as the goal is to store the levels in as few bits as possible.

#### Repetition levels

To support repeated fields we need to store when new lists are starting in a column of values. This is what repetition level is for: it is the level at which we have to create a new list for the current value. In other words, the repetition level can be seen as a marker of when to start a new list and at which level. For example consider the following representation of a list of lists of strings:

```
message nestedLists {
  repeated group level1 {
    repeated string level2;
  }
}
```

This translates to arrays within arrays. Data can be represented as: `[[a,b,c], [d,e,f,g]], [[h], [i,j]]`. Or can be represented:

```
{
  level1: {
    level2: a
    level2: b
    level2: c
  },
  level1: {
    level2: d
    level2: e
    level2: f
    level2: g
  }
},
{
  level1: {
    level2: h
  },
  level1: {
    level2: i
    level2: j
  }
}
```

The column will contain the following repetition levels and values:


| Repetition level | Value |
|------------------|-------|
| 0                | a     |
| 2                | b     |
| 2                | c     |
| 1                | d     |
| 2                | e     |
| 2                | f     |
| 2                | g     |
| 0                | h     |
| 1                | i     |
| 2                | j     |

The repetition level marks the beginning of lists and can be interpreted as follows:
* 0 marks every new record and implies creating a new level1 and level2 list
* 1 marks every new level1 list and implies creating a new level2 list as well.
* 2 marks every new element in a level2 list.

A repetition level of 0 marks the beginning of a new record. In a flat schema there is no repetition and the repetition level is always 0. **Only levels that are repeated need a Repetition level**: optional or required fields are never repeated and can be skipped while attributing repetition levels.

#### Striping and assembly

Now using the two notions together, let’s consider the AddressBook example again. This table shows the maximum repetition and definition levels for each column with explanations on why they are smaller than the depth of the column. Reminder of AddressBook schema:

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

| Column            | Max Definition level | Max Repetition level     |
|-------------------|---------------------|--------------------------|
| owner             | 0 (required type)   | 0 (type is not repeated) |
| ownerPhoneNumbers | 1                   | 1 (repeated)             |
| contacts.name     | 1                   | 1 (repeated)             |
| contacts.phoneNumber         | 2                   | 1                        |

In particular for the column contacts.phoneNumber, a defined phone number will have the maximum definition level of 2, and a contact without phone number will have a definition level of 1. In the case where contacts are absent, it will be 0.

```
AddressBook {
  owner: "Julien Le Dem",
  ownerPhoneNumbers: "555 123 4567",
  ownerPhoneNumbers: "555 666 1337",
  contacts: {
    name: "Dmitriy Ryaboy",
    phoneNumber: "555 987 6543",
  },
  contacts: {
    name: "Chris Aniszczyk"
  }
}

AddressBook {
  owner: "A. Nonymous"
}
```

We’ll now focus on the column contacts.phoneNumber to illustrate this. Once projected the record has the following structure:

```
AddressBook {
  contacts: {
    phoneNumber: "555 987 6543"
  }
  contacts: {
  }
}

AddressBook {
}
```

The data in the column will be as follows:
* contacts.phoneNumber value:"555 987 6543" d:2 r:0
* contacts.phoneNumber value: null d:1 r:1
* contacts value:null d:0 r:0

Another representation of this example could be:

```
{
  AddressBook.owner: "Julien Le Dem"
  AddressBook.ownerPhoneNumbers: ["555 123 4567", "555 666 1337"]
  AddressBook.contacts: [{name: "Dmitriy Ryaboy", phoneNumber: "555 987 6543"}, {name: "Chris Aniszczyk", phoneNumber:null}]
},
{
  AddressBook.owner: "A. Nonymous"
  AddressBook.ownerPhoneNumbers: null
  AddressBook.contacts: null
}
```

If this does not make sense, don't worry. There are more examples, which hopefully will clear things up.

#### Examples

Code for calculating max repetition and max definition so you can build additional examples yourself:

```java
public class Test {

	public static void main(String[] args) {

		final var parquetSchemaString = """
				message Out {
				  optional group a {
				  	optional group b {
				  		optional int32 c;
				  	}
				  }
				}""";

		final var parquetSchema = MessageTypeParser.parseMessageType(parquetSchemaString);

		for (final var column : parquetSchema.getColumns()) {
			System.out.println(column.toString());
			System.out.println("R: " + column.getMaxRepetitionLevel());
			System.out.println("D: " + column.getMaxDefinitionLevel());
		}
	}
}
```


##### Example one

```
message Out {
  required int32 a;
  optional int32 b;
}
```

`a` field is required, that means it will always have a value. It is also not `repeated` type, hence there is no Repetition value. So in this case `a` field's Definition is 0 and Repetition is 0.
`b` field is not `repeated` type, hence Repetition is 0. However, it can either have a value or be null. When it is null, then Definition is set as 0 (just remember that when there is no value there are no bits). If `b` is defined then Definition value is 1.

If you define this schema in the previous Java code example and run - you'll get this printed:

```
[a] required int32 a
R: 0
D: 0
[b] optional int32 b
R: 0
D: 1
```

##### Example two

Moving forward Definition level will be shorten with just D value, and Repetition - R.

```
message Out {
  repeated int32 a;
}
```

Because it is repeated, there are two things to remember. `a` can have an array of values or be set to null. Here are possible variants:
* `a:null` - in this particular case, `a` D:0 R:0
* `a:[1,2,3,4,5]` - D:1 R:1

`a` in this example is very similar to "Example One" `optional b`. It is either a `null` or not, hence D is either 0 or 1. While R is 1, which indicates at what level array is.

##### Example three

```
message Out {
  repeated group a {
    optional int32 b;
  }
}
```

Possible variants:
* `a:null` 
* `a:[null]`
* `a:[1,2,3]`

From `b` perspective:
* `a:null` - D:0 R:0
* `a.b:null`- D:1 R:0
* `a.b:1` - D:2 R:0
* `a.b:1, a.b:2` - The first entry is D:2, R:0 (becaues it signals array start), the following entry - `a.b:2` is D:2 R:1 (because it tells at which array level it belongs to).

##### Example four

```
message Out {
  optional group one {
    repeated group two {
      repeated group three {
        optional int32 four;
      }
    }
  }
}
```

To make it easier, lets build all possible values:
* `one: null` - d:0 r:0
* `one.two: null` - d:1 r:0
* `one.two: []` - d:2 r:1
* `one.two.three: [null]` - d:3 r:1
* `one.two.three: [[]]` - d:4 r:2

Different perspective to this structure:

```
one: {
  two: [
    three: [
      four: 1 //R:0 (start of 1 and 2 level arrays)
      four: 2 //R:2 (value in 2 level array)
      four: 3 //R:2 (value in 2 level array)
    ],
    three: [
      four: 4 //R:1 (start of 2 level array)
      four: 5 //R:2 (value in 2 level array)
      four: 6 //R:2 (value in 2 level array)
    ]
  ]
}
```

### Resources

1. [Dremel made simple with Parquet](https://blog.twitter.com/engineering/en_us/a/2013/dremel-made-simple-with-parquet)
2. [Dremel: Interactive Analysis of Web-Scale Datasets](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/36632.pdf)
3. [Storing and Querying Tree-Structured Records in Dremel](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/43119.pdf)
4. [Dremel made simple with Parquet](https://blog.twitter.com/engineering/en_us/a/2013/dremel-made-simple-with-parquet)


## Encoding

This section's prerequisite is reading [Designing Data-Intensive Applications](https://www.amazon.com/Designing-Data-Intensive-Applications-Reliable-Maintainable/dp/1449373321) book's Chapter 4: Encoding and Evolution. It lays down the fundamentals very well. Some of information in this section will be copy pasted from named book's chapter.

### Formats for Encoding Data

Programs usually work with data in (at least) two different representations:
*  In memory, data is kept in objects, structs, lists, arrays, hash tables, trees, and so on. These data structures are optimized for efficient access and manipulation by the CPU (typically using pointers).
*  When you want to write data to a file or send it over the network, you have to encode it as some kind of self-contained sequence of bytes (for example, a JSON document). Since a pointer wouldn’t make sense to any other process, this sequence-of-bytes representation looks quite different from the data structures that are normally used in memory.

Thus, we need some kind of translation between the two representations. The translation from the in-memory representation to a byte sequence is called encoding (also known as serialization or marshalling), and the reverse is called decoding (parsing, deserialization, unmarshalling).

As this is such a common problem, there are a myriad different libraries and encoding formats to choose from.

### Language-Specific Formats

Many programming languages come with built-in support for encoding in-memory objects into byte sequences (e.g. Java's `java.io.Serializable`). These encoding libraries are very convenient, because they allow in-memory objects to be saved and restored with minimal additional code. However, they also have a number of deep problems:
* The encoding is often tied to a particular programming language, and reading the data in another language is very difficult.
* In order to restore data in the same object types, the decoding process needs to be able to instantiate arbitrary classes. This is frequently a source of security problems.
* Versioning data is often an afterthought in these libraries.
* Efficiency (CPU time taken to encode or decode, and the size of the encoded structure) is also often an afterthought. For example, Java’s built-in serialization is notorious for its bad performance and bloated encoding.

For these reasons it’s generally a bad idea to use your language’s built-in encoding for anything other than very transient purposes.

### JSON, XML, and Binary Variants

Moving to standardized encodings that can be written and read by many programming languages, JSON and XML are the obvious contenders. They are widely known, widely supported.

For data that is used only internally within your organization, there is less pressure to use a lowest-common-denominator encoding format (e.g. JSON). For example, you could choose a format that is more compact or faster to parse. For a small dataset, the gains are negligible, but once you get into the terabytes, the choice of data format can have a big impact.

**JSON is less verbose than XML, but both still use a lot of space compared to binary formats**.

### Practical Examples

Thrift and Avro examples will rely on lower level constructs of named libraries in order to encode **only** data as required. However, in How to Guides of named encoding documentations, provided examples usually use higher level constructs which ease usage/requires less boilerplate, but does not necessarily translate to expected data bytes/adds additional metadata (e.g. Avro's `DataFileWriter`, which is provided in their official documentation embeds Avro schema along with data hence making files way bigger).

#### JSON

JSON encoding currently is one of more prominent encodings. It is defined by The Internet Engineering Task Force document [RFC4627](https://www.ietf.org/rfc/rfc4627.txt). The JSON object structure is very simple and it does not require elaborate setup to create one by hand.

Let's say I want to encode this JSON message:
```json
{
  "a": 27,
  "b": "foo"
}
```

In the IETF document, there are hexidecimal values provided for all possible structure characters. Using it, I can construct a JSON message.
Firstly, JSON's begin-object starts with a `{` and ends with `}`. Respectable hex values are: `7B` and `7D`. Then quotation mark hex value is `22`, and `a` letter = `61`, `3A` for colon, `20` for space and representation for `27` number in hex is `3237`. Will not continue explictly defining each symbol, but here is the same JSON object, with each line containing it's hexidecimal representation:

```
{		7B
  "a": 27,	22 61 22 3A 20 3237 2C
  "b": "foo"	22 62 22 3A 20 22 666f6f 22
}		7D
```

The code:

```java
import org.apache.commons.codec.DecoderException;
import org.apache.commons.codec.binary.Hex;

import com.fasterxml.jackson.annotation.JsonCreator;
import com.fasterxml.jackson.databind.ObjectMapper;

public class TestFive {

	private static final ObjectMapper objectMapper = new ObjectMapper();

	public static void main(String[] args) throws IOException, DecoderException {

		final var jsonBytes = Hex.decodeHex("7b2261223a2032372c2262223a2022666f6f227d");
		System.out.println(new String(jsonBytes));

		final var test = objectMapper.readValue(jsonBytes, Test.class);
		System.out.println(test);
	}

	public record Test(int a, String b) {

		@JsonCreator
		public Test {
		}
	}
}
```

Running this prints:

```
{"a": 27,"b": "foo"}
Test[a=27, b=foo]
```

Which shows that this is correctly encoded and Java JSON library can deserialize it into a `record`. If I remove spaces, this JSON representation "weights" **18 bytes**.

#### Thrift

Apache Thrift is binary encoding library. Thrift was originally developed at Facebook, and was made open source in 2007.

Thrift requires a schema for any data that is encoded. Continuing JSON example, here is defined Thrift schema:

```
struct Test {
  1: required i64 a,
  2: required string b
}
```

Thrift has two different binary encoding formats: 
* [`BinaryProtocol`](https://github.com/apache/thrift/blob/master/doc/specs/thrift-binary-protocol.md)
* [`CompactProtocol`](https://github.com/apache/thrift/blob/master/doc/specs/thrift-compact-protocol.md)

Firstly, lets encode data with `BinaryProtocol` and analyse it.

**Sidenote!** By default, Thrift recommends to use their "Apache Thrift compiler", which generate classes from Thrift schema. For example, in [this repository](https://github.com/eugenp/tutorials/blob/master/apache-thrift/src/main/resources/cross-platform-service.thrift) `CrossPlatformResource` is a three field struct, which when generated becomes almost 600 lines [monster of a class](https://github.com/eugenp/tutorials/blob/master/apache-thrift/generated/com/baeldung/thrift/impl/CrossPlatformResource.java). I'll go a more simpler route.

##### BinaryProtocol

Code to generate data:

```java
public class ThriftExample {

	public static void main(String[] args) throws TException, IOException, ClassNotFoundException {

		TMemoryBuffer trans = new TMemoryBuffer(100);
		TProtocol protocol = new TBinaryProtocol(trans);

		write(protocol, 27, "foo");

		final var array = trans.getArray();
		System.out.println(Hex.encode(removeTrailingZeros(array)));

		read(array);
	}

	public static void write(TProtocol oprot, long a, String b) throws TException {

		oprot.writeStructBegin(new TStruct("Test"));

		oprot.writeFieldBegin(new TField("a", I64, (short) 1));
		oprot.writeI64(a);
		oprot.writeFieldEnd();

		oprot.writeFieldBegin(new TField("b", STRING, (short) 2));
		oprot.writeString(b);
		oprot.writeFieldEnd();

		oprot.writeFieldStop();
		oprot.writeStructEnd();
	}

	public static void read(byte[] array) throws TException {

		final var tMemoryBuffer = new TMemoryBuffer(array.length);
		tMemoryBuffer.write(array);
		TProtocol protocol = new TBinaryProtocol(tMemoryBuffer);

		protocol.readStructBegin();

		protocol.readFieldBegin();
		final var l = protocol.readI64();
		protocol.readFieldEnd();

		protocol.readFieldBegin();
		final var s = protocol.readString();
		protocol.readFieldEnd();

		protocol.readStructEnd();

		System.out.println(l);
		System.out.println(s);
	}

	/**
	 * Dumb way of removing trailing zeros
	 */
	public static byte[] removeTrailingZeros(byte[] original) {
		int sizeWithoutTrailingZeros = original.length;
		while (original[sizeWithoutTrailingZeros - 1] == 0) {
			--sizeWithoutTrailingZeros;
		}
		return Arrays.copyOf(original, sizeWithoutTrailingZeros);
	}
}


```

This will generate Hex value: `0a0001000000000000001b0b000200000003666f6f`.

Thrift [Struct encoding](https://github.com/apache/thrift/blob/master/doc/specs/thrift-binary-protocol.md#struct-encoding) defines data structure like so:
* field-type (whether it is a string, integer, list etc) is a signed 8 bit integer.
* field-id is a signed 16 bit integer.
* length indication (length of a string, number of items in a list). In our case it is going to be a string which is a signed 32 bit integer.
* field-value.

The bit size of integer is important, because each hex value can represent 4 bits. For example if we have 16 bit integer, that means there will be 4 hex values.

Deconstructing:
* Field type (8 bit, 2 hex values): `0a` - stands for 10. [Thrift Struct encoding](https://github.com/apache/thrift/blob/master/doc/specs/thrift-binary-protocol.md#struct-encoding) tells us that this is `I64`.
* Field id (16 bit, 4 hex values): `0001` - stands for 1.
* Field value (Because ints don't have length indicator it will be just value. Also we have defined `a` as `i64` we expect 8 bytes or 64 bits, or 16 hex values): `000000000000001b`  - stands for 27.
* Field type (8 bit, 2 hex values): `0b` - stands for 11. [Thrift Struct encoding](https://github.com/apache/thrift/blob/master/doc/specs/thrift-binary-protocol.md#struct-encoding) tells us that this is `BINARY` or string in other words.
* Field id (16 bit, 4 hex values): `0002` - stands for 2.
* Field length (because this is a string, it contains field lenght of 32 bit integer or 8 hex values): `00000003` - which stands for 3, the length of encoded "foo" string.
* Field value: `666f6f` - this should be familiar from JSON section and it stands for "foo".

We can manipulate the hex value and say instead of 27, I'd like to print 283. Old vs new hex value:


```
0a0001000000000000001b0b000200000003666f6f

0a0001000000000000011b0b000200000003666f6f
                   ^
```

And provide manipulated bytes to `read` method:

```java
final var manipulatedBytes = Hex.decode("0a0001000000000000011b0b000200000003666f6f");
read(manipulatedBytes);
```

Which will print as expected:

```
283
foo
```

As you can see, the big difference compared to JSON is that there are no field names (`a` or `b`). Instead, the encoded data contains field tags, which are numbers (1, 2, and 3). Those are the numbers that appear in the schema definition. Field tags are like aliases for fields—they are a compact way of saying what field we’re talking about, without having to spell out the field name.


##### CompactProtocol

The Thrift `CompactProtocol` encoding is semantically equivalent to `BinaryProtocol`, but it manages to pack the same information into fewer bytes.

It does this by packing the field type and field id into a single byte, and use variable-length integers. Rather than using a full eight bytes for the number 27, it is encoded in one byte, with the top bit of each byte used to indicate whether there are still more bytes to come. This means numbers between –64 and 63 are encoded in one byte, numbers between –8192 and 8191 are encoded in two bytes, etc. Bigger numbers use more bytes.

```java
public class ThriftExample {

	public static void main(String[] args) throws TException, IOException, ClassNotFoundException {

		TMemoryBuffer trans = new TMemoryBuffer(100);
		TProtocol protocol = new TCompactProtocol(trans);

		write(protocol, 27, "foo");

		final var array = trans.getArray();
		System.out.println(Hex.encode(removeTrailingZeros(array)));

		read(array);
	}

	public static void write(TProtocol oprot, long a, String b) throws TException {

		oprot.writeStructBegin(new TStruct("Test"));

		oprot.writeFieldBegin(new TField("a", I64, (short) 1));
		oprot.writeI64(a);
		oprot.writeFieldEnd();

		oprot.writeFieldBegin(new TField("b", STRING, (short) 2));
		oprot.writeString(b);
		oprot.writeFieldEnd();

		oprot.writeFieldStop();
		oprot.writeStructEnd();
	}

	public static void read(byte[] array) throws TException {

		final var tMemoryBuffer = new TMemoryBuffer(array.length);
		tMemoryBuffer.write(array);
		TProtocol protocol = new TCompactProtocol(tMemoryBuffer);

		protocol.readStructBegin();

		protocol.readFieldBegin();
		final var l = protocol.readI64();
		protocol.readFieldEnd();

		protocol.readFieldBegin();
		final var s = protocol.readString();
		protocol.readFieldEnd();

		protocol.readStructEnd();

		System.out.println(l);
		System.out.println(s);
	}

	/**
	 * Dumb way of removing trailing zeros
	 */
	public static byte[] removeTrailingZeros(byte[] original) {
		int sizeWithoutTrailingZeros = original.length;
		while (original[sizeWithoutTrailingZeros - 1] == 0) {
			--sizeWithoutTrailingZeros;
		}
		return Arrays.copyOf(original, sizeWithoutTrailingZeros);
	}
}
```

It is completely the same as in `BinaryProtocol` section, but the difference is that instead of `new TBinaryProtocol()` I'm using `new TCompactProtocol()`. 

Running main yields hex value: `16361803666f6f`.

Thrift [Struct encoding](https://github.com/apache/thrift/blob/master/doc/specs/thrift-compact-protocol.md#struct-encoding) for Compact protocol defines data structure like so:
* Instead of field-id now there is field-delta. "The field id delta can be computed by `current-field-id - previous-field-id`, or just `current-field-id` if this is the first of the struct". It is unsigned 4 bits integer, strictly positive.
* field-type (whether it is a string, integer, list etc) is an unsigned 4 bit integer instead of signed 8 bit integer.
* length indication (length of a string, number of items in a list). In our case it is going to be a string, hence the leght size is encoded with  Unsigned LEB128.
* field-value

From the definitions there are a couple of observable optimisations:
* Instead of separate field id and field type now there is one byte or 2 hex value field representing both.
* String lenght value is minimized using Unsigned LEB128 encoding.
* Integer values are compacted by using zigzag int encoding, then additionally with Unsigned LEB128.

These encodings are beyond this documentation scope, but there is a good blog post on [Variable length integers](https://golb.hplar.ch/2019/06/variable-length-int-java.html).

Helper functions to encode/decode these values (built by inspecting Thrift source code):

```java
import java.util.Arrays;

public class ThriftHelperUtils {
	
	public static long readI64(byte[] bytes) {
		return zigzagToLong(readVarint64(bytes));
	}

	public static long readVarint64(byte[] bytes) {
		int shift = 0;
		long result = 0;

		for (var b : bytes) {
			result |= (long) (b & 0x7f) << shift;
			if ((b & 0x80) != 0x80) break;
			shift += 7;
		}

		return result;
	}

	public static long zigzagToLong(long n) {
		return (n >>> 1) ^ -(n & 1);
	}

	public static byte[] writeI64(long i64) {
		return writeVarint64(longToZigzag(i64));
	}

	public static long longToZigzag(long l) {
		return (l << 1) ^ (l >> 63);
	}

	public static byte[] writeVarint64(long n) {
		final byte[] temp = new byte[10];
		int idx = 0;
		while (true) {
			if ((n & ~0x7FL) == 0) {
				temp[idx++] = (byte) n;
				break;
			} else {
				temp[idx++] = ((byte) ((n & 0x7F) | 0x80));
				n >>>= 7;
			}
		}
		return Arrays.copyOf(temp, idx);
	}
}
```

Analyse:
* Field Id delta + Field Type: `16` - in bits it is `0001 0110`. The first part of 4 bits represent the delta or if it is a first entry, then current id. `0001` bits traslate to `1`. The second portion of 4 bits represent field type. `0110` converts to `6` which in [Struct encoding](https://github.com/apache/thrift/blob/master/doc/specs/thrift-compact-protocol.md#struct-encoding) means `I64`.
* Field value: `36` - Because we know it is `I64` type, we expect at least 10 bytes to represent a number (as per documentation). Because we know that `int` and `long` values encoded with zigzag and then Unsigned LEB128, we have to use Helper functions to decode the number: `final var bytes = Hex.decodeHex("36"); System.out.println(ThriftHelperUtils.readI64(bytes));`. This will print 27, which is our number.
* Field Id delta + Field Type: `18` - translates to pair of 4 bits: `0001 1000`. First pair is the same as previously, the second translates to number 8, which means `BINARY` as per documentation.
* Field length (because it is string, from documentation, we know it is encoded with Unsigned LEB128): `03` - using Helper functions to decode the number: `System.out.println(readVarint64(Hex.decodeHex("03")))` prints `3`. This is our string lenght.
* Field value: `666f6f` - this should be familiar from JSON section and it stands for "foo" in UTF-8.

Just like before, let's manipulate Hex values directly to return different data. 

```
16361803666f6f

169ed303180a68656c6c6f776f726c64
  ^        ^
```

Use newly constructed hex value within `main` method:

```java
final var array = Hex.decode("169ed303180a68656c6c6f776f726c64");
read(array);
```

Which prints:

```
29903
helloworld
```

##### Conclusion

I'll repeat myself, but as you understood, differently from JSON, Thrift does not include names of the variables, but relies on schema and field tags which are represented as numbers. Thrift also encodes the type of the variable, differently from JSON, which tries to guess the type. Hence for small messages, Thrift Binary encoded message is bigger in size than JSON. If we use Compact, it is obviously smaller. To reach that, Thrift developers use elaborate bit encoding algorithms like zigzag and Unsigned LEB128. Furthermore, compacts how field ids and types are represented by concatinating bytes.

But Avro manages to compact data even further.

#### Avro

Apache Avro is another binary encoding format that is interestingly different from Thrift. It was started in 2009 as a subproject of Hadoop, as a result of Thrift not being a good fit for Hadoop’s use cases. Avro also uses a schema to specify the structure of the data being encoded.

There is a [great and lengthy blog post](https://writeitdifferently.com/avro/binary/encoding/2020/07/26/avro-binary-encoding-in-kafka.html), which goes into detail how byte values are constructed with Avro, as well as [Avro documentation](https://avro.apache.org/docs/1.8.1/spec.html#binary_encoding).

However, I will continue with my example. As stated, and just like with Thrift I will have to define Avro schema and use it to write data:

```java
public class WriteAvroBytes {

	static final String avroSchema = """
			{
			  "type": "record",
			  "name": "FooTest",
			  "fields" : [
				{"name": "a", "type": "long"},
				{"name": "b", "type": "string"}
			  ]
			}""";

	static final Schema schema = new Schema.Parser().parse(avroSchema);

	public static void main(String[] args) throws Exception {
		final var data = new GenericData.Record(schema);
		data.put("a", 27);
		data.put("b", "foo");

		try (final var baos = new ByteArrayOutputStream()) {
			writeToInputStream(data, baos);
			readInputStream(baos);
		}
	}

	private static void writeToInputStream(GenericData.Record data, ByteArrayOutputStream baos) throws IOException {
		BinaryEncoder encoder = EncoderFactory.get().binaryEncoder(baos, null);
		final var writer = new GenericDatumWriter<GenericRecord>(schema);

		writer.write(data, encoder);
		encoder.flush();
	}

	private static void readInputStream(ByteArrayOutputStream baos) throws IOException {
		final var binaryDecoder = DecoderFactory.get().binaryDecoder(new ByteArrayInputStream(baos.toByteArray()), null);
		
		System.out.println("Hex representation: " + new String(Hex.encodeHex(baos.toByteArray())));
		System.out.println("Byte size: " + baos.toByteArray().length);
		System.out.println("Value a: " + binaryDecoder.readLong());
		System.out.println("Value b: " + binaryDecoder.readString());
	}
}
```

Running this will print:

```shell
Hex representation: 3606666f6f
Byte size: 5
Value a: 27
Value b: foo
```

Avro message "weights" only 5 bytes, compared to 18 bytes in JSON format and Thrift's 7 bytes. 

To parse the binary data, I have to go through the fields in the order that they appear in the schema and use the schema to tell me the datatype of each field. This means that the binary data can only be decoded correctly if the code reading the data is using the exact same schema as the code that wrote the data. This is how Avro manages to shrink the size of the data further - there are no indication of field id or field type.

The encoding simply consists of values concatenated together. A string is just a length prefix followed by UTF-8 bytes, but there’s nothing in the encoded data that tells you that it is a string. It could just as well be an integer, or something else entirely. By the way, an integer is encoded using a variable-length zig-zag (same as Thrift).

NOTE! In provided code example I have chose to explicitly use specific methods to write and read Avro bytes. However, Avro library takes care of reading data out of the box without being this verbose.

Let's examine each encoded value separately.

* `36` - as stated per documentation, for "int and long values are written using variable-length zig-zag coding". Once again, we can use `ThriftHelperUtils` to decode this value as so: `System.out.println(readI64(Hex.decodeHex("36")));`. It will print `27`.
* `06` - again, this is encoded same way as ints and longs. To decode it, use Helper class: `System.out.println(readI64(Hex.decodeHex("06")));` which prints `3`.
* `666f6f` - this should be familiar from other section and it stands for "foo" in UTF-8 (you can check with `System.out.println(new String(Hex.decode("666f6f")));`).

Again, let's manipulate the hex value. In this case, instead of "foo" I'll repeat 600 times "La". The code to generate and read:

```java

final var byteArrayOutputStream = new ByteArrayOutputStream();
final var stringLength = ThriftHelperUtils.writeI64(1200);
final var stringLengthHex = Hex.encodeHex(stringLength);

final var repeat = "La".repeat(600);
final var stringValue = Hex.encodeHex(repeat.getBytes(StandardCharsets.UTF_8));

byteArrayOutputStream.writeBytes(Hex.decodeHex("36"));
byteArrayOutputStream.writeBytes(Hex.decodeHex(stringLengthHex));
byteArrayOutputStream.writeBytes(Hex.decodeHex(stringValue));

readInputStream(byteArrayOutputStream);
```

Which prints:
```
Hex representation: 36e0124c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c614c61
Byte size: 1203
Value a: 27
Value b: LaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLaLa
```

What fun!

### Conclusion

Correct encoding solves several problems:
* Space - encoding data can save space - both sending less data via network and storing in storages;
* Speed - using bloated encoding leads to slower encoding/decoding processes which puts more pressure on CPU;
* Schema evolution - some encoding types allow for fluent schema evolution.

In this last section I'm just going to add several benchmarks which tried to measure how fast encoding/decoding in JSON, Thrift, Avro:
* [Serialization performance in .NET: JSON, BSON, Protobuf, Avro](https://blog.devgenius.io/serialization-performance-in-net-json-bson-protobuf-avro-a25e8207d9de)
* [An Introduction and Comparison of Several Common Java Serialization Frameworks](https://www.alibabacloud.com/blog/an-introduction-and-comparison-of-several-common-java-serialization-frameworks_597900)
* [JVM serializers](https://github.com/eishay/jvm-serializers/wiki/)
* [Apache Thrift vs Protocol Buffers vs Fast Buffers](https://www.eprosima.com/index.php/resources-all/performance/apache-thrift-vs-protocol-buffers-vs-fast-buffers)
* [How Uber Engineering Evaluated JSON Encoding and Compression Algorithms to Put the Squeeze on Trip Data](https://www.uber.com/en-GB/blog/trip-data-squeeze-json-encoding-compression/)
* [Performance evaluation of object serialization libraries in XML, JSON and binary formats](https://www.semanticscholar.org/paper/Performance-evaluation-of-object-serialization-in-Maeda/676669064d37a904d503dc8c99338766cbdd96e7)

The results are a mixed bag and most of the time it seems that implementation details of particular language and library is more important rather than protocols itself.

## Conclusion

So the nested columnar data is represented in Protobuf schema, the metada is represented in Thrift encoding, columnar data applies compactions algos.

# Parquet file anatomy via Java implementation


## Stuff without place


https://towardsdatascience.com/understanding-apache-parquet-7197ba6462a9

The metadata is always written in the footer of the file as this allows a single pass write. In plain English, the data is written first, then the metadata can be accurately written knowing all the locations, size and encoding of the written data. Many formats write their metadata in the header. However, this requires multiple passes as data is written after the header. Parquet makes this efficient to read metadata and the data itself.


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
* https://medium.com/data-rocks/protobuf-as-an-encoding-format-for-apache-kafka-cad4709a668d (protobuf is not good?)


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
* [Avro Binary encoding based on messages in Kafka](https://writeitdifferently.com/avro/binary/encoding/2020/07/26/avro-binary-encoding-in-kafka.html)
* [An Introduction and Comparison of Several Common Java Serialization Frameworks](https://www.alibabacloud.com/blog/an-introduction-and-comparison-of-several-common-java-serialization-frameworks_597900)
* [Programmer’s Guide to Apache Thrift](https://www.manning.com/books/programmers-guide-to-apache-thrift)
* [Thrift Binary protocol encoding](https://github.com/apache/thrift/blob/master/doc/specs/thrift-binary-protocol.md)
* [Thrift Compact protocol encoding](https://github.com/apache/thrift/blob/master/doc/specs/thrift-compact-protocol.md)
* [Designing Data-Intensive Applications, 1st Edition](https://www.amazon.com/Designing-Data-Intensive-Applications-Reliable-Maintainable/dp/1449373321)








