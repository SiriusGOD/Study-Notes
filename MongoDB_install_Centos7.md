# Centos7 安裝MongoDB
###### tags: `DB` `NoSQL`
### 1. 建立檔案
在 /etc/yum.repos.d/mongodb-org-4.4.repo 建立檔案
```
vi /etc/yum.repos.d/mongodb-org-4.4.repo
```
寫入如下(此為4.4版)
```
[mongodb-org-4.4]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.4/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.4.asc
```
保存後並退出
### 2.安裝MongoDB
```
sudo yum install -y mongodb-org
```
### 3.操控MongoDB
#### 1.啟動
```
sudo systemctl start mongod
```
如果出現如下錯誤
```
Failed to start mongod.service: Unit mongod.service not found.
```
請先執行下方指令後，再啟動MongoDB
```
sudo systemctl daemon-reload
```
#### 2.設定開機啟動
```
sudo systemctl enable mongod
```
#### 3.查詢MongoDB狀態
```
sudo systemctl status mongod
```
#### 4.停止
```
sudo systemctl stop mongod
```
#### 5.重啟
```
sudo systemctl restart mongod
```
#### 6.登入mongoDB文字操作模式
```
mongo
```
### 4.刪除MongoDB
#### 1.停止MongoDB
```
sudo systemctl stop mongod
```
#### 2.移除MongoDB
```
sudo yum erase $(rpm -qa | grep mongodb-org)
```
#### 3.刪除MongoDB Log
```
sudo rm -r /var/log/mongodb
sudo rm -r /var/lib/mongo
```

參考來源
[官網](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/)