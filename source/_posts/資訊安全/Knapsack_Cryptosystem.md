---
title: Knapsack Cryptosystem（Merkle–Hellman 背包公鑰密碼）
categories:
  - 資訊安全
  - Cryptography
tags:
  - Cryptography
  - 資訊安全
abbrlink: af1af203
date: 2025-12-18 20:00:00
mathjax: true
---

# Knapsack Cryptosystem（Merkle–Hellman 背包公鑰密碼）

> 本章介紹經典的 **Merkle–Hellman Knapsack Cryptosystem**，利用「超遞增序列」讓解密容易，但透過模運算與乘法偽裝，使攻擊者面對困難的 subset-sum 問題。

---
- 前提:
    - SIK : superincreasing knapsack
    - GK : general knapsack

---
## 1. Knapsack（背包）問題直覺

給定一組數字(weight)：

$$
b_1, b_2, \dots, b_n
$$

以及一個二進位訊息向量(是否選取?)：

$$
\mathbf{x} = (x_1, \dots, x_n), \quad x_i \in \{0,1\}
$$

加密方式為：

$$
C = \sum_{i=1}^{n} x_i b_i
$$

這稱為 **subset sum problem（子集合和問題）**，General knapsack is NPC(NP-complete)。

---

## 2. 超遞增序列（Superincreasing Sequence）

定義一個序列：

$$
a_1, a_2, \dots, a_n
$$

若滿足：

$$
a_i > \sum_{j=1}^{i-1} a_j
$$

每個重量都要大於前面所有重量的總和
則稱為 **超遞增序列**。

### 特性（為什麼好解？）

給定總和：

$$
S = \sum x_i a_i
$$

可用「Greedy algorithm」唯一解出 $x$：

1. 從最大 $a_n$ 開始
2. 若 $S \ge a_n$，則 $x_n=1$，$S \leftarrow S-a_n$
3. 否則 $x_n=0$
4. 依序往下

---

## 3. 金鑰設計（核心概念）

### 私鑰（Secret Key）

1. 超遞增序列：
   $$
   \mathbf{a} = (a_1, \dots, a_n)
   $$
2. 模數： m > 集合的總和
   $$
   m > \sum_{i=1}^{n} a_i
   $$
3. 乘數： n 要跟 m 的最大公因數 = 1
   $$
   \gcd(n, m) = 1
   $$

> ⚠️ 註：本文採用記號  
> - m 為模數（modulus）  
> - n 為乘數（multiplier）  
> 與部分教科書符號相反，但數學意義完全相同。

---

### 公鑰（Public Key）

將私鑰背包「偽裝」：將好解的SIK 偽裝成 難解的GK

$$
b_i \equiv n \cdot a_i \pmod m
$$

得到：

$$
\mathbf{b} = (b_1, \dots, b_n)
$$

---

## 4. 加密（Encryption）

給定訊息：

$$
\mathbf{x} = (x_1, \dots, x_n)
$$

計算：

$$
C = \sum_{i=1}^{n} x_i b_i
$$

並送出密文 $ C $。

---

## 5. 解密（Decryption）

### Step 1：去除偽裝

由於：

$$
C \equiv n \sum x_i a_i \pmod m
$$

乘上 $n^{-1} \pmod m$：

$$
C' \equiv n^{-1} C \pmod m
$$

因為 $m > \sum a_i$，所以：

$$
C' = \sum x_i a_i
$$


### Step 2：用超遞增性解出訊息

對 $C'$ 使用貪婪法，即可還原： $\mathbf{x}$

---

## 6. 完整手算範例

### 私鑰選擇

$$
\mathbf{a} = (2, 3, 7, 14, 30, 57)
$$

驗證：

$$
\sum a_i = 113
$$

選擇：

- \(m = 131 > 113\)
- \(n = 17\)，且 \(\gcd(17,131)=1\)

---

### 公鑰計算

$$
b_i = 17 a_i \bmod 131
$$

結果：

$$
\mathbf{b} = (34, 51, 119, 107, 117, 52)
$$

---

### 加密

訊息：

$$
\mathbf{x} = (1,0,1,1,0,1)
$$

$$
C = 34 + 119 + 107 + 52 = 312
$$

---

### 解密

#### 1️⃣ 找反元素

$$
17^{-1} \equiv 54 \pmod{131}
$$

#### 2️⃣ 去偽裝

$$
C' = 54 \cdot 312 \bmod 131 = 80
$$

#### 3️⃣ 貪婪解超遞增

$$
80 = 57 + 14 + 7 + 2
$$

得到：

$$
\mathbf{x} = (1,0,1,1,0,1)
$$

成功還原。

---

## 7. 參數選擇重點（考試必背）

- 超遞增序列：確保可解
- 模數 \(m\)：避免模繞回  
  $$
  m > \sum a_i
  $$
- 乘數 \(n\)：必須可逆  
  $$
  \gcd(n,m)=1
  $$

---

## 8. 安全性補充（歷史背景）

Merkle–Hellman 屬於 **低密度背包系統**，後來被 **LLL lattice attack** 破解，因此不再用於現代加密。

👉 但它是理解「**結構隱藏（trapdoor）**」的經典教材。

---
