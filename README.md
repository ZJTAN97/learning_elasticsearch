## Intro to Elastic Search and the Elastic Stack
- Data is stored as documents (a unit of information , corressponds to row in relational dbs)
- Document's data is seperated into fields (Similar to columns in relational dbs)
- Querying Elasticsearch with REST API.
- ELK stack (ElasticSearch, Logstash, Kibana)
- Logstash --> An event processing pipeline
- Kibana --> An analytics and visualization platform
- Beats --> A collection of data shippers that send data to Elasticsearch or Logstash
- X-Pack --> A collection of features such as security, monitoring, alerting, reporting, etc.

<br>
<hr>
<br>


## Common architectures of Elastic Search

How does the data get into Elasticsearch?
- Data will be stored both in the DB and Elasticsearch (sort of duplicated)
