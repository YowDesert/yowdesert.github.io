---
title: (計算機組織) 計算機組織摘要-Register
categories:
  - 計算機組織
  - 第一章
tags:
  - 計算機組織
abbrlink: 3f087bff
mathjax: true
date: '2026-04-03 13:20:00'
---

# MIPS 暫存器架構筆記

- 所有暫存器都是 32 bits

## 2. CPU 暫存器分類

CPU 內的暫存器可以分成兩大類：

### (1) General-Purpose Register, GPR (一般目的暫存器) 

一般用途暫存器，用來存放一般資料、運算元、參數等。

例子有：

* `$s0 ~ $s7`
* `$t0 ~ $t9`
* `$a0 ~ $a3`
* 還有其他暫存器

可整理成：

* **$s** 系列：通常用來放需要保留的資料
* **$t** 系列：通常用來放暫時資料
* **$a** 系列：通常用來放函式參數

### (2) Special-Purpose Register, SPR (特殊目的暫存器)

特殊用途暫存器，用來支援 CPU 的特定功能。

* **Hi** (乘積左半餘數)
* **Lo** (乘積右半商數)
* **PC**

其用途：

* **Hi / Lo**：通常和乘法、除法運算結果有關
* **PC (Program Counter)**：存放下一個要執行的指令位址

---

## 3. CPU 

* CPU 有一組 **Registers**（編號 `$0` 到 `$31`）
* CPU 內有 **Arithmetic unit**（算術單元）
* CPU 內有 **Multiply divide** 單元
* Multiply/divide 單元會搭配：

  * **Lo**
  * **Hi**

* 一般加減等運算交給 **Arithmetic unit**
* 乘法、除法等運算交給 **Multiply divide** 單元
* 而乘除結果會和 **Hi / Lo** 這兩個特殊暫存器配合

---

## 4. FPU（浮點運算單元）

### FPU 暫存器

圖中寫：

* **FPU：`$f0 ~ $f31`**

表示浮點運算單元有一組自己的暫存器：

* 共 32 個浮點暫存器
* 編號從 **`$f0` 到 `$f31`**

### FPU 方塊圖重點

* Coprocessor 1 就是 **FPU**
* 裡面有一組 Registers（`$0 ~ $31`）
* 也有自己的 **Arithmetic unit**

### CPU/FPU

* CPU 主要處理整數與一般控制工作
* **FPU 專門處理浮點數運算**
* 因此 FPU 會有自己獨立的暫存器和算術單元

---

## 5. Coprocessor 0 (協同處理器)

* **Status**：記錄處理器狀態
* **Cause**：記錄例外發生原因
* **EPC**：記錄發生例外時的程式位置
* **BadVAddr**：記錄造成錯誤的位址

---

## PC（Program Counter）：

- 存放「目前正在執行的指令位址」的暫存器。

---

## GPR 

## GPR 總表

| Type | Name | Number | Usage |
|---|---|---|---|
| Assembler related | `$at` | `$1` | Preserved for Assembler |
| OS related | `$k0 - $k1` | `$26 - $27` | Preserved for OS |
| Procedure call | `$v0 - $v1` | `$2 - $3` | Values for Results |
| Procedure call | `$a0 - $a3` | `$4 - $7` | Values for Arguments |
| Procedure call | `$ra` | `$31` | Value for Return address |
| Variables / constants | `$s0 - $s7` | `$16 - $23` | Saved |
| Variables / constants | `$t0 - $t7` | `$8 - $15` | Temporaries |
| Variables / constants | `$t8 - $t9` | `$24 - $25` | More temporaries |
| Variables / constants | `$zero` | `$0` | The constant value 0 |
| Memory management | `$gp` | `$28` | Global pointer |
| Memory management | `$sp` | `$29` | Stack pointer |
| Memory management | `$fp` | `$30` | Frame pointer |

