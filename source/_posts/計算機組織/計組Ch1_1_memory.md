---
title: (計算機組織) 計算機組織摘要-Memory
categories:
  - 計算機組織
  - 第一章
tags:
  - 計算機組織
abbrlink: 5b29cb9e
mathjax: true
date: '2026-1-29 17:20:00'
---

> 我是根據張凡老師的講義整理的筆記 我想說可以考前看比較快!!!

# Memory Function
- OS copy Data 到 Memory Procedure
- Procedure的Code處理資料

![alt text](/img/MemoryFunction.png)

# Memory Structure
- 記憶體就像一個很大的「一維陣列」，而位址(2進制編碼)就像這個陣列的「索引」，從 0 開始。

# Memory Addressing
- 一個Address如果只放一Byte的Data，叫做**Byte Addressing**
- 一個Address如果只放一Word的Data，叫做**Word Addressing**
- Word Addressing 對CPU存取有益 一次丟4的倍數的位置 可對齊

![alt text](/img/MemoryAdd.png)

# Memory Capacity
- 假設 Address 有 `k` 個位元
  則格子數總共有 $2^k$ 個
- 假設 資料量有`8`個bit
- 則Capacity為 
$$
格子數 \times 資料量 = 2^k \times 8(bits) = 2^k \times 1(byte)
$$


![alt text](/img/MemoryCapacity.png)

## 1. Control Line（控制線）
- **Read Line / Write Line** 都是 **Control Line**
- **Read 與 Write 不可同時為 `1`**
  - 避免同一個週期同時讀寫造成衝突
  - 通常同一時間只會是：
    - `Read=1, Write=0`（讀）
    - `Read=0, Write=1`（寫）
    - `Read=0, Write=0`（不動作 / idle）

---

## 2. Address Line（位址線）
- **Address Line 有幾條 = Address 有幾個 bit**
- 若 Address Line = `k` 條  
  - 可定址位置數（Memory locations）  
    \[
    M = 2^k
    \]
- 你筆記的重點：
  - **32 條 Address Line → 32-bit address → \(2^{32}\) 個位址**
  - **這個位址寬度是系統/ISA（例如 MIPS 常見）設定的概念**，不是 ALU 之類的單元「自己決定」就能改

---

## 3. Data Line（資料線）
- **Data Line 通常跟 CPU 的資料寬度（word size / bus width）相關**
- 一次可以傳輸多少資料，取決於 Data Line 寬度
- 以 MIPS 常見情境：
  - 一次抓一個運算元（1 word）= **4 bytes**
  - 因此常見 Data Line = **32-bit**

> 注意：Data Line 表示「一次傳輸的寬度」，不一定代表「每個 address 對應多少 bytes」。

---

## 4. Memory Address（位址本身）
- **Memory Address 的 bit 數 = Address Line 的條數**
- 若 Address Line = 32 條：
  - address 範圍：`0 ~ 2^32 - 1`

---

## 5. 為什麼不能隨便代換（很重要）
- **Address Line 多少就決定格子數（locations）多少：\(2^k\)**
- 所以不能在不改前提下，把 \(2^{32}\) 隨便換成：
  - `1G × 4 bytes`
  - `2G × 2 bytes`
  - 等等
- 因為這些換法等於你偷偷改了：
  - Address Line 的條數（k），或
  - 每個位址對應的大小（byte / word addressing）

---

## 6. 一句話總結
- **Control line** 決定「讀/寫」且讀寫不可同時 1  
- **Address line** 決定「有幾個位址（格子數）」：\(2^k\)  
- **Data line** 決定「一次搬多少資料（傳輸寬度）」  
- **線數定了，格子數就定了，不能亂換**

![alt text](/img/MemoryRead.png)

## Byte/Word Addressing
- 丟一次Address開四格
- Data Line 32 bits 為 4bytes
- MMU (Memory Management Unit)
- CPU 只要換 MMU 就會換
![alt text](/img/BWAd.png)

## 例題:

### 題目一 : 
> The Sequential word addresses in machine with byte addressing differ by one
- Ans : False
- byte addressing : 
  - 每個位址對應 1 byte
  - 但 1 word = 好幾個 bytes（常見 4 bytes）
- 如果改成Word Addressing 就會成立

# Memory Read/Write operation

