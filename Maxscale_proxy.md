# MaxScale 讀寫分離教學

[![hackmd-github-sync-badge](https://hackmd.io/8HyTLPkkRa6ZegfrvEyP8w/badge)](https://hackmd.io/8HyTLPkkRa6ZegfrvEyP8w)

###### tags: `proxy` `DB` `MariaDB` 
## 1. 環境準備
```
system version: CentOS 7
mariadb version: 10.2
maxscale version: 2.5.6 GA
mariadb master: 172.17.0.4 (143底下的docker)
mariadb slave1: 172.17.0.5 (143底下的docker)
mariadb slave2: 172.17.0.6 (143底下的docker)
maxscale proxy: 192.168.1.143
```

## 2.創建 MariaDB Master 的 Docker 自訂my.cnf文件
```
docker run -d -P --name Mxs_Master -v /home/my/mx_master:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=29057419 -d mariadb:10.2
```
自訂my.cnf文件如下(檔案儲存路徑/home/my/mx_master):
```
cd /home/my/mx_master
vi config-file.cnf

[mysqld]
log-bin=mariadb-bin
server_id=200
binlog_format=row
```
## 3.創建2台 MariaDB Slave 的 Docker
```
docker run -d -P --name Mxs_Slave_01 -e MYSQL_ROOT_PASSWORD=29057419 mariadb:10.2
docker run -d -P --name Mxs_Slave_02 -e MYSQL_ROOT_PASSWORD=29057419 mariadb:10.2
```

## 4.Docker 運行結果
```
docker ps -a

5e6db7d41b0d        mariadb:10.2        "docker-entrypoint..."   2 days ago          Up 47 hours         0.0.0.0:1040->3306/tcp   Mxs_Master
1ea610bcbdbc        mariadb:10.2        "docker-entrypoint..."   2 days ago          Up 2 days           0.0.0.0:1038->3306/tcp   Mxs_Slave_02
679941f787da        mariadb:10.2        "docker-entrypoint..."   2 days ago          Up 2 days           0.0.0.0:1037->3306/tcp   Mxs_Slave_01

```
會出現如下圖所示:
![](https://i.imgur.com/snGC0GZ.png)

## 5.查詢Master BinLog的日誌偏移量
```
# 進入docker
doceker exec -it Mxs_Master bash

# 進入DB
mysql -u root -p 

# 添加用戶slave授予遠程連接的權限,供從節點複製binlog
GRANT REPLICATION SLAVE ON *.* TO 'slave'@'10.10.110.%' IDENTIFIED BY '123456';

# 查看主庫的binlog記錄日誌信息偏移量position
mariadb [(none)]> show master status;
+--------------------+----------+--------------+------------------+
| File               | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+--------------------+----------+--------------+------------------+
| mariadb-bin.000001 |      529 |              |                  |
+--------------------+----------+--------------+------------------+
```

## 6.配置2台Slave 
```
# 進入docker
doceker exec -it Mxs_Slave_01 bash
doceker exec -it Mxs_Slave_02 bash

# 進入DB
mysql -u root -p 

mariadb [(none)]> change master to master_host='172.17.0.4',
    -> master_user='slave',
    -> master_password='123456',
    -> master_port=3306,
    -> master_use_gtid=current_pos,
    -> master_connect_retry=30;
Query OK, 0 rows affected (0.017 sec)

# 啟動slave線程,若要更改指定的主庫信息,需先執行stop slave,修改完成後執行start slave
mariadb [(none)]> start slave;
Query OK, 0 rows affected (0.003 sec)

# 查看slave狀態(slave_IO_Running和slave_SQL_Running都為Yes狀態)
mariadb [(none)]> show slave status\G
*************************** 1. row ***************************
                slave_IO_State: Waiting for master to send event
                   master_Host: 10.10.110.80
                   master_User: slave
                   master_Port: 53306
                 Connect_Retry: 30
               master_Log_File: mariadb-bin.000001
           Read_master_Log_Pos: 529
                Relay_Log_File: localhost-relay-bin.000002
                 Relay_Log_Pos: 830
         Relay_master_Log_File: mariadb-bin.000001
              slave_IO_Running: Yes
             slave_SQL_Running: Yes
               Replicate_Do_DB: 
           Replicate_Ignore_DB: 
            Replicate_Do_Table: 
        Replicate_Ignore_Table: 
       Replicate_Wild_Do_Table: 
   Replicate_Wild_Ignore_Table: 
                    Last_Errno: 0
                    Last_Error: 
                  Skip_Counter: 0
           Exec_master_Log_Pos: 529
               Relay_Log_Space: 1143
               Until_Condition: None
                Until_Log_File: 
                 Until_Log_Pos: 0
            master_SSL_Allowed: No
            master_SSL_CA_File: 
            master_SSL_CA_Path: 
               master_SSL_Cert: 
             master_SSL_Cipher: 
                master_SSL_Key: 
         Seconds_Behind_master: 0
 master_SSL_Verify_Server_Cert: No
                 Last_IO_Errno: 0
                 Last_IO_Error: 
                Last_SQL_Errno: 0
                Last_SQL_Error: 
   Replicate_Ignore_Server_Ids: 
              master_Server_Id: 80
                master_SSL_Crl: 
            master_SSL_Crlpath: 
                    Using_Gtid: Current_Pos
                   Gtid_IO_Pos: 0-80-1
       Replicate_Do_Domain_Ids: 
   Replicate_Ignore_Domain_Ids: 
                 Parallel_Mode: optimistic
                     SQL_Delay: 0
           SQL_Remaining_Delay: NULL
       slave_SQL_Running_State: slave has read all relay log; waiting for more updates
              slave_DDL_Groups: 1
slave_Non_Transactional_Groups: 0
    slave_Transactional_Groups: 0
1 row in set (0.001 sec)
```
這時理論上mastser與slave已經建立好連線。

## 7. 安裝MaxScale
```
wget https://dlm.mariadb.com/1495309/MaxScale/2.5.6/packages/rhel/7/maxscale-2.5.6-1.rhel.7.x86_64.rpm
yum -y install maxscale-2.5.6-1.rhel.7.x86_64.rpm
```

### 在Master上創建需要使用的帳號
在開始配置之前，需要在 mariadb master 中為 maxscale 創建兩個用戶，用於 maxscale 的監控模塊和路由模塊

monitor_user：該賬號監控集群狀態，如果發現某個從服務器複製線程停掉了，那麼就不向其轉發請求了
```
# 創建監控用戶,用於[MariaDB-Monitor]配置
CREATE USER 'monitor_user'@'%' IDENTIFIED BY '123456';
GRANT REPLICATION CLIENT ON *.* TO 'monitor_user'@'%';
# 如果使用 MariaDB Monitor 的自動故障轉移，用戶將需要額外的授權
GRANT SUPER, RELOAD ON *.* TO 'monitor_user'@'%';
```
routing_user：該賬號將不同的請求分發到不同的節點上，當客戶端連接到maxscale 這個節點上時，maxscale 節點會使用該賬號去查後端數據庫，檢查客戶端登陸的用戶是否有權限或密碼是否正確等等
```
# 創建routing user,用於[Read-Write-Service]配置
CREATE USER 'routing_user'@'%' IDENTIFIED BY '123456';
GRANT SELECT ON mysql.user TO 'routing_user'@'%';
GRANT SELECT ON mysql.db TO 'routing_user'@'%';
GRANT SELECT ON mysql.tables_priv TO 'routing_user'@'%';
GRANT SELECT ON mysql.columns_priv TO 'routing_user'@'%';
GRANT SELECT ON mysql.proxies_priv TO 'routing_user'@'%';
GRANT SELECT ON mysql.roles_mapping TO 'routing_user'@'%';
GRANT SHOW DATABASES ON *.* TO 'routing_user'@'%';
```
### 配置加密密碼
我們創建的數據庫用戶資料需要填寫到 maxscale 配置文件中，為了防止配置文件出現明文密碼，我們可以使用秘鑰為密碼加密，然後將加密後的字符串填寫在 maxscale 配置文件中
```
# 生成秘鑰,密鑰將保存到/var/lib/maxscale/.secrets
maxkeys
# 基於秘鑰生成123456加密後的字符串(記錄下來)
maxpasswd /var/lib/maxscale/ 123456
```
### Maxscale 配置文件
```
[server1]
type=server
address=172.17.0.4
port=3006
protocol=MariaDBBackend

[server2]
type=server
address=172.17.0.5
port=3006
protocol=MariaDBBackend

[server3]
type=server
address=172.17.0.6
port=3006
protocol=MariaDBBackend

[MariaDB-Monitor]
type=monitor
module=mariadbmon
servers=server1,server2,server3
user=routing_user
password=62C6107D704B862E9C345994189286CC9A2787E62F876F5E7D6E5340C24E3143
monitor_interval=2000

[Read-Write-Service]
type=service
router=readwritesplit   
servers=server1,server2,server3
user=routing_user
password=2C6107D704B862E9C345994189286CC9A2787E62F876F5E7D6E5340C24E3143

[Read-Write-Listener]
type=listener
service=Read-Write-Service    
protocol=MariaDBClient
port=4006

```
### 啟動 Maxscale 服務
```
systemctl start maxscale.service
```

### Maxctrl 管理工具的使用
```
maxctrl -h 192.168.1.143:8080 list servers
┌─────────┬────────────┬──────┬─────────────┬─────────────────┬────────────┐
│ Server  │ Address    │ Port │ Connections │ State           │ GTID       │
├─────────┼────────────┼──────┼─────────────┼─────────────────┼────────────┤
│ server1 │ 172.17.0.4 │ 3306 │ 0           │ Master, Running │ 0-200-7282 │
├─────────┼────────────┼──────┼─────────────┼─────────────────┼────────────┤
│ server2 │ 172.17.0.5 │ 3306 │ 0           │ Slave, Running  │ 0-200-7282 │
├─────────┼────────────┼──────┼─────────────┼─────────────────┼────────────┤
│ server3 │ 172.17.0.6 │ 3306 │ 0           │ Slave, Running  │ 0-200-7282 │
└─────────┴────────────┴──────┴─────────────┴─────────────────┴────────────
```
### Maxscale 測試讀寫分離
```
# 分別在兩個Slave上創建數據
create database slave;
use slave;
create table info(name varchar(25),ip int);
insert into slave.info values("slave",inet_aton('172.17.0.5'));

create database slave;
use slave;
create table info(name varchar(25),ip int);
insert into slave.info values("slave",inet_aton('172.17.0.6'));

# 在Master上創建測試用戶
grant all on *.* to 'check'@'%' identified by '123456';

# 連接maxscale查詢數據驗證讀寫分離(讀操作自動負載均衡)
mysql -ucheck -p123456 -P4006 -h 192.168.1.143

MariaDB [(none)]> select name,inet_ntoa(ip) from slave.info;
+-------+---------------+
| name | inet_ntoa(ip) |
+-------+---------------+
| slave | 172.17.0.6 |
+-------+---------------+
1 row in set (0.002 sec)

MariaDB [(none)]> select name,inet_ntoa(ip) from slave.info;
+-------+---------------+
| name | inet_ntoa(ip) |
+-------+---------------+
| slave | 172.17.0.5 |
+-------+---------------+
1 row in set (0.001 sec)
```
驗證讀寫分離的 “寫” 操作是否在Master上
```
# 連接maxscale往裡寫數據,看slave上數據有沒有同步過來
create database test;
use test;
create table test(name varchar(25),city varchar(30),age int);
insert into test.test values("mariadb","china",11);
```