---
title: (作業系統) Process and Concurrency - Synchronization Concepts
categories:
  - 作業系統
  - Process and Concurrency
tags:
  - 作業系統
abbrlink: 90aa57da
mathjax: true
date: 2025-08-14 08:40:00
---

# 簡短說明：
- 主題：Synchronization Concepts
- 本章重點：
    - 什麼情況下會發生 Race Condition？
    - 作業系統必須滿足哪些條件，才能正確管理共享資源？
    - 有哪些同步機制（Synchronization Mechanisms）可以保證程式正確性？

---
<!-- <span style="color:red">data inconsistency</span> -->
# Background

## 問題與解決

- The Problem :
    - **Concurrent access** to shared data may result in **data inconsistency**
    - Outcome depends on the **order of execution**(race condition)

- The Solution :
    - **Ensure orderly execution**
    - **Synchronization** : 協調資源的分享與存取
    - 保證 **Data Integrity** 和 **Correctness**

## Consumer and Producer Problem :
- 變數意義 :
    - in：下一個要放入的位置（next free position）
    - out：下一個要取出的項目位置（first available item）
    - buffer 是 circular queue（環狀），所以會用 ```% BUFFER_SIZE```
- Consumer and Producer Problem :
    - Producer 端邏輯
        - ```while (((in+1) % BUFFER_SIZE) == out) {}```
            → 代表 buffer 滿了（下一格就是 out，表示沒有空位）所以一直等
    - Consumer 端邏輯
        - ```while (in == out) {}```
        → 代表 buffer 空（沒有可取的）

- 如果加上 Counter : 直接記錄 buffer 內「目前有幾個 item」
    - 新增變數 : ```counter = 0```：buffer 目前 item 數量
    - Producer（用 counter）
        - while ```(counter == BUFFER_SIZE) {}``` → 滿了就等
        - 放入後 ```counter++```
    - Consumer（用 counter）
        - while ```(counter == 0) {}``` → 空了就等
        - 取走後 ```counter--```
- 簡化法 : counter ++ 與 counter -- 其實是3步驟
    - 把 counter 從記憶體讀到暫存器
    - 暫存器 +1
    - 寫回記憶體
- Problem : 
    - 在這 3 步之間，可能發生 **context switch** 各自讀到舊的值，最後互相覆蓋對方寫回的結果 => Race Condition

## Race Condition
- Race Condition 會發生在以下情況 :
    1. Multiple processes access and manipulate **shared data concurrently**
    2. The final value depends on which process finishes last
    3. Outcome is **unpredictable**（timing-dependent） 

- Prevention : Synchronization（同步）
    - **Ensure only one process accesses shared data at a time**

- Simple Prevention (只適用於單核心)
    1. Disable Interrupts（關閉中斷）
        - 關閉中斷 → CPU 不會發生 context switch
    2. Non-Preemptive Scheduling（非搶占式排程）
        - Process 一旦開始執行，不**會被 OS 強制切換**
- Both only work on single-processor systems

## DeadLock

- 一開始為假設count = 0 
- Consumer 進入 CS First
    1. entry-section() → 拿到鎖
    2.  ```while (counter == 0) { }```
    3. Producer 想進來 ， entry-section()（鎖被 Consumer 拿走）
    - Result: 
        - Consumer：等 Producer 產生資料
        - Producer：等 Consumer 放鎖
    - 互相等待 → Deadlock

## 原則
- 原則 1
    - 不要在 critical section 裡面「等條件」 → 會 deadlock
- 原則 2
    - critical section 要「越小越好」，只包真正需要保護的**共享資料**
- 原則3
    - 互斥 ≠ 同步

# Critical-Section

## The Critical-Section Problem
- Purpose（目的）:
    - Design a **protocol** : 允許處理器們協調與安全的存取共用的資料
