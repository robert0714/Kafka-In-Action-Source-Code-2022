## 3.3 Format of your data
One of the easiest things to skip, but critical to cover in our design, is the format of our data. XML and JSON are pretty standard formats that help define some sort of structure to our data. However, even with a clear syntax format, there can be information missing in our data. What is the meaning of the first column or the third one? What is the data type of the field in the second column of a file? The knowledge of how to parse or analyze our data can be hidden in applications that repeatedly pull the data from its storage location. Schemas are a means of providing some of this needed information in a way that can be used by our code or by other applications that may need the same data.

If you look at the Kafka documentation, you may have noticed references to another serialization system called ``Apache Avro``. Avro provides schema definition support as well as schema storage in Avro files [6]. In our opinion, Avro is likely what you will see in Kafka code that you might encounter in the real world and why we will focus on this choice out of all the available options. Let’s take a closer look at why this format is commonly used in Kafka.

### 3.3.1 Plan for data
One of the significant gains of using Kafka is that the producers and consumers are not tied directly to each other. Further, Kafka does not do any data validation by default. However, there is likely a need for each process or application to understand what that data means and what format is in use. By using a schema, we provide a way for our application’s developers to understand the structure and intent of the data. The definition doesn’t have to be posted in a README file for others in the organization to determine data types or to try to reverse-engineer from data dumps.

Listing 3.8 shows an example of an Avro schema defined as JSON. Fields can be created with details such as name, type, and any default values. For example, looking at the field daysOverDue, the schema tells us that the days a book is overdue is an int with a default value of 0. Knowing that this value is numeric and not text (such as one week) helps to create a clear contract for the data producers and consumers.

Listing 3.8 Avro schema example
```json
{
    "type" : "record",                         ❶
    "name" : "kinaction_libraryCheckout",
    ...
    "fields" : [{"name" : "materialName",
                 "type" : "string",
                 "default" : ""},
 
                {"name" : "daysOverDue",       ❷
                 "type" : "int",               ❸
                  "default" : 0},              ❹
 
                 {"name" : "checkoutDate",
                  "type" : "int",
                  "logicalType": "date",
                  "default" : "-1"},
 
                  {"name" : "borrower",
                   "type" : {
                         "type" : "record",
                         "name" : "borrowerDetails",
                         "fields" : [
                            {"name" : "cardNumber",
                             "type" : "string",
                             "default" : "NONE"}
                          ]},
                          "default" : {}
                }
    ]
}
```
❶ JSON-defined Avro schema

❷ Maps directly to a field name

❸ Defines a field with a name, type, and default value

❹ Provides the default value

By looking at the example of the Avro schema in listing 3.8, we can see that questions such as “Do we parse the cardNumber as a number or a string (in this case, string)” are easily answered by a developer looking at the schema. Applications could automatically use this information to generate data objects for this data, which helps to avoid parsing data type errors.

Schemas can be used by tools like Apache Avro to handle data that evolves. Most of us have dealt with altered statements or tools like Liquibase to work around these changes in relational databases. With schemas, we start with the knowledge that our data will probably change.

Do we need a schema when we are first starting with our data designs? One of the main concerns is that if our system’s scale keeps getting larger, will we be able to control the correctness of data? The more consumers we have could lead to a burden on the testing that we would need to do. Besides the growth in numbers alone, we might not even know all of the consumers of that data.

### 3.3.2 Dependency setup
Now that we have discussed some of the advantages of using a schema, why would we look at Avro? First of all, Avro always is serialized with its schema [7]. Although not a schema itself, Avro supports schemas when reading and writing data and can apply rules to handle schemas that change over time. Also, if you have ever seen JSON, it is pretty easy to understand Avro. Besides the data, the schema language itself is defined in JSON as well. If the schema changes, you can still process data [7]. The old data uses the schema that existed as part of its data. On the other hand, any new formats will use the schema present in their data. Clients are the ones who gain the benefit of using Avro.