- `$at`：assembler 用
- `$k0 ~ $k1`：OS 用
- `$v0 ~ $v1`：回傳值
    - 常用 : 用之前存，跳回去的時候刪除
- `$a0 ~ $a3`：參數
- `$ra`：返回位址
- `$s0 ~ $s7`：saved
    - 一定要從 `$s0` 開始排
- `$t0 ~ $t9`：temporary
- `$zero`：固定 0
    - 0 是最常用的常數之一，尤其常拿來做比較，所以 MIPS 把 $0 固定成 0，讓 CPU 可以隨時直接使用。
- `$gp`：global pointer
- `$sp`：stack pointer
- `$fp`：frame pointer

## Caller / Callee

- Caller
    呼叫別人的函式。

- Callee
    被呼叫的函式。

## Argument / Parameter
- Argument
    呼叫函式時，實際傳進去的值。
- Parameter
    函式定義裡接收 argument 的變數。

- 如果有大於4個parameters ?
    - 剩下的放memory

## Caller / Callee 為什麼會覆蓋 register ? 
- 不同 procedure 若共用同一批 register，彼此的值可能被覆蓋。

### 解法

- 方法 1：用 saved register 規則
callee 先把原本值存起來，結束前再恢復。

- 方法 2：把值存到 stack
這也是後面 stack / frame 出現的理由。

### 方法 1：用 saved register 規則

####  概念
這個方法是利用 MIPS 的 **saved register** 規則。

在 MIPS 中：

- `$s0 ~ $s7` 是 **saved registers**
- 如果 **callee（被呼叫函式）** 要使用這些暫存器
- 就必須：
  1. 先保存原本值
  2. 用完後再恢復

### 重點
- 這是一種 **規則 / calling convention**
- 重點在於：**哪些暫存器不能隨便破壞**
- 如果用了 `$s` 暫存器，函式結束前要還原

### 例子
```asm
foo:
    addi $sp, $sp, -4
    sw   $s0, 0($sp)     # 先保存原本的 $s0

    li   $s0, 123        # 函式內使用 $s0

    lw   $s0, 0($sp)     # 結束前恢復
    addi $sp, $sp, 4
    jr   $ra
```

### 特點
- 適合放 **需要保留較久的值**
- 由 **callee 負責保護**
- 常和 `$s0 ~ $s7` 一起出現



### 方法 2：把值存到 stack

#### 概念
如果某個值怕被覆蓋，就先把它存到 **stack（堆疊）** 裡，等需要時再取回。

#### 做法
通常步驟是：

1. 用 `$sp` 挪出空間
2. 用 `sw` 把值存進 stack
3. 之後再用 `lw` 讀回
4. 最後把空間還回去

#### 例子
```asm
addi $sp, $sp, -4
sw   $t0, 0($sp)    # 把 $t0 存到 stack

...

lw   $t0, 0($sp)    # 從 stack 取回
addi $sp, $sp, 4
```

#### 重點
- 這是一種 **實作方式**
- 重點在於：**資料實際存到哪裡**
- 存放位置是 **主記憶體中的 stack 區域**

---

### 兩種方法的差別

| 方法 | 重點 | 本質 |
|---|---|---|
| 方法 1：用 saved register 規則 | 哪些暫存器必須保留、誰負責保留 | 規則 |
| 方法 2：把值存到 stack | 保留時資料實際放在哪裡 | 做法 |


## 記憶體區域概念

- Code
存程式碼

- Static
存全域變數、靜態變數

- Heap
動態配置記憶體（如 malloc()）

- Stack
區域變數、程序呼叫資訊、saved registers 等
- Reserved
保留區

### $gp = Global Pointer

- $gp 指向 static area

- 存取：
    - global variables
    - static variables

### $sp = Stack Pointer

- $sp 指向 stack 中「最新配置的位置」

- stack 是：
    後進先出（LIFO）
    有 push / pop
    程序呼叫時會動到

