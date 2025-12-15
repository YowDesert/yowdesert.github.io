---
title: (資料庫系統) Normal Forms (NF)
categories:
  - 資料庫系統
  - Normal Forms
tags:
  - 資料庫系統
abbrlink: b4af80a7
mathjax: true
---

> 本章我要統整一下 Normal Forms 究竟是甚麼東西
> 核心目的 :　消除資料重複，避免更新異常，同時不產生假值組（spurious tuple）

---

# 快速導覽一下

- 基本正規式(Normal Forms Based on Primary Keys)
    - 1NF (First Normal Form)
    - 2NF (Second Normal Form)
    - 3NF (Third Normal Form)
> Tips : 實作通常可以只到3NF就好

- 高等正規式
    - BCNF (Boyce-Codd Normal Form)
    - 4NF (Forth Normal Form)
    - 5NF (Join Dependencies and Fifth Normal Form)

---
# 1NF

- 每個屬性是**簡單**且**單值**
- 只要合法的關聯綱目，也就是只要屬性值為 atomic value，都滿足1NF

## 那如果有多值呢 ? 
### Solution 1 : 將多值的那個屬性也設為主鍵並將它分開好幾列
- Ex. 原圖

|   tNo | transMid | method | transTime           | products                 |
| ----: | -------- | ------ | ------------------- | ------------------------ |
| 91100 | a0911234 | cart   | 2005-02-02 18:30:00 | {b30999}                 |
| 92666 | c0927777 | cart   | 2005-10-10 22:10:30 | {d11222, d20777, v00111} |
| 92333 | c0927777 | email  | 2005-10-15 09:00:00 | {b51111}                 |
| 91888 | a0910001 | fax    | 2005-09-10 10:10:00 | {b40555, d03333}         |
| 90111 | b0905555 | cart   | 2005-05-05 12:30:30 | {v01888}                 |
| 92555 | b0922468 | cart   | 2005-11-11 09:10:00 | {b10234, b40555}         |

- Fix: 修改後

|   tNo | transMid | method | transTime           | product |
| ----: | -------- | ------ | ------------------- | ------- |
| 91100 | a0911234 | cart   | 2005-02-02 18:30:00 | b30999  |
| 92666 | c0927777 | cart   | 2005-10-10 22:10:30 | d11222  |
| 92666 | c0927777 | cart   | 2005-10-10 22:10:30 | d20777  |
| 92666 | c0927777 | cart   | 2005-10-10 22:10:30 | v00111  |
| 92333 | c0927777 | email  | 2005-10-15 09:00:00 | b51111  |
| 91888 | a0910001 | fax    | 2005-09-10 10:10:00 | b40555  |
| 91888 | a0910001 | fax    | 2005-09-10 10:10:00 | d03333  |
| 90111 | b0905555 | cart   | 2005-05-05 12:30:30 | v01888  |
| 92555 | b0922468 | cart   | 2005-11-11 09:10:00 | b10234  |
| 92555 | b0922468 | cart   | 2005-11-11 09:10:00 | b40555  |
ㄋ
### Solution 2 : 將有多值的那個欄位 ， 連同主鍵一起再新增一個TABLE接著皆**設為主鍵**

- EX.原圖

![alt text](/img/FNF.png)

- Solution : 拆成兩張圖 ， 多值那個圖皆為主鍵

|    pNo | pName        | unitPrice |
| ----- | ------------ | -------- |
| b30999 | 資料庫理論與實務     |       500 |
| d11222 | 任賢齊專輯三       |       300 |
| b20666 | OLAP進階       |       500 |
| b10234 | 管理資訊系統概論     |       600 |
| b40555 | 系統分析理論與實務    |       550 |
| d20777 | 蔡依林專輯二       |       350 |
| v01888 | 哈利波特：混血王子的背叛 |       450 |
| d03333 | 5566專輯       |       450 |
| b51111 | 電子商務理論與實務    |       700 |
| v00111 | 英雄           |       400 |

|    pNo | name    | title |
| ----- | ------- | ----- |
| b30999 | Huang   | Prof. |
| d11222 | William | Mr.   |
| b20666 | Sandra  | Prof. |
| b10234 | Lin     | Prof. |
| b40555 | Wu      | Prof. |
| d20777 | Jolin   | Ms.   |
| v01888 | J.K.    | Mrs.  |
| d03333 | Jackey  | Mr.   |
| d03333 | David   | Mr.   |
| d03333 | Tom     | Mr.   |
| b51111 | Lai     | Dr.   |
| b51111 | Huang   | Prof. |
| b51111 | Lin     | Prof. |

# 2NF
- 滿足1NF
- 不允許部分函數相依
    -定義 : X → Y ，那如果可以找到 X 的一部分 X' ， X' → Y 成立那就是部分函數相依
    - EX. {mId,cartTime,pNo} → {pName} 
        那如果存在 {pNo} → {pName} 則部分函數相依
> （前提：主鍵為複合鍵時）

## 解決方法

- 原圖 : 

![alt text](/img/TNFF.png)

- 可以把 pNo 單獨聯的那部分拆開

![alt text](/img/TNFT.png)

# 3NF
- 滿足2NF
- 各欄位之間沒有存在「遞移相依」的關係

> 遞移相依 : 一函數 X → Y ， 如果存在 X → Z , Z → Y，且Z不為超級鍵，則稱 X → Y 為遞移相依函數

## 違反3NF的例子

- {tNo} → {name}是一個遞移函數相依
    - 因為{tNo} → {transMid} 且 {transMid} → {name} 且 transMid 不是 superkey

![alt text](/img/ThNF.png)


## 解決方法

- 把中間那個transMid分別拆開到另一張圖

![alt text](/img/ThNFT.png)

# BCNF
- 對關聯 R 的每一條函數相依 X → Y，左邊 X 必須是 superkey
- 滿足3NF

## 滿足3NF 但不滿足 BCNF

- invNo 可以決定 tNo ， 但invNo不是超級鍵

![alt text](/img/BCNFF.png)
 
 ## 解決方法

 - 把invNo 拆出來 ， 可拆成兩張表

 ### 第一張

![alt text](/img/BCNFB.png)

 ### 第二張

![alt text](/img/BCNFB-1.png)

## BCNF 小結
- 拆到 BCNF 後，常會失去 函數相依 ，需要跨表 join 才能檢查某些 FD（dependency preservation）。

# 4NF
- 源於多值相依
    - 給定一個 X 的屬性值 ， 便有一組 Y 的屬性值 : X ↠ Y
 
# 5NF
- 處理 Join Dependency
- 關聯R無法再被分解成 數個關聯
- R 可以被分解成數個關聯 $ R_i $ ， 使得 $ R_1 * R_2 * ... * R_k = R $， 但 $ R_i $ 都是 R 的super Key
> 幾乎不在實務中用

# Inclusion Dependence（包含相依）
- 考慮兩個關聯 R 與 S，以及它們的屬性集合 X 與 Y。
- R(X) 中出現的每一個值，都一定也出現在 S(Y) 中 : R(X) ⊆ S(Y)