- Problem Discription : 
    - **N processes** compete to **use shared data**
    - Each process has a **critical section** - 每個 process 都有一段程式碼，會「**讀 / 寫共享資料**」
    - Challenge（挑戰）: 當某個 process 正在 critical section ， 其他 process 不能 同時進入它們的 critical section
    - 這個性質叫 **mutual exclusion（互斥）**

## Requirements for a Valid Solution
1. Mutual Exclusion(互斥性):
    - 同一時間最多一個 process 在 critical section
    - 目的 : 防止 **Race condition** 、 確保資料完整性
2. Progress (進展性):
    - 如果：critical section 是空的，有 process 想進去
    - 那麼：**不能無限期拖延**「誰可以進去」的決定
    - 決定一定要在 **有限時間內完成**
    - 目的 : 防止 **DeadLock**
3. Bounded Waiting(有界等待):
 - 一個 process 請求進入後 ， 其他 process 最多只能插隊有限次
 - 目的 : 防止 **starvation**（飢餓）

## Critical Section Solutions – Software Solutions
1. Software Solutions
    - Characteristics:
        - Use only softwarealgorithms
        - 在任何系統都能用

    - Example: 
        - Peterson’s Algorithm（2 個 process）
        - Dekker’s Algorithm（2 個）
        - Bakery Algorithm（N 個）
2. Synchronization Hardware（硬體支援）
    - Characteristics:
        - Relies on **atomic hardware instructions**
        > atomic hardware instructions（原子硬體指令） : 
        > 是由 CPU 保證「整個操作不可被中斷、不可被拆開」的一條指令（或指令組）。
        > 要嘛 全部做完，要嘛 完全沒做
    - Hardware Support
        - Test-and-Set
        - Compare-and-Swap (CAS)
        - Atomic variables
        > 這些保證「一整個操作不可被切割」
3. Semaphore
    - Characteristics:
        - 一個 **整數變數**，透過**兩個原子操作(atomic operation)** 存取
        - 提供By OS
        - More abstract than hardware instructions
    - Operations:
        - wait() / P() - dercrement and block : 會嘗試將 semaphore 減 1 並 阻塞。
        - signal() / V() - increment and wake up wating process : 會將 semaphore 加 1 並喚醒等待者。
    - Types:
        - Binary semaphore(0 / 1) - 類似 mutex
        - Counting semaphore(0 ~ N) - resource counting
        > Mutex（互斥鎖）: Mutex 保證「同一時間只有一個 thread 能進 critical section」
    - Mutual exclusion
4. Monitor
    - Characteristics:
        - 高階抽象（High-level abstraction）
        - 封裝共享資料與程序 - 共享變數不能被外面直接存取
        - 自動保證 Mutual Exclusion - 同一時間，只能一個 process / thread 在 monitor 裡
            - 進入 Monitor → 自動上鎖
            - 離開 Monitor → 自動解鎖
        - Synchronization : 使用 condition variable，使用 wait()、signal()


    - Features : 
        - Only one process can be active
        - Condition Variables（條件變數）for waiting and signaling
        - Compiler enforces synchronization rules

## Comparison

| 方法          | 抽象層級  | 複雜度   | 主要問題            |
| ----------- | ----- | ----- | --------------- |
| Software    | 低     | 高     | 複雜、busy waiting |
| Hardware    | 低     | 中     | 只是基礎積木          |
| Semaphore   | 中     | 中     | **很容易誤用**       |
| **Monitor** | **高** | **低** | 需要語言/編譯器支援      |

- Key Points
    - Hardware instructions 是一切的基礎
    - Semaphore 與 Monitor 是不同抽象建於硬體之上


# Software Solutions

## Algorithm for two Processes

- Setup : Two Processes(P0 and P1)
- Shared Variables : ```int turn```;
    - $ turn_i $ 代表 輪到 Pi 可以進 critical section;
- How It Work : Processes **Alternate** entering the critical section
    - Process P0:
        - 一直等到 turn == 0 才進去 ， 出來後把 turn = 1
    - Process P1:
        - 一直等到 turn == 1 才進去出來後把 turn = 0
