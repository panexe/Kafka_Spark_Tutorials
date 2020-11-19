# Setup Kafka Server
### 1.  Install Ubuntu (this tutorial uses 20.04 LTS)
### 2.  Create non root user with sudo priviliges
Create User 
````
$ sudo adduser <username
````
Give sudo access
````
$ sudo usermod -aG sudo <username>
````
Change user to the created one
````
$ su <username>
````
  
### 3. Install Kafka & dependencies
Install Java
````
$ sudo apt-get update && sudo apt-get upgrade
$ sudo apt-get install default-jre
````
Download Kafka & the PGP Signature (KEYS file and .asc file)
( this tutorial uses Kafka 2.6.0 Scala 2.13, you can find the latest version of kafka [here](https://downloads.apache.org/kafka/2.6.0/kafka_2.13-2.6.0.tgz.asc) )
````
$ cd ~/Downloads/
$ wget https://downloads.apache.org/kafka/KEYS 
$ wget https://downloads.apache.org/kafka/2.6.0/kafka_2.13-2.6.0.tgz.asc
$ wget https://downloads.apache.org/kafka/2.6.0/kafka_2.13-2.6.0.tgz
````
Verify the downloaded file
````
$ gpg --import KEYS
$ gpg --verify kafka_2.13-2.6.0.tgz.asc kafka_2.13-2.6.0.tgz
````
If the verification was succesful you can carry on

Extract Kafka and create directory for the logs
````
$ tar -zxf kafka_2.13-2.6.0.tgz
$ sudo mv kafka_2.13-2.6.0.tgz /usr/local/kafka
$ sudo mkdir /tmp/kafka-logs 
````

### 4. Start Zookeeper & Kafka 
Start zookeeper as a daemon
````
$ ./bin/zookeeper-server-start.sh -daemon ./config/zookeeper.properties
````
Now we start our Kafka brokers. If you have them running on multiple machines you need to use the address of your host, that runs zookeeper ([ip-address]:2181) for the zookeper-argument. For each broker you need a seperate config file. In the config file, the *broker-id*, the *host ip/port* and *logs-dir* should be changed. 

Copy and edit a config file
````
$ cp /usr/local/kafka/config/server.properties /usr/local/kafka/config/server-1.properties 
$ sudo nano /usr/local/kafka/config/server-1.properties
````
In "server-1.properties" :
````
...
broker.id = 1
... 
listeners=PLAINTEXT://<ip-address>:9093
... 
log.dirs=/tmp/kafka-logs-1/
...
````
Start the broker itself 
````
$ /usr/local/kafka/bin/kafka-server-start.sh -daemon /usr/local/kafka/config/server.properties 
````
You can repeat the last steps for as many brokers as you want. 
To start multiple brokers at once or in general make starting Kafka easier, you could create a shell script that starts zookeeper & all brokers and/or use *systemctl/systemd*. 

To stop the kafka and zookeeper servers use: 
````
$ /usr/local/kafka/bin/kafka-server-stop.sh
$ /usr/local/kafka/bin/zookeeper-server-stop.sh 
````

### 5. Create a topic 
To create a topic your kafka instances must already be running. The script `kafka-topics.sh` thats used to create a topic takes a few arguments:
- --create : create a new topic 
- --describe : get information about an existing topic
- --delete : delete an existing topic 
- --zookeeper [string: ip:port ] : give the location of the zookeeper instance
- --replication-factor : number of replications for this topic, must be <= number of brokers
- --partitions : number of partitions 
- --topic : name of the topic to be created, altered or deleted

Example:
````
$ /usr/local/kafka/bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 1 --topic myTopic 
````

### 6 Test your Kafka instance 
Kafka comes with to a console-producer and console-consumer script to test 


