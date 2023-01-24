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


# crc files

[Disabling crc file generation](https://groups.google.com/a/cloudera.org/g/cdk-dev/c/JR45MsLeyTE) points to Hadoop The Definitive Guide, 4th Edition chapter of Data Integrity. TODO - READ and summarize.

> Basically, they are used to make sure data hasn't been corrupted and in some cases replace corrupt copies with good ones. The overhead is fairly minimal for the utility you get, so I don't think it's a good idea to add an option to turn it off. The main concern over a bunch of tiny files is MR performance, but these are not used when calculating splits.

However, somebody has a problem which might be a problem to me:

> I have completely agreed with you that .crc file is good for data integrity and it is not adding any overhead on NN. Still, there are few cases where we need to avoid .crc file, for e.g. in my case I have mounted S3 on S3FS and saving data from rdd to mounting point. It is creating lots of .crc file on S3 which we don't require, to overcome this we need to write an extra utility to filter out all the .crc file which degrade our performance. The interesting observation is that there is a .crc file for `_SUCCESS` file too. and that .crc files is 8 bytes of size while the `_SUCCESS` file is 0 byte. If we are having 1000 million part files than we are using extra `1000M*12` bytes.






# Sources
* Hadoop The Definitive Guide, 4th Edition
* [Parquet Types](https://parquet.apache.org/docs/file-format/types/)
* [Disabling crc file generation](https://groups.google.com/a/cloudera.org/g/cdk-dev/c/JR45MsLeyTE)













