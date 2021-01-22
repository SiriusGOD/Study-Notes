# **Maxwell 安裝步驟 (MySQL / MariaDB CDC)**

[![hackmd-github-sync-badge](https://hackmd.io/plZGACyhS1CHkYKCiyL7dw/badge)](https://hackmd.io/plZGACyhS1CHkYKCiyL7dw)

###### tags: `data stream` `CDC`
### 1.下載Maxwell
```
curl -sLo - https://github.com/zendesk/maxwell/releases/download/v1.26.2/maxwell-1.26.2.tar.gz \
       | tar zxvf -
```
## 2.設定MySQL / MariaDB 
在/etc/my.conf添加以下內容:

```
vi /etc/my.cnf
```
```
[mysqld]
server_id=1
log-bin=master
binlog_format=row
```
解釋：
MySQL必須開啟了binlogs,即log-bin指定了目錄,
binlog_format必須是row,
server_id指定mysql的全局唯一id
## 3.在修改mysql conf後，需要重啟mysql服務

```
[root@MobilePayment ~]#systemctl stop mysql 
[root@MobilePayment ~]#systemctl start mysql

```
## 4.權限配置
Maxwell需要儲存它自己的一些狀態數據，啟動參數schema_database選型來指定，默認是maxwell。
MySQL 用戶及權限配置（SQL） 創建maxwell用戶，並設置密碼為』123456』。
```
[root@MobilePayment ~]# mysql -u root
```
創建maxwell資料庫，存儲maxwell工具的一些狀態數據。
```
mysql> CREATE DATABASE IF NOT EXISTS maxwell default charset utf8 COLLATE utf8_general_ci;
```
給予用戶及權限配置，如果資料庫不在本地端
```
mysql> CREATE USER 'maxwell'@'%' IDENTIFIED BY '123456';
mysql> GRANT ALL ON maxwell.* TO 'maxwell'@'%';
mysql> GRANT SELECT, REPLICATION CLIENT, REPLICATION SLAVE ON *.* TO 'maxwell'@'%';
```
反之，資料庫在本地端
```
mysql> CREATE USER 'maxwell'@'localhost' IDENTIFIED BY '123456';
mysql> GRANT ALL ON maxwell.* TO 'maxwell'@'localhost';
mysql> GRANT SELECT, REPLICATION CLIENT, REPLICATION SLAVE ON *.* TO 'maxwell'@'localhost';
```
## 5.本地端執行Mexwell
```
[root@Mobile kite]# cd maxwell-1.26.2/
[root@Mobile kite]# bin/maxwell --user='maxwell' --password='123456' --host='127.0.0.1' --producer=stdout
```
執行成功如下圖:
![](https://i.imgur.com/2OrI83r.png)
當資料庫有操作時，Mexwell就會推送出JSON格式的資料
```
{
    "database":"stimulus_ticket",
    "table":"test_cus1",
    "type":"insert",
    "ts":1590548948,
    "xid":661,
    "commit":true,
    "data":{
        "cus_id":"123",
        "remain_money":500
    }
}
```

## 6.連接Kafka
當要連接Kafka時，請先確認主機的防火牆有把 9092 port 打開。
開啟防火牆9092port，並重啟防火牆。
```
[root@kafaka kafka]# firewall-cmd --zone=public --add-port=9092/tcp --permanent
[root@kafaka kafka]# firewall-cmd --reload
```
確認是否開啟成功
```
[root@kafaka kafka]# firewall-cmd --zone=public --query-port=9092/tcp
yes

```
進入maxwell資料夾下，並創建一個連線Kafka的設定檔
```
[root@MobilePayment Download]# cd maxwell-1.26.2/
[root@MobilePayment maxwell-1.26.2]# vi config.properties
```
設定檔內容如下(請依自身狀況修改):
```
log_level=info
producer=kafka
kafka.bootstrap.servers=192.168.1.190:9092
host=localhost
user=maxwell
password=123456
kafka_topic=mexwellcht
kafka.compression.type=snappy
kafka.metadata.fetch.timeout.ms=5000
kafka.retries=3
kafka.acks=all
kinesis_stream=maxwell
```
用設定檔執行Mexwell
```
[root@MobilePayment maxwell-1.26.2]# bin/maxwell --config config.properties
```
成功如下圖:
![](https://i.imgur.com/6EPJDqC.png)

