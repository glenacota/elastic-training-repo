# Logstash es2csv scenario

A sample Logstash pipeline to export all the documents of an Elasticsearch index as a csv file.

## The architecture
The *es2csv* environment consists of a `Logstash` server (v`7.10.1`) running the `es2csv_pipeline.conf` pipeline.

## How to run the scenario
The Logstash server in the *es2csv* scenario runs in a Docker container. The Logstash configuration is organised as follows:
- `logstash-conf/config`: Logstash configuration files
- `logstash-conf/data`: the empty csv file where to export the Elasticsearch documents
- `logstash-conf/pipelines`: the definition of the Logstash pipeline

Edit the configuration of the elasticsearch *input* plugin of the `logstash-conf/pipelines/es2csv_pipeline.conf` pipeline, specifying the cluster coordinates as well as the name of the index to export. Then, edit the `fields` property of the *csv* output plugin with the list of document fields to export. Finally, spin up the whole environment by using docker compose.

```
> cd path/to/elastic-training-repo/scenarios-to-go/logstash-es2csv
> docker-compose up
```

To verify that the Logstash pipeline has successfully exported data from the cluster, check out the content of the `logstash-conf/data/data.csv` file.