- Read（讀取）的流程
  - CPU 把 address=500 放到 Address Line
  - CPU 把 MemRead=1、MemWrite=0 放到控制線
  - Memory 依 address 找到 `Memory[500]`
  - Memory 把該內容放到 Data Line
  - CPU 從 Data Line 取值，寫回暫存器：`$s1 ← Memory[500]`

- Write（寫入）的流程
  - CPU 把 address=200 放到 Address Line
  - CPU 把「要寫入的資料」放到 Data Line（例如 $t1 的值是 1736）
  - CPU 把 MemRead=0、MemWrite=1 放到控制線
  - Memory 在該 address 位置寫入 Data Line 的值：`Memory[200] ← 1736`

# Memory Alignment (對齊)

## 什麼是 Alignment（對齊）？
在 **byte addressing** 的機器中，每個 address 對應 **1 byte**。  
但 CPU 常常一次要讀/寫 **word（4 bytes）** 或 **halfword（2 bytes）**，因此硬體會規定「一次抓多個 byte」時，**起始位址必須落在特定邊界**，這就叫 **alignment（對齊）**。

## 題目
## 0) 核心觀念：什麼叫 aligned / misaligned？
- **對齊 (aligned)**：位址是「資料大小」的倍數  
  - size = 2^k bytes  ⇒  address 的 **最低 k bits 必須全為 0**
- **未對齊 (misaligned)**：不滿足上述條件

等價判斷：
- aligned ⇔ `addr % size == 0`
- misaligned ⇔ `addr % size != 0`


## 1) 題型 A：32-bit word（4 bytes）哪些位址是 misaligned？
> 32-bit word = **4 bytes** ⇒ 需要 **4-byte aligned** ⇒ `addr % 4 == 0` 才 aligned  
> 也就是位址最後兩個 bit 必須是 `00`

題目選項：
- (a) `4N + 2`
- (b) `4N + 3`
- (c) `8N + 4`
- (d) `4N`
- (e) `8N + 1`
其中 N 是整數 (N ≥ 0)

### 判斷方法：看對 4 取餘數
- `4N`     ⇒ `mod 4 = 0` ✅ aligned
- `4N + 2` ⇒ `mod 4 = 2` ❌ misaligned
- `4N + 3` ⇒ `mod 4 = 3` ❌ misaligned
- `8N + 4` ⇒ `8N mod4=0` 且 `4 mod4=0` ⇒ `mod4=0` ✅ aligned
- `8N + 1` ⇒ `8N mod4=0` ⇒ `mod4=1` ❌ misaligned

✅ **答案： (a), (b), (e)**


## 2) 題型 B：判斷某位址在「某個對齊單位」下是否 aligned
常見對齊單位（都是 2 的次方）：
- word size 4B = 2^2  ⇒ 低 **2 bits** = 0
- word size 8B = 2^3  ⇒ 低 **3 bits** = 0
- cache line 32B = 2^5 ⇒ 低 **5 bits** = 0
- page size 8KB = 2^13 ⇒ 低 **13 bits** = 0

> 快速技巧（十六進位最好用）  
> - 1 個 hex digit = 4 bits  
> - 要檢查低 k bits 是否為 0：看最後幾個 hex digit 是否符合
>   - k=2 (4B)：最後一個 hex digit 必須是 0/4/8/C  
>   - k=3 (8B)：最後一個 hex digit 必須是 0/8  
>   - k=5 (32B)：最後 **5 bits**=0 ⇒ 最後一個 hex digit 要是 0，且倒數第二個 hex digit 的最低 1 bit 要是 0  

## 3) 題型 B（題目給的四個敘述）逐條整理

題目：
a) word size = 4 bytes，32-bit address `0xF00ABCCC` is aligned  
b) word size = 8 bytes，32-bit address `0xF00ABCCC` is aligned  
c) cache line size = 32 bytes，32-bit address `0xF0ABCCE0` is aligned  
d) page size = 8KB，32-bit address `0xF0ABCC00` is aligned  


### (a) 4 bytes 對齊：0xF00ABCCC
- 4B 對齊要 `addr % 4 == 0`
- 看最後一個 hex digit：`...CCC` 的最後一碼是 `C` (=12)
- `C` 是 0/4/8/C 其中之一 ⇒ 低 2 bits = 0  
✅ **aligned** ⇒ (a) True


