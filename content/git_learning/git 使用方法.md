
![](assets/git%20使用方法/file-20251228221804132.png)
初始化數據庫
```
git init
```
查看 git 版本
```
git --version
```
### 設定個人資料

輸入姓名
```
git config --global user.name "your_name"
```
輸入個人email
```
git config --global user.email "your.email@gmail.com"
```
查詢 git 設定內容 
```
git config --list | cat
```

### git 使用

查看當前 project 資料夾檔案狀態
1. Untracked 未追蹤 (U)
2. Tracked   已追蹤 (A)
3. Staged     已暫存 (M)
4. Committed 已提交
```
git status
```

新增追蹤檔案 (可一次新增多個檔案)
```
git add {檔案名稱} + {檔案名稱} ...
```

新增提交檔案
```
git commit -m "對於這次提交的概述"
```

查看全部修改版本
```
git log --oneline    
```

比較版本差異 (當前版本比指定版本新增or刪除什麼內容)
```
git diff {版本號} {檔案名稱}
```
版本2 比 版本1 (新增or刪除什麼內容)
```
git diff {版本號_1} {版本號_2} {檔案名稱}
```
