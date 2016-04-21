title: Google Web Speech API 語音辨識 持續收音
date: 2016-04-21 10:06:51
categories:
- study
tags:
- javascript
- web speech API
permalink: Google-Web-Speech-API-continous-recording
---
Google 的 web speech API 已經推出一段時間了，最近剛好有機會來試試。
Web speech API 的操作並不困難，基本上就是 `var recognition = new webkitSpeechRecognition();`，可以在下方參考資料中看到相關原始碼，唯獨我預設的需求是必須可持續收音，因使用者可能是在不適合任何物理碰觸的環境下操作，所以鍵盤滑鼠及觸控螢幕皆不適合在此當作 input 來源。然而 web speech API 的持續收音 `recognition.continuous = true;` 若發現麥克風閒置太長的話，一樣會自動停止收音，因此還是要再重新觸發 `recognition.start();`，所幸我就將 `recognition.start();` 寫到 `onend` event handler，天真的以為這樣就解決問題了。

![要求使用麥克風](/blog/images/allowMicrophone.png "要求使用麥克風")
當每次收音結束後又重新觸發開始時，就會跳出這個允許授權的視窗，而當然此時是不能收音的，可以在此[範例](http://codepen.io/ssk7833/pen/YqepJb)體驗一下。

經過一番查證後，發現其實這個問題也不算問題，只是因為 chrome 要求安全性而在 http 上做出每次收音前都須要先確認允許才行，若是使用 https 的話只要允許過第一次後就再也不會詢問了，下方範例即是上方同一個範例的 https 版本。
<iframe height='300' scrolling='no' src='https://codepen.io/ssk7833/embed/YqepJb/?height=300&theme-id=0&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/ssk7833/pen/YqepJb/'>Web Speech API Demo</a> by North (<a href='http://codepen.io/ssk7833'>@ssk7833</a>) on <a href='http://codepen.io'>CodePen</a>.
</iframe>

允許之後就可以在麥克風例外狀況中看到已經預設允許的清單，在此清單中被允許的就再也不會跳出詢問視窗了，而 localhost 也是在允許範圍內，因此就可以做出很多有趣的事情了！
![麥克風例外狀況](/blog/images/microphoneExceptions.png "麥克風例外狀況")

使用上的話，目前好像也沒看到什麼限制，也沒找到限制相關的文件，自己親自掛了好幾小時也還是活得好好的。

參考資料：
1. [Voice Driven Web Apps: Introduction to the Web Speech API](https://developers.google.com/web/updates/2013/01/Voice-Driven-Web-Apps-Introduction-to-the-Web-Speech-API)
2. [Web Speech API Demonstration](https://www.google.com/intl/en/chrome/demos/speech.html)
3. [Google 語音辨識 API](http://www.oxxostudio.tw/articles/201509/web-speech-api.html)
