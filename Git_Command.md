# Git Command

[![hackmd-github-sync-badge](https://hackmd.io/Kg-jJl3TT0ydxIAUW7BEoA/badge)](https://hackmd.io/Kg-jJl3TT0ydxIAUW7BEoA)

###### tags: `Git`

1. 查詢git版本
```
git --version
```

2. 設定用戶名稱和電子郵件地址(只需要做一次)
```
git config --global user.name "<使用者名字>"
git config --global user.email "<電子信箱>"
```

3. 查看目前設定內容
```
git config -l
```

4. 新建數據庫(在目前資料夾，初始化本地端儲存庫。代表我們要讓 Git 開始對這個目錄進行版本控制。)
```
git init
```

5. 確認工作目錄與索引的狀態
```
git status
```
6. 將檔案加入到索引
```
git add <file>
```
將目前所在的目錄，以及它以下的子目錄、子子目錄 ...等裡的變動都加到暫存區，但如果是在這個目錄外的東西，就不會被影響。
```
git add .
```

不管在哪一層目錄，只要是這個專案裡的東西，所有的變動都會加至暫存區。
```
git add --all
```


7. 提交檔案 (將暫存區的檔案提交到儲存庫儲存)
```
git commit -m "<提交訊息>"
```

8. 查詢數據庫的歷史提交記錄
```
git log

# --oneline - 僅用一行顯示每次的 commit
# --graph - 顯示 commit 的樹狀結構
git log --oneline --graph
```
9. 請 Git 幫忙刪除檔案，可以減少一次步驟，直接加至暫存區（Staging Area）
```
git rm 可將原本兩段式需要透過 git add 提交的動作，直接縮短成一個指令。
git rm <file>
git rm <file> --cached # 移除 file.html 將之不再被 git 控管
```

10. Git 幫忙更改檔名
```
git mv <old file name> <new file name>
```

11. 檔案管理 - 忽略檔案 .gitignore
https://github.com/github/gitignore

如有符合規則的忽略檔案，但是想要將他加進 GIt
```
git add -f <檔案名稱> // -f 參數為強制刪除的意思，同等於 --force
```

一口氣清除已經被忽略掉的檔案：git clean 指令並配合 -x 參數
```
git claen -fx ＃強制清除已被忽略的檔案 // -f 參數為強制刪除的意思，同等於 --force
```

12. 還原檔案內容
```
git checkout <file> ＃ 還原 file 的檔案內容
```

使用 git checkout 指令，切換到指定 commit 版本
```
git checkout SHA-1編號前四碼（識別碼） ＃切換到指定版本
```

狀況題｜如果想將版本切回現在（最新）的版本呢？
```
git checkout main
```

13. 還原「檔案狀態」
使用 git reset 指令之前，要先了解 reset 的意思。從英文翻譯來看 reset 代表的是重新設定，但其實在中文解釋裡「前往」、「變成」，會較符合該指令的用途
```
git reset HEAD <file> # 還原 <file> 檔案狀態（使檔案回到前一個狀態）
```

#### Mixed 模式（預設）
此模式會將暫存區的檔案丟掉，但不會影響到工作目錄的檔案。也就是說我們指令中的 commit 除了移動 HEAD 及 Branch 外，也會從暫存區移出，但會留在工作目錄
```
git reset --mixed [commit]
```

#### Sort 模式
使用 Sort 模式的 reset，拆出來的 Commit 只是單純移動 HEAD 與其 Branch，暫存區與工作目錄中的檔案並**不會被丟棄**
```
git reset --sort [commit]
```

#### Hard 模式
使用 Hard 模式的 reset，除了移動 HEAD 與其 Branch 外，暫存區與工作目錄中的檔案也都會被移出
```
git reset --hard [commit]
```

14. 可對當前在staging area的檔案的修改回復至前一個狀態
```
git restore --staged <file>
```

15. 標籤（tag）是什麼呢？
跟分支一樣標籤也會指向某個 Commit ，但是並不會像分支一樣隨著 Commit 移動。因此，兩者之間最大的差異在於：分支會隨著 Commit 移動，而標籤則是固定留在某個 Commit 上。
#### 輕量標籤（lightweight tag）
```
git tag [標籤名稱] ［指定 Commit 的 SHA-1 值]
```

#### 附註標籤 （annotated tag）—> -a 參數
```
git tag -a [Tag名稱]［指定 Commit 的 SHA-1 值] -m"紀錄訊息"
```

#### 查詢標籤
```
git tag
```

#### 查看標籤的差別
```
git show [標籤名稱] # 查看標籤內容
```

#### 刪除標籤
```
git tag -d [標籤名稱] # 刪除特定標籤
```

16. 檢視分支
```
git branch
```

#### 新增分支
```
git branch <分支名稱> # 新增一個名為 <分支名稱> 的分支 
```

#### 重新命名分支名稱 -m
```
git branch -m [舊名稱] [新名稱] # 將 [舊名稱] 分支名稱更改為 [新名稱] 
```

#### 刪除分支 -d (-D 參數，大寫 D 為強制刪除之意)
```
git branch -d [分支名稱] # 刪除 [分支名稱] 分支
```

#### 切換分支 git checkout
```
git checkout [分支名稱] # 切換到 [分支名稱] 分支
```

17. merge 合併分支
```
git merge [分支名稱] # 合併 [分支名稱] 分支
```

#### --no-ff 參數
預設的 fast-forward 模式 ，從 SourceTree 上來原來的新增的分支並不會特別表示，也可以稱作是無線圖，如果想要特別表示這裡也有個其他分支（有線圖），可以加上 --no-ff 參數即可
```
git merge [分支名稱] --no-ff
```

#### 救回被砍掉的未合併分支
```
git branch [新分支名稱] [Commit 的 SHA-1 值]
```

#### merge衝突範例:
a.發現合併衝突訊息
b.使用 git status 指令查看狀態
c.打開有錯誤的檔案（GIt 會很貼心的幫你找出錯誤的那一行 code ）
d.修改 - 留下最後需要的 code
e.使用 git add 指令將修改後的檔案加回暫存區
f.git commit 指令繼續提交新的版本到儲存庫


18. (非必要盡量不要使用) rebase 合併分支
使用 git rebase 指令等於是修改歷史，他會使分支移動到不同的 Commit 重新定義基準點。
而 git rebase 指令與 git merge 指令最大的差異是什麼呢？
👉 git rebase 指令不會額外產生新的 Commit 來合併兩個分支。
```
git rebase [分支名稱] # 合併 [分支名稱] 分支
```

#### 如何取消 rebase ?
a. git reflog → git reset [指定的 SHA-1 值] --hard
b. ORIG_HEAD 會紀錄「危險操作」之前的 HEAD 位置。
```
git reset ORIG_HEAD --hard
```

非文字檔的合併衝突
```
git checkout --ours [檔名] // 如果決定使用當前分支的檔案
git checkout --theirs [檔名] // 如果決定使用對方的檔案
```

解決方式｜
查看錯誤訊息提示
決定要使用哪個分支的檔案作為最後的選擇
根據當前的分支，來決定使用的參數 --ours / --theirs
```
git add 指令加回暫存區
git commit 指令提交至儲存庫
```

19. 暫存檔案
假設今天不想要為還沒完成的檔案額外新增一個 Commit ，也不想要使用切換分支再回來 Reset 的動作，那麼在 Git 的指令中， git stash 指令也可以達到同樣暫存檔案的效果。
講解 git stash 指令前，先整理關於 git stash 常用到的指令
```
git stash - 暫存目前檔案
git stash list - 查看暫存的檔案列表
git stash pop - 叫回暫存的檔案，並從 stash list 中移除
git stash pop stash@{1} － 指定拿取某個暫存檔

git stash drop - 清除暫存檔案（最新的暫存檔案）
git stash drop [指定檔案] # 移除暫存檔案中的某個指定檔案

git stash apply - 將某個暫存檔案套用在目前分支，原本的 Stash 依然保留著
git stash apply [指定檔案] 

git stash clear - 清除全部暫存
```
20. 從 Git 中移除重要個資或徹底清除檔案
```
git filter-branch --tree-filter "rm -f 檔案名稱"
```

#### 將檔案徹底從 Git 中移除
先前執行 filter-branch 指令依然會保留備份的檔案在其他處，因此我們可以加上 -f 參數，意指強制覆寫 filter-branch 的備份點。
```
git filter-branch -f --tree-filter "rm -f 檔案名稱"
```

整理流程：
執行 filter-branch 強制覆寫指令 git filter-branch -f --tree-filter "rm -f 檔案名稱"
移除備份點 rm refs/orignal/refs/heads/one
Reflog 中的也移除並要求立馬過期 
```
git Reflog expire --all --expire=now
```
啟動資源回收機制將垃圾載走 
```
git gc --prune-now
```

21. 將本地端數據庫的修改歷史共享到遠程數據庫
a. 設定一個端節的節點
```
git remote add origin https://<GITHUB_ACCESS_TOKEN>@github.com/<GITHUB_USERNAME>/<REPOSITORY_NAME>.git
git remote add origin https://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@github.com/SiriusGOD/test_push_project.git

git remote 指令 - 跟遠端有關係的操作（ remote ，遠端之意）
add 指令 → 加入一個遠端的節點
origin 為指向位置的代名詞，這裡代表後面提供的伺服器位置（此位置與我們先前複製的相同）。
```
B. 將檔案推上遠端數據庫 
```
git push -u origin main
```

說明 Push 指令：
將 main 這個分支的內容推向 origin 這個位置。
先前 origin 已經有代指我們 GitHub 要上傳的位子
origin 如果沒有 main ，則自動建立一個同樣叫做 main 的分支。如果本來就有 main 分支，則會將此分支指向最新進度。
-u 參數，設定 upstream

強制推送
```
git push -f
```


22. 假使遠端的數據庫有了新的 Commit 版本，需要將這些更新取回本地端，此時使用此指令。
```
git fetch [遠端節點名稱] 
git fetch origin 
```

預設情況下，git fecth 會將所有有更新的分支取回，如有想指定特定的分支，也可以執行以下指令：
```
git fetch [遠端節點名稱] [分支名]
```

使用 git fecth 抓下來的更新，如果要在本地端讀取的話，需要使用「遠端節點名稱 / 分支名」的格式讀取。
例如：遠端的節點名稱為 origin、分支為 master，今天要將抓下來的分支合併到當前進度，那麼應該透過以下指令才有辦法讀取到最新進度：
```
git merge oringin/master
```

23. 同步其他人push的修改內容到自己的本地端數據庫
Rebase 的優點在於不會產生額外的 Commit 來紀錄合併這個動作。
```
git pull --rebase ＃使用 rebase 模式合併抓取檔案
git pull <repository> <refspec>
```

24. 複製遠端數據庫
```
git clone [url]
```

25. 顯示遠端數據庫清單
```
git remote
加上 -v 後即可顯示遠端數據庫的詳細情況。
```


