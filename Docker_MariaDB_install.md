# Docker安裝與建立MariaDB

[![hackmd-github-sync-badge](https://hackmd.io/Jjtl_boeRQajFNCgxy5trA/badge)](https://hackmd.io/Jjtl_boeRQajFNCgxy5trA)

###### tags: `Docker`
## CentOS7
CentOS7 系統 CentOS-Extras 庫中已內建 Docker，可以直接安裝：
```
sudo yum install docker
```
安裝之後啟動 Docker 服務，並讓它隨系統啟動自動載入。
```
sudo service docker start
sudo chkconfig docker on
```
## 取得映像檔
可以使用 docker pull 命令從倉庫取得所需要的映像檔。
下面的例子將從 [Docker Hub](https://hub.docker.com/) 倉庫下載一個 MariaDB 的映像檔。
```
docker pull mariadb
```
![](https://i.imgur.com/nKsnqDo.png)
## MariaDB 镜像使用
接下來，我們將啟動一個 MariaDB 容器
```
# 建立一個 MariaDB 容器 Mariadb_Master
# root 密碼 29057419
docker run --name Mariadb_Master -e MYSQL_ROOT_PASSWORD=29057419 -d mariadb

# 列出容器
docker ps -a
```
![](https://i.imgur.com/x2A8eq7.png)

我們可以看到，一個 ID 為 fccbc0697b43 的 MariaDB 容器已經在運行中。
接下來，我們將進入容器，並查看數據庫
```
# 進入容器並查看數據
# CONTAINER ID fccbc0697b43
docker exec -it fccbc0697b43 bash
```
![](https://i.imgur.com/T54tD9E.png)

這時的 MariaDB 已經可以正常使用，但是無法遠程連接，因此我們需要映射端口來讓我們的數據庫能被遠程訪問。
所以我們先刪除容器，重新建立一個可以被外部訪問的MariaDB。

先關閉容器
```
docker stop fccbc0697b43

```
刪除容器
```
docker rm Mariadb_Master
```
建立可以被外部連線的MariaDB容器
```
docker run -d -P --name Mariadb_Master -e MYSQL_ROOT_PASSWORD=29057419 mariadb
```
建立好並啟動後可以看到會隨機映射port給外部連線
![](https://i.imgur.com/RZaunOX.png)
我們可以看到，通過 -P 參數，Docker 會為我們自動分配一個未被使用的port，這裡是 32768，接下來，我們可以通過 Workbench 工具來測試一下是否能連接。
![](https://i.imgur.com/M87nlAH.png)
![](https://i.imgur.com/UpH0riv.png)


其他操作如下:
啟動容器
```
docker start (CONTAINER ID)
```
重啟容器
```
docker restart (CONTAINER ID)
```
創建自訂MariaDB設定檔的容器:
自訂/home/my/master/config-file.cnf
```
docker run -d -P --name Mariadb_Master -v /home/my/master:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=29057419 -d mariadb
```
