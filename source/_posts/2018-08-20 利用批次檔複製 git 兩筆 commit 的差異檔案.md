title: 利用批次檔複製 git 兩筆 commit 的差異檔案
date: 2018-08-20 15:33:00
categories:
- study
tags:
- windows
- batch
- git
permalink: copy-all-files-changed-between-commits-using-batch-script
---
上週不曉得為什麼躺著就掉進坑中，開始自願協助改善某個論壇功能，一加入才發現長期以來沒有使用版本控制，且更新聽說是直接將檔案拉到正式網站上的 FTP 測試。聽起來有點毛骨悚然，因此先幫忙建了一個 git，先別想 CI/CD 了，需要的成本有點太高了，考量到種種狀況，於是決定先寫個 script 來協助上傳到 FTP 的功能。

`diffcopy.bat`
```Bash
@echo off

SETLOCAL ENABLEDELAYEDEXPANSION
for /f %%a in ('git diff --name-only %1 %2') do (
    set "filepath=%%a"
	set "filepath=!filepath:/=\!"
	set "destFilepath=diffExport\!filepath!"
    echo !destFilepath!
    xcopy !filepath! !destFilepath!* /Y /I /Q
)
ENDLOCAL
```

使用方法：
```shell
diffcopy ae40e29 05b7a26
```
![執行結果](/blog/images/2018-08-20-copy-all-files-changed-between-commits-using-batch-script/01.png "執行結果")

如此一來即可取得 commit ae40e29 到 05b7a26 之間的差異檔案，並複製到 `diffExport` 資料夾，原本想將資料夾如 [Stack Overflow 這篇](https://stackoverflow.com/a/31341016/4968420)一樣設為第三個參數，後來想想簡單就好，不過記得於每次下指令前須先手動將 diffExport 清空才行。

雖然有找到 [GitPython](https://github.com/gitpython-developers/GitPython) 看似蠻方便的 library，但想想還是寫個 batch 好了。

參考資料：
1. [git - Copy all files changed in last commit - Stack Overflow](https://stackoverflow.com/a/31341016/4968420)
