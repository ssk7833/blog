title: 隱藏 console 中的 source 來源
date: 2016-05-25 15:02:30
categories:
- study
tags:
- javascript
permalink: hide-source-of-log-messages-in-console
---
先前有發現 Facebook 在開發人員工具的 console 中最右邊不會顯示 log 的來源行數，覺得神奇，但並沒有深入了解如何實作此效果。而最近看到有人提問同樣的問題，同樣以 Facebook 的 console 為例來發問，激起了我的求知慾望。
![Facebook 的 console，最右端沒顯示來源](/blog/images/2016-05-25-hide-source-of-log-messages-in-console/01.png "Facebook 的 console，最右端沒顯示來源")
![正常呼叫 console.log 時，最右端會顯示來源](/blog/images/2016-05-25-hide-source-of-log-messages-in-console/02.png "正常呼叫 console.log 時，最右端會顯示來源")

單純要查到此功能似乎不難，剛好在四個月前有人在 [stackoverflow](http://stackoverflow.com/questions/34762774/how-to-hide-source-of-log-messages-in-console) 問了同樣的問題，而方法就是利用下方寫法即可：
```javascript
setTimeout(console.log.bind(console, "Hello world!"));
```

看到 Facebook 將 console 的預設樣式改掉，因此我也試著去看該怎麼做，果然樣式是用 CSS 去設定，而要設定的部分前面記得放個 `%c`，若有多段需要不同樣式則放入多個 `%c`，每個 `%c` 都是獨立的，樣式不會互相覆蓋；而 CSS 以額外參數的方式傳遞，若有 N 個 `%c`，則需要另外傳入 N 個參數。範例如下：
```javascript
console.log("%cRed color", "color:red;"); // 顯示為紅色字
console.log("%cRed color with blue background", "color:red; background:blue;"); // 顯示紅色字與藍色背景

console.log("%cRed color and %cgreen color", "color:red;", "color:green;"); // 顯示紅色字即綠色字
console.log("%cRed color%c and %cgreen color", "color:red;", "", "color:green;"); // 顯示紅色字即綠色字，將 "and" 改回預設樣式
```

兩者結合起來的話，一樣將樣式參數放於之後：
```javascript
setTimeout(console.log.bind(console, "%cRed color", "color:red;"));
```

因此我寫了一個範例，可以開啟開發人員工具的 console 觀看結果：
<p data-height="265" data-theme-id="0" data-slug-hash="JKPYoX" data-default-tab="js,result" data-user="ssk7833" data-embed-version="2" class="codepen">See the Pen <a href="http://codepen.io/ssk7833/pen/JKPYoX/">Custom console log messages</a> by North (<a href="http://codepen.io/ssk7833">@ssk7833</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

好，問題解決了！
但痛苦來臨，我走不出自己這關 XD，深問 why？為什麼 `setTimeout(console.log.bind(console, "something"))` 能有如此魔力使來源行數隱藏起來？式了很多方式去包裝 console 還是徒勞無功，且 setTimeout 好像是必要的？自行寫了一個具有 callback function 的 function 也無法達到一樣效果。

後來想到或許能在 stack trace 中得到一點線索，因此改下 `console.error` 試試看，結果還是深深地打了我一巴掌，stack trace 竟然完全是空的。
![比較 error 上下的差別](/blog/images/2016-05-25-hide-source-of-log-messages-in-console/03.png "比較 error 上下的差別")

有人跟我說，會不會是 call 到 browser 的 console 再從 console 下 command 才會這樣顯示出來？結果也不是，也是跳出了來源。
![使用開發者工具下指令結果一樣](/blog/images/2016-05-25-hide-source-of-log-messages-in-console/04.png "使用開發者工具下指令結果一樣")

真是奇怪啊！百思不得其解後，決定先「不求甚解」了，等待哪一天突然想到再補充吧。

2016-05-26 Update:

最後跑去 [stackoverflow](http://stackoverflow.com/q/37430531/4968420) 問了，也有人回答了讓我似懂非懂的答案，因有些不確定性的部分並沒提到，總之可能就先這樣，哪天想到再繼續研究。


參考資料：
1. [How to hide source of Log messages in Console?](http://stackoverflow.com/questions/34762774/how-to-hide-source-of-log-messages-in-console)
2. [Colors in JavaScript console](http://stackoverflow.com/questions/7505623/colors-in-javascript-console)
3. [Console API Reference | Web Tools - Google Developers](https://developers.google.com/web/tools/chrome-devtools/debug/console/console-reference)
4. [Format Specifiers](https://github.com/DeveloperToolsWG/console-object/blob/master/api.md#format-specifiers)
