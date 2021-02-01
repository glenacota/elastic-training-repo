# RDBMS Sync scenario

A sample environment to showcase data synchronization between a relational database to an Elasticsearch cluster.

## The architecture
The *RDBMS Sync* environment consists of:
- A `MySQL server` hosting the `carsretailing` database (see below)
- A one-node `Elasticsearch` cluster.
- A single instance of `Kibana` connected to the Elasticsearch cluster.
- A `Logstash` server running the data migration pipeline that (i) queries the MySQL server every 5 minutes, (2) denormalises and enriches the query results, (3) indexes the processed queries into the Elasticsearch cluster (either creating a new document or updating an existing one).

```
                +--------------+          +-------------------+
           :5601|              |     :9200|                   |
USER+----------->    KIBANA    <---------->   ELASTICSEARCH   |
                |              |:5601     |                   |
                +--------------+          +---------^---------+
                                                    |:9200
                                                    |
                                          +---------+---------+       +--------------+
                                          |                   |  :3306|              |
                                          |     LOGSTASH      +------->   MYSQL DB   |
                                          |                   |       |              |
                                          +-------------------+       +--------------+
```

Elastic components version: `7.10.1`. MySQL version: `8.x`.

The `carsretailing` database is based on [this sample database](https://www.mysqltutorial.org/mysql-sample-database.aspx) created by mysqltutorial.org. The schema is displayed below:
![](https://sp.mysqltutorial.org/wp-content/uploads/2009/12/MySQL-Sample-Database-Schema.png)

## How to run the scenario
All components of the *RDBMS Sync* scenario run in Docker containers, in the same virtual Docker network. Spin up the whole environment using docker compose.

```
> cd path/to/elastic-training-repo/scenarios-to-go/rdbms-sync
> docker-compose up
```

To verify that the Logstash pipeline has successfully ingested data into the cluster, search for the `products`, `customers`, and `orderdetails` indices in the [Kibana DevTools](http://localhost:5601/app/dev_tools#/console).

```
GET products/_search
GET customers/_search
GET orderdetails/_search
```

To verify that the synchronization is effective, add a new record or edit an existing one into the MySQL `carsretailing` database, and verify that a corresponding document into Elasticseach has been created or updates, respectively.