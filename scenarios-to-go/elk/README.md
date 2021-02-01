# ELK scenario

A sample environment to showcase integration and interplay between Elasticsearch, Logstash and Kibana (ELK stack).

## The architecture
The *ELK* environment consists of:
- A one-node `Elasticsearch` cluster.
- A single instance of `Kibana` connected to the Elasticsearch cluster.
- A `Logstash` server running a sample data pipeline that indexes random documents into the Elasticsearch cluster.

```
                +--------------+          +-------------------+
           :5601|              |     :9200|                   |
USER +---------->    Kibana    <---------->   Elasticsearch   |
                |              |:5601     |                   |
                +--------------+          +---------^---------+
                                                    |:9200
                                                    |
                                          +---------+---------+
                                          |                   |
                                          |     Logstash      |
                                          |                   |
                                          +-------------------+
```

Elastic components version: `7.10.1`.

## How to run the scenario
All components of the *ELK* scenario run in Docker containers, in the same virtual Docker network. Spin up the whole environment using docker compose.

```
> cd path/to/elastic-training-repo/scenarios-to-go/elk
> docker-compose up
```

To verify that the Logstash pipeline has successfully ingested data into the cluster, search for the `sample` index in the [Kibana DevTools](http://localhost:5601/app/dev_tools#/console).

```
GET sample/_search
```