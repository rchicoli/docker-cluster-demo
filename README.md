# Docker Microservices

## Docker Compose

```bash
docker-compose -f networks.yml -f elasticsearch.yml -f logstash.yml -f app.yml up
```

## Manual Set Up

1. Create one node of Elasticsearch service?

```bash
# create and run elasticsearch docker container
docker run -d --name elasticsearch elasticsearch:5-alpine elasticsearch -E network.host=0.0.0.0
```

2. Configure Logstash input and output?

```bash
# create and run logstash docker container
docker run -d --name logstash --link elasticsearch:elasticsearch -v $PWD/logstash/config:/etc/logstash/conf.d logstash:5-alpine logstash -f /etc/logstash/conf.d/logstash.conf
```

3. Deploy any application to test the logging process

```bash
# find out the logstash's IP address
LOGSTASH_IP="$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' logstash)"

# create and run a docker webapplication
docker run --name webapper --log-driver gelf --log-opt gelf-address="udp://${LOGSTASH_IP}:5002" rchicoli/webapper
```

4. Check if the logs are going to Elasticsearch

```bash
# find out the webapplication's IP address
APP_IP="$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' webapper)"

# query webapplication's container log
curl "http://${APP_IP}:8080/hostname"

# find out the elasticsearch's IP address
ELASTICSEARCH_IP="$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' elasticsearch)"

# query elasticsearch
curl "http://${ELASTICSEARCH_IP}:9200/logstash-$(date +%Y.%m.%d)/_search" | jq .
```