---
title: AWS 小探險 - 建立 VPC 與 Public Subnet
date: 2024-07-25 18:16:22
category: AWS
---

全名為 Amazon Virtual Private Cloud，為虛擬的私有雲端，

當我們的 EC2 (機器) 有多台，並且想對這些機器做區隔的時候，就可以透過 VPC 來達成這個效果

在 VPC 裡面，會有 Subnet (子網路)，我們可以在 Subnet 底下，架設 EC2

![VPC](https://imgur.com/OubK0tQ.jpg)

這邊讓大家對於 VPC 先有一點架構概念，接著我們會邊設定邊介紹

## 事前作業

還沒有 AWS 帳號可以先去[官網](https://aws.amazon.com/free/?sc_channel=ps&all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsf.Free%20Tier%20Types=*all&awsf.Free%20Tier%20Categories=*all)辦一個

### Step 1. 開始架設 VPC

已經有帳號就可以直接進入並且搜尋 VPC

![VPC](https://imgur.com/knctqAC.jpg)

接著，點選 `Create VPC`

![VPC](https://imgur.com/rabYp0t.jpg)

接著來設定吧

`Resource to create` 請選擇 `VPC only` (但如果你要選 VPC and more 也行，~~但這樣就不用看這篇文章了~~)
`Name` 填寫 VPC 的名稱
`IPv4 CIDR block` 由於我們這次做得不需要用到那麼多 IP，只要一個小範圍就好，因此選擇 `IPv4 CIDR manual input` 就好
`IPv4 CIDR` 填寫他提供的隨機 IP
`IPv6 CIDR block` 一樣我們不需要那麼多 IP ，IPv4 就足夠，所以選 `No IPv6 CIDR block`
`Tenancy` 我們不需要專用硬體，所以選 `Default` 即可

![VPC](https://imgur.com/CczWhUY.jpg)

都填完後，就可以 Create VPC 了

![VPC](https://imgur.com/n7tgKYe.jpg)

接著我們就可以設定 Subnet 了

### Step 2. 設定 Subnet

![VPC](https://imgur.com/OubK0tQ.jpg)

Subnet 分成很多種，先介紹 Public Subnet / Private Sunbet 這兩種

他們都是為在 VPC 裡面的 (詳見架構圖)

`Public Subnet` 可以透過 Internet Gateway 來跟外網溝通
`Private Subnet` 沒辦法透過 Internet Gateway 來跟外網溝通，只能透過 NAT Gateway，且外網沒辦法無法跟 Private Subnet 溝通

我們先到左邊選單選取 Subnets

![Subnet](https://imgur.com/yDuh3j0.jpg)

再來點選 `Create subnet`

![Subnet](https://imgur.com/bGtKTqT.jpg)

點選我們剛剛建立的 `VPC ID`

![Subnet](https://imgur.com/rEgie9C.jpg)

接著填寫設定

`Subnet name` 填寫你的 Subnet 名稱(會特別加上 public 讓自己知道這是 Public Subnet)
`Availability Zone` 選取離你最近的 AZ
`IPv4 subnet CIDR block` 給你的 Subnet 一個 IP 位置，但必須要在 VPC 的 IP 範圍內

![Subnet](https://imgur.com/5RnOTU4.jpg)

都填寫完就可以 `Create subnet` 囉

![Subnet](https://imgur.com/rbafwzY.jpg)

Subnet 建立完成後，我們就要來建立 Internet Gateway

### Step 3. 設定 Internet Gateway

![Internet Gateway](https://imgur.com/d5PeorZ.jpg)

Internet Gateway 就像對外溝通的橋樑，EC2 必須透過 Internet Gateway 才能跟外網做聯繫

而 Internet Gateway 是設定在 VPC 上的 (詳見架構圖)

那我們就來開始吧！

先在左邊選單點選 Internet gateways

![Internet Gateway](https://imgur.com/gPUuMXk.jpg)

接著點選 `Create internet gateway`

![Internet Gateway](https://imgur.com/jH4MOfa.jpg)

填寫 Internet Gateway 的名稱

![Internet Gateway](https://imgur.com/vj9FOlX.jpg)

再點選 `Create internet gateway`

![Internet Gateway](https://imgur.com/gooV7Rs.jpg)

這樣就建立成功囉！

不過我們還要幫他掛在 VPC

先到右邊 `Actions` 點選 `Attach to VPC`

![Internet Gateway](https://imgur.com/05SWK9e.jpg)

接著選取要掛上去的 VPC

![Internet Gateway](https://imgur.com/NDDjmc2.jpg)

最後點選 `Attach internet gateway`

Internet Gateway 就設定好了

不過要讓 EC2 有辦法跟外網溝通，我們還要在 Subnet 的 Route Table 加上 Internet Gateway

### Step 4. 設定 Route Table

![Route Table](https://imgur.com/odINb6T.jpg)

每個 Subnet 會有一個 Route Table 

Route Table 中會記錄有哪些 Route 

我們先點選到 Subnet 

![Route Table](https://imgur.com/yDuh3j0.jpg)

選取你的 Subnet 後，點選到 Route Table

![Route Table](https://imgur.com/Mc3ZnQv.jpg)

接著再點選該連結進去

![Route Table](https://imgur.com/g0E3uTy.jpg)

進入到 Route Table 介面後，再點選連結進入

![Route Table](https://imgur.com/MdD0pfk.jpg)

點選 `Edit routes`

![Route Table](https://imgur.com/jkMOEBc.jpg)

再來點 `Add route` 後填寫設定

`Destination` 指哪些網路流量可以透過 Internet Gateway 進行溝通，而 `0.0.0.0/0` 是所有 IPv4 的網路流量
`Target` 溝通目標，也就是 Internet Gateway 了，記得選取你剛剛建立的 Internet Gateway

![Route Table](https://imgur.com/lRDaRNZ.jpg)

按下 `Save changes` 就完成囉

![Route Table](https://imgur.com/RJxofkE.jpg)

到這邊我們的架構都設定的差不多囉，接著可以來設定機器 (EC2) 了

### Step 4. 建立 EC2

![EC2](https://imgur.com/2tzVZLV.jpg)

每個 Subnet 下都可以放置多台 EC2

在 EC2 中，我們會將應用程式或專案跑起來

可以把 EC2 想像成一台電腦，我們要在這台電腦跑起專案或應用程式

開始來建立吧！點選 `EC2`

![EC2](https://imgur.com/1Iitjpz.jpg)

接著點選 `Launch instance`

![EC2](https://imgur.com/Xf7hsBE.jpg)

填寫 instance 的名字

![EC2](https://imgur.com/c4EB0D2.jpg)

選取該機器要用的系統 (通常我都選 Amazon Linux ，並且使用免費方案可用的 image)

![EC2](https://imgur.com/AYVlp8B.jpg)

Instance type 選取專案適用的規格(專案小就可以選免費方案可用的)

![EC2](https://imgur.com/dl49XeD.jpg)

建立金鑰，並下載 (登入 ssh 的時候驗證用)

![EC2](https://imgur.com/bHQYaEe.jpg)

機器的設定就差不多到這邊，接著下面有 Network Settings 區塊

我們就要進入設定 Network 與 Security Group 囉


### Step 5. 設定 Network 與 Security Group

![EC2](https://imgur.com/AZmZ9nt.jpg)

Network 是綁在 Subnet 上的，而 Security Group 是管控 Network 網際網路協議存取權限的 (不是管 EC2 的喔！！！)

EC2 透過該 Network 才會有 IP 並且跟外網溝通

我們選取剛剛建立的 VPC / Subnet

並且給這個 EC2 一個公開的 IP ，好方便跟外網做聯繫

![EC2](https://imgur.com/e8KAXNj.jpg)

當 Network 設定完成後，我們要幫這個 Network 設定 `Security Group`

來管理網際網路協議的存取權限

因為之前沒有建立過 Security Group 

所以我們這次就建立一個新的，並給他一個名稱及描述

接著我們要來看 Inbound Security Group Rule

所謂的 Inbound 指的是外部要連進來的流量

OutBound 指的是內部要出去的流量

![EC2](https://imgur.com/D4kGVhl.jpg)

預設會帶 ssh ，這也是為什麼我們可以透過 ssh 可以進入到機器中

我們今天只有要試試是否能 ping 到 EC2 的 IP，所以只要設定 All ICMP - IPv4

如果是要讓其他使用者存取到該應用程式，或者使用該網頁，就需要設定 TCP 這個網際網路協議

因為我們要讓外部所有地方都可以 ping 到，所以 Source Type 為 Anywhere

Source 就會自動帶 0.0.0.0/0 囉

![EC2](https://imgur.com/W2q2c71.jpg)

如果沒有額外要設定的，就可以到右方的 Launch Instance 建立 EC2 了

到這邊基本上就完成了 VPC & Public Subnet & EC2 的建立囉

我們來看一下架構圖，就可以接著去試試 ping IP 了

![EC2](https://imgur.com/AZmZ9nt.jpg)

### Step 6. 進入機器 & 外部 ping 機器 IP

先到 EC2 並且點選 `connect`

![EC2](https://imgur.com/bW4X3Ca.jpg)

我們來透過本地的終端機進入機器(不過要直接從 EC2 console 進去也可以唷)，點選 `SSH client`

接著在終端機先到剛剛下載金鑰的地方，下指令

`chmod 400 "0722.pem"`

改變該 pem 的權限為只有所有者能讀

接著輸入指令進入 EC2，並且同意繼續連線

`ssh -i "0722.pem" ec2-user@54.180.247.118`

![EC2](https://imgur.com/UvsVMuI.jpg)

接著就進入到機器中了，我們可以在機器中去 ping 外部的網路，以 google IP 為例

當你收到 xxx bytes from 8.8.8.8 ，並且有回應時間時，就代表成功囉

![EC2](https://imgur.com/mM3VjvK.jpg)

接著我們可以離開機器，在一般的終端機中 ping 機器的 ip

機器的 ip 哪裡看，在剛剛 `SSH client`

![EC2](https://imgur.com/4L1j1p1.jpg)

接著輸入 `ping 54.180.247.118` ，有回應時間就可以囉

![EC2](https://imgur.com/N420Mfx.jpg)

在 AWS 架設服務是會被收錢的唷，

分別要

1. 關掉 EC2 
2. 刪除 VPC (刪除後需要檢查一下 VPC / Subnet / Internet Gateway / Route Table / Network ACLs ) 
3. 釋放 IP address  (在 EC2 左邊選單的 Elastic IP addresses，點進去把 IP release 即可)

上面步驟可以參照官方文件去做

以上若有任何疑問或指教也歡迎留言！有幫助的話也歡迎給個讚，謝謝！
