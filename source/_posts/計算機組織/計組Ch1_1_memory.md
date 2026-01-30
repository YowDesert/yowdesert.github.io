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

# Chapter 1：計算機組織與指令集架構（ISA）

## 一、Computer Organization 在學什麼？
計算機組織（Computer Organization）探討的是**程式如何在硬體上被執行**，也就是從高階程式碼一路到 CPU、記憶體、電路層級的整體運作方式，是**軟體與硬體的橋樑**。

![alt text](/img/CSch1_1.png)

>圖中元件簡介（由上到下）

### 軟體層

- Word / Media player / Edge（應用程式 App）
使用者直接操作的軟體，例如文書、播放器、瀏覽器。  
通常用**高階語言**寫成，CPU 不能直接執行。

- Operating System (OS)（作業系統）
系統軟體核心，負責管理資源並提供服務，例如：
  - 行程/排程（Process & Scheduling）
  - 記憶體管理（Memory Management）
  - 檔案系統（File System）
  - I/O 裝置管理（Device / I-O）

- Compiler（編譯器）
把**高階語言（如 C/C++）**轉換成**組合語言（Assembly）**，產生較接近硬體的指令表示。

- Assembly language（組合語言）
用符號表示的低階語言（人可讀、CPU 仍不能直接跑）。  
例：`add`、`lw`、`sw`、`beq`

- Assembler（組譯器）
把**組合語言**轉成**機器語言（二進位機器碼）**，完成 CPU 可執行的指令編碼。

- Machine language（機器語言）
CPU 可直接執行的二進位指令（0/1）。  
每條指令的 bit pattern 由 ISA 規定。

### 軟硬體交界 ISA

- Instruction Set Architecture (ISA)
**軟體與硬體的「契約/介面」**，規定：
  - CPU 有哪些指令（instruction set）
  - 暫存器有哪些、怎麼用
  - 指令格式與編碼方式
  - 記憶體存取規則等

> 同一個 ISA 可以有不同硬體實作（例如 pipeline / non-pipeline），但都能跑同一套程式。

### 硬體層（ISA 以下）

- Datapath（資料路徑）
負責「做運算與搬資料」的硬體部分：  
例如 ALU、register file、mux、加法器等。

- Controller（控制單元）
負責「發控制訊號」，決定每個 clock 週期 datapath 要做什麼（選路徑、寫暫存器、讀寫記憶體）。

- Memory（記憶體）
存放指令與資料的地方（主記憶體/快取等）。  
CPU 透過 load/store 讀寫資料。

- Input/Output（I/O）
與外部裝置溝通的介面，例如鍵盤、螢幕、磁碟、網路等。

- Logic Circuit（邏輯電路）
用邏輯閘（AND/OR/NOT…）組成，實作各種運算與控制。

- Switching Circuit（開關電路）
用電晶體（transistor）當開關，實現 0/1 的切換與邏輯行為。

- VLSI Technology（VLSI 製程）
把大量電晶體做在晶片上的製程技術（Very Large Scale Integration），是能把 CPU 做出來的基礎。

---

## 二、Stored-Program Concept（儲存程式概念）

### 定義
Stored-program concept 指的是：
- **指令（instruction）與資料（data）都以數字形式存放在記憶體中**
- 程式必須先載入記憶體，CPU 才能取指並執行

此概念源自 **John von Neumann**。

- 題目 : 在計算機結構的脈絡下，簡要描述 *stored-program concept*
Stored-program concept（儲存程式概念）指的是：  
> **各種形式的指令（instructions）與資料（data）都能以「數字（numbers）」的形式存放在記憶體（memory）中**，  
> 因此電腦可以像存取資料一樣存取與執行程式，形成「儲存程式電腦（stored-program computer）」。
---

## 三、Von Neumann Architecture

### 五大經典組成元件
1. **Input**
2. **Output**
3. **Memory**
4. **Datapath**
5. **Control**

其中：
- Datapath + Control 又可以叫做 **Processor** (CPU)

### 各元件功能
- **CPU**：執行指令的核心，負責運算與控制
- **Datapath**：實際進行算術與邏輯運算（ALU、Registers）
- **Control Unit**：控制資料流向與操作時序
- **Memory**：存放指令與資料
- **I/O**：與外部世界互動

![alt text](/img/FiveComputer.png)

### Von Neumann Bottleneck（重點）
- 指令存取與資料存取**共用同一條匯流排**
- 無法同時進行，造成效能瓶頸
- **取指令（instruction fetch）** 和 **存取資料（data operation）**  
  **無法同時進行** 
