title: 使用 Travis CI 自動部署 GitHub Pages
date: 2016-01-21 16:23:05
categories:
- study
tags:
- hexo
- travis ci
- gf-pages
permalink: using-TravisCI-to-deploy-on-GitHub-pages
---
## 前言
先前看到了[程式人就該有個部落格]( https://samkuo.me/post/2015/09/why-developers-should-have-a-blog/)，想想這個主意不錯，之前有在 Blogger 及 Logdown 零星寫了幾篇文章，也曾經想過自己架靜態或動態網站，只是一直沒有去做，看完這篇讓我更有動力去做這件事。研究一陣子後我選擇直接利用 Github pages 來發佈網站，於是開始調查 [StaticGen](https://www.staticgen.com/) 上排名較高的幾個，而最後挑選了 [Hexo](https://hexo.io)：是台灣人寫的，而且是 javascript。

自己完成 Hexo 初步設定，產生第一版頁面後不久就遇到兩個問題：
1. 能不能將每次都要發佈的這個動作自動化。
2. 如果我換到其他電腦上，而其他電腦可能沒有 Hexo 時會很不方便，可能要選擇安裝一次 node.js 及 hexo，甚至根本沒安裝權限；若是能直接利用 GitHub 線上編輯 markdown 文件就能產生的話有多棒。

因為這兩點，我決定開始我的 Travis CI 初體驗。

## 關於 Travis CI
簡單來說，持續整合 (Continuous integration，縮寫為 CI)是在開發過程中，有任何變更都自動且持續的整合到目前的版本中。整合包含測試及發佈，可根據自訂的測試內容產生可視化的結果，方便開發人員快速找到問題所在，並且在測試通過後自動執行已撰寫的腳本，以達到自動發佈的功能。要達到持續整合，需有一個伺服器專門監聽程式版本的改動，一旦有變動就執行事先撰寫的測試及部署腳本。

Travis CI 提供在 GitHub 上的任何公開的 repo 都可以免費的使用 CI 服務，Travis CI 與 GitHub 的適性很好（也只提供使用 GitHub 帳號登入），廣受 GitHub 上使用，因此在這裡也使用 Travis CI 所提供的服務來產生靜態網站。

初次接觸 CI 可以先從官方提供的範例檔開始：[Travis CI for Complete Beginners](https://docs.travis-ci.com/user/for-beginners)，以便能有一些基礎概念，接著再開始挑選 [Getting started](https://docs.travis-ci.com/user/getting-started/) 中的項目學習設定與操作。

我的目標很明確，想要弄出在同一個 repo 下，一個 branch 是放 source code 的 master，另一個 branch 則是發佈用的 gh-pages。每當我 master 有更新時 gh-pages 也會自動透過 Travis CI 更新，如下圖，經過幾次測試後終於成功，最後 branch 的點呈現交錯成長：

![branch](/blog/images/sourceTree.png "branch")

## 給予 Travis CI push 的權限
由於發佈到 gh-pages 要交給 Travis CI 處理，需要 GitHub 帳號的驗證，而在 public repo 下不可能直接把密碼直接放在 source 中，因此在這裡選擇 GitHub 所提供的 [Personal access tokens](https://github.com/settings/tokens) 來處理權限的問題，用 Personal access token 的好處在於是個人創建的，可以隨時刪除 token 以取消存取權限，再加上 Travis CI 在文件中提到的 [Encryption keys](https://docs.travis-ci.com/user/encryption-keys/) 來處理敏感資料，通過環境變數的方式傳遞給腳本，以避免密碼及 token 公開出來。

首先先產生一個 access token，因為目的只有讓 Travis CI 可以讀取 public repo，因此勾選 public repo 即可。

![personal access token](/blog/images/personalAccessToken.png "personal access token")

接著先將產生的 token 妥善複製，未來只能 regenerate 一組新的 token，再也無法從 GitHub 調出目前這組。

![generated token](/blog/images/generatedToken.png "generated token")

接著利用 Travis CLI 來處理敏感資料，較方便的方式是利用 ruby 的 gem 來安裝 Travis CLI：

```
gem install travis
```

安裝完畢後，接著到想設定 Travis CI 的 repo 目錄中執行 `travis login` 來驗證身分，之後執行 `travis init`，會先詢問使用的語言，且產生 `.travis.yml`，接著在同一目錄下執行此指令，記得將 `<Personal Access Token>` 取代成先前複製的那組：

```
travis encrypt 'GIT_NAME="North" GIT_EMAIL=ssk7833@gmail.com GH_TOKEN=<Personal Access Token>' --add
```

即可看到在 .travis.yml 中多了

```
env:
  global:
    secrue: "long secure base64 string"
```

這一串將在每次 CI 進行時設定環境變數，這邊環境變數即可在接下來的腳本中使用。

## 設定 .travis.yml 檔
編輯 .travis.yml 前，可以先閱讀一下 Travis CI 的 [Build Lifecycle](https://docs.travis-ci.com/user/customizing-the-build/)，以下是我粗略的設定：

```
language: node_js

node_js:
  - "4.0"

env:
  global:
    secure:  "long secure base64 string"

install:
  - npm install

script:
  # Set Git config
  - git config --global user.name "$GIT_NAME"
  - git config --global user.email "$GIT_EMAIL"
  - git config --global push.default simple
  - git clone --depth 1 --branch gh-pages https://$GH_TOKEN@github.com/ssk7833/blog public
  # Generate Hexo static pages
  - npm run generate
  - cd public
  - git add -A .
  - MESSAGE=`date +\ %Y-%m-%d\ %H:%M:%S`
  - git commit -m "Site updated:$MESSAGE"
  - git push --quiet
```

node.js 的套件 dependencies 都已先用 package.json 存下，因此在 install 的部分只需使用 npm install；在 script 中完成部分指令，但因為沒特殊需求，只有設定 git 及產生靜態頁面，因此讓它一路到底。

**注意：git push 時一定要加 `--quiet`，否則先前設定的 Personal Access Token 將會印出，這樣就失去加密意義了。**

結果可以在 Travis CI 的網頁上看到，可以瀏覽各次的狀況，像我最近的 [push 結果](https://travis-ci.org/ssk7833/blog/builds/101307260 )及先前測試的[失敗結果]( https://travis-ci.org/ssk7833/blog/builds/100311173)都可以在 Build history 中瀏覽到。

## 在 GitHub 上發佈／編輯
若是設定無誤，接下來要發佈或編輯文章即可直接利用 GitHub 網頁版來作編輯，不需要擔心作業系統沒有安裝相關環境而無法發佈或編輯文章囉！

![edit post](/blog/images/editPost.png "edit post")

**UPDATE：**發現 Travis CI 發佈的結果可能會跟實際時間對不起來，如圖：

![time zone incorrect](/blog/images/TZIncorrect.png "time zone incorrect")

後來發現是因為我在 Hexo 中設定了時區為 Asia/Taipei，而 Travis CI 所提供的機器時區不一樣而造成的，將 Travis CI 一樣設定為 Asia/Taipei 即可解決問題。

```
before_install:
  - export TZ=Asia/Taipei
```

這是我最後的 [.travis.yml 設定](https://github.com/ssk7833/blog/blob/master/.travis.yml)。

參考資料：
1. [用 Travis-CI 生成 Github Pages 博客 ](https://farseerfc.me/zhs/travis-push-to-github-pages-blog.html)
2. [When Hexo Meets GitHub Pages and Travis CI plus Raspberry Pi](http://changyuheng.me/2015/when-hexo-static-site-meets-github-pages-and-travis-ci/)
