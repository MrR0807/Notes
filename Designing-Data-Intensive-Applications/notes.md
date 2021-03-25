# Chapter 1. Reliable, Scalable, and Maintainable Applications

Many applications today are data-intensive, as opposed to compute-intensive. Raw CPU power is rarely a limiting factor for these applications — bigger problems are usually the amount of data, the complexity of data, and the speed at which it is changing.
A data-intensive application is typically built from standard building blocks that provide commonly needed functionality. For example, many applications need to:
* Store data so that they, or another application, can find it again later **(databases)**
* Remember the result of an expensive operation, to speed up reads **(caches)**
* Allow users to search data by keyword or filter it in various ways **(search indexes)**
* Send a message to another process, to be handled asynchronously **(stream processing)**
* Periodically crunch a large amount of accumulated data **(batch processing)**

If that sounds painfully obvious, that’s just because these data systems are such a successful abstraction: we use them all the time without thinking too much. But reality is not that simple. There are many database systems with different characteristics, because different applications have different requirements. There are various approaches to caching, several ways of building search indexes, and so on.
