---
title: "Https Github Pages With Hugo"
date: 2018-12-14T21:23:48+08:00
categories: [Golang, GitHub, 架站]
tags: [GitHub, Cloudflare, Hugo]
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
Github Pages 允許你使用自定義域名