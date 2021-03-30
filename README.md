# OR-for-ProcurementStrategy

最佳化訂貨策略實作
+ 使用Gurobi取得最佳解
+ 設計Heuristic演算法逼近最佳解

## 公司簡介
10/10 Apothecary<br>
客林國際股份有限公司主要經營化妝品、保養品與香氛商品的進口業務，旗下商品種類多樣，於許多知名百貨公司皆有設櫃。
<br>
<br>

## 問題描述
+ 結合需求預測模組，打造訂貨決策模組
![image](resources/4.jpg)

+ 建立自動化系統最小化公司訂貨成本
由於目前公司的訂貨與存貨控制方式較仰賴於決策者的經驗，若能建立自動化的系統針對訂貨、存貨策略提供建議，便能夠讓決策者在決策時有更明確的資訊來做衡量及判斷。本專案將透過建立模型，協助公司在給定的訂購時點找出最佳訂購量，在缺貨成本與存貨成本之間取得最佳平衡，並維持公司的穩定運作。
<br>

## 決策目標
+ 目標函數<br>
  以最小化總成本 - 訂貨成本＋存貨成本（多訂商品所造成的成本）+ 缺貨成本（少訂商品所造成的成本）為目標，來決定本期的訂貨量及訂貨管道。

+ 決策變數<br>
  決定每期要分別使用快遞／空運／海運訂多少貨。
<br>

## 公司現行訂貨相關資訊
+ 每個月月初訂貨，未來 1~6 期每期皆可訂貨
+ 每次訂貨時會考慮未來 6 個月的需求，不考慮需求的隨機性。
+ 三種訂貨方式 lead time：快遞 0.5 個月、空運 1.5 個月、海運 2.5 個月。
+ 這個月到的貨下個月開始才能使用。舉例來說，1/1 使用 lead time 為 1.5 個月的空運訂的貨，預期在 2/15 送達，但這批貨從 3 月開始才能被用來銷售，同時也從 3 月開始才會產生存貨成本。
+ 運費分為以商品重量計價的單位成本，及每次使用該運送方式的固定成本。
+ 每月底若存貨＞0，會有存貨成本；反之若存貨＜0，則會有分別有 lost sales 成本和 backorder 成本 。
+ 訂貨時部分商品有 by package 的批量限制。
+ 可訂貨數量的最大值為：第1期期初存貨不能滿足未來六個月所有需求的部分。
<br>


## 數學模型

