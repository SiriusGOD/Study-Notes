# Nginx 設定多版本PHP (Mac)
###### tags: `web` `Nginx`
## 前言
請先安裝各個版本PHP，安裝方式可以參考這篇
https://hackmd.io/@WL-WTIRiRlOr-R2wORqerA/S1sFUaTsn

### 切換PHP版本方式
```
brew unlink php@7.0
brew link php@8.0
```
### 更改各个版本php-fpm監聽的port
```
vi /usr/local/etc/php/7.0/php-fpm.d/www.conf
vi /usr/local/etc/php/8.0/php-fpm.d/www.conf
```
![](https://hackmd.io/_uploads/BkXFHs0R3.png)

重啟php-fpm
```
brew services restart php@8.0
```
## 安装配置nginx
### 安装nginx
```
brew install nginx
```
nginx的設定檔所在目錄為

> /usr/local/etc/nginx

該目錄下的nginx.conf 引入了./servers下的所有設定檔
```
cd /usr/local/etc/nginx/servers
vi test1.conf # 創建兩個網站測試
vi test2.conf
```
test1.conf
```
server {
    listen       80;
    server_name  test71.com;

    root /Users/kelly/web/test71;
    index index.html index.php;

    #charset koi8-r;

    #access_log  logs/host.access.log  main;


    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }

    location ~ \.php$ {
            # 将將php請求轉發到9071端口，也就是php7.1進行處理
            root           /Users/kelly/web/test71/;
            fastcgi_pass   127.0.0.1:9071;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include        fastcgi_params;
    }
```
test2.conf 同理
```
server {
    listen       80;
    server_name  test80.com;

    root /Users/kelly/web/test80;
    index index.html index.php;

    #charset koi8-r;

    #access_log  logs/host.access.log  main;


    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }

    location ~ \.php$ {
            # 将將php請求轉發到9071端口，也就是php7.1進行處理
            root           /Users/kelly/web/test80/;
            fastcgi_pass   127.0.0.1:9080;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include        fastcgi_params;
    }
```
使用nginx -t 测试配置文件是否正确

![](https://hackmd.io/_uploads/ByI3e3ACh.png)

重啟nginx
```
brew services start nginx
# 重新加載設定檔
brew services reload nginx
```