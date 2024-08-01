### Install confluent Kafka setup 

Ref: [See here](https://zakir-hossain.medium.com/running-confluent-platform-locally-with-docker-compose-ba6d9ad113e7)

#### Step 1: Download the Docker Compose File
```
curl --silent --output docker-compose.yml \
  https://raw.githubusercontent.com/confluentinc/cp-all-in-one/6.2.13-post/cp-all-in-one/docker-compose.yml
```

#### Step 2: Edit the Docker Compose File
Add a volume to the ‘connect’ service, making it look as follows:

```
connect:
    image: cnfldemos/cp-server-connect-datagen:0.6.2-7.5.0
    hostname: connect
    container_name: connect
    # ... (rest of the service configuration)
    volumes:
        - "./confluent-hub-components/:/usr/share/confluent-hub-components/
```
#### Step 3: Start Confluent Platform

Navigate to the ‘confluent-platform’ folder and run the following command:

```
docker-compose up -d
```

#### Step 4: Verify Services

Check that the services are up and running by accessing the Control Center web interface at [http://localhost:9021/clusters](http://localhost:9021/clusters).


#### Step 5: Stop the Confluent Platform

To halt the local Confluent Platform instance and free up resources, execute the following command in the `confluent-platform [this]` folder:

```
docker-compose stop
```