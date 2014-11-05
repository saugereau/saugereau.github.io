---
layout: post
title: Solr partial update
category: Architecture
tags: solr search
summary: When sometimes you only want to partially update your data
---

Solr supports several modifiers that atomically update values of a document.

- set – set or replace a particular value, or remove the value if null is specified as the new value
- add – adds an additional value to a list
- remove – removes a value (or a list of values) from a list
- inc – increments a numeric value by a specific amount (use a negative value to decrement)
Note: All original source fields must be stored for field modifiers to work correctly.  This is the default in Solr.

## Via updateHandler :

```
$ curl http://localhost:8080/solr/update -H 'Content-type:application/json' -d '
[
 {"id"       : "311237",
  "author"   : {"set":"Charles Perrault"},
  "sold" : {"inc":3},
  "category"      : {"add":"Contes"}
 }
]'
```

## Via SolrJ (Java Api)

```java
SolrInputDocument sdoc = new SolrInputDocument();
//the uniquekey to determine which document to delete
sdoc.addField("id","311237");
//the author
sdoc.addField("author", Collections.singletonMap("set","Charles Perrault");
// send it to the solr server
client.add(sdoc);
```

During some development, I already had this error on partial update :

```java
org.apache.solr.client.solrj.SolrServerException: java.lang.NumberFormatException: For input string: "true"
```

The reason is that I first initialized a document defining an integer fieldtype with a boolean value (yes, it's seems to be possible without exception),
I literaly had the "true" value on my document for this field.
So on update (on other field of this same document), Solr probably check this field type before do the commit.