- Analysis:
    - Mutual Exclusion : ✅
    - Progress : ❌（強制輪流），即使 critical section 是空的也不能進
    - Bound Waiting : ✅

## Peterson’s Solution

- 不只看「輪到誰」，還看**對方想不想進**
- Shared variables
    - Boolean **Flag**[2]
        - ```flag[i] = true```：Pi 想進 CS（有意圖）
        - ```flag[i] = false```：Pi 不想進
    - int **turn**
        - 用來在「兩個都想進」時決定誰先
- How It Work :
    1. ```flag[i] = true;``` → 宣告：「我想進」
    2. ```turn = j;``` → 禮讓：「如果你也想進，那先讓你」
    3. ```while (flag[j] && turn == j) ;``` → 只有在「**對方想進**」而且「**輪到對方**」時才等
- Key : 
    - 如果兩個同時想進:
        - Both set their flag to **ture**
        - **Last Process** to set **turn** will wait
- Analysis:
    - Mutual Exclusion : ✅
    - Progress : ✅
    - Bound Waiting : ✅

- Cons : 
    - Busy waiting（一直 while 空轉浪費 CPU）
    - 只適用 2 個 process

### Correctness Proof of Peterson’s Solution
- Mutual Exclusion : Contradiction(矛盾)
    - 假設一個不可能的事 => P0、P1同時在 critical section
    - flag[0]、flag[1] = true ， 只能靠turn來決定誰能進
    - 但turn不能同時是 0 or 1 => 矛盾！
- Progress :
    - Scenario 1：只有 P0 想進
        - ```flag[1]=false```
        - P0 while：```flag[1] && turn==1 → false```
        - ✅ P0 立刻進 → 不會被無限拖延 
    - Scenario 2：兩個都想進
        - ```Flag[0] Flag[1] = true;```
        - 決定只看turn，而turn 只能是 0 or 1，所以只有一個人會進 ✅
- Bounded waiting :
    - Case 1：P1 退出後不再回來
        - P1 出來會 ```flag[1]=false```
        - P0 的 while 立刻變 false
        - ✅ P0 直接進
    - Case 2：P1 退出後又想再進
        - P1 再次想進時會：
            - ```flag[1]=true```
            - 並且 設 ```turn=0```（禮讓 P0）
            - 這會覆蓋掉 P0 可能設的 ```turn=1```
            - 此時對 P0 而言：
                - while：```flag[1] && turn==1```
                - 因為 turn 已經變 0 → false ✅ → P0 進
    - **P0 請求進入後，P1 最多只能進一次，P0 不會無限等待。**

### Turn Algorithm V.S. Peterson's Algorithm

| Feature（特性）               | Turn Algorithm（Strict Alternation） | Peterson’s Algorithm      |
| ------------------------- | ---------------------------------- | ------------------------- |
| **Mutual Exclusion（互斥）**  | ✓ 滿足                               | ✓ 滿足                      |
| **Progress（進展性）**         | ✗ **不滿足**<br>（對方不想進也會被卡）           | ✓ 滿足                      |
| **Bounded Waiting（有界等待）** | ✓ 滿足                               | ✓ 滿足                      |
| **Flexibility（彈性）**       | Low（只看 turn）                       | High（看 intent + priority） |


| 項目    | Turn Algorithm  | Peterson’s Algorithm     |
| ----- | --------------- | ------------------------ |
| 使用的變數 | 只有 `turn`       | `flag[]` + `turn`        |
| 解決的問題 | 只保證互斥           | 同時保證互斥、進展、公平             |
| 最大缺陷  | **違反 Progress** | 無（理論上正確）                 |
| 核心概念  | 嚴格輪流            | 意圖（intent）＋優先權（priority） |

## Dekker’s Algorithm

- 使用的共享變數
- 第一個「完全正確」的純軟體 critical section 解法（1960s）
    
