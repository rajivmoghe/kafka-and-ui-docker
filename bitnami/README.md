## Kafka Develepment Cluster with UI
  1. Run the command      
    `docker compose -f docker-compose-cluster.yml up -d`
  2. Wait till the logs show cluster is running. Then navigate to  
    a. Provectus UI - [http://localhost:8080](http://localhost:8080)  
    or  
    b. Kafbat UI - [http://localhost:9090](http://localhost:9090)  
  3. Connect to cluster  
    a. Give a name - any  string to identify the cluster  
    b. Bootstrap server host - `kafka-0`, port `9092`
  4. Use the cluster  
    a. Create a new Topic or create Messages in an existing Topic (use the wizard)  
    b. Update the Characteristics of a Topic using wizard.  
  .