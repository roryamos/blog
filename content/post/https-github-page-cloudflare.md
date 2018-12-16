---
title: "Https Github Pages With Hugo"
date: 2018-12-14T21:23:48+08:00
categories: [Golang, GitHub, 架站]
tags: [GitHub, Cloudflare, Hugo, HTTPS]
---

### Hugo
> hugo 是基于 Go 语言的静态网站生成器。有两种方式发布生成的静态网站文件：

### GitHub Pages
> GitHub Pages是 GitHub 提供的一種網站託管服務，用於託管GitHub用戶，用戶博客，項目文檔, 甚至整本書的靜態網頁。  
> GitHub Pages於2008年底推出。與 GitHub 的其他部分一樣，它包括免費和付費的服務層。通過此服務生成的網站作為github.io的子域名託管，或作為通過第三方域名註冊商購買的自定義域名託管。

### 安裝 Hugo
Mac 下直接使用 homebrew
```
$ brew install hugo
```
Windows 或其他作業系統 https://github.com/gohugoio/hugo/releases 到官方發佈頁下載.  
解壓縮後將檔案 copy 到環境變數 path 裡, 以Windows 為例就是在 C:\Windows\System32. 然後到命令列下執行 :
```
$ hugo version
Hugo Static Site Generator v0.52 windows/amd64 BuildDate: 2018-11-28T14:07:10Z
```
#### 生成站點
```
$ hugo new site test
$ tree test
D:\DOCUMENTS\TEST
├─archetypes //放一些template的地方, 可以客制化自已的 md template
├─content //文章的md檔
├─data //存放配置檔, 在生成網頁時參考
├─layouts //存放生成html時所用的template
├─static //存放靜態文件，在生成網頁時會一併複制過去
└─themes //存放主題, 可以挑選喜歡的主題使用
```
#### 創建文章
```
hugo new post/hello-world.md
```
編輯 content/post/hello-world.md
```markdown
\-\-\-
date: 2018-12-14T21:23:48+08:00
title: "hello-world"
\-\-\-

\#\#\# Hello World
```
#### 挑選主題
可以到官網去挑選一個喜歡的主題 https://themes.gohugo.io/  
我們以 maupassant 為例
```
git clone https://github.com/rujews/maupassant-hugo.git themes/maupassant
```
編輯 config.toml, https://github.com/rujews/maupassant-hugo 的readme 有更為詳細的配置
```
# 設置佈景主題
theme = "maupassant"
# 設置網頁發佈的目錄
publishDir = "docs"
```
啟動本地運行演示
```
hugo server
```
發佈網站頁面
```
hugo
```
### Github
建立 github 倉庫後, 按以下命令初始化倉庫
```
git init
git submodule add https://github.com/rujews/maupassant-hugo.git themes/maupassant
git add .
git commit -m "first commit"
git remote add origin git@github.com:roryamos/test.git
git push -u origin master
```
#### Github Pages
啟用倉庫的Github Page
1. 進入食庫的Settings
2. 在 Github Pages Source 的下拉選擇 master branch /docs folder
3. 按下 Save
4. 打開瀏覽器訪問 Github Pages 地址，例如: https://roryamos.github.io/test/

因為 Hugo baseurl 沒有配置正確的地址, 所以 js css 會加載不完全.
```
# 配置為正確地址
baseURL = "https://roryamos.github.io/test/"
```

#### Custom domain
Github Pages 允許使用自定義域名, 但經過測試子域名是無法使用https的喔.
如果想要使用自定義域名, 只需要在 Custom domain 欄位裡埴上自已的域名. 
然後去DNS管理把域名 CNAME 到 github.io 對應的域名去.
最後經過一段時間的等待後, 就可以 Enforce HTTPS 了

### Cloudflare
> Cloudflare 以向客戶提供網站安全管理、性能優化及相關的技術支持為主要業務。
通過基於反向代理的內容分發網絡（Content Delivery Network,CDN）
及分佈式域名解析服務（Distributed Domain Name Server），
Cloudflare可以幫助受保護站點抵禦包括拒絕服務攻擊在內的大多數網絡攻擊，
確保該網站長期在線，同時提升網站的性能、加載速度以改善用戶體驗。  
> Cloudflare 也提供免費且不限流量的 CDN 及 SSL 憑證, 
你可以輕易的讓你的域名加上https 的保護.

#### 域名管理
1. 首先要將域名 NS 配置到 Cloudflare 來做管理
1. 打開 Cloudflare 裡的 DNS 管理
1. 將要解析的子域名 CNAME 到 Github Pages 域名
1. 在解析的部份有分成兩種模式, 我們選擇 (ii)
    1. DNS only (直接將域名解析至地址)
    2. DNS and HTTP Proxy (也就是會透過Cloudflare的CDN 節點來代理你的http流量)
1. 打開 Crypto 管理, 在 SSL 區裡的下拉選單中選到 full

經過一段時間的等待就可以生效啦, Cloudflare 還有很多像是 Firewall, Page Rules 等好用的功能可以一個一個去體驗看看.