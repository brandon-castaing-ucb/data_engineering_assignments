## Download assignment dataset
  789  curl -L -o assessment-attempts-20180128-121051-nested.json https://goo.gl/ME6hjp
  
## Start docker cluster
  800  docker-compose up -d

## Create Kafka topic for commits
  801  docker-compose exec kafka kafka-topics --create --topic commits --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181

## Publish dataset JSON records to commits Kafka topic
  802  docker-compose exec mids bash -c "cat /w205/assessment-attempts-20180128-121051-nested.json | jq '.[]' -c | kafkacat -P -b kafka:29092 -t commits"

## Start PySpark
  804  docker-compose exec spark pyspark

## Commands executed in PySpark
raw_commits = spark.read.format("kafka").option("kafka.bootstrap.servers", "kafka:29092").option("subscribe","commits").option("startingOffsets", "earliest").option("endingOffsets", "latest").load() 
raw_commits.cache()

import json
extracted_commits = commits.rdd.map(lambda x: json.loads(x.value)).toDF()
extracted_commits.show()
extracted_commits.registerTempTable('commits')

some_commit_info = spark.sql("select base_exam_id from commits limit 10")
some_commit_info.write.parquet("/tmp/some_commit_info")



## Check Cloudera HDFS for parquet files created in PySpark
  808  docker-compose exec cloudera hadoop fs -ls /tmp/
  809  docker-compose exec cloudera hadoop fs -ls /tmp/some_commit_info
  

## Clean up Docker containers
  810  docker-compose down
  811  docker ps -a
  812  ls
  813  history > brandon-castaing-history.txt
