title: 取得 Parse server 推播通知送出狀態
date: 2016-06-06 11:02:00
categories:
- study
tags:
- node.js
- Parse server
- push notification
permalink: retrieve-push-status
---
在 [Parse REST API Developers Guide](https://parse.com/docs/rest/guide) 中，可以在 Quick Reference 看到所有使用的方法。在 Push Notification 分類中只有 POST 的方法，不像 Installations 可以透過 GET 方法到 `/installations/<objectId>` 來獲取已安裝裝置的資訊；然而若需要去擷取推播傳遞的狀態時，可以利用存於 mongoDB 中的 _PushStatus 來取得資訊，既然 push 的方法只有 POST，就繞個路把 _PushStatus 當成 Objects 來處理，Objects 的使用方法就多了。

| URL                                         | HTTP Verb | Functionality                                                                      |
|---------------------------------------------|-----------|------------------------------------------------------------------------------------|
| /classes/&lt;className&gt;                  | POST      | [Creating Objects](https://parse.com/docs/rest/guide#objects-creating-objects)     |
| /classes/&lt;className&gt;/&lt;objectId&gt; | GET       | [Retrieving Objects](https://parse.com/docs/rest/guide#objects-retrieving-objects) |
| /classes/&lt;className&gt;/&lt;objectId&gt; | PUT       | [Updating Objects](https://parse.com/docs/rest/guide#objects-updating-objects)     |
| /classes/&lt;className&gt;                  | GET       | [Queries](https://parse.com/docs/rest/guide#queries)                               |
| /classes/&lt;className&gt;/&lt;objectId&gt; | DELETE    | [Deleting Objects](https://parse.com/docs/rest/guide#objects-deleting-objects)     |

要取得推播通知狀態則是利用上表兩個 GET 的功能，若指定 objectId 則只回傳 objectId 那筆資訊，若不指定則回傳最近的幾筆資訊。

```bash
curl -X GET \
  -H "X-Parse-Application-Id: parseAppId" \
  -H "X-Parse-Master-Key: parseMasterKey" \
  http://localhost:1337/parse/classes/_PushStatus/
```

正常下會得到：

```javascript
{
  "results":[
    {"ACL":{}, "objectId": "xzBThEHVyc", "createdAt": "2016-05-19T10:00:59.827Z",…},
    {"ACL":{}, "objectId": "VGo2rHGXjK", "createdAt": "2016-05-19T10:01:59.790Z",…},
    {"ACL":{}, "objectId": "9rnRdakYzD", "createdAt": "2016-05-19T10:04:59.798Z",…},
    {"ACL":{}, "objectId": "7pde1mbqzY", "createdAt": "2016-05-19T13:00:09.126Z",…},
    {"ACL":{}, "objectId": "M2PmduKLH0", "createdAt": "2016-05-20T02:26:59.997Z",…},
    {"ACL":{}, "objectId": "URiaNdDBps", "createdAt": "2016-05-20T02:35:59.991Z",…},
    {"ACL":{}, "objectId": "eN8R4gVDEx", "createdAt": "2016-05-20T02:37:00.077Z",…},
    {"ACL":{}, "objectId": "KRBif8iM6u", "createdAt": "2016-05-20T03:13:59.892Z",…},
    {"ACL":{}, "objectId": "BP5FYt2GVX", "createdAt": "2016-05-20T03:24:59.846Z",…},
    {"ACL":{}, "objectId": "1nSmDpZ3Yz", "createdAt": "2016-05-24T02:55:01.595Z",…},
    {"ACL":{}, "objectId": "FDFjpei6rP", "createdAt": "2016-05-24T02:58:01.327Z",…},
    {"ACL":{}, "objectId": "hTkD1EKO9U", "createdAt": "2016-05-24T03:02:01.608Z",…},
    {"ACL":{}, "objectId": "TDT2bONPEF", "createdAt": "2016-05-24T07:51:02.636Z",…},
    {"ACL":{}, "objectId": "tGuEYvGLbp", "createdAt": "2016-05-25T07:35:20.034Z",…}
  ]
}
```

而若加上 objectId，則會取得此 objectId 單筆資訊：

```bash
curl -X GET \
  -H "X-Parse-Application-Id: parseAppId" \
  -H "X-Parse-Master-Key: parseMasterKey" \
  http://localhost:1337/parse/classes/_PushStatus/tGuEYvGLbp
```

正常下會得到：

```javascript
{
  "ACL": {},
  "objectId": "tGuEYvGLbp",
  "createdAt": "2016-05-25T07:35:20.034Z",
  "pushTime": "2016-05-25T07:35:20.034Z",
  "query": "{\"deviceType\":{\"$in\":[\"ios\"]},\"gender\":{\"$in\":[\"female\"]}}",
  "payload": "{\"alert\":\"All work and no play makes Jack a dull boy.\"}",
  "source": "rest",
  "status": "succeeded",
  "numSent": 1,
  "pushHash": "cd3f8c70e6cfecc66ab69a0ee4c3c564",
  "updatedAt": "2016-05-25T07:35:22.600Z",
  "numFailed": 0,
  "sentPerType": {
    "ios": 1
  }
}
```

兩者都可取得 `numSent`, `numFailed` 及 `sentPerType`，可利於推播通知的統計分析。

參考資料：
1. [REST API Developers Guide | Parse](https://parse.com/docs/rest/guide)