| 變數        | 意義                         |
| --------- | -------------------------- |
| `flag[i]` | Pi 是否「想進」 critical section |
| `turn`    | 當兩個都想進時，誰優先                |

- Dekker 的 Entry Section（重點流程表）
- 假設 Pi 想進，Pj 是對方：

| 步驟 | 程式動作                | 白話解釋         |
| -- | ------------------- | ------------ |
| 1  | `flag[i] = true`    | 我想進          |
| 2  | `while (flag[j])`   | 如果對方也想進      |
| 3  | `if (turn == j)`    | 而且現在輪到他      |
| 4  | `flag[i] = false`   | 那我先退讓（暫時不想進） |
| 5  | `while (turn == j)` | 等對方用完        |
| 6  | `flag[i] = true`    | 再次宣告我想進      |
| 7  | 進 critical section  | 條件滿足才進       |

- Dekker 最特別的地方 
    → 會「暫時撤回意圖（withdraw interest）」

### Dekker vs Peterson

| Feature        | Dekker’s Algorithm | Peterson’s Algorithm |
| -------------- | ------------------ | -------------------- |
| 歷史             | 第一個正確解（1960s）      | 後來提出      |
| Entry protocol | 退讓＋重新宣告       | 宣告一次即可     |
| 複雜度            | ❌ 複雜          | ✅ 簡潔         |
| Correctness    | ✓                  | ✓                    |

- 關鍵差異

| 比較點      | Dekker           | Peterson |
| -------- | ---------------- | -------- |
| 等待時的行為   | **會暫時放棄意圖**      | 一直保持想進   |
| flag 的使用 | 可能變 false 再 true | 一次設 true |
| 可讀性      | 差                | 好        |

### Limitations of Dekker's Algorithm

| 缺點              | 說明                     |
| --------------- | ---------------------- |
| 太複雜             | entry protocol 很難寫對    |
| Busy waiting    | 浪費 CPU                 |
| 只適用 2 processes | 無法擴充                   |
| 記憶體排序要求高        | 現代 CPU 可能壞掉            |
| 實務不使用           | 被 mutex / semaphore 取代 |

## Bakery Algorithm
- 第一個能解「**N 個 processes**」的純軟體 critical section 解法
- 核心比喻

| 現實世界    | Bakery Algorithm          |
| ------- | ------------------------- |
| 客人抽號碼   | Process 取 ticket          |
| 號碼小的先服務 | ticket 小的先進 CS            |
| 號碼一樣比先後 | 用 process ID 當 tiebreaker |
| 公平、不插隊  | 無 starvation              |

- Numbering scheme :
    - Generate numbers in non-decresing order (後一個數字 <= 前一個數字)
    - Duplicate numbers are possible!
- Tie-breaking rule:
    - Use **Process ID** as the tiebreaker

### 排序規則：Lexicographic Ordering (字典順序)
- Bakery 比較的是 **(number, process_id)**
- Definition:
    $$
        (a,b) < (c,d)　if　a < c　OR　(a==c　AND　b<d)
    $$
- 先比號碼 ， 號碼一樣 → 比 process ID

- 共享變數

| 變數            | 意義               |
| ------------- | ---------------- |
| `number[i]`   | Pi 的號碼牌（0 = 不想進） |
| `choosing[i]` | Pi 是否正在選號碼       |

### Bakery Entry Section
- Step 1：拿號碼（Get Ticket）

| 程式碼                             | 目的          |
| ------------------------------- | ----------- |
| `choosing[i] = true`            | 告訴大家：我在選號碼  |
| `number[i] = max(number[]) + 1` | 拿一個比目前最大還大的 |
| `choosing[i] = false`           | 我選完了        |

- Step 2：等輪到我（Wait for Turn）

| 等待條件                                                        | 意義            |
| ----------------------------------------------------------- | ------------- |
| `while (choosing[j])`                                       | 等對方選完號碼       |
| `while (number[j] != 0 && (number[j], j) < (number[i], i))` | 如果對方號碼比較小 → 等 |

