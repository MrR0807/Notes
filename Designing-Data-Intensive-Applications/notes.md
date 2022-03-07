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

## The Object-Relational Mismatch

The JSON representation has better locality than the multi-table schema in Figure 2-1. If you want to fetch a profile in the relational example, you need to either perform multiple queries (query each table by user_id) or perform a messy multiway join between the users table and its subordinate tables. In the JSON representation, all the relevant information is in one place, and one query is sufficient.

## Many-to-One and Many-to-Many Relationships

If the user interface has free-text fields for entering the region and the industry, it makes sense to store them as plain-text strings. But there are advantages to having standardized lists of geographic regions and industries, and letting users choose from a drop-down list or autocompleter:
* Consistent style and spelling across profiles
* Avoiding ambiguity (e.g., if there are several cities with the same name)
* Ease of updating—the name is stored in only one place, so it is easy to update across the board if it ever needs to be changed (e.g., change of a city name due to political events)
* Localization support—when the site is translated into other languages, the stand‐ ardized lists can be localized, so the region and industry can be displayed in the viewer’s language
* Better search—e.g., a search for philanthropists in the state of Washington can match this profile, because the list of regions can encode the fact that Seattle is in Washington (which is not apparent from the string "Greater Seattle Area")

## Are Document Databases Repeating History?

While many-to-many relationships and joins are routinely used in relational databases, document databases and NoSQL reopened the debate on how best to represent such relationships in a database. This debate is much older than NoSQL — in fact, it goes back to the very earliest computerized database systems.

Like document databases, IMS worked well for one-to-many relationships, but it made many-to-many relationships difficult, and it didn’t support joins.

Various solutions were proposed to solve the limitations of the hierarchical model. The two most prominent were the relational model (which became SQL, and took over the world) and the network model (which initially had a large following but eventually faded into obscurity). The “great debate” between these two camps lasted for much of the 1970s.

### The network model

The network model was standardized by a committee called the Conference on Data Systems Languages (CODASYL) and implemented by several different database ven‐ dors; it is also known as the CODASYL model. 

The CODASYL model was a generalization of the hierarchical model. In the tree structure of the hierarchical model, every record has exactly one parent; in the net‐ work model, a record could have multiple parents. For example, there could be one record for the "Greater Seattle Area" region, and every user who lived in that region could be linked to it. This allowed many-to-one and many-to-many relation‐ ships to be modeled.

The links between records in the network model were not foreign keys, but more like pointers in a programming language (while still being stored on disk). The only way of accessing a record was to follow a path from a root record along these chains of links. This was called an access path.

In the simplest case, an access path could be like the traversal of a linked list.

### The relational model

### Comparison to document databases

Document databases reverted back to the hierarchical model in one aspect: storing nested records (one-to-many relationships, like positions, education, and contact_info in Figure 2-1) within their parent record rather than in a separate table.

However, when it comes to representing many-to-one and many-to-many relationships, relational and document databases are not fundamentally different: in both cases, the related item is referenced by a *unique identifier*, which is called a *foreign key* in the relational model and a *document reference* in the document model. That identifier is resolved at read time by using a join or follow-up queries. **To date, document databases have not followed the path of CODASYL.**

## Relational Versus Document Databases Today











