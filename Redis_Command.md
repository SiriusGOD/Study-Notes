# Redis Command

[![hackmd-github-sync-badge](https://hackmd.io/7IBar0-ZT6Ohuhu0pxm4eQ/badge)](https://hackmd.io/7IBar0-ZT6Ohuhu0pxm4eQ)

###### tags: `Redis`

1. 進入redis cli
```
redis-cli
```

2. 列出現有所有的 key
```
keys *
```

3. 查詢該key_name的值
```
get {key_name}
```

4. 切換到不同的資料庫以select執行，預設是使用0
```
Select 1
```

5. 刪除指定的 key
```
del {key_name}
```

## *** Strings(Integers) ***

6. 遞增
```
incr {key_name}

# 如果指定一次累加多少可用 incrby
incrby {key_name} {num}
```

7. 遞減
```
decr {key_name}

# 如果指定一次遞減多少可用 decrby 
decrby {key_name} {num}
```



## *** Lists ***

8. 給list key塞值
```
lpush {key_name} {value}
```

9. 列出所有訊息編號
```
lrange {key_name} 0 -1
```

10. 只保留前三個訊息
```
ltrim {key_name} 0 2
```

11. 刪掉倒數第一個55的訊息
```
lrem {key_name} -1 55
```

12. 從左邊pop出第一個
```
lpop {key_name}
```


## *** Hashes ***

13. 同時設好幾個key value用hmset
```
hmset {key_name} type tech subject "建立API為中心的輕量級網站"
```

14. 一次抓所有的值
```
hgetall {key_name}
```

15. 一次只加一個key value用 hset
```
hset {key_name} created_at "2012-10-09 23:34:26"
```

16. 一次抓一個key值
```
hget {key_name} type
```

## *** Sets ***
沒有分序列的集合，不同於有序列的陣列，元素也不會有重覆。

17. SADD 將元素推進集合
```
sadd {key_name} {value}
```

18. SMEMBERS 列出該集合所有元素
```
smembers {key_name}
```

19. SISMEMBER 查某元素是否屬該集合
```
sismember {key_name} {value}
```

20. SINTER 列出兩集合的交集的元素
```
sinter {key_name1} {key_name2}
```

21. SUNION 列出兩集合聯集的元素
```
sunion {key_name1} {key_name2}
```

22. SPOP 隨機移出集合裡的一元素
```
stop {key_name}
```

23. SREM 從集合移出一個或多個元素
```
srem {key_name} {value}...
```


## *** Sorted Sets ***
同樣是集合，但元素是再加上數值，而可依每元素的數值做排序

24. 添添加元素到集合，元素在集合中存在則更新對應score
```
zdd {key_name} {score} {value}
```

25. 升冪排序
```
zrange {key_name} 0 -1
```

26. 降冪排序
```
zrevrange {key_name} 0 -1
```

27. 加上 WITHSCORES 列出每元素的分數
```
zrevrange {key_name} 0 -1 WITHSCORES
```

28. 算出分數 5000 ~ 10000 的元素個數
```
zcount {key_name} 5000 10000
```

29. 列出分數 5000 ~ 10000 的元素
```
zrangebyscore {key_name} 5000 10000
```

30. 增加某元素的分數score
```
zincrby {key_name} {score} {value}
```

31. 列出某元素的分數
```
zincrby {key_name} {value}
```

## *** 有時效的key ***

32. 透過 expire 的指令，可設定某key在幾秒後 key 被刪除
```
expire {key_name} {seconds}
```

33. TTL 可看key剩多少秒將被刪除
```
ttl {key_name}
```

34. PERSIST 可取消掉倒數刪除的作用，而讓該key沒有expire的屬性
```
persist {key_name}
```

35. 也可指定某個時間expire，時間指定需用unix timestamp格式
```
expireat {key_name} {unix timestamp}
```

36. 設key的同時也設key的有效時間
```
set {key_name} {seconds} {value}
```
