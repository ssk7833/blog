title: 利用 canvas 於 Chrome 實作 iOS 模糊背景效果
date: 2016-10-18 11:02:30
categories:
- study
tags:
- javascript
- CSS
- canvas
permalink: canvas-solution-of-implement-iOS-blurry-overlay-view-on-Chrome
---
![iOS blurry overlay](/blog/images/2016-10-18-canvas-solution-of-implement-iOS-blurry-overlay-view-on-Chrome/01.png "iOS blurry overlay")
這次要處理的就是上面的背景模糊背景效果，雖 iOS7 在某些版面上已經有了，但在瀏覽器上要做到同樣的事通常都要疊兩張一樣的圖再做 filter 處理，若今天需要模糊的是比圖片複雜的，例如影片、包含其他動態元件的 html 等就會變得不好處理。
在 Safari 上可以使用 `-webkit-backdrop-filter: blur(5px);` CSS 屬性來處理，且可以根據上層的 overflow:hidden; 來改變模糊的區域範圍；而在 Google Chrome 上，雖然可以透過 `enable-experimental-web-platform-features` 來開啟 `backdrop-filter: blur(5px);` 功能，但在配合其他 CSS 屬性時很容易出現 bug。

backdrop-filter 的實作範例：Chrome 需開啟 `enable-experimental-web-platform-features` 才能看到結果
Safari 下正常，而 Chrome 也正常，但我需要圓弧邊框：[JSFiddle](https://jsfiddle.net/ssk7833/8ybsqz4k/14/)
Safari 下正常，而 Chrome 下若上層有 overflow: hidden; 時則完全無效：[JSFiddle](https://jsfiddle.net/ssk7833/8ybsqz4k/19/)

網路上能找到許多解法，像是放兩張圖或放兩個影片，前景的圖層加上 filter:blur 即可，但最後我選擇使用 canvas 來實作看看。
<iframe width="100%" height="300" src="//jsfiddle.net/ssk7833/8ybsqz4k/36/embedded/result,html,js,css/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

背景是影片的狀況：
<iframe width="100%" height="300" src="//jsfiddle.net/ssk7833/zjzdmbcj/4/embedded/result,html,js,css/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

不過 canvas 解法反而不是用於 Safari，因為 Safari 不支援 canvas filter...XD

參考資料：
1. [CanvasRenderingContext2D.filter - Web APIs | MDN](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/filter)
