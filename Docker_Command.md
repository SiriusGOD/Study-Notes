# Docker Command
1. 下載image
```
docker pull {image name}
```

2. 列出已下載image
```
docker image ls
```

3. 刪除image
```
docker image rm {IMAGE ID}
```
4. 列出已執行Container
```
docker ps
```
5. 執行Container
```
docker run --name nginx -p 8080:80 -v /Users/mac/Desktop/Project/:/usr/share/nginx/html -d nginx
docker run --name nginx_php8 -p 8080:80 -v /Users/mac/Desktop/Project/:/var/www/html -d 0dda2a604bd4
```


6. 刪除Container
```
docker rm {IMAGE ID}
```

7. 進入docker
```
docker exec -it {IMAGE ID} bash
```

8. 停止docker
```
docker stop
```

## LaraDock 

9. 進入docker
```
docker-compose exec {container_name} bash
```

10. 列出此專案容器 (要進到專案目錄下)
```
docker-compose ps
```

11. 關閉所有正在運行的容器
```
docker-compose stop
docker-compose stop {container_name}
```

12. 刪除所有現有的容器
```
docker-compose down
```

13. 構建/重建容器
```
docker-compose build {container_name}

// --no-cache如果您想要完全重建
docker-compose build --no-cache {container_name}
```

14. 查看日誌文件
```
docker-compose logs {container-name}
docker-compose logs -f {container-name}
```
