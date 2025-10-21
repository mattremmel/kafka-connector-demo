 - Start containers

```sh
docker compose up -d --build
```

- Register connector (delete if already exists)

```sh
# If the connector exists, delete it first:
curl -s -X DELETE http://localhost:8083/connectors/s3-sink
curl -s -X DELETE http://localhost:8083/connectors/s3-glacier-sink

# Re-create with the updated JSON:
curl -s -X POST http://localhost:8083/connectors \
  -H 'Content-Type: application/json' \
  --data @connector-s3/s3-sink.json | jq .
curl -s -X POST http://localhost:8083/connectors \
   -H 'Content-Type: application/json' \
  --data @connector-s3/s3-glacier-sink.json | jq .
```

- Check status of connector

```sh
curl -s http://localhost:8083/connectors/s3-sink/status | jq .
curl -s http://localhost:8083/connectors/s3-glacier-sink/status | jq .
```

- Create Kafka topic (optional if topic auto creation is enabled)

```sh
docker compose exec kafka kafka-topics --create \
  --topic orders --bootstrap-server kafka:9092 --partitions 1 --replication-factor 1 || true
```

- Post messages to Kafka topic

```sh
docker compose exec -T kafka kafka-console-producer \
  --bootstrap-server kafka:9092 --topic orders <<'EOF'
{"orderId":1,"item":"beans","qty":3}
{"orderId":2,"item":"filters","qty":1}
{"orderId":3,"item":"kettle","qty":2}
{"orderId":4,"item":"scale","qty":1}
EOF
```

- Go to [http://localhost:9001](http://localhost:9001) to view data in Minio (S3)
