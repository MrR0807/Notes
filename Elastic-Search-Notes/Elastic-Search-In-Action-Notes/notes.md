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

















