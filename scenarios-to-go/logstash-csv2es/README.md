# Logstash csv2es scenario

A sample Logstash pipeline to parse a csv file and index its data into a running Elasticsearch cluster.

## The architecture
The *csv2es* environment consists of a `Logstash` server (v`7.10.1`) running the `csv2es_pipeline.conf` pipeline.

## How to run the scenario
The Logstash server in the *csv2es* scenario runs in a Docker container. The Logstash configuration is organised as follows:
- `logstash-conf/config`: Logstash configuration files
- `logstash-conf/data`: the csv file to parse and index
- `logstash-conf/pipelines`: the definition of the Logstash pipeline
- `logstash-conf/templates`: the index template to apply to the output index

Copy-paste the csv data to parse into the `logstash-conf/data/data.csv` file (or specify a different `path` into the *file* input plugin of the `logstash-conf/pipelines/csv2es_pipeline.conf` pipeline). Then, spin up the whole environment by using docker compose.

```
> cd path/to/elastic-training-repo/scenarios-to-go/logstash-csv2es
> docker-compose up
```

To verify that the Logstash pipeline has successfully ingested data into the cluster, search for the `an_index` index in the [Kibana DevTools](http://localhost:5601/app/dev_tools#/console).

```
GET an_index/_search
```