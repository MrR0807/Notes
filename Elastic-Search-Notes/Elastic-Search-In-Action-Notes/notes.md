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





















