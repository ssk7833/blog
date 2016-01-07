title: 將 cd 跟 ls 合併為一個指令
date: 2011-01-18 01:45:05
updated: 2016-01-08 00:47:05
categories:
- study
tags:
- shell script
permalink: combine-cd-and-ls-into-one-command
---
用 alias, script, unix programing 都無法達到更換資料夾後並印出資料夾內容的功能
最後測試只有 function 可以。

若是 bash shell，將 .bashrc 增加下列程式碼後重新載入 .bashrc 即可。

    cds() { cd "$1"; ls; }

接著只要執行 cds 指令即可。
