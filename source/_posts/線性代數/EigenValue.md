---
title: (線性代數) EigenValue and EigenVector計算
categories:
  - 線性代數
  - 對角化及其應用
tags:
  - 線性代數
  - 對角化及其應用
abbrlink: 52554cd4
date: 2026-03-11 20:08:00
mathjax: true
---

# 求EigenValue 方法

## 一般法
$$
Av = λv (v != 0)
=> (A - λI) v = 0
=> det(A-λI) = 0
$$
又叫做 **characteristic equation**

## 技巧

### 對角矩陣 : 直接看對角線
如果

$$
A=
\begin{pmatrix}
a_1&0&0\\\\
0&a_2&0\\\\
0&0&a_3
\end{pmatrix}
$$

那 eigenvalue 直接就是對角線：

$$
a_1,a_2,a_3
$$

### 三角矩陣
如果 `A` 是上三角或下三角矩陣，eigenvalue 也是直接看對角線。

### 每列 or 每行 加起來 = c

如果 每行加起來皆等於c ， 則有一個EigenValue為c。
 
- 因為 𝐴 和 $𝐴^𝑇$ 特徵值相同

$$
A=
\begin{pmatrix}
1&2&3\\\\
2&2&2\\\\
0&3&3
\end{pmatrix}
$$

每列加起來都是 `6`，則有一個EigenValue為`6`

###  $2\times 2$ 矩陣公式
若

$$
A=
\begin{pmatrix}
a&b\\\\
c&d
\end{pmatrix}
$$

則 characteristic polynomial 是

$$
\lambda^2-\operatorname{tr}(A)\lambda+\det(A)=0
$$

其中

$$
\operatorname{tr}(A)=a+d
$$

$$
\det(A)=ad-bc
$$

所以：

- eigenvalue 的和 = $ \operatorname{tr}(A) $
- eigenvalue 的積 = $ \det(A) $

## 特殊矩陣的技巧

### 冪等矩陣
若

$$
A^2=A
$$

則 eigenvalue 只能是

$$
0 \text{ 或 } 1
$$

因為若 $ Av=\lambda v $，則

$$
A^2v=\lambda^2v
$$

又因為 $ A^2=A $，所以

$$
\lambda^2=\lambda
$$

$$
\lambda(\lambda-1)=0
$$

因此

$$
\lambda=0 \quad \text{或} \quad \lambda=1
$$

### 冪零矩陣

- EigenValue 只能是 `0`

- 若 $Av=\lambda v$，其中 $v\neq 0$，  
則再作用 $k$ 次可得

$$
A^k v=\lambda^k v
$$

但因為 $A$ 是冪零矩陣，所以存在某個 $k$ 使得

$$
A^k=0
$$

因此

$$
A^k v=0
$$

所以

$$
\lambda^k v=0
$$

因為 $v\neq 0$，故必有

$$
\lambda^k=0
$$

所以

$$
\lambda=0
$$

因此冪零矩陣的所有特徵值都只能是 $0$。

### 對合矩陣
若

$$
A^2=I
$$

則 eigenvalue 只能是

$$
1 \text{ 或 } -1
$$

因為

$$
\lambda^2=1
$$



### Nilpotent 矩陣
若

$$
A^k=0
$$

則 eigenvalue 只能是

$$
0
$$

## 冪等矩陣的重要空間

對冪等矩陣 $ A $：

### $V(0)$
$$
V(0)=\ker(A)
$$

因為特徵值 $0$ 的 eigenspace 定義為

$$
V(0)=\ker(A-0I)=\ker(A)
$$



### $V(1)$
$$
V(1)=\ker(A-I)
$$

而且對冪等矩陣還有

$$
V(1)=CS(A)
$$

其中 $CS(A)$ 是 column space。



## 為什麼 $V(1)=CS(A)$

因為

$$
V(1)=\ker(A-I)
$$

只要證明

$$
\ker(A-I)=CS(A)
$$

### 證 $CS(A)\subseteq \ker(A-I)$

若 $ y\in CS(A) $，則 $ y=Ax $ 某個 $x$。

所以

$$
(A-I)y=(A-I)Ax=A^2x-Ax=Ax-Ax=0
$$

因此

$$
y\in \ker(A-I)
$$



### 證 $\ker(A-I)\subseteq CS(A)$

若 $ y\in \ker(A-I) $，則

$$
(A-I)y=0
$$

所以

$$
Ay=y
$$

因此

$$
y=Ay
$$

這表示 $ y $ 是 $A$ 的某個像，所以 $ y\in CS(A) $。



