title: 在 Logdown 上移除右上方 dashboard
date: 2016-06-27 14:55:30
categories:
- study
tags:
- javascript
- logdown
permalink: remove-dashboard-button-in-Logdown-page
---
最近朋友想開個部落格試試，我推薦了 Logdown、Blogger 或是全自己來的 GitHub Pages，最後友人選擇了 Logdown，然而友人跟我說：「Logdown 有個缺點，就是右上方的 Dashboard 很醜，要是能拿掉就太好了。」於是為了讓友人安心使用，我便看了一下 Logdown 原始碼，發現其實要拿掉或隱藏 Dashboard 並不困難。
![被嫌醜的 Dashboard](/blog/images/2016-06-27-hide-dashboard-button-in-Logdown-page/01.png "被嫌醜的 Dashboard")

到自己的 [Dashboard](http://logdown.com/dashboard) 中點選 `Blog 設定`，接著點選`佈景主題`分頁，點選`編輯 HTML`，如下圖位置。
![Blog 設定→佈景主題→編輯 HTML](/blog/images/2016-06-27-hide-dashboard-button-in-Logdown-page/02.png "Blog 設定→佈景主題→編輯 HTML")

在最下方 `</body></html>` 前面空白處插入以下程式碼：

```javascript
<script>
  document.addEventListener("DOMContentLoaded", function(event) {
  	var elem = document.querySelector('iframe[src="http://logdown.com/pages/top_controls"]');
  	elem.parentNode.removeChild(elem);
  });
</script>
```
插入後呈現如下：
![插入 javascript](/blog/images/2016-06-27-hide-dashboard-button-in-Logdown-page/03.png "插入 javascript")

接著重新開啟自己的 Logdown 看看就能看到成果囉！

※此段語法不適用於 [IE8](http://caniuse.com/#search=DOMContentLoaded)。
