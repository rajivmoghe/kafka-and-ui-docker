#### Install confluent Kafka setup 

Ref: [See here](https://docs.confluent.io/platform/current/platform-quickstart.html)

Download or copy the contents of the Confluent Platform KRaft all-in-one Docker Compose file [from here](https://github.com/confluentinc/cp-all-in-one/tree/7.6.2-post/cp-all-in-one-kraft/docker-compose.yml) (this is done.)

#### Run the containers

Takes  a while to download the first time around.

Then `docker compose up --detach` does the trick.

To connect to the UI: [http://localhost:9021](http://localhost:9021)

Shutdown with `docker compose down`

Volumes are persisted, containers are destroyed.