### $fp = Frame Pointer


- $fp 指向目前 procedure 的 frame（activation record）中的固定位置。

- 定義
    - $fp 指向目前 procedure 的 frame（activation record）中的固定位置。
    - 為什麼還要 $fp？
     因為 $sp 在函式執行過程中可能一直變：
     如果只靠 $sp，
    那某個 local variable 的 offset 可能前後變來變去。

- $fp 提供一個穩定基準點，
讓區域變數、saved registers、額外參數都能用固定 offset 存取。

## Procedure Frame
- 又叫：activation record

### 定義
**Procedure frame** 就是某個函式在執行時，在 stack 上切出來的一塊專屬空間。

這塊空間通常拿來存放：

- 被保存的暫存器
- 區域變數
- 返回位址
- 與函式執行有關的暫存資料

- 為什麼需要兩個 pointer：$sp 與 $fp？
    - $sp 會改變
    - $fp 保持穩定

- 只有MIPS有$fp ， 其他只有$sp

## Spilling Register

- 因為：
程式裡的變數很多
register 數量有限
所以不可能所有變數都一直待在 register 裡。

- 定義 : 
Spilling registers：
把比較不常用、暫時不用、之後才會用的變數，
先放到記憶體裡。

需要時再：
- load 回 register
- 不需要時再 store 回 memory

- 最常使用的變數留在 registers
其他變數放 memory
- 透過 **load/store** 在 register 與 memory 間搬移


### 為什麼不乾脆做超多 registers？
- 理由 1：decoder 變慢
register 太多時，
存取 register 要經過 decoder，
decoder 會更複雜、更慢。

- 理由 2：clock cycle 變長
訊號傳得更遠、更複雜，
CPU 時脈週期可能變長，
整體效能反而下降。

- 理由 3：功耗增加
更多 register 會增加 power consumption。

- 核心設計原則
Smaller is faster

- 但少也有可能變爛
    - 因為CPU與memory 交換次數多就會變爛

## RISC-V

### (1) RISC-V 的 register file
- RISC-V 有 **32 個 general-purpose registers**
- 編號為：
  - `x0 ~ x31`
- 每個 register 都是 **32-bit**
- 因此可說是：
  - **32 × 32-bit register file**
- 這些 registers 主要用來存放：
  - 常用資料
  - 運算中的中間值
  - 函式參數
  - 回傳值
  - 指標與位址資訊

---

### (2) 32-bit data 叫做什麼？
- **32-bit data 稱為一個 word**

也就是：
- 1 word = 32 bits = 4 bytes

---

### 2. RISC-V 暫存器總表

| Register | Symbol name | Description | Saver |
|---|---|---|---|
| `x0` | `zero` | Hard-wired zero | - |
| `x1` | `ra` | Return address | Caller |
| `x2` | `sp` | Stack pointer | Callee |
| `x3` | `gp` | Global pointer | - |
| `x4` | `tp` | Thread pointer | - |
| `x5 ~ x7` | `t0 ~ t2` | Temporaries | Caller |
| `x8` | `s0 / fp` | Saved register / frame pointer | Callee |
| `x9` | `s1` | Saved register | Callee |
| `x10 ~ x11` | `a0 ~ a1 / v0 ~ v1` | Function arguments / return values | Caller |
| `x12 ~ x17` | `a2 ~ a7` | Function arguments | Caller |
| `x18 ~ x27` | `s2 ~ s11` | Saved registers | Callee |
| `x28 ~ x31` | `t3 ~ t6` | Temporaries | Caller |

## Translation Hierarchy

High-level language
→ Compiler
→ Assembly language
→ Assembler
→ Machine language object module
→ Linker
→ Executable (.EXE)
→ Loader
→ 載入 memory 執行

### 各工具功能
- Compiler
把高階語言翻成組合語言

- Assembler
把組合語言翻成機器語言（object module）

- Linker
把多個 object modules 與 library routines 結合，
並解析符號參照

