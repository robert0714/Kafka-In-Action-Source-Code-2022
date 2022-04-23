## 7.3 Testing with EmbeddedKafkaCluster
With all of the configuration options we have, it might be nice to test them as well. What if we could spin up a Kafka cluster without having a real production-ready cluster handy? Kafka Streams provides an integration utility class called EmbeddedKafkaCluster that serves as a middle ground between mock objects and a full-blown cluster. This class provides an in-memory Kafka cluster [13]. Although built with Kafka Streams in mind, we can use it to test our Kafka clients.

Listing 7.6 is set up like the tests found in the book Kafka Streams in Action by William P. Bejeck Jr., for example, his KafkaStreamsYellingIntegrationTest class [14]. That book and his following book, Event Streaming with Kafka Streams and ksqlDB, show more in-depth testing examples. We recommend checking those out, including his suggestion of using Testcontainers (https://www.testcontainers.org/). The following listing shows testing with EmbeddedKafkaCluster and JUnit 4.

Listing 7.6 Testing with EmbeddedKafkaCluster
```java
@ClassRule
public static final EmbeddedKafkaCluster embeddedKafkaCluster
    = new EmbeddedKafkaCluster(BROKER_NUMBER);                     ❶
 
private Properties kaProducerProperties;
private Properties kaConsumerProperties;
 
@Before
public void setUpBeforeClass() throws Exception {
    embeddedKafkaCluster.createTopic(TOPIC,
      PARTITION_NUMBER, REPLICATION_NUMBER);
    kaProducerProperties = TestUtils.producerConfig(
      embeddedKafkaCluster.bootstrapServers(),
      AlertKeySerde.class,
      StringSerializer.class);                                     ❷
 
    kaConsumerProperties = TestUtils.consumerConfig(
      embeddedKafkaCluster.bootstrapServers(),
      AlertKeySerde.class, 
      StringDeserializer.class);                                   ❷
}
 
@Test
public void testAlertPartitioner() throws InterruptedException {
    AlertProducer alertProducer =  new AlertProducer();
    try {
        alertProducer.sendMessage(kaProducerProperties);           ❸
    } catch (Exception ex) {
        fail("kinaction_error EmbeddedKafkaCluster exception"
        ➥ + ex.getMessage());
    }
 
    AlertConsumer alertConsumer = new AlertConsumer();
    ConsumerRecords<Alert, String> records =
      alertConsumer.getAlertMessages(kaConsumerProperties);
    TopicPartition partition = new TopicPartition(TOPIC, 0);
    List<ConsumerRecord<Alert, String>> results = records.records(partition);
    assertEquals(0, results.get(0).partition());                   ❹
}
```
❶ Uses JUnit-specific annotation to create the cluster with a specific number of brokers

❷ Sets the consumer configuration to point to the embedded cluster brokers

❸ Calls the client without any changes, which is clueless of the underlying cluster being embedded

❹ Asserts that the embedded cluster handled the message from production to consumption

When testing with EmbeddedKafkaCluster, one of the most important parts of the setup is to make sure that the embedded cluster is started before the actual testing begins. Because this cluster is temporary, another key point is to make sure that the producer and consumer clients know how to point to this in-memory cluster. To discover those endpoints, we can use the method bootstrapServers() to provide the needed configuration to the clients. Injecting that configuration into the client instances is again up to your configuration strategy, but it can be as simple as setting the values with a method call. Besides these configurations, the clients should be able to test away without the need to provide mock Kafka features!

The test in listing 7.6 verifies that the AlertLevelPartitioner logic was correct. Using that custom partitioner logic with a critical message should have landed the alert on partition 0 with our example code in chapter 4. By retrieving the messages for TopicPartition(TOPIC, 0) and looking at the included messages, the message partition location was confirmed. Overall, this level of testing is usually considered integration testing and moves you beyond just a single component under test. At this point, we have tested our client logic together with a Kafka cluster, integrating more than one module.

NOTE Make sure that you reference the pom.xml changes in the source code for chapter 7. There are various JARs that were not needed in previous chapters. Also, some JARs are only included with specific classifiers, noting that they are only needed for test scenarios.

### 7.3.1 Using Kafka Testcontainers
If you find that you are having to create and then tear down your infrastructure, one option that you can use (especially for integration testing) is Testcontainers (https://www.testcontainers.org/modules/kafka/). This Java library uses Docker and one of a variety of JVM testing frameworks like JUnit. Testcontainers depends on Docker images to provide you with a running cluster. If your workflow is Docker-based or a development technique your team uses well, Testcontainers is worth looking into to get a Kafka cluster set up for testing.

NOTE One of the coauthors of this book, Viktor Gamov, maintains a repository (https://github.com/gAmUssA/testcontainers-java-module-confluent-platform) of integration testing Confluent Platform components (including Kafka, Schema Registry, ksqlDB).


# Summary
* Topics are non-concrete rather than physical structures. To understand the topic’s behavior, a consumer of that topic needs to know about the number of partitions and the replication factors in play.
* Partitions make up topics and are the basic unit for parallel processing of data inside a topic.
* Log file segments are written in partition directories and are managed by the broker.
* Testing can be used to help validate partition logic and may use an in-memory cluster.
* Topic compaction is a way to provide a view of the latest value of a specific record.

# References
1. “Main Concepts and Terminology.” Confluent documentation (n.d.). https://docs.confluent.io/platform/current/kafka/introduction.html#main-concepts-and-terminology (accessed August 28, 2021).
1. J. Rao. “How to choose the number of topics/partitions in a Kafka cluster?” (March 12, 2015). Confluent blog. https://www.confluent.io/blog/how-choose-number-topics-partitions-kafka-cluster/ (accessed May 19, 2019).
1. “Documentation: Modifying topics.” Apache Software Foundation (n.d.). https://kafka.apache.org/documentation/#basic_ops_modify_topic (accessed May 19, 2018).
1. “Documentation: Adding and removing topics.” Apache Software Foundation (n.d.). https://kafka.apache.org/documentation/#basic_ops_add_topic (accessed December 11, 2019).
1. “delete.topic.enable.” Confluent documentation (n.d.). https://docs.confluent.io/platform/current/installation/configuration/broker-configs.html#brokerconfigs_delete.topic.enable (accessed January 15, 2021).
1. Topics.java. Apache Kafka GitHub. https://github.com/apache/kafka/blob/99b9b3e84f4e98c3f07714e1de6a139a004cbc5b/clients/src/main/java/org/apache/kafka/common/internals/Topic.java (accessed August 27, 2021).
1. “auto.create.topics.enable.” Apache Software Foundation (n.d.). https://docs. confluent.io/platform/current/installation/configuration/broker-configs.html#brokerconfigs_auto.create.topics.enable (accessed December 19, 2019).
1. AdminUtils.scala. Apache Kafka GitHub. https://github.com/apache/kafka/blob/d9b898b678158626bd2872bbfef883ca60a41c43/core/src/main/scala/kafka/admin/AdminUtils.scala (accessed August 27, 2021).
1. “Documentation: index.interval.bytes.” Apache Kafka documentation. https://kafka.apache.org/documentation/#topicconfigs_index.interval.bytes (accessed August 27, 2021).
1. “Log Compaction.” Confluent documentation (n.d.). https://docs.confluent.io/platform/current/kafka/design.html#log-compaction (accessed August 20, 2021).
1. “Configuring The Log Cleaner.” Confluent documentation (n.d.). https://docs.confluent.io/platform/current/kafka/design.html#configuring-the-log-cleaner (accessed August 27, 2021).
1. “CLI Tools for Confluent Platform.” Confluent documentation (n.d.). https://docs.confluent.io/platform/current/installation/cli-reference.html (accessed August 25, 2021).
1. EmbeddedKafkaCluster.java. Apache Kafka GitHub. https://github.com/apache/kafka/blob/9af81955c497b31b211b1e21d8323c875518df39/streams/src/test/java/org/apache/kafka/streams/integration/utils/EmbeddedKafka Cluster.java (accessed August 27, 2021).
1. W. P. Bejeck Jr. Kafka Streams in Action. Shelter Island, NY, USA: Manning, 2018.
1. “cleanup.policy.” Confluent documentation (n.d.). https://docs.confluent.io/platform/current/installation/configuration/topic-configs.html#topicconfigs_cleanup.policy (accessed November 22, 2020).
1. “Log Compaction Basics.” Confluent documentation (n.d.). https://docs.confluent.io/platform/current/kafka/design.html#log-compaction-basics (accessed August 20, 2021).