### (b) 8 bytes 對齊：0xF00ABCCC
- 8B 對齊要 `addr % 8 == 0`
- 最後一個 hex digit 必須是 0 或 8（低 3 bits = 0）
- 這裡最後一碼是 `C` (=1100₂)，低 3 bits = 100 ≠ 000  
❌ **not aligned** ⇒ (b) False

### (c) 32 bytes 對齊：0xF0ABCCE0
- 32B 對齊要 `addr % 32 == 0`（低 5 bits 全 0）
- `...E0`：最後 1 byte 是 `0xE0` (=1110 0000₂)
- 低 5 bits 是 `00000`  
✅ **aligned** ⇒ (c) True


### (d) 8KB 對齊：0xF0ABCC00
- 8KB = 8192 = 2^13 ⇒ 低 13 bits 全 0
- `0x...CC00` 只有低 8 bits = 0（最後兩個 hex digit 是 00）  
  但要低 13 bits = 0 還需要再多 5 bits 也是 0（等價於最後至少要能整除 0x2000）
- `0xCC00` 不是 0x2000 的倍數（通常 8KB 對齊會看到最後是 `...0000` 或特定能被 0x2000 整除的形式）  
❌ **not aligned** ⇒ (d) False

✅ **答案： (a), (c)**


## 4) 速記
- 4B 對齊：最後 hex digit ∈ {0,4,8,C}
- 8B 對齊：最後 hex digit ∈ {0,8}
- 16B 對齊：最後 hex digit = 0
- 32B 對齊：最後 2 個 hex digit 必須是 {00,20,40,60,80,A0,C0,E0}
- 4KB 對齊(2^12)：最後 3 個 hex digit = 000
- 8KB 對齊(2^13)：要能被 0x2000 整除（低 13 bits = 0）

---
# Memory Endianness （位元組序）
## 名詞：Byte / Word / MSB / LSB
- **32-bit data = 4 bytes**（一個 word = 4 個 byte）
- **MSB (Most Significant Byte)**：最高位元組（最重要的 byte）
- **LSB (Least Significant Byte)**：最低位元組（最不重要的 byte）
- 例如 32-bit 數值寫成 4 個 byte：
  - `[B3 B2 B1 B0]`
  - `B3 = MSB`，`B0 = LSB`

##  Endianness 決定什麼？
> 決定「**同一個 word 的 4 個 byte**」要怎麼放進 memory 的連續位址中  
> （假設 word-aligned 起始位址是 `4N`，則該 word 會用到 `4N, 4N+1, 4N+2, 4N+3`）

##  Big Endian（大端）
### 定義
- **最小位址放 MSB，最大位址放 LSB**
- “Big” 的意思是「大的那端（MSB）放前面（小位址）」

### 4-byte mapping（從 `4N` 開始存一個 32-bit word）
| Address | 存的 byte |
|---|---|
| `4N` | `B3` (MSB) |
| `4N + 1` | `B2` |
| `4N + 2` | `B1` |
| `4N + 3` | `B0` (LSB) |

### 例子
若 32-bit value = `0x12 34 56 78`（B3=12, B2=34, B1=56, B0=78）  
Big Endian 存法：
- `mem[4N]   = 0x12`
- `mem[4N+1] = 0x34`
- `mem[4N+2] = 0x56`
- `mem[4N+3] = 0x78`



##  Little Endian（小端）
### 定義
- **最小位址放 LSB，最大位址放 MSB**
- “Little” 的意思是「小的那端（LSB）放前面（小位址）」

### 4-byte mapping（從 `4N` 開始存一個 32-bit word）
| Address | 存的 byte |
|---|---|
| `4N` | `B0` (LSB) |
| `4N + 1` | `B1` |
| `4N + 2` | `B2` |
| `4N + 3` | `B3` (MSB) |

### 例子
同樣 value = `0x12 34 56 78`  
Little Endian 存法：
- `mem[4N]   = 0x78`
- `mem[4N+1] = 0x56`
- `mem[4N+2] = 0x34`
- `mem[4N+3] = 0x12`

## 題目

## 題型 1：哪些 32-bit hex 在 Big/Little endian 下「memory 儲存序列相同」？

### 觀念
32-bit = 4 bytes，把資料寫成 4 個 byte：
- 32-bit 資料 = `[w x y z]`（w=MSB, z=LSB）

