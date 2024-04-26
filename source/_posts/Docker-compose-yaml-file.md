---
title: Docker - 什麼是 Compose
date: 2024-02-22 23:04:11
category: Docker
---

之前我們介紹了 Dockerfile 的基本指令，

這次的主題我們要來介紹 docker-compose.yml 

### 關於 docker-compose.yml

#### 是什麼？

官方的說明


> Compose is a tool for defining and running multi-container Docker applications. 


Compose 是運行多個 container 的工具。

什麼意思呢？

如果專案中需要跑多個 container 起來，

我們就需要使用到 Compose 把他們集結起來，組合成我們的專案。

用生活中的例子來說明，

某個工廠有四位成員，

這四個成員就像是每個 container 一樣，在工廠中付出勞力或心力，

而 Compose 就像是工廠，

將這四位成員集中起來，並且保持產線的運作

#### 什麼時候該用它？

> 當我們的專案會同時跑多個 container 時，就會需要用到 compose

舉例來說，用 Rails 寫的專案，

我們需要跑一個 `Ruby` container，

要用 `Postgresql` 作為資料庫的話，就會需要另外再跑一個 container 

那如果不要用呢？

我們就要用指令一個一個設定 networks 是哪個，是否需要用 volumes 

指令會變成一大串，聽起來其實蠻麻煩的吧

#### docker-compose.yml 可以做什麼設定

docker-compose.yml 可以做很多設定，我們會透過這個檔案，告訴 Docker Compose 需要用哪些服務、要去監聽哪個 ports，甚至是設定環境變數

- version

docker-compose.yml 是使用哪個版本

- services

我們會設定這個專案會使用到哪些服務，例如由這個專案包起來的 image、資料庫的 image 等等

- platform

如果我們要運行的機器是特定的作業系統，就必須在這邊做設定
還有一個情況是，假設我們本機的作業系統與機器作業系統不同，也需要設定這個參數

- ports

Docker 會依照我們設定的 port ，來取決於 Docker 的哪個 port 要去監聽機器上的哪個 port

- environment

我們也可以在 docker-compose.yml 中設定環境變數。

不過環境變數什麼時候該在 dockerfile 設定，什麼時候又該在 docker-compose.yml 設定？

取決於 dockerfile 這顆做出來的 image 是否要重複使用，且環境變數是否一致，一致的話就可以設定在 dockerfile 中，不一致的話我們就可以設定在 docker-compose.yml 中

- restart

告訴 Docker 什麼時候開重新啟動

- networks

告訴 Docker 這顆 contailer 是使用哪個 networks 

- volumes

告訴 Docker 這顆 contailer 存放的資料放在哪個 volumes 


以上是我們在寫 docker-compose.yml 常用到的設定

`networks` 、 `volumes` 我們將在之後會提到。


### 撰寫 docker-compose.yml 檔案

接著我們來試做一個簡單的 `docker-compose.yml` 檔案


```
version: "3.9"

services:
  app:
    image: "ruby:3.2.0-alpine3.17"
```

寫好 docker-compose.yml 後，我們將 image 跑起來

```
> docker compose up
```

```
[+] Running 2/1
 ⠿ Network ishop_default  Created                                                                      0.1s
 ⠿ Container ishop-app-1  Created                                                                      0.0s
Attaching to ishop-app-1
ishop-app-1  | Switch to inspect mode.
ishop-app-1  | 
ishop-app-1 exited with code 0
```

#### 這個指令背後做了哪些事？

當我們下了 `docker compose up` 時， Docker 會先從 docker-compose.yml 解析，並且開始建立 image ，建立完成後就會跑 container

基本上就是 `build` + `run container` 的指令

![](https://hackmd.io/_uploads/ryf5H6ff6.jpg)

那如果我們更新了專案，還要再重新跑一次指令嗎？

不需要， `docker compose` 會去查看哪個地方改了，會更新修改的地方並且 `recreate container`

![](https://hackmd.io/_uploads/r1NUnTzMT.jpg)

### 將 container 停止

當我們下完以下指令，他就會 stop 所有的 container

```
> docker compose stop
```

```
[+] Running 3/3
 ⠿ Container ning_lab_blog-app-1       Stopped                                                                               0.4s
 ⠿ Container ning_lab_blog-traefik-1   Stopped                                                                               0.0s
 ⠿ Container ning_lab_blog-database-1  Stopped                                                                               0.3s
```

### 將 Docker Compose 中的 container 清理掉

如果我們要將所有 container 刪除，就可以使用這個指令

```
> docker compose down
```

```
[+] Running 3/3
 ⠿ Container ning_lab_blog-app-1       Removed                                                                               0.0s
 ⠿ Container ning_lab_blog-traefik-1   Removed                                                                               0.0s
 ⠿ Container ning_lab_blog-database-1  Removed                                                                               0.0s
```

什麼時候要用 docker compose stop 什麼時候要用 docker compose down ?

![](https://hackmd.io/_uploads/SkXDn6MMa.jpg)


以我自己的習慣，如果只是要讓專案暫停，就會使用 `docker compose stop`

但如果我今天是要重新建立 image 並且重跑，就會使用 `docker compose down` 請他將 Docker Compose 清乾淨


### 將 image 建立並且跑起來

其實這個指令跟 `docker compose up` 蠻像的，差別在於他會強制 `build image` 

```
> docker compose up --build
```

什麼時候要用 `docker compose up` 什麼時候要用 `docker compose up --build` ?

我也被這兩個指令搞混蠻久的，總結是

![](https://hackmd.io/_uploads/ryjPhpzfT.jpg)


當我們在包 image 的時候，就會用 `docker compose up --build` 讓他強制重新建立 image 

而 `docker compose up` 會偵測到專案更新的時候，會 recreate container ，所以通常用在已經不會再去改 image 只有修改專案內容的時候

