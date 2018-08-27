title: 於 iOS 裝置上開發 Web App 筆記
date: 2018-08-27 18:52:10
categories:
- study
tags:
- iOS
- PHP
- Web Application
permalink: develop-a-web-app-for-ios
---
修改案例為一個已營運多年的 Discuz 論壇，聽到使用者的聲音，希望能於手機上用 App 瀏覽論壇，基於考量，開發一個 Native App 及上架是不太可能的，而論壇本身已有一版手機版網站，因此決定以 Web App 的方式來實驗看看，不過坑比想像中的大。

Apple 有一篇基本上講的蠻清楚的文章：[Configuring Web Applications](https://developer.apple.com/library/archive/documentation/AppleApplications/Reference/SafariWebContent/ConfiguringWebApplications/ConfiguringWebApplications.html)
不過實際測試會發現問題蠻多的，以下一一列出自己碰到的狀況：
1. App 圖示
    文件中明確的說出有幾個放法：
    - 要讓整個網站都有 icon，可以將 PNG 格式的圖片放置於根目錄，命名為：`apple-touch-icon.png`
    - 要在特定頁面加上自訂的 icon 可以於 &lt;head&gt; 中加入如：
        ```html
        <link rel="apple-touch-icon" href="/custom_icon.png">
        ```
    - 要在不同解析度的裝置上能讀取不同尺寸的圖示，可以於 &lt;head&gt; 中加入如：
        ```html
        <link rel="apple-touch-icon" href="touch-icon-iphone.png">
        <link rel="apple-touch-icon" sizes="152x152" href="touch-icon-ipad.png">
        <link rel="apple-touch-icon" sizes="180x180" href="touch-icon-iphone-retina.png">
        <link rel="apple-touch-icon" sizes="167x167" href="touch-icon-ipad-retina.png">
        ```
        圖示大小可參考於 iOS Human Interface Guidelines 的 Graphics 一章，實際上這章已經變成 Icons and Images 內的[一部分](https://developer.apple.com/design/human-interface-guidelines/ios/icons-and-images/app-icon/)了。

  在這個區塊文件還特別註明了 iOS 7 以前 Safari 的特殊狀況，因此要特別注意，雖然這點對於 App icon 沒什麼影響，卻讓我對文件下一區塊 `Specifying a Launch Screen Image` 沒有任何特別註明任何狀況而浪費了很多時間，這區塊理應給個提醒，不過稍後再提。

  App icon 會於首次要加入主畫面時去伺服器要相關檔案，之後基本上便會使用這個 icon 當作 app 的圖示；而問題是若 icon 需要更新時該如何更新？於 iOS 10.3.2 在同一天內測試發現除非移除 Web App 再重新加入主畫面，否則不會更新 App icon，而隨後試了 [stackoverflow 這篇](https://stackoverflow.com/a/4587917/4968420)的方法發現可行，雖然需要重開機才會更新，但起碼是個應急的方法：
    ```php
    <link rel="apple-touch-icon" sizes="152x152" href="/static/webapp/icons/icon-152x152.png?m=<?php echo filemtime('apple-touch-icon.png'); ?>">
    <link rel="apple-touch-icon" sizes="180x180" href="/static/webapp/icons/icon-180x180.png?m=<?php echo filemtime('apple-touch-icon.png'); ?>">
    <link rel="apple-touch-icon" sizes="167x167" href="/static/webapp/icons/icon-167x167.pngm=<?php echo filemtime('apple-touch-icon.png'); ?>">
    ```
2. App 標題
    這個沒遇到什麼狀況，文件提到預設抓 &lt;title&gt;，若需預設於 &lt;head&gt; 加入並修改 content 內容：
    ```html
    <meta name="apple-mobile-web-app-title" content="AppTitle">
    ```
3. 隱藏 Safari UI
    將 Web App 視為獨立模式 App (standalone mode)，於 &lt;head&gt; 中加入：
    ```html
    <meta name="apple-mobile-web-app-capable" content="yes">
    ```
    為獨立 App 時預設所有超連結將會導回至 Safari App 中，因此須注意在 standalone mode 時將站內相關超連結改為用 AJAX 處理或更改 window.location 位置。
    可以參考作法：[Stay Standalone: Prevent links in standalone web apps opening Mobile Safari](https://gist.github.com/irae/1042167)

4. 改變 Status Bar 外觀
    只有為獨立 App 時有效，不過只有三種選擇：預設（淺灰色）、黑色、透明。
    以下為黑色範例：
    ```html
    <meta name="apple-mobile-web-app-status-bar-style" content="black">
    ```
    可參考 [stackoverflow 這篇](https://stackoverflow.com/a/40786240/4968420) 及 [Supported Meta Tags](https://developer.apple.com/library/archive/documentation/AppleApplications/Reference/SafariHTMLRef/Articles/MetaTags.html)

5. 加上 Launch Screen Image / Splash Screens
    於 App 開啟時秀出的畫面，如一般 native App 開啟時的效果一樣，如[這篇](https://medium.com/appscope/adding-custom-ios-splash-screens-to-your-progressive-web-app-41a9b18bdca3)開頭圖片效果，不過在此需特別提一下 `iOS 8 9 10 實測不支援此效果`，由於官方文件沒有特別提到，讓我一直以為自己哪邊寫錯了；另外要使用此功能也`必須`為獨立 App 才行。
    ```html
    <link rel="apple-touch-startup-image" href="/launch.png">
    ```
    這個也支援不同解析度的圖片：可參考[這篇](https://developer.apple.com/design/human-interface-guidelines/ios/icons-and-images/launch-screen/)，不過網路上已經有實用的轉換工具 [About splash-screens - Appscope](https://appsco.pe/developer/splash-screens) 可供轉換成各種尺寸的圖片及對應的程式碼。
    想要先測試手機上支不支援，也可以先上[這篇](https://medium.com/@applification/progressive-web-app-splash-screens-80340b45d210)最底下的範例 [PWA Splash Screens Demo](https://pwa-splash.now.sh/) 來檢查手機支援的狀況如何。

除這些基本設定外，遇到 web App 切到背景後再切回前景，其行為跟 native App 不一樣，web App 只要到了背景 native App 被 terminated 一樣，什麼狀態都要重來，因此預設情況下開啟 web App → 到了頁面 A → 切換至背景（例如收到推播去看個訊息等等） → 回到 web App，此時會回到預設的首頁，而非最後瀏覽的頁面 A；為了解決這個問題，可以利用 sessionStorage 紀錄是否為本次首次開啟 web App，以及 localStorage 紀錄最後去的連結頁面。[這篇](http://www.andymercer.net/blog/2016/02/full-screen-web-apps-on-ios/)結合了上面所提的超連結問題及 web App 前景背景切換問題，並有一步一步的解說及範例，相當詳細。

最後，[Appetize.io](https://appetize.io/demo) 提供了各種不同機型及 OS 版本的線上模擬功能對於測試 Web App 來說算是蠻方便的工具。
相較之下 Android 在這塊的設定上相當方便，也沒遇到什麼大問題，甚至幫 iOS 寫了部分教學範例呢......

參考資料：
1. [Configuring Web Applications](https://developer.apple.com/library/archive/documentation/AppleApplications/Reference/SafariWebContent/ConfiguringWebApplications/ConfiguringWebApplications.html)
2. [html - Is it possible to force iphone/ipod to update apple-touch-icon once webapp is added to home screen? - Stack Overflow](https://stackoverflow.com/a/4587917/4968420)
3. [Stay Standalone: Prevent links in standalone web apps opening Mobile Safari](https://gist.github.com/irae/1042167)
4. [html - apple-mobile-web-app-status-bar-style in ios 10 - Stack Overflow](https://stackoverflow.com/a/40786240/4968420)
5. [Adding Custom iOS Splash Screens To Your Progressive Web App](https://medium.com/appscope/adding-custom-ios-splash-screens-to-your-progressive-web-app-41a9b18bdca3)
6. [Progressive Web App Splash Screens – Dave Hudson – Medium](https://medium.com/@applification/progressive-web-app-splash-screens-80340b45d210)
7. [Full Screen Web Apps on iOS - Andy Mercer](http://www.andymercer.net/blog/2016/02/full-screen-web-apps-on-ios/)
