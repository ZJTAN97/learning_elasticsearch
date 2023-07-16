# Search for information

There are two main ways to search

1. Queries
2. Aggregations

# Queries

Queries retrieve documents that match the criteria.

```
GET enter_name_of_index/_search

// Example
GET questions/_search

```

To improve the response speed on large datasets, ES limits count to 10,000 by default. If you want exact total number of hits, use the following query

```
GET enter_name_of_index/_search
{
    "track_total_hits": true
}

// Example
GET questions/_search
{
    "track_total_hits": true
}

```

To search for data within specific time range

```
GET enter_name_of_the_index_here/_search
{
  "query": {
    "Specify the type of query here": {
      "Enter name of the field here": {
        "gte": "Enter lowest value of the range here",
        "lte": "Enter highest value of the range here"
      }
    }
  }
}

// Example
GET news_headlines/_search
{
  "query": {
    "range": {
      "date": {
        "gte": "2015-06-20",
        "lte": "2015-09-22"
      }
    }
  }
}

```

# Aggregations

An aggregation summarizes your data as metrics, statistics and other analytics

Syntax

```

GET enter_name_of_the_index_here/_search
{
  "aggs": {
    "name your aggregation here": {
      "specify aggregation type here": {
        "field": "name the field you want to aggregate here",
        "size": state how many buckets you want returned here
      }
    }
  }
}

// Example
GET news_headlines/_search
{
  "aggs": {
    "by_category": {
      "terms": {
        "field": "category",
        "size": 100
      }
    }
  }
}

```

# Combination of query and aggregation request

Search for the most significant term in a category

Syntax

```
GET enter_name_of_the_index_here/_search
{
  "query": {
    "match": {
      "Enter the name of the field": "Enter the value you are looking for"
    }
  },
  "aggregations": {
    "Name your aggregation here": {
      "significant_text": {
        "field": "Enter the name of the field you are searching for"
      }
    }
  }
}

// Example
GET news_headlines/_search
{
  "query": {
    "match": {
      "category": "ENTERTAINMENT"
    }
  },
  "aggregations": {
    "popular_in_entertainment": {
      "significant_text": {
        "field": "headline"
      }
    }
  }
}

```

# Precision and Recall

Increasing Recall (more documents but less precise)

Syntax

```

GET enter_name_of_index_here/_search
{
  "query": {
    "match": {
      "Specify the field you want to search": {
        "query": "Enter search terms"
      }
    }
  }
}

// Example
GET news_headlines/_search
{
  "query": {
    "match": {
      "headline": {
        "query": "Khloe Kardashian Kendall Jenner"
      }
    }
  }
}

```

By default the match query uses an "OR" logic. "OR" logic results in higher number of hits, thereby increasing recall. However, the hits are loosely related to the query and lowering precision as a result.

Increasing Precision

Syntax

```

GET enter_name_of_index_here/_search
{
  "query": {
    "match": {
      "Specify the field you want to search": {
        "query": "Enter search terms",
        "operator": "and"
      }
    }
  }
}

// Example
GET news_headlines/_search
{
  "query": {
    "match": {
      "headline": {
        "query": "Khloe Kardashian Kendall Jenner",
        "operator": "and"
      }
    }
  }
}

```

We can increase precision by adding an "and" operator to the query. "AND" operator will result in getting more precise matches, thereby increasing precision. However, it will reduce the number of hits returned, resulting in lower recall.

We can use `minimum_should_match` to specify minimum number of terms a document should have to be included in the search result. This gives you more control over fine tuning precision and recall of your search

Syntax

```

GET enter_name_of_index_here/_search
{
  "query": {
    "match": {
      "headline": {
        "query": "Enter search term here",
        "minimum_should_match": Enter a number here
      }
    }
  }
}

// Example
GET news_headlines/_search
{
  "query": {
    "match": {
      "headline": {
        "query": "Khloe Kardashian Kendall Jenner",
        "minimum_should_match": 3
      }
    }
  }
}


```
