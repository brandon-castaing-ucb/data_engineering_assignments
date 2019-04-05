# Project 3 Setup

- You're a data scientist at a game development company.  
- Your latest mobile game has two events you're interested in tracking: 
- `buy a sword` & `join guild`...
- Each has metadata

## Project 3 Task
- Your task: instrument your API server to catch and analyze these two
event types.
- This task will be spread out over the last four assignments (9-12).

---

# Assignment 09

## Follow the steps we did in class 
- for both the simple flask app and the more complex one.

### Turn in your `/assignment-09-<user-name>/README.md` file. It should include:
1) A summary type explanation of the example. 
  * For example, for Week 6's activity, a summary would be: "We spun up a cluster with kafka, zookeeper, and the mids container. Then we published and consumed messages with kafka."
2) Your `docker-compose.yml`
3) Source code for the flask application(s) used.
4) Each important step in the process. For each step, include:
  * The command(s) 
  * The output (if there is any).  Be sure to include examples of generated events when available.
  * An explanation for what it achieves 
    * The explanation should be fairly detailed, e.g., instead of "publish to kafka" say what you're publishing, where it's coming from, going to etc.

## Assignment 09 - Submission
### Summary
This assignment walks through setting up a container cluster with Kafka, Zookeeper, and the MIDS container.  I created an API server with 2 versions, one which publishes simple events to Kafka and another which publishes the same event with additional web request information.  Finally, this information is consumed from Kafka via Kafkacat.

### Commands and Output
####docker-compose up -d
Create container cluster


####docker-compose exec kafka kafka-topics --create --topic events --partitions 1 --replication-factor 1 --if-not-exists --zookeeper zookeeper:32181
Create Kafka events topic

####docker-compose exec mids env FLASK_APP=/w205/assignment-09-brandon-castaing-ucb/game_api_with_json_events.py flask run --host 0.0.0.0
Start simple Flask API Server

####docker-compose exec mids curl http://localhost:5000/
####docker-compose exec mids curl http://localhost:5000/purchase_a_sword
Make calls to API server
Output: 
127.0.0.1 - - [15/Mar/2019 23:32:41] "GET /purchase_a_sword HTTP/1.1" 200 -
127.0.0.1 - - [15/Mar/2019 23:33:01] "GET / HTTP/1.1" 200 -
 
####docker-compose exec mids kafkacat -C -b kafka:29092 -t events -o beginning -e
Consume Kafka messages in events topic
Output:
{"event_type": "default"}
{"event_type": "purchase_sword"}
% Reached end of topic events [0] at offset 2: exiting

####docker-compose exec mids env FLASK_APP=/w205/assignment-09-brandon-castaing-ucb/game_api_posting_to_kafka.py flask run --host 0.0.0.0
Note, stopped previous server before starting this one.  Start Flask API Server with additional web request information added to events published to Kafka.

####docker-compose exec mids curl http://localhost:5000/purchase_a_sword
####docker-compose exec mids curl http://localhost:5000/
Make calls to API server again
Output:
127.0.0.1 - - [15/Mar/2019 23:34:04] "GET /purchase_a_sword HTTP/1.1" 200 -
127.0.0.1 - - [15/Mar/2019 23:34:11] "GET / HTTP/1.1" 200 -

####docker-compose exec mids kafkacat -C -b kafka:29092 -t events -o beginning -e
Consume Kafka messages in events topic
Output:
{"event_type": "default"}
{"event_type": "purchase_sword"}
{"Host": "localhost:5000", "event_type": "purchase_sword", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
{"Host": "localhost:5000", "event_type": "default", "Accept": "*/*", "User-Agent": "curl/7.47.0"}
% Reached end of topic events [0] at offset 4: exiting

####docker-compose down
Clean up container cluster