Another benefit of looking at Avro is the popularity of its usage. We first saw it used on various Hadoop efforts, but it can be used in many other applications. Confluent also has built-in support for most parts of their tooling [6]. Bindings exist for many programming languages and should not be hard to find, in general. Those who have past “bad” experiences and prefer to avoid generated code can use Avro dynamically without code generation.

Let’s get started with using Avro by adding it to our pom.xml file as the following listing shows [8]. If you are not used to pom.xml or Maven, you can find this file in our project’s root directory.

Listing 3.9 Adding Avro to pom.xml
```xml
<dependency>
  <groupId>org.apache.avro</groupId>    ❶
  <artifactId>avro</artifactId>
  <version>${avro.version}</version>
</dependency>
```
❶ Adds this entry as a dependency to the project’s pom.xml file

Because we are already modifying the POM file, let’s go ahead and include a plugin that generates the Java source code for our schema definitions. As a side note, you can also generate the sources from a standalone Java JAR, avro-tools, if you do not want to use a Maven plugin. For those who do not prefer code generation in their source code projects, this is not a hard requirement [9].

Listing 3.10 shows how to add the avro-maven-plugin to our pom.xml as suggested by the Apache Avro Getting Started with Java documentation site [8]. The code in this listing omits the configuration XML block. Adding the needed configuration also lets Maven know that we want to generate source code for the Avro files found in the source directory we list and to output the generated code to the specified output directory. If you like, you can change the source and output locations to match your specific project structure.

Listing 3.10 Adding the Avro Maven plugin to pom.xml
```xml
<plugin>
  <groupId>org.apache.avro</groupId>
  <artifactId>avro-maven-plugin</artifactId>   ❶
  <version>${avro.version}</version>
  <executions>
    <execution>
      <phase>generate-sources</phase>          ❷
      <goals>
        <goal>schema</goal>                    ❸
      </goals>
      ...
    </execution>
  </executions>
</plugin>
```
❶ Sets the artifact ID needed in our pom.xml as a plugin

❷ Configures the Maven phase

❸ Configures the Maven goal

Let’s start defining our schema by thinking about the data types we want to use, beginning with our alert status scenario. To start, we’ll create a new file named kinaction _alert.avsc with a text editor. The following listing shows the schema definition. We will name our Java class Alert as we will interact with it after the generation of source code from this file.

Listing 3.11 Alert schema: kinaction_alert.avsc
```json
{
  ...
  "type": "record",
  "name": "Alert",       ❶
  "fields": [            ❷
    {
      "name": "sensor_id",
      "type": "long",
      "doc": "The unique id that identifies the sensor"
    },
    {
      "name": "time",
      "type": "long",
      "doc":
        "Time alert generated as UTC milliseconds from epoch"
    },
    {
      "name": "status",
      "type": {
        "type": "enum",
        "name": "AlertStatus",
        "symbols": [
          "Critical",
          "Major",
          "Minor",
          "Warning"
        ]
      },
      "doc":
       "Allowed values sensors use for current status"
    }
  ]
}
```
❶ Names the created Java class

❷ Defines the data types and documentation notes

In listing 3.11, which shows a definition of alerts, one thing to note is that "doc" is not a required part of the definition. However, there is certainly value in adding details that will help future producer or consumer developers understand what the data means. The hope is to stop others from inferring our data’s meaning and to be more explicit about the content. For example, the field "time" always seems to invoke developer anxiety when seen. Is it stored in a string format? Is time zone information included? Does it include leap seconds? The "doc" field can provide that information. A namespace field, not shown in listing 3.11, turns into the Java package for the generated Java class. You can view the full example in the source code for the book. The various field definitions include the name as well as a type.

Now that we have the schema defined, let’s run the Maven build to see what we are working with. The commands mvn generate-sources or mvn install can generate the sources in our project. This should give us a couple of classes, Alert.java and AlertStatus.java, that we can now use in our examples.

Although we have focused on Avro itself, the remaining part of the setup is related to the changes we need to make in our producer and consumer clients to use the schema that we created. We can always define our own serializer for Avro, but we already have an excellent example provided by Confluent. Access to the existing classes is accomplished by adding the kafka-avro-serializer dependency to our build [10]. The following listing shows the pom.xml entry that we’ll add. This is needed to avoid having to create our own Avro serializer and deserializer for the keys and values of our events.

