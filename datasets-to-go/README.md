# Datasets-to-go
A collection of datasets, useful for practicing with ElasticSearch. Each dataset might be ingested into Elasticsearch using a different strategy, e.g., via `_bulk` API, LogStash, FileBeat.

To index the offered datasets into your running Elasticsearch cluster, please follow the instructions below.

## Shakespeare's plays
| index | `shakespeare`|
|:--|:---|
| Nr.Documents | 111395 |
| Ingest Strategy | `_bulk` API |

HOW-TO:
1. Download [shakespeare_dataset.json](shakespeare_dataset.json), which will be the body of our `_bulk` request
2. Start ElasticSearch at, e.g., http://elasticsearch:9200
3. Fire the `_bulk` request. Using curl, it'll look something like:

```zsh
curl -XPOST "elasticsearch:9200/shakespeare/_doc/_bulk?pretty" \
-H "Content-Type: application/json" \
--data-binary @path/to/shakespeare_dataset.json
```

## IMDB Title Dataset
| index | `imdb_titles`|
|:--|:---|
| Nr.Documents | 5604662 (3 Feb 2019) |
| Ingest Strategy | Logstash |

HOW-TO:
1. Download Logstash, a data-processing pipeline tool, part of the Elastic Stack
2. Download the title.basics.tsv.gz archive from https://datasets.imdbws.com (or [from here](title.basics.tsv.gz))
3. Extract the archive in, e.g., `path/to/title.basics.tsv`
4. Start ElasticSearch at, e.g., http://elasticsearch:9200
5. Create a configuration file that tells Logstash how to parse the .tsv dataset and how to transform its records into Elasticsearch documents to index. For example, consider the `imdb_titles-logstash.conf` configuration file below:

```
# imdb_titles-logstash.conf
input {
    file {
        path => "path/to/title.basics.tsv"
        start_position => "beginning"
        sincedb_path => "/dev/null"
    }
}

filter {
    csv {
        columns => [ "tconst", "type", "primary_title", "original_title", "is_adult",
                        "year_start", "year_end", "runtime", "genres"]
        separator => "   "
        remove_field => "tconst"
    }
    if [runtime] == "\N" {
        mutate { remove_field => [ "runtime" ] }
    }
    if [year_start] == "\N" {
        mutate { remove_field => [ "year_start" ] }
    }
    if [year_end] == "\N" {
        mutate { remove_field => [ "year_end" ] }
    }
    if [genres] == "\N" {
        mutate { remove_field => [ "genres" ] }
    }
    mutate {
        convert => { "is_adult" => "boolean" }
        split => { "genres" => "," }
        remove_field => ["@version", "path", "host", "message", "tags", "@timestamp"]
    }
}

output {
    stdout {
        codec => rubydebug
    }
    elasticsearch {
        hosts => [ "http://elasticsearch:9200" ]
        index => "imdb_titles"
        document_type => "_doc"
    }
}
```

6. Run Logstash and get the blog posts indexed into Elasticsearch

```bash
sudo $LOGSTASH_PATH/bin/logstash -f path/to/imdb_titles-logstash.conf
```
