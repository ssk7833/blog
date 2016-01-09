title: HTML5 fullscreen API 將 iframe 以全螢幕顯示
date: 2016-01-10 04:10:05
categories:
- study
tags:
- javascript
permalink: show-iframe-in-fullscreen-by-html5-fullscreen-api
---
最近因為有把 iframe 內容以全螢幕顯示的需求，因此研究了一下 HTML5 fullscreen API。已有現成的 library 可以用如 [screenfull.js](https://sindresorhus.com/screenfull.js/) 及 [BigScreen](https://brad.is/coding/BigScreen/)，但大致上並不難，因此我選擇了純 javascript 來撰寫看看。

要全螢幕其實並不難，只要呼叫 `requestFullscreen()` 即可做到，以下是簡易範例：

```
var button = document.querySelector('#container .button');
button.addEventListener('click', fullscreen);

function fullscreen() {
  // check if fullscreen mode is available
  if (document.fullscreenEnabled ||
    document.webkitFullscreenEnabled ||
    document.mozFullScreenEnabled ||
    document.msFullscreenEnabled) {

    // which element will be fullscreen
    var iframe = document.querySelector('#container iframe');
    // Do fullscreen
    if (iframe.requestFullscreen) {
      iframe.requestFullscreen();
    } else if (iframe.webkitRequestFullscreen) {
      iframe.webkitRequestFullscreen();
    } else if (iframe.mozRequestFullScreen) {
      iframe.mozRequestFullScreen();
    } else if (iframe.msRequestFullscreen) {
      iframe.msRequestFullscreen();
    }
  }
  else {
    document.querySelector('.error').innerHTML = 'Your browser is not supported';
  }
}
```

此範例即可讓當所選擇的 `#container .button` 被點擊時，讓 `#container iframe` 全螢幕。

但因為 iframe 的內容並非我可以控制的，有些 iframe 的內容沒有處理 RWD，因此當頁面縮放時可能會呈現未預期的效果，如：[ぶつからないように動くビークル](http://codepen.io/kanaparty/pen/eJYXeZ)（找了一下 codepen 才找到一個可用範例）。要做到這點，我目前選擇當 iframe 被全螢幕時則重新載入一次，當然，當 iframe 從全螢幕離開時也會再 resize 一次，因此也要注意離開全螢幕時也得處理。

```
// reload
iframe.src = iframe.src
```

原本我認為應將重新載入寫在 request fullscreen 之後，而當觸發 keydown event 時再觸發一次重新載入，後來發現在全螢幕時按下 ESC 時 keydown event 都不會被觸發(chrome, firefox)，而按下 F11 則是 Firefox 會觸發而 Chrome 不會，因此認為這應該不是個好寫法。

後來在 [How to Use the HTML5 Full-Screen API (Again)](http://www.sitepoint.com/use-html5-full-screen-api/) 發現有 fullscreenchange event 可以用，因此也改用這個，原本放在全螢幕後的重新載入也改成放於 event listener 內，程式碼也簡潔多了！

```
// when you are in fullscreen, ESC and F11 may not be trigger by keydown listener.
// so don't use it to detect exit fullscreen
document.addEventListener('keydown', function (e) {
  console.log('key press' + e.keyCode);
});

// detect enter or exit fullscreen mode
document.addEventListener('webkitfullscreenchange', fullscreenChange);
document.addEventListener('mozfullscreenchange', fullscreenChange);
document.addEventListener('fullscreenchange', fullscreenChange);
document.addEventListener('MSFullscreenChange', fullscreenChange);

function fullscreenChange() {
  if (document.fullscreenEnabled ||
       document.webkitIsFullScreen ||
       document.mozFullScreen ||
       document.msFullscreenElement) {
    console.log('enter fullscreen');
  }
  else {
    console.log('exit fullscreen');
  }
  // force to reload iframe once to prevent the iframe source didn't care about trying to resize the window
  // comment this line and you will see
  var iframe = document.querySelector('iframe');
  iframe.src = iframe.src;
}
```

以下為測試的範例，可以試著把 `iframe.src = iframe.src;` 註解掉，即可看到改造前後的差異：

<p data-height="268" data-theme-id="0" data-slug-hash="mVOXXp" data-default-tab="result" data-user="ssk7833" class='codepen'>See the Pen <a href='http://codepen.io/ssk7833/pen/mVOXXp/'>Fullscreen API on iframe</a> by North (<a href='http://codepen.io/ssk7833'>@ssk7833</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

參考資料：
[ぶつからないように動くビークル](http://codepen.io/kanaparty/pen/eJYXeZ)
[Using fullscreen mode - Web APIs | MDN](https://developer.mozilla.org/en-US/docs/Web/API/Fullscreen_API)
[How to Use the HTML5 Full-Screen API (Again) - SitePoint](http://www.sitepoint.com/use-html5-full-screen-api/)
