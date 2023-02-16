# Wekan使用Docker安裝教學

[![hackmd-github-sync-badge](https://hackmd.io/SIr5nxXwRA-m-RX58IPTWQ/badge)](https://hackmd.io/SIr5nxXwRA-m-RX58IPTWQ)


###### tags: `Docker` `Wekan` 
Trello 作為專業的專案管理軟體，在開源的世界中也會隨之誕生一些類似操作的工具。今天要簡介的 Wekan 可以說是其中較為流行的，如果你曾用過 Trello ，像是要上手並不是件難事。
使用docker-compose.yml 安裝

## docker-compose.yml
#### 1. 先使用指令從[github](https://github.com/wekan/wekan)上下載docker-compose.yml檔案
```
wget https://github.com/wekan/wekan/blob/master/docker-compose.yml
```

#### 2. 修改docker-compose.yml設定
修改對應的port，如下
```
ports:
      # Docker outsideport:insideport. Do not add anything extra here.
      # For example, if you want to have wekan on port 3001,
      # use 3001:8080 . Do not add any extra address etc here, that way it does not work.
      # remove port mapping if you use nginx reverse proxy, port 8080 is already exposed to wekan-tier network
      - 80:8080  #   <=== 修改這裡
```
由 80port 改為 9020
![](https://i.imgur.com/09sMJAs.png)

修改ROOT_URL並加上對應的port
```
environment:
      #---------------------------------------------------------------
      # ==== ROOT_URL SETTING ====
      # Change ROOT_URL to your real Wekan URL, for example:
      # If you have Caddy/Nginx/Apache providing SSL
      #  - https://example.com
      #  - https://boards.example.com
      # This can be problematic with avatars https://github.com/wekan/wekan/issues/1776
      #  - https://example.com/wekan
      # If without https, can be only wekan node, no need for Caddy/Nginx/Apache if you don't need them
      #  - http://example.com
      #  - http://boards.example.com
      #  - http://192.168.1.100    <=== using at local LAN
      - ROOT_URL=http://localhost  #   <=== 修改這裡
      #---------------------------------------------------------------
```

由 http://localhost 改為 http://<主機IP>:9020，如下圖
![](https://i.imgur.com/UreP1sy.png)

## 步驟
#### 1.執行啟動指令
```
docker-compose up -d
```
上述指令執行成功後執行docker ps 就會看到2台docker啟動
```
docker ps
```
![](https://i.imgur.com/2xLJVT4.png)

#### 2.開啟網站
輸入 http://<遠端主機的 IP 位置>:<對外 port>，你就會看到以下登入畫面
![](https://i.imgur.com/H9Vaf86.png)

參考網址：
[Wekan Github](https://github.com/wekan/wekan)
[Day 25 似 Trello 的開源看板管理工具 - Weka](https://ithelp.ithome.com.tw/articles/10279889)
