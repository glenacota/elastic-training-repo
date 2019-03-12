# Clusters-to-go
Below is a collection of docker-compose files to spin up simple Elastic clusters, useful for practicing with ElasticSearch.
For more details on how to setup Elasticsearch with docker, please refer to the [Elastic documentation](https://www.elastic.co/guide/en/elasticsearch/reference/6.6/docker.html).

## 1 ES node, 1 Kibana instance
(v6.6.1)

```yml
version: '3'
services:
  esnode:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.6.1
    container_name: esnode
    environment:
      - node.name=esnode
      - cluster.name=elastic-cluster
      - discovery.type=single-node
      - bootstrap.memory_lock=true
      - ES_JAVA_OPTS=-Xms512m -Xmx512m
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esnode-data:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - elastic-net
  kibana:
    image: docker.elastic.co/kibana/kibana:6.6.1
    container_name: kibana
    environment:
      - elasticsearch.url=http://elasticsearch:9200
    ulimits:
      memlock:
        soft: -1
        hard: -1
    ports:
      - 5601:5601
    networks:
      - elastic-net
    links:
      - esnode:elasticsearch

volumes:
  esnode-data:
    driver: local

networks:
  elastic-net:
```


## 1 ES dedicated master node, 1 ES data node, 1 Kibana instance
(v6.6.1)

```yml
version: '3'
services:
  master:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.6.1
    container_name: master
    environment:
      - cluster.name=elastic-cluster
      - node.name=master
      - node.master=true
      - node.data=false
      - node.ingest=false
      - bootstrap.memory_lock=true
      - ES_JAVA_OPTS=-Xms512m -Xmx512m
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - master-data:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
      - 9300:9300
    networks:
      - elastic-net
  datanode:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.6.1
    container_name: datanode
    environment:
      - cluster.name=elastic-cluster
      - node.name=datanode
      - node.master=false
      - node.data=true
      - bootstrap.memory_lock=true
      - discovery.zen.ping.unicast.hosts=master
      - ES_JAVA_OPTS=-Xms512m -Xmx512m
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - datanode-data:/usr/share/elasticsearch/data
    networks:
      - elastic-net
  kibana:
    image: docker.elastic.co/kibana/kibana:6.6.1
    container_name: kibana
    environment:
      - elasticsearch.url=http://elasticsearch:9200
    ulimits:
      memlock:
        soft: -1
        hard: -1
    ports:
      - 5601:5601
    networks:
      - elastic-net
    links:
      - datanode:elasticsearch

volumes:
  master-data:
    driver: local
  datanode-data:
    driver: local

networks:
  elastic-net:
```


## 2 clusters
The first cluster has 1 ES node and a Kibana instance, whilst the second cluster has only one ES node.
(v6.6.1)

```yml
version: '3'
services:
  esnode-earth:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.6.1
    container_name: esnode-earth
    environment:
      - cluster.name=elastic-cluster-earth
      - node.name=esnode-earth
      - bootstrap.memory_lock=true
      - ES_JAVA_OPTS=-Xms512m -Xmx512m
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esnode-earth-data:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - elastic-net
  kibana:
    image: docker.elastic.co/kibana/kibana:6.6.1
    container_name: kibana
    environment:
      - elasticsearch.url=http://elasticsearch:9300
    ulimits:
      memlock:
        soft: -1
        hard: -1
    ports:
      - 5601:5601
    networks:
      - elastic-net
    links:
      - esnode-earth:elasticsearch

  esnode-mars:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.6.1
    container_name: esnode-mars
    environment:
      - cluster.name=elastic-cluster-mars
      - node.name=esnode-mars
      - bootstrap.memory_lock=true
      - ES_JAVA_OPTS=-Xms512m -Xmx512m
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esnode-mars-data:/usr/share/elasticsearch/data
    ports:
      - 9201:9200
      - 9301:9300
    networks:
      - elastic-net

volumes:
  esnode-earth-data:
    driver: local
  esnode-mars-data:
    driver: local

networks:
  elastic-net:
```


## 1 ES dedicated master node, 2 ES data nodes, 1 Kibana instance
The two data nodes support a hot/warm architecture.
(v6.6.1)

```yml
version: '3'
services:
  master:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.6.1
    container_name: master
    environment:
      - cluster.name=elastic-cluster
      - node.name=master
      - node.master=true
      - node.data=false
      - node.ingest=false
      - bootstrap.memory_lock=true
      - xpack.license.self_generated.type=basic
      - ES_JAVA_OPTS=-Xms512m -Xmx512m
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - master-data:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - elastic-net
  datanode-europe:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.6.1
    container_name: datanode-europe
    environment:
      - cluster.name=elastic-cluster
      - node.name=datanode-europe
      - node.master=false
      - node.data=true
      - node.attr.my_zone=europe
      - bootstrap.memory_lock=true
      - discovery.zen.ping.unicast.hosts=master,datanode-asia
      - xpack.license.self_generated.type=basic
      - ES_JAVA_OPTS=-Xms512m -Xmx512m
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - datanode-europe-data:/usr/share/elasticsearch/data
    networks:
      - elastic-net
  datanode-asia:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.6.1
    container_name: datanode-asia
    environment:
      - cluster.name=elastic-cluster
      - node.name=datanode-asia
      - node.master=false
      - node.data=true
      - node.attr.my_zone=asia
      - bootstrap.memory_lock=true
      - discovery.zen.ping.unicast.hosts=master,datanode-europe
      - xpack.license.self_generated.type=basic
      - ES_JAVA_OPTS=-Xms512m -Xmx512m
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - datanode-asia-data:/usr/share/elasticsearch/data
    networks:
      - elastic-net
  kibana:
    image: docker.elastic.co/kibana/kibana:6.6.1
    container_name: kibana
    environment:
      - elasticsearch.url=http://elasticsearch:9200
    ulimits:
      memlock:
        soft: -1
        hard: -1
    ports:
      - 5601:5601
    networks:
      - elastic-net
    links:
      - datanode-europe:elasticsearch

volumes:
  master-data:
    driver: local
  datanode-europe-data:
    driver: local
  datanode-asia-data:
    driver: local

networks:
  elastic-net:
```


## 1 ES dedicated master node, 2 ES data nodes, 1 Kibana instance
The two data nodes support allocation awareness based on attribute's values.
(v6.6.1)

```yml
version: '3'
services:
  master:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.6.1
    container_name: master
    environment:
      - cluster.name=elastic-cluster
      - node.name=master
      - node.master=true
      - node.data=false
      - node.ingest=false
      - bootstrap.memory_lock=true
      - xpack.license.self_generated.type=basic
      - ES_JAVA_OPTS=-Xms512m -Xmx512m
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - master-data:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - elastic-net
  datanode-europe:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.6.1
    container_name: datanode-europe
    environment:
      - cluster.name=elastic-cluster
      - node.name=datanode-europe
      - node.master=false
      - node.data=true
      - node.attr.my_zone=europe
      - bootstrap.memory_lock=true
      - discovery.zen.ping.unicast.hosts=master,datanode-asia
      - xpack.license.self_generated.type=basic
      - ES_JAVA_OPTS=-Xms512m -Xmx512m
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - datanode-europe-data:/usr/share/elasticsearch/data
    networks:
      - elastic-net
  datanode-asia:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.6.1
    container_name: datanode-asia
    environment:
      - cluster.name=elastic-cluster
      - node.name=datanode-asia
      - node.master=false
      - node.data=true
      - node.attr.my_zone=asia
      - bootstrap.memory_lock=true
      - discovery.zen.ping.unicast.hosts=master,datanode-europe
      - xpack.license.self_generated.type=basic
      - ES_JAVA_OPTS=-Xms512m -Xmx512m
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - datanode-asia-data:/usr/share/elasticsearch/data
    networks:
      - elastic-net
  kibana:
    image: docker.elastic.co/kibana/kibana:6.6.1
    container_name: kibana
    environment:
      - elasticsearch.url=http://elasticsearch:9200
    ulimits:
      memlock:
        soft: -1
        hard: -1
    ports:
      - 5601:5601
    networks:
      - elastic-net
    links:
      - datanode-europe:elasticsearch

volumes:
  master-data:
    driver: local
  datanode-europe-data:
    driver: local
  datanode-asia-data:
    driver: local

networks:
  elastic-net:
```