### 架構
![image](resources/5.jpg)
### Sets & Indices
+ <img src="http://chart.googleapis.com/chart?cht=tx&chl= T" style="border:none;"> 是所有期數的集合
+ <img src="http://chart.googleapis.com/chart?cht=tx&chl= I" style="border:none;"> 是所有商品的集合
+ <img src="http://chart.googleapis.com/chart?cht=tx&chl= F" style="border:none;"> 是所有運輸方式的集合
+ <img src="https://render.githubusercontent.com/render/math?math=t \in T"> 代表整個訂貨計畫中的第 t 期；<img src="https://render.githubusercontent.com/render/math?math=T = \{1,...,6\}">
+ <img src="https://render.githubusercontent.com/render/math?math=i \in I"> 代表整個訂貨計畫中的第 t 期；<img src="https://render.githubusercontent.com/render/math?math=I = \{1,...,96\}">
+ <img src="https://render.githubusercontent.com/render/math?math=f \in F"> 代表整個訂貨計畫中的第 t 期；<img src="https://render.githubusercontent.com/render/math?math=F = \{1,2,3\}">
+ <img src="http://chart.googleapis.com/chart?cht=tx&chl= F = 1 " style="border:none;"> 代表快遞，<img src="http://chart.googleapis.com/chart?cht=tx&chl= F = 2 " style="border:none;"> 代表空運，<img src="http://chart.googleapis.com/chart?cht=tx&chl= F = 3 " style="border:none;"> 代表海運
### Decision Variables
+ <img src="https://render.githubusercontent.com/render/math?math=x_{tif}">：第 t 期商品 i 利用運輸方式 f 訂貨量（單位：package）
+ <img src="https://render.githubusercontent.com/render/math?math=y_{ti}">：第 t 期商品 i 的「正」期末存貨（單位：package）；<img src="https://render.githubusercontent.com/render/math?math=\forall t = 0,...,6">
+ <img src="https://render.githubusercontent.com/render/math?math=z_{ti}">：第 t 期商品 i 的「負」期末存貨（單位：package）；<img src="https://render.githubusercontent.com/render/math?math=\forall t = 0,...,6">
+ <img src="https://render.githubusercontent.com/render/math?math=w_{tf}">：第 t 期有利用運輸方式 f 訂購任何產品的二元變數，若有，則<img src="https://render.githubusercontent.com/render/math?math=w_{tf} = 1">；若無，則<img src="https://render.githubusercontent.com/render/math?math=w_{tf} = 0">
### Parameters
+ <img src="https://render.githubusercontent.com/render/math?math=Q_{ti}">：商品 i 在第 t 期可用的在途存貨（單位：package）
+ <img src="https://render.githubusercontent.com/render/math?math=I_{0i}">：商品 i 在第 0 期的期末存貨（單位：package）
+ <img src="https://render.githubusercontent.com/render/math?math=D_{ti}">：商品 i 在第 t 期的需求（單位：package）
+ <img src="https://render.githubusercontent.com/render/math?math=M_i">：商品 i 每 package 的進貨價格
+ <img src="https://render.githubusercontent.com/render/math?math=C_i^o">：商品 i 每 package 的存貨成本
+ <img src="https://render.githubusercontent.com/render/math?math=C_i^b">：商品 i 每 package 的 backorder 缺貨成本
+ <img src="https://render.githubusercontent.com/render/math?math=C_i^l">：商品 i 每 package 的 lost sales 缺貨成本
+ <img src="https://render.githubusercontent.com/render/math?math=G_{if}">：商品 i 每 package 利用運輸方式 f 的單位成本
+ <img src="https://render.githubusercontent.com/render/math?math=K_f">：利用運輸方式 f 的固定成本
+ <img src="https://render.githubusercontent.com/render/math?math=L_f">：運輸方式 f 的lead time（單位：月）
+ <img src="https://render.githubusercontent.com/render/math?math=R_{if}">：商品 i 是否存在運輸方式 f 的二元參數。若有，則<img src="https://render.githubusercontent.com/render/math?math=R_{if} = 1">；若無，則<img src="https://render.githubusercontent.com/render/math?math=R_{if} = 0">
+ <img src="https://render.githubusercontent.com/render/math?math=\beta_i">：商品 i 的 backorder 比例
### Objective function
<img src="http://chart.googleapis.com/chart?cht=tx&chl= min" style="border:none;">&nbsp;&nbsp;<img src="https://render.githubusercontent.com/render/math?math=\sum_{t\in T}\sum_{i\in I}\sum_{f\in F} (M_i + G_{if}) x_{tif} ">&nbsp;+&nbsp;<img src="https://render.githubusercontent.com/render/math?math=\sum_{t\in T}\sum_{f\in F}  K_{f} w_{tf}">&nbsp;+&nbsp;<img src="https://render.githubusercontent.com/render/math?math=\sum_{t \in T}\sum_{i\in I} (C_{i}^o y_{ti}">&nbsp;+&nbsp;<img src="https://render.githubusercontent.com/render/math?math=\beta_i C_{i}^b z_{ti}">&nbsp;+&nbsp;<img src="https://render.githubusercontent.com/render/math?math=(1 - \beta_i)C_{i}^l z_{ti})">


### Constraints 
+ 期末存貨平衡式：
  + <img src="https://render.githubusercontent.com/render/math?math=y_{ti} - z_{ti} = y_{t-1,i}-\beta_i z_{t-1,i}">&nbsp;+&nbsp;<img src="https://render.githubusercontent.com/render/math?math=\sum_{f \in F ,\ L_f \lt t}">
