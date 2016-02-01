title: Facebook Graph API 回傳指定語言/地區化姓名
date: 2015-07-12 18:49:43
categories:
- study
tags:
- Facebook
- localization
- Facebook-Graph-API
permalink: facebook-graph-api-returns-language-specific-name
---
玩 Facebook Graph API 玩了一陣子才發現回傳的姓名總是是英文的，才想到若是有回傳中文姓名的需求時該怎麼辦，如此下去一找才發現關鍵字是 [Language-specific name](https://www.facebook.com/help/217868321565724)，而要如何在 Facebook Graph API 中顯示為中文則可以參考這篇中的 locale：[Modifying API Requests](https://developers.facebook.com/docs/graph-api/using-graph-api/v2.0#readmodifiers)。

其實只要在 API request 中加上 `&locale=zh_TW` 即可得到中文姓名，如：`me?fields=id,name&locale=zh_TW`，只是有趣的是我稍微測了一下 locale 給以開頭 `en_` 以外的任何值都會取得中文名稱，還以為預設會以英文為主。
