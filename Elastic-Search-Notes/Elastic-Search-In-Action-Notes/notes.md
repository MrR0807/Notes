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







