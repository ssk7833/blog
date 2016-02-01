title: 使用 Parse.com Cloud Code Hosting 進行 Facebook 登入存取 1
date: 2015-06-20 22:13:10
categories:
- study
tags:
- Facebook
- node.js
- Parse.com
- CloudCode
permalink: using-parsecom-cloud-code-hosting-to-log-in-with-facebook-1
---
[Parse.com](http://parse.com/) 在 [javascript SDK](https://parse.com/docs/tw/js/guide) 中提供了使用者的存取，其中包含 Facebook 的整合，但 javascript 終究是前端，有些不想讓 client end知道的還是放在後端處理比較好。然而 Parse.com 所提供的 cloud code 所使用的目前雖是 node.js，會讓開發者很想直接把 javascript SDK 的 Facebook 部分塞進去看能不能跑，乍看之下很合理，但實際上就是不行，因為以 Parse.com Javascript SDK在要求登入時會跳出另一個瀏覽器視窗以要求登入 Facebook 及權限給予，而這個視窗當然沒有辦法在 server end 中呈現並要求 client end 進行認證。

在官方論壇上也有人發表過此問題：[Interacting with the Facebook API in Cloud Code](https://www.parse.com/questions/interacting-with-the-facebook-api-in-cloud-code)
所得到的回答是：
**Unfortunately, the Facebook JavaScript SDK is not made to work outside of a browser, so using it directly from Cloud Code is not supported at the moment.**

**You can, however, get the authData from the current user in cloud code and use that to make a REST call to Facebook's graph API manually.**

因此我轉而研究使用OAuth方式來登入Facebook，除了跳轉出的頁面比較美觀外（不會產生另一個瀏覽器視窗），也不用擔心暴露資訊給clent end，參考資料有這兩篇：
[網站利用 Facebook 帳號登入 (使用 OAuth)](http://sweeteason.pixnet.net/blog/post/40581580-%E7%B6%B2%E7%AB%99%E5%88%A9%E7%94%A8-facebook-%E5%B8%B3%E8%99%9F%E7%99%BB%E5%85%A5-%28%E4%BD%BF%E7%94%A8-oauth%29)
[10分鐘理解OAuth和facebook登入原理](https://gigenchang.wordpress.com/2014/01/26/10%E5%88%86%E9%90%98%E7%90%86%E8%A7%A3oauth%E5%92%8Cfacebook%E7%99%BB%E5%85%A5%E5%8E%9F%E7%90%86/)
這篇以python Django framework實作，其實看code好像也蠻容易理解的：[Ghetto Facebook Registration with Django](http://nthn.me/posts/2012/facebook-registration.html)

後來正當我開始打算實作時，我找到了 [parse-facebook-user-session](https://github.com/ParsePlatform/parse-facebook-user-session) ，原來 Parse.com 早就把這個寫好了，根本不用自己去寫了，只需要按照他的說明一步一步來就行了（吧）！

結果證明事情果然不是我想的這麼簡單，不管怎麼弄就是跳出 **209 invalid session token** ，花了一段時間後找到 Parse.com 自己發的文章 [Session Migration Tutorial](https://www.parse.com/tutorials/session-migration-tutorial) ，才知道把這個選項關掉就行了，我花了這麼久到底在幹麻！

![User sessions](/blog/images/userSessions.png "User sessions")
總之，關閉這個選項後就成功了， Parse.com 的資料庫也會成功紀錄登入過的使用者，事情完成一半，其餘的就是登入的使用者資料該怎麼讀取出來了～

**UPDATE：**在GitHub上新增了 [demo](https://loginexample.parseapp.com/)：[GitHub](https://github.com/ssk7833/Parse-Facebook-OAuth-login-example)
