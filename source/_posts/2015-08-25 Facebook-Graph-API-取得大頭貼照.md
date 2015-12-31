title: Facebook Graph API 取得大頭貼照
date: 2015-08-25 03:11:56
categories:
- study
tags:
- Facebook
- Facebook-Graph-API
permalink: facebook-graph-api-get-picture
---
要取得大頭貼照，如果使用 API request ，加上 picture 即可得到大頭貼照，如：`me?fields=id,name,picture`，即可在回傳的 JSON 中取得圖片的位址，但這時取回來的圖片會比正常大小還要小，而且此方法還需要 access_token。

要取得不同大小的大頭貼照，有個更輕鬆的方法：`http://graph.facebook.com/{id}/picture?type=normal` 直接使用此網址，將 id 換成想呈現的 userId 即可，此網址將會 redirect 到對應的圖片位址，且此方法不需要 access_token。

在這個網址中，type 可為 `small`, `normal`, `album`, `large`, `square`，分別為不同解析度的照片大小。

以 Facebook 的創始人 Mark Zuckerberg 為例，userId 為 4，則要顯示的網址如下：
50*50:
`http://graph.facebook.com/4/picture?type=small`
![Small](http://graph.facebook.com/4/picture?type=small "Mark Zuckerberg")
`http://graph.facebook.com/4/picture?type=album`
![Album](http://graph.facebook.com/4/picture?type=album "Mark Zuckerberg")
`http://graph.facebook.com/4/picture?type=square`
![Square](http://graph.facebook.com/4/picture?type=square "Mark Zuckerberg")

100*100:
`http://graph.facebook.com/4/picture?type=normal`
![Normal](http://graph.facebook.com/4/picture?type=normal "Mark Zuckerberg")

200*200:
`http://graph.facebook.com/4/picture?type=large`
![Large](http://graph.facebook.com/4/picture?type=large "Mark Zuckerberg")


不能理解的是為什麼 small, album, square 所得到的大小都一樣，還不知道差在哪。