- Step 3：進入 CS
    - 當「所有優先權比我高的都完成」→ 我就能進

- Exit Section（Release Ticket）

| 程式碼             | 意義        |
| --------------- | --------- |
| `number[i] = 0` | 退還號碼，不想進了 |

### 為什麼一定要 choosing[]?
- 沒有 choosing[]

| 步驟 | 發生的事                  |
| -- | --------------------- |
| 1  | P0 讀 number → max=0   |
| 2  | context switch        |
| 3  | P1 讀 number → max=0   |
| 4  | P1 設 number=1，進 CS    |
| 5  | context switch        |
| 6  | P0 設 number=1，也進 CS ❌ |

- 兩個人拿到一樣的號碼，還同時進去

- 加上 choosing[] 後

| 步驟               | 改善          |
| ---------------- | ----------- |
| P0 choosing=true | P1 看到，先等    |
| P0 完成 choosing   | number[0]=1 |
| P1 再選            | 拿到 number=2 |

### Correctness Proof

1. Mutual Exclusion

- 假設 P0、P1 同時在 CS：

| 推論      | 結果                            |
| ------- | ----------------------------- |
| P0 通過等待 | (number[1],1) ≥ (number[0],0) |
| P1 通過等待 | (number[0],0) ≥ (number[1],1) |
| 合併      | 兩者互相小於 ❌                      |
| 結論      | **字典序不可能 → 矛盾**               |

2. Progress

| 理由        | 說明                 |
| --------- | ------------------ |
| 有限 number | 一定存在最小 (number,id) |
| 最小者       | 不會再等任何人            |
| 結果        | 一定能進（finite time）  |

3. Bounded Waiting

| 性質   | 說明                |
| ----  | ----------------- |
| FIFO  | 依 ticket 排隊       |
| 上限   | Pi 拿號後，最多 n−1 人先進 |
| 結果   | 無 starvation      |

### Bekery's algorithm優缺點總表

- ✅ 優點

| 項目             | 說明                       |
| -------------- | ------------------------ |
| 支援 N processes | ✔                        |
| 公平             | First-come, first-served |
| 無 starvation   | ✔                        |
| 純軟體            | 不靠硬體                     |

- ❌ 缺點

| 項目           | 說明                   |
| ------------ | -------------------- |
| Busy waiting | 浪費 CPU               |
| Entry O(n)   | 要檢查所有 process        |
| Space O(n)   | number[], choosing[] |
| 不實用          | 現代系統不用               |

### 前面的總結

| Algorithm  | Processes | Mutual | Progress | Bounded | 實用性 |
| ---------- | --------- | ------ | -------- | ------- | --- |
| Turn       | 2         | ✓      | ✗        | ✓       | ❌   |
| Peterson   | 2         | ✓      | ✓        | ✓       | 教學  |
| Dekker     | 2         | ✓      | ✓        | ✓       | 歷史  |
| **Bakery** | **N**     | ✓      | ✓        | ✓       | 理論  |


## Pthread Lock/Mutex Routines

- 在 Unix/Linux 用 POSIX Threads（pthread）寫多執行緒時，最常用的互斥工具就是 mutex
    - 保證同一時間只有一個 thread 進入 critical section（共享資料區） = Mutual Exclusion（互斥）
> Mutex（互斥鎖）: Mutex 保證「同一時間只有一個 thread 能進 critical section」
🔒 mutex = 鎖
🚶 thread = 人

- Pthread Mutex 的基本流程
    ```
    pthread_mutex_t mutex;

    // 1. 初始化
    pthread_mutex_init(&mutex, NULL);

    // 2. 上鎖（進 critical section）
    pthread_mutex_lock(&mutex);

    /* CRITICAL SECTION
    存取 shared data
    */

    // 3. 解鎖（離開 critical section）
    pthread_mutex_unlock(&mutex);

    // 4. 不用時銷毀
    pthread_mutex_destroy(&mutex);

    ```

## Condition Variables（CV）

