# 利用MySQL WorkBench匯入csv資料

[![hackmd-github-sync-badge](https://hackmd.io/lBXvvN2STAGtCjNXqSKrFg/badge)](https://hackmd.io/lBXvvN2STAGtCjNXqSKrFg)

###### tags: `DB`
### 方法一
1、開啟MySQL WorkBench，新建一個Schema，如圖所示
&#160;&#160;&#160;&#160;&#160;&#160;![](https://i.imgur.com/bJbkpUo.png)
2、在上圖中的Tables上右鍵，點選Table Data Import Wizard選項，進入如下所示對話方塊
&#160;&#160;&#160;&#160;&#160;&#160;![](https://i.imgur.com/ZOvN8JE.png)
3、選擇要匯入的檔案和Schema，一直next即可，如圖所示
&#160;&#160;&#160;&#160;&#160;&#160;![](https://i.imgur.com/4cXLesw.png)

### 方法二
有時csv檔案過大或編碼有問題時，無法使用WorkBench匯入,這時可使用下sql語法吃入csv或txt檔
```
Load Data InFile 'test.txt' Into Table market_store_2020
fields terminated BY ','
lines terminated by '\r\n'
(market_id, market_name, tax_id, b_name, b_name_db, in_code1, name1, @a_103, @a_104, @a_105, @a_106, @a_107, @a_108, rank_103, @g_103, rank_104, @g_104, rank_105, @g_105, rank_106, @g_106, rank_107, @g_107)
set
stuff_103 = nullif(@a_103,0),
stuff_104 = nullif(@a_104,0),
stuff_105 = nullif(@a_105,0),
stuff_106 = nullif(@a_106,0),
stuff_107 = nullif(@a_107,0),
stuff_108 = nullif(@a_108,0),
grow_103 = nullif(@g_103,0),
grow_104 = nullif(@g_104,0),
grow_105 = nullif(@g_105,0),
grow_106 = nullif(@g_106,0),
grow_107 = nullif(@g_107,0)
```