Listing 3.12 Adding the Kafka serializer to pom.xml

<dependency>
    <groupId>io.confluent</groupId>
    <artifactId>kafka-avro-serializer</artifactId>    ❶
    <version>${confluent.version}</version>
</dependency>
❶ Adds this entry as a dependency in the project’s pom.xml file

If you are using Maven to follow along, make sure that you place the Confluent repository in your pom file. This information is needed to let Maven know where to get specific dependencies [11].

<repository>
    <id>confluent</id>
    <url>https://packages.confluent.io/maven/</url>
</repository>
With the build set up and our Avro object ready to use, let’s take our example producer, HelloWorldProducer, from the last chapter and slightly modify the class to use Avro. Listing 3.13 shows the pertinent changes to the producer class (not including imports). Notice the use of io.confluent.kafka.serializers.KafkaAvroSerializer as the value of the property value.serializer. This handles the Alert object that we created and sent to our new kinaction_schematest topic.

Before, we could use a string serializer, but with Avro, we need to define a specific value serializer to tell the client how to deal with our data. The use of an Alert object rather than a string shows how we can utilize types in our applications as long as we can serialize them. This example also makes use of the Schema Registry. We will cover more details about the Schema Registry in chapter 11. This registry can have a versioned history of schemas to help us manage schema evolution.

Listing 3.13 Producer using Avro serialization

public class HelloWorldProducer {
 
  static final Logger log =
    LoggerFactory.getLogger(HelloWorldProducer.class);
 
  public static void main(String[] args) {
    Properties kaProperties = new Properties();
    kaProperties.put("bootstrap.servers",
      "localhost:9092,localhost:9093,localhost:9094");
    kaProperties.put("key.serializer",
      "org.apache.kafka.common.serialization.LongSerializer");
    kaProperties.put("value.serializer",                        ❶
      "io.confluent.kafka.serializers.KafkaAvroSerializer");
    kaProperties.put("schema.registry.url",
      "http://localhost:8081");
 
    try (Producer<Long, Alert> producer =
      new KafkaProducer<>(kaProperties)) {
      Alert alert =
        new Alert(12345L,
          Instant.now().toEpochMilli(),
          Critical);                                            ❷
 
      log.info("kinaction_info Alert -> {}", alert);
 
      ProducerRecord<Long, Alert> producerRecord =
          new ProducerRecord<>("kinaction_schematest",
                               alert.getSensorId(),
                               alert);
 
      producer.send(producerRecord);
    }
  }
}
❶ Sets value.serializer to the KafkaAvroSerializer class for our custom Alert value

❷ Creates a critical alert

The differences are pretty minor. The type changes for our Producer and ProducerRecord definitions, as do the configuration settings for the value.serializer.

Now that we have produced messages using Alert, the other changes would be on the consumption side of the messages. For a consumer to get the values produced to our new topic, it will have to use a value deserializer; in this case, KafkaAvroDeserializer [10]. This deserializer works to get back the value that was serialized by the producer. This code can also reference the same Alert class generated in the project. The following listing shows the significant changes for the consumer class HelloWorldConsumer.

Listing 3.14 Consumer using Avro serialization

public class HelloWorldConsumer {
 
  final static Logger log =
    LoggerFactory.getLogger(HelloWorldConsumer.class);
 
  private volatile boolean keepConsuming = true;
 
  public static void main(String[] args) {
    Properties kaProperties = new Properties();
    kaProperties.put("bootstrap.servers", "localhost:9094");
    ...
    kaProperties.put("key.deserializer",
      "org.apache.kafka.common.serialization.LongDeserializer");
    kaProperties.put("value.deserializer",                            ❶
      "io.confluent.kafka.serializers.KafkaAvroDeserializer");
    kaProperties.put("schema.registry.url", "http://localhost:8081");
 
    HelloWorldConsumer helloWorldConsumer = new HelloWorldConsumer();
    helloWorldConsumer.consume(kaProperties);
 
    Runtime.getRuntime()
      .addShutdownHook(
        new Thread(helloWorldConsumer::shutdown)
      );
  }
 
