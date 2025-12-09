---
title: "[PicoCTF] EVEN RSA CAN BE BROKEN??? 解題筆記"
date: 2025-08-11T11:15:00.000Z
categories:
  - PicoCTF
  - 資訊安全
  - Cryptography
  - Easy
tags:
  - Easy
  - Cryptography
  - PicoCTF
hidden: true
---
[PicoCTF] EVEN RSA CAN BE BROKEN???
Category:
Cryptography
Difficulty:
Easy
Description:
This service provides you an encrypted flag. Can you decrypt it with just N & e?
Connect to the program with netcat:

```text
1
```

```text
nc verbal-sleep.picoctf.net 51510
```

The program’s source code can be downloaded here.
題目筆記
題目提供了一個 RSA 加密的環境，只給了
N
、
e
與加密後的密文。
一般 RSA 需要將 N 分解為兩個質數 p 和 q，才可以計算私鑰並解密。
題目的關鍵：
N 是偶數
這在正常 RSA 中是不可能的（因為 p 與 q 都是奇質數），因此可以推測
其中一個質因數是 2
。
解題步驟
1. 連線取得題目數據
使用題目提供的
nc
指令：

```text
1
```

```text
$ nc verbal-sleep.picoctf.net 51510
```

會得到3組數字：
點我展開 N, e, cyphertext

```text
1
2
3
```

```text
N: 23699934164118883579301772868468610522855398699101470432464301706604093983993724690398866051915307099172198837189221549627296870046186474519602638094970754
e: 65537
cyphertext: 11596761688120110826821498998827014653094511592088849547225306574877488206530053305458943666206403492895885743847965127326690553567226913671355611361014981
```

2.
RSA 的基本概念，可以參考我有關RSA的文章
名稱
說明
公開金鑰
$(e, N)$
私密金鑰
$(d, N)$
N 的組成
$N = p \times q$，其中 p、q 為大質數
加密公式
$c \equiv m^{e} \pmod{N}$
解密公式
$m \equiv c^{d} \pmod{N}$
歐拉函數 φ(N)
$\varphi(N) = (p - 1)(q - 1)$
私鑰 d
$d \equiv e^{-1} \pmod{\varphi(N)}$
3.開始計算 - 分解N
從RSA的數學概念裡可以知道 N = p * q
而 p q 都是
質數
但 題目給的 N 是
偶數
所以可以得知 p 或 q 有一個是
2
(2是質數裡唯一的偶數)
而另一個就為是
N / 2
得知 p與q 就可以解密了！
4. 計算歐拉函數φ(N)
RSA 中：
$φ(N) = (p-1)(q-1)$
因為 p = 2 :
$ φ(N) = (2−1)(q−1)=q−1$
5.計算 私鑰
d
RSA中，
d
的公式:
$d \equiv e^{-1} \pmod{\varphi(N)}$
6. 解密密文
RSA中，解密公式:
$m \equiv c^{d} \pmod{N}$
我們可以使用
Python
幫助我們解碼

```text
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
```

```text
from
 Crypto.Util.number 
import
 long_to_bytes, inverse
N = 
23699934164118883579301772868468610522855398699101470432464301706604093983993724690398866051915307099172198837189221549627296870046186474519602638094970754
e = 
65537
c = 
11596761688120110826821498998827014653094511592088849547225306574877488206530053305458943666206403492895885743847965127326690553567226913671355611361014981
p = 
2
q = N // p
phi = (p - 
1
) * (q - 
1
)
d = inverse(e, phi)
m = 
pow
(c, d, N)
flag = long_to_bytes(m).decode()
print
(flag)
```

成功獲取 Flag
恭喜你拿到 Flag 了！
補充說明
如果 N 是偶數，RSA 幾乎可以瞬間被破解，因為其中一個質數是 2 ， 那就可以知道另一個數字是多少。
常規 RSA 中，p 和 q 都大多是大質數且為奇數。
Python 的 Crypto.Util.number 提供了 inverse 與 long_to_bytes，方便進行 RSA 解密。
