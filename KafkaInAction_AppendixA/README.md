# Appendix A. Installation
Despite having a sophisticated feature set, the Apache Kafka installation process is straightforward. Let’s look at setup concerns first.

## A.1 Operating system (OS) requirements
Linux is the most likely home for Kafka, and that seems to be where many user support forums continue to focus their questions and answers. We’ve used macOS with Bash (a default terminal before macOS Catalina) or zsh (the default terminal since macOS Catalina). Though it’s totally fine to run Kafka on Microsoft® Windows® for development, it’s not recommended in a production environment [1].

> **⚠ NOTE:**    In a later section, we also explain installation using Docker (http://docker.com).

## A.2 Kafka versions
Apache Kafka is an active Apache Software Foundation project, and over time, the versions of Kafka are updated. Kafka releases have, in general, taken backward compatibility seriously. If you want to use a new version, do so and update any parts of the code marked as deprecated.



> **⚠ TIP:**  Generally, Apache ZooKeeper and Kafka should not run on one physical server in a production environment if you want fault tolerance. For this book, we wanted to make sure you focus on learning Kafka features instead of managing mul> **⚠ TIP:** le servers while you’re learning.

## A.3 Installing Kafka on your local machine
When some of the authors started using Kafka, one of the more straightforward options was to create a cluster on a single node by hand. Michael Noll, in the article “Running a Multi-Broker Apache Kafka 0.8 Cluster on a Single Node,” laid out the steps in a clear manner, as reflected in this section’s setup steps [2].

Although written in 2013, this setup option is still a great way to see the details involved and changes needed that might be missed in a more automated local setup. Docker setup is also an option for local setup, provided later in this appendix if you feel more comfortable with that.

From our personal experience, you can install Kafka on a workstation with the following minimum requirements (however your results may vary from our preferences). Then use the instructions in the following sections to install Java and Apache Kafka (which includes ZooKeeper) on your workstation:

* Minimum number of **CPUs** (physical or logical): ``2``
* Minimum amount of **RAM**: ``4 GB``
* Minimum hard drive free space: ``10 GB``

### A.3.1 Prerequisite: Java
Java is a prerequisite that you should install first. For the examples in this book, we use the Java Development Kit (JDK) version 11. You can download Java versions from https://jdk.dev/download/. We recommend using the SDKMAN CLI at http://sdkman.io to install and manage Java versions on your machine.

### A.3.2 Prerequisite: ZooKeeper
At the time of writing, Kafka also requires ZooKeeper, which is bundled with the Kafka download. Even with the reduced dependency on ZooKeeper from the client side in recent versions, Kafka needs a running installation of ZooKeeper to work. The Apache Kafka distribution includes a compatible version of ZooKeeper; you don’t need to download and install it separately. The required scripts to start and stop ZooKeeper are also included in the Kafka distribution.

### A.3.3 Prerequisite: Kafka download
[Confluent Platform and Apache Kafka Compatibility](https://docs.confluent.io/platform/current/installation/versions-interoperability.html#cp-and-apache-ak-compatibility)
| Confluent Platform | Apache Kafka® | Release Date       | Standard End of Support | Platinum End of Support |
|--------------------|---------------|--------------------|-------------------------|-------------------------|
| 7.1.x              | 3.1.x         | April 5, 2022      | April 5, 2024           | April 5, 2025           |
| 7.0.x              | 3.0.x         | October 27, 2021   | October 27, 2023        | October 27, 2024        |
| 6.2.x              | 2.8.x         | June 8, 2021       | June 8, 2023            | June 8, 2024            |
| 6.1.x              | 2.7.x         | February 9, 2021   | February 9, 2023        | February 9, 2024        |
| 6.0.x              | 2.6.x         | September 24, 2020 | September 24, 2022      | September 24, 2023      |
| 5.5.x              | 2.5.x         | April 24, 2020     | April 24, 2022          | April 24, 2023          |
| 5.4.x              | 2.4.x         | January 10, 2020   | January 10, 2022        | January 10, 2023        |
| 5.3.x              | 2.3.x         | July 19, 2019      | July 19, 2021           | July 19, 2022           |
| 5.2.x              | 2.2.x         | March 28, 2019     | March 28, 2021          | March 28, 2022          |
| 5.1.x              | 2.1.x         | December 14, 2018  | December 14, 2020       | December 14, 2021       |
| 5.0.x              | 2.0.x         | July 31, 2018      | July 31, 2020           | July 31, 2021           |
| 4.1.x              | 1.1.x         | April 16, 2018     | April 16, 2020          | April 16, 2021          |
| 4.0.x              | 1.0.x         | November 28, 2017  | November 28, 2019       | November 28, 2020       |
| 3.3.x              | 0.11.0.x      | August 1, 2017     | August 1, 2019          | August 1, 2020          |
| 3.2.x              | 0.10.2.x      | March 2, 2017      | March 2, 2019           | March 2, 2020           |
| 3.1.x              | 0.10.1.x      | November 15, 2016  | November 15, 2018       | November 15, 2019       |
| 3.0.x              | 0.10.0.x      | May 24, 2016       | May 24, 2018            | May 24, 2019            |
| 2.0.x              | 0.9.0.x       | December 7, 2015   | December 7, 2017        | December 7, 2018        |
| 1.0.0              | –             | February 25, 2015  | February 25, 2017       | February 25, 2018       |

[Confluent for Kubernetes(CFK)](https://docs.confluent.io/platform/current/installation/versions-interoperability.html#operator-cp-compatibility)
| CFK Version | Compatible Confluent Platform Versions | Compatible Kubernetes Versions     | Release Date  | End of Support |
|-------------|----------------------------------------|------------------------------------|---------------|----------------|
| 2.3.x       | 7.0.x, 7.1.x                           | 1.18 - 1.23 (OpenShift 4.6 - 4.10) | April 5, 2022 | April 5, 2023  |
| 2.2.x       | 6.2.x, 7.0.x                           | 1.17 - 1.22 (OpenShift 4.6 - 4.9)  | Nov 3, 2021   | Nov 3, 2022    |
| 2.1.x       | 6.0.x, 6.1.x, 6.2.x                    | 1.17 - 1.22 (OpenShift 4.6 - 4.9)  | Oct 12, 2021  | Oct 12, 2022   |
| 2.0.x       | 6.0.x, 6.1.x, 6.2.x                    | 1.15 - 1.20                        | May 12, 2021  | May 12, 2022   |

[Kafka Client Compatibility](https://github.com/spring-cloud/spring-cloud-stream/wiki/Kafka-Client-Compatibility)
| Spring Cloud Stream Version | Spring for Apache Kafka Version | Spring Integration for Apache Kafka Version | kafka-clients | Spring Boot  | Spring Cloud |
|-----------------------------|---------------------------------|---------------------------------------------|---------------|--------------|--------------|
| 3.1.x (2020.0.x)            | 2.6.x                           | 5.4.x                                       | 2.6.x         | 2.4.x        | 2020.0.x     |
| 3.0.x (Horsham)*            | 2.5.x, 2.3.x                    | 3.3.x, 3.2.x                                | 2.5.x, 2.3.x  | 2.3.x, 2.2.x | Hoxton*      |


[upgrade-guide](https://kafka.apache.org/31/documentation/streams/upgrade-guide)

At the time of this book’s publication, Kafka version 2.7.1 (the version used in our examples) was a recent release. The Apache® project has mirrors, and you can search for the version to download in that way. ``To be automatically redirected to the nearest mirror, use this URL``: [http://mng.bz/aZo7](https://archive.apache.org/dist/kafka/2.7.1/kafka_2.13-2.7.1.tgz).

After downloading the file, take a look at the actual binary filename. It might seem a little confusing at first. For example, kafka_2.13-2.7.1 means the Kafka version is 2.7.1 (the information after the hyphen).

To get the most out of the examples in this book while still making things easy to get started, we recommend that you set up a three-node cluster on a single machine. This is not a recommended strategy for production, however, but it will allow you to understand critical concepts without the overhead of spending a lot of time on the setup.

> **⚠ NOTE:**  Why bother to use a three-node cluster? Kafka’s various parts as a distributed system lend themselves to more than one node. Our examples simulate a cluster without the need for different machines in the hope of clarifying what you are learning.

After you install Kafka, you need to configure a three-node cluster. First, you need to unpack the binary and locate the bin directory.

Listing A.1 shows the tar command used to unpack the JAR file, but you might need to use unzip or another tool, depending on your downloaded compression format [3]. It’s a good idea to include the Kafka scripts bin directory in your $PATH environment variable. In this case, the commands are available without specifying a full path to them.

Listing A.1 Unpacking the Kafka binary
```shell
$ tar -xzf kafka_2.13-2.7.1.tgz
$ mv kafka_2.13-2.7.1 ~/
$ cd ~/kafka_2.13-2.7.1
$ export PATH=$PATH:~/kafka_2.13-2.7.1/bin     ❶
```
❶ Adds the bin directory to your shell $PATH

> **⚠ NOTE:**  For Windows users, you’ll find the .bat scripts under the bin/windows folder with the same names as the shell scripts used in the following examples. You can use Windows Subsystem for Linux 2 (WSL2) and run the same commands as you would use on Linux [1].

### A.3.4 Starting a ZooKeeper server
The examples in this book use a single, local ZooKeeper server. The command in listing A.2 starts a single ZooKeeper server [2]. > **⚠ NOTE:**  that you’ll want to start ZooKeeper before you begin any Kafka brokers.

Listing A.2 Starting ZooKeeper
```shell
$ cd ~/kafka_2.13-2.7.1
$ bin/zookeeper-server-start.sh config/zookeeper.properties
```
### A.3.5 Creating and configuring a cluster by hand
The next step is to create and configure a three-node cluster. To create your Kafka cluster, you’ll set up three servers (brokers): server0, server1, and server2. We will modify the property files for each server [2].

Kafka comes with a set of predefined defaults. Run the commands in listing A.3 to create configuration files for each server in your cluster [2]. We will use the default server.properties file as a starting point. Then run the command in listing A.4 to open each configuration file and change the properties file [2].

Listing A.3 Creating mul> **⚠ TIP:** le Kafka brokers
```shell
$ cd ~/kafka_2.13-2.7.1                                  ❶
$ cp config/server.properties config/server0.properties
$ cp config/server.properties config/server1.properties
$ cp config/server.properties config/server2.properties
```
❶ After moving to your Kafka directory, makes three copies of the default server.properties file

> **⚠ NOTE:**  In our examples, we use vi as our text editor, but you can edit these files with a text editor of your choice.

Listing A.4 Configure server 0
```shell
$ vi config/server0.properties           ❶
 
broker.id=0
listeners=PLAINTEXT://localhost:9092
log.dirs= /tmp/kafkainaction/kafka-logs-0
 
$ vi config/server1.properties           ❷
 
broker.id=1
listeners=PLAINTEXT://localhost:9093
log.dirs= /tmp/kafkainaction/kafka-logs-1
 
$ vi config/server2.properties           ❸
broker.id=2
listeners=PLAINTEXT://localhost:9094
log.dirs= /tmp/kafkainaction/kafka-logs-2
```
❶ Updates id, port, and log directory for broker ID 0

❷ Updates id, port, and log directory for broker ID 1

❸ Updates id, port, and log directory for broker ID 2

> **⚠ NOTE:**  Each Kafka broker runs on its port and uses a separate log directory. It is also critical that each configuration file has a unique ID for each broker because each broker uses its own ID to register itself as a member of the cluster. You will usually see your broker IDs start at 0, following a zero-based array-indexing scheme.

After this, you can start each broker using the built-in scripts that are part of the initial installation (along with the configuration files that you updated in listing A.4). If you want to observe the Kafka broker output in the terminal, we recommend starting each process in a separate terminal tab or window and leaving them running. The following listing starts Kafka in a console window [2].

Listing A.5 Starting Kafka in a console window
```shell
$ cd ~/kafka_2.13-2.7.1                                  ❶
$ bin/kafka-server-start.sh config/server0.properties
$ bin/kafka-server-start.sh config/server1.properties
$ bin/kafka-server-start.sh config/server2.properties
```
❶ After moving to your Kafka directory, starts each broker process (3 total)

> **⚠ TIP:**  If you close a terminal or your process hangs, do not forget about running the jps command [4]. That command will help you find the Java processes you might need to kill.

Listing A.6 shows an example from one author’s machine where you can get the brokers’ PIDs and ZooKeeper’s JVM process label (QuorumPeerMain) in the output from the three brokers and the ZooKeeper instance. The process ID numbers for each instance are on the left and will likely be different each time you run the start scripts.

Listing A.6 jps output for ZooKeeper and three brokers
```shell
2532 Kafka              ❶
2745 Kafka              ❶
2318 Kafka              ❶
2085 QuorumPeerMain     ❷
```
❶ Kafka JVM process label and ID for each broker

❷ ZooKeeper JVM process label and ID

Now that you know how to configure a local installation manually, let’s look at using the Confluent Platform. Confluent Inc. (https://www.confluent.io/) offers the Confluent Platform, a platform based on Apache Kafka.

## A.4 Confluent Platform
The Confluent Platform (find more at https://www.confluent.io/) is an enterprise-ready packaging option that complements Apache Kafka with essential development capabilities. It includes packages for Docker, Kubernetes, Ansible, and various others. Confluent actively develops and supports Kafka clients for C++, C#/.NET, Python, and Go. It also includes the Schema Registry, which we talk about in chapters 3 and 11. Further, the Confluent Platform Community Edition includes ksqlDB. You learn about stream processing with ksqlDB in chapter 12.

Confluent also provides a fully managed, cloud-native Kafka service, which might come in handy for later projects. A managed service provides Apache Kafka experience without requiring knowledge on how to run it. This is a characteristic that keeps developers focused on what matters, which is coding. The Confluent version 6.1.1 download includes Apache Kafka version 2.7.1, which is used throughout this book. You can follow easy installation steps from official Confluent documentation at http://mng.bz/g1oV.

## A.4.1 Confluent command line interface (CLI)
Confluent, Inc. also has command line tools to quickly start and manage its Confluent Platform from the command line. A README.md on https://github.com/confluentinc/confluent-cli contains more details on the script usage and can be installed with instructions from http://mng.bz/RqNR. The CLI is helpful in that it starts mul> **⚠ TIP:** le parts of your product as needed.

## A.4.2 Docker
Apache Kafka doesn’t provide official Docker images at this time, but Confluent does. Those images are tested, supported, and used by many developers in production. In the repository of examples for this book, you’ll find a docker-compose.yaml file with preconfigured Kafka, ZooKeeper, and other components. To get all the components up and running, issue the command docker-compose up -d in the directory with the YAML file as the following listing shows.

> **⚠ NOTE:**  If you are unfamiliar with Docker or don’t have it installed, check out the official documentation at https://www.docker.com/get-started. You’ll find instructions on installation at that site as well.

Listing A.7 filename.sh for a Docker image
```shell
$ git clone \                                      ❶
  https://github.com/Kafka-In-Action-Book/Kafka-In-Action-Source-Code.git
$ cd ./Kafka-In-Action-Source-Code
$ docker-compose up -d                             ❷
 
Creating network "kafka-in-action-code_default" with the default driver
Creating Zookeeper... done                         ❸
Creating broker2   ... done
Creating broker1   ... done
Creating broker3   ... done
Creating schema-registry ... done
Creating ksqldb-server   ... done
Creating ksqldb-cli      ... done
 
$ docker ps --format "{{.Names}}: {{.State}}"      ❹
 
ksqldb-cli: running
ksqldb-server: running
schema-registry: running
broker1: running
broker2: running
broker3: running
zookeeper: running
```
❶ Clones a repository with book examples from GitHub

❷ Starts Docker Compose in the examples directory

❸ Observe the following output.

❹ Validates that all components are up and running

Runs a [Confluent Control Center](https://docs.confluent.io/platform/current/control-center/index.html) that exposes a UI at `http://localhost:9021/` .

We can follow the startup by monitoring the output :
```shell
docker-compose logs -f
```