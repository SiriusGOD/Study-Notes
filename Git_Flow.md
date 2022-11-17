# Git Flow

[![hackmd-github-sync-badge](https://hackmd.io/rgei9x1fTcWWZo1YsAZpMw/badge)](https://hackmd.io/rgei9x1fTcWWZo1YsAZpMw)

###### tags: `Git`

是最早出現的一套流程，且廣泛被應用。
Git flow 提出不同的分支功能，分別有 master、develop 、hotfix、release、feature 五種分支。
而這五個分支，根據他們的性質，又有其他稱法：
* 長期分支 - master 分支、develop 分支
原因：因 Master 與 Develop 分支會一直存活在整個 Git flow 裡，並不會被刪除掉。
* 短期分支 - hotfix 分支、release 分支、feature 分支
原因：當完成專案後，這些更新的版本都會被合併進 Master 或 Develop 分支 ，之後就會被刪除掉。

## 分支應用情用
### Master 分支
master 是當 Git 建立版本控制時，就預設的分支，主要用來放穩定、隨時可上線的版本。
永遠處在 production-ready 狀態
來源是從其他分支合併過來，開發者不會直接 Commit 到此分支上。
因為是穩定版本，所以通常會在 master 分支上的 Commit 貼上「 版本標籤 」。

### Develop 分支
作為主要開發的分支，是所有短期分支的基礎分支。
最新的下次發佈開發狀態
Feature 分支從 Develop 分支切出去，當功能完成後，在合併回來 Develop 分支。
創建 Develop 分支的指令：
```
$ git checkout -b develop master
```
當要發布時，在 Master 分支上對 Develop 分支進行合併：
```
$ git checkout master # 切換到 master 分支
$ git merge --no-ff develop # master 分支對 develop 分支進行合併
-no-ff 參數代表不要快轉模式（fast-forward），會額外產生一個 Commit 物件。
```

### Hotfix 分支
主要功能是用來進行修復
從 Master 分支切出來的。
當修復完後，會合併回 Master 分支， 也同時會合併回 Develop 分支。
👉 合併回 Develop 分支的原因：
更新 Develop 分支的版本，避免遇到將 Develop 分支並回 Master 分支時，同樣的問題又出現。
👉 避免直接從 Develop 分支切出 Hoxfix 分支的原因：
Develop 分支為功能開發階段的分支，如果直接從 Develop 分支切 Hotfix 分支，再合併回 Master 分支，可能會造成管理上的問題/災難。
建立一個 Hotfix 分支
```
$ git checkout -b fixbug-0.1 master
```

常用 fixbug-* 的形式命名
修補結束後，合併到 Master 分支
```
$ git checkout master # 切換到 master 分支
$ git merge --no-ff fixbug-0.1 # 搭配非快轉模式參數合併 hotfix 分支
$ git tag -a 0.1.1 # 貼上訊息標籤
```

同時也要再合併到 Develop 分支
```
$ git checkout develop # 切換到 develop 分支
$ git merge --no-ff fixbug-0.1 # 搭配非快轉模式參數合併 hotfix 分支
```
最後再將 Hotfix 分支刪除
```
$ git branch -d fixbug-0.1 # 刪除 fixbug-0.1 分支
-d 參數為刪除之意
```

### Release 分支
在 Develop 分支發布正式版本到 Master 分支之前，可以先進行一個預備發布的本版本進行測試。
從 Develop 分支切出來
完成後需合併到 Master 分支與 Develop 分支
👉 為何又需要再合併 Develop 分支？
Master 分支為上限版本，而合併 Develop 分支的原因是為了後續可能 Release 分支上還會偵測到其他問題，所以需與 Develop 分支同步，以防再度出現同樣的問題。

### 建立 Release 分支
```
$ git checkout -b release-1.2 develop
```
release-* 的形式命名
修正完成後，合併到 Master 分支
```
$ git checkout master # 切換到 master 分支
$ git merge --no-ff release-1.2 # 合併 release-1.2 分支，使用非快轉模式合併
$ git tag -a 1.2 # 為新的節點新增訊息標籤
```
之後也要記得合併到 Develop 分支上
```
$ git checkout develop # 切換到 develop 分支
$ git merge --no-ff release-1.2 # 合併 release-1.2 分支，使用非快轉模式合併
```
兩個長期分支都合併完後，再刪除掉 Release 分支
```
$ git branch -d release-1.2
```

Feature 分支
主要用來新增功能的分支。
都是從 Develop 分支切出來的，完成後再合併回 Develop 分支。
建立一個 Feature 分支
```
$ git checkout -b feature-x develop
```
feature-* 的形式命名
功能開發完成後，再將 Feature 分支合併到 Develop 分支上
```
$ git checkout develop # 切換到 develop 分支
$ git merge --no-ff feature-x # 合併 feature-x 分支，使用非快轉模式合併
```

同樣完成後，會刪除掉分支
```
$ git branch -d feature-x # 刪除 feature-x 分支
```