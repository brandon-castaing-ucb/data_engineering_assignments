# Project 3 Setup

- You're a data scientist at a game development company.  
- Your latest mobile game has two events you're interested in tracking: 
- `buy a sword` & `join guild`...
- Each has metadata

## Project 3 Task
- Your task: instrument your API server to catch and analyze event types.
- This task will be spread out over the last four assignments (9-12).

## Project 3 Task Options 

- All: Game shopping cart data used for homework 
- Advanced option 1: Generate and filter more types of items.
- Advanced option 2: Enhance the API to accept parameters for purchases (sword/item type) and filter
- Advanced option 3: Shopping cart data & track state (e.g., user's inventory) and filter


---

# Assignment 10

## Follow the steps we did in class 


### Turn in your `/assignment-10-<user-name>/README.md` file. It should include:
1) A summary type explanation of the example. 
  * For example, for Week 6's activity, a summary would be: "We spun up a cluster with kafka, zookeeper, and the mids container. Then we published and consumed messages with kafka."
2) Your `docker-compose.yml`
3) Source code for the flask application(s) used.
4) Each important step in the process. For each step, include:
  * The command(s) 
  * The output (if there is any).  Be sure to include examples of generated events when available.
  * An explanation for what it achieves 
    * The explanation should be fairly detailed, e.g., instead of "publish to kafka" say what you're publishing, where it's coming from, going to etc.


# Summary 
This assignment walks through the creation of a container cluster which consists of Cloudera HDFS, Kafka, Spark, Zookeeper, and the MIDS container.  I created a Flask Game server which will upload events to Kafka for analysis in a Spark job.  These spark jobs will be submitted to the Spark container and result in transformed data being uploaded to the Cloudera container.

# Steps
## docker-compose up -d
Creating network "assignment10brandoncastaingucb_default" with the default driver
Creating assignment10brandoncastaingucb_cloudera_1
Creating assignment10brandoncastaingucb_mids_1
Creating assignment10brandoncastaingucb_zookeeper_1
Creating assignment10brandoncastaingucb_kafka_1
Creating assignment10brandoncastaingucb_spark_1

Creates the container cluster using docker compose

## docker-compose exec kafka kafka-topics --create --topic events --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
Created topic "events".

Creates a new Kafka topic for events.

## docker-compose exec mids env FLASK_APP=/w205/spark-from-files/game_api.py flask run --host 0.0.0.0
 * Serving Flask app "game_api"
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
184.9.150.215 - - [23/Mar/2019 17:27:13] "GET / HTTP/1.1" 200 -
184.9.150.215 - - [23/Mar/2019 17:27:15] "GET / HTTP/1.1" 200 -
184.9.150.215 - - [23/Mar/2019 17:27:20] "GET /purchase_a_sword HTTP/1.1" 200 -

Start the Flask Game server. Visited my droplet's IP address in a Chrome browser to register a few events.

## docker-compose exec mids kafkacat -C -b kafka:29092 -t events -o beginning -e
{"Accept-Language": "en-US,en;q=0.9", "event_type": "default", "Host": "157.230.138.147:5000", "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8", "Upgrade-Insecure-Requests": "1", "Connection": "keep-alive", "Cookie": "_xsrf=2|f13bdc4c|2f430a93c4ecb1f045d659f6993d2c55|1552618574", "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.109 Safari/537.36", "Accept-Encoding": "gzip, deflate"}
{"Accept-Language": "en-US,en;q=0.9", "event_type": "default", "Host": "157.230.138.147:5000", "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8", "Upgrade-Insecure-Requests": "1", "Connection": "keep-alive", "Cookie": "_xsrf=2|f13bdc4c|2f430a93c4ecb1f045d659f6993d2c55|1552618574", "Cache-Control": "max-age=0", "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.109 Safari/537.36", "Accept-Encoding": "gzip, deflate"}
{"Accept-Language": "en-US,en;q=0.9", "event_type": "purchase_sword", "Host": "157.230.138.147:5000", "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8", "Upgrade-Insecure-Requests": "1", "Connection": "keep-alive", "Cookie": "_xsrf=2|f13bdc4c|2f430a93c4ecb1f045d659f6993d2c55|1552618574", "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.109 Safari/537.36", "Accept-Encoding": "gzip, deflate"}

View the messages published in the events Kafka topic.

## docker-compose exec spark spark-submit /w205/spark-from-files/extract_events.py
Output omitted since it is hundreds of lines.

Submits a Spark job to the Spark container to extract JSON formatted data from the Kafka messages in the events topic.  Finishes by publishing to Cloudera HDFS.

## docker-compose exec cloudera hadoop fs -ls /tmp/extracted_events/
Found 2 items
-rw-r--r--   1 root supergroup          0 2019-03-23 17:48 /tmp/extracted_events/_SUCCESS
-rw-r--r--   1 root supergroup       4192 2019-03-23 17:48 /tmp/extracted_events/part-00000-c382ba30-78ca-4ed6-88f5-13e88791ef46-c000.snappy.parquet

Show the extracted data in parquet format within Cloudera HDFS

## docker-compose exec spark spark-submit /w205/spark-from-files/transform_events.py
Output omitted since it is hundreds of lines.  

Submits a Spark job with a custom python function to update Host to "moe" and Cache-Control to "no-cache" to the Spark container to extract JSON formatted data from the Kafka messa
ges in the events topic.  Finishes by publishing to Cloudera HDFS.

## docker-compose exec spark spark-submit /w205/spark-from-files/separate_events.py
Output omitted since it is hundreds of lines.

Submits a Spark job to the Spark container to extract JSON formatted data from the Kafka messages in the events topic.  Finishes by publishing the default and buy_sword data into seperate parquet files in Cloudera HDFS.

## docker-compose down
Stopping assignment10brandoncastaingucb_spark_1 ... done
Stopping assignment10brandoncastaingucb_kafka_1 ... done
Stopping assignment10brandoncastaingucb_zookeeper_1 ... done
Stopping assignment10brandoncastaingucb_mids_1 ... done
Stopping assignment10brandoncastaingucb_cloudera_1 ... done
Removing assignment10brandoncastaingucb_spark_1 ... done
Removing assignment10brandoncastaingucb_kafka_1 ... done
Removing assignment10brandoncastaingucb_zookeeper_1 ... done
Removing assignment10brandoncastaingucb_mids_1 ... done
Removing assignment10brandoncastaingucb_cloudera_1 ... done
Removing network assignment10brandoncastaingucb_default

Clean up container cluster