- Loader
把 executable 放進記憶體並初始化執行環境

### Loader 做什麼？

1. 讀 executable header，決定 text / data segment 大小
2. 建立足夠大的 address space
3. 把 instructions 與 data 複製到 memory
4. 把程式參數放到 stack
5. 初始化 machine registers，設定 stack pointer
6. 跳到 start-up routine，之後呼叫 main，main 結束後用 system call 結束程式

## Program 與 Procedure 差別
- Program：執行單位
- Procedure：編譯 / 組譯單位

## Compilation 與 Interpretation
### 重點
- 高階語言先經過 **compiler** 轉成目標機器可直接執行的 **machine code**
- 產生出來的 machine code 是針對特定硬體平台的
- 編譯完成後，執行時不需要再逐行翻譯

### 優點
- **高效能（High performance）**
- 可做較多 **最佳化（Optimization）**

### 缺點
- **可攜性較低（Low portability）**
- 因為不同硬體（如 x86 / MIPS / ARM）通常要重新編譯

### Interpretation（直譯 / 解譯式執行）
### 重點
- Java 原始程式先被編成 **Java bytecode**
- bytecode 不是直接給硬體執行
- 而是交給 **JVM** 來解讀與執行

### 優點
- **高可攜性（High portability）**
- 同一份 bytecode 只要有 JVM 的平台就能執行

### 缺點
- **效能較低（Low performance）**
- 因為執行時還要經過 JVM 解譯

### JVM 與 JIT

#### JVM（Java Virtual Machine）
- Java 的解譯器 / 執行環境
- 負責執行 Java bytecode

#### JIT（Just In Time Compiler）
- 為了提升效能，JVM 可以啟動 **JIT compiler**
- JIT 會在程式執行時分析哪些方法最常用（hot methods）
- 再把這些方法編譯成本地平台的 native code

## JAVA優勢
Java 並不是「完全純直譯」
而是：

```text
Java source
→ bytecode
→ JVM 執行
→ 必要時 JIT 再編成本地碼
```

- Java 的最大優勢：**Machine independence（機器獨立性）**
- 代價：通常效能不如直接編譯成 machine code 的 C/C++

###  常考題目整理
#### 題型 1
**Java 最重要的優點是什麼？**  
答：**Machine independence**

#### 題型 2
**為什麼 C 通常比 Java / Python 快？**  
答：
- C 屬於編譯式語言
- 可直接轉成 machine code
- 不需像直譯式系統一樣執行時再多一層翻譯

#### 題型 3
**JIT 的作用是什麼？**  
答：
- 找出執行中的熱點程式碼
- 將其編譯為 native code
- 提升執行效能


## Instruction Types


| 大類 | 子類 | 細項 / 功能 |
|---|---|---|
| Data Movement（Data Transfer） | - | load（from memory） |
| Data Movement（Data Transfer） | - | store（to memory） |
| Data Movement（Data Transfer） | - | memory-to-memory move |
| Data Movement（Data Transfer） | - | register-to-register move |
| Data Movement（Data Transfer） | - | input（from I/O device） |
| Data Movement（Data Transfer） | - | output（to I/O device） |
| Data Movement（Data Transfer） | - | push, pop（to/from stack） |
| Operation | Arithmetic | (integer or FP) add, sub, multiply, divide |
| Operation | Logical | shift, rotate, set, clear |
| Operation | Logical | not, and, or |
| Flow Control | Intra program | unconditional, conditional |
| Flow Control | Inter program | call, return |
| Flow Control | System call | trap, return |


##  Instruction Set 的三大類

###  Data Movement（Data Transfer）

- 把資料從一個地方搬到另一個地方
- 例如在 memory、register、I/O、stack 之間傳送資料

圖中列出的例子有：

- **load**（from memory）
- **store**（to memory）
- **memory-to-memory move**
- **register-to-register move**
- **input**（from I/O device）
- **output**（to I/O device）
- **push, pop**（to/from stack）

