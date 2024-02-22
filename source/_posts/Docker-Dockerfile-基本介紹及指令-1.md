---
title: Docker - Dockerfile 基本介紹及指令
date: 2024-02-22 23:02:14
tags:
---

我們在先前文章提過，Dockerfile 是我們用來建立 Image 的檔案 

我們可以把 Dockerfile 想像成是千層蛋糕 每一層為一個 Layer Image 就是層層堆疊後的千層蛋糕 

![](https://i.imgur.com/ZTNfcdt.jpg) 

讓我們來看一下 DockerHub 裡的 Ruby 按到 Tags 頁籤，任意按一個版號進去，

就會看到像是歷史紀錄的畫面，這就是 Dockerfile 跑出來的層層 Layer 

到這邊應該還是難想像，我們來動手建立一個 Dockerfile 應該就會比較好理解 

![](https://i.imgur.com/GSspiC9.png) 

我們來依序介紹 Dockerfile 的指令，並且邊按照下列步驟練習 

``` 
# 用什麼語言寫的，AS node 可以當作是貼上一個貼紙 
FROM node:14-alpine AS node 

# 稍後會需要 yarn install，所以先安裝 yarn 
RUN apk add --no-cache yarn 

# 建立資料夾，可放入我們需要的檔案，或者在此執行 
WORKDIR /app 

# 複製 package.json 及 yarn.lock 到 /app 這個資料夾下 
COPY package.json yarn.lock /app/ 

# 有 package.json 及 yarn.lock，就可以先把套件裝起來了！ 
RUN yarn install 

# 套件安裝完畢，把所有檔案複製到 /app 中 
COPY . /app 

# 將 host 的 port 指向 Image 中 Node.js 的 port 
EXPOSE 9229:9229 

``` 
上面是針對一個由 Node.js 寫的專案建立的 Dockerfile 

基本上就是把我們平常架起環境的步驟 只是會再切分的細一點 

還有一些指令我們還沒提到，再來為大家統整一下 

``` 
FROM：描述這個 Image 的基底，假設我們今天要做的專案是以 ruby 寫的，就可以設定為 ruby:alpine3.17 
WORKDIR：建立一個資料夾，將需要的資料放到裡面，或者在其中執行指令 
RUN：要執行的指令，就像是我們要跑一個 Rails 的專案，需要先 bundle 
COPY：複製前面的資料 到 後面的資料夾 ADD：加入檔案 CMD：預設的指令 
EXPORT：對應到的 Port ENV：設定環境變數 
ARG：設定變數(將應用在 build 過程中) 
LABEL：此 Image 的註解 
VOLUME：指定 Container 的 Volume (通常會用來存放資料) ``` 當 Dockerfile 寫完後，我們就可以來把他打包成一顆 Image 了！ 

``` 
$ docker build -t  . 

# -t 為 Image 的 Tag 名稱 
#最後面的 . 是要告訴 Docker 你現在在哪兒 

``` 

想要更改 Tag 名稱的時候 

``` 
$ docker image tag ruby:latest ruby:Ning 

# 為了使用上方便，可以把 Image 修正為自己慣用的格式 

``` 

建立好 Image 後，就可以來跑 Container 

``` 
$ docker container run -d -p 3000:3000  

# -d 為背景執行的意思 # -p 為要將 Port 指到哪 
``` 

停止 Container 

``` 
$ docker container stop  
``` 

移除 Container 

``` 
$ docker container rm  

``` 

移除 Image 

``` 
$ docker image rm  
``` 

查看 Image 歷史紀錄 

``` 
$ docker image history  

# 顯示出 Image 每層 Layer，就像 Dockerhub 中的 Layer 

``` 

查看 Image 的 Metadata 

``` 
$ docker image inspect  

# 確認 Image 中的基本資訊 

``` 

以上是基本的 Image 及 Container 常用指令 Docker 的指令非常多，所以想要學好 Docker 就要常常來練習～