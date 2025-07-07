# __:material-file-cog: Config__

A full list of configurations can be found [here](https://kafka.apache.org/documentation/#configuration).

## Broker Configurations

??? note "[`listeners`](https://kafka.apache.org/documentation/#connectconfigs_listeners)"

    Each server must define the set of listeners that are used to receive requests from clients as well as other brokers. These channels have to be secured. And each listener maybe configured to authenticate clients using various mechanisms and to establish encrypted channel between client and server, this can be configured by `listener.security.protocol.map`. ==Defines the IP addresses and ports that the Kafka broker actually binds to and listens on.==

    Kafka servers support listening for connections on multiple ports. `listeners` are used to congiure this(==at least one required==).

    ==By default, if `listeners` is not explicitly set, Kafka will use the values in `advertised.listeners` as the addresses and ports to listen on.==

    ``` properties
    # format: {LISTENER_NAME}://{hostname}:{port}

    # LISTENER_NAME: usually descriptive name, defines the purpose of the listener
    listeners=CLIENT://localhost:9092,BROKKER://localhost:9093       # separate listener for client traffic
    ```

    <figure markdown="span">
        <figcaption>Listeners</figcaption>
        ![Component Lifecycle](./img/listeners.png){ width="80%" }
    </figure>

??? note "[`advertised.listeners`](https://kafka.apache.org/documentation/#brokerconfigs_advertised.listeners)"

    The `advertised.listeners` configuration specifies the addresses and ports that Kafka will advertise to clients for connecting to the broker. This setting is used when the address on which the broker listens (listeners) is different from the address clients should use to connect to it.

    For clients outside kafka's internal network, we use of public endpoints as listed in `advertised.listeners`. For clients and brokers within kafka's internal network, we use endpoints listed in `listeners`.

    ``` properties
    advertised.listeners=INTERNAL://kafka1:19092,EXTERNAL://${DOCKER_HOST_IP:-127.0.0.1}:9092,DOCKER://host.docker.internal:29092
    # Within Kafka bridge network use INTERNAL, i.e inter-broker communication
    # Other Containers in the Docker network, should use DOCKER
    # Clients outside docker & kafka bridge network, should use EXTERNAL
    ```

    <figure markdown="span">
        <figcaption>Advertised Listener for external connection</figcaption>
        ![Component Lifecycle](./img/advertised-listeners.png){ width="80%" }
    </figure>

!!! warning "Client Broker connection flow"

    1. Client initiates connection with a bootstrap server. Only for discovering metadata about the Kafka cluster, including brokers, partitions, and leader information, etc...
    2. Client requests metadata. All advertised.listeners(INTERNAL, EXTERNAL, DOCKER) of each broker is returned.
    3. Client is responsible to pick the correct accessible advertised.listener from the response list. Multiple listeners maybe tried using some fallback logic.

??? note "[`listener.security.protocol.map`](https://kafka.apache.org/documentation/#brokerconfigs_listener.security.protocol.map)"

    Defines security protocol of each listener. The value is a comma-separated list of each listener mapped to its security protocol.

    ``` properties
    # CLIENT listener will use SSL while the BROKER listener will use plaintext.
    listener.security.protocol.map=CLIENT:SSL,BROKER:PLAINTEXT
    ```

    Possible options (case-insensitive) for the security protocol are given below:
    
    - PLAINTEXT
    - SSL
    - SASL_PLAINTEXT
    - SASL_SSL

    If each required listener uses a separate security protocol, it is also possible to use the security protocol name as the listener name in listeners

    ``` properties
    listeners=SSL://localhost:9092,PLAINTEXT://localhost:9093
    ```

??? note "[`inter.broker.listener.name`](https://kafka.apache.org/documentation/#brokerconfigs_inter.broker.listener.name)"

    To declare the listener used for inter-broker. The primary purpose of the inter-broker listener is partition replication. If not defined, then the inter-broker listener is determined by the security protocol defined by `security.inter.broker.protocol`

    ``` properties
    # CLIENT listener will use SSL while the BROKER listener will use plaintext.
    listener.security.protocol.map=CLIENT:SSL,BROKER:PLAINTEXT
    ```

??? note "[`security.inter.broker.protocol`](https://kafka.apache.org/documentation/#brokerconfigs_security.inter.broker.protocol)"

    Security protocol used for inter-broker communication, which defaults to `PLAINTEXT`.

??? note "[`control.plane.listener.name`](https://kafka.apache.org/documentation/#brokerconfigs_control.plane.listener.name)"

    For legacy clusters which rely on Zookeeper to store cluster metadata, it is possible to declare a separate listener to be used for metadata propagation from active controller to the brokers. The active controller will use this listener when it needs to push metadata updates to the brokers in the cluster.
    
    ==This benefits from a separate processing thread, which makes it less likely for application traffic to impede timely propagation of metadata changes (such as partition leader and ISR updates). Defaults to listener defined by `inter.broker.listener`

!!! note "[`controller.listener.names`](https://kafka.apache.org/documentation/#brokerconfigs_controller.listener.names)"

    In a KRaft cluster, listener configuration depends on the role(broker, controller). The listener defined by inter.broker.listener.name is used exclusively for requests between brokers. Controllers, on the other hand, must use separate listener defined by `controller.listener.names`. This cannot be set to the same value as the inter-broker listener.

    Controllers receive requests both from other controllers and from brokers. For this reason, servers with `broker` role enabled must define the controller listener along with any security properties that are needed to configure it.

    ``` properties title="Broker role configuration"
    process.roles=broker
    # Notice how this doesn't expose CONTOLLER listener, as this is just a broker
    # But controller security protocol is still configured to SASL_SSL,
    # as brokers need to communicate with the controller
    listeners=BROKER://localhost:9092
    inter.broker.listener.name=BROKER
    controller.quorum.bootstrap.servers=localhost:9093
    controller.listener.names=CONTROLLER
    listener.security.protocol.map=BROKER:SASL_SSL,CONTROLLER:SASL_SSL
    ```

    ``` properties title="Shared role configuration"
    process.roles=broker,controller
    listeners=BROKER://localhost:9092,CONTROLLER://localhost:9093
    inter.broker.listener.name=BROKER
    controller.quorum.bootstrap.servers=localhost:9093
    controller.listener.names=CONTROLLER
    listener.security.protocol.map=BROKER:SASL_SSL,CONTROLLER:SASL_SSL
    ```

    When communicating with the controller quorum, the broker will always use the first listener in this list.

??? note "[`controller.quorum.bootstrap.servers`]()"

    It is a requirement that the host and port defined in controller.quorum.bootstrap.servers is routed to the exposed controller listeners. List of endpoints to use for bootstrapping the cluster metadata. The endpoints are specified in comma-separated list of {host}:{port} entries. For example: `localhost:9092,localhost:9093,localhost:9094`

??? note "[`zookeeper.connect`](https://kafka.apache.org/documentation/#brokerconfigs_zookeeper.connect)"

    Specifies the ZooKeeper connection string in the form hostname:port. Can also specify multiple hosts in the form hostname1:port1,hostname2:port2,hostname3:port3.

??? note "[`broker.id`](https://kafka.apache.org/documentation/#brokerconfigs_broker.id)"

    The broker id for this server. If unset, a unique broker id will be generated.

??? note "[`process.roles`](https://kafka.apache.org/documentation/#brokerconfigs_process.roles)"

    The roles that this process plays: 'broker', 'controller', or 'broker,controller' if it is both. This configuration is only applicable for clusters in KRaft (Kafka Raft) mode (instead of ZooKeeper). Leave this config undefined or empty for ZooKeeper clusters.

??? note "[`node.id`](https://kafka.apache.org/documentation/#brokerconfigs_node.id)"

    The node ID associated with the roles this process is playing when process.roles is non-empty. This is required configuration when running in KRaft mode.

??? note "[`offsets.topic.replication.factor`](https://kafka.apache.org/documentation/#brokerconfigs_offsets.topic.replication.factor)"

    The replication factor for the internal topic used to store consumer offsets, typically named `__consumer_offsets`. ==Internal topic creation will fail until the cluster size meets this replication factor requirement==.

??? note "[`offsets.topic.num.partitions`](https://kafka.apache.org/documentation/#brokerconfigs_offsets.topic.num.partitions)"

    Defines the number of partitions for the internal __consumer_offsets topic. ==default: 50==

??? note "[`log.dirs`](https://kafka.apache.org/documentation/#brokerconfigs_log.dirs)"

    A comma-separated list of the directories where the log data is stored. If not set, the value in `log.dir` is used.

??? note "[`auto.create.topics.enable`](https://kafka.apache.org/documentation/#brokerconfigs_auto.create.topics.enable)"

    Enable auto creation of topic on the server. Broker-Level Setting

??? note "[`num.partitions`](https://kafka.apache.org/documentation/#brokerconfigs_num.partitions)"

    The default number of log partitions per topic

??? note "[`default.replication.factor`](https://kafka.apache.org/documentation/#brokerconfigs_default.replication.factor)"

    The replication factor for automatically created topics, and for topics created with -1 as the replication factor. ==default: 1==


## Producer Configurations

??? note "[`client.id`](https://kafka.apache.org/documentation/#producerconfigs_client.id)"

    An id string to pass to the server when making requests. Enables traking of requests source beyond just ip/port by allowing a logical application name to be included in server-side request logging.

??? note "[`key.serializer`](https://kafka.apache.org/documentation/#producerconfigs_key.serializer)"

    Serializer class for key that implements the org.apache.kafka.common.serialization.Serializer interface.

??? note "[`value.serializer`](https://kafka.apache.org/documentation/#producerconfigs_value.serializer)"

    Serializer class for value that implements the org.apache.kafka.common.serialization.Serializer interface.

??? note "[`bootstrap.servers`](https://kafka.apache.org/documentation/#producerconfigs_bootstrap.servers)"

    A list of host/port pairs used to establish the initial connection to the Kafka cluster. Clients use this list to bootstrap and discover the full set of Kafka brokers. While the order of servers in the list does not matter, we recommend including more than one server to ensure resilience if any servers are down. This list does not need to contain the entire set of brokers, as Kafka clients automatically manage and update connections to the cluster efficiently. This list must be in the form `host1:port1,host2:port2,....`

??? note "[`partitioner.class`](https://kafka.apache.org/documentation/#producerconfigs_partitioner.class)"

    Determines which partition to send a record to when records are produced. Available options are:

    - If not set, the default partitioning logic is used. This strategy send records to a partition until at least `batch.size` bytes is produced to the partition.
        - If no partition is specified but a key is present, choose a partition based on a hash of the key.
        - If no partition or key is present, choose the sticky partition that changes when at least batch.size bytes are produced to the partition.
    - `org.apache.kafka.clients.producer.RoundRobinPartitioner`: Each record in a series of consecutive records is sent to a different partition, regardless of whether the 'key' is provided or not.
    - Implementing the org.apache.kafka.clients.producer.Partitioner interface allows you to plug in a custom partitioner.

??? note "[`metadata.max.age.ms`](https://kafka.apache.org/documentation/#producerconfigs_metadata.max.age.ms)"

    The period of time in milliseconds after which we force a refresh of metadata even if we haven't seen any partition leadership changes to proactively discover any new brokers or partitions.

??? note "[`batch.size`](https://kafka.apache.org/documentation/#producerconfigs_batch.size)"

    The producer will attempt to batch records together into fewer requests whenever multiple records are being sent to the same partition. This helps performance on both the client and the server. This configuration controls the default batch size in bytes.

    No attempt will be made to batch records larger than this size.

    ==Requests sent to brokers will contain multiple batches, one for each partition with data available to be sent.==

??? note "[`linger.ms`](https://kafka.apache.org/documentation/#producerconfigs_linger.ms)"

    The producer groups together any records that arrive in between request transmissions into a single batched request.

    If we have fewer than `batch.size` many bytes accumulated for a particular partition, we will 'linger' for the `linger.ms` time waiting for more records to show up.
    
    Defaults to 0, which means we'll immediately send out a record even when the accumulated batch size is under this `batch.size` setting.


## Consumer Configurations

??? note "[`group.id`](https://kafka.apache.org/documentation/#consumerconfigs_group.id)"

    A unique string that identifies the consumer group this consumer belongs to. This property is required if the consumer uses either the group management functionality by using subscribe(topic) or the Kafka-based offset management strategy.

??? note "[`group.instance.id`](https://kafka.apache.org/documentation/#consumerconfigs_group.instance.id)"

    A unique identifier of the consumer instance provided by the end user. Only non-empty strings are permitted. If set, the consumer is treated as a static member, which means that only one instance with this ID is allowed in the consumer group at any time. This can be used in combination with a larger session timeout to avoid group rebalances caused by transient unavailability (e.g. process restarts). If not set, the consumer will join the group as a dynamic member, which is the traditional behavior.

??? note "[`max.poll.interval.ms`](https://kafka.apache.org/documentation/#consumerconfigs_max.poll.interval.ms)"

    The maximum delay between invocations of poll() when using consumer group management. This places an upper bound on the amount of time that the consumer can be idle before fetching more records. If poll() is not called before expiration of this timeout, then the consumer is considered failed and the group will rebalance in order to reassign the partitions to another member. For consumers using a non-null `group.instance.id` which reach this timeout, partitions will not be immediately reassigned. Instead, the consumer will stop sending heartbeats and partitions will be reassigned after expiration of `session.timeout.ms`. This mirrors the behavior of a static consumer which has shutdown.

??? note "[`max.poll.records`](https://kafka.apache.org/documentation/#consumerconfigs_max.poll.records)"

    The maximum number of records returned in a single call to poll(). Note, that max.poll.records does not impact the underlying fetching behavior. The consumer will cache the records from each fetch request and returns them incrementally from each poll.

??? note "[`auto.offset.reset`](https://kafka.apache.org/documentation/#consumerconfigs_auto.offset.reset)"

    What to do when there is no initial offset in Kafka or if the current offset does not exist any more on the server (e.g. because that data has been deleted):

    - __earliest__: automatically reset the offset to the earliest offset
    - __latest__: automatically reset the offset to the latest offset
    - __none__: throw exception to the consumer if no previous offset is found for the consumer's group
    - __anything else__: throw exception to the consumer.

??? note "[`enable.auto.commit`](https://kafka.apache.org/documentation/#consumerconfigs_enable.auto.commit)"

    If true the consumer's offset will be periodically committed in the background.

??? note "[`auto.commit.interval.ms`](https://kafka.apache.org/documentation/#consumerconfigs_auto.commit.interval.ms)"

    The frequency in milliseconds that the consumer offsets are auto-committed to Kafka if `enable.auto.commit` is set to `true`.

??? note "[`fetch.min.bytes`](https://kafka.apache.org/documentation/#consumerconfigs_fetch.min.bytes)"

    The minimum amount of data the server should return for a fetch request. If insufficient data is available the request will wait for that much data to accumulate before answering the request. The default setting of 1 byte means that fetch requests are answered as soon as that many byte(s) of data is available or the fetch request times out waiting for data to arrive. Setting this to a larger value will cause the server to wait for larger amounts of data to accumulate which can improve server throughput a bit at the cost of some additional latency.

??? note "[`fetch.max.wait.ms`](https://kafka.apache.org/documentation/#consumerconfigs_fetch.max.wait.ms)"

    The maximum amount of time the server will block before answering the fetch request there isn't sufficient data to immediately satisfy the requirement given by fetch.min.bytes. This config is used only for local log fetch. To tune the remote fetch maximum wait time, please refer to 'remote.fetch.max.wait.ms' broker config


## [Logging](https://docs.confluent.io/platform/current/monitor/cp-logging.html)

Apache Kafka uses the Java-based logging utility Apache Log4j. You can find the `log4j.properties` file under `${CONFLUENT_HOME}/etc/kafka`. Connect Log4j properties file is also located under the kafka directory and is named `connect-log4j.properties`.

You can make configuration changes in the existing file or you can specify a configuration file at component start-up by specifying the component and file using the `${COMPONENT}_LOG4J_OPTS` environment variable.

``` bash
KAFKA_LOG4J_OPTS="-Dlog4j.configuration=file:/path/to/log4j.properties" ./bin/kafka-server-start ./etc/kafka/kraft/server.properties
```

## Monitoring

Java Management Extensions (JMX) and Managed Beans (MBeans) are technologies for monitoring and managing Java applications, and they are enabled by default for Kafka and provide metrics for its components; brokers, controllers, producers, and consumers.

### [Configure JMX](https://docs.confluent.io/platform/current/kafka/monitoring.html#kafka-monitoring-metrics-broker)

You can use the following JVM environment variables to configure JMX monitoring when you start Kafka or use Java system properties to enable remote JMX programmatically.

!!! note "`JMX_PORT`"

    The port that JMX listens on for incoming connections

!!! note "`JMX_HOSTNAME`"

    The hostname associated with locally created remote objects.

!!! note "`KAFKA_JMX_OPTS`"

    JMX options. Use this variable to override the default JMX options such as whether authentication is enabled. The default options are set as follows:
    
    ``` bash
    -Djava.rmi.server.hostname=127.0.0.1 -Dcom.sun.management.jmxremote=true -Dcom.sun.management.jmxremote.authenticate=false  -Dcom.sun.management.jmxremote.ssl=false
    ```

    - `server.hostname`: Specifies the host a JMX client connects to. The default is localhost (127.0.0.1)
    - `jmxremote=true`: Enables the JMX remote agent and enables the connector to listen through a specific port.
    - `jmxremote.authenticate=false`: Indicates that authentication is off by default.
    - `jmxremote.ssl=false`: Indicates that TLS/SSL is off by default.  


For example, to start Kafka, and specify a JMX port, you can use the following command:
``` bash
JMX_PORT=2020 ./bin/kafka-server-start.sh ./etc/kafka/kraft/server.properties
```