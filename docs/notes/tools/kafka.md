## kafka
### Docker安装kafka
```shell
docker search zookeeper
docker pull  zookeeper
docker run -d -p 2181:2181 --name zookeeper zookeeper

docker pull wurstmeister/kafka
docker run -d --name kafka -p 9092:9092 -e KAFKA_BROKER_ID=1 -e KAFKA_ZOOKEEPER_CONNECT=127.0.0.1:2181 -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://127.0.0.1:9092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 -v /etc/localtime:/etc/localtime -t wurstmeister/kafka:latest
```

### kafka-map
```shell
docker pull dushixiang/kafka-map
docker run -d --name kafka-map -p 8049:8080 -e DEFAULT_USERNAME=admin -e DEFAULT_PASSWORD=admin  dushixiang/kafka-map:latest
```