存進 byte-addressable memory 時：
- **Big endian**：`w x y z`
- **Little endian**：`z y x w`（整個 4-byte 順序反轉）

要「兩者儲存序列一模一樣」 ⇒ 必須滿足：
- `[w x y z] = [z y x w]`
- 所以 **w = z 且 x = y**
- 也就是 4 bytes 要長成：`[A B B A]`（回文 / 鏡像）

> 口訣：**32-bit 要在兩種 endian 下存法一樣 ⇒ 4 個 byte 必須是「左右對稱」**

### 套到選項（把 8 個 hex digits 視為 4 bytes：每 2 個 hex = 1 byte）
a) `AABBAABB` → bytes = `AA BB AA BB`（w=AA, x=BB, y=AA, z=BB）  
- w≠z（AA≠BB） ❌

b) `ABBAABBA` → bytes = `AB BA AB BA`  
- w≠z（AB≠BA） ❌

c) `ABBBBBAB` → bytes = `AB BB BB AB`  
- w=z=AB 且 x=y=BB ✅

d) `ABCDCDAB` → bytes = `AB CD CD AB`  
- w=z=AB 且 x=y=CD ✅

e) `ABCDDCBA` → bytes = `AB CD DC BA`  
- w≠z（AB≠BA）且 x≠y（CD≠DC） ❌

✅ **答案： (c), (d)**

---

## 題型 2：用 Little-endian 填入字串（每 row = 4 bytes）

題目：把 personal record `"Tom Lien"` 填入表格，假設 **每列 4 bytes**，使用 **little-endian**。

### 重要觀念（很容易搞混）
- **字元在記憶體中的「位址順序」仍然是照 address 由小到大放進去**
- 但題目說「用 little-endian」通常表示：**每 4 bytes 當作一個 word，word 內部 byte 反過來放**
  - 也就是：每 4 個字元一組 `[c0 c1 c2 c3]`  
    little-endian 存成 `[c3 c2 c1 c0]`


### 以 `"Tom Lien"` 為例（忽略空白通常當作字元也可以放；投影片的答案是把 8 個字元排成兩個 word）
投影片呈現的 8 個字元為：`T o m` + `L i e n`（共 8 bytes）

#### 分成兩個 4-byte word
- Word0（第 0~3 byte）：`T o m`（缺的那格可視為空白或下一字元，投影片做法是顯示成反轉後的排列）
- Word1（第 4~7 byte）：`L i e n`

#### Little-endian：每個 word 內反轉
- Word0: `[T o m ?]` → memory 依序放成 `? m o T`
- Word1: `[L i e n]` → memory 依序放成 `n e i L`

---

### 依投影片表格（位址 0~7）的填法
| Addr | 0 | 1 | 2 | 3 |
|---:|---|---|---|---|
| Content | *(空/補位)* | m | o | T |

| Addr | 4 | 5 | 6 | 7 |
|---:|---|---|---|---|
| Content | n | e | i | L |

> 你只要記得：**little-endian = 同一個 word 內的 byte 反過來**  
> 所以 `"Lien"` 會在 memory 變成 `n e i L`


## 快速總結
- **32-bit endian 存法相同** ⇔ 4 bytes 必須是 `[A B B A]`（左右對稱）
- **little-endian 填字串（每 4 bytes 一列）**：每列 4 個字元 **反過來放**

# 容量表

|Decimal Term|Abbreviation|Value|Binary term|Abbreviation|Value|
|---|---|---|---|---|---|
|KiloByte|KB| $10^3$ |Kibibyte|KiB| $ 2^{10} $ |
|MegaByte|KB| $10^6$ |Mibibyte|MiB| $ 2^{20} $ |
|GigaByte|KB| $10^{9}$ |Gibibyte|GiB| $ 2^{30} $ |
|TeraByte|KB| $10^{12}$ |Tibibyte|TiB| $ 2^{40} $ |
|PetaByte|KB| $10^{15}$ |Pibibyte|PiB| $ 2^{50} $ |
|ExaByte|KB| $10^{18}$ |Exbibyte|EiB| $ 2^{60} $ |
|ZettaByte|KB| $10^{21}$ |Zebibyte|ZiB| $ 2^{70} $ |
|YottaByte|KB| $10^{24}$ |Yobibyte|YiB| $ 2^{80} $ |

