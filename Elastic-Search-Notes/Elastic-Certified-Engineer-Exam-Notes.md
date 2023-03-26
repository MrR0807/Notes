# Introduction

Elasticstack:
* Beats
* Logstash - input, parse, enrich and output live data stream
* Kibana

Cluster - is a distributed entity which allows for horizontal scaling and redundancy.

Nodes (main nodes):
* Master-eligible nodes can be elected as the master node which coordinates the cluster
* Data nodes index, store and analyze data
* Ingest nodes can collect and process data

Indices - a logical structure composed of shards. Shards contain *portion* of index. If you have three nodes, the data can be distributed through 3 nodes by placing 33% of index data on each node. Replica shards are copies of primary shard. They cannot be allocated to the same node of their primary shard.

Cluster states:
* Green state means all primary and replica shards are allocated
* Yellow state means all primary shards, but not all replica shards are allocated
* Red state means not all primary shards are allocated

Credentials for Cloud Playground:
* user: elastic
* password: elastic_acg

Index Anatomy:
* aliases - allow us to simplify how we reference one or many indices.
* mappings - allow us to dynamically or explicitly determine how our data is indexed.
* settings - allow us to customize index behavior and allocation.

First things to check when logged in into Elastic:
```shell
GET _cat
GET _cat/nodes?v
GET _cat/indices?v
```

Index Templates
* index patterns - automatically manage any index that matches the index pattern
* advanced features - data streams, index lifecycyle management, snapshotting etc
* component templates - reusable building blocks for constructing index templates

```shell
# Legacy endpoint was _template
PUT _index_template/earthquakes_template
{
  # This property has to be defined. Wildcard at the end is a pattern
  "index_patterns": ["earthquakes-*"],
  # This is new syntax
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 0
    }
  }
}
```

```shell
GET _index_template/earthquakes_template

# Will create with index_template
PUT earthquakes-1
GET earthquakes-1
```

```shell
PUT _component_template/shards-component
{
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 0
    }
  }
}

GET _component_template/shards-component
```

Use component template

```shell
PUT _component_template/shards-component
{
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 0
    }
  }
}
```

```shell
PUT _index_template/earthquakes_template
{
  "index_patterns": ["earthquakes-*"],
  "composed_of": ["shards-component"]
}
```

```shell
PUT earthquakes-1
GET earthquakes-1
```

Will create an index with defined template.


## Data Visualizer

Understand the fields of a dataset by quickly uploading and analyzing it in order to determine whether to ingest it into Elasticsearch.

## Index Lifecycle Management (ILM)

Automatically manage your indices based on your usage requirements:
* phases - prioritize your data based on how you use it with hot, warm, cold and delete phases
* phase transition - move indices through the lifecycle based on age, size, or document count.
* phase executions - perform actions on indices at each phase, like rollover, force merge, migrate, shrink, freeze and delete.

Index Lifecycle:
* hot - the index is actively being written to and frequently queried.
* warm - the index is no longer being written to, but is still queried.
* cold - the index is not being written to and is infrequently queried.
* delete - the index is no longer needed and can be deleted from the cluster.

```shell
PUT _ilm/policy/applogs-policy
{
  "policy": {
    "phases": {
      # This means that data is right away placed into hot phase
      "hot": {
        "actions": {
        }
      }
    }
  }
}
```

Adding "warm":

```shell
PUT _ilm/policy/applogs-policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
        }
      },
      "warm": {
        # It will rollover after one day
        "min_age": "1d",
        "actions": {
          # This is very good for read performance
          "forcemerge": {
            "max_num_segments": 1
          }
        }
      }
    }
  }
}
```


