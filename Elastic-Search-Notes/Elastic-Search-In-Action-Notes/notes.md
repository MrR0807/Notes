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
"hits": [{
  "_score" : 7.300332,
  "_source" : {
  "title" : "Effective Java",
  "synopsis" : "A must-have book for every Java programmer and Java ...",
}]}
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















