- CV:
    - Thread 想要等「某個條件成立」再繼續做事
    - Notify other waiting thread that the condition has occured
- 三個核心操作:
    - wait() : 讓呼叫者「睡著」，直到有人通知它 、 wait 會原子性地「釋放 mutex 並進入等待
    - signal() : 喚醒 一個 正在 wait 的 thread 、 如果沒人 wait：訊號會消失
    - broadcast() : 喚醒 全部 wait 的 threads、大家會搶 mutex ， 仍是一次一個能真正繼續


# Hardware Support for Synchronization

## 純軟體的限制
- ❌　共享變數的修改會被「中斷」可能被 → context switch 打斷
- ❌　多步驟操作不是 atomic → ```load → modify → store``` 不是不可分割的
- ❌　純軟體解法太依賴「嚴格記憶體順序」

- **需要硬體支援的 atomic operations（原子操作）**

## Approach 1: Disable Interrupts

- Logic : 不讓中斷發生 → 就不會被 context switch → critical section 安全

- 單核心問題 :
    - ⛔ **clock interrupt 被關** → 系統時間亂掉
    - ⛔ **I/O interrupt 被吃掉** → 裝置可能壞掉
    - ⛔ **安全風險**：user process 若能關中斷，系統直接爆炸

- 多核心問題 : 
    - ⛔ 關的是「本 CPU」的中斷 → 其他 CPU 還是能同時存取 shared data

- **❌ 關中斷 不是通用解法**

## Approach 2: Hardware Atomic Instructions

- Atomic Instruction : 
    - **一次完成**
    - **不可被中斷**
    - CPU 硬體保證

- Common Atomic Instruction : 
    1. Test-and-Set
        - 讀＋寫一次完成
    2. Compare-and-Swap (CAS)
        - 只有在「值沒變」時才更新
    3. Swap / Exchange
        - 直接交換兩個值

## Mutual Exclusion 3 Common Atomic Instruction

| 項目               | **Test-and-Set**      | **Compare-and-Swap (CAS)** | **Swap / Exchange**   |
| ---------------- | --------------------- | -------------------------- | --------------------- |
| 原子指令功能           | 讀舊值並設為 true           | 比較後才交換                     | 直接交換兩個值               |
| 是否硬體原子操作         | ✅                     | ✅                          | ✅                     |
| 共享變數             | `boolean lock`        | `boolean lock`             | `boolean lock`        |
| 輔助變數             | 無                     | 無                          | `boolean key`         |
| lock 初始值         | `false`               | `false`                    | `false`               |
| 進入臨界區條件          | TestAndSet 回傳 `false` | CAS 成功（false → true）       | Swap 後 `key == false` |
| 失敗時行為            | 一直 spin               | 一直 spin                    | 一直 spin               |
| 等待方式             | Busy waiting (spin)   | Busy waiting (spin)        | Busy waiting (spin)   |
| 解鎖方式             | `lock = false`        | `lock = false`             | `lock = false`        |
| Mutual Exclusion | ✅                     | ✅                          | ✅                     |
| Progress         | ✅                     | ✅                          | ✅                     |
| Bounded Waiting  | ❌                     | ❌                          | ❌                     |
| 公平性              | ❌                     | ❌                          | ❌                     |
| CPU 使用效率         | 低（浪費 CPU）             | 低                          | 低                     |
| 現代實務使用           | 基本 spinlock           | ⭐ 最常用（lock-free）           | 少（舊系統）                |

- Key Points:
    -   硬體支援讓同步變簡單又快
    - Atomic instructions 是 mutex 、 semaphore的地基
    - busy waiting 仍存在 、 bounded waiting 仍需設計

# Semaphore
- Purpose（目的）
    - 它是一個「**同步工具**」，用來一般化 critical section 問題
    - 比硬體原子指令**更高階**

- Definition
    - 本質是一個 整數 counter
    - 存取只能透過兩**個原子操作** : wait() 、 signal()
    - Key property：Operations are **indivisible**（不可分割/原子）
    - **wait/signal 一定要 atomic**