# 求EigenVector 方法

已知 eigenvalue $\lambda$ 之後，要找對應的 eigenvector，就是解

$$
(A-\lambda I)v=0
$$

只要找 **非零解** 即可。

## 一般步驟

### 第一步：先求 eigenvalue

先解

$$
\det(A-\lambda I)=0
$$

得到所有 eigenvalue。

### 第二步：代回去求 eigenvector

對每一個 eigenvalue $\lambda$，去解

$$
(A-\lambda I)v=0
$$

解出來的所有非零向量，就是對應的 eigenvector。

## 例子

設

$$
A=
\begin{pmatrix}
2&1\\\\
1&2
\end{pmatrix}
$$

### 先求 eigenvalue

$$
A-\lambda I=
\begin{pmatrix}
2-\lambda&1\\\\
1&2-\lambda
\end{pmatrix}
$$

所以

$$
\det(A-\lambda I)=(2-\lambda)^2-1
$$

$$
=4-4\lambda+\lambda^2-1
$$

$$
=\lambda^2-4\lambda+3
$$

令它等於 $0$

$$
\lambda^2-4\lambda+3=0
$$

$$
(\lambda-1)(\lambda-3)=0
$$

所以 eigenvalue 為

$$
\lambda=1,3
$$

### 當 $\lambda=3$ 時

$$
A-3I=
\begin{pmatrix}
-1&1\\\\
1&-1
\end{pmatrix}
$$

解

$$
(A-3I)v=0
$$

也就是

$$
\begin{pmatrix}
-1&1\\\\
1&-1
\end{pmatrix}\begin{pmatrix}
x\\\\
y
\end{pmatrix}=\begin{pmatrix}
0\\\\
0
\end{pmatrix}
$$

得到

$$
-x+y=0
$$

所以

$$
y=x
$$

因此 eigenvector 為

$$
\begin{pmatrix}
x\\\\
x
\end{pmatrix}
=x
\begin{pmatrix}
1\\\\
1
\end{pmatrix}
\quad (x\neq 0)
$$

所以對應 $\lambda=3$ 的 eigenvector 可寫成

$$
c
\begin{pmatrix}
1\\\\
1
\end{pmatrix}
\quad (c\neq 0)
$$

### 當 $\lambda=1$ 時

$$
A-I=
\begin{pmatrix}
1&1\\\\
1&1
\end{pmatrix}
$$

解

$$
(A-I)v=0
$$

也就是

$$
\begin{pmatrix}
1&1\\\\
1&1
\end{pmatrix}\begin{pmatrix}
x\\\\
y
\end{pmatrix}=\begin{pmatrix}
0\\\\
0
\end{pmatrix}
$$

得到

$$
x+y=0
$$

所以

$$
y=-x
$$

因此 eigenvector 為

$$
\begin{pmatrix}
x\\\\
-x
\end{pmatrix}
=x
\begin{pmatrix}
1\\\\
-1
\end{pmatrix}
\quad (x\neq 0)
$$

所以對應 $\lambda=1$ 的 eigenvector 可寫成

$$
c
\begin{pmatrix}
1\\\\
-1
\end{pmatrix}
\quad (c\neq 0)
$$

## Eigenspace

對應 eigenvalue $\lambda$ 的 eigenspace 定義為

$$
V(\lambda)=\ker(A-\lambda I)
$$

也就是所有 eigenvector 再加上零向量形成的空間。

例如上面的例子：

### 對 $\lambda=3$

$$
V(3)=\operatorname{span}
\left\\{
\begin{pmatrix}
1\\\\
1
\end{pmatrix}
\right\\}
$$

### 對 $\lambda=1$

$$
V(1)=\operatorname{span}
\left\\{
\begin{pmatrix}
1\\\\
-1
\end{pmatrix}
\right\\}
$$

## 冪等矩陣的 EigenVector

若

$$
A^2=A
$$

則 eigenvalue 只能是 $0$ 或 $1$。

### 當 $\lambda=0$

解

$$
Av=0
$$

所以

$$
V(0)=\ker(A)
$$

### 當 $\lambda=1$

解

$$
(A-I)v=0
$$

所以

$$
V(1)=\ker(A-I)
$$

而且對冪等矩陣還有

$$
V(1)=CS(A)
$$

---

# 整理

## 求 EigenValue

解

$$
\det(A-\lambda I)=0
$$

## 求 EigenVector

對每個 $\lambda$ 解

$$
(A-\lambda I)v=0
$$

## 求 Eigenspace

$$
V(\lambda)=\ker(A-\lambda I)
$$