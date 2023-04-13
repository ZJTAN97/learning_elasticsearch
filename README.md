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

# Optimistic Concurrency Control

-   Prevents overwriting documents inadvertently due to concurrent operations
-   makes use of `_seq_no` and `_primary_term`

```
POST /products/_update/100?if_primary_term=1&if_seq_no=8
{
  "doc": {
    "in_stock": 123
  }
}
```

<br>

# Update by query

-   update multiple documents within a single query
-   this uses Primary Terms, Sequence numbers and Optimistic Concurrency control
-   Search queries and bulk requests are sent to replication groups sequentially
    -   Elasticsearch retries these queries up to ten times
    -   if queries still fail, whole query is aborted
        -   any changes made to documents will not be rolled back
-   API returns information about failures

```

POST /products/_update_by_query
{
  "script": {
    "source": "ctx._source.in_stock--"
  },
  "query": {
    "match_all": {}
  }
}

```

<br>

# Delete by query

-   delete multiple documents within a singe query

```
POST /products/_delete_by_query
{
  "query": {
    "match_all": {}
  }
}
```

<br>

# Batch Processing

-   Bulk API
-   Content-Type: application/x-ndjson
-   application/json is accepted, but not correct way
-   Each line must end with a newline character (\n or \r\n)
-   a failed action will not affect other actions

```
POST /_bulk
{ "index": { "_index": "products", "_id": 200 } }
{ "name": "Espresso Machine", "price": "199", "in_stock": 5 }
{ "create": { "_index": "products", "_id": 201 } }
{ "name": "Espresso Machine 2", "price": "199", "in_stock": 10 }


GET /products/_search
{
  "query": {
    "match_all": {}
  }
}


POST /products/_bulk
{ "update": { "_id": 201 } }
{ "doc": { "price": 129 } }
{ "delete": { "_id": 200 } }

```

<br>

# Importing data with cURL

```
curl -H "Content-Type: application/x-ndjson" -XPOST http://localhost:9200/products/_bulk --data-binary "@products-bulk.json"
```

<br>

# Analysis

-   applicable to text fields/values
-   text values are analyzed when indexing documents
-   the result is stored in data structures that are efficient for searching etc.
-   Standard Analyzer Process: Character filters --> Tokenizer --> Token Filters

<br>

# Using Analyze API

```
POST /_analyze
{
  "text": "2 guys walk into   a bar, but the third... DUCKS! :)",
  "analyzer": "standard"
}

```

<br>

# Understanding Inverted Indices

-   Mapping between terms (tokenizer) and which documents contain them
-   Map the documents to the terms (tokenized words)
-   Terms are sorted alphabetically
-   every text field will consist of inverted indices (one inverted index per text field)
-   other data types use BKD trees
-   An inverted index is a mapping between terms and which documents contain them
-   Each field has a dedicated inverted index
-   inverted indices enable fast searches

<br>

# Introduction to mapping

-   Defines the structure of documents
    -   used to configure how values are indexed
-   Similar to a table's schema in a relational database
-   Explicit mapping
    -   define field mapping ourselves
-   Dynamic mapping
    -   Elasticsearch generates field mappings for us

<br>

# Overview of data types

-   `object` data type

    -   used for any JSON object
    -   `objects` may be nested
    -   mapped using the `properties` parameter

-   `nested` data type

    -   similar to `object` data type, but maintains object relationships
        -   useful when indexing array of objects
    -   enables us to query objects independently
        -   must use the `nested` query
    -   `nested` objects are stored as hidden documents

-   `keyword` data type
    -   used for exact matching of values
    -   typically used for filtering, aggregations and sorting
    -   e.g. searching for articles with a status of PUBLISHED
    -   for full-text searches, use the `test` data type instead
        -   e.g. searching the body text of an artile

<br>

# How keyword datatype works

-   keyword fields are analyzed with the keyword analyzer
-   keyword analyzer is a no-op analyzer
    -   it outputs the unmodified string as a single token
    -   this token is then placed into the inverted index
-   keyword fields are used for exact matching, aggregations and sorting

<br>

# Understanding type coercion

