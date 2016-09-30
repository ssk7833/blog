title: 利用 Google Chrome 作為 kiosk 應用時的設定項目
date: 2016-08-02 16:03:45
categories:
- study
tags:
- windows
- chrome
- kiosk
permalink: setup-Google-Chrome-as-kiosk-application-settings
---
紀錄一下利用 Google Chrome 作為 kiosk 時，需要注意的地方：
本文所使用的 Google Chrome 版本：52.0.2743.82 m

## 鎖定右鍵內容
當觸控啟用時，預設長按螢幕就會觸發滑鼠右鍵的功能表。如同一般網頁防止滑鼠右鍵內容一樣，於 HTML 中加入 Javascript 來防止右鍵清單的跳出。
```javascript
window.addEventListener('contextmenu', function(e) { e.preventDefault(); });
```

## 禁止文字被選取
當長按螢幕時會觸發與滑鼠左鍵按住拖曳來選取文字同樣的效果，如下圖中間搜尋視窗：
![長按螢幕字串時](/blog/images/2016-08-02-setup-Google-Chrome-as-kiosk-application-settings/01.png "長按螢幕字串時")
若不想被選取，可以利用 CSS 處理，根據自己不想被選的部分作出篩選，下方範例以整份 HTML 當作範例：
```CSS
html {
  -webkit-user-select: none;
}
```

## 關閉觸控滑動拖曳到上一頁／下一頁
若有超連結會導到其他頁面時，導向其他頁面後按著螢幕並往左或往右滑動，可能會觸發如手持裝置般的上下頁功能：
![當有歷史分頁時，碰觸畫面並往右滑將會回到上一頁](/blog/images/2016-08-02-setup-Google-Chrome-as-kiosk-application-settings/02.png "當有歷史分頁時，碰觸畫面並往右滑將會回到上一頁")
解決方式有兩種：
1. 利用 CSS 去解決：
  ```CSS
  html {
    touch-action: none;
    -webkit-user-drag: none;
  }
  ```
  **但有缺點！**若在 HTML 內容中有需要橫向卷軸的部分，此橫向卷軸部分不可能套用上方 CSS，需再另外對 `touch-action` 及 `-webkit-user-drag` 允許橫向操作，而若允許之後，在此橫向卷軸部分拖曳到最頂／底端時再繼續拖曳，依然會觸發道上下頁的功能。

2. 修改 Chrome flags 內的設定：
  開啟 Chrome 新分頁，在網址列輸入 `chrome://flags` 並找到 `橫向捲動紀錄導覽 Mac, Windows, Linux, Chrome OS, Android`，預設為`已啟用`，把它改成`已停用`。
  ![更改設定為已停用](/blog/images/2016-08-02-setup-Google-Chrome-as-kiosk-application-settings/03.png "更改設定為已停用")
  如此一來連 CSS 也不用寫了，缺點就是這個為全域設定，改成停用後任何網站都不能使用，不過既然是 kiosk，改成已停用理應影響不大。

---
接下來設定皆與命令列(command line)參數有關，使用方法有：
1. 利用建立捷徑的方式來填參數：
  ![利用建立捷徑方式填參數](/blog/images/2016-08-02-setup-Google-Chrome-as-kiosk-application-settings/04.png "利用建立捷徑方式填參數")
2. 直接開啟執行(Windows + R)並輸入指令及參數：
  ![在執行中輸入](/blog/images/2016-08-02-setup-Google-Chrome-as-kiosk-application-settings/05.png "在執行中輸入")
3. 於命令提示字元(cmd)中執行其中一行：
  ```
  start chrome
  ```
  ```
  "C:\Program Files (x86)\Google\Chrome\Application\chrome.exe"
  ```

## 開啟固定位址的內容
開啟時直接進入某個網頁。
在參數列加上網址即可，如 `start chrome ssk7833.github.io`。
若是本機上的靜態檔案則須加上 `file:///`，如 `start chrome file:///C:/dist/index.html`

## 運行 kiosk 模式
開啟時直接進入全螢幕模式，且無法利用 `F11` 跟 `ESC` 來離開全螢幕模式，可以用 `ALT + F4` 或 `CTRL + W` 來關閉。
在參數列加上 `--kiosk` 即可。

## 關閉詢問「您要翻譯這個網頁嗎？」
![您要翻譯這個網頁嗎？](/blog/images/2016-08-02-setup-Google-Chrome-as-kiosk-application-settings/06.png "您要翻譯這個網頁嗎？")
使用 kiosk 模式後，依然可能會因 HTML 撰寫或內文而自動跳出這個訊息，在 kiosk 模式這當然是不想要的。
在參數列加上 `--disable-translate` 即可。

## 允許 Chrome 無視跨來源資源共享(CORS)限制
如果 kiosk 的內容是靜態網頁而無後台，又需要利用 AJAX 其他地方拿取資料，很可能會在開發時出現這種錯誤：
`XMLHttpRequest cannot load https://www.example.com. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'null' is therefore not allowed access.`
![CORS 錯誤](/blog/images/2016-08-02-setup-Google-Chrome-as-kiosk-application-settings/07.png "CORS 錯誤")
若不想額外增設後台可在參數列加上 `--disable-web-security`，但加上後錯誤會依然存在，需配何下方參數才能正常運行。

## 開啟新的使用者資料夾來放置內容
如同一台電腦支援多個 Chrome 使用者一樣，另外開一個額外的資料夾來放置 user data，使其不會各自汙染。
在參數列加上 `--user-data-dir="C:/Chrome dev session"` 即可，`C:/Chrome dev session` 即是接下來這個 Chrome 視窗將會存放資訊的位置，此資料夾無需自行建立，系統會自動幫忙建立。
`--disable-web-security` 搭配 `--user-data-dir="C:/Chrome dev session"` 即可無視 CORS 限制，但使用上也須注意是否有其他安全性的疑慮，不過既然都是 kiosk 了，理應不太會有這個問題。

## 移除上方跳出的警告訊息
前兩項做到後，其實會跳出此訊息：`您正在使用不受支援的命令列標識：--disable-web-security。這可能會危及穩定性與安全性。`
![--disable-web-security 警告](/blog/images/2016-08-02-setup-Google-Chrome-as-kiosk-application-settings/08.png "--disable-web-security 警告")
在 kiosk 模式當然不可能放出此行，可以在參數列加上 `--test-type` 即可。~~為什麼這幾個不一次講完呢？~~

## TL;DR
上述這幾個的參數組合技已經蠻夠用的，我目前下的指令如下：
```bash
chrome.exe "file:///C:/dist/index.html" --user-data-dir="C:/Chrome dev session" --disable-web-security --test-type --disable-translate --kiosk
```
為什麼要用指令呢？因為能撰寫於 batch 中來達到部分的自動化。
不確定某些指令是否可能有潛藏的風險，若有思慮不周的地方也請您分享！

**UPDATE：** 下一篇出來了：[利用 windows batch 自動初始化、檢查 chrome 開啟狀態](/blog/2016/09/30/using-windows-batch-to-initialize-and-check-chrome-status/)

參考資料：
1. [Google Chrome browser, how to permanently disable this disturbing toolbar?](http://askubuntu.com/a/310521)
2. [Protractor error message “unsupported command-line flag” in Chrome?](http://stackoverflow.com/a/24018881/4968420)
