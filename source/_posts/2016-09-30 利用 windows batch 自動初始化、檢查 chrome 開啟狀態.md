title: 利用 windows batch 自動初始化、檢查 Google Chrome 開啟狀態
date: 2016-09-30 15:04:20
categories:
- study
tags:
- windows
- batch
- chrome
- kiosk
permalink: using-windows-batch-to-initialize-and-check-chrome-status
---
延續上一篇[利用 Google Chrome 作為 kiosk 應用時的設定項目](/blog/2016/08/02/setup-Google-Chrome-as-kiosk-application-settings/)所提到的，這個最終要應用在 batch 上，來達到在 kiosk 應用上能自動開機便能自動初始化、自動檢查 Chrome 是否因為不明原因被關閉了，減少作業系統曝光的情況，否則哪天說不定就會被拍照下來留念。

![Windows 出來呼吸了：http://t17.techbang.com/topics/20686-tv-advertising-is-running-windows-7-11](https://cdn0-t17-techbang.pixcdn.tw/system/attached_images/2013/06/99221/show/660a0705c6416315e5146d622080be1a.jpg?1372171678 "Windows 出來呼吸了！")

batch 檔內容如下：
```sh
:: detectChrome.bat
@echo off
:: 初始化
echo Initialize...
:: 更改資料夾到正確位置
cd C:\

:: 檢查網路狀態
echo Waiting for Interent connection...
:checkInternet
ping www.google.com -n 1 -w 10000 > nul
IF errorlevel 1 GOTO checkInternet

:: 這裡可放入自己想放的 code
::node update.js

:: 進入迴圈
:start
:: 若 processEnd.tmp 則跳脫迴圈結束程式
IF EXIST processEnd.tmp (
  echo processEnd.tmp detected! End the process.
  GOTO end
)

tasklist /FI "IMAGENAME eq chrome.exe" /FO TABLE > process.log
FOR /F %%c in ('find /v /c "" ^< process.log') DO set lineCount=%%c
:: 刪除暫存檔案 process.log
DEL process.log
:: 若 process 存在，則 lineCount 會大於 1，不存在則等於 1，因此若大於 1 則跳過執行部分
IF %lineCount% GTR 1 GOTO skip
:: 執行程式
echo Execute Chrome
::"C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --user-data-dir="C:/Chrome dev session" --disable-web-security --new-window "file:///C:/dist/index.html" --test-type --kiosk
start /WAIT /MAX chrome.exe --user-data-dir="C:/Chrome dev session" --disable-web-security --new-window "file:///C:/dist/index.html" --test-type --disable-translate --kiosk
:skip

REM 等待一段時間後再次檢查(秒)
timeout /t 5
GOTO start
:end
DEL processEnd.tmp
```

主要流程為需要開機後直接執行更新，但我的環境曾遇到開機後網路異常的現象，因此檢查網路是否有沒有通，如果不擔心此問題的話則可考慮把`檢查網路狀態`那幾行拿掉。接著再執行自己需要的程式，我這邊已有一現成用 node.js 撰寫的程式，因此就讓他執行 node update.js。接著就進入 Chrome 判斷的迴圈了，為避免意外重複開啟 Chrome，因此先判斷了是否有 Chrome 在開啟狀態，若都沒有再開啟。

接著把這隻 batch 檔案丟到「所有程式」的「啟動」中就行了，或是利用「工作排程器」也可以達到同樣效果，唯一不方便的是有需要對電腦做操作時，要關掉這隻程式不是很方便，因此我又另外寫了一隻關閉這隻程式的 code，很懶惰的用了檔案檢查的方式去處理，在 detectChrome.bat 的迴圈中判斷是否有個暫存的檔案 processEnd.tmp，若有的話則不要中止程式。

```sh
:: end.bat
@echo off
type nul > C:\processEnd.tmp
echo 等待倒數後結束程式
```

如此一來，不太懂的操作者也可以輕鬆的用 alt+tab, windows+d 等按鍵從 chrome kiosk 切換出去，然後開啟這隻 end.bat，再把 chrome alt+f4 關閉來完整結束這個程式帶來的影響。

bash 寫慣了，要寫 batch 還真不太適應，所幸有朋友的 [Batch 溫故/知新](http://www.slideshare.net/yipotw/batch-56466561) 支援省了我不少時間。

參考資料：
1. [Batch 溫故/知新](http://www.slideshare.net/yipotw/batch-56466561)
2. [不用寫程式，偵測執行檔沒running就再次執行它](https://dotblogs.com.tw/rolence0515/2011/08/15/33157)
3. [Count number of lines in a text file from a windows batch file](https://social.technet.microsoft.com/Forums/scriptcenter/en-US/1867323d-e6c7-440f-83a4-2bdc9b4432d5/count-number-of-lines-in-a-text-file-from-a-windows-batch-file?forum=ITCG)
4. [如何在批次檔(Batch)中實現 sleep 命令讓任務暫停執行 n 秒](http://blog.miniasp.com/post/2009/06/24/Sleep-command-in-Batch.aspx)
5. [windows - If greater than batch files - Stack Overflow](http://stackoverflow.com/questions/9278614/if-greater-than-batch-files/9278668#9278668)
