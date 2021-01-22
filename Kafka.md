# Kafka 安裝步驟

[![hackmd-github-sync-badge](https://hackmd.io/8V9VcBJCQw2MNusaYjDERQ/badge)](https://hackmd.io/8V9VcBJCQw2MNusaYjDERQ)

###### tags: `data stream`
## 1.Kafka需要Java JDK8以上，所以要先安裝Java
```
[root@Mobile maxwell-1.26.2]# sudo yum install java-11-openjdk-devel
```
查看Java版本號指令
```
[root@Mobile maxwell-1.26.2]# java -version
openjdk version "11.0.7" 2020-04-14 LTS
OpenJDK Runtime Environment 18.9 (build 11.0.7+10-LTS)
OpenJDK 64-Bit Server VM 18.9 (build 11.0.7+10-LTS, mixed mode, sharing)
```
## 2.下載Kafka
```
curl http://apache.stu.edu.tw/kafka/2.5.0/kafka_2.13-2.5.0.tgz -o /root/Downloads/kafka.tgz
tar -xvzf Downloads/kafka.tgz --strip 1
```
## 3.修改Kafka設定檔
```
[root@Mobile maxwell-1.26.2]# cd config
[root@Mobile maxwell-1.26.2]# vi server.properties
```
請依自己狀況去修改參數
```
listeners=PLAINTEXT://192.168.1.190:9092
advertised.listeners=PLAINTEXT://192.168.1.190:9092
zookeeper.connect=192.168.1.190:2181
advertised.host.name=192.168.1.190
```
## 4.啟動Zookeeper
要啟動Kafka前必須先啟動Zookeeper
```
bin/zookeeper-server-start.sh config/zookeeper.properties
```
## 5.啟動Kafka
```
 bin/kafka-server-start.sh config/server.properties
```
## 6.Kafka常用指令

創建Topic
```
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic mexwell
```
呼叫範例消費者(Consumer)
```
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic mexwell --from-beginning
```
呼叫範例生產者(Producer)
```
echo "112233" | /home/kafka/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic mexwellcht > /dev/null
```
查詢Kafka內所有的Topic清單
```
bin/kafka-topics.sh --zookeeper 127.0.0.1:2181 --list
```
查詢Kafka內特定Topic的基本資訊
```
bin/kafka-topics.sh --zookeeper 127.0.0.1:2181 --topic Your_topic --describe
```
查詢Kafka內所有的group清單
```
bin/kafka-consumer-groups.sh  --bootstrap-server 127.0.0.1:9092 --list
```
查詢Kafka某一group的資訊
```
bin/kafka-consumer-groups.sh --bootstrap-server 127.0.0.1:9092 --describe --group Your_group
```
删除topic，慎用，只會删除zookeeper中的資料，Log需手動刪除
```
bin/kafka-run-class.sh kafka.admin.DeleteTopicCommand --zookeeper node01:2181 --topic t_cdr
```