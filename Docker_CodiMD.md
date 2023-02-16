# CodiMD(HackMD)使用Docker安裝教學

[![hackmd-github-sync-badge](https://hackmd.io/Y3TV4sABT3eZzGz92EoJpg/badge)](https://hackmd.io/Y3TV4sABT3eZzGz92EoJpg)


###### tags: `Docker` `HackMD` `CodiMD` `Nginx` 
網路上的即時文件協作中，除了 Google Documents 系列外，HackMD 也走出了一條自己的路。它採用 Markdown 語法，用起來非常的順手。CodiMD是其開源的版本。

## docker-compose.yml
#### 1. 創建docker-compose.yml檔案
```
version: "3"
services:
  database:
    image: postgres:11.6-alpine
    environment:
      - POSTGRES_USER=<DB_USER_NAME> <=== 自訂DB User Name
      - POSTGRES_PASSWORD=<DB_PASSWORD> <=== 自訂DB User Password
      - POSTGRES_DB=<DB_NAME> <=== 自訂DB Name
    volumes:
      - "database-data:/var/lib/postgresql/data"
    restart: always
  codimd:
    image: hackmdio/hackmd:2.4.2
    environment:
      - CMD_DB_URL=postgres://<DB_USER_NAME>:<DB_PASSWORD>@database/<DB_NAME>
      - CMD_USECDN=false
      - CMD_PROTOCOL_USESSL=true
    depends_on:
      - database
    ports:
      - "9010:3000" <=== 修改前端port 9010
    volumes:
      - upload-data:/home/hackmd/app/public/uploads
    restart: always
volumes:
  database-data: {}
  upload-data: {}
```

## 步驟
#### 1.執行啟動指令
```
docker-compose up -d
```
上述指令執行成功後執行docker ps 就會看到2台docker啟動
```
docker ps
```
![](https://i.imgur.com/xU6IimI.png)
#### 2.開啟網站
輸入 http://<遠端主機的 IP 位置>:<對外 port>，你就會看到以下登入畫面
![](https://i.imgur.com/AaZOcCe.png)

#### 3.Nginx
可以用Nginx進行反向代理
##### Configuration Example
```
# setup a upstream point to CodiMD server
upstream @codimd {
    server 127.0.0.1:3000;
    keepalive 300;
}

# for socket.io (http upgrade)
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

# redirect all http traffic to https
server {
    listen 80;
    server_name codimd.example.com;
    return 301 https://$server_name$request_uri;
}

# https server
server {
    listen 443 ssl http2;
    server_name codimd.example.com;
    
    # setup certificate
    ssl_certificate /etc/ssl/codimd.example.com.full.crt;
    ssl_certificate_key /etc/ssl/codimd.example.com.key;

    location / {
      proxy_http_version 1.1;
      
      # set header for proxy protocol
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection $connection_upgrade;
      
      # setup for image upload
      client_max_body_size 8192m;
      
      # adjust proxy buffer setting
      proxy_buffers 8 32k; 
      proxy_buffer_size 32k; 
      proxy_busy_buffers_size 64k;

      proxy_max_temp_file_size 8192m;
      
      proxy_read_timeout 300;
      proxy_connect_timeout 300;
      proxy_pass http://@codimd;
    }
}
```


參考網址：
[CodiMD GitHub](https://github.com/hackmdio/codimd)
[CodiMD Documentation](https://hackmd.io/c/codimd-documentation/%2Fs%2Fcodimd-documentation)
[Day 20 架設開源的 CodiMD 服務](https://ithelp.ithome.com.tw/articles/10277695)

