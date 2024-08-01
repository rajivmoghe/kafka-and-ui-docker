!!! danger 
    <span style="color:red">Everything inside here may be wrong. </span>


### Install Bitnami Kafka setup 

Ref: [See here](https://medium.com/@tetianaokhotnik/setting-up-a-local-kafka-environment-in-kraft-mode-with-docker-compose-and-bitnami-image-enhanced-29a2dcabf2a9)
also [see here][(https://github.com/nanthakumaran-s/Learn-Kafka/blob/main/docker-setup.md) for tweaks

### About
A Docker environment for a Kafka broker without authentication, providing comfortable UI access to our data.

### Background

[Docker container from Bitnami](https://hub.docker.com/r/bitnami/kafka/)  
[Provectus Kafka UI](https://github.com/provectus/kafka-ui)

Note: widely used [wurstmeister kafka](https://hub.docker.com/r/wurstmeister/kafka) supports only Zookeeper, while the Bitnami container can operate in both modes.

Also, watch out for the following:

##### Issue #1 :

The `KAFKA_KRAFT_CLUSTER_ID` does not need to be specified for a single broker, contrary to what some tutorials may suggest. Kafka will generate an ID on the fly and will post it in logs. If you’re setting up a cluster, take this ID from the logs and propagate it to all Kafka services in the docker-compose. All nodes in a single cluster should have the same `KAFKA_KRAFT_CLUSTER_ID`.

##### Issue #2:
*kafka_kafka_1 exited with code 1*  
Kafka may fail silently without throwing an exception. Check for incorrect environment configs and copy paste errors.  
The only way to identify the real problem is to enable debug logging via BITNAMI_DEBUG=yes. This allows to find the root cause of the issues.
It is recommend using this config for troubleshooting.

##### Issue #3 :
Be cautious with `KAFKA_AUTO_CREATE_TOPICS_ENABLE`, as you might inadvertently copy an incorrect value from a tutorial. 

The `KAFKA_CFG_LISTENERS`, `KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP`, and `KAFKA_CFG_ADVERTISED_LISTENERS` are configured in such a way that you can connect to Kafka outside of the Docker container via the `EXTERNAL` listener, with `host kafka_b` and `port 9094`. 

Ensure that the hostname in the config matches the advertised listeners.

##### Issue #4:
Volume given by the first cna be changed as follows: 



#### Step 1: Setup Kafka cluster

The [Bitnami docker-compose](https://github.com/bitnami/containers/blob/main/bitnami/kafka/docker-compose.yml) needs configuration. The following is a recommended configured one:
```
# version: "3" <-- no longer needed
services:
  kafka_b:
    image: docker.io/bitnami/kafka:3.4
    hostname: kafka_b
    ports:
      - "9092:9092"
      - "9094:9094"
    volumes:
      - "kafka_data:/bitnami"
    environment:
      - KAFKA_ENABLE_KRAFT=yes
      - KAFKA_CFG_PROCESS_ROLES=broker,controller
      - KAFKA_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093,EXTERNAL://:9094
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,EXTERNAL:PLAINTEXT
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://127.0.0.1:9092,EXTERNAL://kafka_b:9094
      - KAFKA_BROKER_ID=1
      - KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=1@127.0.0.1:9093
      - ALLOW_PLAINTEXT_LISTENER=yes
      - KAFKA_CFG_NODE_ID=1
      - KAFKA_AUTO_CREATE_TOPICS_ENABLE=true
      - BITNAMI_DEBUG=yes
      - KAFKA_CFG_NUM_PARTITIONS=2
volumes:
  kafka_data:
    driver: local
```

#### Step 2: Setup Provectus Kafka UI
Use the Provectus kafka-ui in another [dir](../prov-kafka-ui/).

To get started with kafka-ui, [visit here](https://docs.kafka-ui.provectus.io/configuration/quick-start).

There’s a trick with this tool: by default, you cannot change cluster configuration in real time. To do this, you need to set up the environment variable `DYNAMIC_CONFIG_ENABLED`. 
Even with this variable, the button for cluster configuration may be disabled. If so directly navigate to the [cluster](http://localhost:8080/ui/clusters/create-new-cluster).

For a successful connection to the local Kafka, specify the correct hostname and port, as configured in the docker-compose for Kafka itself.

Also, the docker-compose provided in the kafka-ui tutorial doesn’t work well for Windows, as the volume source should be specified with an absolute path and it should exist. Otherwise, you will encounter additional errors.

Final docker-compose for kafka-ui

Note: LOGGING_LEVEL_ROOT is added for troubleshooting and is not required at all.
```
services:
  kafka-ui:
    container_name: kafka-ui
    image: provectuslabs/kafka-ui:latest
    ports:
      - 8080:8080
    environment:
      DYNAMIC_CONFIG_ENABLED: 'true'
      LOGGING_LEVEL_ROOT: 'DEBUG'
    volumes:
      - /c/tools/kafka/kui/config.yml:/etc/kafkaui/dynamic_config.yaml
```

##### From provectus.io:
Since version 0.6 we added an ability to change cluster configs in runtime. 

This option is disabled by default and should be implicitly enabled. To enable it, you should set `DYNAMIC_CONFIG_ENABLED` env property to true or add `dynamic.config.enabled: true` property to your yaml config file.

You can even omit all vars other than `DYNAMIC_CONFIG_ENABLED` to start the application with empty configs and setup it up after startup.

#### Step 3: Start the Platforms

Navigate to the `bitnami` and `prov-kafka-ui` folders and run the following command in each:

```
docker-compose up -d
```

#### Step 4: Verify Services

Check that the services are up and running by accessing the Control Center web interface at [http://localhost:9021/clusters](http://localhost:9021/clusters).


#### Step 5: Stop the Platforms

To halt the local Confluent Platform instance and free up resources, execute the following command in the respectiove folders:

```
docker-compose stop
```

To connect to Kafka Services without Zookeeper, we need to specify the bootstrap.servers property.