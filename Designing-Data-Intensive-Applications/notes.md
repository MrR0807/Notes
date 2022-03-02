# Chapter 1. Reliable, Scalable, and Maintainable Applications

Many applications today are data-intensive, as opposed to compute-intensive. Raw CPU power is rarely a limiting factor for these applications — bigger problems are usually the amount of data, the complexity of data, and the speed at which it is changing.
A data-intensive application is typically built from standard building blocks that provide commonly needed functionality. For example, many applications need to:
* Store data so that they, or another application, can find it again later **(databases)**
* Remember the result of an expensive operation, to speed up reads **(caches)**
* Allow users to search data by keyword or filter it in various ways **(search indexes)**
* Send a message to another process, to be handled asynchronously **(stream processing)**
* Periodically crunch a large amount of accumulated data **(batch processing)**

If that sounds painfully obvious, that’s just because these data systems are such a successful abstraction: we use them all the time without thinking too much. But reality is not that simple. There are many database systems with different characteristics, because different applications have different requirements. There are various approaches to caching, several ways of building search indexes, and so on.

----------------------------

In order to figure out how bad your outliers are, you can look at higher percentiles: the 95th, 99th, and 99.9th percentiles are common (abbreviated p95, p99, and p999). They are the response time thresholds at which 95%, 99%, or 99.9% of requests are faster than that particular threshold. For example, if the 95th percentile response time is 1.5 seconds, that means 95 out of 100 requests take less than 1.5 seconds, and 5 out of 100 requests take 1.5 seconds or more.

## Approaches for Coping with Load

Common wisdom until recently was to keep your database on a single node (scale up) until scaling cost or high- availability requirements forced you to make it distributed.

The architecture of systems that operate at large scale is usually highly specific to the application—there is no such thing as a generic, one-size-fits-all scalable architecture (informally known as magic scaling sauce). The problem may be the volume of reads, the volume of writes, the volume of data to store, the complexity of the data, the response time requirements, the access patterns, or (usually) some mixture of all of these plus many more issues.

An architecture that scales well for a particular application is built around assumptions of which operations will be common and which will be rare — the load parameters. If those assumptions turn out to be wrong, the engineering effort for scaling is at best wasted, and at worst counterproductive.

Reliability means making systems work correctly, even when faults occur. Faults can be in hardware (typically random and uncorrelated), software (bugs are typically systematic and hard to deal with), and humans (who inevitably make mistakes from time to time). Fault-tolerance techniques can hide certain types of faults from the end user.

# Chapter 2. Data Models and Query Languages

Since the data model has such a profound effect on what the software above it can and can’t do, it’s important to choose one that is appropriate to the application.

## Relational Model Versus Document Model



















