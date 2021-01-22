# **SSL憑證購買與安裝流程**

[![hackmd-github-sync-badge](https://hackmd.io/MdrBYrtKSleQf1GnksRwUQ/badge)](https://hackmd.io/MdrBYrtKSleQf1GnksRwUQ)

###### tags: `web`
### 分為兩種情況
### 情況一 :  已經有買過憑證(憑證過期)
1. (測試機) 進入某個資料夾位置
```	
cd /home/kelly/request_ssl/
```
2. 產生憑證請求檔(CSR)與金鑰檔(key)
```
openssl req -new -newkey rsa:2048 -nodes -sha256 -out www.easycook888.com.csr -keyout easycook888.com.key -subj "/C=TW/ST=Taiwan/L=Taipei/O=TingYu Network Technology Co., Ltd./OU=MIS/CN=www.easycook888.com"
```
3. 去Webnic網站(https://www.webnic.cc/)購買SSL憑證 PS(使用CSR)
4. 購買後網站會驗證網址，驗證方式常用為email驗證與filebase驗證,如果easycook888.com沒有指到我們的IP，就必須使用email驗證，讓網域持有者按確認，反之則可使用filebase驗證。 下載fileauth.txt檔案放到 /usr/share/nginx/html/.well-known/pki-validation/
```
mv fileauth.txt /usr/share/nginx/html/.well-known/pki-validation/
```
  *PS(要放驗證檔時，要先確認easycook888.com是否有指到正式機，簡單測試方法: 使用小黑視窗去ping該網址，確認IP是否為正式機IP)*

5. Webnic網站會驗證，驗證完後會寄信裡面有憑證檔。
6. 要再去下載中繼憑證檔(CA)與根憑證檔(CA_ROOT)並與原始憑證(CRT)合併為(CER)
*PS(順序為 CRT — CA — CAROOT)*
7. 把CER檔案與步驟2產生的金鑰檔放入/etc/nginx/ssl/
```
mv www.easycook888.com.cer /etc/nginx/ssl/
mv www.easycook888.com.key /etc/nginx/ssl/
```
8. 重新讀取設定檔 nginx
```
nginx -s reload
```
9. 開啟網站確認憑證是否安裝完成
10. 結束!!

### 情況二 : 新網站(第一次購買憑證)
1. 接著上述步驟7後要到/etc/nginx/sites-enabled/ 去新增一個設定檔
 *PS(用於設定轉導資料夾位置與憑證檔位置)*
```
cd /etc/nginx/sites-enabled
vi www.easycook888.com.conf
```
ex.
```
server {
    server_name ~^(www.)?easycook888.com;
location = /website/ {
       try_files $uri $uri/ =404;
       if ($http_host ~* "^(www.)?easycook888.com$"){
         rewrite ^(.*)$ /easycook888/index.php redirect;
       }
}
    include /etc/nginx/default_setting.conf;
    #include /etc/nginx/default_ssl.conf;
    ssl_certificate /etc/nginx/ssl/www.easycook888.com.cer;
    ssl_certificate_key /etc/nginx/ssl/www.easycook888.com.key;
}
```
2. 測試www.easycook888.com.conf檔案語法是否有錯誤
```
nginx -t
```
3. 重新讀取設定檔 nginx
```
nginx -s reload
```
4. 開啟網站確認憑證是否安裝完成
5. 結束!!