<img src="https://render.githubusercontent.com/render/math?math=(x_{t-L_f,i,f}">&nbsp;+&nbsp;<img src="https://render.githubusercontent.com/render/math?math=Q_{ti} - D_{ti})">&nbsp;&nbsp;&nbsp;&nbsp;<img src="https://render.githubusercontent.com/render/math?math=\forall t \in T ,\ \forall i \in I">
+ 設定<img src="https://render.githubusercontent.com/render/math?math=y_{0,i}">和<img src="https://render.githubusercontent.com/render/math?math=z_{0,i}">的初始值：
  + <img src="https://render.githubusercontent.com/render/math?math=y_{0,i} = {\rm max} {(I_i^0, 0)}">&nbsp;&nbsp;&nbsp;<img src="https://render.githubusercontent.com/render/math?math=\forall i \in I">
  + <img src="https://render.githubusercontent.com/render/math?math=z_{0,i} = {\rm max} {(-I_i^0, 0)}">&nbsp;&nbsp;&nbsp;<img src="https://render.githubusercontent.com/render/math?math=\forall i \in I">
  + <img src="https://render.githubusercontent.com/render/math?math=y_{ti} \ge 0">&nbsp;&nbsp;&nbsp;<img src="https://render.githubusercontent.com/render/math?math=\forall t \in T ,\ \forall i \in I">
  + <img src="https://render.githubusercontent.com/render/math?math=z_{ti} \ge 0">&nbsp;&nbsp;&nbsp;<img src="https://render.githubusercontent.com/render/math?math=\forall t \in T ,\ \forall i \in I">
+ 下面限制式滿足以下三個條件：
  1. 連結&nbsp;<img src="https://render.githubusercontent.com/render/math?math=x_{tif}">&nbsp;和&nbsp;<img src="https://render.githubusercontent.com/render/math?math=R_{if}">：每個商品只能透過允許的運輸方式訂購
  2. 連結&nbsp;<img src="https://render.githubusercontent.com/render/math?math=x_{tif}">&nbsp;和&nbsp;<img src="https://render.githubusercontent.com/render/math?math=w_{tf}">
  3. 設定&nbsp;<img src="https://render.githubusercontent.com/render/math?math=x_{tif}">&nbsp;的最大值
    + &nbsp;<img src="https://render.githubusercontent.com/render/math?math=x_{tif} \le R_{if} w_{tf} \cdot {\rm max} \{ \sum_{t^{'}\in T} (D_{t^{'}i} - Q_{t^{'}i}) - I_i^0">&nbsp;+&nbsp;<img src="https://render.githubusercontent.com/render/math?math=1,0 \}">&nbsp;&nbsp;&nbsp;<img src="https://render.githubusercontent.com/render/math?math=\forall t \in T ,\ \forall i \in I ,\ \forall f \in F">
+ <img src="https://render.githubusercontent.com/render/math?math=x_{tif}">是非負整數：
  + <img src="https://render.githubusercontent.com/render/math?math=x_{tif} \in \mathbb{Z}">+&nbsp;&nbsp;&nbsp;<img src="https://render.githubusercontent.com/render/math?math=\forall t \in T ,\ \forall i \in I ,\ \forall f \in F">
+ <img src="https://render.githubusercontent.com/render/math?math=w_{tf}">是二元變數：
  + <img src="https://render.githubusercontent.com/render/math?math=w_{tf} \in \{0, 1\}">&nbsp;&nbsp;&nbsp;<img src="https://render.githubusercontent.com/render/math?math=\forall t \in T ,\ \forall f \in F">
<br>

## 電腦模型：架構與結果
![image](resources/18.jpg)
<br>

## Heuristic Algorithm實作（啟發式演算法）
> 每期需求都要被滿足（第一期除外）<br>
> 每期只選擇一種到貨運輸方式
<br>
1. 計算每期當下缺少的商品 package 數<br>
2. 每期選擇成本最低之運輸方式（考量各商品是否有該運輸方式）<br>
3. 從最後一期回推，決定是否提前訂貨（考量存貨成本）<br>
4. 計算訂貨量（考量各商品是否有該運輸方式）<br>


## 成果
+ 常見的方法與最佳解相差 8.3%
+ 啟發式演算法與最佳解相差 0.35%


![image](resources/19.jpg)
