# Appendix B. Client example
Although the code samples in this book focus on the Java Kafka clients, one of the easiest ways to quickly draw parallels for new users might be to look at examples in programming languages that they are more familiar with. The Confluent Platform also has a list of included clients that it supports [1]. In this appendix, we’ll look at Kafka Python clients and then provide some notes on testing your Java clients.

## B.1 Python Kafka clients
For this example, we’ll look at the Confluent Python Client [2]. The benefit of using a Confluent client is that you have a higher level of confidence that the clients are compatible, not only with Apache Kafka itself but also with the whole of Confluent’s platform offerings. Let’s take a look at how to get started using Python with two (one producer and one consumer) client examples. But first, a brief discussion on installing Python.

### B.1.1 Installing Python
Assuming you are a Python user, you probably already have moved to Python 3 by now. Otherwise, you will need to install librdkafka. If you are using Homebrew, you can use the following command: brew install librdkafka [2].

Next, you will need the client package that your code uses as a dependency. The wheels package for Confluent Kafka can be installed with Pip using pip install confluent-kafka [2]. With these prerequisites on your workstation, let’s look at building a simple Python producer client.

### B.1.2 Python producer example
The following listing shows a simple Python producer client using confluent-kafka-python [2]. It sends two messages to a topic called kinaction-python-topic.

Listing B.1 Python producer example[pythonproducer.py]
```python
from confluent_kafka import Producer                   ❶
 
producer = Producer(
    {'bootstrap.servers': 'localhost:9092'})           ❷
 
def result(err, message):                              ❸
    if err:
        print('kinaction_error %s\n' % err)
    else:
        print('kinaction_info : topic=%s, and kinaction_offset=%d\n' %
        (message.topic(), message.offset()))
 
messages = ["hello python", "hello again"]             ❹
 
for msg in messages:
    producer.poll(0)
    producer.produce("kinaction-python-topic",         ❺
    value=msg.encode('utf-8'), callback=result)
 
producer.flush()                                       ❻
 
# Output                                               ❼
#kinaction_info: topic=kinaction-python-topic, and kinaction_offset=8
 
#kinaction_info: topic=kinaction-python-topic, and kinaction_offset=9
```
❶ Imports the Confluent package first

❷ Configures the producer client to connect to a specific Kafka broker

❸ Acts as a callback for success and failure handling

❹ The array containing the messages to send

❺ Sends all messages to Kafka

❻ Ensures that the messages are sent and not only buffered

❼ Sample output shows the metadata about the two sent messages

To use the Confluent package, you first need to make sure to import the dependency confluent_kafka. You can then set up a Producer client with a set of configuration values, including the address of the broker to connect to. In the listing, the result callback is triggered to run some logic after each call to the produce method, whether the call succeeds or fails. The sample code then iterates over the messages array to send each message in turn. It then calls flush() to make sure that the messages are actually sent to the broker as opposed to only being queued to be sent at a later time. Finally, some sample output is printed to the console. Let’s now turn to the consuming side and see how that works with Python.

## B.1.3 Python consumer
The following listing shows a sample Kafka consumer client using confluent-kafka-python [3]. We will use it to read the messages produced by the Python Kafka producer in listing B.1.

Listing B.2 Python consumer example[pythonconsumer.py]
```python
from confluent_kafka import Consumer             ❶
 
consumer = Consumer({
    'bootstrap.servers': 'localhost:9094',       ❷
    'group.id': 'kinaction_team0group',
    'auto.offset.reset': 'earliest'
})
 
consumer.subscribe(['kinaction-python-topic'])   ❸
 
try:
    while True:
        message = consumer.poll(2.5)             ❹
 
        if message is None:
            continue
        if message.error():
            print('kinaction_error: %s' % message.error())
            continue
        else:
            print('kinaction_info: %s for topic: %s\n'  %
                (message.value().decode('utf-8'),
                 message.topic()))
 
except KeyboardInterrupt:
    print('kinaction_info: stopping\n')
finally:
    consumer.close()                             ❺
 
# Output                                         ❻
# kinaction_info: hello python for topic: kinaction-python-topic
```

❶ Imports the Confluent package first

❷ Configures the consumer client to connect to a specific Kafka broker

❸ Subscribes the consumer to a list of topics

❹ Polls messages inside an infinite loop

❺ Some cleanup to free resources

❻ Prints the consumed message to the console

Similarly to the producer example in listing B.1, we first need to make sure that the confluent_kafka dependency is declared. A Consumer client can then be set up with configuration values, including the address of the broker to connect to. The consumer client then subscribes to an array of topics it wants to consume messages from; in this case, the single topic named kinaction-python-topic. And in the same way as we did with the Java consumer client, we then use a never-ending loop in which the consumer regularly polls Kafka for new messages. The sample output shows a successful message as well as the offset of that message. In the event that the consumer is shut down, the finally block attempts to gracefully close the client by leaving the consumer group after committing any offsets consumed.

The Python examples provided in this section are simple but aim at showing non-Java developers that interacting with Kafka can be done with not only Python, but with most programming languages. Just remember that not all clients support the same level of features as the Java clients do.

### B.1.4 Python commands

* `brew install librdkafka` - optional 
* `pip install confluent-kafka`
* `python pythonproducer.py`
* `python pythonconsumer.py`


## B.2 Client testing
Testing with EmbeddedKafkaCluster is briefly touched on in chapter 7. Now, we’ll explore a few different alternatives to test Kafka code before deploying it to production.

### B.2.1 Unit testing in Java
Unit testing focuses on checking a single unit of software. This isolated testing should, ideally, not depend on any other components. But, how is it possible to test a Kafka client class without connecting to an actual Kafka cluster?

If you are familiar with testing frameworks like Mockito (https://site.mockito.org/), you might decide to create a mock producer object to stand in for the real one. Luckily, the official Kafka client library already provides such a mock, named MockProducer, that implements the Producer interface [4]. No real Kafka cluster is needed to verify that the producer logic works! The mock producer also features a clear method that can be called to clear the messages that have been recorded by the mock producer so that other subsequent tests can be run [4]. Conveniently, the consumer also has a mocked implementation to use as well [4].

### B.2.2 Kafka Testcontainers
As also mentioned in chapter 7, Testcontainers (https://www.testcontainers.org/modules/kafka/) are another option. Whereas the EmbeddedKafkaCluster option depends on a process running the Kafka brokers and ZooKeeper nodes in memory, Testcontainers depend on Docker images.

# References
1. “Kafka Clients.” Confluent documentation (n.d.). https://docs.confluent.io/current/clients/index.html (accessed June 15, 2020).
1. confluent-kafka-python. Confluent Inc. GitHub (n.d.). https://github.com/confluentinc/confluent-kafka-python (accessed June 12, 2020).
1. consumer.py. Confluent Inc. GitHub (n.d.). https://github.com/confluentinc/confluent-kafka-python/blob/master/examples/consumer.py (accessed August 21, 2021).
1. MockProducer<K,V>. Kafka 2.7.0 API. Apache Software Foundation (n.d.). https://kafka.apache.org/27/javadoc/org/apache/kafka/clients/producer/MockProducer.html (accessed May 30, 2021).