## Type of Semaphore
- Binary Semaphore（值只有 0 或 1）
    - value=1：資源可用 / 沒鎖（unlocked）
    - value=0：資源不可用 / 上鎖（locked）
    - 用途：互斥（mutex lock）（一次只能一個人進 CS）

- Counting Semaphore（值是 2…N）
    - 代表「同時允許最多 N 個」使用資源
    - 用途：資源數量控制

### Two Atomic Operation :
- wait(S) / P(S) / down(S)
    - 做的事：S--
    - 如果 S 變成負數 → 表示「資源不夠」→ 呼叫者要 block（睡眠）

- signal(S) / V(S) / up(S)
    - 做的事：S++
    - 如果有人在等 → wakeup 一個

- Spinlock Implementation : 
    - wait(S)：當 S->value <= 0 就一直 while 轉圈（busy waiting），直到有資源再 value--
    - signal(S)：value++

### Issue
- **浪費 CPU cycles**：一直 while 檢查
- **單核心更糟**：等的人一直占 CPU，持鎖者可能根本拿不到 CPU 去解鎖

### 適合時機
- critical section 很短（一下就解鎖）
- 多核心 multiprocessor：別的 CPU 仍能前進（持鎖者能跑）

### 不適合時機
- CS 很長
- CPU 資源少
- 單核心

## Non-busy waiting（有等待隊列的 semaphore，真正 OS 會用）

- 不要忙等，改成 block/wakeup

- wait(S)：先 S->value--
    - 如果變成 < 0：代表要等
    → 把自己放到等待隊列 S->list
    → 呼叫 block()（睡眠，OS 把你移出 ready queue）

- signal(S)：先 S->value++
    - 如果結果 <= 0：表示「還有人在等」（因為負值代表等待人數）
    → 從 S->list 取出一個
    → wakeup(P) 把它放回 ready queue

- Value意義:
    - S > 0：還有 S 個資源可用（下一個 wait 不會 block）
    - S = 0：資源剛好用完（下一個 wait 會 block）
    - S < 0：
         - |S| = 正在等待（被 block）的 process 數量
    例：S = -3 代表有 3 個 process 被擋在 wait 上

## POSIX Semaphor
- Semaphore 是 **POSIX 標準**的一部分 ， **不是 pthread 的一部分**
- Semaphore 可以用在：
    - process ↔ process（跨行程同步）
    - thread ↔ thread（同一行程內多執行緒同步）

##　Correctness Properties
- Mutual Exclusion：可以
- Progress：通常可以
- Bounded Waiting：不保證
    - 會不會餓死starvation取決於等待隊列怎麼選人
        - FIFO：比較有 bounded waiting
        - LIFO / priority：可能有人永遠被插隊 → starvation

## Advantages and Disadvantages
- 優點:
    - 抽象層更高、可移植、不依賴特定硬體、可用 block 避免 busy waiting
    - 也支援「互斥」+「同步（順序約束）」兩種用途
- 缺點:
    - 容易寫錯
    - 不具結構性
    - 沒有 ownership（所有權）

## Cooperation Synchronization（合作式同步：強制順序）
- 需求
    - P1 做 S1
    - P2 做 S2
    - 規定：S2 必須等 S1 做完

- 看到 wait(x)：代表「我需要 x 這個事件先完成」
- 看到 signal(x)：代表「我完成了某個事件，放行後面的人」

## Deadlocks and Starvation
- Deadlock
    - 兩個或更多 process 都在等
        - 而「能讓我繼續的事件」只能由「正在等的那些人」造成
    → 形成閉環，永遠沒人能先動

- Starvation
    - 某個 process 一直等不到，但系統其實一直在前進
    - 常見原因：
        - semaphore 等待隊列是 LIFO 或不公平策略
        → 新來的永遠插隊，老的永遠輪不到。

# Classical Synchronization Problems