  private void consume(Properties kaProperties) {
 
    try (KafkaConsumer<Long, Alert> consumer =                        ❷
      new KafkaConsumer<>(kaProperties)) {
      consumer.subscribe(
        List.of("kinaction_schematest")
      );
 
      while (keepConsuming) {
        ConsumerRecords<Long, Alert> records =
          consumer.poll(Duration.ofMillis(250));
        for (ConsumerRecord<Long, Alert> record :                     ❸
          records) {
            log.info("kinaction_info offset = {}, kinaction_value = {}",
              record.offset(),
              record.value());
        }
      }
    }
  }
 
  private void shutdown() {
    keepConsuming = false;
  }
}
❶ Sets value.serializer to the KafkaAvroSerializer class due to the Alert usage

❷ KafkaConsumer typed to handle Alert values

❸ Updates ConsumerRecord to handle Alert values

As with the producer, the consumer client does not require many changes due to the power of updating the configuration deserializer and Avro! Now that we have some ideas about the what we want to accomplish and our data format, we are well equipped to tackle the how in our next chapter. We will cover more schema-related topics in chapter 11 and move on to a different way to handle our object types in the example project in chapters 4 and 5. Although the task of sending data to Kafka is straightforward, there are various configuration-driven behaviors that we can use to help us satisfy our specific requirements.

# Summary
Designing a Kafka solution involves understanding our data first. These details include how we need to handle data loss, ordering of messages, and grouping in our use cases.

The need to group data determines whether we will key the messages in Kafka.

Leveraging schema definitions not only helps us generate code, but it also helps us handle future data changes. Additionally, we can use these schemas with our own custom Kafka clients.

Kafka Connect provides existing connectors to write to and from various data sources.

# References
1. J. MSV. “Apache Kafka: The Cornerstone of an Internet-of-Things Data Platform” (February 15, 2017). https://thenewstack.io/apache-kafka-cornerstone.iot-data-platform/ (accessed August 10, 2017).
1. “Quickstart.” Confluent documentation (n.d.). https://docs.confluent.io/3.1.2/connect/quickstart.html (accessed November 22, 2019).
1. “JDBC Source Connector for Confluent Platform.” Confluent documentation (n.d.). https://docs.confluent.io/kafka-connect-jdbc/current/source-connector/index.html (accessed October 15, 2021).
1. “Running Kafka in Production: Memory.” Confluent documentation (n.d.). https://docs.confluent.io/platform/current/kafka/deployment.html#memory (accessed June 16, 2021).
1. “Download.” Apache Software Foundation (n.d.). https://kafka.apache.org/downloads (accessed November 21, 2019).
1. J. Kreps. “Why Avro for Kafka Data?” Confluent blog (February 25, 2015). https://www.confluent.io/blog/avro-kafka-data/ (accessed November 23, 2017).
1. “Apache Avro 1.8.2 Documentation.” Apache Software Foundation (n.d.). https://avro.apache.org/docs/1.8.2/index.html (accessed November 19, 2019).
1. “Apache Avro 1.8.2 Getting Started (Java)): Serializing and deserializing without code generation.” Apache Software Foundation (n.d.). https://avro.apache.org/docs/1.8.2/gettingstartedjava.html#download_install (accessed November 19, 2019).
1. “Apache Avro 1.8.2 Getting Started (Java): Serializing and deserializing without code generation.” Apache Software Foundation (n.d.). https://avro.apache.org/docs/1.8.2/gettingstartedjava.html#Serializing+and+deserializing+without+code+generation (accessed November 19, 2019).
1. “Application Development: Java.” Confluent documentation (n.d.). https://docs.confluent.io/platform/current/app-development/index.html#java (accessed November 20, 2019).
1. “Installation: Maven repository for jars.” Confluent documentation (n.d.). https://docs.confluent.io/3.1.2/installation.html#maven-repository-for-jars (accessed November 20, 2019).