-   Data types are inspected when indexing documents
    -   they are validated, and some invalid values are rejected
    -   e.g. trying to index an object for a text field

<br>

# Introduction to arrays

-   There is no such thing as an `array` data type
-   any field may contain zero or more values
    -   No configuration or mapping needed
    -   simply supply an array when indexing a document
-   array values should be the same data type
-   Coercion only works for fields that are already mapped
-   Arrays may contain nested arrays
-   Arrays are flattened during indexing [1, [2, 3]] --> [1, 2, 3]
-   Remember to use the `nested` data type for arrays of objects if you need to query the objects independently.

```
// can use analyze api to see that they are simply merged together

POST /_analyze
{
  "text": ["strings are simply", "merged together."],
  "analyzer": "standard"
}

```

<br>

# Adding explicit mappings

Example

```
PUT /reviews
{
  "mappings": {
    "properties": {
      "rating": { "type":  "float" },
      "content": { "type":  "text" },
      "product_id": { "type":  "integer" },
      "author": {
        "properties": {
          "first_name": { "type":  "text" },
          "last_name": { "type":  "text" },
          "email": { "type":  "keyword" }
        } }
    }
  }
}

PUT /reviews (with dot notation)
{
  "mappings": {
    "properties": {
      "rating": { "type":  "float" },
      "content": { "type":  "text" },
      "product_id": { "type":  "integer" },
      "author.email": { "type":  "keyword" },
      "author.first_name": { "type":  "text" },
      "author.last_name": { "type":  "text" },
      }
    }
}

```

Adding to existing indices

```
// can just add like so

PUT /reviews/_mapping
{
  "properties": {
    "created_at": {
      "type": "date"
    }
  }
}

```

<br>

# Retrieve Mappings

```
// retrieve for entire object
GET /reviews/_mapping

// retrieve for particular field
GET /reviews/_mapping/field/content

// retrieve for particular field (nested)
GET /reviews/_mapping/field/author.email
```

<br>

# `date` datatype

-   Specified in one of three ways
    1. Specially formatted strings
    2. Milliseconds since the epoch (long)
    3. Seconds since the epoch (integer)
-   Epoch refers to the 1st of january 1970
-   custom formats are supported

Default Behavior of `date` fields

-   Three supported formats
    -   date without time
    -   date with time
    -   Milliseconds since the epoch (long)
-   UTC timezone assumed if none is specified
-   Dates must be formatted according to the ISO 8601 specification

How `date` field are stored

-   stored internally as milliseconds since the epoch (long)
-   dates are converted to the UTC timezone
-   Do not provide UNIX timestamps for default `date` fields

<br>

# How missing fields are handled

-   All fields in Elasticsearch are optional
-   Can leave out a field when indexing documents
-   Some integrity checks need to be done at the application level (e.g. having required fields)
-   searching automatically handle missing fields

<br>

# Overview of mapping parameters

`format` parameter

-   used to customize the format for `date` fields ("strict_date_optional_time||epoch_millis")
-   "dd/MM/yyyy"

`properties` parameter (you already used it plenty of times)

-   Define nested fields for object and nested fields

`coerce` parameter

-   used to enable or disable coercion of values (enabled by default)

`norms` parameter

-   Normalization factors used for relevance scoring
-   often we don't want to just filter results, but also rank them
-   norms can be disabled to save disk space
    -   useful for fields that will not be used for relevance scoring

`index` parameter

-   disables indexing for a field
-   values are stil stored within \_source
-   useful if you will not use field for search queries
-   save disk space and slightly improves indexing throughput
-   often used for time series data
-   Fields with indexing disabled can still be used for aggregations

`null_value` parameter

-   NULL values cannot be indexed or searched
-   use this parameter to replace NULL values with another value
-   only works for explicit NULL values
-   the replacement value must be of the same data type as the field

`copy_to` parameter

-   used to copy multiple field values into a "group field"
-   simply specify the name of the target field as the value
-   e.g. first_name and last_name --> full_name

<br>

# Updating existing mappings

-   Generally ElasticSearch field mappings cannot be changed
-   Can add new field mappings but thats about it
-   A few mapping parameters can be updated for existing mappings

<br>
