# __:octicons-terminal-16: CLI__

You can find a comprehensive list [here](https://docs.confluent.io/kafka/operations-tools/kafka-tools.html).

??? info "`kafka-topics`"

    Create, delete, describe, or change a topic.

    ```{.bash .copy}
	# Help
	kafka-topics --command-config <property_file> --bootstrap-server <brokerIP:brokerPort> --help

    # Lists topics
    kafka-topics --command-config <property_file> --bootstrap-server <brokerIP:brokerPort> --list

    # Create topic first_topic with 5 partitions, each with 3 replicas
    kafka-topics --command-config <property_file> --bootstrap-server <brokerIP:brokerPort> --topic first_topic --create --partitions 3 --replication-factor 3

    # Describe topic
    kafka-topics --command-config <property_file> --bootstrap-server <brokerIP:brokerPort> --topic first_topic --describe

    # Delete topic
    kafka-topics --command-config <property_file> --bootstrap-server <brokerIP:brokerPort> --topic first_topic --delete
    ```

??? info "`kafka-console-producer.sh`"

    This tool helps to read data from standard input and publish it to Kafka.

    ```{.bash .copy}
	# Help
	kafka-console-producer --producer.config <property_file> --bootstrap-server <brokerIP:brokerPort> --help

    # Produce to a topic
    kafka-console-producer --producer.config <property_file> --bootstrap-server <brokerIP:brokerPort> --topic first_topic

    # Produce to a topic, with different acknowledgments
    kafka-console-producer --producer.config <property_file> --bootstrap-server <brokerIP:brokerPort> --topic first_topic --producer-property acks=all
	# > Message 1 goes to random partition
	# > Message 2 goes to random partition

	# Produce to a topic, don't create the topic if it doesn't exist
	# Even if brokers are set to auto.create.topics.enable=true 
	kafka-console-producer --producer.config <property_file> --bootstrap-server <brokerIP:brokerPort> --topic first_topic --producer-property 'allow.auto.create.topics=false'

	# Produce message to a topic, round robin between partitions
	kafka-console-producer --producer.config <property_file> --bootstrap-server <brokerIP:brokerPort> --topic first_topic --producer-property 'partitioner.class=org.apache.kafka.clients.producer.RoundRobinPartitioner'

	# Produce to a topic with keys
	kafka-console-producer --producer.config <property_file> --bootstrap-server <brokerIP:brokerPort> --topic first_topic --property parse.key=true --property key.separator=:
	# > example key:example value
	# > name:Jhon
    ```

??? info "`kafka-producer-perf-test.sh`"

    This tool enables you to produce a large quantity of data to test producer performance for the Kafka cluster.

    ```{.bash .copy}
	# Help
	kafka-producer-perf-test --producer.config <property_file> --bootstrap-server <brokerIP:brokerPort> --help

	# Producer performance test. record size = 1000 bytes, # of records = 3000, throughput = 200
	# linger.ms = 0 and batch.size = 16 kilobytes
	kafka-producer-perf-test --producer.config <property_file> \
    --throughput 200 \
    --record-size 1000 \
    --num-records 3000 \
    --topic perf-test-topic \
    --producer-props linger.ms=0 batch.size=16384 \
    --print-metrics
	```

??? info "`kafka-console-consumer.sh`"

    This tool helps to read data from Kafka topics and outputs it to standard output.

    ```{.bash .copy hl_lines="22-24 28-29"}
	# Help
	kafka-console-consumer --consumer.config <property_file> --bootstrap-server <brokerIP:brokerPort> --help

	# Consume from the end and from a specific partition
	kafka-console-consumer --consumer.config <property_file> --bootstrap-server <brokerIP:brokerPort> --topic second_topic \
	--partition <partition_id>

	# Consume from the beginning
	kafka-console-consumer --consumer.config <property_file> --bootstrap-server <brokerIP:brokerPort> --topic second_topic --from-beginning

	# Display key, values and timestamp in consumer
	kafka-console-consumer --consumer.config <property_file> --bootstrap-server <brokerIP:brokerPort> --topic second_topic \
	--formatter kafka.tools.DefaultMessageFormatter \
	--property print.timestamp=true \
	--property print.key=true \
	--property print.value=true \
	--property print.partition=true \
	--property print.offset=true \
	--from-beginning

	# Consume messages from a group
	# 1. To add more consumer to group1 run the same command over and again
	# 2. If consumer stops and reconnects it picks up from where it left off and not from where it reconnects,
	# because of consumer group offsets get committed
	kafka-console-consumer --consumer.config <property_file> --bootstrap-server <brokerIP:brokerPort> --topic third_topic \
	--group group1

	# 3. Running --from-beginning reads all mesages from start for new consumer groups only.
	# Again as the offsets get committed.
	kafka-console-consumer --consumer.config <property_file> --bootstrap-server <brokerIP:brokerPort> --topic third_topic \
	--group group1 --from-beginning
	```

??? info "`kafka-consumer-groups.sh`"

    This tool helps to list all consumer groups, describe a consumer group, delete consumer group info, or reset consumer group offsets.

    ```{.bash .copy hl_lines="7-9"}
	# Help
	kafka-consumer-groups --command-config <property_file> --bootstrap-server <brokerIP:brokerPort> --help

	# Describe group
	kafka-consumer-groups --command-config <property_file> --bootstrap-server <brokerIP:brokerPort> --group group1 --describe
	# ex:
	# GROUP           TOPIC         PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG  CONSUMER-ID       HOST            CLIENT-ID
	# group1          topic1        0          15              20              5    consumer-1       /192.168.1.10   consumer-client-1
	# group1          topic1        1          12              15              3    consumer-2       /127.0.0.1      consumer-client-2

	# Delete group
	# Doesn't work if the group is active(i.e no active consumers are connected to the group)
	kafka-consumer-groups --command-config <property_file> --bootstrap-server <brokerIP:brokerPort> --group group1 --delete

	# Reset offsets
	kafka-consumer-groups --command-config <property_file> --bootstrap-server <brokerIP:brokerPort> --group group1 \
	--topic third_topic \
	--reset-offsets \
	--to-earliest \ # (1)!
	--execute # (2)!
	```

	1. 
		- `#!bash --shift-by`: Reset offsets shifting current offset by 'n', where 'n' can be positive or negative.
		- `#!bash --to-current`: Reset offsets to current offset.
		- `#!bash --to-datetime`: Reset offsets to offset from datetime. Format: 'YYYY-MM-DDTHH\:mm:SS.sss'
		- `#!bash --to-earliest`: Reset offsets to earliest offset.
		- `#!bash --to-latest`: Reset offsets to latest offset.
	2. 
		- `#!bash --dry-run`: to validate output of the command before running it
		- `#!bash --execute`: to execute the command