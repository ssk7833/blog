title: Parse Server APNS(iOS) 推播通知設定
date: 2016-04-19 15:56:37
categories:
- study
tags:
- node.js
- Parse server
- APNS
- push notification
permalink: parse-server-push-notification-APNS
---
Parse 的一大方便之處，即是他提供了推播通知(push notification)的整合，使開發者可以在短時間內完成推播功能；而推播通知也已經移植到 Parse Server 版本上，步驟與先前在 Parse 上差不了多少，在這裡紀錄一下 APNS(iOS) 設定的步驟。

## 建立 SSL 憑證
在發送推播前必須給予 Parse Server 推播的權限，因此要先建立對應的 App ID 及 SSL 憑證。
###  1. 建立 Explicit App ID
如果原本就已經有建立 Explicit App ID 的話，請跳過此步驟到 [2. 設定推播通知](#2-設定推播通知)。
1. 到 [Apple Developer Member Center](https://developer.apple.com/membercenter/) 登入，點選 [Certificates, Identifiers & Profiles](https://developer.apple.com/account/ios/certificate/)。
2. 點選左欄中 Identifiers 底下的 [App IDs](https://developer.apple.com/account/ios/identifier/bundle)。
3. 點選右上角 + 的按鈕。
![建立 App ID](/blog/images/2016-04-19-Parse-server-APNS/01.png "建立 App ID")
4. 在 App ID Description 填上想要的名稱。
5. 選擇 App ID Prefix，我的只有一組，好像預設就會自己選了～
6. App ID suffix 要注意選擇 Explicit App ID，而 Bundle ID 可參考 Apple 推薦的填法或是自己偏好的格式，須注意這組之後會在 Xcode 中使用到。
![Explicit App ID](/blog/images/2016-04-19-Parse-server-APNS/02.png "Explicit App ID")
7. 將有需要用到的服務打勾，Push Notifications 記得要打勾！其餘就看自己有沒有要用到再開啟。
![Apple Services](/blog/images/2016-04-19-Parse-server-APNS/03.png "Apple Services")
8. 點選 Contiune 進行下一步，確認無誤後就可以送出了。

### 2. 設定推播通知
到這裡，應該已經建立好一個 Explicit App ID。
1. 點開  [App IDs](https://developer.apple.com/account/ios/identifier/bundle) 底下已建立的 App ID，再點選 Edit。
![App IDs Edit](/blog/images/2016-04-19-Parse-server-APNS/04.png "App IDs Edit")
2. 找到 Push Notifications 項目，若沒 Enable 則將他打勾，可以在此建立 Development 跟 Production 的憑證，建議先從開發模式開始，點選 Development SSL Certificate 中底下的 Create Certificate...。
![Create Certificate](/blog/images/2016-04-19-Parse-server-APNS/05.png "Create Certificate")
3. 接下來他會教你怎麼做，然後要你做完再點選 Contiune，因為我英文對應中文 MAC OS 不太熟，所以在這裡也把步驟打出來了，開啟 MAC 中的「鑰匙圈存取」。
![鑰匙圈存取](/blog/images/2016-04-19-Parse-server-APNS/06.png "鑰匙圈存取")
4. 點選「鑰匙圈存取」→「憑證輔助程式」→「從憑證授權要求憑證…」（蠻饒舌的）。
![從憑證授權要求憑證…](/blog/images/2016-04-19-Parse-server-APNS/07.png "從憑證授權要求憑證…")
5. 在「使用者電子郵件位址」輸入自己的 Email，「一般名稱」輸入自己想要的名稱，「CA 電子郵件位址」留白，「已將要求」選擇「儲存到硬碟」，接著點選「繼續」來產生 CSR 檔。
![憑證輔助程式](/blog/images/2016-04-19-Parse-server-APNS/08.png "憑證輔助程式")
6. 將剛剛儲存的 CSR 檔案上傳。
![Upload CSR file](/blog/images/2016-04-19-Parse-server-APNS/09.png "Upload CSR file")
7. 下載憑證，下載完成後點選兩下此檔案，使檔案安裝到「鑰匙圈存取」中。
![Download certificate](/blog/images/2016-04-19-Parse-server-APNS/10.png "Download certificate")
8. 開啟「鑰匙圈存取」，在左欄點選「我的憑證」，你可能會看到 Apple Development Push Services: 及 Apple Push Services:，這兩個分別對應了 development 憑證及 production 憑證，端看你使用哪一個。
![Apple Development Push Services: & Apple Push Services:](/blog/images/2016-04-19-Parse-server-APNS/11.png "Apple Development Push Services: & Apple Push Services:")
9. 在要使用的憑證上點選右鍵，選擇「輸出」項目，儲存名稱依自己喜歡而定。
![Export certificate](/blog/images/2016-04-19-Parse-server-APNS/12.png "Export certificate")
10. 在按儲存時會跳出密碼詢問的視窗，記得**留白**！
![Leave it blank](/blog/images/2016-04-19-Parse-server-APNS/13.png "Leave it blank")
11. 最後產生的檔案，將此檔案放到 Parse Server 的目錄內。
![.p12](/blog/images/2016-04-19-Parse-server-APNS/14.png ".p12")

## Parse Server 設定 APNS
延續 [Parse Server 架設教學](/blog/2016/04/09/setup-parse-server/)中所使用的程式碼，在 `new ParseServer` 中加入 push 相關的程式碼，如下：
```javascript
var api = new ParseServer({
  databaseURI: databaseUri || 'mongodb://localhost:27017/dev',
  cloud: process.env.CLOUD_CODE_MAIN || __dirname + '/cloud/main.js',
  appId: process.env.APP_ID || '7c6a1d1470fed0313b5044c4eb83def0',
  masterKey: process.env.MASTER_KEY || '98584a6e0a2592c274d1e4eae44b0a7b', // Add your master key here. Keep it secret!
  serverURL: process.env.SERVER_URL || 'http://localhost:1337/parse',  // Don't forget to change to https if needed
  liveQuery: {
    classNames: ["Posts", "Comments"] // List of classes to support for query subscriptions
  },
  // 以下為新增部分
  push: {
    // 此篇未提到 Android，因此註解掉
    // android: {
    //   senderId: '...',
    //   apiKey: '...'
    // },
    ios: {
      pfx: 'pushDevelopmentCertificate.p12', // 與 index.js 目錄同層
      bundleId: 'com.pushTest', // 填入先前填的 Bundle ID
      production: false // false: development, true: production
    }
  }
});
```

而若需要同時使用 development 及 production 的 APNS 時，可以將設定改為這樣：
```javascript
  push: {
    ios: [
      {
        pfx: '', // Dev PFX or P12
        bundleId: '',
        production: false // development
      },
      {
        pfx: '', // Prod PFX or P12
        bundleId: '',  
        production: true // production
      }
    ]
  }
```

## 設定 client apps
App 部分的設定與先前 Parse 的設定一樣，因此在這裡省略，可以參考 [Push Configuring Clients](https://github.com/ParsePlatform/Parse-Server/wiki/Push-Configuring-Clients) 尋找自己需要的程式語言寫法。

## 測試推播通知
推播傳送的方式一樣可用 curl 或是 cloud code，唯一要注意的是傳送推播通知需要 `masterKey`，可以參考 [Send Push Notifications](https://github.com/ParsePlatform/parse-server/wiki/Push#4-send-push-notifications)。而 [Parse Dashboard](https://github.com/ParsePlatform/parse-dashboard)目前也可以傳送推播通知了，只可惜目前只可以傳而不能看傳送紀錄，下圖為 parse-dashboard 1.0.8 的畫面。
![Parse Dashboard send push](/blog/images/2016-04-19-Parse-server-APNS/15.png "Parse Dashboard send push")

## 疑難雜症
正常來說過沒多久就 app 就能收到推播通知，如果沒有成功的話可以新增這兩個環境變數 `VERBOSE=1` 及 `DEBUG=apn`，若 `VERBOSE=1` 看不出結果再觀察 `DEBUG=apn`，其餘問題可能就要爬 issues 了。

筆者遇到的問題很蠢，就是所在的網路環境中 port 2195 被鎖了，因此試了半天都推播失敗 Orz，請先確定自己到 `gateway.sandbox.push.apple.com:2195` (development) 及 `gateway.push.apple.com:2195` (production) 是否能通～

參考資料：
1. [parse-server wiki - push](https://github.com/ParsePlatform/parse-server/wiki/Push)
2. [PushTutorial - Push Notification Sample App](https://github.com/ParsePlatform/PushTutorial/blob/master/iOS/README.md)
3. [Parse Dashboard](https://github.com/ParsePlatform/parse-dashboard)
