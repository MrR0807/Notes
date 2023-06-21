```yaml
version: '3'
services:

  elasticsearch:
    container_name: elasticsearch
    image: docker.elastic.co/elasticsearch/elasticsearch:8.1.0
    environment:
      - xpack.security.enabled=false
      - xpack.security.http.ssl.enabled=false
      - xpack.security.transport.ssl.enabled=false
      - discovery.type=single-node
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"

  kibana:
    container_name: kibana
    image: docker.elastic.co/kibana/kibana:8.1.0
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch

volumes:
  elasticsearch-data:
    driver: local
```


By default, every field in a document is indexed (has an inverted index) and thus is searchable.

---

```shell
PUT /megacorp/_doc/1
    {
        "first_name" : "John",
        "last_name" :  "Smith",
        "age" :        25,
        "about" :      "I love to go rock climbing",
        "interests": [ "sports", "music" ]
}
```

```shell
GET megacorp/_doc/1
```

---
To add data to Elasticsearch, we need an index—a place to store related data. In real‐ ity, an index is just a logical namespace that points to one or more physical shards.

A shard is a low-level worker unit that holds just a slice of all the data in the index. A shard is a single instance of Lucene, and is a complete search engine in its own right. Our documents are stored and indexed in shards, but our applications don’t talk to them directly. Instead, they talk to an index.
