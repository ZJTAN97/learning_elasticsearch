# Setup

```

# Run an elasticsearch container

docker run \
      --name elasticsearch \
      --net elastic \
      -d \
      -p 9200:9200 \
      -e discovery.type=single-node \
      -e ES_JAVA_OPTS="-Xms1g -Xmx1g"\
      -e xpack.security.enabled=false \
      -it \
      docker.elastic.co/elasticsearch/elasticsearch:8.6.2

# Run Kibana container

docker run \
    --name kibana \
    --net elastic \
    -d \
    -p 5601:5601 \
    docker.elastic.co/kibana/kibana:8.6.2

```

<br>

# Intro to Elastic Search and the Elastic Stack

-   Data is stored as documents (a unit of information , corressponds to row in relational dbs)
-   Document's data is seperated into fields (Similar to columns in relational dbs)
-   Querying Elasticsearch with REST API.
-   ELK stack (ElasticSearch, Logstash, Kibana)
-   Logstash --> An event processing pipeline
-   Kibana --> An analytics and visualization platform
-   Beats --> A collection of data shippers that send data to Elasticsearch or Logstash
-   X-Pack --> A collection of features such as security, monitoring, alerting, reporting, etc.

<br>

# Common architectures of Elastic Search

How does the data get into Elasticsearch?

-   Data will be stored both in the DB and Elasticsearch (sort of duplicated)

<br>

# Creating and Deleting Indices

```

PUT /products
{
    "settings": {
        "number_of_shards": 2,
        "number_of_replicas": 2
    }
}

DELETE /products

```

<br>

# Indexing Documents

```
POST /products/_doc
{
  "name": "Coffee maker",
  "price": 64,
  "in_stock": 10
}

PUT /products/_doc/<custom_id>
{
  "name": "Coffee maker",
  "price": 64,
  "in_stock": 10
}

```

-   `action.auto_create_index`: if enabled, indices will automatically be created when adding documents

<br>

# Retrieving Documents by ID

```
GET /products/_doc/<id>
```

<br>

# Updating documents

-   ElasticSearch Documents are immutable!
-   we actually replaced documents
-   The `update` API makes it look like we updated the documents
    1. Current document is retrieved
    2. field values are changed
    3. existing document is replaced with the modified document

```
PUT /products/_doc/100
{
  "doc": {
    "in_stock": 3
    "tags": ["electronics"]
  }
}
```

<br>

# Scripted Updates

```
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock--"
  }
}


// with params

POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock -= params.quantity",
    "params": {
      "quantity": 4
    }
  }
}


// noop example

POST /products/_update/100
{
  "script": {
    "source": """
      if(ctx._source.in_stock == 0) {
        ctx.op = 'noop';
      }

      ctx._source.in_stock--;
    """
  }
}

```

<br>

# Upserts

-   document already exist, update else create new index

```
POST /products/_update/101
{
  "script": {
    "source": "ctx._source.in_stock++"
  },
  "upsert": {
    "name": "Docker",
    "price": 399,
    "in_stock": 5
  }
}

```

<br>

# Replace documents

```
PUT /products/_doc/101
{
  "name": "Docker 2",
  "price": 20,
  "in_stock": 10
}

```

<br>

# Deleting documents

```
DELETE /products/_doc/101

```

<br>

# Understanding Routing

-   Routing is the process of resolving a shard for a document
-   Formula is used when indexing, retrieving and updating documents
-   Routing can be customized
-   default routing strategy distributes documents evenly
-   One of the reason why an index shards cannot be changed, is that the routing formula would then yield different results

<br>

# Understand Document Versioning

-   Not a revision history of documents, only store latest
-   Elasticsearch stores an `_version` metadata field with every document
    -   value is an integer
    -   incremented by one when modifying a document
    -   the value is retained for 60 seconds when deleting a document
        -   configured with the `index.gc_deletes` setting
    -   the `_version` field is returned when retrieving documents
-   Default type is called internal versioning
-   there is also external versioning type
-   versioning shows how many times a document has been modified
-   used to do optimistic concurrency control (previously)

<br>

