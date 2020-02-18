---
title: "使用 GitHub Action 部署部落格 (Hugo)"
date: 2020-02-16T15:48:03+08:00
tags: [ci/cd,hugo,github actions]
categories: [engineering]
---

這篇想稍微紀錄一下自己改採 GitHub Action 部署的過程，會想使用 GitHub 這個新功能，一部分是想知道有什麼東西之後可以利用的，另外一個讓自己更懶惰一點。哈哈。

---

## 什麼是 GitHub Action

[Github Action](https://help.github.com/en/actions) 簡單說起來就是 GitHub 自家推出的 CI/CD 工具，好處是跟放在 GitHub 上面的 Repository 有更好的整合，也可以更集中管理，有好有壞，但至少多了一個選擇。

設定的方式可以參考[官方文件](https://help.github.com/en/actions/configuring-and-managing-workflows/configuring-a-workflow)，基本上需要加入一個 `.github/workflows/<workflow_name>.yml`，然後在裡面定義不同的工作和工作內部的步驟，詳細設定就自己參考官方文件吧。

GitHub Action 有一個特色是開發人員可以開發 Action 並發布給大家使用，目前已經可以找到很多人已經做好的 Action 來使用，像是發布訊息到 Slack，寄信之類的，非常多。當然，這次要使用的 Hugo 編譯和 GitHub Page 部署也有人做好了。下面就快速帶過我的設定吧


## 部落格的內容管理和部署架構

我的部落格管理和部署是分屬在兩個不同的 repository，一個部落文章的資料庫，另一個是編譯後的靜態網站。

前者我是使用 [Hugo](https://gohugo.io/)，透過這個工具我可以滿方便的去維護我的文章內容，當時的考量是1、部落格主要是靜態內容，2、用 Markdown 來撰寫文章，3、使用 git 控管文章歷史。

後者我是採用 [GitHub Pages](https://help.github.com/en/github/working-with-github-pages/about-github-pages)，主要考量是部署的方便性以及懶得自己去管理另一台機器。因此，我要更新部落格的時候，主要是產生新的內容，並 `git commit` 到 `jimytc/jimytc.github.io` 的 `master` branch。剩下的真實部署部分，就交給 GitHub 負責了。

## 整體流程，從寫作到部署

整體來說寫文章跟部署來說都挺簡單的，大概也就下面這個步驟。

1. 建立新文章 ```$ hugo new```
2. 寫文章 VS Code
3. 存擋 ```$ git commit -m ""```
4. 編譯及部署 ```$ ./deploy.sh```

是的，`deploy.sh` 已經把很多步驟整合起來了，下面會稍微描述一下做了哪些事情。不過嘛，既然有更自動的工具可以完成，那就讓我連執行腳本的力氣都省下來吧。同時也順帶了解 GitHub Action，說不準之後就可以用上了。（當然啦，因為編譯和發布都不在本地端執行了，我的文章管理歷史會更乾淨一點）

所以呢，這次的目標是把上面的步驟變成如下。

1. 建立新文章 ```$ hugo new```
2. 寫文章 VS Code
3. 存擋 ```$ git add . && git commit -m "" && git push```

然後等待 GitHub Action 幫我做完剩下的事情。

## 開始吧

發布這個部落格主要是下面這兩個步驟。

### 編譯網站

執行 `hugo` 編譯網站，結果會產生在 `./public` 資料夾裡。

### 部署網站

把 `./public` 資料夾的變化透過 git 記錄下來後，上傳到 `jimytc/jimytc.github.io`。然後實際的網站，GitHub 會幫我們處理完。

### 需要完成的事情

1. 讓 GitHub Action Runner 把部落格和 Github Page repo checkout 到他的環境中
2. 要能夠執行 `hugo`
3. 要能夠從部落格 repo 的 GitHub Action Runner 提交改變到另一個 repo 上。

原先執行 `./deploy.sh` 的時候都在我的電腦上完成的時候，所有的工具和權限都不會有問題，但是因為現在要透過 GitHub Action 處理，就必須要去處理第二個人（雖然是機器人）相關的操作權限問題。

### 以上三件事情都有人幫我們做好了

1. Checkout -> 使用 `actions/checkout@v1`
2. 編譯部落格 -> 使用 `peaceiris/actions-hugo@v2` 然後執行 `hugo`
3. 部署到 GitHub Page repo -> 使用並設定 `peaceiris/actions-gh-pages@v3`

> 事情有這麼簡單就好了...自己挖的坑，自己跳進去，自己補起來。

### 設定 GitHub Action 的對於遠端 Repository 的存取權限

沒錯，因為我的部落格和 GitHub Page 分屬不同的 Git Repository，所以原本這件事情應該是在設定部署的時候才會發生，變成在 Checkout 的時候就踩坑。（謎之音：誰叫你把他們分開。）

沒關係，該做的還是要做，這邊基本上就是要產生一組公私金鑰，把私鑰放在部落格 repo 的設定中（GitHub 說會加密它，為了安全。），然後把公鑰加到 GitHub Page 的 Deploy Keys裡頭；對了，記得要把 `Allow write access` 打勾勾，不然做第三個設定的時候又要踩一次坑了。詳細步驟可以參考下面三個連結

* 設定公私鑰的[說明](https://github.com/peaceiris/actions-gh-pages#1-add-ssh-deploy-key
)
* 設定遠端 repo 的[說明](https://github.com/peaceiris/actions-gh-pages#%EF%B8%8F-deploy-to-external-repository)
* 指定分支為 `master`，[說明文件](https://github.com/peaceiris/actions-gh-pages#%EF%B8%8F-repository-type---user-and-organization)

### 設定完的 workflow 長這樣

```yaml
name: Hugo Build and Deploy

on:
  push:
    branches:
      - master

jobs:
  build-deploy:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v1  # v2 does not have submodules option now
        with:
          submodules: true

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.64.1'

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          deploy_key: ${{ secrets.ACTIONS_DEPLOY_KEY }}
          external_repository: jimytc/jimytc.github.io # <- 改成你自己的 user github page
          publish_dir: ./public # <- 指定編譯後的成品放置的資料夾
          publish_branch: master # <- 遠端 repo 的目標分支
```

## 感想

* 其實滿有趣的，GitHub Action 目前看起來挺方便的，但是一些設定的部分還是有一點隱藏。
* 透過設定和引用社群開發的功能，基本上就能達到想要的目的。
* 如果找不到合用的，可以寫自己的 Action。
* 其實可以把它拿來跑單元測試。
* 目前還沒看到可以拿來編譯 MacOS/iOS/iPadOS 的 Action，希望之後會有。


> 這篇更新後也是透過 GitHub Action 編譯和部署的喔！