- Purpose
    - **Benchmark problems** : 把同步問題「標準化」
    - 都要拿來「實戰驗證」
    - 光說「我這個 lock 很棒」不夠，要拿**經典問題來測**

- 三大經典問題

| 問題                                | 重點                               |
| --------------------------------- | -------------------------------- |
| Producer–Consumer（Bounded Buffer） | **容量限制 + 同步**                    |
| Readers–Writers                   | **讀寫衝突 + 優先權**                   |
| Dining Philosophers               | **資源共享 + deadlock / starvation** |

- Producer–Consumer vs Readers–Writers vs Dining Philosophers    

| 項目                 | **Bounded-Buffer** | **Readers–Writers** | **Dining Philosophers** |
| ------------------ | --------------------------------------------------- | ----------------------------- | --------------------------------- |
| **問題類型**           | 生產 / 消費                                             | 讀 / 寫                         | 資源競爭                              |
| **共享資源**           | 有限個 buffer（n 個）                                     | 共享資料（DB / file）               | 筷子（資源）                            |
| **角色**             | Producer、Consumer                                   | Reader、Writer                 | Philosopher                       |
| **資源數量**           | 有限（n）                                               | 1 份資料                         | 每人左右各 1 支                         |
| **核心限制**           | 有容量上限                                               | Writer 必須獨佔                   | 一次要拿 2 支                          |
| **允許並行**           | 多個 Producer / Consumer                              | 多個 Reader 可同時                 | 多人思考                              |
| **不允許並行**          | 同一 buffer 同時使用                                      | Writer 與任何人                   | 同一筷子被兩人拿                          |
| **主要同步目的**         | ✔ 不超生產<br>✔ 不空消費                                    | ✔ 保護資料一致性                     | ✔ 避免系統卡死                          |
| **典型錯誤**           | 生產太多或消費太快                                           | 優先權設計錯誤                       | 拿筷子順序錯                            |
| **Deadlock 可能？**   | ❌（正常設計不會）                                           | ❌（除非設計很爛）                     | ✔ **非常典型**                        |
| **Starvation 可能？** | ❌                                                   | ✔（Readers 或 Writers）          | ✔                                 |

# Monitors

- Motivation of Monitors
    - Semaphore 很有效，但「**正確性完全靠程式員**」

- Monitor 是更高階的同步結構
- 核心功能：**自動互斥（automatic mutual exclusion）**
- Monitor 由兩部分構成：
    - 共享資料（variables / state）
    - 操作共享資料的方法（procedures/functions）
        而且外面只能透過方法存取資料。
- 同一時間只允許**一個** thread/process 在 monitor 裡執行
→ 這就是 monitor **自帶的 mutual exclusion**。

## Similarity to Object-Oriented Classes

- Monitor 很像物件導向 class：
    - 封裝（Encapsulation）：資料 + 操作放一起
    - 資訊隱藏（Data hiding）：內部狀態 private，外面不能直接碰
    - 公開介面（Public interface）：外界只能呼叫方法

- 外部 code **不能直接存取** monitor 內部變數

## Condition Variables in Monitor

- Monitor **只保證 mutual exclusion**，不保證你要的「條件同步」。
- 很多問題需要「等到條件成立」才繼續，例如 bounded buffer：
    - buffer 滿：producer 不能做 → 要等 notFull
    - buffer 空：consumer 不能拿 → 要等 notEmpty

-  monitor 提供 **condition variables**
    - x.wait()
        - 把自己睡著（suspend）
        - 釋放 monitor lock（不然別人進不來改條件）
        - 等被喚醒後，再回來繼續
    - x.signal()
        - 喚醒 一個 等在 x 的人
        - 若沒人等 → 沒效果（signal lost）

---
> 作者最後的感想 : 我知道又臭又長，但我真的不知道要怎麼縮減了，可能作業系統就是這樣吧...! <3
> 因為時間緊急，我下次會記得美化的 可能會請求GPT的支援吧