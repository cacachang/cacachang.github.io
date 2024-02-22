---
title: Docker - 一起探索大鯨魚的奧妙
date: 2024-02-22 23:00:54
tags:
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
    唯讀，由 layer 建構而成

* volume：
    在容器外運行，可連接不同的 container

* daemon：
    管理 Docker image、開啟跟關閉 container
    
* Docker Hub：
    類似 GitHub 的概念，是 image 的倉庫，可從上面拉取所需要的 image (權限可以設定為私有，只限定有權限的人才可以拉取)

## 更多 Docker 的專有名詞

在介紹 Docker 運作方式之前，我們需要了解更多專有名詞：

* Docker Engine：
    Docker 的核心，負責管理 image。
    包括 Docker Daemon、Docker API 和CLI。
    在建立 image 時，如果本地沒有相同 image，會從 Docker Hub 拉取。

* Docker API：

    使用 RESTful 標準向 Docker Daemon發送請求。

* Docker CLI：

    我們可以在這個介面下指令，並通過 Docker API 向 Docker Engine 發出指令。

## Docker 運作流程

我們可以將 Docker 想像成電腦中的一個盒子，在這個盒子內部有一個容器化的環境。

Docker 使用電腦的 CPU、記憶體和操作系統來運行。

為了更清楚地理解這個概念，請參考下面的圖表：
![](https://i.imgur.com/JMyuMf2.jpg)

有這個基礎概念後，我們就來看一下當建立 image 時， Docker 是怎麼運作的

![](https://imgur.com/RxbBMNF.jpg)


當我們下指令 `docker build -t xxxx .` 時，運作流程會是這樣：

1. 在 CLI 中輸入指令。
2. CLI 發送給 Docker API。
3. Docker API 以 RESTful 標準發送請求給 Docker daemon 去尋找 image。
4. 如果本地找不到 image，會從 Docker Hub 下載 image。
5. 建立 image。

如果沒有錯誤或缺少套件，容器就可以成功建立。

image 建立起來後，還需要下指令才能讓他建立及運作 container
這時候會使用電腦中的 CPU、記憶體以及作業系統

6. 在 CLI 輸入 `docker container run xxxx .`
7. container 開始運作

如果要停止 container

8. 在 CLI 輸入 `docker container stop xxxx`

這時候 container 就會停止，而原本佔用電腦中的資源都會被釋放。

現在我們對 Docker 的基本概念有深度理解，下一篇文章將會比較著重在實際練習！

## English Version

## Why use Docker?

In certain situations, starting up a web server failed due to device incompatibility.

There are multiple steps involved in resolving the issue, and it can be challenging for developers to overcome these obstacles.

Integrating Docker into the project can avoid these awkward situations.

Docker simplifies the development and maintenance process for developers.

## What is Docker

Docker creates a virtual environment that contains services or applications. 
This environment is separated from your computer and other containers built from the Docker images.

Based on the above explanation, an application can be started up and developed within the virtual environment, which provides the necessary dependencies version and tool.

## Basic terminology docker beginners have to learn

* container:
    Created from an image, it represents an instance of that image.
    A project typically usually consists of multiple containers.
    (Probably one project only has one container, it's depending on the number of Dockerfiles)

* image
    Immutable and consists of layers
    
* Volume
    Exists outside the container, and allows for connection between different containers.

* daemon
    Manage Docker images, starts and stops containers.
    
* Docker Hub
    Similar to GitHub, is a platform to used to store images that anyone can pull image here.
    

### Additional Docker terminology you should know

Before we mention how docker works, it's essential to understand a few more terms:

* Docker Engine:
    The Core of Docker management, it's responsible for managing images.
    It includes Docker daemon, Docker API, and CLI.
    It will pull down images from Docker Hub if the same image is not found locally when we build an image.
    
* Docker API
    Sends requests to Docker daemon using Restful standards.
    
* Docker CLI
    An interface we can command to Docker Engine via Docker API.
    
    
## How does Docker operate?

We can image docker as a box within computers, there is a containerized environment inside the box, which has its hostname, IP address, and disk for data storage.
Docker uses the CPU, memory, and operating system resources of the device to run containers.

To clarify the concept, refer to the picture below.

![](https://imgur.com/JMyuMf2.jpg)

    
So far, we have understood the fundamental concept of Docker, the important point of concept we will introduce in the chapter:

![](https://imgur.com/RxbBMNF.jpg)

When we run `docker build -t xxxx .` in the beginning, Docker process follows these steps:

1. We enter a command in the CLI.
2. The CLI sends a message to Docker API.
3. Docker API requests Docker Daemon to find the image by Restful standards.
4. Pull the image from Docker Hub if the same image is not available locally.
5. A image is built.

Once these steps are completed without errors or missing dependencies, the container is successfully created.

The steps mentioned above only build a image, but it's not building and running container. To do these, follow these steps:

6. Enter `docker container run xxxx`
7. Container will run successfully without errors.

Docker accesses CPU, memory, and operating system resources from the computer when a container is running.

If we want to stop the container:

8. Enter `docker container stop xxxx`

The resources container used will be released when stopping the container.

Now We have a solid understanding of the basic Docker concepts,  the next article will focus on practical exercises, see u soon.