```shell
PUT _ilm/policy/applogs-policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {}
      },
      "warm": {
        "min_age": "1d",
        "actions": {
          "forcemerge": {
            "max_num_segments": 1
          }
        }
      },
      "cold": {
        "min_age": "7d",
        "actions": {
          "freeze": {}
        }
      },
      "delete": {
        "min_age": "30d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

```shell
PUT _index_template/applogs-template
{
  "index_patterns": ["applogs-*"],
  "composed_of": ["shards-component"],
  "template": {
    "settings": {
      "index.lifecycle.name": "applogs-policy"
    }
  }
}
```

```shell
GET _index_template/applogs-template
{
  "index_templates" : [
    {
      "name" : "applogs-template",
      "index_template" : {
        "index_patterns" : [
          "applogs-*"
        ],
        "template" : {
          "settings" : {
            "index" : {
              "lifecycle" : {
                "name" : "applogs-policy"
              }
            }
          }
        },
        "composed_of" : [
          "shards-component"
        ]
      }
    }
  ]
}
```

```shell
PUT applogs-1
GET applogs-1
{
  "applogs-1" : {
    "aliases" : { },
    "mappings" : { },
    "settings" : {
      "index" : {
        "lifecycle" : {
          "name" : "applogs-policy"
        },
        "routing" : {
          "allocation" : {
            "include" : {
              "_tier_preference" : "data_content"
            }
          }
        },
        "number_of_shards" : "1",
        "provided_name" : "applogs-1",
        "creation_date" : "1679155611390",
        "number_of_replicas" : "0",
        "uuid" : "Myxfa4UeSdaxtToVvr5kng",
        "version" : {
          "created" : "7130499"
        }
      }
    }
  }
}
```

## Creating Data Streams

Data Streams - store and search time series data spread across multiple indices with a single resource.

```shell
PUT _ilm/policy/weblogs
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_primary_shard_size": "5gb"
          },
          "forcemerge": {
            "max_num_segments": 1
          }
        }
      },
      # The min_age is from the point of rollover. If rollover did not exist, then it is from the minute the 
      # data is indexed. So in this case, if say, I'd like to move data to cold immediately after rollover
      # then min_age: "0ms"
      "cold": {
        "min_age": "7d",
        "actions": {
          "freeze": {}
        }
      },
      "delete": {
        "min_age": "30d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

Create data-streams:

```shell
PUT _index_template/weblogs
{
  # This name: weblogs has to match in _data_stream/weblogs !
  "index_patterns": ["weblogs"],
  "data_stream": {},
  "composed_of": ["shards-component"],
  "template": {
    "settings": {
      "index.lifecycle.name": "weblogs"
    }
  }
}
```

```shell
PUT _data_stream/weblogs
GET _data_stream/weblogs
```

# Introduction to Searching Data

## Understanding the ElasticSearch Query DSL

## Analyzed vs Non-Analyzed

Original text: "The students learned a NEW concept".

Analyzed: 
* tokenized - students | learned | NEW | concept;
* normalized - student | learn | new | concept;


Non-Analyzed:
* tokenized - "The students learned a NEW concept";
* normalized - "the students learned a new concept";

Relevancy Scoring:
* Term Frequency - how often does the term appear in the field. The more it appears, the higher the score;
* Inverse Document Frequency - How often does the term appear in the index?
* Field-Length Normalization - How many other terms are in the field? 
The more terms are in the field along with my search term the less relevant is the score.

## Query and Filter Context

Query Context
* How well does the search clause match the document? Answer: <relevancy_score>

Filter Context
* Does the search clause match the document? Answer: True or False.

This is important to remember! 

You might be asked, during the exam, to create a complex, compound query, where certain parts of the query do not have an effect 
on relevancy score. That should be a red flag that whenever I do that part of the query in needs to be in filter context, everything
else that needs to affect the relevancy_score needs to be in query context.

## Term-Level Queries

Term-level queries do not analyze search terms. Search for documents based on exact values.

```shell
# Returns 0 results
GET shakespeare/_search
{
  "query": {
    "term": {
      "text_entry.keyword": {
        "value": "yes"
      }
    }
  }
}

# Returns 8 results
GET shakespeare/_search
{
  "query": {
    "term": {
      "text_entry.keyword": {
        "value": "Yes."
      }
    }
  }
}
```

**Avoid using term-level queries on analyzed fields**

When you autocomplete on searchable field. You can see the field type: "keyword" or "text". If it is keyword, 
that means it is non-analyzable type.

```shell
GET flights/_search
{
  "query": {
    "term": {
      "DestCountry": {
        "value": "US"
      }
    }
  }
}
```

```shell
GET flights/_search
{
  "query": {
    "terms": {
      "Carrier": [
        "Kibana Airlines",
        "Logstash Airways"
      ]
    }
  }
}
```

```shell

# https://www.elastic.co/guide/en/elasticsearch/reference/8.1/common-options.html#date-math
GET flights/_search
{
  "query": {
    "range": {
      "timestamp": {
        "gte": "now-1h",
        "lte": "now"
      }
    }
  }
}

GET flights/_search
{
  "query": {
    "range": {
      "timestamp": {
        "gte": "2023-03-30T08:00:00",
        "lte": "2023-03-30T08:55:49"
      }
    }
  }
}
```

## Lab

```shell
GET ecommerce/_search
{
  "query": {
    "term": {
      "order_id": {
        "value": "725676"
      }
    }
  }
}

GET ecommerce/_search
{
  "query": {
    "terms": {
      "customer_full_name.keyword": [
        "Samir Garza",
        "Selena Gibbs",
        "Tariq Duncan"
      ]
    }
  },
  "fields": [
    "geoip.city_name"
  ],
  "_source": false
}

GET ecommerce/_search
{
  "query": {
    "range": {
      "taxful_total_price": {
        "gte": 10,
        "lte": 20
      }
    }
  }
}
```

## Full-Text Queries

* The query is processed using the same analyzer as the text field.
* Text fields can be indexed and searched using custom analyzers.
* Avoid using full-text queries on non-analyzed fields.

```shell
GET shakespeare/_search
{
  "query": {
    "match": {
      "text_entry": "wherefore art thou romeo"
    }
  }
}
```

It will return results containing:
* "text_entry" : "O Romeo, Romeo! wherefore art thou Romeo?"
* "text_entry" : "Why, Romeo, art thou mad?"
* "text_entry" : "now art thou sociable, now art thou Romeo; now art"
* "text_entry" : "Art thou not Romeo and a Montague?"
* "text_entry" : "Wherefore is that? and what art thou that darest"
* "text_entry" : "Thou art Dromio, thou art my man, thou art thyself."

ElasticSearch tries to overlap the search query and result. Not necessarily all words have to be in the "text_entry".

```shell
GET shakespeare/_search
{
  "query": {
    "match_phrase": {
      "text_entry": "wherefore art thou romeo"
    }
  }
}
```

This will return only one result, because it matches exact query.

## Compound Search Queries

### Boolean Query

Search for documents matching a boolean combination of queries. With boolean queries, the scores of `must` and `should` clauses will be added together to calculate the final relevancy score for each result:
* `must` - The search term must appear and is scored;
* `should` - The search term should appear and is scored. you can configure how many should clauses must match;
* `must_not` - The search term must not appear and is not scored;
* `filter` - The search term must appear but is not scored.

```shell
GET ecommerce/_search

GET ecommerce/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "range": {
            "taxful_total_price": {
              "lte": 20
            }
          }
        }
      ],
      "should": [
        {
          "term": {
            "category.keyword": {
              "value": "Men's Clothing"
            }
          }
        },
        {
          "term": {
            "category.keyword": {
              "value": "Men's Shoes"
            }
          }
        }
      ],
      "minimum_should_match": 1,
      "must_not": [
        {
          "match": {
            "products.product_name": "sweatshirt"
          }
        }
      ],
      "filter": [
        {
          "range": {
            "order_date": {
              "gte": "now/d",
              "lte": "now"
            }
          }
        }
      ]
    }
  }
}
```


### Lab

```shell
GET _cat/indices

GET logs/_search

GET logs/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match_phrase_prefix": {
            "request": "apm-server"
          }
        },
        {
          "terms": {
            "extension.keyword": [
              "rpm",
              "deb"
            ]
          }
        },
        {
          "term": {
            "machine.os.keyword": {
              "value": "osx"
            }
          }
        }
      ],
      "filter": [
        {
          "range": {
            "bytes": {
              "gte": 1024,
              "lte": 8192
            }
          }
        }
      ]
    }
  }
}


GET logs/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "response.keyword": {
              "value": "200"
            }
          }
        }
      ],
      "should": [
        {
          "term": {
            "tags.keyword": {
              "value": "success"
            }
          }
        },
        {
          "term": {
            "tags.keyword": {
              "value": "security"
            }
          }
        },
        {
          "term": {
            "tags.keyword": {
              "value": "info"
            }
          }
        }
      ],
      "minimum_should_match": 2
      , "must_not": [
        {
          "match": {
            "machine.os": "win"
          }
        }
      ]
    }
  }
}




GET logs/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "terms": {
            "geo.src": [
              "US",
              "CN"
            ]
          }
        }
      ],
      "must_not": [
        {
          "term": {
            "response.keyword": {
              "value": "200"
            }
          }
        }
      ]
    }
  }
}
```

## Executing Asynchronous Search Queries

Instead of GET to `_search` API, do a POST to `_async` API. Then we'll get an ID. With the ID, we can check the status of `_async` search. You can get partial results with `_async` search. This is useful when search is taking a long time. Lastly, we can delete the search.

Flow:
submit search -> get ID -> check status of the search -> retrieve partial results -> delete search.

If the `async` call is under <2sec, then it will be execute as `sync` call.

```shell

POST ecommerce/_async_search
GET _async_search/status/FmVwOE4wRnpPVEstWnFoekxTZXo1MEEccDEzdkptZ0FUQXF6ZlFRZDh0YVhyUToxMTQ0OA
GET _async_search/FmVwOE4wRnpPVEstWnFoekxTZXo1MEEccDEzdkptZ0FUQXF6ZlFRZDh0YVhyUToxMTQ0OA
DELETE _async_search/FmVwOE4wRnpPVEstWnFoekxTZXo1MEEccDEzdkptZ0FUQXF6ZlFRZDh0YVhyUToxMTQ0OA
```

### Lab

```shell
DELETE _async_search/FnBrdFFoSm1ZUkN5OUR3bzd2c2U3cXccQzg0WXpYd29SQ3lXcm9uTG9HUHVxQToxMTUxMQ==

GET _async_search/FnBrdFFoSm1ZUkN5OUR3bzd2c2U3cXccQzg0WXpYd29SQ3lXcm9uTG9HUHVxQToxMTUxMQ==

GET _async_search/status/FnBrdFFoSm1ZUkN5OUR3bzd2c2U3cXccQzg0WXpYd29SQ3lXcm9uTG9HUHVxQToxMTUxMQ==

POST filebeat-7.13.4/_async_search?wait_for_completion_timeout=0
{
  "size": 100, 
  "query": {
    "match": {
      "message": "SSH"
    }
  }
}
```

## Cross-Cluster Search

Run a search request against 1 or more remote clusters. You can search remote clusters that are 1 major version behind or ahead of the coordinating node. 

Create a new cluster in Playground. And then link them:

```shell
# Private remote cluster IP 172.31.26.160
# There are two ports that Elastic uses: 9200 (http), 9300 (inter-node communication)


# This searches both clusters
GET filebeat-7.13.4,cluster_2:filebeat-7.13.4/_search
{
  "query": {
    "range": {
      "@timestamp": {
        "gte": "now-1h",
        "lte": "now"
      }
    }
  }
}


# This searches the local cluster
GET filebeat-7.13.4/_search
{
  "query": {
    "range": {
      "@timestamp": {
        "gte": "now-1h",
        "lte": "now"
      }
    }
  }
}

# This searches the remote cluster
GET cluster_2:filebeat-7.13.4/_search
{
  "query": {
    "range": {
      "@timestamp": {
        "gte": "now-1h",
        "lte": "now"
      }
    }
  }
}

PUT _cluster/settings
{
  "persistent": {
    "cluster": {
      "remote": {
        "cluster_2": {
          "seeds": ["172.31.26.160:9300"]
        }
      }
    }
  }
}
```

### Lab

```shell
PUT _cluster/settings
{
  "persistent": {
    "cluster": {
      "remote": {
        "es2": {
          "seeds": ["10.0.1.102:9300"]
        },
        "es3": {
          "seeds": ["10.0.1.103:9300"]
        }
      }
    }
  }
}

GET es2:metricbeat-7.13.4/_search

GET es3:metricbeat-7.13.4/_search

GET metricbeat-7.13.4,es2:metricbeat-7.13.4,es3:metricbeat-7.13.4/_search
```

# Aggregating Data

## Metrics Aggregation

Computes numeric values. Metrics aggregations are either single or multi-value aggregations that can operate on a variaty of non-analyzed fields to produce a numerical value. 

```shell
# For aggregation we still use the same _search

GET ecommerce/_search
{
  "size": 0, # simplify the output, because we don't care about hits and results
  "aggs": {
    "total_sales": {
      "sum": {
        "field": "taxless_total_price"
      }
    }
  }
}
```

```shell
GET _cat/indices

GET earthquakes/_search

# These are single value aggregations
GET earthquakes/_search
{
  "size": 0, 
  "aggs": {
    "does_not_matter_what_name_is_here": {
      "avg": {
        "field": "Magnitude"
      }
    },
    "another": {
      "max": {
        "field": "Magnitude"
      }
    }
  }
}

# These are multi-value aggregations

GET earthquakes/_search
{
  "size": 0,
  "aggs": {
    "NAME": {
      "stats": {
        "field": "Magnitude"
      }
    }
  }
}

GET earthquakes/_search
{
  "size": 0
  , "aggs": {
    "NAME": {
      "terms": {
        "field": "Type"
      }
    }
  }
}
```

## Bucket Aggregations

Creates buckets of documents. Establish a criterion to categorize documents into groups or buckets.

```shell
GET earthquakes/_search
{
  "size": 0,
  "aggs": {
    "NAME": {
      "date_histogram": {
        "field": "Date",
        "calendar_interval": "month"
      }
    }
  }
}

GET earthquakes/_search
{
  "size": 0,
  "aggs": {
    "NAME": {
      "terms": {
        "field": "Type",
        "size": 10
      }
    }
  }
}
```

## Writing Sub-Aggregations

Aggregates per bucket or per previous aggregation. Each bucket of a parent pipeline aggregation can have sub-aggregations performed on it.

```shell
GET ecommerce/_search
{
  "size": 0,
  "aggs": {
    "total_sales_per_day": { # any name
      "date_histogram": { # bucket agg
        "field": "order_date",
        "calendar_interval": "day"
      },
      "aggs": { # sub-agg
        "total_sales": { # any name
          "sum": { # metrics agg
            "field": "taxless_total_price"
          }
        }
      }
    }
  }
}
```

```shell
GET earthquakes/_search
{
  "size": 0,
  "aggs": {
    "NAME": {
      "date_histogram": {
        "field": "Date",
        "calendar_interval": "month"
      }
    }
  }
}
```

This returns:


```shell
 "aggregations" : {
    "NAME" : {
      "buckets" : [
        {
          "key_as_string" : "01/01/1965",
          "key" : -157766400000,
          "doc_count" : 13
        },
        {
          "key_as_string" : "02/01/1965",
          "key" : -155088000000,
          "doc_count" : 54
        },
        {
          "key_as_string" : "03/01/1965",
          "key" : -152668800000,
          "doc_count" : 38
        },
        ...
```

Adding sub-aggregation:

```shell
GET earthquakes/_search
{
  "size": 0,
  "aggs": {
    "NAME": {
      "date_histogram": {
        "field": "Date",
        "calendar_interval": "month"
      },
      "aggs": {
        "avg_magnituted_whatever": {
          "avg": {
            "field": "Magnitude"
          }
        }
      }
    }
  }
}
```

Generates:

```shell
 "aggregations" : {
    "NAME" : {
      "buckets" : [
        {
          "key_as_string" : "01/01/1965",
          "key" : -157766400000,
          "doc_count" : 13,
          "avg_magnituted_whatever" : {
            "value" : 6.123076923076923
          }
        },
        {
          "key_as_string" : "02/01/1965",
          "key" : -155088000000,
          "doc_count" : 54,
          "avg_magnituted_whatever" : {
            "value" : 5.955555555555556 # This is per month
          }
        },
        ...
```


```shell
GET earthquakes/_search
{
  "size": 0,
  "aggs": {
    "per_year": {
      "date_histogram": {
        "field": "Date",
        "calendar_interval": "year"
      },
      "aggs": {
        "types": {
          "terms": {
            "field": "Type",
            "size": 10
          }
        }
      }
    }
  }
}
```

Generates:

```shell
"aggregations" : {
    "per_year" : {
      "buckets" : [
        {
          "key_as_string" : "01/01/1965",
          "key" : -157766400000,
          "doc_count" : 339,
          "types" : {
            "doc_count_error_upper_bound" : 0,
            "sum_other_doc_count" : 0,
            "buckets" : [
              {
                "key" : "Earthquake",
                "doc_count" : 339
              }
            ]
          }
        },
        {
          "key_as_string" : "01/01/1966",
          "key" : -126230400000,
          "doc_count" : 234,
          "types" : {
            "doc_count_error_upper_bound" : 0,
            "sum_other_doc_count" : 0,
            "buckets" : [
              {
                "key" : "Earthquake",
                "doc_count" : 233
              },
              {
                "key" : "Nuclear Explosion",
                "doc_count" : 1
              }
            ]
          }
        },
        ...
```

Nest even further:

```shell
GET earthquakes/_search
{
  "size": 0,
  "aggs": {
    "per_year": {
      "date_histogram": {
        "field": "Date",
        "calendar_interval": "year"
      },
      "aggs": {
        "types": {
          "terms": {
            "field": "Type",
            "size": 10
          },
          "aggs": {
            "max_magniture": {
              "max": {
                "field": "Magnitude"
              }
            }
          }
        }
      }
    }
  }
}
```

### Lab

```shell
GET ecommerce/_search

GET ecommerce/_search
{
  "size": 0,
  "aggs": {
    "orders_per_day": {
      "date_histogram": {
        "field": "order_date",
        "calendar_interval": "day"
      },
      "aggs": {
        "tatal_sales_before_tax_per_day": {
          "sum": {
            "field": "taxless_total_price"
          }
        }
      }
    }
  }
}


GET ecommerce/_search
{
  "size": 0,
  "aggs": {
    "orders_per_day": {
      "date_histogram": {
        "field": "order_date",
        "calendar_interval": "day"
      }
    }
  }
}


GET ecommerce/_search
{
  "size": 0,
  "aggs": {
    "total_lifetime_sales_before_tax": {
      "sum": {
        "field": "taxless_total_price"
      }
    }
  }
}
```

## Pipeline Aggregations


**Parent Pipeline Aggregation** - takes the output of a parent aggregation. Using the output of a parent aggregation, parent pipeline aggregations create new buckets or new values for existing buckets.

**Sibling Pipeline Aggregation** - takes the output of a sibling aggregation. Using the ouput of a sibling aggregation, sibling pipeline aggregations create new outputs at the same elvel as the sibling aggregation.

Parent aggregation:

```shell
GET earthquakes/_search
{
  "size": 0,
  "query": {
    "term": {
      "Type": {
        "value": "Nuclear Explosion"
      }
    }
  },
  "aggs": {
    "per_year": {
      "date_histogram": {
        "field": "Date",
        "calendar_interval": "year"
      },
      "aggs": {
        "max_mag": {
          "max": {
            "field": "Magnitude"
          }
        },
        "change_in_max_magnitude_parent_agg_whatever_name": {
          "derivative": {
            "buckets_path": "max_mag"
          }
        }
      }
    }
  }
}
```

Result:

```shell
"aggregations" : {
    "per_year" : {
      "buckets" : [
        {
          "key_as_string" : "01/01/1966",
          "key" : -126230400000,
          "doc_count" : 1,
          "max_mag" : {
            "value" : 5.62
          }
        },
        {
          "key_as_string" : "01/01/1967",
          "key" : -94694400000,
          "doc_count" : 0,
          "max_mag" : {
            "value" : null
          },
          "change_in_max_magnitude_parent_agg_whatever_name" : {
            "value" : null
          }
        },
        {
          "key_as_string" : "01/01/1968",
          "key" : -63158400000,
          "doc_count" : 2,
          "max_mag" : {
            "value" : 5.63
          },
          "change_in_max_magnitude_parent_agg_whatever_name" : {
            "value" : null
          }
        },
        {
          "key_as_string" : "01/01/1969",
          "key" : -31536000000,
          "doc_count" : 1,
          "max_mag" : {
            "value" : 5.82
          },
          "change_in_max_magnitude_parent_agg_whatever_name" : {
            "value" : 0.1900000000000004
          }
        },
```

Now I want to do a stats calculation across all buckets. Hence I need to do a sibling aggregation:

```shell
GET earthquakes/_search
{
  "size": 0,
  "query": {
    "term": {
      "Type": {
        "value": "Nuclear Explosion"
      }
    }
  },
  "aggs": {
    "per_year": {
      "date_histogram": {
        "field": "Date",
        "calendar_interval": "year"
      },
      "aggs": {
        "max_mag": {
          "max": {
            "field": "Magnitude"
          }
        },
        "change_in_max_magnitude_parent_agg_whatever_name": {
          "derivative": {
            "buckets_path": "max_mag"
          }
        }
      }
    },
    "something_name_stats": {
      "stats_bucket": {
        "buckets_path": "per_year>change_in_max_magnitude_parent_agg_whatever_name"
      }
    }
  }
}
```

Result:

```shell
 "aggregations" : {
    "per_year" : {
      "buckets" : [
        {
          "key_as_string" : "01/01/1966",
          "key" : -126230400000,
          "doc_count" : 1,
          "max_mag" : {
            "value" : 5.62
          }
        },
        {
          "key_as_string" : "01/01/1967",
          "key" : -94694400000,
          "doc_count" : 0,
          "max_mag" : {
            "value" : null
          },
          "change_in_max_magnitude_parent_agg_whatever_name" : {
            "value" : null
          }
        },
        {
          "key_as_string" : "01/01/1968",
          "key" : -63158400000,
          "doc_count" : 2,
          "max_mag" : {
            "value" : 5.63
          },
          "change_in_max_magnitude_parent_agg_whatever_name" : {
            "value" : null
          }
        },
        {
          "key_as_string" : "01/01/1969",
          "key" : -31536000000,
          "doc_count" : 1,
          "max_mag" : {
            "value" : 5.82
          },
          "change_in_max_magnitude_parent_agg_whatever_name" : {
            "value" : 0.1900000000000004
          }
        },
        ...
 
 "something_name_stats" : {
      "count" : 25,
      "min" : -0.7999999999999998,
      "max" : 1.0,
      "avg" : -0.04359999999999999,
      "sum" : -1.0899999999999999
    }
```
































