---
title: Docker - 一起探索大鯨魚的奧妙
date: 2022-11-10 00:21:54
category: Docker
description: Docker 是什麼，為什麼大家這麼喜歡使用 Docker ？
---

## 為什麼要用 Docker 

不曉得大家是不是常遇到要開啟一個專案，卻因為裝置不相容，在安裝的過程中要一直 debug 的狀況，有時候甚至架不起來？

Docker 就是為了解決這個問題而出現的

## 什麼是 Docker

Docker 是一種架設出虛擬環境的工具，我們可以透過 Docker 將應用程式單獨切分出來，並且用容器包裝成一個虛擬環境，並且載入你本機沒有但專案會需要的工具或版本，讓應用程式在虛擬環境中開發、執行

## 學 Docker 必須要知道的幾個基本名詞

* container：容器
    映像檔的具體化，通常一個專案會用一個以上的 container 
(可能 1 個專案開 1 個 container 或者 
資料庫 1 個 container 專案 1 個 container)
    Docker 會去開啟主機並監聽訊息，並把這個訊息傳給 container
    
* image：映像檔
    唯獨，由 layer 建構而成
    映像檔可能是一個資料庫、一個框架

## Docker 運作流程

1. 我們會把環境設定寫在 Dockerfile 
2. Dockerfile 會去找本地有沒有 image
   沒有的話會從 Docker hub 中下載下來
   Docker hub 預設為 registry

3. 用 build 的方式建立 image
4. 將 image 實體化跑 container


![](https://i.imgur.com/8Yk0nzw.jpg)

### Docker Hub

跟 GitHub 概念差不多，我們可以將我們寫的專案上傳上去，讓其他人下載

### Docker Registry

以 Ruby 來說，就像是 RubyGems
以 JavaScript 來說，就像是 npm Registry 
在 Docker Registry 有各種不同的 image 
Docker 在本地找不到 image 的時候會往這邊找並且下載
另外可以設定成非公開，有權限的人才能下載
