By default, every field in a document is indexed (has an inverted index) and thus is searchable.

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
