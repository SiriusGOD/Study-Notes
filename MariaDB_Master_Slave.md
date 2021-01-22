 # MariaDB安裝與設定Master Slave

[![hackmd-github-sync-badge](https://hackmd.io/l8rfzHX3TkCVtReksSAZqg/badge)](https://hackmd.io/l8rfzHX3TkCVtReksSAZqg)

 ###### tags: `DB`
###  1. 請先準備2台電腦一台要設定為Master，一台為Slave
###  2. 在2台電腦上安裝MariaDB
* 安裝 mariadb 與 mariadb-server 套件。
請注意mariadb版本號必須大於10.1版本，有些預設安裝原是舊版的，必須去官網依你的作業系統查詢安裝方式。
[MariaDB官網](https://mariadb.org/download/#entry-header)
```
$ sudo yum groupinstall mariadb mariasb-client -y
$ sudo yum install mariadb mariadb-server -y
```
* 啟動 MariaDB 服務。
```
$ sudo systemctl start mariadb
```
* 啟用 MariaDB 服務，讓 MariaDB 服務每次開機就會自動被帶起來。
```
$ sudo systemctl enable mariadb
Created symlink from /etc/systemd/system/multi-user.target.wants/mariadb.service to /usr/lib/systemd/system/mariadb.service.
```
* 檢視 MariaDB 服務啟動狀態。
```
$ sudo systemctl status mariadb
  ● mariadb.service - MariaDB database server
     Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disabled)
     Active: active (running) since Tue 2019-09-10 10:05:12 CST; 4min 57s ago
   Main PID: 2067 (mysqld_safe)
     CGroup: /system.slice/mariadb.service
             ├─2067 /bin/sh /usr/bin/mysqld_safe --basedir=/usr
             └─2229 /usr/libexec/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/plugin --log-error=/var/log/mariadb/mariadb.log --pid-file=/var/run/mariadb/mariadb.pid --socket=/var/lib/mysql/mysql.sock

  Sep 10 10:05:10 centos7-cli.lab.example.com systemd[1]: Starting MariaDB database server...
  Sep 10 10:05:10 centos7-cli.lab.example.com mariadb-prepare-db-dir[2035]: Database MariaDB is probably initialized in /var/lib/mysql already, nothing is done.
  Sep 10 10:05:10 centos7-cli.lab.example.com mariadb-prepare-db-dir[2035]: If this is not the case, make sure the /var/lib/mysql is empty before running mariadb-prepare-db-dir.
  Sep 10 10:05:10 centos7-cli.lab.example.com mysqld_safe[2067]: 190910 10:05:10 mysqld_safe Logging to '/var/log/mariadb/mariadb.log'.
  Sep 10 10:05:10 centos7-cli.lab.example.com mysqld_safe[2067]: 190910 10:05:10 mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql
  Sep 10 10:05:12 centos7-cli.lab.example.com systemd[1]: Started MariaDB database server.
```
* 加強 MariaDB 安裝的安全性(可自行考慮是否需要設定)。
```
$ sudo mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] Y
New password:
Re-enter new password:
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] Y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] Y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] Y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] Y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```
這段 script 主要在做以下動作：
1. 設定 root 帳號的密碼（此 root 是 MySQL 資料庫系統內的帳號，跟作業系統層的 root 無關）。
1. 禁止 root 從非本機登入。
1. 移除 anonymous 匿名者帳號。
1. 移除測試資料庫。



* 查版本。
```
$ mysql -V
mysql  Ver 15.1 Distrib 5.5.60-MariaDB, for Linux (x86_64) using readline 5.1
```
* 登入資料庫進行簡單測試。

```
$ mysql -u root -h localhost -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 3
Server version: 5.5.60-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.00 sec)

MariaDB [(none)]> exit
Bye
```
## 3.先到電腦A設定Master
* 修改MariaDB設定檔。
在 my.cnf ( CentOS 7 檔案位於 /etc/my.cnf , CentOS 8 檔案位於 /etc/my.cnf.d/mariadb-server.cnf ) 設定檔 [mysqld] 區塊加入以下與 Master replication 相關設定
```
[mysqld]
server_id=1
log-basename=master
log-bin
binlog-format=row
binlog-do-db=MasterA
```
* * server_id=1
主資料庫 Master 需設定為 1.
* * log-basename=master
* * log-bin
Master Slave Replication 主要是透過 Master 儲存會變動到資料的指令,包含 CREATE , ALTER TABLE , INSERT , UPDATE , DELETE … 到 Binlog 並與 Slave 的 Binlog 同步,Slave 再依據此 Binlog 執行指令來確保兩邊資料的同步.
這個 Binlog 可以透過 Linux 下的指令 mysqlbinlog 來查看內容.
```
mysqlbinlog /var/lib/mysql/mariadb-bin.000001
```
或是在 Mariadb 資料庫操作介面下指令
```
MariaDB [(none)]> show binlog events \G
```
* * binlog-format=row
記錄模式有三種 statement ， row ， mixed (沒特別研究).
* *  binlog-do-db=testdb
指定哪一個資料庫需要被同步，當有多個資料庫時就依序寫下資料庫名稱,如下所示。
```
binlog-do-db=MasterA
binlog-do-db=MasterB
```
沒設定時就會同步所有資料庫，或是使用下面忽略的方式來設定。

1. replicate_ignore_db=MasterA (忽略指定資料庫)
1. replicate_ignore_table=MasterA.table1 (忽略指定資料表)
* 重啟資料庫服務。
```
systemctl restart mariadb
```
* 登入資料庫系統，並新增剛剛要同步的資料庫 MasterA (如資料庫已存在請跳過這個步驟)。
```
[root@localhost ~]# mysql -u root -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 2
Server version: 5.5.64-MariaDB MariaDB Server
 
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
 
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
 
MariaDB [(none)]> CREATE DATABASE MasterA;
Query OK, 1 row affected (0.00 sec)
```
* 新增需要同步資料的使用者 repliuser 與其密碼。
```
MariaDB [(none)]> GRANT REPLICATION SLAVE ON *.* TO 'repliuser'@'%' IDENTIFIED BY '29057419';
MariaDB [(none)]> FLUSH PRIVILEGES;
```
* 查看log狀態，用於Slave連線Master時，所需輸入的設定。
```
MariaDB [(none)]> SHOW MASTER STATUS;
+-------------------+----------+--------------+------------------+
| File              | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+-------------------+----------+--------------+------------------+
| master-bin.000001 |      780 | MasterA      |                  |
+-------------------+----------+--------------+------------------+

```
* 防火牆開通3306 Port。
```
##Centos7 防火牆開啟埠號
firewall-cmd --zone=public --add-port=3306/tcp --permanent

#下面3行是引數說明
#–zone				        #作用域
#–add-port=80/tcp		#新增埠，格式為：埠/通訊協議
#–permanent			        #永久生效，沒有此引數重啟後失效

#重啟防火牆後看看是否生效
firewall-cmd --reload		#重啟firewall
firewall-cmd --list-ports	#檢視已經開放的埠


#如果想永久停止防火牆，執行下面操作
systemctl stop firewalld.service         #停止firewall
systemctl disable firewalld.service      #禁止firewall開機啟動

#檢視防火牆狀態
firewall-cmd --state		#檢視預設防火牆狀態（關閉後顯示notrunning，開啟後顯示running）
```
## 4.到電腦B設定Slave
* 修改MariaDB設定檔。
在 my.cnf ( CentOS 7 檔案位於 /etc/my.cnf ， CentOS 8 檔案位於 /etc/my.cnf.d/mariadb-server.cnf ) 設定檔 [mysqld] 區塊加入以下與 Slave replication 相關設定。
```
[mysqld]
skip_name_resolve = ON
server-id = 2
relay-log = slave-log
read_only = 1
report-port = 3306
report-host = 34.80.160.17
replicate-do-db = MasterA
replicate-do-db = MasterB
replicate_wild_do_table=MasterA.%
replicate_wild_do_table=MasterB.%
```
* * server_id=2
Slave 資料庫設定為 2 與 Master (為 1) 不相同即可。
* * replicate-do-db
指定哪一個資料庫需要同步，當有多個資料庫時就依序寫下資料庫名稱，如下所示
```
replicate-do-db = MasterA
replicate-do-db = MasterB
```
* * replicate_wild_do_table
指定需要同步的資料表，當有多個資料庫的資料表時就依序寫下資料庫名稱，如下所示
```
replicate_wild_do_table=MasterA.%
replicate_wild_do_table=MasterB.%
```
* 重啟資料庫服務。
```
systemctl restart mariadb
```
* 先確認Master 與 Slave需要同步的資料庫與資料表是否都存在雙方的MariaDB中，如果沒有請先建立或匯入。
* 連線進入MariaDB。
```
mysql -u root -p
```
* 創建Slave連線。
```
CHANGE MASTER 'MasterA' TO MASTER_HOST='59.124.115.145',
MASTER_USER='repliuser',
MASTER_PORT=3306,
MASTER_PASSWORD='29057419',
MASTER_LOG_FILE='master-bin.000001',
MASTER_LOG_POS=780;
```
* 起動Slave連線。
```
START SLAVE 'MasterA';
```
* 檢視一下 Slave 的狀態, Slave_IO_Running 與 Slave_SQL_Running 須為 Yes,才代表成功。
```
MariaDB [(none)]> show slave 'MasterA' status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 59.124.115.145
                  Master_User: repliuser
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mariadb-bin.000001
          Read_Master_Log_Pos: 780
               Relay_Log_File: mariadb-relay-bin.000002
                Relay_Log_Pos: 780
        Relay_Master_Log_File: mariadb-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: MasterA,MasterB
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: MasterA.%,MasterB.%
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 554
              Relay_Log_Space: 827
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 1
1 row in set (0.00 sec)
```
## 5.測試 MariaDB Replication
* Master
在 Master MasterA 資料庫新增 test 資料表，並新增一筆資料。
```
[root@localhost ~]# mysql -u root -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 6
Server version: 5.5.64-MariaDB MariaDB Server
 
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
 
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
 
MariaDB [(none)]> USE MasterA;
Database changed
MariaDB [MasterA]> CREATE TABLE test (ID INT);
Query OK, 0 rows affected (0.00 sec)
 
MariaDB [MasterA]> INSERT INTO test VALUES(1);
Query OK, 1 row affected (0.00 sec)
 
MariaDB [MasterA]> SELECT * FROM test;
+------+
| ID   |
+------+
|    1 |
+------+
1 row in set (0.00 sec)
```
* Slave
在 Slave 也會看到剛剛在 Master 新增的資料。
```
MariaDB [(none)]> USE MasterA;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A
 
Database changed
MariaDB [MasterA]> SELECT * FROM test;
+------+
| ID   |
+------+
|    1 |
+------+
1 row in set (0.00 sec)
```