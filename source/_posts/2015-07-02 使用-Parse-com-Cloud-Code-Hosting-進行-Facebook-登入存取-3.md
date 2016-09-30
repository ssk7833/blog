title: 使用 Parse.com Cloud Code Hosting 進行 Facebook 登入存取 3
date: 2015-07-02 00:11:49
categories:
- study
tags:
- Facebook
- node.js
- Parse.com
- CloudCode
permalink: using-parsecom-cloud-code-hosting-to-log-in-with-facebook-3
---
繼上一篇成功截取出使用者資料後，發現除了基本資料外，朋友、按讚的資訊等資料其實都抓不出來，原因是因為沒有給予 app 存取這些資訊的權限。要求權限的話可以透過 OAuth 來索取 access token ，其範例網址如下：

`https://www.facebook.com/dialog/oauth?client_id={appId}&redirect_uri={redirectURI}`

![Profile](/blog/images/profile.png "Profile")
![Profile details](/blog/images/profile02.png "Profile Details")

這是一個截取基本權限的網址，appId 指的是每一個 app 獨立的 ID，而 redirectURI 是當 OAuth 通過後，會送發一串 code 到這個 redirectURI 去，而若需要要求其他權限，可以增加 scope 屬性如下：

`https://www.facebook.com/dialog/oauth?client_id={appId}&redirect_uri={redirectURI}&scope={accessPermissions}`

這個 scope 以逗號作為分隔，填在裡面的將會在 Facebook dialog 中要求權限。

![Profile with friends](/blog/images/user_friends.png "Profile with friends")
![Profile with friends details](/blog/images/user_friends02.png "Profile with friends details")

講了這麼多，但以[第一篇](/blog/2015/06/20/using-parsecom-cloud-code-hosting-to-log-in-with-facebook-1/)中使用了[parse-facebook-user-session](https://github.com/ParsePlatform/parse-facebook-user-session)該怎麼修改呢？稍微翻了它的 source code 後發現它在實作上並沒有保留 scope 欄位，因此我便把 scope 加上去了，可以由此瀏覽：[parse-facebook-user-session](https://github.com/ssk7833/parse-facebook-user-session)
**UPDATE：**原 repository 已經將此功能 merge上 去，直接使用原本的即可

使用方式的話則與先前的沒什麼差別，只是可以選擇多填一個 scope 欄位，範例如下：
```
app.use(parseFacebookUserSession({
  clientId: 'YOUR_FB_CLIENT_ID',
  appSecret: 'YOUR_FB_APP_SECRET',
  redirectUri: '/login',
  scope: 'user_friends,user_likes', // 要求friends與like資訊
}));
```
至於有哪些權限可以要求，可以當[https://developers.facebook.com/tools/explorer/](https://developers.facebook.com/tools/explorer/)中，點選 Get Access Token 來參考，並且在下面做測試。

不過要注意的有像是 `user_friends` 這項，如果在 API v2.0 以上的版本上要求資訊的話，只會列出同樣有授權此 app 的好友出來，開了幾個 test users 測試的確如此：

很可憐沒有朋友授權此 APP：
```
{
  "id": "104342733239984",
  "name": "Hello world",
  "friends": {
    "data": [],
    "summary": {
      "total_count": 1
    }
  }
}
```
可以看到 summary 中，total_count 為 1，但 data 中無資料。

有朋友也授權此 APP：
```
{
  "id": "1421116644879628",
  "name": "Doraemon Cat",
  "friends": {
    "data": [
      {
        "name": "Open Graph Test User",
        "id": "1414470195545509"
      },
      {
        "name": "Monkey D Luffy",
        "id": "100347703641647"
      }
    ],
    "paging": {
      "next": "https://graph.facebook.com/1421116644879628/friends?limit=25&offset=25&__after_id=enc_AdAMpWdRxSLZAvND6bEd0htyyGsZAZBvzP6jzoAIZBKS9EiBSndZCNZC3S1AC5TEYchbuuBSV0xvg7ziwO4Cdt843yZApF"
    },
    "summary": {
      "total_count": 2
    }
  }
}
```
可以看到此 test user 有兩個朋友也都有安裝此 app。

至於怎麼得到 test user 的 access token，我是利用 Parse.com 的 API Console，Endpoint 填入 users 且 Use Master Key 改成 Yes，send request 後即可在 response 中看到 access token，即可複製此 token 到 [https://developers.facebook.com/tools/explorer/](https://developers.facebook.com/tools/explorer/) 中做測試，如下圖。

![API console](/blog/images/APIconsole02.png "API console")
