# Chapter 1. Overview

To satisfy full-text search requirements along with other advanced functions, search engines came into existence with an approach different from that of a relational database query. The data undergoes an analysis phase in a search engine before it gets stored for later retrieval. This upfront analysis helps answer queries with ease.

### 1.4.1 Core areas

* Elastic Enterprise search
* Elastic Observability
* Elastic Security

### 1.4.2 Elastic Stack

The suite of products is called Elastic Stack, which includes Kibana, Logstash, Beats, and Elasticsearch. It was formally called ELK Stack but renamed Elastic Stack after Beats was introduced into the product suite in recent years.

# Chapter 2. Getting started

### 2.1.4 Using curl

```bash

curl -XPUT "http://localhost:9200/books/_doc/1" -H 'Content-Type: application/json' -d'
{
  "title": "Effective Java",
  "author": "Joshua Bloch",
  "release_date": "2001-06-01",
  "amazon_rating": 4.7,
  "best_seller": true,
  "prices": {
    "usd": 9.95,
    "gbp": 7.95,
    "eur": 8.95
  }
}'
```

### 2.1.5 Indexing our first document

The flow steps are:
* Kibana posts the request to the Elasticsearch server with the required input parameters.
* On receiving the request, the server
  * Analyzes the document data and, for speedier access, stores it in the inverted index (a high-performing data structure, which is the heart and soul of the search engine.
  * Creates a new index (we did not create the index upfront) and stores the documents. The server also creates required mappings and data schemas.
  * Sends the response back to the client.
* Kibana receives the response and displays it in the right panel (figure 2.6) for our consumption.

### 2.1.6 Constituents of the request

```PUT books/_doc/1```. 

The books in our URL is called an index, which is a bucket for collecting all book documents. 

The ``_doc`` in our URL is the endpoint. This is a constant part of the path that’s associated with the operation being performed. In earlier versions of Elasticsearch (version < 7.0), the ``_doc’s`` place used to be filled up by a document’s mapping type. The mapping types were deprecated and ``_doc`` came to replace the document’s mapping types as a generic constant endpoint path in the URL.

## 2.2 Retrieving Data

### 2.2.1 Counting all documents

Knowing the total number of documents in an index is a requirement; one that’s fulfilled by the ``_count`` API.


``GET books,movies/_count``.

Returns:

```json
{
  "count": 3,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  }
}
```

We can fetch the number of documents in all indices, too, by issuing a ``GET _count`` call.

### 2.2.2 Retrieving documents

#### RETRIEVING A SINGLE DOCUMENT

``GET <index>/_doc/<id>``

``GET books/_doc/1``

To fetch the original source document (no metadata):

``GET books/_source/1``

#### RETRIEVING MULTIPLE DOCUMENTS BY ID S

```bash
GET books/_search
{
  "query": {
    "ids": {
      "values": [1,2,3]
    }
  }
}
```

#### RETRIEVING ALL DOCUMENTS

```GET books/_search```

Is equal to:

```bash
GET books/_search
{
  "query": {
    "match_all": { }
  }
}

```

### 2.2.3 Full text queries

#### SEARCHING A BOOK WRITTEN BY A SPECIFIC AUTHOR

```bash
GET books/_search
{
  "query": {
    "match": {
      "author": "Joshua"
    }
  }
}
```

```bash
GET books/_search
{
  "query": {
    "prefix": {
      "author": "josh"
    }
  }
}
```

If we search for a full name like “Joshua Bloch”, we will get the books returned as expected. However, if we tweak the query with “Joshua Doe”, what do you expect? We don’t have any books written by Joshua Doe, so shouldn’t return any results right? That's not the case, we will still get the books returned written by Joshua Bloch, although there is no Joshua Doe in our author’s list. **The reason for this is that the engine is searching all books written by Joshua OR Doe.**

#### SEARCHING A BOOK WITH AN EXACT AUTHOR

```bash
GET books/_search
{
  "query": {
    "match": {
      "author": { #A The author field is now having inner properties defined
        "query": "Joshua Schildt", #B provide your query here
        "operator": "AND" #C The AND operator (default is OR)
      }
    }
  }
}
```

### 2.2.4 Indexing more documents using the `_bulk` API

```bash

POST _bulk
{"index": {"_index":"books","_id":"1"}}
{
  "title": "Core Java Volume I – Fundamentals",
  "author": "Cay S. Horstmann",
  "edition": 11,
  "synopsis": "Java reference book that offers a detailed explanation of various features of Core Java, including exception handling, interfaces, and lambda expressions. Significant highlights of the book include simple language, conciseness, and detailed examples.",
  "amazon_rating": 4.6,
  "release_date": "2018-08-27",
  "tags": ["Programming Languages, Java Programming"]
}

{"index":{"_index":"books","_id":"2"}}
{
  "title": "Effective Java",
  "author": "Joshua Bloch",
  "edition": 3,
  "synopsis": "A must-have book for every Java programmer and Java aspirant, Effective Java makes up for an excellent complementary read with other Java books or learning material. The book offers 78 best practices to follow for making the code better.", 
  "amazon_rating": 4.7,
  "release_date": "2017-12-27",
  "tags": ["Object Oriented Software Design"]}
```

```bash
curl -X POST 'http://localhost:9200/_bulk?pretty' -H "Content-Type: application/x-ndjson" --data-binary '
{"index":{"_index":"books", "_type" : "_doc", "_id":"1"}}
{"title": "Core Java Volume I – Fundamentals","author": "Cay S. Horstmann","edition": 11, "synopsis": "Java reference book that offers a detailed explanation of various features of Core Java, including exception handling, interfaces, and lambda expressions. Significant highlights of the book include simple language, conciseness, and detailed examples.","amazon_rating": 4.6,"release_date": "2018-08-27","tags": ["Programming Languages, Java Programming"]}
{"index":{"_index":"books", "_type" : "_doc", "_id":"2"}}
{"title": "Effective Java","author": "Joshua Bloch", "edition": 3,"synopsis": "A must-have book for every Java programmer and Java aspirant, Effective Java makes up for an excellent complementary read with other Java books or learning material. The book offers 78 best practices to follow for making the code better.", "amazon_rating": 4.7, "release_date": "2017-12-27", "tags": ["Object Oriented Software Design"]}'
```

### 2.2.5 Searching across multiple fields

When a customer searches for something in a search bar, the search doesn’t necessarily restrict to just one field. For example, we want to search all the documents where the word Java appears, not just in the title field, but also in other fields like synopsis, tags, and so on. This is where we enable a multi-field search. We use a multi_match query, which searches the criteria across multiple fields.

Let’s see an example where we create a query to search for Java in two fields, title and synopsis.

```bash
GET books/_search
{
    "query": {
        "multi_match": { #A Multi match query that searches across multiple fields
            "query": "Java", #B The search words
            "fields": ["title","synopsis"] #C Searching across two fields
        }
    } 
}
```

As expected, we’ve searched across multiple fields and got our results. But, say, we want to bump up the priority of a result based on a field. For example, if Java is found in the title field, boost that search result twice while keeping the other documents at a normal priority.

#### BOOSTING RESULTS

Elasticsearch let’s us bump up or boost priority for certain fields in our queries by providing the boost factor next to the field. That is, if we need bump up title’s field by factor three, we set the boost on the field as `title^3`.

```shell
GET books/_search
{
  "query": {
    "multi_match": { #A We are searching through multiple fields
      "query": "Java",
      "fields": ["title^3","synopsis"] #B Caret followed by the boost number
    } 
  }
}
```

### 2.2.6 Search on a phrase

At times we wish to search for a sequence of words, exactly in that order, like finding out all books that have a phrase: “must-have book for every Java  programmer” amongst synopsis fields in our books. We can write a `match_phrase` query for this purpose.

```shell
GET books/_search
{
  "query": {
    "match_phrase": { #A The match_phrase query expects a sequence of words
      "synopsis": "must-have book for every Java programmer" #B Our phrase
    }
  } 
}
```

Result:

```json
{
  "hits": [{
    "_score" : 7.300332,
    "_source" : {
    "title" : "Effective Java",
    "synopsis" : "A must-have book for every Java programmer and Java ...",
  }]
}
```

#### PHRASES WITH MISSING WORDS

The `match_prefix` query expects a full phrase: a phrase without any missing words in between. However it is not always the case that we have a phrase without missing words - for example, instead of searching for “Elasticsearch in action”, users may search for “Elasticsearch action”. To honour this, we set `match_phrase` with a `slop` parameter. A `slop` expects a positive integer indicating how many words that the phrase is missing when searching.

```shell
GET books/_search
{
  "query": {
    "match_phrase": {
      "synopsis": {
        "query": "must-have book every Java programmer", #A missing “for” word
        "slop": 1 #B The slop is set to 1, indicating one word is missing
      } 
    }
  } 
}
```

#### MATCHING PHRASES WITH A PREFIX

```shell
GET books/_search
{
  "query": {
    "match_phrase_prefix": {
      "title": "Java co"
    }
  } 
}
```

### 2.2.7 Fuzzy queries

```shell
GET books/_search
{
  "query": {
    "fuzzy": {          #A Fuzzy query to support spelling mistakes
      "title": {
        "value": "kava",#B The incorrectly spelt criteria
        "fuzziness": 1  #C Fuzziness 1 indicates one letter forgiveness
      } 
    }
  } 
}
```

We set fuzziness as 1 because we expect a single letter change (k -> j) is required to match the subject.

### 2.2.8 Term-level queries

Elasticsearch creates a separate form of queries, known as term-level queries, to support querying structured data. Numbers, dates, range, IP addresses, etc., belong to a structured text category.

Elasticsearch treats the structured and unstructured data in different ways: the unstructured (full- text) data gets analyzed, while the structured fields are stored as is.

They produce a binary output: fetch the result if the query matches with the criteria.

#### FETCHING A PARTICULAR EDITION BOOK (TERM QUERY)

```shell
GET books/_search
{
  "_source": ["title","edition"], #A Only two fields are returned
  "query": {
    "term": {                     #B Declare the query as a term level query
      "edition": {                #C Provide the field and the value as search criteria
        "value": 3 
      }
    } 
  }
}
```

This query returns all third edition books. 

#### THE RANGE QUERY

```shell
GET books/_search
{
  "query": {
    "range": { #A Range query declaration
      "amazon_rating": {#B Mention the range to match
        "gte": 4.5,#C gte - greater than or equal to
        "lte": 5 #D lte - less than or equal to
      } 
    }
  } 
}
```

## 2.3 Compound queries

Compound queries combine individual queries, called leaf queries, to build powerful and robust queries providing us the capability to cater to complex scenarios.

### 2.3.1 Boolean (bool) query

A boolean, commonly called a ``bool`` query, is used to create a sophisticated query logic by combining other queries based on boolean conditions. A ``bool`` query expects the search to be built using a set of four clauses: ``must``, ``must_not``, ``should``, and ``filter``.

```shell
GET books/_search
{
  "query": {
    "bool": {#A A bool query is a combination of conditional boolean clauses
      "must": [{ }],#B The criteria must match with the documents
      "must_not": [{ }],#C The criteria must-not match (no score contribution)
      "should": [{ }],#D The query should match
      "filter": [{ }] #E The query must match (no score contribution)
    } 
  }
}
```

### 2.3.2 The must (must) clause

```shell
GET books/_search
{
  "query": {
    "bool": { #A A boolean query
      "must": [{# A must clause - the documents must match to the criteria
          "match": {#A One of the queries - a match query
            "author": "Joshua Bloch"
          }
      }] 
    }
  } 
}
```

``must`` clause accepts a set of multiple queries.

```shell
GET books/_search
{
  "query": {
    "bool": {
      "must": [
        { #A Must query with two leaf queries
          "match": {#B A match query finding books authored by Joshua
            "author": "Joshua Bloch"
           }
        },
        {
           "match_phrase": {#C A second query searching for a phrase
             "synopsis": "best Java programming books"
           }
        }     
      ]
    } 
  }
}
```

### 2.3.3 The must not (must_not) clause

```shell
GET books/_search
{
  "query": {
    "bool": {
      "must": [ #A Must clause
        { 
          "match": { 
            "author": "Joshua" 
          } 
        }
      ],
      "must_not": [ #B A must_not clause with a range query 
        { 
          "range": { 
            "amazon_rating": { 
              "lt": 4.7
            }
          }
        }
      ]
    } 
  }
}
```

### 2.3.4 The should (should) clause

The ``should`` clause behaves like an ``OR`` operator. That is, the search words are matched against the ``should`` query, and if they match, the relevancy score is bumped up.

```shell
GET books/_search
{
  "query": {
    "bool": {
      "must": [{"match": {"author": "Joshua"}}], 
      "must_not":[{"range":{"amazon_rating":{"lt":4.7}}}], 
      "should": [{"match": {"tags": "Software"}}]
    } 
  }
}
```

### 2.3.5 The filter (filter) clause

``filter`` clause works exactly like the ``must`` clause except it doesn’t affect the score. Any results that don’t match the ``filter`` criteria are dropped.

```shell
GET books/_search
{
  "query": {
    "bool": {
      "must": [{"match": {"author": "Joshua"}}], 
      "must_not":[{"range":{"amazon_rating":{"lt":4.7}}}], 
      "should": [{"match": {"tags": "Software"}}], 
      "filter": [{"range":{"release_date":{"gte": "2015-01-01"}}}]}
  } 
}
```

## 2.4 Aggregations

Analytics enables organizations to find insights into the data. So far, we've looked at searching for the documents from a given corpus of documents. Analytics is looking at the big picture and analyzing the data from a very high level to draw conclusions about it. We use aggregation APIs to provide analytics in Elasticsearch. Aggregations fall into three categories:
* Metric aggregations — Simple aggregations like sum, min, max, and average fall into this category of aggregations. They provide an aggregate value across a set of document data.
* Bucket aggregations — Bucket aggregations help collect data into buckets, segregated by intervals like days, age groups, etc. These help us build histograms, pie charts and other visualizations.
* Pipeline aggregations — Pipeline aggregations work on the output from the other aggregations.

Data snippet:

```shell
POST covid/_bulk
{"index":{}}
{"country":"USA","date":"2021-03-26","deaths":561142,"recovered":23275268}
{"index":{}}
{"country":"Brazil","date":"2021-03-26","deaths":307326,"recovered":10824095}
...
```

### 2.4.1 Metrics

```shell
GET covid/_search # _search endpoint used for aggregations too
{
  "aggs": { #A Writing an aggregation query, short for aggregations is the cue for type of operation
    "critical_patients": { #B User defined query output name
      "sum": { #C The sum metric - sum of all the critical patients
        "field": "critical" #D The field on which the aggregation is applied
      } 
    }
  } 
}
```

Response:

```shell
"aggregations" : {
  "critical_patients" : {
    "value" : 88090.0
  }
}
```

Note the response will consist of all documents returned if not asked explicitly to suppress them. We can set size=0 as the root level to stop the response containing documents.

```json
GET covid/_search
{
  "size": 0,
  "aggs": { 
    ...
  }
}
```

#### USING OTHER METRICS

```shell
GET covid/_search
{
  "size": 0,
  "aggs": {
    "max_deaths": {
      "max": {
        "field": "deaths"
      }
    } 
  }
}
```

In the similar vein, if we want to find the maximum number of deaths across all countries in our COVID data, we use a max aggregation.

```shell
GET covid/_search
{
  "size": 0,
  "aggs": {
    "all_stats": {
      "stats": { #A stats query returns all five core metrics in one go
        "field": "deaths"
      }
    } 
  }
}
```

This stats query returns:

```json
"aggregations" : {
  "all_stats" : {
    "count" : 20,
    "min" : 30772.0,
    "max" : 561142.0,
    "avg" : 163689.1,
    "sum" : 3273782.0
  } 
}
```

### 2.4.2 Bucketing

Bucketing is all about segregating data into various groups or so-called buckets. For example, we can add these groups to our buckets: a survey of adult  groups according to their age bracket (20–, 31–40, 41–50).

#### HISTOGRAM BUCKETS

The histogram bucketing aggregation creates a list of buckets on a numerical value by going over all the documents.

```shell
GET covid/_search
{
  "size": 0,
  "aggs": {
    "critical_patients_as_histogram": {#A The user-defined name of the report
      "histogram": {#B The type of the bucketing aggregation - histogram
        "field": "critical", #C The field the aggregation applied on
        "interval": 2500 #D The bucket interval
      }
    } 
  }
}
```

```json
"aggregations": {
  "critical_patients_as_histogram" : {
    "buckets" : [
      {
       "key" : 0.0,
       "doc_count" : 8
      }, 
      {
       "key" : 2500.0,
       "doc_count" : 6
      },
      {
       "key" : 5000.0,
       "doc_count" : 0
      }, 
      {
       "key" : 7500.0,
       "doc_count" : 6
      }
    ]
  } 
}
```

#### RANGE BUCKETS

The range bucketing defines a set of buckets based on predefined ranges. For example, say we want to segregate the number of COVID casualties by country (casualties up to 60000, 60000–70000, 70000–80000, 80000–120000).

```shell
GET covid/_search
{
  "size": 0,
  "aggs": {
    "range_countries": {
      "range": {            #A The range bucketing aggregation
        "field": "deaths",  #B Field on which we apply the agg
        "ranges": [         #C Define the custom ranges
          {"to": 60000},
          {"from": 60000,"to": 70000},
          {"from": 70000,"to": 80000},
          {"from": 80000,"to": 120000}
        ] 
      }
    } 
  }
}
```

# Chapter 3. Architecture

## 3.1 A 10,000 foot overview

### 3.1.1 Data in

The data can be indexed into Elasticsearch from multiple sources and in various ways: extracting from a database, copying files from file systems, or even loading from other systems including real-time streaming systems and so on.

### 3.1.2 Processing the data

The basic unit of information is represented by a JSON document in Elasticsearch.

```json
{
  "title":"Is Remote Working the New Norm?",
  "author":"John Doe",
  "synopsis":"Covid changed lives. It changed the way we work..",
  "publish_date":"2021-01-01",
  "number_of_words":3500
}
```

#### COLLECTING THE DATA

To house the data, Elasticsearch creates a set of buckets based on each type of data. The news articles, for instance, will be housed in a bucket called  news, which is a name we chose. In Elasticsearch lingo, we call this bucket an **index**. **An index is a logical collection of documents. Index in  Elastic Search is what table in database.**

During the process of indexing the data, Elasticsearch analyzes the incoming data field-by-field.

#### ANALYZING THE DATA

The data represented as text is analysed during the text analysis phase. The text is broken down into words (called tokens) using a set of rules. Fundamentally, two processes happen during the analysis process: **tokenization and normalization.**

Tokenization is the process of breaking the text into tokens based on a set of rules. Example:

Untokenized String: Covid changed our lives. It changed the way we work..
Tokenized String: `[Covid,changed,our,lives,it,changed,the,way,we,work]`

**Normalization** helps build a rich user experience by creating additional data around the tokens. It is a process of reducing (stemming) these tokens to root words or creating synonyms for the tokens. For example:
* The *lives* token can be stemmed to create alternate words like life;
* The *covid* token can be stemmed to produce corona, coronavirus, and sars.

### 3.1.3 Data out

When a search query is issued, if the field is a full-text field, it undergoes an analysis phase similar to what was performed during the indexing of that field. That is, the query is tokenized and normalized as per the same analyzers associated with those fields.

## 3.2 The building blocks

### 3.2.1 Document

A document is the basic unit of information that gets indexed by Elasticsearch for storage.

#### DOCUMENT OPERATION APIS

You can index or retrieve documents one by one using the single document APIs or batch them up using multi-document APIs:
* Single document APIs — Perform actions on individual documents, one by one;
* Multiple document APIs — Work with multiple documents in one go (bulk).

### 3.2.2 Removal of types

The data we persist has a specific shape: a movie document has properties related to a movie, data for a car has properties related to cars, or an employee document has data relevant to the context of employment and business. We are expected to index these JSON documents into respective buckets or collections: movie documents consisting of movie data need to be held in an index named *movies*, for example. So, in essence we index a document of type *Movie* into  movies index, *Car* into cars index and so on.

Beginning with version 6.0, a single type per index was introduced, meaning the cars index is expected to hold just car documents.

However, APIs were upgraded from 7.0.0: you are advised to use `_doc` as the endpoint going forward. The type of the document is replaced by a default  document type `_doc`, which later on has become a permanent fixture in the url as an endpoint. Hence, the URL becomes ``PUT cars/_doc/1``.

### 3.2.3 Index

Elasticsearch keeps the data documents in an index. They are backed by shards.

*Shards* are the physical instances of Apache Lucene, the workhorses behind the scenes in getting our data in and out of storage. In other words, shards take care of the physical storage and retrieval of our data. Any index created by default is set to be backed up by a single shard and a replica.

### 3.2.4 Data streams

We have been working on indices (such as movies, movie_reviews etc) which will hold and collect data over time. If the data gets huge, we could add  additional indices to copy (or move) data across to accommodate. The expectation is that this type of data doesn’t need to be rolled over into newer indices periodically, like hourly, daily or monthly.

As the name indicates, the time-series data is time sensitive and time dependent.

The logs are continuously logged to a current day’s log file. For each of the log statements, a timestamp is associated with it. At midnight, the file will be backed up with a date stamp and a new file will be created for the brand new day.

If we wish to hold the log data in Elasticsearch, we need to rethink the strategy of indexing the data that changes/rolls over periodically into indices.  Surely, we can write an index-rollover script that could potentially rollover the indices at midnight every day. But there’s more to this than just rolling over the data. For example, we also need to take care of directing the search requests against a single mother index rather than multiple rolling indices. We will be creating an alias for this purpose.

#### TIME SERIES DATA

Data streams accommodate time series data in Elasticsearch - they let us hold the data in multiple indices but allow access as a single resource for search and analytical related queries. As discussed earlier, the data that is tagged to a date or time axis such as logs, automated car’s events, pollution levels in a city etc, is expected to be hosted in timed indices. These indices on a high level are called data streams. Behind the scenes, each of the data streams has a set of indices for each of the time points. These indices are auto generated by Elasticsearch and hidden.

The data stream itself is nothing more than an alias for the time-series (rolling) hidden indices behind the scenes. While the search/read requests are  spanned across all the data stream’s backing hidden indices, the indexing requests will be only directed to the new (current) index.

Data streams are created using a matching indexing template. Templates are the blueprints consisting of settings and configuration values when creating resources like indices.

### 3.2.5 Shards and replicas

Shards are the software components holding data, creating the supported data structures (like inverted index), managing queries, and analysing the data in Elasticsearch.

During the process of indexing, the document travels through to the shard. Shards creates immutable file segments to hold the document on to a durable file system.

As duplicate copies of shards, the replica shards serve the redundancy and high availability in an application. By serving the read requests, replicas enable distributing the read load during peak times.

#### DISTRIBUTION OF SHARDS AND REPLICAS

Say, we have created an index `virus_mutations` for holding the Corona virus mutations’ data. According to our strategy, this index will be catered by three shards. When we started our first node (Node A), not all shards would’ve been created for the index. This usually happens when the server is just starting up. Once the Node A has come up, based on the settings, three shards are created on this node for the ``virus_mutations``. We know that all these shards are on a single node, and for whatever the reason, if this single node crashes, we will lose everything. To avoid data loss, we decided to start a second node to join the existing cluster. Once the the new node (Node B) is created and added to the cluster, Elasticsearch distributes the original three shards as follows:
* The shard 2 and shard 3 are removed from Node A;
* The shard 2 and shard 3 are then moved to Node B.

#### HEALTH STATUS OF A CLUSTER

There are three states for a cluster:
* Red - not all shards are assigned and ready;
* Yellow - shards are assigned and ready but replicas aren't assigned and ready;
* Green - shards and replicas are all assigned and ready.

#### REBALANCING SHARDS

#### SHARD SIZING

The industry’s best practice is to size an individual shard with no more than 50 GB. But I have seen shards going up to 400 GB in size too. In fact,  GitHub’s indices are spread across 128 shards with 120 GB each. My personal advice is to keep them between 25 GB and 40 GB keeping the node’s heap memory in mind.

There is also one more parameter to consider for sizing the shards: the heap memory. As we know, the nodes have a limited set of computing resources such as memory and disk space. Each Elasticsearch instance can be tweaked to use heap memory based on the available memory. My advice is to host up to 20 shards per GB of heap memory.

### 3.2.6 Nodes and clusters

#### Manage your node disk space judiciously

To counteract read performance, we can add additional replicas but with that comes higher memory and disk space requirements. While it is not unusual to create clusters with terabytes or even petabytes when working with Elasticsearch, we must give forethought to our data sizing requirements.

For example, if we have a three-shards-and-15-replicas-per-shard strategy with each shard sized at 50 GB, we must ensure all the 15 replicas have enough capacity for not only storing the documents on disk but also for heap memory. This means:

Shards memory: 3 ✕ 50 GB/shard = 150 GB
Replicas memory/per shard: 15 ✕ 50 GB/replica = 750 GB/per shard
(Replicas memory for 3 shards = 3 x 750 GB = 2250 GB)
Total memory for both shards and replicas on a given node = 150GB + 750GB = 900 GB (Grand total for 20 nodes = 18 TB)

That is, a whopping 18 TB that is required for one index with three-shards-and-15-replicas-per-shard strategy. In addition to this initial disk space, we also need further disk space for running the server smoothly. So, we must work through the capacity requirements judiciously.

#### MULTI-NODE MULTI CLUSTERS

Bundling all sorts of data into a single cluster is not unusual, but it might not be a best practice. It might be a better strategy to create multiple clusters for varied data shapes with customized configurations for each cluster. For example, a business-critical data cluster might be running on an on-premise cluster with higher memory and disk space options configured, while an application-monitoring data cluster will have a slightly different setup.

#### NODE ROLES

Master node - Its primary responsibility is cluster management.
Data node - Responsible for document persistence and retrieval. 
Ingest node - Responsible for the transformation of data via pipeline ingestion before indexing.
Machine learning node - Handles machine learning jobs and requests.
Transform node - Handles transformations requests.
Coordination node - This role is the default role. It takes care of incoming client’s requests.

**Master Node**: A master node is involved in high-level operations such as creating and deleting indexes, node operations, and other admin-related jobs for cluster management. These admin operations are light-weight processes; hence, one master is enough for an entire cluster. If this master node crashes, the cluster will elect one of the other nodes as the master so the baton continues.
**Data Node**: A data node is where the actual indexing, searching, deleting, and other document-related operations happen. These nodes host the indexed 
documents. Once an index request is received, they jump into action to save the document to its index by calling a writer on the Lucene segment. As you can imagine, they talk to the disk frequently during CRUD operations and, hence, they are disk I/O and memory-intensive operations.  There are specific variants of a data node role that will come to use when we deploy multi-tiered deployments. They are **data_hot**, **data_warm**, **data_cold** and **data_frozen** roles.
**Ingest node**: An ingest node handles the ingest operations such as transformations and enrichment before the indexing kicks in.
**Transform node**: The transform node role is the latest addition to the list. It’s used for the aggregated summary of data.
**Coordinating node**: While these roles are assigned to a node by the user on purpose (or by default), there’s one special role that all the nodes take on irrespective of the user’s intervention: a coordinating node. After accepting the requests, the coordinator asks the other nodes in the cluster for the processing of the request. It awaits the response before collecting and collating the results and sending them back to the client. It essentially acts as a work manager, distributing the in-coming requests to appropriate nodes and responding back to the client. It's just a smart load balancer.

#### CONFIGURING ROLES

When we start up the Elasticsearch in development mode, the node is by default set with master, data, and ingest roles (and of course each node is by default a coordinator - there is no special flag to enable or disable a coordinator). We can configure these roles as per our needs, for example, in a cluster of 10 nodes, we can enable one node as a master, 6 as data nodes, 2 as ingest nodes and so on.

All we need to do is tweak a `node.roles` setting in the `elasticsearch.yml` configuration file to configure a role on a node.

Multiple node roles can be set as shown in the following example:

```yaml
// This node dons four roles: master, data, ingest, and machine learning
node.roles: [master, data, ingest, ml]
```

## 3.3 Inverted indexes

If you look at the back of any book, usually you’ll find an index which maps keywords to the pages where they are found. This is actually nothing but a physical representation of an inverted index.

### 3.3.1 Example

Say we have two documents with one text field greeting:

```json
 //Document 1
{
  "greeting":"Hello, World"
}
//Document 2
{
  "greeting":"Hello, Mate"
}
```

In Elasticsearch, the analysis process is a complex function carried out by an analyzer module. The analyzer module is further composed of character filters, a tokenizer, and token filters. When the first document is indexed, as in the greeting field (a text field), an inverted index is created. Every full-text field is backed up by an inverted index. The value of the greeting “Hello, World” is analyzed so that it gets tokenized and normalized into two words, hello and world, by the end of the process. But there are few steps in between.

`<h2>Hello WORLD</h2> -> Hello WORLD -> [Hello, WORLD] -> [hello, world]`

After these steps, an inverted index is created for this field.

| Word  | Frequency | Document ID |
|-------|-----------|-------------|
| hello | 1         | 1           |
| world | 1         | 1           |

After ingesting second document:

| Word  | Frequency | Document ID |
|-------|-----------|-------------|
| hello | 2         | 1,2         |
| world | 1         | 1           |
| mate  | 1         | 2           |

## 3.4 Relevancy

Modern search engines not only return results based on our query’s criteria but also analyze and return the most relevant results.

### 3.4.1 Relevancy algorithms

Stack Overflow applies a set of relevancy algorithms to sort the results it returns to the user. Similarly, Elasticsearch returns the results for full-text queries sorted, usually, by a score it calls a relevancy score. Relevancy is a positive floating-point number that determines the ranking of the search results. Elasticsearch uses the **BM25 (Best Match)** relevancy algorithm by default for scoring the return results so the client can expect relevant results.

### 3.4.2 Relevancy (similarity) algorithms

Elasticsearch employs a handful of relevance algorithms, the default being the Okapi Best Matching 25 (BM25) algorithm. Elasticsearch provides a module  called similarity that lets us apply the most appropriate algorithms if the default isn’t suited for our requirements.

#### THE OKAPI BM25 ALGORITHM

There are three main factors involved in associating a relevancy score with the results:
* The term frequency (TF) - Term frequency (TF) represents the number of times the search word appears in the current document’s field. In other words if we search Java and in three different titles: Head First Java, Effective Java, Mastering Java: Learning Core Java and Enterprise Java With Examples the last one would have the highest;
* Inverse document frequency (IDF) - The number of times the search word appears across the whole set of documents is the document frequency. If the  document frequency of a word is higher, we can deduce that the search word is indeed common across the whole index. This means that if the word appears  multiple times across all the documents in an index, it is a common term and, accordingly, it’s not that relevant. The words that appear often are not  significant. Words like a, an, the, it, and so forth are pretty common in a natural language; hence, they can be ignored.
* Field length norm - The field-length norm provides a score based on the length of that field: the search word occurring multiple times in a short field is more relevant. For example, a field with 100 words having 5 occurrences of a search word is less relevant than a field with 10 words with 3  occurrences.

#### Tweaking the similarity functions

Each of the similarity functions carry additional parameters so we can alter them to reflect precise search results. For example, although the BM25 function is already set with the optimal parameters, should we want to modify the function, we can do that easily by using the index settings API.

| Property | Default Value | Description |
|----------|---------------|-------------|
| k1       | 1.2           | Nonlinear term frequency saturation variable            |
| b        | 0.75          | TF normalization factor based on the document’s length            |

## 3.5 Routing Algorithm

Every document has a permanent home, i.e, it must belong to a particular primary shard. Elasticsearch uses a routing algorithm to distribute the document to the underlying shard when indexing. Routing is a process of allocating a home for a document to a certain shard with each of the documents stored into one and only one primary shard. Retrieving the same document will be easy too as the same routing function will be employed to find out the shard where that document belongs to.

The routing algorithm is a simple formula where Elasticsearch deduces the shard for a document during indexing or searching:

```shard_number = hash(document_id) % number_of_shards```

**That means, once an index is created, we cannot change the number of shards.**

What if we have not anticipated the data growth and unfortunately the shards may have been exhausted with the spike in data. Well, all is not lost, there's a way out - reindex your data. Reindexing effectively creates a new index with appropriate settings and copies the data from the old index to a new index.

## 3.6 Scaling

### 3.6.1 Scaling up (vertical scaling)

### 3.6.2 Scaling out (horizontal scaling)

# Chapter 4. Mapping

## 4.1 Overview of mapping
 
**Mapping is a process of defining and developing a schema definition representing the document's data fields and their associated data types**. Mapping tells the engine the shape and form of the data that’s being indexed. Being a document-oriented database, Elasticsearch expects a single mapping definition per index.

For example, a string field is treated as a text field, a number field is stored as an integer, a date field is indexed as a date to allow for date-related operations, and so on.

### 4.1.1 Mapping definition

### 4.1.2 Indexing a document for the first time

Say we have a movie document that we want to index:

```shell
PUT movies/_doc/1
{
  "title":"Godfather", #A The title of the movie
  "rating":4.9, #B The rating given to the movie
  "release_year":"1972/08/01" #C Movie’s release year(note the date format)
}
```

This would be our first document sent to Elasticsearch to get indexed. Remember, we didn’t create an index (movies) or the schema for this document data prior to the document ingestion. Here’s what that happens when this document hits the engine:
* A new index (movies) is created automatically with default settings.
* A new schema is created for the movies index with the data types (we learn about data types shortly) deduced from this document’s fields. For example, title is set to a text and keyword types, rating to a float, and release_year to a date type.
* The document gets indexed and stored in the Elasticsearch data store.
* Subsequent documents get indexed without undergoing the previous steps as Elasticsearch consults the newly created schema for further indexing.

Elasticsearch uses a feature called dynamic mapping to deduce the data types of the fields when a document is indexed for the first time by looking at the field values and deriving these types.

While dynamic mapping is intelligent and convenient, be aware that it can get the schema definitions wrong too. Elasticsearch can only go so far when deriving the schema based on our document’s field values.

## 4.2 Dynamic mapping

### 4.2.1 The deducing types mechanism

### 4.2.2 Limitations of dynamic mapping

## 4.3 Explicit mapping

The following lists two possible ways of creating (or updating) a schema explicitly:
* Indexing APIS - We can create a scheam definition at the time of index creation using the create index API (no the mapping API) for this purpose. The create index API expects a request cosisting of the required schema definition in the form of a JSON document.
* Mapping APIs — As our data model matures, at times there will be a need to update the schema definition with new attributes. Elasticsearch provides a ``_mapping`` endpoint to carry out this action, allowing us to add the additional fields and their data types.

```shell
PUT movies
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text"
      }
    }
  }
}
```

```shell
PUT movies/_mapping
{
  "properties": {
    "release_date": {
      "type": "date",
      "format": "dd-mm-yyyy"
    }
  }
}
```

### 4.3.1 Mapping using the indexing API

```shell
PUT employees
{
  "mappings": {
    "properties": {
      "name":{ "type": "text"},#A The name field is a text data type
      "age": {"type": "integer"},#B The age field is a number type
      "email": {"type": "keyword"},#C keyword type (email is structured data) 
      "address":{#D Address object in an inner object with further fields
        "properties": { #E Inner object properties 
          "street":{ "type":"text" },
          "country":{ "type":"text" }
        } 
      }
    } 
  }
}
```

### 4.3.2 Updating schema using the mapping API

```shell
PUT employees/_mapping #A The mapping endpoint to update the existing index 
{
  "properties":{
    "joining_date":{ #A The joining date field as date type
      "type":"date",
      "format":"dd-mm-yyyy" #C The expected date format
    },
    "phone_number":{
      "type":"keyword" #D Phone numbers are stored as-is
    } 
  }
}
```

### 4.3.3 Modifying the existing fields is not allowed

Once an index is live (the index was created with some data fields and is operational), any modifications of the existing fields on the live index is prohibited.

**The alternative is re-indexing technique.** 

Re-indexing operations source the data from the original index to a new index (with updated schema definitions perhaps). The idea is that we:
* Create a new index with the updated schema definitions.
* Copy the data from the old index into the new index using re-indexing APIs. The new index with new schema will be ready to use once the re-indexing is complete. The index is open for both read and write operations.
* Once the new index is ready, our application switches over to the new index.
* We shelf the old index once we confirm the new index works as expected.

```shell
POST _reindex
{
  "source": {"index": "orders"},
  "dest": {"index": "orders_new"}
}
```

### 4.3.4 Type coercion

At times, the JSON documents might have incorrect values, differing from the ones that are present in the schema definition. For example, an integer-defined field may be indexed with a string value. Elasticsearch tries to convert such inconsistent types, thus avoiding indexing issues. **This is a process known as type coercion.**

## 4.4 Data types

### 4.4.1 Data type classifications

Data types can be broadly classified under the following categories:

* Simple types — The common data types, representing strings (textual information), dates, numbers, and other basic data variants. The examples are text, boolean, long, date, double, binary, etc.
* Complex types — The complex types are created by composing additional types, similar to an object construction in a programming language where the objects can hold inner objects. The complex types can be flattened or nested to create even more complex data structures based on the requirements at hand. The examples are object, nested, flattened, join, etc.
* Specialized types — These types are predominantly used for specialized cases such as geolocation and IP addresses. The common example types are geo_shape, geo_point, ip, and range types such as date_range, ip_range and others.

| Type                    | Description                                                               |
|-------------------------|---------------------------------------------------------------------------|
| text                    | Represents textual information (unstructured text)                        |
| integer/long/short/byte | Represents a number                                                       |
| float/double            | Represents a floating-point number                                        |
| boolean                 | Represents a binary choice, true or false                                 |
| keyword                 | Represents structured text, text that must not be broken down or analyzed |
| object                  | Represents a JSON object                                                  |
| nested                  | Represents an array of objects                                            |

As you can imagine, this is not a comprehensive list of data types. As of writing this book for version 8.x, there are about 29 data types defined by Elasticsearch. Elasticsearch defines the types microscopically in some cases for the benefit of optimizing the search queries. For example, the text types are further classified into more specific types such as search_as_you_type, match_only_text, completion, token_count, and others.

### 4.4.2 Developing mapping schemas

## 4.5 Core Data Types

### 4.5.1 The text data type

#### ANALYZING TEXT FIELDS

Elasticsearch supports two types of text data: structured text and unstructured text. The unstructured text is the full-text data, usually written in a human-readable language such as English, Chinese, Portuguese, and so on. Efficient and effective search on unstructured text is what makes a search engine stand out. The unstructured or as commonly called the full text, undergoes an analysis process whereby the data is split into tokens, characters are filtered out, words are reduced to its root word (stemming), synonyms are added, and other natural language processing rules applied.

```"The movie was sick!!! Hilarious :) :) and WITTY ;) a KiLLer :emoji:"```

The tags, punctuation and special characters are stripped away using character filters. This is how it looks after this step:

```The movie was sick Hilarious and WITTY a KiLLer```

The sentence is broken down into tokens using a tokenizer resulting in:

```[the, movie, was, sick, Hilarious, and, WITTY, a, KiLLer]```

The tokens are changed to lowercase using token filters, so it looks like this:

```[the, movie, was, sick, hilarious, and, witty, a, killer]```

For example, if you choose an English analyzer, the tokens are deduced to be root words (stemming):

```[movi,sick,hilari, witti, killer]```

Did you notice the stemmed words like movi, hilari, witti? They are actually not real words per say but the incorrect spellings don’t matter as long as all the derived forms can match the stemmed words.

Remember, I mentioned earlier Elasticsearch defines the types microscopically, for example, further classifying the text fields into more specific types such as search_as_you_type, match_only_text, completion, token_count, and others? Let’s go over these specialized text types briefly.

#### TOKEN COUNT (TOKEN_COUNT) DATA TYPE
 
A specialized form of text data type, the token_count will help define a field that captures the number of tokens in that field. That is, say if we have defined a book’s title as a token_count, we could retrieve all the books based on the number of tokens a book has. Let’s create a mapping for this by creating an index with a title field.

```shell
PUT tech_books
{
  "mappings": {
    "properties": {
        "title": { #A The field’s name
          "type": "token_count", #B The title’s data type is token_count 
          "analyzer": "standard" #C The analyzer is expected to be provided
        } 
    }
  } 
}
```

```shell
PUT tech_books/_doc/1
{
  "title":"Elasticsearch in Action"
}
PUT tech_books/_doc/2
{
  "title":"Elasticsearch for Java Developers"
}
PUT tech_books/_doc/3
{
  "title":"Elastic Stack in Action"
}
```

We write a range query to fetch the books with title composed of more than 3 words (gt is short for greater than) but less than or equal to 5 (lte is short form for less than or equal to) words:

```shell
GET tech_books/_search
{
  "query": {
   "range": {
     "title": {
       "gt": 3,
       "lte": 5
      } 
    }
  } 
}
```

We can indeed combine the title field as a text type as well as a token_type too as Elasticsearch allows a single field to be declared with multiple data types.

```shell
PUT tech_books
{
  "mappings": {
    "properties": {
      "title": { #A The title field is defined as text data type
        "type": "text",
        "fields": { #B The title field is declared to have multiple data types
          "word_count": { #C The word_count is the additional field
            "type": "token_count", #D The type of word_count
            "analyzer": "standard"#E mandatory to provide the analyzer
          } 
        }
      } 
    }
  } 
}
```

### 4.5.2 The keywords data types

The keywords family of data types is composed of keyword, constant_keyword and wildcard field types.

#### THE KEYWORD TYPE

The structured data, such as pincodes, bank accounts, phone numbers, don’t need to be searched as partial matches or produce relevant results. The results tend to provide a binary output: returns results if a match or return none.

The ``keyword`` data type leaves the fields untouched. The field is untokenized and not analyzed.

```shell
PUT faculty
{
  "mappings": {
    "properties": {
      "email": { #A Define the email property
        "type": "keyword" #B Declaring email as keyword type
      }
    }
  }
}
```

#### THE CONSTANT_KEYWORD TYPE

Constant keyword is a specialization of the keyword field for the case that all documents in the index have the same value.

```shell
PUT census {
  "mappings": {
    "properties": {
      "country":{
        "type": "constant_keyword",
        "value":"United Kingdom"
      } 
    }
  } 
}
```

Now, we index a document for John Doe, with just his name (no country field):

```shell
PUT census/_doc/1
{
  "name":"John Doe"
}
```

#### THE WILDCARD DATA TYPE

The wildcard data is another special data type that belongs to the keywords family which supports searching data using wildcards and regular expressions.

##### Mapping unstructured content

You can map a field containing unstructured content to either a `text` or `keyword` family field.

Use the ``text`` field type if:
* The content is human-readable, such as an email body or product description.
* You plan to search the field for individual words or phrases, such as the brown fox jumped, using full text queries.

Use a ``keyword`` family field type if:
* The content is machine-generated, such as a log message or HTTP request information.
* You plan to search the field for exact full values, such as org.foo.bar, or partial character sequences, such as org.foo.*, using term-level queries.

### 4.5.3 Thedatedatatype

Elasticsearch provides a date data type for supporting indexing and searching date-based operations. The date fields are considered to be structured data; hence, you can use them in sorting, filtering, and aggregations.

```shell
PUT flights {
  "mappings": {
    "properties": {
      "departure_date_time":{
        "type": "date"
      } 
    }
  } 
}
```

We can of course change the format of the date.

```shell
"departure_date_time":{ #A Customizing the date format
  "type": "date",
  "format": "dd-MM-yyyy||dd-MM-yy" #B Date is set in any of these two values
}
```

In addition to accepting the date as a string value, we can also provide it in a number format - either seconds or milliseconds since epoch (1st January 1970).

```shell
{
    ...
    "string_date":{ "type": "date", "format": "dd-MM-yyyy" }, 
    "millis_date":{ "type": "date", "format": "epoch_millis" }, 
    "seconds_date":{ "type": "date", "format": "epoch_second"}
}
```

```shell
"range": {#A A range query fetching documents between two dates
  "departure_date_time": {
    "gte": "2021-08-06T05:00:00",#B In between 5 and 5.30am
    "lte": "2021-08-06T05:30:00"
    }
}
```

### 4.5.4 Numeric data types

Elasticsearch supplies a handful of numeric data types to handle integer and floating-point data:
* byte - Signed 8 bit integer;
* short - Signed 16 bit integer;
* integer - Signed 32 bit integer;
* long - Signed 64 bit integer;
* unsigned_long - 64 bit unsigned integer;
* float - 32 bit single precision floating-point number;
* double - 64 bit single precision floating-point number;
* half_float - 16 bit half precision floating-point number;
* scaled_float - Floating-point number backed by long;

### 4.5.5 The boolean data type

In addition to setting the field as JSON’s boolean type (true or false), the field also accepts “stringified” boolean values such as "true" or "false”.

### 4.5.6 The range data type

The range data types represent lower and upper bounds for a field. For example, if we want to select a group of volunteers for a vaccine trial, we can  segregate the volunteers based on some categories such as age 25–50, 51–70. 

The range is defined by operators such as lte (less than or equal to) and lt (less than) for upper bounds and gte (greater than or equal to) and gt  (greater than) for lower bounds.

#### THE DATE_RANGE TYPE EXAMPLE

```shell
PUT trainings
{
  "mappings": {
    "properties": {
      "name":{ #A: The name of the training session
        "type": "text"
      },
      "training_dates":{ #B The training_dates is declared as date_range type
        "type": "date_range"
      }
    } 
  }
}
```

```shell
PUT trainings/_doc/1 #A First document
{
  "name":"Functional Programming in Java",
  "training_dates":{ #B Set of training dates
    "gte":"2021-08-07",
    "lte":"2021-08-10"
  }
}
PUT trainings/_doc/2 #C First document
{
  "name":"Programming Kotlin",
  "training_dates":{#D Set of training dates
    "gte":"2021-08-09",
    "lte":"2021-08-12"
  }
}
PUT trainings/_doc/3 #E First document
{
  "name":"Reactive Programming",
  "training_dates":{ #F Set of training dates
    "gte":"2021-08-17",
    "lte":"2021-08-20"
  }
}
```

The data_range type field expects two values: an upper bound and a lower bound. In addition to date_range, we can create other ranges like ip_range,  float_range, double_range, integer_range, and so on.

### 4.5.7 The IP (ip) address data type

Elasticsearch provides a specific data type to support internet protocol (IP) addresses. This data type supports both IPv4 and IPv6 IP addresses.

```shell
PUT networks
{
  "mappings": {
    "properties": {
      "router_ip":{ "type": "ip" } #A Type of the field is ip }
    } 
  }
}
```

## 4.6 Advanced data types

### 4.6.1 The Geopoint (geo_point) data type

With the advent of smartphones and devices, location services and searching for nearest items have become more common. Location data is expressed as a  geo_point data type, which represents longitude and latitude.

```shell
PUT restaurants/_doc/1
{
  "name":"Sticky Fingers",
  "address":{ #A The address is provided as a pair of longitude and latitude
    "lon":"0.1278",
    "lat":"51.5074"
  }
}
```

We can fetch restaurants using a geo_bounding_box query which is used for searching data involving geographical addresses. It takes inputs of top_left and bottom_right points to create a boxed up area around our point of interest.

```shell
GET restaurants/_search
{
  "query": {
    "geo_bounding_box":{
      "address":{#A The top_left part of the box
        "top_left":{
          "lon":"0",
          "lat":"52"
        },
        "bottom_right":{#B The bottom_right part of the box
          "lon":"1",
          "lat":"50"
        } 
      }
    } 
  }
}
```

The query searches the restaurants that fall in a geo bounding box, represented as two geopoints (top_left and bottom_right in the query). This query  fetches our restaurant because the geo bounding box encompasses our restaurant.

We can provide the location information in various formats.

| Format  | Explanation                                                                                                                                             | Example               |
|---------|---------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------|
| Array   | Geo-point represented as an array. Note the order of geo-point inputs - it takes lon and lat (not the other easy as string format considers).           | "address":[0.12,51.5] |
| String  | Geo-point as string data with lat and lon data.                                                                                                         | "address":"51.5,0.12" |
| Geohash | Geohash is an encoded string formed from hashing the longitude and latitude coordinates. The alphanumeric string points to a place on the earth         | u10j4 |
| Point   | Known as a well-known-text (KWT), the POINT represents a precise location on a map. The WKT is a standard mechanism to represent the geometrical data.  | POINT(51.5,-0.12) |

### 4.6.2 The object data type

Often we find data in a hierarchical manner - for example - an email object consisting of top level fields like subject as well as inner object to hold  attachments, which in turn may have few more properties such as attachment file name, its type and so on.

```shell
PUT emails {
  "mappings": {
    "properties": { #A The top level properties for the emails index
      "to": {
        "type": "text"
      },
      "subject":{
        "type": "text"
      },
      "attachments":{ #B The inner object consisting of second level properties "properties": {
          "filename":{
            "type":"text"
          },
          "filetype":{
            "type":"text"
          }
      } 
    }
  } 
}
```

The attachments property is something we should draw our attention to. The type of this field is an object as it encapsulates the two other fields.

Once the command is executed successfully, we can check the schema by invoking ``GET emails/_mapping`` command:

```shell
{
  emails" : {
    "mappings" : {
      "properties" : {#A Attachments is an inner object with other fields
        "attachments" : {#B The type is hidden here but it’s object by default
          "properties" : {
            "filename" : {
              "type" : "text",#C Field’s type shown as expected
              ...
}
```

When you fetch the mapping (GET `emails/_mapping`), while all other fields show their associated data types, the ``attachments`` wouldn’t. **The object type of an inner object is inferred by Elasticsearch as default.**

#### LIMITATION OF AN OBJECT TYPE

In our earlier emails example, the attachments field was declared as an object. While we create the email with just one attachment object, there’s nothing stopping us creating multiple of these attachments.

```shell
PUT emails/_doc/2
{
  "to:":"mrs.doe@johndoe.com",
  "subject":"Multi attachments test",
  "attachments":[{
    "filename":"file2.txt",
    "filetype":"confidential"
  },{
    "filename":"file3.txt",
    "filetype":"private"
  }]
}
```

Searching for the matching documents given a filename ``file2.txt`` and file type ``private`` should return no result. 

```shell
GET myemails/_search #A Bool query search for a match with a filename and filetype
{
  "query": {
    "bool": {#B The query is defined as a bool query
      "must": [ #C The must clause defining the mandatory clauses 
        {"term": { "attachments.filename.keyword": "file2.txt"}}, 
        {"term": { "attachments.filetype.keyword": "private" }}
      ] 
    }
  } 
}
```

However, this returns results. And this is where the object data type breaks down, i.e., it can’t honour the relationships between the inner objects. The reason for this is that the inner objects are not stored as individual documents, they are flattened as shown below:

```shell
{
...
  "attachments.filename" :["file1.txt","file2.txt","file3.txt"]
  "attachments.filetype":["private","confidential"]
}
```

### 4.6.3 The nested data type

We can fix this issue by introducing a new type called nested data type. The nested type is a specialized form of an object type where the relationship  between the arrays of objects in a document is maintained.

Going with the same example of our emails and attachments, this time let’s define the attachments fields as nested data type, rather than letting Elasticsearch derive it as an object type.

```shell
PUT emails_nested
{
  "mappings": {
    "properties": {
      "attachments": {
        "type": "nested", #A The attachments field is declared as nested type
        "properties": {
          "filename": {#B The field is declared as keyword to avoid tokenizing
            "type": "keyword"
          },
          "filetype": {
            "type": "text" #C We can leave this field as text }
          } 
        }
      } 
    }
  }
}
```

In addition to creating the attachments as nested types, we declared the ``filename`` as a ``keyword`` type for a reason. The value, for example: ``file1.txt``, gets tokenized and gets split up as ``file1`` and ``txt``. As a result, search query may get matched with a ``txt`` and ``confidential`` or ``txt`` and ``private`` as both records have ``txt`` as common token. To avoid this, we simply use the ``filename`` field as a ``keyword`` field.


```shell
GET emails_nested/_search
{
  "query": {
    "nested": { #A Formulating a nested query to fetch data from nested fields
      "path": "attachments",#B Path points to the name of the nested field
      "query": {
        "bool": {
          "must": [#C Search clauses: must match with file1.txt and private
            { "match": { "attachments.filename": "file1.txt" }},
            { "match": { "attachments.filetype":  "private" }}
          ]
        } 
      }
    } 
  }
}
```

### 4.6.4 Flattened (flattened) data type

So far we’ve looked at indexing the individual fields parsed from a JSON document. Each of the fields is treated as an individual and independent field when analyzing and storing it. However, sometimes we may not need to index all the subfields as individual fields thus going through the analysis process.

A ``flattened`` data type holds information in the form of one or more subfields, each subfield’s value indexed as a keyword. That is, none of the values are treated as text fields, thus do not undergo the text analysis process.

Let’s consider an example of a doctor taking running notes about his/her patient during the consultation. The mapping consists of two fields: the name of the patient and the doctor notes.


```shell
PUT consultations
{
  "mappings": {
    "properties": {
      "patient_name":{
        "type": "text"
      },
      "doctor_notes":{# A This field composes of any number of fields
        "type": "flattened" #B Field is declared as flattened
      }
    } 
  }
}
```

```shell
PUT consultations/_doc/1
{
  "patient_name":"John Doe",
  "doctor_notes":{# The flattened field can hold any number fields
    "temperature":103, # All these fields are indexed as keywords
    "symptoms":["chills","fever","headache"],
    "history":"none",
    "medication":["Antibiotics","Paracetamol"]
  } 
}
```

As the doctor_notes is a flattened type, all the values are indexed as keywords, that is, the analyzers are at bay when we index flattened field data.

```shell
GET consultations/_search
{
  "query": {
    "match": {
      "doctor_notes": "Paracetamol" # Searching for patient’s medication
    }
  } 
}
```

The flattened data types come handy especially when we are expecting a lot of fields on an adhoc basis and having to define the mapping definitions for all of them beforehand isn’t feasible. **Be mindful that the subfiles of a flattened field are always ``keyword`` types.**

### 4.6.5 The Join (join) data type

If you are from a relational database world, you would know the relationships between data - the joins that enable the parent-child relationships.

While maintaining and managing relationships in Elasticsearch is advised under caution, Elasticsearch provides a join data type to consider parent-child relationships should we need them.

Let’s learn about join data in action by considering an example of doctor - patients (one-to-many) relationship: one doctor can have multiple patients and each patient is assigned to one doctor.

```shell
PUT doctors {
  "mappings": {
    "properties": {
      "relationship":{#A declare a property as join type
        "type": "join",
        "relations":{
          "doctor":"patient" #B Names of the relations
        }
      } 
    }
  } 
}
```

The query has two important points to note:
* Declare a relationship property of type ``join``
* Declare a ``relations`` attribute and mention the names of the relations (doctor:patient, teacher:students, parent-children etc)

Once we have the schema ready and indexed, we index two types of documents: one representing the doctor (parent) and the other patients (child).

```shell
PUT doctors/_doc/1
{
  "name":"Dr Mary Montgomery",
  "relationship":{
    "name":"doctor" #A The relationship attribute must be one of the relations }
}
```

The relationship object declares the type of the document this is: doctor. The name attribute must be parent value (doctor) as declared in the mapping schema.

```shell
PUT doctors/_doc/2?routing=mary #A Documents must have the routing flag set
{
  "name":"John Doe",
  "relationship":{ #B Define the type of relationship in this object
    "name":"patient", #C The document is a patient
    "parent":1 #D Patient’s patent (doctor) is document with ID 1
  }
}
PUT doctors/_doc/3?routing=mary
{
  "name":"Mrs Doe",
  "relationship":{
    "name":"patient",
    "parent":1 
  }
}
```

**The parents and associated children will be indexed into the same shard to avoid the multi-shard search overheads. And as the documents should co-exist, we need to use a mandatory routing parameter in the URL. Routing is a function that would determine the shard where the document will reside.**

```shell
GET doctors/_search
{
  "query": {
    "parent_id":{
      "type":"patient",
      "id":1 
    }
  } 
}
```

When we wish to fetch the patients belonging to a doctor, we use a search query called parent_id that would expect the child type (patient) and the parent’s ID (Dr Montgomery document ID is 1). This query will return Dr Montgomery’s patients - Mr and Mrs Doe.

### 4.6.6 Search as you type data type

Most search engines suggest words and phrases as we type in a search bar. Elasticsearch provides a convenient data type - ``search_as_you_type`` - to support this feature.

```shell
PUT tech_books
{
  "mappings": {
    "properties": {
      "title": {
        "type": "search_as_you_type"#A The title supports typeahead feature
      }
    } 
  }
}
```

```shell
PUT tech_books4/_doc/1
{
  "title":"Elasticsearch in Action"
}
PUT tech_books4/_doc/2
{
  "title":"Elasticsearch for Java Developers"
}
PUT tech_books4/_doc/3
{
  "title":"Elastic Stack in Action"
}
```

As the title field’s type is of search_as_you_type data type, Elasticsearch creates a set of subfields called ngrams, in addition to the root field(title), with various shingle token filters. For example:
* `title._2gram`: For example, the 2grams for “action” word are: ["ac","ct","ti","io","on"]
* `title._3gram`: For example, the 3grams for “action” word ar: ["act", "cti","tio","ion"]

```shell
GET tech_books4/_search
{
  "query": {
    "multi_match": {
      "query": "in",
      "type": "bool_prefix",
      "fields": ["title","title._2gram","title._3gram"]
    } 
  }
}
```

This query should return the Elasticsearch in Action and Elastic Stack in Action books. We use a multi match query because we are searching for a value  across multiple fields - title, `title._2gram`, `title._3gram`, `title._index_prefix`.

#### Ngrams, Edge ngrams and Shingles

The ngrams are a sequence of words for a given size. You can have 2-ngrams, 3-ngrams etc. For example, if the word is “action”, the 3-ngram (ngrams for size3) are: ["act", "cti","tio","ion"] and bi-grams (size 2) are: ["ac", "ct","ti","io","on"] and so on.

Edge ngrams, on the other hand, are ngrams of every word, where the start of the n-gram is anchored to the beginning of the word. Considering the “action” word as our example, the edge n-gram produces: ["a", "ac","act","acti","actio","action"].

Shingles on the other hand are word n-grams. For example, the sentence “Elasticsearch in Action” will outputting: ["Elasticsearch", "Elasticsearch in", "Elasticsearch in Action", "in", "in Action", "Action"]

## 4.7 Multiple data types

We learned that each field in a document is associated with a data type. However, Elasticsearch is flexible to let us define the fields with multiple data types too.

```shell
{
  "my_field1":{
    "type": "text", #A Declare the type of my_field1
    "fields": {#B Define a fields object to enclose more types
      "kw":{ "type":"keyword" }#C Declare additional field with a label "kw" }
}
```

```shell
PUT emails {
  "mappings": {
    "properties": {
      "subject":{
        "type": "text", #A The text type 
        "fields": {
          "kw":{ "type":"keyword" }, #B The subject is also a keyword
          "comp":{ "type":"completion" } #C Subject is completion type too 
        }
      } 
    }
  } 
}
```

The subject field now has three types associated with it: text, keyword, and completion. If you want to access these, you have to use the format `subject.kw` for the keyword type field or `subject.comp` for the completion type.

# Chapter 5. Working with documents

Elasticsearch classifies the APIs into two categories: single document APIs and multi-document APIs.

## 5.1 Indexing documents

#### DOCUMENT IDENTIFIERS

Each document that we index can have an identifier, usually specified by the user. These identifiers, similar to a primary key in a relational database, are associated with that document for its lifetime.

On the other hand, there are times when the client (user) doesn’t really need to give an identifier to the document. Each of these messages doesn’t need to have a sequence of identifiers. They can exist with a random ID, albeit a unique one. In this case, the system generates a random identifier (UUID) for the document that’s getting indexed.

Document APIs allow us to index documents with and without an identifier. There is a subtle difference in using the HTTP verb such as POST or PUT, however.
* If a document has an identifier provided by the client, we use the HTTP PUT method to invoke a document API for indexing the document.
* If the document does not have an identifier provided by the client, we use the HTTP POST method when indexing. In which case, the document inherits the system-generated identifier once indexed.

#### INDEXING A DOCUMENT WITH AN IDENTIFIER (PUT)

```shell
PUT <index_name>/_doc/<identifier>
```

```shell
PUT movies/_doc/1 #A Document indexing url 
{ #B Body of the request
  "title":"The Godfather",
  "synopsis":"The aging patriarch of an organized crime dynasty transfers control of his clandestine empire to his reluctant son"
}
```

#### INDEXING A DOCUMENT WITHOUT AN IDENTIFIER (POST)

```shell
POST myindex/_doc #A The URL has no identifier attached to it 
{
  "title":"Elasticsearch in Action" #B The request body is a JSON document 
}
```

#### USING `_CREATE` TO AVOID OVERRIDING A DOCUMENT

```shell
PUT movies/_doc/1
{
  "tweet":"Elasticsearch in Action 2e is here!" #A Not a movie but a tweet 
}
```

In the example, we are indexing a document with a tweet into a movies index with a document ID of 1. Hold on a second; didn’t we already have a movie document (The Godfather) with that ID in our store? Yes, we do. Elasticsearch has no control on such overwriting operations. **The responsibility is passed down to the application or to the user.**

We can use the ``_create`` endpoint in place of ``_doc`` when indexing a document to avoid overriding the existing one.

```shell
PUT movies/_create/100 {
  "title":"Mission Impossible",
  "director":"James Cameron" 
}
```

```shell
PUT movies/_create/100 #A Index a tweet in place of an existing movie 
{
  "tweet":"A movie with popcorn is so cool!" #B Not a movie but a tweet 
}
```

```shell
{
    "type" : "version_conflict_engine_exception",
    "reason":"[100]:version conflict,document already exists(current version[1])"
}
```

Elasticsearch didn’t let the data be overwritten.

Elasticsearch, by default, auto-creates a required index if the index doesn't already exist. If we want to restrict this feature, we need to set a flag called ``action.auto_create_index`` to ``false``.

### 5.1.2 Mechanics of indexing

Shards are Lucene instances that hold the physical data that’s logically associated with an index.

When we index a document, the engine decides which shard it will be housed in, based on the routing algorithm. Each shard comes with heap memory, and when a document is indexed, the document is first pushed into the shard's in-memory buffer. The document is held in this in-memory buffer until a refresh occurs.
Lucene’s scheduler issues a refresh every 1 second to collect all the documents available in the in-memory buffer, and creates a new segment with these documents. The segment consists of the document data and inverted indexes. The data is first written to the filesystem cache and then committed to the physical disk.

Apache Lucene is an intelligent library when dealing with data writes and reads. After pushing the documents to a new segment (during the refresh operation), it waits until three segments are formed. It uses a three-segments-merge pattern to merge the segments to create new segments: that is, whenever the three segments are ready, Lucene will instantiate a new one by merging these three segments. And awaits for three more segments to be created so it can create a new one and so on.

### 5.1.3 Customizing the refresh

Documents that get indexed live in memory until the refresh cycle kicks in. The documents get moved as segments into the filesystem during the refresh process and, thus, are available to search.

The good news is that we can configure this refresh setup. We can reset the time interval from the default 1 second to, say, 60 seconds.

```shell
PUT movies/_settings {
"index":{ "refresh_interval":"60s"} 
}
```

To switch off the refresh operation completely, set the value to -1. The in-memory buffer will accumulate the incoming documents if the refresh operation is off. The use case for this scenario might be that we are migrating tons of documents from a database into Elasticsearch, and we don’t want the data to be searchable until the migration completes successfully.

We can also control the refresh operation from the client side for CRUD operations on documents by setting the refresh query parameter.

The refresh query parameter can expect three values:
* refresh=false
* refresh=true
* refresh=wait_for

## 5.2 Retrieving documents

Elasticsearch provides two types of document APIs for retrieving documents:
* The single document API that returns one document given an ID, and
* The multi-document API that returns multiple documents given an array of IDs.

### 5.2.1 Using the single document API

```shell
GET <index_name>/_doc/<id>
```

The JSON response has two parts: the metadata and the original document. The metadata consists of `_id`, `_type`, `_version`, and so on. The original document is enclosed under the _source attribute.

If the document is not found, we’ll get a response with the attribute found set to false.

```shell
{
  "_index" : "movies", "_type" : "_doc", "_id" : "999", "found" : false
}
```

### 5.2.2 Retrieving multiple documents

Elasticsearch exposes a multi-document API (``_mget``).

```shell
GET movies/_mget
{
"ids" : ["1", "12", "19", "34"] 
}
```

```shell
GET _mget # A _mget call with no index mentioned in the url 
{
"docs":[ 
  {
    "_index":"classic_movies", #B The index is provided here
    "_id":11 
  },
  {
    "_index":"international_movies", #C index 2 
    "_id":22
  }, 
  {
    "_index":"top100_movies", #D index 3
    "_id":33 
  }
]}
```

```shell
GET classic_movies/_search
{
"query": {
  "ids": {
    "values": [1,2,3,4]
    } 
  }
}
```

## 5.3 Manipulating responses

The response returned to the client can contain a lot of information, and the client may not be interested in receiving all of it.

### 5.3.1 Removing metadata from the response

We can fetch just the source (original document) without the metadata by:

```shell
GET <index_name>/_source/<id>
```

Notice that the ``_doc`` endpoint is replaced with ``_source``.

### 5.3.2 Suppressing the source document

```shell
GET movies/_doc/1?_source=false
```

Only the metadata will be returned.

### 5.3.3 Including and excluding fields

```shell
GET movies/_doc/3?_source_includes=title,rating,genre
```

This returns the document with these three fields.

#### Exclude fields (_source_excludes)

We can exclude fields that we don’t want to be returned in the response using the ``_source_excludes`` parameter.

```shell
GET movies/_source/3?_source_excludes=actors,synopsis
```

#### Include and exclude fields

We can mix and match what the return attributes we wish to have in our response.

```shell
GET movies/_source/3?_source_includes=rating*&_source_excludes=rating_amazon
```

We’ve enabled a wildcard field, ``_source_includes=rating*``, in this query to fetch all the attributes that are prefixed with the word rating.

## 5.4 Updating documents

Similar to indexing documents, Elasticsearch provides two types of update queries: one for working against single documents and other for working on  multiple documents:
* ``_update`` API will update a single document
* ``_update_by_query`` will allow us to modify multiple documents in one go

### 5.4.1 Document update mechanics

Elasticsearch first fetches the document, modifies it, and then re- indexes it. Essentially, it replaces the old document with a new document. Behind the scenes, Elasticsearch creates a new document for updates. During this update, Elasticsearch increments the version of the document once the update  operation completes. When the newer version of the document (with the modified values or new fields) is ready, it marks the older version for deletion.

### 5.4.2 The _update API

All update operations are performed using the ``_update`` API. The format is simple, and it goes like this:

```shell
POST <index_name>/_update/<id>
```

#### ADDING NEW FIELDS

To amend the document with new fields, we pass a request body consisting of the new fields to the `_update` API. The new fields are wrapped in a doc object as the API expects in that manner.

```shell
POST movies/_update/1 {
  "doc": {
  "actors":["Marldon Brando","Al Pacino","James Cann"], "director":"Frances Ford Coppola"
  } 
}
```

#### MODIFYING THE EXISTING FIELDS

```shell
POST movies/_update/1 {
  "doc": {
    "title":"The Godfather (Original)"
  } 
}
```

When it comes to updating an element in an array (like adding a new actor to the list of actors field), we must provide both new and old values together.

```shell
POST movies/_update/1 #A Updating the document ID 1 
{
    "doc": { #B the updates must be enclosed in the doc object "actors":
        ["Marldon Brando", "Al Pacino", "James Cann", "Robert Duvall"] #C Old and new values together
    } 
}
```

### 5.4.3 Scripted updates

In addition to doing this on a field-by-field basis, we can also execute updates using scripts. Scripted updates allow us to update the document based on some conditions.

Scripts are provided in a request body using the same `_update` endpoint, with the updates wrapped in a script object which in turn consist of the source as the key.

#### UPDATE ACTORS USING A SCRIPT

Let’s update our movie document by adding an additional actor to the existing array.

```shell
POST movies/_update/1 {
    "script": {
      "source": "ctx._source.actors.add('Diane Keaton')"#A Additional actor
    } 
}
```

#### REMOVING AN ACTOR

The ``remove`` method takes an integer that points to the index of the actor we want to remove.

```shell
POST movies/_update/1 
{
  "script":{
    "source": "ctx._source.actors.remove(ctx._source.actors.indexOf('Diane Keaton'))"
  }
}
```

#### ADDING A NEW FIELD

Here we add a new field, ``imdb_user_rating``, with a value of 9.2:

```shell
POST movies/_update/1 {
    "script": {
      "source": "ctx._source.imdb_user_rating = 9.2"
    } 
}
```

#### REMOVING A FIELD

```shell
POST movies/_update/1 {
    "script": {
      "source": "ctx._source.remove('metacritic_rating')"
    }
}
```

Note that if you try to remove a nonexistent field, you will not get an error message letting you know you are trying to remove a field that doesn’t exist on the schema.

#### ADDING MULTIPLE FIELDS

```shell
POST movies/_update/1 {
    "script": {
        "source": """
          ctx._source.runtime_in_minutes = 175;
          ctx._source.metacritic_rating= 100;
          ctx._source.tomatometer = 97;
          ctx._source.boxoffice_gross_in_millions = 134.8;
          """
    }
}
```

Each key-value pair is segregated with a semicolon (;).

#### ADDING A CONDITIONAL UPDATE SCRIPT

```shell
POST movies/_update/1 {
"script": {
    "source": """ 
    if(ctx._source.boxoffice_gross_in_millions > 125)
      {ctx._source.blockbuster = true} 
    else
      {ctx._source.blockbuster = false}
    """
  } 
}
```

The if clause in the listing checks for the value of the field ``boxoffice_gross_in_millions``. It then creates a new blockbuster field automatically.

#### ANATOMY OF A SCRIPT

The source is where we provide the logic, while the params are the parameters that the script expects separated by a vertical bar (or pipe) character. We can provide the language our script is written in (for example, one of painless, expression, moustache, or java, where the default is painless).

#### PASSING DATA TO THE SCRIPTS

```shell
POST movies/_update/1 {
    "script": { #A Business logic goes here
        "source": """ #B checking the value against params
        if(ctx._source.boxoffice_gross_in_millions > params.gross_earnings_threshold) 
          {ctx._source.blockbuster = true}
        else
          {ctx._source.blockbuster = false}
        """,
        "params": {#C provide the parameter values
          "gross_earnings_threshold":150 
        }
    }
}
```

The script has two notable changes from the previous version:
* The if clause is now compared against a value read from the params object (params.gross_earnings_threshold), and
* The `gross_earnings_threshold` is set to 150 via the params block

### 5.4.4 Replacing documents

```shell
PUT movies/_doc/1
{
  "title":”Avatar”
}
```

### 5.4.5 Upsert

Let’s say we want to update gross_earnings for the movie Top Gun. Remember, we don’t have this movie in our store yet.

```shell
POST movies/_update/5 {
  "script": {
    "source": "ctx._source.gross_earnings = '357.1m'"
  },
  "upsert": {
    "title":"Top Gun",
    "gross_earnings":"357.5m" 
  }
}
```

### 5.4.6 Updating by a query

Sometimes we wish to update a set of documents matching to a search criteria - for example update all movies whose rating is above 4 stars as popular movies. We can run a query to search for such a set of documents and apply the updates on those by using the `_update_by_query` endpoint. Say, we need to update all the movies with the actor’s name matching Al Pacino to Oscar Winner Al Pacino in all the movies that he acted in.

```shell
POST movies/_update_by_query #A We search for the docs and update the set {
  "query": { #B Search query - search all movies of Pacino "match": {
    "actors": "Al Pacino" 
  }
  },
  "script": { #C Apply the following script logic for the matching documents
    "source": """
      ctx._source.actors.add('Oscar Winner Al Pacino'); 
      ctx._source.actors.remove(ctx._source.actors.indexOf('Al Pacino'))
      """,
      "lang": "painless"
    }
}
```

## 5.5 Deleting documents

When you want to delete documents, there are essentially two methods: using an ID or using a query. In the former case, we can delete a single document, while in the latter, we can delete multiple documents in one go.

### 5.5.1 Deleting with an ID

```shell
DELETE <index_name>/_doc/<id>
```

### 5.5.2 Deleting by query (`_delete_by_query`)

```shell
POST movies/_delete_by_query {
  "query":{
    "match":{
      "director":"James Cameron" 
    }
  } 
}
```

Similar to the basic search queries we can pass a variety of attributes like ``term``, ``range``, ``match``.

## 5.7 Reindexing documents

```shell
POST _reindex
{
    "source": {"index": "<source_index>"},
    "dest": {"index": "<destination_index>"} 
}
```

```shell
POST _reindex
{
    "source": {"index": "movies"},
    "dest": {"index": "movies_new"} 
}
```

# Chapter 6. Indexing operations

Indices come with three sets of configurations: settings, mappings, and aliases. Each of these configurations modify the index in one form or another.

For example, we use settings for creating the number of shards and replicas as well as other properties for the index.

## 6.1 Indexing operations

A newly created index is associated with a set number of shards and replicas along with other attributes by default.

## 6.2 Creating indices

### 6.2.1 Creating indices implicity (automatic creation)

When we index a document for the first time, Elasticsearch won't make any complaints about a non existing index; instead, it happily creates one for us. 
When an index is created this way, Elasticsearch uses default settings such as setting the number of primary and replica shards to one.

Each index is made of **mappings, settings, and aliases**. This will be reflected in the response.

```shell
PUT _cluster/settings #A Updates the settings across the whole cluster
{
  "persistent": { #B The changes can be persistent or transient.
    "action.auto_create_index":false #C Shuts down autocreation
  }
}
```

The ``persistent`` property indicates that the settings will be permanent. On the contrary, using the ``transient`` property saves the settings only until the next reboot of the Elasticsearch server.

While disabling this feature sounds cool, **in reality this is not advisable**. Although we are restricting automatic creation, there may be an instance of an application or Kibana that may need to create an index for administrative purposes; Kibana creates hidden indices often.

Having said that, there is a mechanism to tweak this property beyond binary options. This setting allows the automatic creation of hidden indices with admin as the prefix.

```shell
action.auto_create_index: .admin*, cars*, *books*, -x*,-y*,+z*
```

### 6.2.2 Creating indices explicitly

```shell
PUT <index_name>
```

### 6.2.3 Index with custom settings

For this purpose, Elasticsearch exposes the `_settings` API to update the settings on a live index. However, as we mentioned, not all properties can be altered on a live index, only dynamic properties.

#### STATIC SETTINGS

Static settings can only be applied during the process of index creation and cannot be changed while the index is in operation. These are properties like the number of shards, compression codec, data checks on startup, and others.

#### DYNAMIC SETTINGS

Dynamic settings are those settings that we can modify on a live (the index that’s in operation) index. For example, we can change properties like number of replicas, allowing or disallowing writes, refresh intervals, and others on indices that are in operation.

```shell
PUT cars_with_custom_settings #A
{
  "settings":{ #B
    "number_of_shards":3, #C
    "number_of_replicas":5, #D
    "codec": "best_compression", #E
    "max_script_fields":128, #F
    "refresh_interval": "60s" #G
  } 
}
```

```shell
PUT cars_with_custom_settings/_settings
{
  "settings": {
    "number_of_replicas": 2
  }
}
```

Fetch index settings:
```shell
GET cars_with_custom_settings/_settings
```

You can also fetch settings of multiple indices:
```shell
GET cars1,cars2,cars3/_settings #A
GET cars*/_settings #B
```

Fetching a single attribute:
```shell
GET cars_with_custom_settings/_settings/index.number_of_shards
```

### 6.2.4 Index with mappings

```shell
PUT cars_with_mappings
{
  "mappings": { #A The mappings object encloses the properties.
    "properties": { #B The fields with data types of the car are declared here.
      "make":{
        "type": "text" #C Declaring the make as a text type
      },
      "model":{
        "type": "text"
      },
      "registration_year":{#D Declaring the registration_year as a date type
        "type": "date",
        "format": "dd-MM-yyyy" #E The field’s custom format
      }
    } 
  }
}
```

We can also combine both settings and mappings.

```shell
PUT cars_with_settings_and_mappings #A Index with both settings and mappings
{
  "settings": { #B Settings on the index
    "number_of_replicas": 3
  },
  "mappings": { #C Mappings schema definition
    "properties": {
      .. 
    }
  } 
}
```

### 6.2.5 Index with aliases

Aliases are alternate names given to indices for various purposes such as:
* Searching or aggregating data from multiple indices (as a single alias)
* Enabling zero downtime during re-indexing

We can group multiple indices and assign an alias to them so one can write queries against a single alias rather than dozen indices.

```shell
PUT cars_for_aliases #A Creates an index with an alias
{
  "aliases": {
    "my_new_cars_alias": {} #B Points the alias to the index
  }
}
```

However, there’s another way to create aliases rather than using the index APIs: using an alias API.

```PUT|POST <index_name>/_alias/<alias_name>```

```shell
PUT cars_for_aliases/_alias/my_new_cars_alias
```

Create a single alias pointing to multiple indices:

```shell
PUT cars1,cars2,cars3/_alias/multi_cars_alias
```

```shell
PUT cars*/_alias/wildcard_cars_alias #A All indices prefixed with cars
```

GET ``cars`` returns the index with all aliases created on that index.

Fetch the alias details:

```shell
GET my_cars_alias/_alias #A Fetch alias details for single index 
```

```shell
GET all_cars_alias,my_cars_alias/_alias #A Fetch aliases associated with multiple indexes
```

#### MIGRATING DATA WITH ZERO DOWNTIME USING ALIASES

1. Create an alias called vintage_cars_alias to refer to the current index vintage_cars.
2. Because the new properties are incompatible with the existing index, create a new index, say vintage_cars_new with the new settings.
3. Copy (i.e., reindex) the data from the old index (vintage_cars) to the new index (vintage_cars_new).
4. Recreate your existing alias (vintage_cars_alias), which pointed to the old index, to refer to the new index. Thus, vintage_cars_alias will now be pointed to vintage_cars_new.
5. Now all the queries that were executed against the vintage_cars_alias are carried out on the new index.

#### MULTIPLE ALIASING OPERATIONS USING THE _ALIASES API

In addition to working with aliases using the ``_alias`` API, there is another API for working on multiple aliasing actions: the ``_aliases`` API. It combines a few actions such as adding and removing aliases, as well as deleting the indices, all in one go. Whereas the ``_alias`` API is used for creating an alias, the ``_aliases`` API helps create multiple actions on indices related to aliasing.

```shell
POST _aliases #A Uses the _aliases API to execute multiple actions
{
  "actions": [#B Lists the individual actions
    {
      "remove": {#C Removes the alias that points to an old index
        "index": "vintage_cars",
        "alias": "vintage_cars_alias"
      }
    }, {
      "add": { #D Adds an alias that points to a new index
          "index": "vintage_cars_new",
          "alias": "vintage_cars_alias"
      }
   } 
  ]
}
```

Create an alias for multiple indices using the same `_aliases` API by using the indices parameter to set up the list of indices.

```shell
POST _aliases
{
  "actions": [
    {
      "add": {
        "indices": ["vintage_cars","power_cars","rare_cars","luxury_cars"],
        "alias": "high_end_cars_alais"
      }
    } 
  ]
}
```

## 6.3 Reading indices

### 6.3.1 Reading public indices

```shell
GET cars1,cars2,cars3  #A Retrieves details of three indices
```

```shell
GET ca* #A Returns all indices prefixed with ca
```

```shell
GET mov*,stu* #A Gets all indices prefixed with mov and stu
```

```shell
GET cars/_settings #A Gets the settings from the _settings endpoint
GET cars/_mapping  #B Gets the mappings from the _mapping endpoint
GET cars/_alias    #C Gets the aliases from the _alias endpoint
```

### 6.3.2 Reading hidden indices

Hidden indices are usually reserved for system management.

We can create a hidden index by simply executing the command ``PUT .old_cars``.

The ``GET _all`` or ``GET *`` calls fetch all the indices including the hidden indices.

## 6.4 Deleting indices

```shell
DELETE cars
```

### DELETING MULTIPLE INDICES

```shell
DELETE cars,movies,order
```

### DELETING ONLY ALIASES

```shell
DELETE cars/_alias/cars_alias  #A Delete the cars_alias
```

## 6.5 Closing and opening indices

### 6.5.1 Closing indices

Closing the index means that any operations on it will cease. There will be no indexing of documents or search and analytic queries.

The close index API (``_close``) shuts down the index.

```shell
POST <index_name>/_close
```

#### CLOSING ALL OR MULTIPLE INDICES

We can close multiple indices by using comma-separated indices (including wildcards).

```shell
POST cars1,*mov*,students*/_close?wait_for_active_shards=index-setting
```

#### AVOID DESTABILIZING THE SYSTEM

As you can imagine, closing (or opening) all indices may destabilize the system. It’s one of those super admin capabilities that if executed without forethought, could lead to disastrous results, including taking the system down or making it irreparable. Hence, only administrators should turn this feature off (especially in production). This is done by setting a flag called `action.destructive_requires_name` to `true` in the `elasticsearch.yml` configuration file.

### 6.5.2 Opening indices

Opening an index kick starts the shards back into business; they are open for indexing and searching once ready.

```shell
POST cars/_open
```

## 6.6 Index templates

Copying the same settings across various indices, especially one by one, is tedious and at times erroneous too. If we define an indexing template with a schema upfront, creating a new index will implicitly be molded from this schema if the index name matches the template.

One use case for an indexing template might be to create a set of patterns based on environments. For example, indices for a development environment would have 3 shards and 2 replicas, while indices for a production environment would have 10 shards with 5 replicas each.

Additionally, we can create a template based on a glob (global command) pattern, such as wildcards, prefixes, and others.

### 6.6.1 Creating index templates

Since Elasticsearch version 7.8, the index templates can be classified into two categories: 
* composable index templates (or simply index templates) - are composed of zero or more component templates
* component templates - a template on its own, it's not very useful if it’s not associated with an index template. They can, however, be associated with many index templates

For example: 

Index Template A composed of Component Template 1, Component Template 2, Component Template 3.
Index Template B composed of no component templates.
Index Template C composed of Component Template 1, Component Template 2.
Index Template D composed of Component Template 2, Component Template 3, Component Template 4.

Rules when creating templates:
* An index created with configurations explicitly takes precedence over the configurations defined in the template or in component templates.
* Legacy templates carry a lower priority than the composable templates.

#### CREATING COMPOSABLE (INDEX) TEMPLATES

To create an index template, we use an ``_index_template`` endpoint, providing all the required mappings, settings, and aliases as an index pattern in this template.

Let’s say our requirement is to create a template for cars, represented with a pattern having wildcards: \*cars\*. This template will have certain properties and settings, such `created_by` and `created_at` properties, as well as shards and replica numbers. Any index that gets matched with this template during its creation inherits the configurations defined in the template.

```shell
POST _index_template/cars_template
{
  "index_patterns": ["*cars*"],
  "priority": 1,
  "template": {
    "mappings": {
      "properties":{
        "created_at":{
          "type":"date"
        },
        "created_by":{
          "type":"text"
        }
      } 
    }
  } 
}
```

#### CREATING COMPONENT TEMPLATES

A component template is nothing but a reusable block of configurations that we can use to make up more index templates. The component templates are of no value unless they are clubbed with index templates.

They are exposed via a ``_component_template`` endpoint.

```shell
POST _component_template/dev_settings_component_template
{
  "template":{
    "settings":{
      "number_of_shards":3,
      "number_of_replicas":3
    }
  } 
}
```

We use the ``_component_template`` endpoint to create a template. The body of the request holds the template information in a ``template`` object. Once executed, the ``dev_settings_component_template`` becomes available for use elsewhere in the index templates.

```shell
POST _component_template/dev_mapping_component_template
{
  "template": {
    "mappings": {
      "properties": {
        "created_by": {
          "type": "text"
        },
        "version": {
          "type": "float"
        }
      } 
    }
  } 
}
```

The ``dev_mapping_component_template`` consists of a mapping schema predefined with two properties, ``created_by`` and ``version``.

```shell
POST _index_template/composed_cars_template
{
    "index_patterns": ["*cars*"],
    "priority": 200,
    "composed_of": ["dev_settings_component_template", "dev_mapping_component_template"]
}
```

The highlighted composed_of tag is a collection of all component templates.

Once this script executes and we create an index with cars in the index name (``vintage_cars``, ``my_cars_old``, ``cars_sold_in_feb``, etc), the index gets created with the configuration derived from both component templates.

## 6.7 Monitoring and managing indices

### 6.7.1 Index statistics

The ``_stats`` API helps us retrieve statistics of an index, both for primary shards and for replica shards.

``GET cars/_stats``

```shell
{
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_all": {
    "primaries": {},
    "total": {}
  },
  "indices": {
    "cars": {
      "uuid": "1fajlfaflkj"
      "primaries": {},
      "total": {}
    }
  }
}
```

#### UNDERSTANDING THE RESPONSE

The response consists of two blocks primarily:
* The ``_all`` block, where we see the aggregated statistics of all indices combined
* The ``indices`` block, where we see the statistics for the individual indices

Both these blocks consist of two buckets of statistics: the ``primaries`` bucket contains the statistics related to just the primary shards, while the ``total`` bucket indicates the statistics for both primary and replica shards.

### 6.7.2 Multiple indices and statistics

```shell
GET cars1,cars2,cars3/_stats
```

```shell
GET cars*/_stats
```

```shell
GET */_stats # Fetch statistics for all indices
```

```shell
GET cars/_stats/segments # Only segments stats from statistics endpoint
```

## 6.8 Advanced operations

### 6.8.1 Splitting an index

Sometimes the indices may be overloaded with data so additional shards need to be added to the index to manage memory and distribute them evenly.

For example, if an index (cars) with 5 primary shards is overloaded, we can split the index into a new index with more primary shards, say 15 shards. This operation of expanding indices from a small size to a larger size is called **splitting**. Splitting is nothing more than creating a new index with more shards and copying the data from the old index into the new index.

Elasticsearch provides a ``_split`` API for splitting an index.

**Before we invoke the split operation on the all_cars index, we must make sure the index is disabled for indexing business (the index is changed to a read-only index)**.

```shell
PUT all_cars/_settings #A Uses the settings API
{
  "settings":{
    "index.blocks.write":"true" #B Closes the index for writing operations
  }
}
```

Now that the prerequisite of making the index non operational is complete, we can move to the next step of splitting it by invoking the split API.

```shell
POST <source_index>/_split/<target_index>
```

```shell
POST all_cars/_split/all_cars_new #A
{
  "settings": {
    "index.number_of_shards": 12 #B
  }
}
```

Rules and conditions:
* The target index must not exist before this operation.
* The number of shards in the target index must be a multiple of the number of shards in the source index. If source has 3 shards then multiples of 3 are valid (6, 9, 12 ...).
* The target index's primary shards can never be less than source.
* The target index's node must have adequate space.

### 6.8.2 Shrinking an index

Elasticsearch exposes ``_shrink`` API for this purpose.

Similar to what we did with a splitting operation, the first step is to make sure our all_cars index is read-only, so we’ll set the ``index.blocks.write`` property to ``true``.

```shell
PUT all_cars/_settings
{
  "settings": {
    "index.blocks.write": true,
    "index.routing.allocation.require._name": "node1"
  }
}
```

```shell
PUT all_cars/_shrink/all_cars_new2 #A
{
  "settings":{
    "index.blocks.write":null,#B
    "index.routing.allocation.require._name":null,#C
    "index.number_of_shards":1,#D
    "index.number_of_replicas":5
  } 
}
```

The source index was set with two properties: read-only and the allocation index node name. These settings will be carried over to the new target index if we do not reset them. Hence, in the script, we nullify these properties so the target index wouldn’t have these restrictions imposed on when it’s created.

There are also a bunch of actions that must be done prior to shrinking indices. The following list provides these actions:
* The source index must be switched off (made read-only) for indexing. Although not mandatory, but advised, we can turn off the replicas too before shrinking kicks in.
* The target index mustn’t be created or exists before the shrinking activity.
* All target index shards must reside on the same shard. There is a property called `index.routing.allocation.require.<node_name>` on the index that we must set with the node name to achieve this.
* The target index’s shard number can only be a factor of the source index’s shard number. Our all_cars index with 50 shards can only be shrunk to 25, 10, 5, or 1 shard.
* The target index’s node satisfies the memory requirements.

### 6.8.3 Rolling over an index alias

Unlike a splitting operation, in a rollover, the documents are not copied to the new index. The old index becomes read-only, and any new documents will be indexed into this rolled over index going forward.

The rollover operation is heavily used when dealing with time-series data. The time-series data (data that’s usually generated for a specific period like every day, weekly, or monthly) is usually held in an index created for a particular time period.

Elasticsearch performs a few steps to rollover the cars_2021-000001 index.
1. Elasticsearch creates an alias pointing to the index. Before Elasticsearch creates this alias, we must make sure the index is writable by setting `is_write_index` to true. The idea behind this step is that the alias must have at least one writable backing index.
2. Elasticsearch invokes a rollover command on the alias using the `_roller` API. This creates a new rollover index (for example, cars_2021-000002).

#### CREATING AN ALIAS FOR ROLLOVER OPERATIONS

```shell
POST _aliases #A
{
  "actions": [ #B
    {
        "add": { #C
          "index": "cars_2021-000001", #D 
          "alias": "latest_cars_a",
          "is_write_index": true
        } 
    }
  ] 
}
```

If the alias points to multiple indices, at least one must be a writable index.

#### ISSUING A ROLLOVER OPERATION

Now that we have an alias created, the next step is to invoke the rollover API endpoint.

```shell
POST latest_cars_a/_rollover
```

As you can clearly see, the `_rollover` endpoint is invoked on the alias not the index.

```shell
{
    "acknowledged" : true,
    "shards_acknowledged" : true,
    "old_index" : "latest_cars-000001", #A Old index name 
    "new_index" : "latest_cars-000002", #B New index name "rolled_over" : true,
    "dry_run" : false,
    "conditions" : { }
}
```

Naming convetions for rolling api.

```shell
POST <index_alias>/_rollover
or
POST <index_alias>/_rollover/<target_index_name>
```

Specifying a target index name as given in the second option lets the rollover API create the index with the given parameter as the target index name. However, the first option, where we don’t provide an index name, has a special convention: <index_name>-00000N. The number (after the dash) is **always made up of 6 digits with padded zeros**.

* Can we automatically rollover the index when the shard’s size has crossed a certain threshold?
* Can we instantiate a new index for everyday logs?

Although we have seen the mechanism of rollover in this section, we can satisfy these questions using the relatively new index life-cycle management (ILM) feature.

As the name suggests, the ILM is all about managing the indices based on a life-cycle policy. The policy is a definition that declares some rules that are executed by the engine when the conditions of the rules are met. For example, we can define rules based on rolling over the current index to a new index when:
* The index reaches a certain size (say 40 GB, for example)
* The number of documents in the index crossed, say, 10,000
* The day is rolled over

### 6.9.1 About the index life cycle

There are five stages in index life cycle. 

* Hot — The index is in full operation mode. It is available for both read and write, thus enabling the index for both indexing and querying.
* Warm — The index is in read-only mode. Indexing is switched off but open for querying so that the search and aggregation queries can still be served by this index.
* Cold — The index is in read-only mode. Similar to the warm phase, where indexing is switched off, and it's open for querying, albeit the queries are expected to be infrequent. When the index is in this phase, the search queries might result in slow response times.
* Frozen — This is similar to the cold phase, where the index is switched off for indexing but querying is allowed. However, the queries are more infrequent or even rare. When the index is in this phase, users may seem to notice longer response times for their queries.
* Delete — This is the index’s final stage, where the index gets deleted permanently. As such, the data is erased and resources are freed. Usually, it’s expected that we take a snapshot of the index before deleting so that the data from the snapshot can be restored at some point in the future.

Transitioning from the hot phase into every other phase is optional. That is, once created in the hot phase, the index can remain in that phase or transition to any of the other four phases.

### 6.9.2 Managing life cycle manually

Elasticsearch exposes an API for working with the index life-cycle policy, and the format goes like this: ``_ilm/policy/<index_policy_name>``. The process is split into two steps:
* defining a life cycle policy.
* associating policy with an index for execution.

```shell
PUT _ilm/policy/hot_delete_policy #A
{
  "policy": { #B
    "phases": {
      "hot": { #C
        "min_age": "1d", #D
        "actions": {    #E
          "set_priority": {  #F
            "priority": 250
          }
        } 
      },
      "delete": {  #G
        "actions": {
          "delete" : { }
        }
      } 
    }
  } 
}
```

Here’s what the definition states:
* Hot phase — The index is expected to live for at least one day before carrying out the actions. The actions block defined in the “actions” object sets a priority (250 in this example). The indices with a higher priority are acted on first during node recovery.
* Delete phase — The index is deleted once the hot phase completes all the actions. As there is no min_age on the delete phase, the delete action kicks in immediately once the hot phase finishes.

#### STEP 2: ASSOCIATING THE POLICY WITH AN INDEX

```shell
PUT hot_delete_policy_index
{
  "settings": { #A We use settings object to set the property
    "index.lifecycle.name":"hot_delete_policy" # Name of the policy
  }
}
```

### 6.9.3 Life cycle with rollover

In this section, we’ll set conditions on a time-series index to magically roll when those conditions are met. Let’s say we want to the index to be rolled over based on the following conditions:
* On each new day.
* When the maximum number of documents hits 10,000.
* When the maximum index size reaches 10 GB.

```shell
PUT _ilm/policy/hot_simple_policy
{
"policy": {
    "phases": {
      "hot": {   #A declares a hot phase
        "min_age": "0ms", #B index enters this phase instantly
        "actions": {
          "rollover": {   #C index rolls over if any of the conditions are met
            "max_age": "1d",
            "max_docs": 10000,
            "max_size": "10gb"
          } 
        }
      } 
    }
  } 
}
```

In our policy, we declared one phase, the hot phase, with rollover as the action to be performed when any of the conditions declared in the rollover actions are met. For example, if the maximum number of documents is 10,000 or the size of the index exceeds 10 GB or if the index is one day old, the index rolls over. As we declared the minimum age (min_age) to be 0ms, as soon as the index is created, it gets moved into the hot phase instantly and then rolled over.

```shell
PUT _index_template/mysql_logs_template
{
  "index_patterns": ["mysql-*"], #A The index pattern for all mysql indices
  "template":{
    "settings":{
      "index.lifecycle.name":"hot_simple_policy", #B Attaching the policy
      "index.lifecycle.rollover_alias":"mysql-logs-alias" #C Attaching an alias
    }
  } 
}
```

The final step is to create an index matching the index pattern defined in the index template, with a number as a suffix so the rollover indices are generated correctly.

```shell
PUT mysql-index-000001   #A
{
  "aliases": {
    "mysql-logs-alias": {
      "is_write_index": true
    } 
  }
}
```

By default, policies are scanned every 10 minutes. You need to update the cluster settings using the `_cluster` endpoint if you want to alter this scan period.

```shell
PUT _ilm/policy/hot_warm_delete_policy
{
"policy": {
    "phases": {
      "hot": {
        "min_age": "1d", #A The index waits for one day before becoming hot
        "actions": {
          "rollover": {  #B Rolls over when one of the conditions are met
            "max_size": "40gb",
            "max_age": "6d"
          },
          "set_priority": { #C Sets the priority
            "priority": 50
          }
        } 
      },
      "warm": {
        "min_age": "7d", #D Waits for 7 days before carrying out the actions
        "actions": {
          "shrink": {    #E Shrinks the index
            "number_of_shards": 1
          }
        } 
      },
      "delete": {      #F Deletes the index
        "min_age": "30d", #G Before deleting, stays in this phase for 30 days
        "actions": {
          "delete": {}
        }
      } 
    }
  } 
}
```

* Hot phase — The index enters into this phase after one day because the min_age attribute was set to 1d. After one day, the index moves into the roll-over stage and waits for the conditions to be satisfied: the maximum size is 40 GB ("max_size": "40gb") or the age is older than 6 days ("max_age": "6d"). Once any of these conditions are met, the index transitions from the hot phase to the warm phase.
* Warm phase — When the index enters the warm phase, it stays in the warm phase for about one week ("min_age": "7d") before any of its actions are implemented. After the seventh day, the index gets shrunk to one node ("number_of_shards": 1), then enters the delete phase.
* Delete phase — The index stays in this phase for 30 days ("min_age": "30d"). Once this time lapses, the index is deleted permanently.

# Chapter 7

In a nutshell, Elasticsearch cleans the text fields, breaks the text data into individual tokens, and enriches the tokens before storing them in the inverted indices.

The groundwork is carried out in the name of text analysis by employing so-called analyzers.

There are a handful of analyzers out of the box. For example, the standard (default) analyzer lets us work easily on English text by tokenizing the words using whitespace and punctuation as well as lowercasing the final tokens. If we want to stop indexing a set of predefined values (either common words like “a”, “an”, “the”, “and”, “if”, and so on or, perhaps, swear words), we can customize the analyzer to do so.

While this analysis process is highly customizable, the out-of-the-box analyzers fit the bill for most circumstances.

## 7.1 Overview

### 7.1.1 Querying unstructured data

Unstructured data is nothing but our day-to-day human language. It consists of free text (like a movie synopsis), a research article, an email body, blog post, and so on.

To build an intelligent and never disappointing search engine, extra assistance is given to the engine during the data indexing process in the name of text analysis and carried out by software modules called **analyzers**.

## 7.2 Analyzer module

The analyzer is a software module essentially tasked with two functions: tokenization and normalization.

### 7.2.1 Tokenization

Tokenization is a process of splitting sentences into individual words. The words can also be split based on nonletters, colons, or some other custom separators.

### 7.2.2 Normalization

Normalization is where the tokens (words) are massaged, transformed, modified, and enriched in the form of stemming, synonyms, stop words, and other features.

Stemming is an operation where the words are reduced (stemmed) to their root words (for example, “author” is a root word for “authors”, “authoring”, and “authored”).

In addition to stemming, normalization also deals with finding appropriate synonyms before adding them to the inverted index. For example, “author” may have additional synonyms such as “wordsmith”, “novelist”, “writer”, and so on.

### 7.2.3 Anatomy of an analyzer

All text fields go through this pipe: the raw text is cleaned by the character filters, and the resulting text is passed on to the tokenizer. The tokenizer then splits the text into tokens (aka individual words). The tokens then pass through the token filters where they get modified, enriched, and enhanced. Finally, the finalized tokens are then stored in the appropriate inverted indices. The search query gets analyzed too, in the same manner as the indexing of the text.

The analyzer is composed of three low-level building blocks. These are:
* Character filters — Applied on the character level, where every character of the text goes through these filters. The filter’s job is to remove unwanted characters from the text string. This process could, for example, purge HTML tags like `<h1>`, `<href>`, `<src>` from the input text;
* Tokenizers — Split the sentences into words by using a delimiter such as whitespace, punctuation, or some form of word boundaries. Every analyzer must have one and only one tokenizer. Elasticsearch provides a handful of these tokenizers to help split the incoming text into individual tokens;
* Token filters — Work on tokens produced by the tokenizers for further processing. For example, the token can change the case, create synonyms, provide the root word (stemming), or produce n-grams and shingles, and so on.

### 7.2.4 Testing analyzers

Elasticsearch exposes an endpoint just for testing the text analysis process. It provides an ``_analyze`` endpoint that helps us understand the process in detail.

```shell
GET _analyze
{
  "text": "James Bond 007"
}
```

#### EXPLICIT ANALYZER TESTS

We can explicitly enable an analyzer too.

```shell
GET _analyze
{
  "text": "James Bond 007",
  "analyzer": "simple"
}
```

The simple analyzer (we will learn about various types of analyzers in the next section) truncates text when a nonletter character is encountered, so this code produces only two tokens, “james” and “bond” (“007” was truncated), as opposed to three tokens from the earlier script that used the standard analyzer.

## 7.3 Built-in analyzers

Elasticsearch provides over half a dozen out-of-the-box analyzers that we can use in the text analysis phase. These analyzers most likely suffice for the basic cases, but should there be a need to create a custom one, one can do that by instantiating a new analyzer module with the required components that make up that module.

| Analyzer | Description                                                                                                                                                     |
|----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Standard analyzer | This is the default analyzer that tokenizes input text based on grammar, punctuation, and whitespace. The output tokens are lowercased. |
| Simple analyzer | A simple tokenizer splits input text on any nonletters such as whitespaces, dashes, numbers, etc. It lowercases the output tokens too. |
| Stop analyzer | It is a simple analyzer with English stop words enabled by default. |
| Whitespace analyzer | The whitespace analyzer’s job is to tokenize input text based on whitespace delimiters. |
| Keyword analyzer | The keyword analyzer doesn’t mutate the input text. The field’s value is stored as is. |
| Language analyzer | As the name suggests, the language analyzer helps work with human languages. Elasticsearch provides dozens of language analyzers such as English, Spanish. |
| Pattern analyzer | The pattern analyzer splits the tokens based on a regular expression (regex). |
| Fingerprint analyzer | The fingerprint analyzer sorts and removes the duplicate tokens to produce a single concatenated token. |

### 7.3.1 Standard analyzer

The standard analyzer is the default analyzer used in Elasticsearch.

#### CONFIGURING THE STANDARD ANALYZER

Elasticsearch allows us to configure a few parameters such as the stop words filter, stop words path, and maximum token length on the standard analyzer. The way to configure the properties is via the index settings. When we create an index, we can configure the analyzer through the settings component.

```shell
PUT <my_index>
{
  "settings": {
    "analysis": { #A The analysis object sets the analyzer
      "analyzer": { #B The analyzer that this index is associated with
        ... 
      }
    } 
  }
}
```

#### STOP WORDS CONFIGURATION

Let’s take an example of enabling English stop words on the standard analyzer.

```shell
PUT my_index_with_stopwords
{
  "settings": {
    "analysis": {
      "analyzer": {
        "standard_with_stopwords":{
          "type":"standard",
          "stopwords":"_english_"
        }
      } 
    }
  } 
}
```

```shell
POST my_index_with_stopwords/_analyze
{
  "text": ["Hot cup of cup and a popcorn is a Weird Combo :(!!"]
  "analyzer": "standard_with_stopwords"
}
```

The output of this call shows that the common (English) stop words such as “of”, “a”, and “is” were removed.

#### FILE-BASED STOPWORDS

We can provide the stop words via an explicit file.

```shell
PUT index_with_swear_stopwords
{
  "settings": {
    "analysis": {
      "analyzer": {
        "swearwords_analyzer":{
          "type":"standard",
          "stopwords_path":"swearwords.txt"
        }
      } 
    }
  } 
}
```

``stopwords_path`` attribute looks for a file in a directory inside the Elasticsearch’s config folder.

```shell
file:swearwords.txt
damn
bugger
bloody hell
what the hell
sucks
```

#### CONFIGURING TOKEN LENGTH

We can also configure the maximum token length; in which case, the token is split based on the length asked for.

The analyzer is configured to have a maximum token length of 7 characters. If we provide a word that is 13 characters long, the word would be split into 7 and 6 characters (for example, Elasticsearch would become “Elastic”, “search”).

### 7.3.2 Simple analyzer

While the standard analyzer breaks down the text into tokens when encountered with whitespaces or punctuation, the simple analyzer tokenizes the sentences at the occurrence of a nonletter like a number, space, apostrophe, or hyphen.

Let’s consider an example of indexing the text "Lukša's K8s in Action".

This results is: ``["lukša","s","k","s","in","action"]``

### 7.3.3 Whitespace analyzer

As the name suggests, the whitespace analyzer splits the text into tokens when it encounters whitespaces.

### 7.3.4 Keyword analyzer

As the name suggests, the keyword analyzer stores the text as is without any modifications and tokenization. That is, the analyzer does not tokenize the text, nor does it undergo any further analysis via filters or tokenizers. Instead, it is stored as a string representing a keyword type.

If we pass “Elasticsearch in Action” through the keyword analyzer, the whole text string is stored as is.

### 7.3.5 Fingerprint analyzer

The fingerprint analyzer removes duplicate words, extended characters, and sorts the words alphabetically to create a single token. It consists of a standard tokenizer along with four token filters: fingerprint, lowercase, stop words, and ASCII folding filters.

```shell
POST _analyze
{
  "text": "A dosa is a thin pancake or crepe originating from South India. It is made from a fermented batter consisting of lentils and rice.",
  "analyzer": "fingerprint"
}
```

```shell
"tokens" : [{
"token" : "a and batter consisting crepe dosa fermented from india is it lentils made or or originating pancake rice south thin",
  "start_offset" : 0,
  "end_offset" : 130,
  "type" : "fingerprint",
  "position" : 0
  }]
```

When you look closely at the response, you will find that the output is made up of only one token. The words are lowercased and sorted, duplicate words (“a”, “of”, “from”) are removed as well before turning the set of words into a single token.

### 7.3.6 Pattern analyzer

Sometimes, we may want to tokenize and analyze text based on a certain pattern (for example, removing the first n number of a phone numbers or removing a dash for every four digits from a card number and so on). Elasticsearch provides a pattern analyzer just for that purpose.

The default pattern analyzer works on splitting the sentences into tokens based on nonword characters. This pattern is represented as \W+ internally.

As the default (standard) analyzer only works on nonletter delimiters, for any other patterns we need to configure the analyzer by providing the required patterns.

Let’s just say we have an e-commerce payments authorizing application and are actually receiving payment authorization requests from various parties. A 16-digit long card number is provided in the format 1234-5678-9000-0000. We want to tokenize this card data on a dash (-) and extract the four tokens individually. We can do so by creating a pattern that splits the field into tokens based on the dash delimiter.

```shell
PUT index_with_dash_pattern_analyzer #A Creates an index with analyzer settings
{
  "settings": {
    "analysis": {
      "analyzer": {
        "pattern_analyzer": { #B In settings, defines the analyzer in the analysis object 
          "type": "pattern", #C Provides the type of analyzer as pattern
          "pattern": "[-]", #D The regex representing the dash
          "lowercase": true #E Attaches a lowercase token filter
        } 
      }
    } 
  }
}
```

```shell
POST index_with_dash_pattern_analyzer/_analyze
{
  "text": "1234-5678-9000-0000",
  "analyzer": "pattern_analyzer"
}
```

The output of this command produces four tokens: ["1234","5678","9000","0000"].

### 7.3.7 Language analyzers

Elasticsearch provides a long list of language analyzers that are suitable when working with most languages.

```shell
POST _analyze
{
  "text": "She sells sea shells",
  "analyzer": "english"
}

# German Language Analyzer
POST _analyze
{
  "text": "Guten Morgen",
  "analyzer": "german"
}
```

We can configure the language analyzers with a few additional parameters to provide our own list of stop words or to ask the analyzers to exclude the stemming operation.

We can configure our stop words as the following listing shows.

```shell
PUT index_with_custom_english_analyzer #A Creates an index with analyzer settings
{
  "settings": {
    "analysis": {
      "analyzer": {
        "index_with_custom_english_analyzer":{ #B Provides a custom name
          "type":"english", #C The type of the analyzer here is english
          "stopwords":["a","an","is","and","for"] #D Provides our own set of stop words
        }
      } 
    }
  } 
}
```

As the code indicates, we created an index with a custom English analyzer and a set of user-defined stop words.

We can bring the ``stem_exclusion`` attribute to configure the words that need to be excluded from the stemming.

```shell
PUT index_with_stem_exclusion_english_analyzer
{
  "settings": {
    "analysis": {
      "analyzer": {
        "stem_exclusion_english_analyzer":{
          "type":"english",
          "stem_exclusion":["authority","authorization"]
        }
      } 
    }
  } 
}
```

```shell
POST index_with_stem_exclusion_english_analyzer/_analyze
{
  "text": "No one can challenge my authority without my authorization",
  "analyzer": "stem_exclusion_english_analyzer"
}
```

The tokens that were spit as a consequence of the code in listing consist of our two words: “authority” and “authorization”. Otherwise it would return "author".

## 7.4 Custom analyzers

Elasticsearch provides much flexibility when it comes to analyzers: if off-the-shelf analyzers won't cut it for you, you can create your own custom analyzers. These custom analyzers can be a mix-and-match of existing components from a large stash of Elasticsearch’s component library.

```shell
PUT index_with_custom_analyzer # Create an index definition with settings for text analysis functionality

{
  "settings": {
    "analysis": {
      "analyzer": {
        "custom_analyzer": {
          "type": "custom", # Define the type as custom
          "char_filter": ["charfilter1", "charfilter2"], # Declare an array of character filters
          "tokenizer": "standard", # Attach a mandatory tokenizer
          "filter": ["tokenfilter1", "tokenfilter2"] # Declare an array of token filters
        }
      }
    }
  }
}
```

We define a custom analyzer on an index by setting the type to `custom`. Our custom analyzer is developed with an array of character filters represented by the `char_filter` object and another array of token filters represented by the `filter` attribute.

### 7.4.1 Advanced customization

Let’s suppose our requirement is to develop a custom analyzer that parses text for Greek letters and produces a list of Greek letters as a result.

```shell
PUT index_with_parse_greek_letters_custom_analyzer
{
  "settings": {
    "analysis": {
      "analyzer": {
        "greek_letter_custom_analyzer":{ #A Creates a custom Greek-letter parser analyzer
          "type":"custom",
          "char_filter":["greek_symbol_mapper"], #B The custom analyzer is made of a custom char_filter (see #E)
          "tokenizer":"standard", #C A bog standard tokenizer tokenizes the text
          "filter":["lowercase", "greek_keep_words"] #D Supplies two token filters. The greek_keep_words is defined in #G.=
        } 
      },
      "char_filter": { #E Defines the Greek letters and maps those to English words
        "greek_symbol_mapper":{
          "type":"mapping",
          "mappings":[ #F The actual mappings: a list of symbol to value
            "α => alpha",
            "β => Beta",
            "γ => Gamma" ]
        } 
      },
      "filter": {
        "greek_keep_words":{ #G We don’t want to index all the field values, only the words that match the keep words
          "type":"keep",
          "keep_words":["alpha", "beta", "gamma"] #H Keep words; all other words are discarded
        }
      } 
    }
  } 
}
```

```shell
POST index_with_parse_greek_letters_custom_analyzer/_analyze
{
  "text": "α and β are roots of a quadratic equation. γ isn't",
  "analyzer": "greek_letter_custom_analyzer"
}
```

## 7.5 Specifying analyzers

Analyzers can be specified at a few levels: index, field and query level.

### 7.5.1 Analyzers for indexing

At times we may have a requirement to set different fields with different analyzers - for example, a name field could have been associated with a simple analyzer while the credit card number field with a pattern analyzer. Fortunately, Elasticsearch let’s us set different analyzers on individual fields as required; Similarly, we can also set a default analyzer per index so that any fields that were not associated with a specific analyzer explicitly during the mapping process will inherit the index level analyzer.

#### FIELD LEVEL ANALYZER

```shell
PUT authors_with_field_level_analyzers
{
  "mappings": {
    "properties": {
      "name":{
        "type": "text" #A Standard analyzer is being used here
      },
      "about":{
        "type": "text",
        "analyzer": "english" #B Set explicitly with an english analyzer
      },
      "description":{
        "type": "text",
        "fields": {
          "my":{
            "type": "text",
            "analyzer": "fingerprint" #C Fingerprint analyzer on a multi-field
          }
        } 
      }
    } 
  }
}
```

As the code shows, the `about` and `description` fields were specified with different analyzers except the `name` field which is implicitly inheriting the `standard` analyzer.

#### INDEX LEVEL ANALYZER

```shell
PUT authors_with_default_analyzer
{
  "settings": {
    "analysis": {
      "analyzer": {
        "default":{ #A Setting this property sets index’s default analyzer
          "type":"keyword"
        }
      } 
    }
  } 
}
```

We are in effect replacing the `standard` analyzer which comes as default to a `keyword` analyzer.

### 7.5.2 Analyzers for searching

Elasticsearch lets us specify a different analyzer during query time than using the same one during indexing.

#### ANALYZER IN A QUERY

```shell
GET authors_index_for_search_analyzer/_search
{
"query": {
    "match": { #A Query to search all the authors with the given criteria
      "author_name": {
        "query": "M Konda",
        "analyzer": "simple" #B The analyzer is specified explicitly, most likely different to the one that field was indexed with
      }
    } 
  }
}
```

#### SETTING THE ANALYZER AT A FIELD LEVEL

The second mechanism to set the search specific analyzer is at the field level. Just as we set an analyzer on a field for indexing purposes, we can add an additional property called the `search_analzyer` on a field to specify the search analyzer.

```shell
PUT authors_index_with_both_analyzers_field_level
{
  "mappings": {
    "properties": {
      "author_name":{
        "type": "text",
        "analyzer": "stop",
        "search_analyzer": "simple"
      } 
    }
  } 
}
```

As the code above shows, the `author_name` is set with a stop analyzer for indexing while a `simple` analyzer for search time.

#### DEFAULT ANALYZER AT INDEX LEVEL

We can also set a default analyzer for search queries too just as we did for indexing time by setting the required analyzer on the index at index creation time.

```shell
PUT authors_index_with_default_analyzer
{
  "settings": {
    "analysis": {
      "analyzer": {
        "default_search":{ #A Setting the default search analyzer per index by using the default_search property
          "type":"simple"
        },
        "default":{ #B The default analyser for the index
          "type":"standard"
        }
      } 
    }
  } 
}
```

You may be wondering if we can set a search analyzer at a field level during the indexing rather than at runtime during the query?

```shell
PUT authors_index_with_both_analyzers_field_level
{
  "mappings": {
    "properties": {
      "author_name":{
        "type": "text",
        "analyzer": "standard",
        "search_analyzer": "simple"
      } 
    }
  } 
}
```

As you can see from the above code, the author_name is going to use a standard analyzer for indexing while a simple analyzer during search.

#### ORDER OF PRECEDENCE

* An analyzer defined at a query level has the highest precedence.
* An analyzer defined by setting search_analyzer property on a field when defining the index mappings.
* An analyzer defined at the index level.
* If neither of the above were not set, the Elasticsearch engine picks up the indexing analyzer set on a field or an index.

## 7.6 Character filters

When a user searches for answers, the expectation is that they won't search with punctuation or special characters. For example, there is a high chance a user may search for “cant find my keys” (without punctuation) rather than “can’t find my keys !!!”. Similarly, the user is not expected to search the string `<h1>Where is my cheese?</h1>` (with the HTML tags). We don’t even expect the user to search using XML tags like `<operation>callMe</operation>`.

Character filters help purge the unwanted characters from the input stream.

The character filter carries out the following specific functions:
* Removes the unwanted characters from an input stream;
* Adds to or replaces additional characters in the existing stream.

### 7.6.1 Types of character filters

There are three character filters that we use to construct an analyzer: 
* HTML strip.
* mapping.
* pattern filters.

#### HTML STRIP (HMTL_STRIP) FILTER

As the name suggests, this filter strips the unwanted HTML tags from the input fields.

We can configure the html_strip filter to add an additional escaped_tags array with the list of tags that needs to be unparsed.

```shell
PUT index_with_html_strip_filter
{
  "settings": {
    "analysis": {
      "analyzer": {
        "custom_html_strip_filter_analyzer":{
          "tokenizer":"keyword",
          "char_filter":["my_html_strip_filter"] #A Declares a custom character filter
        }
      },
      "char_filter": {
        "my_html_strip_filter":{
          "type":"html_strip",
          "escaped_tags":["h1"] #B The escaped_tags attribute ignores parsing h1 tags present in the input field.
        }
      } 
    }
  } 
}
```

#### MAPPING CHARACTER FILTER

The mapping character filter’s sole job is to match a key and replace it with a value, e.g. α as alpha, β as beta,and so on.

#### MAPPINGS VIA A FILE

We can also provide a file with mappings in it, rather than specifying them in the definition.

The file must be present in Elasticsearch’s config directory (`<INSTALL_DIR/elasticsearch/config`) or input with an absolute path where it is located.

```shell
POST _analyze
{
  "text": "FBI and CIA are USA's security organizations",
  "char_filter": [
    {
      "type": "mapping",
      "mappings_path": "secret_organizations.txt"
    }
  ] 
}
```

#### PATTERN REPLACE CHARACTER FILTER

The pattern_replace character filter, as the name suggests, replaces the characters with a new character when the field matches with a regular expression (regex).

```shell
PUT index_with_pattern_replace_filter
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_pattern_replace_analyzer":{
          "tokenizer":"keyword",
          "char_filter":["pattern_replace_filter"]
        }
      },
      "char_filter": {
        "pattern_replace_filter":{
          "type":"pattern_replace",
          "pattern":"_",
          "replacement":"-"
        } 
      }
    } 
  }
}
```

## 7.7 Tokenizers

The job of a tokenizer is to create tokens based on certain criteria.

### 7.7.1 Standard tokenizer

A standard tokenizer splits the words based on word boundaries and punctuation.

The standard analyzer has only one attribute that can be customized, the `max_token_length`. This attribute helps produce tokens of the size defined by the `max_token_length` property (default size is 255).

```shell
PUT index_with_custom_standard_tokenizer
{
  "settings": {
    "analysis": {
      "analyzer": {
        "custom_token_length_analyzer": { #A Creates a custom analyzer with a pointer to the custom tokenizer
          "tokenizer": "custom_token_length_tokenizer"
        }
      },
      "tokenizer": {
        "custom_token_length_tokenizer": {
          "type": "standard",
          "max_token_length": 2 #B The custom tokenizer with max_token_length set to 2 characters
        }
      } 
    }
  } 
}
```

```shell
POST index_with_custom_standard_tokenizer/_analyze
{
  "text": "Bond",
  "analyzer": "custom_token_length_analyzer"
}
```

This code spits out two tokens: “Bo" and “nd".

### 7.7.2 N-gram and edge_ngram tokenizers

The n-grams are a sequence of words for a given size prepared from a given word. Take as an example the word “coffee”. The two-letter n-grams, usually called bi-grams, are “co”, “of”, “ff”, “fe”, and “ee”. Similarly, the three-letter tri-grams are “cof”, “off”, “ffe”, and “fee”. As you can see from these two examples, the n-grams are prepared by sliding the letter window.

On the other hand, the edge_ngrams produce words with letters anchored at the beginning of the word. Considering “coffee” as our example, the edge_ngram produces “c”, “co”, “cof”, “coff”, “coffe”, and “coffee”. 

#### THE N-GRAM TOKENIZER

For correcting spellings and breaking words, we usually use n-grams. The n-gram tokenizer emits n-grams of a minimum size as 1 and a maximum size of 2 by default. For example, this code produces n-grams of the word “Bond”.

```shell
POST _analyze
{
  "text": "Bond",
  "tokenizer": "ngram"
}
```

The output is [B, Bo, o, on, n, nd, d]. You can see that each n-gram is made of one or two letters: this is the default behavior. We can customize the `min_gram` and `max_gram` sizes by specifying the configuration.

```shell
PUT index_with_ngram_tokenizer
{
  "settings": {
    "analysis": {
      "analyzer": {
        "ngram_analyzer":{
          "tokenizer":"ngram_tokenizer"
        }
      },
      "tokenizer": {
        "ngram_tokenizer":{
          "type":"ngram",
          "min_gram":2,
          "max_gram":3,
          "token_chars":["letter" ]
        } 
      }
    } 
  }
}
```

```shell
POST index_with_ngram_tokenizer/_analyze
{
  "text": "bond",
  "analyzer": "ngram_analyzer"
}
```

This produces these n-grams: “bo”, “bon”, “on”, “ond”, and “nd”.

#### THE EDGE_NGRAM TOKENIZER

### 7.7.3 Other tokenizers

|  Tokenizer   | Description                                                                                                                                                     |
|-----|-----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Pattern | The pattern tokenizer splits the field into tokens on a regex match. The default pattern is to split words when encountered by a nonword letter.                |
| URL and email | The uax_url_email tokenizer parses the fields and preserves URLs and emails. The URLs and emails in text will be spit out as they are without any tokenization. |
| Whitespace | The whitespace tokenizer splits the text into tokens when whitespace is encountered.                                                                            |
| Keyword | The keyword tokenizer doesn’t touch the tokens; it spits the text as is.                                                                                        |
| Lowercase | The lowercase tokenizer splits the text into tokens when a nonletter is encountered and lowercases the tokens.                                                  |
| Path hierarchy | The path_hieararchy tokenizer splits hierarchical text such as filesystem folders into tokens based on path separators. |

## 7.8 Token filters

The tokens produced by the tokenizers may need further enriching or enhancements such as lowercasing (or uppercasing) the tokens, providing synonyms, developing stemming words, removing the apostrophes or punctuation, and so on. Token filters work on the tokens to perform such transformations.

Elasticsearch provides almost 50 token filters.

We can test a token filter by simply attaching to a tokenizer and using it in the `_analyze` API:

```shell
GET _analyze
{
    "tokenizer" : "standard",
    "filter" : ["uppercase","reverse"], "text" : "bond"
}
```

The filter accepts an array of token filters; for example, we provided the uppercase and reverse filters in this example. The output would be “DNOB”.

You can also attach the filters to a custom analyzer as the following:

```shell
PUT index_with_token_filters
{
  "settings": {
    "analysis": {
      "analyzer": {
        "token_filter_analyzer": { #A Defines a custom analyzer
          "tokenizer": "standard",
          "filter": [ "uppercase","reverse"] #B Provides token filters as an array of filters
        }
      } 
    }
  } 
}
```

### 7.8.1 Stemmer filter

Stemming is a mechanism to reduce the words to their root words, for example, the word “bark” is the root word for “barking”.

```shell
POST _analyze
{
  "tokenizer": "standard",
  "filter": ["stemmer"],
  "text": "barking is my life"
}
```

### 7.8.2 Shingle filter

Shingles are the word n-grams that are generated at the token level (unlike the n-grams and edge_ngrams that emit n-grams at a letter level). For example, the text, “james bond” emits as “james”, and “james bond”.

```shell
POST _analyze
{
  "tokenizer": "standard",
  "filter": ["shingle"],
  "text": "java python go"
}
```

The result of this code execution is `[java, java python, python, python go, go]`.

The default behavior of the filter is to emit unigrams and two-word n-grams. We can change this default behavior by creating a custom analyzer with a custom shingle filter:

```shell
PUT index_with_shingle
{
  "settings": {
    "analysis": {
      "analyzer": {
        "shingles_analyzer":{
          "tokenizer":"standard",
          "filter":["shingles_filter"] #A Creates a custom analyzer attached with a shingles filter
        }
      },
      "filter": {
        "shingles_filter":{ #B Provides the attributes of the shingle filter (for example, min and max shingle)
          "type":"shingle",
          "min_shingle_size":2,
          "max_shingle_size":3,
          "uutput_unigrams":false #C Turns off the output of single words
        }
      }
    } 
  }
}
```

```shell
POST index_with_shingle/_analyze
{
  "text": "java python go",
  "analyzer": "shingles_analyzer"
}
```

The analyzer returns `[java python, java python go, python go]` because we’ve configured the filter to produce only 2- and 3-word shingles. The unigram (one word shingle) are turned off.

### 7.8.3 Synonym filter

Elasticsearch expects us to provide a set of words and their synonyms by configuring the analyzer with a synonym token filter. We create the synonyms filter on an index’s settings:

```shell
PUT index_with_synonyms
{
  "settings": {
    "analysis": {
      "filter": {
        "synonyms_filter":{
          "type":"synonym",
          "synonyms":[ "soccer => football"]
        }
      } 
    }
  } 
}
```

```shell
POST index_with_synonyms/_analyze
{
  "text": "What's soccer?",
  "tokenizer": "standard",
  "filter": ["synonyms_filter"]
}
```

This produces two tokens: “What’s”, and “football”. 

#### SYNONYMS FROM A FILE

```shell
PUT index_with_synonyms_from_file_analyzer
{
  "settings": {
    "analysis": {
      "analyzer": {
        "synonyms_analyzer":{
          "type":"standard",
          "filter":["synonyms_from_file_filter"]
        }
      }
      ,"filter": {
        "synonyms_from_file_filter":{
          "type":"synonym",
          "synonyms_path":"synonyms.txt" #A Relative path of the synonyms file
        }
      } 
    }
  } 
}
```

We can call the file from a relative or absolute path. The relative path points to the config directory of Elasticsearch’s installation folder.

# Chapter 8. Introducing search

## 8.1 Overview

There are two variants of search in the world of Elasticsearch: **structured search** and **unstructured search**.

**Structured search**, supported by a term-level search functionality, returns results with no relevance scoring associated. Elasticsearch fetches the documents if they match exactly and doesn't bother with if they are a close match or how well they match.

On the other hand, in an **unstructured search**, Elasticsearch retrieves results that are closely related to the query. The results are scored based on how closely they are relevant to the criteria: the highly relevant results get higher scoring and, hence, are positioned on the top of the result hits. Searching on text fields yields relevant results.

Not every result that a response has is accurate. This is due to the fact Elastiseach employs two strategies, called **precision** and **recall**, that affect the relevancy of the returned results.

## 8.2 How does search work?

When a search request is received from a user or a client, the engine forwards that request to one of the available nodes in the cluster. Every node in the cluster is, by default, assigned to a coordinator role; hence, making every node eligible for picking up the client requests on a round-robin basis. Once the request reaches the coordinator node, it then determines the nodes on which the shards of the concerned documents exist.

## 8.3 Search fundamentals

### 8.3.1 Search endpoint

Elasticsearch provides RESTful APIs to query the data, a `search` endpoint to be specific.

There are two ways of accessing the search endpoint:
* URI request — With this method, we pass the search query along with the endpoint as parameters to the query. For example, `GET movies/_search?q=title:Godfather`
* Query DSL — With this method, Elasticsearch implements a domain-specific language (DSL) for the search. The criteria is passed as a JSON object in the payload.

```shell
GET movies/_search
{
"query": {
  "match": {
      "title": "Godfather"
    }
  } 
}
```

### 8.3.2 Query vs filter context

This execution context can be either a **filter context** or **query context**. All queries issued to Elasticsearch are carried out in one of these contexts. Although we have no say in asking Elasticsearch to apply a certain type of context, it is our query that lets Elasticsearch decide on applying the appropriate context.

#### QUERY CONTEXT

We have used a match query to search for a set of documents that match the keywords with the field’s values.

```shell
GET movies/_search
{
"query": {
  "match": {
      "title": "Godfather"
    }
  } 
}
```

```json
"hits": [{
 ...
 "_score" : 2.879596
 "_source" : {
   "title" : "The Godfather"
... }
}, {
...
   "_score" : 2.261362
   "_source" : {
     "title" : "The Godfather: Part II"
... }
}]
```

The code output indicates that the query was executed in a query context because the query searched not only if it matched a document, but also how well the document was matched.

#### FILTER CONTEXT

Let’s rewrite the query by wrapping our match query in a bool query with a filter clause.

```shell
GET movies/_search
{
  "query": {
    "bool": { 
      "filter": [{
        "match": {"title": "Godfather"}
      }]
    } 
  }
}
```
The results do not have a score (score is set to 0.0) because Elasticsearch got a clue from our query that it must be run in a filter context.

The main benefit of running a query in this context is that, because there is no need to calculate scores for the returned search results, Elasticsearch can save on some computing cycles. Because these filter queries are more idempotent, Elasticsearch tries to cache these queries for better performance.

#### COMPOUND QUERIES

The `bool` query is a compound query with a few clauses (`must`, `must_not`, `should`, and `filter`) to wrap leaf queries. In addition to the filter clause, the `must_not` clause gets executed in a filter context. The `constant_score` query is another compound query where we can attach a query in a filter clause.

```shell
GET movies/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "match": {
          "title": "Godfather"
        }
      } 
    }
  }
}
```

## 8.4 Movie sample data

We will create some movie test data along with movie mappings for this chapter.

```shell
PUT movies #A Movies index
{
  "mappings": { #B Mappings schema
    "properties": { #C Fields and their types
      "title": {
        "type": "text",
        "fields": { #D Multi-field construct
          "original": {
            "type": "keyword"
          }
        } 
      },
      "synopsis": {
        "type": "text"
      },
      "actors": {
        "type": "text"
      },
      "director": {
        "type": "text"
      },
      "rating": {
        "type": "half_float"
      },
      "release_date": {
        "type": "date",
        "format": "dd-MM-yyyy"
      },
      "certificate": {
        "type": "keyword"
      },
      "genre": {
        "type": "text"
      } 
    }
  } 
}
```

You can visit my GitHub repository to fetch the [full script](https://github.com/madhusudhankonda/elasticsearch-in-action/blob/main/kibana_scripts/ch8_search_basics.txt).

## 8.5 Anatomy of a request and response

### 8.5.1 Search request

```shell
GET movies/_search # The search scope and the search endpoint which is used for both search and aggregation
{
  "query": { # query component where a user is expected to specify a search type and criteria
    "match": { # The search query type
      "title": "Godfather" # The search query's criteria
    }
  }
}
```

There’s a school of thought that the GET method in a RESTful architecture shouldn’t send any parameters using the body of the request. Instead, one should use a POST method if we are querying the server. **Elasticsearch implements the GET method request that accepts a body**, which helps formulate the query parameters. **You can replace GET with POST as both GET or POST act on the resource exactly the same way**.

We can also include multiple indices, multiple aliases, or even no index in this URL. To specify multiple indices (or aliases), enter comma-separated index (or alias) names. Providing no index or alias on the search request tells the search to run against all the indices across the cluster.

### 8.5.2 Search response

```json
{ 
  "took": 8,
  "timed_out": false,
  "_shards": {},
  "hits": {}
}
```

The `took` attribute, measured in milliseconds, indicates the time it takes for the search request to complete. This is the time measured from when a coordinator node receives the request to the time it manages to aggregate the response before sending it back to the client. It doesn’t include any of the client-to-server marshaling/unmarshalling times.

The `timed_out` attribute is a Boolean flag that indicates if the response has partial results, meaning if any of the shards failed to respond in time. For example, if we have three shards and one of them fails to return the results, the response will consist of the results from the two shards but indicates the failed shard’s state in the next object under the `_shards` attribute.

## 8.6 URI request search

### 8.6.1 Search movies by title

```GET movies/_search?q=title:Godfather```

The URL is composed of the `_search` endpoint followed by the query represented by the letter `q`. If we want to search for movies matching multiple titles:```GET movies/_search?q=title:Godfather Knight Shawshank```.

### 8.6.2 Search a specific movie

To fetch a specific movie, we can combine the criteria with the `AND` operator: `GET movies/_search?q=title:Knight AND actors:Bale`.

**Remember that Elasticsearch uses the OR operator by default.**

### 8.6.3 Additional parameters

```GET movies/_search?q=title:Godfather actors:(Brando OR Pacino) rating:(>=9.0 AND <=9.5)&from=0&size=10&explain=true&sort=rating&default_operator=AND```

## 8.7 Query DSL

### 8.7.1 Sample query

Let’s write a `multi_match` query that searches a keyword, Lord, across two fields, `synopsis` and `title`.

```shell
GET movies/_search
{
"query": {
    "multi_match": {
      "query": "Lord",
      "fields": ["synopsis","title"]
    }
  } 
}
```

### 8.7.3 Query DSL for aggregations

With Query DSL, we use a similar format for aggregations (analytics) with an `aggs` (short for aggregations) object instead of a `query` object.

```shell
GET movies/_search
{
  "size": 0,
  "aggs": {
    "average_movie_rating": {
      "avg": {
        "field": "rating"
      }
    } 
  }
}
```

This query fetches the average rating of all movies by utilizing a metric aggregation called `avg` (short for average).

### 8.7.4 Leaf and compound queries

We call the queries that are straightforward with no clauses a **leaf query**. These are the queries that fetch results based on a certain criteria (for example, getting the top-rated movies, movies that are released during a particular year, the gross earnings of a movie, and so on).

```shell
GET movies/_search
{
"query": {
    "match_phrase": {
      "synopsis": "A meek hobbit from the shire and eight companions"
    }
  } 
}
```

Leaf queries cannot fetch multiple query clauses. For example, they are not designed to search for movies that match a title but must NOT match a particular actor AND released during a specific year.

**Compound queries** allow us to create complex queries by combining leaf queries and even other compound queries using logical operators. A Boolean query, for example, is a popular compound query that supports writing queries with clauses like `must`, `must_not`, `should`, and `filter`.

```shell
GET movies/_search
{
"query": {
    "bool": {
      "must": [{"match": {"title": "Godfather"}}],
      "must_not": [{"range": {"rating": {"lt": 9.0}}}],
      "should": [{"match": {"actors": "Pacino"}}],
      "filter": [{"term": {"actors": "Brando"}}]
    } 
  }
}
```

## 8.8 Search features

### 8.8.1 Pagination

Elasticsearch, by default, sends the top-ten results, but we can change this number by setting the `size` parameter on the query, with a maximum set to 10,000.

```shell
GET movies/_search
{
"size": 20,
"query": {
    "match_all": {}
  }
}
```

While 10 K is a pretty good number for most searches, if your requirement is to get more than that number, you need to reset the `max_result_window` setting on the index.

```shell
GET movies/_search
{
  "size": 100, #A Fetches every page with 100 results
  "from": 3, #B Fetches from the third page, ignoring the first two pages
  "query": {
    "match_all": {}
  }
}
```

If the result set is too large (more than 10 K), rather than working with the pagination using the `size` and `from` attributes, we need to work with the `search_after` attribute.

### 8.8.2 Highlighting

When we search for a keyword(s) on a website in our internet browser using Ctrl-F, we can see the results highlighted so they stand out.

In a Query DSL, we can add a `highlight` object at the same level as the top-level `query` object. The `highlight` object expects a `fields` block, which can have multiple fields that you want to emphasize in the results.

When results are returned from the server, we can ask Elasticsearch to highlight the matches with its default settings by enclosing the matched text in emphasis tags (`<em>match</em>`).

```shell
GET movies/_search
{
  "_source": false, #A Suppresses the source to be returned
  "query": {
    "term": {
      "title": {
        "value": "godfather"
      }
    }
  },
  "highlight": { #B Includes a highlight object along with the fields on which highlights are expected
    "fields": {
      "title": {} #C The field on which we require the highlight
    }
  } 
}
```

### 8.8.3 Explanation

Elasticsearch provides a mechanism to understand the makeup of relevancy scores. This mechanism tells us exactly how the engine calculates the score. This is achieved by using an `explain` flag on a search endpoint or an `explain` API.

#### EXPLAIN FLAG

```shell
# Explanation
GET movies/_search
{
  "explain": true, # This is set to true
  "_source": false, 
  "query": {
    "match": {
      "title": "Lords"
    }
  }
}
```

This will return response which can be found in https://www.elastic.co/guide/en/elasticsearch/reference/current/search-explain.html.

#### EXPLAIN API

Although we use the `explain` attribute to understand the mechanics of relevancy scoring, there’s also an `explain` API that provides insight into why a document matched (or not), in addition to providing the scoring calculations.

```shell
GET movies/_explain/14
{
"query":{
  "match": {
      "title": "Lord"
    }
  } 
}
```

A search query built using the `explain` flag on the `_search` API can produce a lot of results. Asking for an explanation of the scores for all documents at a query level is simply a waste of computing resources in my opinion. Instead, pick one of the documents and ask for an explanation using the `_explain` API.

### 8.8.4 Sorting

The results returned by the engine are sorted by default on the relevancy score (`_score`).

#### SORTING THE RESULTS

```shell
GET movies/_search
{
  "query": {
    "match": {
      "genre": "crime"
    }
  }, "sort": [
     { "rating" :{ "order": "desc" } }
  ]
}
```

#### SORTING ON THE RELEVANCY SCORE

If you want to reverse the order with an ascending sort.

```shell
GET movies/_search
{
  "size": 10,
  "query": {
    "match": {
      "title": "Godfather"
    }
  }, "sort": [
    {"_score":{"order":"asc"}}
  ]
}
```

**When sorting on a field, scores are not computed**. By setting `track_scores` to true, scores will still be computed and tracked.

```shell
GET movies/_search
{
  "size": 10,
  "query": {
    "match": {
      "genre": "crime"
    }
  }, 
  "sort": [
    {"rating":{"order":"asc"}}
  ]
}
```

We can also enable sorting on multiple fields. When we sort on multiple fields, the sort order is important!

```shell
GET movies/_search
{
  "size": 10,
  "query": {
    "match": {
      "genre": "crime"
    }
  }, 
  "sort": [
    {"rating":{"order":"asc"}},
    {"release_date":{"order":"asc"}}
  ]
}
```

### 8.8.5 Manipulating the results

Occasionally, we may want to fetch only a subset of fields. For example, we may need just the title and the rating of a movie when a user searches for a certain type of rating, or we might not need the document sent out in the response by the engine.

#### SUPPRESS THE FULL DOCUMENT

To suppress the document returned in the search response, we simply need to set the flag `_source` to `false` in the query. The following listing returns the response with just the metadata.

```shell
GET movies/_search
{
  "_source": false, #A Setting the _source flag to false removes the source document from the result.
  "query": {
    "match": {
      "certificate": "R"
    }
  } 
}
```

#### FETCHING SELECTED FIELDS

Elasticsearch provides a `fields` object to indicate which fields are expected to be returned.

For example, the query in the following code snippet fetches only the `title` and `rating` fields in the response.

```shell
GET movies/_search
{
  "_source": false,
  "query": {
    "match": {
      "certificate": "R"
    }
  },
  "fields": ["title", "rating" ]
}
```

You can also use wildcards in the field’s mapping. For example, setting `title*` retrieves `title`, `title.original`, `title_long_descripion`, `title_code`, and all other fields that have the title prefix.

#### SCRIPTED FIELDS

We may at times need to compute a field on the fly and add it to the response. To use the scripting feature, append the query with the `script_fields` object.

```shell
GET movies/_search
{
  "_source": ["title*","synopsis", "rating"],
  "query": {
    "match": {
      "certificate": "R"
    }
  },
  "script_fields": {
    "top_rated_movie": {
      "script": {
        "lang": "painless",
        "source": "if (doc['rating'].value > 9.0) 'true'; else 'false'"
      }
    } 
  }
}
```

#### SOURCE FILTERING

```shell
GET movies/_search
{
  "_source": ["title*","synopsis", "rating"],
  "query": {
    "match": {
      "certificate": "R"
    }
  } 
}
```

In fact, you can take the `_source` option even further by setting a list of `includes` and `excludes` to further control the return fields. The following listing demonstrates this in action.

```shell
GET movies/_search
{
  "_source": {
    "includes": ["title*","synopsis","genre"],
    "excludes": ["title.original"]
  },
  "query": {
    "match": {
      "certificate": "R"
    }
  } 
}
```

### 8.8.6 Searching across indices and data streams

Our data is more likely than not spread across indices and data streams.

For example, omitting the index name(s) on the search request is a clue to the engine to search across all indices. The following code snippet shows this technique:

```shell
GET _search
{
  "query": {
    "match": {
      "actors": "Pacino"
    }
  } 
}
```

In fact, you can use GET `*/_search` or GET `_all/_search` too, which is equivalent to the previous query.

#### BOOSTING INDICES

When we search across multiple indices, we may want to have a document found in one index take precedence over the same document found in another index.

To demonstrate this concept, I’ve created two new indices (`index_top` and `index_new`) and indexed The Shawshank Redemption movie in these two new indices.

Now that we have the same movie across three indices, let’s create the query with a requirement of enhancing the score of the document obtained from movies_top so that it’s the topmost result.

```shell
GET movies*/_search
{
  "indices_boost": [
    { "movies": 0.1},#A
    { "movies_new": 0}, #B
    { "movies_top": 2.0} #C
  ], 
  "query": {
      "match": {
        "title": "Redemption"
      }
  } 
}
```

# Chapter 9. Term-level search

Term-level search is designed to work with structured data such as numbers, dates, IP address, enumerations, keyword types, and others. We use term-level queries to find an exact match.

## 9.1 Overview of term-level search

The term-level search produces a Yes or No binary option similar to the database’s WHERE clause.

### 9.1.1 Term-level queries are not analyzed

The terms are matched against the words stored in the inverted index without having to apply the analyzers to match the indexing pattern. This means that the search words must match with the fields indexed in the inverted index.

## 9.2 Term queries
 
The term query fetches the documents that exactly match a given field. The field is not analyzed, instead it is matched against the value that’s stored as is during the indexing in the inverted index. For example, using our movie dataset, if we were to search an R-rated movie.

```shell
GET movies/_search
{
  "query": {
    "term": { #A
      "certificate": "R"
    }
  } 
}
```

The name of the query (`term` in this case) identifies that we are about to perform a term-level search.

### 9.2.1 Term queries on text fields

This brings up an important point to consider when working with term queries: term queries are not suitable when working with `text fields`.

For whatever the reason, if you want to use a term query on a text field, make sure the text field is indexed like an enumeration or a constant. For example, an order status field with CREATED, CANCELLED, FULFILLED states can be a good candidate to use a term query though the field was a text field.

### 9.2.3 Shortened term-level queries

Original full version of term query:

```shell
GET movies/_search
{
"query": {
    "term": {
      "certificate": {
        "value": "R",
        "boost": 2
      }
    } 
  }
}
```

As the code shows, the certificate expects an object with value and other parameters.

## 9.3 Terms queries

As the name suggests, the `terms` (note down the plural) query searches multiple criteria against a single field. That is, we can throw in all the possible values of the field that we would like the search to be performed.

```shell
GET movies/_search
{
  "query": {
    "terms": {
      "certificate": ["PG-13","R"]
    }
  } 
}
```

There’s a limit of how many terms we can set in that array - a whopping of 65,536 terms. If you need to modify this limit (to increase or decrease it), you can use the index’s dynamic property setting to alter the limit: `index.max_terms_count`.

```shell
PUT movies/_settings
{
  "index":{
    "max_terms_count":10
  }
}
```

### 9.3.1 Terms lookup

We create a classic_movies index with two properties: title and director.

```shell
PUT classic_movies
{
  "mappings": {
    "properties": {
      "title": { #A
        "type": "text"
      },
      "director": { #B
        "type": "keyword"
      }
    } 
  }
}
```

As the code illustrates, there is nothing special about this index - except that the notable point is that we are defining the director field as a keyword type - for no better reason than avoiding complexity.

```shell
PUT classic_movies/_doc/1
{
  "title":"Jaws",
  "director":"Steven Spielberg"
}
PUT classic_movies/_doc/2
{
  "title":"Jaws II",
  "director":"Jeannot Szwarc"
}
PUT classic_movies/_doc/3
{
  "title":"Ready Player One",
  "director":"Steven Spielberg"
}
```

Say we wish to fetch all movies directed by the director like Speilberg. However, we wouldn’t want to construct a terms query with the terms upfront, instead we will let the query know to pick up the values of the terms from a document. That is, we let the terms query lookup the criteria from the field values of a document rather than providing them directly. 

```shell
GET classic_movies/_search
{
  "query": {
    "terms": { #A The terms query (with a twist!)
      "director": { #B The field that we are interested in searching against
        "index":"classic_movies", #C The index denotes name of the index where the document resides
        "id":"3", #D Field name which makes up the terms for the query
        "path":"director" #E The search Field in the current document
      } 
    }
  } 
}
```

The code listing requires a bit of explanation: we are creating a terms query with the director being the field against which multiple search terms are arranged. In a usual terms query, we would’ve provided an array with all the list of names. However, here we are asking the query to look up the values of the director from another document instead: the document with id as 3.

The document with this ID 3 is expected to be picked up from the classic_movies index as the index field mentions in the query. And of course the field to fetch the values is called director and is noted as path in the above code listing. Running this query will fetch two documents that were directed by Spielberg.

## 9.4 IDs queries

The following listing shows how to retrieve some documents using a list of document IDs.

```shell
GET movies/_search
{
"query": {
  "ids": {
      "values": [10,4,6,8]
    }
  } 
}
```

## 9.5 Exists queries

Sometimes, I see documents having hundreds of fields in some projects. Fetching all the fields in a response is a waste of bandwidth, and knowing if the field exists before attempting to fetch it is a better precheck. To do that, the `exists` query fetches the documents for a given field if the field exists.

```shell
GET movies/_search
{
  "query": {
    "exists": {
      "field": "title"
    }
  } 
}
```

If document with field `title` exists, we get a response - if it does not, then we get empty `hits` array.

### 9.5.1 Non existent field check

There’s another subtle use case of an exists query: when we want to retrieve all documents that don't have a particular field (a nonexistent field).

For example, we check all the documents that aren’t classified as confidential (assuming classified documents have an additional field called confidential set to true).

```shell
PUT top_secret_files/_doc/1
{
  "code":"Flying Bird",
  "confidential":true
}
PUT top_secret_files/_doc/2
{
  "code":"Cold Rock"
}
GET top_secret_files/_search
{
  "query": {
    "bool": {
      "must_not": [{
        "exists": {
            "field": "confidential"
          }
      }]
    } 
  }
}
```

We then write an exists query in a `must_not` clause of a `bool` query to fetch all the documents that are not categorized as confidential.

## 9.6 Range queries

```shell
GET movies/_search
{
  "query": {
    "range": {
      "rating": {
        "gte": 9.0,
        "lte": 9.5 
      }
    } 
  }
}
```

If you want to fetch all the movies after 1970:

```shell
GET movies/_search
{
  "query": {
    "range": {
      "release_date": {
        "gte": "01-01-1970"
      }
    } 
  },
  "sort": [
    {
      "release_date": {
        "order": "asc"
      }
    }
  ]
}
```

### 9.6.1 Range queries with data math

Elasticsearch supports sophisticated data math in queries. For example, we can ask the engine questions like:
* Fetch the book sales a couple of days back (current day minus two days).
* Find the access denied errors in the last 10 minutes (current hour minus 10 minutes).
* Get the tweets for a particular search criteria from last year.

Elasticsearch expects a specific data expression that deals with data math. An anchor date followed by `||` is the first part of the expression, appended with the time we want to add or subtract from the anchor date.

For example, to fetch movies two days after a specific day: `01-01-2022||+2d`.

```shell
GET movies/_search
{
  "query": {
    "range": {
      "release_date": {
        "lte": "01-03-2022||-2d"
      }
    } 
  }
}
```

Instead of mentioning the current date specifically, Elasticsearch lets us use a specific keyword: `now`. The `now` represents the current date.

```shell
GET movies/_search
{
  "query": {
    "range": {
      "release_date": {
        "lte": "now-1y"
      }
    }
  } 
}
```

## 9.7 Wildcard queries

Wildcard queries, as the name implies, let you search on words with missing characters, suffixes, and prefixes.

The wildcard query accepts:
* asterisk (`*`) - Lets you search for zero or more characters.
* question mark (`?`) - Lets you search for a single character.

```shell
GET movies/_search
{
  "query": {
    "wildcard": {
      "title": {
        "value": "god*"
      }
    } 
  }
}
```

We should see three movies (Godfather, Godfather II, and City of God) returned for this wildcard query.

### 9.7.1 Expensive queries

There are a few queries that can be expensive to run by the engine due to the nature of how we implement them.

**The wildcard query is one of them. The others are the range, prefix, fuzzy, regex, and join queries as well as others. Furthermore, using one of these queries occasionally might not impact server performance, but overusing these expensive queries will perhaps destabilize the cluster, leading to bad user experiences.**

If we want to put a stop to the execution of such expensive queries on the cluster, there’s a setting that we can turn off: setting the `allow_expensive_queries` attribute to `false`.

```shell
PUT _cluster/settings
{
  "transient": {
    "search.allow_expensive_queries": "false"
  }
}
```

## 9.8 Prefix queries

At times we might want to query for words using a prefix, like Leo for Leonardo or Mar for Marlon Brando, Mark Hamill, or Martin Balsam. We can use the prefix query for fetching records that match the beginning part of a word (a prefix).

```shell
GET movies/_search
{
"query": {
    "prefix": { #A
      "actors.original": {
        "value": "Mar" #B
      }
    } 
  }
}
```

The above query fetches three movies with the actors Marlon, Mark, and Martin when we search for the prefix Mar.

### 9.8.1 Speeding up prefix queries

This is because the engine has to derive the results based on a prefix (any lettered word). The prefix queries, hence, are **slow to run**, but there’s a mechanism to speed them up: using the `index_prefixes` parameter on the field.

```shell
PUT boxoffice_hit_movies
{
  "mappings": {
    "properties": {
      "title":{
        "type": "text",
        "index_prefixes":{}
      }
    } 
  }
}
```

This indicates to the engine that, during the indexing process, it should create the field with prebuilt prefixes and store those values. Elasticsearch indexes the prefixes with a minimum character size of 2 and a maximum character size of 5 by default.

Of course, we can change the default min and max sizes of the prefixes that Elasticsearch tries to create during indexing for us.

```shell
PUT boxoffice_hit_movies_custom_prefix_sizes
{
  "mappings": {
    "properties": {
      "title":{
        "type": "text",
        "index_prefixes":{
          "min_chars":4,
          "max_chars":10
        }
      } 
    }
  } 
}
```

## 9.9 Fuzzy queries

Spelling mistakes during a search are common. We may at times search for a word with an incorrect letter or letters; for example, searching for rama movies instead of drama movies.

The search can correct this query and return "drama" movies instead of failing. The principle behind this type of query is called fuzziness, and Elasticsearch employs fuzzy queries to forgive spelling mistakes.

Fuzziness is a process of searching for similar terms based on the Levenshtein distance algorithm.

```shell
GET movies/_search
{
  "query": {
    "fuzzy": {
      "genre": {
        "value": "rama",
        "fuzziness": 1
      }
    } 
  },
  "highlight": {
    "fields": {
      "genre": {} 
    }
  } 
}
```

In this example, we use the edit distance of 1 (one character) to fetch similar words.

This might be a clumsy way of handling things because sometimes you wouldn't know if the user has mistyped one letter or a few letters. This is the reason Elasticsearch provides a default setting for fuzziness: the AUTO setting. If the fuzziness attribute is not supplied, the default setting of AUTO is assumed. The AUTO setting deduces the edit distance based on the length of the word.

| Word length in characters | Fuzziness |
|---------------------------|-----------|
| 0 to 2                    | 0         |
| 3 to 5                    | 1         |
| More than 5               | 2         |

# Chapter 10. Full-text search

A full text search is all about relevancy: fetching the documents that are relevant to the user's search.

## 10.1 Overview

When we talk about relevance, there’s two measures that usually spring up: precision and recall.

### 10.1.1 Precision

Precision is measured as the percentage of relevant documents in the overall number of documents returned. When a query returns results, not all of the results are directly related to the query.

Precision = relevant results/all results.

Search for TV returns 6 TV models and 4 other devices. Precision is 60% then.

### 10.1.2 Recall

Recall is the other side of the coin. It measures how many documents that were returned are relevant. For example, there may have been a few more relevant results (more TVs) that weren’t returned as part of the results set. The percentage of relevant documents that were retrieved is called recall.

Say we have three more TVs that fell into the search bucket but were never returned. These are called false negatives. On the other hand, there are a few products like cameras and projectors that weren’t returned, which are genuinely irrelevant and, hence, not expected to be part of the result. These are termed as true negatives. In the current scenario, recall is calculated as.

Recall = 6 / (6+3) × 100% = 66.6%

Ideally, we want precision and recall to be a perfect match with no omissions (no relevant documents omitted). However, this is pretty much impossible because these two measures always work against each other. They are indeed inversely proportional to each other: the higher the precision (number of best match documents), the lower the recall (the number of documents returned).

We can use match queries, filters, and boost to tweak precision and recall to fine tune the balance.

## 10.2 Sample data

We will work with a fictional book store in this chapter, where we will index a set of 50 technical books into an index named books by invoking the `_bulk` API. We are not tweaking the mapping for this part of the sample data, so you can go ahead and index the books as is. The data for the books is available on my GutHub page here:

https://github.com/madhusudhankonda/elasticsearch-in-action/blob/f334df2dd5f10acc15bfc745f580ec9be0b129d2/datasets/books.txt
https://github.com/madhusudhankonda/elasticsearch-in-action/blob/f334df2dd5f10acc15bfc745f580ec9be0b129d2/kibana_scripts/ch10_full_text_queries.txt

## 10.3 Match all (match_all) queries

As the name suggests, the match all (match_all) query fetches all the documents available in the index. Because this query is expected to return all the available documents, it is the perfect partner for honoring 100% recall.

### 10.3.1 Building the match_all query

We form the query with a match_all object, passing it no parameters.

```shell
GET books/_search
{
"query": {
    "match_all": {}
  }
}
```

This query returns all documents available in the `books` index. The notable point is that the response indicates each of the books with a score of 1.0 by default.

### 10.3.2 Short form of a match_all query

We wrote a match_all query with a query body, however, providing the body is redundant. That is, the same query can be rewritten in a shorter format like this: `GET books/_search`.

## 10.4 Match none (match_none) queries

While the `match_all` query returns all the results from an index or multiple indices, the opposite query, called `match_none`, returns no results.

```shell
GET books/_search
{
  "query": {
    "match_none": {}
  }
}
```

## 10.5 Match queries

The `match` query is the most common and powerful query for multiple use cases. It is a full text search query returning the documents that match the specified criteria.

### 10.5.1 Format of a match query

```shell
GET books/_search
{
  "query": {
    "match": { # The type of the query is a match query
      "FIELD": "SEARCH TEXT" # The query expects a criteria to be specified in the form of a name-value pair.
    }
  } 
}
```

The `match` query expects the search criteria to be defined in the form of a field value. The field can be any of the text fields present in a document, whose values are to be matched.

There are a handful of additional parameters in the query’s full form that we can pass to the `match` query too.

```shell
GET books/_search
{
  "query": {
    "match": {
      "FIELD": { # Declares FIELD as an object with additional parameters
        "query":"<SEARCH TEXT>", # The query attribute holds the search text.
        "<parameter>":"<MY_PARAM>", # Other optional parameters (such as analyzer, operator, prefix_length, fuzziness, etc.) expects a value to be set.
     }
    } 
  }
}
```

### 10.5.2 Searching using a match query

An example where we want to search for Java books with Java in the title field.

```shell
GET books/_search
{
  "query": {
  "match": { # The match query in action
      "title": "Java" # Sets the search criteria, searching for the word Java in the title field
    }
}
```

### 10.5.3 Match query analysis

The match queries that work on text fields are analyzed. The same analyzers used during the indexing process the search words in match queries.

Additionally, the standard analyzer applies the same lowercase token filter.

### 10.5.4 Searching multiple words

```shell
GET books/_search
{
  "query": {
    "match": {
      "title": {
        "query": "Java Complete Guide"
      }
    } 
  },
  "highlight": {
    "fields": {
      "title": {} 
    }
  } 
}
```

If we execute the query with these words, you may be surprised to see more documents than just the one that matches exactly with the search query.

The reason for this behavior is that Elasticsearch employs an OR Boolean operator by default for this query, so it fetches all the documents that match with any of the words.

```shell
GET books/_search
{
  "query": {
    "match": {
      "title": {
        "query": "Java Complete Guide",
        "operator": "AND" #A
      }
    } 
  }
}
```

### 10.5.5 Matching at least a few words

The OR and AND operators are opposing conditions. The OR condition fetches either of the search words, and the AND condition gets matching documents exactly for all of the words. What if we want to find documents that match at least a few words from the given set of words? This is where the `minimum_should_match` attribute comes in handy.

The `minimum_should_match` attribute indicates the minimum number of words that should be used to match the documents.

```shell
GET books/_search
{
  "query": {
    "match": {
      "title": {
        "query": "Java Complete Guide",
        "operator": "OR",
        "minimum_should_match": 2 # Sets the minimum number of words that should match
      } 
    }
  } 
}
```

### 10.5.6 Fixing typos using the key word fuzziness

```shell
GET books/_search
{
  "query": {
    "match": {
      "title": {
        "query": "Kava",
        "fuzziness": 1
      }
    } 
  }
}
```

## 10.6 Match phrase (match_phrase) queries

The match phrase (match_phrase) query finds the documents that match exactly a given phrase. The idea behind the match phrase is to search for the phrase (group of words) in a given field in the same order. For example, if you are looking for the phrase “book for every Java programmer” in the synopsis of a book, documents are searched with those words in that order.

From our previous section on match queries, we learned that words can be split individually and searched with an AND/OR operator when using a match query. The `match_phrase` query is the opposite.

```shell
GET books/_search
{
  "query": {
    "match_phrase": {
      "synopsis": "book for every Java programmer"
    }
  } 
}
```

### 10.6.1 Match phrase with the keyword slop

What if we drop a word or two in between the said phrase? Say, for example, we remove the for or every (or both) from the phrase “book for every Java programmer” and rerun the same query. Unfortunately, the query wouldn't return any results! The reason for this is that match_phrase expects the words in a phrase to match the exact phrase, word by word. Searching “book Java programmer” returns no results. Fortunately, there is a fix to this problem: using a parameter called `slop`.

The slop parameter allows us to ignore the number of words in between the words in that phrase. We can drop the in-between words in the phrase. However, we need to let the engine know how many words to drop.

```shell
GET books/_search
{
"query": {
    "match_phrase": {
      "synopsis": { #A
        "query": "book every Java programmer",#B
        "slop": 1#C
      } 
    }
  } 
}
```

## 10.7 Match phrase prefix (match_phrase_prefix) queries

The match phrase prefix (`match_phrase_prefix`) query is a slight variation of the `match_phrase` query in that, in addition to matching the exact phrase, the query matches all the words, using the last word as a prefix.

```shell
GET books/_search
{
  "query": {
    "match_phrase_prefix": { # Specifies the match_prefix_query query
      "tags": {
        "query": "Co" # Specifies the prefix to search
      }
    } 
  },
  "highlight": {
    "fields": {
      "tags": {} 
    }
  } 
}
```

This query fetches all the books with tags matching Co. This includes prefixes such as Component, Core, Code, and so on.

### 10.7.1 Match phrase prefix using slop

Similar to the match_phrase query, the order of the words is important in the match_phrase_prefix query too. Of course, slop is here to the rescue. For example, when we want to retrieve books with the phrase concepts and foundations across the tags field, we can omit and by adding the keyword slop as the following listing demonstrates.

```shell
GET books/_search
{
  "query": {
    "match_phrase_prefix": {
      "tags": {
        "query": "concepts found", # The phrase has one word (and) omitted as well as has a prefix (“found” instead of “foundations”).
        "Slop":1 # Sets slop to 1 because one word is dropped from the phrase
      }
    } 
  }
}
```

## 10.8 Multi-match (multi_match) queries

The multi-match (`multi_match`) query, as the name suggests, searches the query across multiple fields. For example, if we want to search for the word Java across the three fields `title`, `synopsis`, and `tags`, then the `multi_match` query is the answer.

```shell
GET books/_search
{
  "_source": false, # Suppresses the source document showing up in the results
  "query": {
    "multi_match": { # Specifies the multi_match query
      "query": "Java", # Defines the search criteria as the word Java
      "fields": [ # Searches across multiple fields provided in an array
        "title",
        "synopsis",
        "tags"
      ] 
    }
  },
  "highlight": { # Highlights the matches returned in the results
    "fields": {
      "title": {},
      "tags": {} 
    }
  } 
}
```

### 10.8.1 Best fields

The best_fields type is most useful when you are searching for multiple words best found in the same field. For instance “brown fox” in a single field is more meaningful than “brown” in one field and “fox” in the other.

```shell
GET books/_search
{
  "_source": false,
  "query": {
    "multi_match": {
      "query": "Design Patterns",
      "type": "best_fields", # Sets the type of multi_match query to best_fields
      "fields": ["title","synopsis"]
    } 
  },
  "highlight": { # Suppresses the source but shows the highlights
    "fields": {
      "tags": {},
      "title": {} 
    }
  } 
}
```

### 10.8.2 Disjunction max (dis_max) queries

In the previous section, we looked at the `multi_match` query, where the criteria was searched against multiple fields. How does this query type get executed behind the scenes? Elasticsearch rewrites the `multi_match` query using a disjunction max query (`dis_max`).

```shell
GET books/_search
{
  "_source": false,
  "query": {
    "dis_max": {
      "queries": [
        {"match": {"title": "Design Patterns"}},
        {"match": {"synopsis": "Design Patterns"}}]
    }
  } 
}
```

Multiple fields are split into two match queries under the dis_max query. The query returns the documents with a high relevancy `_score` for the individual field.

### 10.8.3 Tiebreakers

The relevancy score is based on the single field’s score, but if the scores are tied, we can specify `tie_breaker` to relieve the tie. If we use `tie_breaker`, Elasticsearch calculates the overall score slightly differently which we will see in action shortly, but first, let’s checkout an example.

```shell
GET books/_search
{
  "query": {
    "multi_match": {
      "query": "Design Patterns",
      "type": "best_fields",
      "fields": ["title","tags"],
      "tie_breaker": 0.9
    } 
  }
}
```

When we provide the tie breaker, the overall scoring is calculated as:

```
Overall score = _score of the best match field + _score of the other matching fields * tie_breaker
```

**Elasticsearch converts all multi_match queries to the `dis_max` query.**

### 10.8.4 Individual field boosting

```shell
GET books/_search
{
  "query": {
    "multi_match": {
      "query": "C# Guide",
      "fields": ["title^2", "tags"] #Doubles up the title field
    }
  } 
}
```

## 10.9 Query strings

Earlier on in chapter 8, we looked at the URI search method (one of the search query methods in addition to Query DSL).

Example of query string:

```
title:Java and author:Bert and edition:2 and release_date>=2000-01-01
```

## 10.10 Query string(query_string)queries

Same can be achieved via Query DSL:

```shell
GET books/_search
{
  "query": {
    "query_string": { #A
      "query": "author:Bert AND edition:2 AND release_date>=2000-01-01" #B
    }
  } 
}
```

### 10.10.1 Fields in a query string query

In a typical search box, the user does not need to mention the field when searching for something. For example:

```shell
GET books/_search
{
"query": {
    "query_string": {
      "query": "Patterns"
    }
  },
  "highlight": {
    "fields": {
      "title": {},
      "synopsis": {},
      "tags": {}
    } 
  }
}
```

One quick thing to remember is that the query is not asking us to search any fields. It is a generic query that’s actually expected to be executed across **all fields**.

Instead of letting the engine search query against all the available fields, we can assist Elasticsearch by providing the fields to run the search on.

```shell
GET books/_search
{
  "query": {
    "query_string": {
      "query": "Patterns", #A The query criteria with no fields mentioned.
      "fields": ["title","synopsis","tags"] #B Explicitly declares the fields as an array of strings
    }
  } 
}
```

Here, we specify the fields explicitly in an array in the fields parameter and mention the fields that this criteria is expected to be performed against.

```shell
GET books/_search
{
  "query": {
    "query_string": {
      "query": "Patterns",
      "default_field": "title"
    }
  } 
}
```

If a field is not mentioned in the query, the search is carried out against the `title` field.

### 10.10.2 Default operator

If we extend that search to include an additional word such as Design, we may get multiple books (two books with the current dataset) instead of the correct one: Head First Design Patterns. The reason is that Elasticsearch uses the OR operator by default when searching. Hence, it finds books with both words, Design OR Patterns in the title field.

```shell
GET books/_search
{
  "query": {
    "query_string": {
      "query": "Design Patterns",
      "default_field": "title",
      "default_operator": "AND" # Changes the operator from OR to AND
    } 
  }
}
```

## 10.11 Query string with a phrase

The query in the next listing searches for a phrase.

```shell
GET books/_search
{
  "query": {
    "query_string": {
      "query": "\"making the code better\"", # Quotes around the sentences make it a phrase query
      "default_field": "synopsis"
    }
  } 
}
```

As you can expect, this code searches for the phrase "making the code better" in the `synopsis` field and fetches the Effective Java book.

For example, the code in the following listing demonstrates how the phrase_slop parameter allows for a missing word in the phrase (the is dropped from the phrase) and still gets a successful result.

```shell
GET books/_search
{
  "query": {
    "query_string": {
      "query": "\"making code better\"", #A
      "default_field": "synopsis",
      "phrase_slop": 1 # Sets the phrase_slop to 1 so the phrase with one missing word is honored
    } 
  }
}
```

## 10.12 Fuzzy queries

We can also ask Elasticsearch to forgive spelling mistakes by using fuzzy queries with `query_string` queries. All we need to do is suffix the query criteria with a tilde (~) operator.

```shell
GET books/_search
{
  "query": {
    "query_string": {
      "query": "Pattenrs~", #A
      "default_field": "title"
    }
  } 
}
```

By setting the suffix with the ~ operator, we are cueing the engine to consider the query as a fuzzy query.

By default, the edit distance in a query_string query is 2, but we can reduce it if needed by setting the 1 after the tilde like so: "Pattenrs~1".

## 10.13 Simple string queries

## 10.14 Simple_query_stringqueries

As the name suggests, the `simple_query_string` query is a variant of the `query_string` query with a simple and limited syntax. We can use operators such as `+`, `-`, `|`, `*`, `~` and so forth for constructing the query. For example, searching for "Java + Cay" produces a Java book written by Cay.

```shell
GET books/_search
{
  "query": {
    "simple_query_string": {
      "query": "Java + Cay"
    }
  } 
}
```

Unlike the `query_string` query, the `simple_query_string` query doesn’t respond with errors if there’s any syntax error in the input criteria. It takes a quieter side of not returning anything should there be a syntactic error.
