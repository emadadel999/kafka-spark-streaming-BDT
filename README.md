# kafka-spark-streaming-BDT

a project for streaming fake tweets by streaming them from Kafka to Spark and then saving them to HDFS.

Note: for this to work you should have Kafka, Hadoop, Hive, Spark and their respective packages installed and working.

This is done using:

- ubuntu 20 using python3.7
- kafka 2.13-3.2.1
- hadoop 2.10.2
- hive 2.3.9
- spark 2.4.3

### First Terminal --> Zookeeper

We need to start a Zookeeper instance. This is required for running a Kafka cluster

```bash
    cd kafka/
    bin/zookeeper-server-start.sh config/zookeeper.properties
```

### Second Terminal --> Kafka Broker

Now we need to start the Kafka broker.

```bash
    cd kafka/
    bin/kafka-server-start.sh config/server.properties
```

### Third Terminal --> Kafka Config

Now that we have our Kafka cluster running, we need to create the topic that will hold our tweets.

Make a new topic named `tweets`

```bash
    bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic tweets
```

### Fourth Terminal --> Hive Metastore

Now we need to create a database schema for Hive to work with using schematool

~$ schematool -initSchema -dbType derby

Then we start the Hive Metastore server

```bash
    hive --service metastore
```

Make the database for storing our Twitter data:

hive> CREATE TABLE tweets (text STRING, words INT, length INT) > ROW FORMAT DELIMITED FIELDS TERMINATED BY '\\|' > STORED AS TEXTFILE;

### Fifth Terminal --> Stream Producer

Here we run the stream producer

```bash
    ./streamOfTweets.py
```

This script should produce output to the console everytime a tweet is sent to the Kafka cluster.

### Sixth Terminal --> Stream Consumer + Spark Transformer

Now we are ready to run the consumer

```bash
    spark-submit --jars spark-streaming-kafka-0-8-assembly_2.11-2.4.3.jar kafka_transformer.py
```

The results of the MapReduce jobs will show up here.

### Seventh Terminal --> Hive

```bash
    hive

 ...

hive> use default;
hive> select count(*) from tweets;
```
