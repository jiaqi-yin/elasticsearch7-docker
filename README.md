To start a single-node elastic cluster for development or testing
```
docker-compose -f docker-compose-single-node.yml up
```

To bring up the three-node Elasticsearch cluster and Kibana (Make sure Docker Engine is allotted at least 4GiB of memory)
```
docker-compose -f docker-compose-multi-node.yml up
```

To see if the nodes are up and running
```
curl -X GET "localhost:9200/_cat/nodes?v&pretty"
```

To delete the data volumes when you bring down the cluster, specify the -v option: docker-compose down -v.