---

##  Data Movement 各項說明

### load
- 從 **memory 讀資料** 到 CPU / register
- 例如把記憶體中的某個值載入暫存器

### store
- 把資料從 CPU / register **寫回 memory**

> MIPS 只有上面兩個 `load` , `store`

### memory-to-memory move
- 直接把資料從某個記憶體位置搬到另一個記憶體位置

### register-to-register move
- 把某個 register 的值搬到另一個 register

### input
- 從 **I/O device** 把資料送進來

### output
- 把資料從 CPU 送到 **I/O device**

### push / pop
- 和 **stack** 有關
- **push**：把資料放進 stack
- **pop**：從 stack 取出資料

---

##  Operation（運算）

這一類指令負責對資料做運算。

圖中把 Operation 分成兩大類：

- **Arithmetic**
- **Logical**

---

##  Arithmetic（算術運算）

圖中寫：

- **(integer or FP)**
- **add, sub, multiply, divide**

代表算術運算可以包含：

- **整數運算（integer）**
- **浮點數運算（FP = floating point）**

常見算術指令有：

- **add**：加法
- **sub**：減法
- **multiply**：乘法
- **divide**：除法

---

##  Logical（邏輯運算）

圖中列出兩部分：

### 第一部分
- **shift**
- **rotate**
- **set**
- **clear**

### 第二部分
- **not**
- **and**
- **or**

---

##  Logical 指令各項理解

### shift
- 位元左移或右移

### rotate
- 位元旋轉

### set
- 將某些位元設成指定值

### clear
- 將某些位元清成 0

### not
- 位元反相

### and
- 位元 AND 運算

### or
- 位元 OR 運算

這類指令通常和：
- bit manipulation
- mask
- 權限位元
- 條件判斷前處理
很有關係

---

##  Flow Control（流程控制）

這一類指令不是在算數值，
而是在控制：

- 程式接下來要執行哪裡
- 是否跳躍
- 是否呼叫函式
- 是否進入系統服務

圖中把 Flow Control 分成三類：

- **Intra program** : 跳在內部
- **Inter program** : 跳出去別的Procedure
- **System call**

---

##  Intra program（程式內部流程控制）

圖中寫：

- **unconditional**
- **conditional**

這表示在同一個程式內部進行跳轉。

### unconditional
- 無條件跳躍
- 不管條件如何都直接跳到指定位置

### conditional
- 條件跳躍
- 條件成立才跳，不成立就繼續往下執行

---

## Inter program（跨程式 / 函式層級控制）

圖中寫：

- **call**
- **return**

這一類主要是在做：

- 函式呼叫
- 從函式返回

### call
- 跳到另一段程式 / 函式去執行

### return
- 執行完後回到原來呼叫的位置

這和：
- return address
- stack
- procedure frame
有很大關係

---

## System call（系統呼叫）

圖中寫：

- **trap**
- **return**

這一類和作業系統有關。

### trap
- 程式主動或被迫進入系統 / 例外處理流程

### return
- 從系統服務或例外處理返回原程式

---

##  整張圖的總整理

### A. Data Movement
負責搬資料
- load
- store
- move
- input / output
- push / pop

### B. Operation
負責運算
- Arithmetic：加減乘除
- Logical：位元操作與邏輯運算

### C. Flow Control
負責改變程式執行流程
- Intra program：程式內跳躍
- Inter program：call / return
- System call：trap / return

---

##  這頁和前面 register / memory 的關係

這頁其實是在把前面學過的 register、memory、stack、procedure call 串起來：

- **load / store**：對應 register 和 memory 之間搬資料
- **push / pop**：對應 stack
- **call / return**：對應 procedure call、`ra`、stack frame
- **trap / return**：對應 system call、exception、OS

所以這張圖算是整體指令功能的大分類圖。

---
