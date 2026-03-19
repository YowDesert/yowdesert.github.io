---
title: (線性代數) 判斷是否可以對角化
categories:
  - 線性代數
  - 對角化及其應用
tags:
  - 線性代數
  - 對角化及其應用
abbrlink: 1460649e
mathjax: true
date: 2026-03-19 23:08:00
---
# 矩陣是否可以對角化的判斷

對 \(n \times n\) 矩陣 \(A\) 來說，

$$
A \text{ 可對角化 } \iff A \text{ 有 } n \text{ 個線性獨立的特徵向量}
$$

也就是說，若可以找到可逆矩陣 \(P\) 與對角矩陣 \(D\)，使得

$$
A = PDP^{-1}
$$

則 \(A\) 可對角化。

---

## 常用判斷方法

### 方法 1：有 \(n\) 個互異特徵值

如果 \(A\) 有 \(n\) 個互不相同的特徵值，  
那麼 \(A\) 一定可對角化。

---

### 方法 2：比較代數重數與幾何重數

對每個特徵值 \(\lambda\)：

- **代數重數 (algebraic multiplicity)**：  
  \(\lambda\) 在 characteristic polynomial 中出現的次數

- **幾何重數 (geometric multiplicity)**：  
  對應 eigenspace 的維度
  $$
  \dim \ker(A-\lambda I)
  $$

判斷條件為：

$$
A \text{ 可對角化 } \iff \text{對每個特徵值 } \lambda,\ 
\text{幾何重數}=\text{代數重數}
$$

而且所有 eigenspace 維度加總必須等於 \(n\)。

---

## 標準解題流程

### Step 1：求特徵值

解 characteristic equation：

$$
\det(A-\lambda I)=0
$$

得到所有特徵值。

---

### Step 2：看每個特徵值的代數重數

由 characteristic polynomial 判斷每個特徵值重複幾次。

---

### Step 3：求每個特徵值的幾何重數

對每個特徵值 \(\lambda\)，解

$$
(A-\lambda I)x=0
$$

求出 eigenspace 的維度，也就是

$$
\dim \ker(A-\lambda I)
$$

---

### Step 4：比較

- 若對每個特徵值都有  
  $$
  \text{幾何重數}=\text{代數重數}
  $$
  則 \(A\) 可對角化。

- 只要有某個特徵值滿足  
  $$
  \text{幾何重數}<\text{代數重數}
  $$
  則 \(A\) 不可對角化。

---

## 常見結論

### 一定可對角化

- 有 \(n\) 個不同特徵值
- 每個特徵值的幾何重數都等於代數重數

### 不一定可對角化

- 有重複特徵值時，不一定可對角化
- 必須再算 eigenspace 維度才能判斷

### 一定不可對角化

- 某個特徵值的幾何重數小於代數重數

---

## 例子 1：可對角化

$$
A=
\begin{pmatrix}
1 & 1 \\
0 & 2
\end{pmatrix}
$$

特徵值為 \(1,2\)，兩個不同，  
所以 \(A\) 一定可對角化。

---

## 例子 2：不可對角化

$$
A=
\begin{pmatrix}
2 & 1 \\
0 & 2
\end{pmatrix}
$$

特徵值只有 \(2\)，代數重數為 2。

計算：

$$
A-2I=
\begin{pmatrix}
0 & 1 \\
0 & 0
\end{pmatrix}
$$

解

$$
(A-2I)x=0
$$

可得只有一個自由變數，所以

$$
\dim \ker(A-2I)=1
$$

因此：

- 代數重數 = 2
- 幾何重數 = 1

所以 \(A\) **不可對角化**。

---

## 三角矩陣的判斷技巧

若 \(A\) 是三角矩陣，  
則它的特徵值就是對角線元素。

所以可以先看對角線：

- 若對角線元素全部不同  
  \(\Rightarrow\) 一定可對角化
- 若有重複  
  \(\Rightarrow\) 還要再算 eigenspace 維度

---

## 總結

$$
A \text{ 可對角化 } \iff A \text{ 有 } n \text{ 個線性獨立特徵向量}
$$

實際做題時，最重要的是：

1. 求特徵值
2. 看代數重數
3. 算幾何重數
4. 比較兩者是否相等

只要對每個特徵值都有

$$
\text{幾何重數}=\text{代數重數}
$$

就可以判斷 \(A\) 可對角化。