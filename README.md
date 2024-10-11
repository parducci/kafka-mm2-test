# kafka-mm2-test
Locally testing MM2 with ACLs using docker

# Instructions to Run the Setup

## Step 1: Prepare the Environment
- Ensure you have Docker and Docker Compose installed on your MacBook.
- Place all the files above in the same directory.

Make the scripts executable:

```chmod +x start-mirrormaker.sh setup-kafka1.sh setup-kafka2.sh```

## Step 2: Start the Docker Containers

```docker-compose up -d```

## Step 3: Run Setup Scripts

Wait for the containers to fully start (about a minute).

- Setup Kafka 1:
```docker exec -it kafka1 bash -c '/setup-kafka1.sh'```

- Setup Kafka 2:
```docker exec -it kafka2 bash -c '/setup-kafka2.sh'```

## Step 4: Test Producing and Consuming Messages
- Produce Messages to Kafka 1 as 'testuser':
```docker exec -it kafka1 bash```

  - Inside the container:
```# Create 'client.properties' for 'testuser'
echo "
security.protocol=SASL_PLAINTEXT
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username=\"testuser\" password=\"testuser-secret\";
" > /tmp/client.properties

kafka-console-producer --bootstrap-server kafka1:9093 --topic test-topic --producer.config /tmp/client.properties
```
Type some messages and press Ctrl+D to exit.

- Consume Messages from Kafka 2 as 'testuser':
```docker exec -it kafka2 bash```

Inside the container:
```# Create 'client.properties' for 'testuser'
echo "
security.protocol=SASL_PLAINTEXT
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username=\"testuser\" password=\"testuser-secret\";
" > /tmp/client.properties

kafka-console-consumer --bootstrap-server kafka2:9093 --topic test-topic --consumer.config /tmp/client.properties --from-beginning
```

You should see the messages you produced earlier.

# Additional Notes

- Access Control Lists (ACLs): We used kafka-acls to grant testuser permissions to produce and consume messages on test-topic.
- MirrorMaker 2 Configuration: MirrorMaker 2 is configured to replicate the test-topic from Kafka 1 to Kafka 2 using the admin credentials.
- Security Protocols: Both clusters use SASL_PLAINTEXT with the PLAIN mechanism for authentication.

# Cleaning Up
To stop and remove the Docker containers:
```docker-compose down```
