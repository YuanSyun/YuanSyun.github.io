---
layout: post
title:  "具SLAM功能之掃地機器人最佳路徑規劃"
description: ""
date:       2018-03-14 09:00:00
author:     "Yuan-Syun Ye"
header-img: assets/images/robot-path.png
---

傳統的掃地機器人不具有環境偵測的能力，通常會採用撞到物件後隨機轉轉向的方式，然而這種方法並不是有效的路徑規劃。因此這項研究則是提出掃地機器人搭配光達感測技術的方法，讓光達建立環境地圖並依據地圖規劃出路徑，盡可能縮短掃地時間以及提高清潔範圍。

# 掃地機器人與光達 #
這項研究使用的掃地機器人為可程式控制的iRobot Create2以及RP Lidar A1光達感測器，同時我們也使用樹梅派控制掃地機器人以及取得光達感測器建立的環境地圖。
![hardware](/assets/images/robot-rplidar.jpg)

# 掃地機器人控制以及使用SLAM技術建立地圖 #
ROS（Robot Operating System）是一個開源的控制環境，ROS中每個節點（Node）為一個功能，可透過訂閱話題（Topic）取得該節點的輸入輸出資料，藉此可在一個環境中設計多種功能的節點，藉此實現機器人的各種功能。此作品則是針對控制掃地機器人撰寫一個控制節點、讀取光達感測器節點、以及使用SLAM（Simultaneous localization and mapping）技術依照光達資料建立環境地圖的節點。
![path](/assets/images/robot-path.png)

# Client-Server架構分散運算量 #
由於樹梅派的運算能力無法同時執行SLAM功能以及路徑規劃，因此採用Client-Server的方式將路徑規劃的功能轉移到伺服器端，降低樹梅派的負荷，同時在ROS環境中設計一個負責Client-Server溝通的節點，傳送環境地圖以及接收掃地路徑的指令。

# 「深度優先探索分群區塊」路徑規劃 #
當伺服器接收到樹梅派的環境地圖後，就會開始進行路徑規劃。地圖包含了障礙物、無法判斷的區域、以及空曠的區域三種，這裡採用洪水演算法（Flooding Algorithm）保留空曠的區域。接著為了降低路徑規劃的複雜度，我們使用四叉樹的方式（Quad Tree）將地圖分群，並透過二維位置的關係建立無向圖，該區域的面積大小為其權重值。接著使用普林演算法（Prim’s Algorithm）建立最小生成樹，並使用深度優先搜尋演算法（Depth-First Search Algorithm）探索樹上的每一個節點的掃地區域，最後在每一個掃地區域內使用Z自字型的掃地方式。
![process](/assets/images/robot-process.png)