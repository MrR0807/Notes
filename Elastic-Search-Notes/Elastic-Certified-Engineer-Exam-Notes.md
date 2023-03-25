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





































