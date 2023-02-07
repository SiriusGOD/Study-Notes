# PM2 指令

[![hackmd-github-sync-badge](https://hackmd.io/ISk4TEI0TU25UT30xPtkeQ/badge)](https://hackmd.io/ISk4TEI0TU25UT30xPtkeQ)
###### tags: `PM2`

## PM2介紹
pm2 是一個管理 Node.js process 的工具，它可以讓我們用簡單的指令啟動 Node.js cluster 並最大化使用我們的 CPU，可以利用它來簡化很多node應用管理的繁瑣任務，如性能監控、自動重啟、負載均衡等，而且使用非常簡單。

## 安裝
使用npm安裝 
```
npm install pm2 -g
```

## 指令
1. 查看当前启动的所有应用
```
pm2 list
```
2. 重啟服務{app_name}
```
pm2 restart {app_name}
```
3. 啟動服務
```
pm2 start {app_name}
```
4. 停止服務
```
pm2 stop {app_name}
```
5. 停止並刪除服務
```
pm2 delete {app_name}
```
6. 列出目前所有已啟動的process並顯示他們目前的狀態, 按下ctrl+c 可以離開monitor
```
pm2 monit
```
## 指令可附加參數
* 指定 app 一個名字
```
--name
```
* 檔案有變更時，會自動重新啟動
```
--watch
```
* Memory 使用超過這個門檻時，會自動重啟
```
--max-memory-restart
```
* 指定 log 的位址, 若要指定新位址，需將原本的 process 刪掉，再重新啟動指定
```
--log
```
* 指定 output log 位址
```
--output
```
* 指定 error log 位址
```
--output
```
* 指定 log 的格式
```
--log-date-format
```
* 同一個 app 跑多程序時，不要依據程序 id 去分割 log, 全部合在一起
```
--merge-logs
```
* 指派額外的參數
```
--arg1 --arg2 --arg3
```
* 自動重啟時，要 delay 多久
```
--restart-delay
```
* 給 log 加上时间前綴
```
--time
```
* 不要自動重啟
```
--no-autorestart
```
* 指定 cron 規律，強制重啟
```
--cron
```
* 無 daemon 模式， listen log 模式
```
--no-daemon
```
* 限定 serve 使用, 會重導所有的請求到 index.html
```
--spa
```
* 用於靜態檔, 讓該頁面需要帳號密碼方可存取
```
--basic-auth-username 
--basic-auth-password
```

參考網站：
https://ithelp.ithome.com.tw/articles/10214173