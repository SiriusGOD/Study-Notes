# Apache設定不同網域使用不同PHP Version (Mac)
## 注意
以下範例是在MAC裡安裝Xampp的環境，其他環境或非使用Xammp，請自行修改檔案路徑～
## 流程
### 安裝PHP與切換版本方式
1. 安裝Homebrew
```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
2. 安裝PHP版本
```
brew tap shivammathur/php
brew search php
brew install shivammathur/php/php@7.0
brew install shivammathur/php/php@8.0
```
3. 建立軟連結
```
ln -s /usr/local/opt/php@7.0/bin/php /Applications/XAMPP/xamppfiles/bin/php7.0
ln -s /usr/local/opt/php@8.0/bin/php /Applications/XAMPP/xamppfiles/bin/php8.0
```
4. 第一種切換PHP方式
```
// 切換到 PHP 7.0
sudo mv /Applications/XAMPP/xamppfiles/bin/php /Applications/XAMPP/xamppfiles/bin/php_default
sudo ln -s /Applications/XAMPP/xamppfiles/bin/php7.0 /Applications/XAMPP/xamppfiles/bin/php
// 切換到 PHP 8.0
sudo mv /Applications/XAMPP/xamppfiles/bin/php /Applications/XAMPP/xamppfiles/bin/php_default
sudo ln -s /Applications/XAMPP/xamppfiles/bin/php8.0 /Applications/XAMPP/xamppfiles/bin/php
```
5. 第二種切換PHP方式
```
brew unlink php@7.0
brew link php@8.0
```

### Xampp 裡的 Apache 設定

1. 設定第二個php的php-fpm監聽另一port與產生通訊通道
```
vi /usr/local/etc/php/8.0/php-fpm.d/www.conf
listen = 127.0.0.1:9080
listen = /usr/local/var/run/php80.sock
```
![](https://hackmd.io/_uploads/rJd0t50A2.png)

2. 重啟php-fpm
```
brew services restart php@8.0
```
重啟後會在目錄下看見php80.sock
```
cd /usr/local/var/run/
ls
```
![](https://hackmd.io/_uploads/Bypkc9AAh.png)

3. 修改xampp設定
```
vi /Applications/XAMPP/xamppfiles/etc/httpd.conf
```
新增對外監聽port
```
Listen 8080
```
![](https://hackmd.io/_uploads/S1il9c00n.png)

確認下列模組是否安裝或開啟
```
LoadModule proxy_fcgi_module libexec/mod_proxy_fcgi.so
LoadModule proxy_module libexec/mod_proxy.so
LoadModule vhost_alias_module libexec/mod_vhost_alias.so
LoadModule rewrite_module libexec/mod_rewrite.so
```
![](https://hackmd.io/_uploads/S1PNq9C0h.png)


開啟vhosts
![](https://hackmd.io/_uploads/HJOS9qAR3.png)

4. 修改vhost設定
```
vi /Applications/XAMPP/xamppfiles/etc/extra/httpd-vhosts.conf
```
第一部分： 設定php70.local走 <span style="color:red;">80 port</span> , 並走預設 <span style="color:red;">9000 port</span> 的php-fpm(php7.0)
第二部分： 設定php80.local走 <span style="color:red;">8080 port</span> , 並轉導到監聽 <span style="color:red;">9080 port</span> 的php-fpm(php8.0)
```
<VirtualHost *:80>
    DocumentRoot "/Users/{username}/web/php70"
    ServerName php70.local
    <Directory "/Users/{username}/web/php70/">
        Require all granted
        Options All
        AllowOverride All
        Order deny,allow
        Allow from all
    </Directory>
</VirtualHost>
<VirtualHost *:8080>
    DocumentRoot "/Users/{username}/web/php80/"
    ServerName php80.local
    <Proxy "fcgi://127.0.0.1:9080">
        ProxySet timeout=3600
    </Proxy>
    <Directory "/Users/{username}/web/php80/">
        Options Indexes FollowSymlinks MultiViews
        AllowOverride All
        Order allow,deny
        Allow from all
        Require all granted
    </Directory>
    <FilesMatch \.php$>
        SetHandler "proxy::unix:/usr/local/var/run/php80.sock|fcgi://127.0.0.1:9080"
    </FilesMatch>
</VirtualHost>
``` 
5. 在專案目錄下建立index.php
```
vi /Users/{username}/web/php70/index.php
vi /Users/{username}/web/php80/index.php
<!DOCTYPE html>
    <?php echo phpinfo();?>
</html>
```
6. 修改host, 新增網域對應IP
```
vi /etc/hosts
127.0.0.1       php70.local
127.0.0.1       php80.local
```
![](https://hackmd.io/_uploads/ryt8qcR0n.png)

7. 重啟apache
8. 開啟 php70.local 與 php80.local:8080
![](https://hackmd.io/_uploads/By-D99CA3.png)

![](https://hackmd.io/_uploads/rkODq9AR3.png)

