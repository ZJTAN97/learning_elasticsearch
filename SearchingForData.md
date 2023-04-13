# Introduction to searching

-   use Query DSL
-   Search queries are defined as JSON within the request body
-   More verbose, supports all features

```
# Basic search query

GET /products/_search
{
    "query": {
        "match_all": {}
    }
}

```

<br>

# Term Level Queries

-   used to search structured data for exact values (filtering)

    -   e.g. finding products where the brand equals "Nike"

-   Term level queries are not analyzed
    -   search value is used exactly as is for inverted index lookups