- 稱為 **Von Neumann bottleneck**

### Harvard Architecture（對比）

- Von Neumann Architecture （指令/資料共用記憶體）

  ![alt text](/img/VonNeumann.png)

- Harvard Architecture（指令/資料分開記憶體）

  ![alt text](/img/Harvard.png)




| 比較項目      | Von Neumann（儲存程式 / 馮紐曼）                          | Harvard（哈佛）                          |
| --------- | ------------------------------------------------ | ------------------------------------ |
| 記憶體配置     | **指令與資料共用同一個 Memory**                            | **指令 Memory 與資料 Memory 分離**          |
| Bus（匯流排）  | **共用一條 bus**（instruction fetch & data access 競爭） | **兩條 bus**（取指與存取資料可同時）               |
| 主要瓶頸      | **Von Neumann bottleneck**（常見考點）                 | 較不易出現同類瓶頸（但仍可能受限於其他資源）               |
| 並行能力      | 取指與資料存取**難以同時**                                  | 取指與資料存取**可並行**                       |
| 設計複雜度     | 較簡單、成本較低                                         | 較複雜（兩套 memory/bus 管理）                |
| 程式/資料彈性   | 記憶體空間可彈性分配（同一空間）                                 | 空間分離，配置較固定（需設計規劃）                    |
| 安全性/隔離    | 指令與資料同空間（理論上較容易互相影響）                             | 指令與資料天然隔離（設計上較清楚）                    |
| 常見應用      | 通用電腦概念模型、許多系統抽象                                  | 微控制器、DSP、嵌入式；或 CPU 內部採類 Harvard      |
| 現代 CPU 實務 | 多為「概念上 Von Neumann」                              | 多為「**Modified Harvard**」（內部分開 cache） |


---

## 四、Instruction Set Architecture（ISA）

### 什麼是 Instruction？
- **Instruction**：CPU 可以直接執行的「最原始/最基本」操作
- **Instruction Set**：CPU 能執行的所有指令集合

### Statement vs Instruction
- 在**高階語言**（High-level language）中，電腦命令通常叫 **statement**  
  例：`p = x + y;`
- 在**低階語言**（Low-level language，如組語/機器碼）中，命令叫 **instruction**  
  例（MIPS）：`add $s1, $t0, $t1`

### ISA 定義（必背）
> **ISA = Instruction Set + Hardware Information**

ISA 是：
- 最低階軟體 **(Machine Instructions)** 與硬體之間的**抽象介面**
- 不同硬體實作（single-cycle / pipeline）只要 ISA 相同，就能執行相同程式

### ISA 包含的資訊
1. Instruction set
2. Hardware Information
  1. Memory
  2. Registers
  3. Instruction format
  4. Addressing modes

### 同一 ISA，不同 CPU 結構設計

- **Single Cycle Machine（單週期）**
  - 一條指令在 **1 個 clock** 完成
  - 控制簡單，但 clock 週期通常要設得很長（最慢指令決定）

- **Multiple Cycle Machine（多週期）**
  - 一條指令拆成多個步驟（多個 clock）
  - 每個 clock 做一小步，週期可變短，硬體資源可重複利用

- **Pipeline（管線化）**
  - 指令分成多個 stage（如 IF/ID/EX/MEM/WB）
  - 多條指令「重疊」執行，提高吞吐量（throughput）
  - 但會遇到 hazard（資料/控制/結構冒險）需要處理

---

## ISA 的兩大類型：CISC vs RISC（1980s 之後常被拿來比較）

### CISC（Complex Instruction Set Computer）
**核心想法：功能強、指令複雜，讓寫程式更方便**
- 指令集除了基本指令外，也包含「更複雜、功能更強」的指令  
  （一條指令可能做很多事）
- 設計重點偏向：
  - 提供更好的程式開發環境
  - 減輕程式設計師負擔（少寫一些指令）

> 白話：用「更厲害的單條指令」來縮短程式碼、讓人比較好寫。


### RISC（Reduced Instruction Set Computer）
**核心想法：指令簡單、規則一致，提高 CPU 執行效率**
- 以「CPU 執行效率」為主要考量
- 指令數量較精簡，通常只保留基本且常用的操作
- 目的：用簡單指令搭配硬體設計（如 pipeline）  
  來達到更快的指令執行速度（更高吞吐量）

> 白話：不要做很複雜的指令，改成很多「簡單、快、好切 pipeline」的小指令。

### 常考補充

