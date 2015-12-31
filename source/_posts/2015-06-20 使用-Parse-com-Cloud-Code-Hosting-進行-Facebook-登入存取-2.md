title: 使用 Parse.com Cloud Code Hosting 進行 Facebook 登入存取 - 2
date: 2015-06-20 23:43:39
categories:
- study
tags:
- Facebook
- node.js
- Parse.com
- CloudCode
permalink: using-parsecom-cloud-code-hosting-to-log-in-with-facebook-2
---
延續上一篇，成功使用 Facebook 登入 Parse.com 的使用者資訊後，接著就是怎麼從使用者資訊中取得 Facebook 的資料了。

![User](/blog/images/user.png "User")
以上圖的 Facebook Test Users 為例，建立完的使用者可以由 Parse.com 的 Data 中看到，不過 authData 卻只顯示了 Facebook 的 ID，因此我們可以先透過 API Console 來對 users 作存取，這裡要注意的是 Use Master Key 記得要選 Yes，否則會沒有權限看 authData 的內容；users 後面的參數為 objectId，若不放置則會列出全部符合的資料。
![API console](/blog/images/APIconsole.png "API console")
在 Response 中的內容：
```
{
  "authData": {
    "facebook": {
      // 不要想對這組 access_token 亂來，因為是 test user XD
      "access_token": "CAAMCk3Pv7SkBAEjfvRaG4SrC8k3CXak1843iisuUJiIK9gYV9PNFRraXi9gxYVBJO83zsvzFO91dcACevKwinxAVPNCUeEv0UPWsmv7DZBlqPjtZCCnEBcMBKpU7ikoj9OKo1ZCwzi3wmTycsB2avHT1SiBxLUF5ZAHTaT9XDNtz1phGZCk0lltOY5agj0JGQ9ezNGmOsvUmdpKFASx5K",
      "expiration_date": "2015-08-15T17:52:46.495Z",
      "id": "118563748477765"
    }
  },
  "createdAt": "2015-06-16T17:52:48.623Z",
  "name": "Super Mario",
  "objectId": "wmVm7Qb1Fc",
  "sessionToken": "jEKKzfbDcIN0CmBIvsdZR9Aoc",
  "updatedAt": "2015-06-17T07:57:08.934Z",
  "username": "3GwDpgmuqmfnyGYINNI27W9fO"
}
```
已經知道存放在哪裡之後，接下來就是利用 cloud code function 建立資料存取了！在這裡我用名字與圖片作範例，參考了 [stackoverflow 這篇的回答](http://stackoverflow.com/a/16445118/4968420)稍微修改了一下：
當然，別忘記 userMasterKey...
```
Parse.Cloud.define("facebook", function(request, response) {
  Parse.Cloud.useMasterKey();

  new Parse.Query(Parse.User).get(request.params.user_id).then(function(user) {
    var authData = user.get("authData");

    // Quit early for users who aren't linked with Facebook
    if (authData === undefined || authData.facebook === undefined) {
      response.success(null);
       return;
    }

    return Parse.Cloud.httpRequest({
      method: "GET",
      url: "https://graph.facebook.com/me",
      params: {
        access_token: authData.facebook.access_token,
        fields: "name, friends",
      },
    });

  }).then(function(json) {
    response.success(json.data);

  // Promises will let you bubble up any error, similar to a catch statement
  }, function(error) {
    response.error(error);
  });
});
```
在 Express 中 call 建立好的 cloud host function：
```
app.get('/test', fbLogin, function(req, res) {
  var user = Parse.User.current();
  Parse.Cloud.run('facebook', { user_id: user.id }, {
    success: function(results) {
      res.send('Congrats, you are logged in, ' + results.name + '!' +  '<img src="https://graph.facebook.com/'+ results.id +'/picture?type=normal">');
    },
    error: function(error) {
      console.log("error :" + error);
    }
  });
});
```
若是成功的話就能在自己的 URL 中看到如下圖的結果了！
![Facebook success](/blog/images/facebookSuccess.png "Facebook success")
