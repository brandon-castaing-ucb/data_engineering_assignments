## Change directory to assignment 6 and output docker compose .yml
  549  cd assignment-06-brandon-castaing-ucb/
  550  ls
  551  cat docker-compose.yml

## docker compose as daemon process and check 3 containers are created
  552  docker compose up -d
  554  docker ps -a

## Create foo topic in Kafka
  555  docker-compose exec kafka kafka-topics --create --topic foo --partitions 1 --replication-factor 1 --if-not-exists --zookeeper localhost:32181

## Publish messages from assessment data set to Kafka foo topic
  560  docker-compose exec mids bash -c "cat /w205/assessment-attempts-20180128-121051-nested.json | jq '.[]' -c | kafkacat -P -b kafka:29092 -t foo && echo 'Produced 100 messages.'"

## Consume messages from Kafka foo topic
  561  docker-compose exec mids bash -c "kafkacat -C -b kafka:29092 -t foo -o beginning -e"

## Consume messages and count lines
  567  docker-compose exec mids bash -c "kafkacat -C -b kafka:29092 -t foo -o beginning -e" | wc -l

## Tear down cluster and create history
  568  docker-compose down
  569  history > bcastaing-history.txt
