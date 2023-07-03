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
To add data to Elasticsearch, we need an index — a place to store related data. In reality, an index is just a logical namespace that points to one or more physical shards.

A shard is a low-level worker unit that holds just a slice of all the data in the index. A shard is a single instance of Lucene, and is a complete search engine in its own right. Our documents are stored and indexed in shards, but our applications don’t talk to them directly. Instead, they talk to an index.

The number of primary shards in an index is fixed at the time that an index is cre‐ ated, but the number of replica shards can be changed at any time.

To scale reads, we can increase node replica count. However, writes cannot be scaled beyond initial primary shard count.

---

Even though the document doesn’t exist (found is false), the _version number has still been incremented. This is part of the internal bookkeeping, which ensures that changes are applied in the correct order across multiple nodes.

---

When updating a document with the index API, we read the original document, make our changes, and then reindex the whole document in one go. The most recent indexing request wins: whichever document was indexed last is the one stored in Elasticsearch. If somebody else had changed the document in the meantime, their changes would be lost.