### 為什麼 1980s 開始強調 RISC？
- 指令越複雜，硬體控制越難做、時脈可能被拖慢
- RISC 指令固定且簡單 → **更適合 pipeline** → 整體效能常更好

### 一句話記憶
- **CISC：指令多而強 → 程式碼短、硬體較複雜**
- **RISC：指令少而簡 → 程式碼可能較長、硬體更好加速（pipeline）**

---


## MIPS 與 Word（資料大小）重點整理

### 有很多種ISA
- Intel's IA-32,MIPS,ARM,還有MIPS ISA
> ARM 是 3C家電用的
> x86貴 MIPS 便宜
> 2/3用MIPS 1/3用RISC-V(ex.台大)


### 1) MIPS 是什麼？
- **MIPS** 全名：**Microprocessor without Interlocked Pipe Stages**
- 起源：Stanford University 的研究計畫
- 計畫主持/代表人物：**John L. Hennessy**
  - 也是經典教科書 *Computer Organization and Design: The Hardware/Software Interface* 的作者之一 (白算盤)

---

### 2) MIPS 家族與本課使用版本
MIPS 是一個處理器/指令集家族，常見成員包含：
- **R2000 / R3000**：32-bit
- **R4000 / R4400**：64-bit
- **R8000 / R10000**：較偏科學計算 / 圖形應用

本課主要介紹：**MIPS R2000 instruction set**。

---

### 3) MIPS 屬於哪一類 ISA？
- MIPS 屬於 **RISC（Reduced Instruction Set Computer）**
- 特性：指令較精簡、規則一致，適合做 pipeline、提升執行效率

---

## Word 與 32-bit CPU

### 4) 什麼是 Word？
- **Word**：CPU 一次能處理的資料（operand）大小
- Word 可能是：16 bits / 32 bits / 64 bits …（依 CPU 設計而定）
- 若 CPU 的 word size = **32 bits**  
  → 稱為 **32-bit CPU**

> 直覺：CPU「一次能算多大」就叫它的 word size。

---

### 5) MIPS R2000 是 32-bit CPU
- 以 **MIPS R2000** 指令集為基礎建的 CPU，屬於 **32-bit CPU**
  - 因為它的 word size 為 32 bits

---

### 6) Bit / Byte / Word 基本單位
- **Bit**：最小單位，0 或 1
- **Byte**：**8 bits**
- **Word**：**N bytes**（由 CPU 決定）
  - 例如 32-bit CPU：1 word = 32 bits = 4 bytes

---

### 7) 從硬體規格看 CPU 是幾 bit
CPU 能一次處理的資料大小，也可以從硬體規格推得：
- **32-bit CPU** 通常具有：
  - **32-bit ALU（Arithmetic Logic Unit）**
  - **32-bit General Purpose Registers（一般用途暫存器）**

> 簡單判斷：ALU 幾 bit、暫存器幾 bit，通常就對應 CPU 幾 bit。


---

## 七、Translation Hierarchy（程式轉換流程）

### 完整流程
```text
High-level Language (C)
        ↓ Compiler
Assembly Language
        ↓ Assembler
Machine Language
        ↓ Linker
Executable File
        ↓ Loader
Memory → CPU 執行
```

### 各工具功能
- **Compiler**：高階語言 → 組合語言
- **Assembler**：組合語言 → 機器語言
- **Linker**：整合多個 object 檔，解決符號參考
- **Loader**：
  - 建立記憶體空間
  - 複製程式碼與資料
  - 初始化暫存器
  - 設定 PC

---

## 八、Compiler vs Interpreter

### Compiler（如 C）
- 先翻譯成機器碼再執行
- 效能高
- 可攜性較低

### Interpreter（如 Java、Python）
- 執行時逐行解譯
- 效能較低
- **高度可攜**

Java：
- 先編譯成 bytecode
- 由 JVM 解譯
- JIT 將 hot code 即時編譯提升效能

---

## 九、Instruction Types（指令分類）

### Data Movement
- load / store
- register ↔ register
- memory ↔ memory
- input / output
- push / pop

### Arithmetic / Logic
- add, sub, mul, div
- and, or, not
- shift, rotate

### Flow Control
- branch（條件 / 無條件）
- jump
- procedure call / return
- system call（trap）

---

## 十、重點總整理（考前速記）
- Stored-program：指令與資料都在 memory
- ISA 是軟硬體的抽象介面
- Von Neumann bottleneck：共用 bus
- RISC：指令少、快、適合 pipeline
- MIPS：32-bit、RISC
- Compiler ≠ Assembler ≠ Linker ≠ Loader

---
