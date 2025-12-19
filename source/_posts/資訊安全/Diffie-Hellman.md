---
title: Diffie–Hellman (DH) Key Exchange (金鑰交換)
categories:
  - 資訊安全
  - Cryptography
tags:
  - Cryptography
  - 資訊安全
abbrlink: 862ca019
mathjax: true
date: 2025-12-18 20:00:00
---
>  Diffie–Hellman 是一個金鑰交換的演算法
---
# DH 在解決什麼問題？
- 在不安全通道上，Alice 與 Bob 協議出共同秘密金鑰 $K$，用於對稱加密（如 AES）。

# 公開參數
- 質數 $p$
- 生成元 $g$
所有運算皆在 $\bmod p$ 下進行。

# 協議流程
Alice 選私鑰 $a$，公開 $A=g^a \bmod p$。  
Bob 選私鑰 $b$，公開 $B=g^b \bmod p$。  
共享秘密：
- Alice：$K=B^a \bmod p$
- Bob：$K=A^b \bmod p$
正確性：$(g^b)^a \equiv (g^a)^b \equiv g^{ab}\pmod p$。
>注意 : 不是 $(g^a) \times cross (g^b) = g^{a+b}$

# 手算例
選 $p=23, g=5$。  
Alice：$a=6 \Rightarrow A=5^6 \bmod 23=8$。  
Bob：$b=15 \Rightarrow B=5^{15} \bmod 23=19$。  
共享：
- $K=19^6 \bmod 23=2$
- $K=8^{15} \bmod 23=2$

# 安全性直覺
Eve 看到 $p,g,A,B$，想求 $K=g^{ab}$ 通常需先解離散對數 $a=\log_g(A)$ 或 $b=\log_g(B)$，在參數夠大時很困難。

# 重要弱點：MITM(Man-in-the-middle)
DH 本身不做身分認證，若被中間人改寫公開值，Alice/Bob 會各自與 Eve 建立不同的 key。
解法：用簽章、憑證、PSK 等方式對 DH 公開值做認證。

# 延伸
- DHE/ECDHE：每次連線使用臨時私鑰，提供 Forward Secrecy
- ECDH：用橢圓曲線點乘取代指數運算
- 實務會用 KDF 將共享秘密導出 session key

