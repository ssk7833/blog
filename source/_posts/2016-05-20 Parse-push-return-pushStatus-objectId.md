title: Parse 推播回傳 _pushStatus 的 id
date: 2016-04-21 10:06:51
categories:
- study
tags:
- node.js
- Parse server
- push notification
permalink: Parse-push-return-pushStatus-objectId
---
在使用 parse server REST API 的推播功能時，推播成功送到 parse server 只會時回傳 `{"result":true}`，只有這資訊對於後台串接 parse server 不是很方便，因缺乏 `_pushStatus` 中的 id，導致資料送出後就一去無回，不方便針對各個送出到 parse server 的資料做對應。

```bash
curl -X POST \
  -H "X-Parse-Application-Id: parseAppId" \
  -H "X-Parse-Master-Key: parseMasterKey" \
  -H "Content-Type: application/json" \
  -d '{
        "where": {
          "deviceType": {
            "$in": [
              "ios"
            ]
          }
        },
        "data": {
          "title": "The Shining",
          "alert": "All work and no play makes Jack a dull boy."
        }
      }' \
  http://localhost:1337/parse/push/
```

正常下會得到：

```javascript
{"result":true}
```

所幸在 parse server 2.2.7 版時新增了 `X-Parse-Push-Status-Id` 這個 header，如同現行的 Parse.com 一樣，因為考量到回傳 object id 可能會暴露給 clients，因此放在 response 的 header 中。如何驗證 header 確實存在 `X-Parse-Push-Status-Id`，可以利用 curl 加上 -v 來看整個 request, response 的結果。

```bash
curl -X POST \
  -H "X-Parse-Application-Id: parseAppId" \
  -H "X-Parse-Master-Key: parseMasterKey" \
  -H "Content-Type: application/json" \
  -d '{
        "where": {
          "deviceType": {
            "$in": [
              "ios"
            ]
          }
        },
        "data": {
          "title": "The Shining",
          "alert": "All work and no play makes Jack a dull boy."
        }
      }' \
  http://localhost:1337/parse/push/ \
  -v
```

得到類似結果，回傳的 header 中就能看到 X-Parse-Push-Status-Id：

```bash
*   Trying 127.0.0.1...
* Connected to localhost (127.0.0.1) port 1337 (#0)
> POST /parse/push/ HTTP/1.1
> User-Agent: curl/7.41.0
> Host: localhost:1337
> Accept: */*
> X-Parse-Application-Id: parseAppId
> X-Parse-Master-Key: parseMasterKey
> Content-Type: application/json
> Content-Length: 125
>
* upload completely sent off: 125 out of 125 bytes
< HTTP/1.1 200 OK
< X-Powered-By: Express
< Access-Control-Allow-Origin: *
< Access-Control-Allow-Methods: GET,PUT,POST,DELETE,OPTIONS
< Access-Control-Allow-Headers: X-Parse-Master-Key, X-Parse-REST-API-Key, X-Parse-Javascript-Key, X-Parse-Application-Id, X-Parse-Client-Version, X-Parse-Session-Token, X-Requested-With, X-Parse-Revocable-Session, Content-Type
< X-Parse-Push-Status-Id: gLdaMNg12i
< Content-Type: application/json; charset=utf-8
< Content-Length: 15
< Date: Fri, 20 May 2016 07:51:58 GMT
< Connection: keep-alive
<
{"result":true}* Connection #0 to host localhost left intact
```

有了這個資訊後，就可以在後台中做到許多變化，以下為 node.js 的範例：

```javascript
var http = require('http');

var config = {
  parseLocation: 'localhost',
  parsePort: 1337,
  parsePush: '/parse/push',
  parseAppId: 'parseAppId',
  parseMasterKey:'parseMasterKey',
};

// 範例用 object
var obj = {
  "where": {
    "deviceType": {
      "$in": ["ios"]
    }
  },
  "data": {
    "title": "The Shining",
    "alert": "All work and no play makes Jack a dull boy."
  }
}

var req = http.request({
  host: config.parseLocation,
  port: config.parsePort,
  path: config.parsePush,
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-Parse-Application-Id': config.parseAppId,
    'X-Parse-Master-Key': config.parseMasterKey
  }
}, function(res) {
  var jsonString = '';
  res.on('data', (chunk) => {
    jsonString += chunk;
  });

  res.on('end', () => {
    var json = JSON.parse(jsonString);
    if(json.result===true) {
      // 在這裡確認 result 為 true 後，將 x-parse-push-status-id 存在某個 obj 中來做對應
      console.log(res.headers['x-parse-push-status-id']); // 印出 x-parse-push-status-id
      obj.pushId = res.headers['x-parse-push-status-id'];
    }
  })
});

var postData = JSON.stringify(obj);

req.write(postData);
req.end();
```
最後 obj 會得到類似結果：
```javascript
{
  "where": {
    "deviceType": {
      "$in": ["ios"]
    }
  },
  "data": {
    "title": "The Shining",
    "alert": "All work and no play makes Jack a dull boy."
  },
  "pushId": "kAuJhPqpt9"
}
```

接下來怎麼儲存物件及使用這些資料就是另一個課題了～

參考資料：
1. [Adds X-Parse-Push-Status-Id header](https://github.com/ParsePlatform/parse-server/pull/1412)
2. [Return PushStatus ID from push endpoint.](https://github.com/ParsePlatform/parse-server/issues/1157)
3. [Sending Pushes](https://parse.com/docs/rest/guide#push-notifications-sending-pushes)
