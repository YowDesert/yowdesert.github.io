---
title: (作業系統) Deadlock Definition, Avoidance and Resolution
categories:
  - 作業系統
  - Process and Concurrency
tags:
  - 作業系統
abbrlink: 67c68839
mathjax: true
date: '2025-12-13 16:15:00'
---

# Deadlock基本概念

## 1. Deadlock 是什麼？
**Deadlock** 指的是：
> 一組 processes 彼此等待對方所持有的資源，導致所有 process 都無法繼續執行。

### Deadlock 的特徵
- 每個 process：
  - 已持有至少一個資源
  - 同時等待另一個 process 持有的資源
- 結果：大家互等，系統停住

---

## 2. Deadlock 範例

### Example 1：兩個 processes、兩個資源
- P1 持有 R1，等待 R2
- P2 持有 R2，等待 R1  
→ **互相等待，發生 deadlock**

### Example 2：Semaphore A 與 B
```c
P1: wait(A); wait(B); ...; signal(B); signal(A);
P2: wait(B); wait(A); ...; signal(A); signal(B);
```
- 若 P1 先拿到 A、P2 先拿到 B
- 下一步彼此都在等對方 → **deadlock**

---

# Deadlock 的四個必要條件

Deadlock 當且僅當下列四個條件同時成立：

1. **Mutual Exclusion（互斥）**
2. **Hold and Wait（占有且等待）**
3. **No Preemption（不可搶占）**
4. **Circular Wait（循環等待）**

📌 破壞其中一個條件即可避免 deadlock。

---

# System Model（系統資源模型）

## Resource Types 與 Instances
- 資源種類：R1, R2, ...
- 每種資源有多個 instances（例：2 CPUs）

## Resource Usage Pattern
1. Request
2. Use
3. Release

---

# Resource-Allocation Graph（RAG）

## 圖的元素
- Process：圓形
- Resource Type：矩形
- Instance：矩形內的點

## 邊的種類
- Request edge：P → R
- Assignment edge：R → P

![alt text](/img/RAG.png)

---

# Cycle 與 Deadlock 關係

- 單一 instance：Cycle ⇒ Deadlock
- 多 instances：Cycle 為必要但非充分

---

# Deadlock Detection

- 無 cycle：無 deadlock
- 有 cycle：
  - 單一 instance ⇒ deadlock
  - 多 instances ⇒ 需進一步分析
- P2、P4不在Cycle，不會DeadLock

![alt text](/img/NotHaveDeadlock.png)

- 會DeadLock

![alt text](/img/HaveDeadLock.png)

---

# Deadlock 處理策略

1. **Prevention / Avoidance**
2. **Detection & Recovery**
3. **Ignore（Ostrich Algorithm）**


---


# Deadlock Prevention

Deadlock Prevention 的核心概念是：  
👉 **確保四個必要條件中，至少有一個「不成立」**，使 deadlock 不可能發生。

---

## 1. Preventing Mutual Exclusion（破壞互斥）(做不到) 

### 方法
- 允許資源共享（只要可行）
- 對「可共享資源」不強制 mutual exclusion

### 範例
- Read-only files 可被多個 processes 同時存取

### 限制
- 許多資源本質上不可共享：
  - Printers
  - Tape drives
  - File write access

### 結論
- 適用範圍有限
- **不是通用解法**



## 2. Preventing Hold and Wait（破壞占有且等待）(有效率但沒效能)

### 核心想法
- 不允許 process 在「持有資源」時再等待其他資源

### Protocol 1：Request all at once (全拿or全不拿)
- Process 必須在開始前一次請求所有所需資源
- 未取得全部資源前，不得持有任何資源

### Protocol 2：Release before request (用前先Release)
- Process 只有在「未持有任何資源」時才能請求新資源
- 若需新資源，必須先釋放目前全部資源

### 優點
- 能有效防止 deadlock

### 缺點
- 低資源使用率
- 可能產生 starvation (要很多Process 要 等待所有的Process)
- 程式不一定事先知道所需資源



## 3. Preventing No Preemption（破壞不可搶占）(可以使用,實作有限,效能損失)

### 核心想法
- 允許系統強制回收（preempt）資源

### Protocol 1：Preempt waiting process
- 若 process 持有資源且請求新資源失敗：
  - 回收該 process 所有資源
  - 該 process 重新等待舊資源與新資源
- 你既然要等，那你先把你手上的資源通通吐出來

### Protocol 2：Preempt holder of requested resource
- 若 P1 請求的資源 R 被 P2 持有：
  - 若 P2 正在等待其他資源 → preempt R
     - 表示 P2 自己也卡住了 → 可以把 R 從 P2 手上搶走給 P1
  - 否則 P1 等待
     - 表示 P2 很可能快做完、快釋放 ， 讓 P1 等，不要搶

### 適用資源
- whose state can be easily saved and restored
✅ CPU registers  
✅ Memory（swap）  
✅ Database locks（rollback）  

- whose state cannot be easily saved and restored
❌ Printers  (cannot undo)
❌ Tape drives  
❌ CD/DVD burners  



## 4. Preventing Circular Wait（破壞循環等待）

### 方法：資源排序（Total Ordering）
- 為每種資源指定唯一編號 F(Ri)
- Process 必須依照編號遞增順序請求資源

### Resource Request Rule
- 若 process 已持有 Ri
- 只能請求滿足 F(Rj) > F(Ri) 的資源
- 要 2,4 ，需要先拿2再拿4

### Alternative Protocol
- 請求 Rk 前，先釋放所有 F(Ri) ≥ F(Rk) 的資源

> F(R0) 不可能 < F(R0)

### 優點
- 有效防止 circular wait
- Runtime overhead 低

### 缺點
- 小的號碼很搶手
- 排序設計困難
- 可能限制資源請求彈性

---

# Deadlock Avoidance（死結避免）

## 核心概念
- 系統在分配資源前進行檢查
- **確保永遠不進入 unsafe state**



## Algorithm Selection

### Case 1：Single Instance per Resource Type
- 使用 Resource-Allocation Graph（RAG）
- 檢查是否產生 cycle

### Case 2：Multiple Instances per Resource Type
- 使用 Banker’s Algorithm
- 檢查是否存在 safe sequence



## Safe State（安全狀態）

- **Safe state**：存在某種資源配置順序，使所有 process 可完成
- Safe ⇒ 一定不會 deadlock
- Unsafe ⇒ deadlock 可能發生（但不一定）

### Avoidance 的目標
👉 **系統永遠不進入 unsafe state**

---


# Resource-Allocation Graph (RAG) Algorithm

## 一、RAG Algorithm 在做什麼？
RAG Algorithm 是 **Deadlock Avoidance** 的方法之一，  
適用於 **每種資源只有一個 instance（single instance per resource type）** 的系統。

👉 核心想法：
> **在資源真正分配前，先檢查是否會形成 cycle，避免系統進入 deadlock。**

---

## 二、Graph Components（圖的邊種類）

### 1. Request Edge（實線）
`Pi → Rj`
- Process Pi 正在請求資源 Rj
- 表示目前尚未被滿足的請求



### 2. Assignment Edge（實線）
`Rj → Pi`
- 資源 Rj 已分配給 Process Pi
- 表示 Pi 正在持有並使用該資源



### 3. Claim Edge（虛線）
`Pi ⇢ Rj`
- Process Pi **未來可能會請求** 資源 Rj
- 表示潛在的最大需求

📌 重要規定：
- Claim edge **必須在 process 執行前就宣告**
- 系統必須事先知道 process 的最大資源需求

---

## 三、Edge Transitions（邊的轉換）

### Transition 1：Claim → Request
- 當 Pi 實際開始請求 Rj
- 虛線 claim edge 轉為實線 request edge

### Transition 2：Request → Assignment
- 當系統核准請求並分配資源
- request edge 轉為 assignment edge

### Transition 3：Assignment → Claim
- 當 Pi 釋放資源 Rj
- assignment edge 轉回 claim edge

![alt text](/img/RAGAlgo.png)

---

## 四、Deadlock Avoidance Protocol（RAG 演算法流程）

### 前提條件
- 所有 claim edges 必須在 process 開始前建立



### 當 Process Pi 請求資源 Rj 時：

1. **假設分配**
   - 將 claim → request → assignment（假想）

2. **Cycle 檢查**
   - 使用 cycle-detection algorithm（時間複雜度 O(n²)）

3. **判斷結果**
   - ✅ 若 **不產生 cycle**：允許分配（safe）
   - ❌ 若 **會產生 cycle**：拒絕請求，process 等待（unsafe）

---

## 五、Key Insight（關鍵觀念）

在 **single instance per resource type** 的系統中：

> **Cycle ⇔ Deadlock**

因此：
- 防止 cycle
- 就能保證系統不會發生 deadlock

---

## 六、為什麼資源「現在空著」也不能給？

即使某資源目前未被使用：
- 若分配後會形成 cycle
- 在單一 instance 系統中就等同於 deadlock

👉 **RAG Algorithm 是預防未來死結，而不是只看當下。**

---


# Deadlock Avoidance：Safe State Concept

## 一、核心概念：Safe vs. Unsafe
Deadlock Avoidance 的根本思想在於區分 **Safe state** 與 **Unsafe state**。

### Safe State（安全狀態）
系統若存在一個 process 執行順序：
\<P1, P2, …, Pn\>，使得對每個 Pi：
- Pi 尚需的資源 ≤（目前可用資源 + 所有 Pj（j < i）完成後釋放的資源）
- 若 Pi 無法立即取得資源，則等待前面的 Pj 完成
- Pi 完成後會釋放其資源，供後續 process 使用

👉 此順序稱為 **Safe Sequence**


### Safe / Unsafe / Deadlock 關係（必考）
- **Safe state ⇒ 一定不會 deadlock**
- **Unsafe state ⇒ 可能 deadlock（但不一定）**
- **Deadlock ⇒ 一定是 unsafe state**

📌 Deadlock Avoidance 的策略：  
👉 **確保系統隨時維持在 safe state**

---

# Safe State with Safe Sequence（範例解析）

## 系統設定
- 總共有 **12 台 tape drives**
- Process 與需求如下：

| Process | Max Needs | Current Holding |
|------|-----------|-----------------|
| P0 | 10 | 5 |
| P1 | 4 | 2 |
| P2 | 9 | 2 |

目前 Available = 12 − (5 + 2 + 2) = **3**

---

## 判斷 Safe Sequence：⟨P1, P0, P2⟩

### Step 1：P1
- Need(P1) = 4 − 2 = 2
- Available = 3 ≥ 2 ✅
- P1 完成後釋放 2  
→ Available = 3 + 2 = **5**



### Step 2：P0
- Need(P0) = 10 − 5 = 5
- Available = 5 ≥ 5 ✅
- P0 完成後釋放 5  
→ Available = 5 + 5 = **10**



### Step 3：P2
- Need(P2) = 9 − 2 = 7
- Available = 10 ≥ 7 ✅
- P2 完成後釋放 2  
→ Available = **12**



### 結論
- 存在 Safe Sequence：⟨P1, P0, P2⟩
- 👉 **系統處於 Safe State**

---

# Unsafe State（沒有 Safe Sequence）

## 系統狀態（t1）
| Process | Max Needs | Current Holding |
|------|-----------|-----------------|
| P0 | 10 | 5 |
| P1 | 4 | 2 |
| P2 | 9 | 3 |

Available = 12 − (5 + 2 + 3) = **2**

---

## 分析
- Need(P0) = 5 > 2 ❌
- Need(P1) = 2 ≤ 2（但釋放後仍不足以滿足其他 process）
- Need(P2) = 6 > 2 ❌

👉 **不存在任何 Safe Sequence**



### 結論
- 系統進入 **Unsafe State**
- Deadlock **可能發生（但尚未發生）**

📌 Deadlock Avoidance 原則：  
👉 **只有在資源分配後仍為 Safe State，請求才會被允許**

---


# Banker’s Algorithm

## 一、用途與適用情境
- **Deadlock Avoidance（死結避免）** 演算法
- **適用於：每種資源有「多個 instances」的系統**
- 核心原則：
  > **只有在分配後仍然存在 Safe Sequence，才允許資源分配**



## 二、資料結構（一定要會）

對每個 process Pi 與每種資源型別（A, B, C…）：

- **Max[i]**：Pi 宣告的最大需求量
- **Allocation[i]**：目前已分配給 Pi 的資源量
- **Need[i] = Max[i] − Allocation[i]**
- **Available**：系統目前可用的資源量



## 三、Safety Algorithm（安全性檢查）

### 目標
判斷「目前狀態是否為 Safe State」，並找出 **Safe Sequence（若存在）**。

### 步驟
1. **初始化**
   - `Work = Available`
   - `Finish[i] = false`（對所有 i）

2. **尋找可完成的 process**
   - 找一個 `Finish[i] == false` 且 `Need[i] ≤ Work` 的 Pi
   - 若找不到 → 跳到步驟 4

3. **模擬完成**
   - `Work = Work + Allocation[i]`
   - `Finish[i] = true`
   - 記錄 Pi 到 Safe Sequence
   - 回到步驟 2

4. **判斷**
   - 若所有 `Finish[i] == true` → **Safe**
   - 否則 → **Unsafe**



## 四、Safety Algorithm 範例（逐步跑表）

### 系統資源
- Total instances：A=10, B=5, C=7
- Initial Available：A=3, B=3, C=2

### 初始表格

| Process | Max (A B C) | Allocation (A B C) | Need (A B C) |
|-------|--------------|--------------------|--------------|
| P0 | 7 5 3 | 0 1 0 | 7 4 3 |
| P1 | 3 2 2 | 2 0 0 | 1 2 2 |
| P2 | 9 0 2 | 3 0 2 | 6 0 0 |
| P3 | 2 2 2 | 2 1 1 | 0 1 1 |
| P4 | 4 3 3 | 0 0 2 | 4 3 1 |

### Step-by-step Safe Sequence
1. **P1**：Need(1,2,2) ≤ Available(3,3,2) ✔  
   - Work = (3,3,2) + (2,0,0) = **(5,3,2)**

2. **P3**：Need(0,1,1) ≤ (5,3,2) ✔  
   - Work = (5,3,2) + (2,1,1) = **(7,4,3)**

3. **P4**：Need(4,3,1) ≤ (7,4,3) ✔  
   - Work = (7,4,3) + (0,0,2) = **(7,4,5)**

4. **P2**：Need(6,0,0) ≤ (7,4,5) ✔  
   - Work = (7,4,5) + (3,0,2) = **(10,4,7)**

5. **P0**：Need(7,4,3) ≤ (10,4,7) ✔

### 結論
- **Safe Sequence**：⟨P1, P3, P4, P2, P0⟩
- 系統處於 **Safe State**



## 五、Resource-Request Algorithm（請求檢查）

當 Process Pi 提出請求 `Request[i]` 時：

### 檢查順序
1. `Request[i] ≤ Need[i]`（不可超過最大需求）
2. `Request[i] ≤ Available`（系統必須有資源）
3. **假設分配後，執行 Safety Algorithm**
   - 若 Safe → **允許分配**
   - 若 Unsafe → **拒絕請求（等待）**



## 六、請求範例（Safe）

- Available：A=2, B=3, C=0
- Request(P1) = (1,0,2)

檢查：
- Request ≤ Need ✔
- Request ≤ Available ✔
- Safety check 後仍有 Safe Sequence

👉 **允許請求，系統仍為 Safe State**


## 七、請求範例（Unsafe）

- Available：A=0, B=0, C=2
- Request(P4) = (3,3,0)

檢查：
- Request ≤ Need ✔
- Request ≤ Available ❌ 或 Safety check 失敗

👉 **拒絕請求（會進入 Unsafe State，無 Safe Sequence）**

---

# Deadlock Detection（死結偵測）

## 一、Deadlock Detection 的核心概念
與 **Prevention / Avoidance** 不同，Deadlock Detection 的策略是：

> **允許 deadlock 發生，再定期偵測並處理。**

適合用在：
- Deadlock 發生頻率低
- 預防與避免成本太高的系統



## 二、Algorithm 1：Wait-For Graph（單一 instance）

### 適用情境
- **每種資源只有一個 instance**
- 可將 Resource-Allocation Graph（RAG）簡化

![alt text](/img/RAGWFG.png)


### Wait-For Graph（WFG）
- 節點：Processes（P1, P2, …）
- 邊：`Pi → Pj`
  - 表示 **Pi 正在等待 Pj 釋放資源**

![alt text](/img/WFG.png)

### 建立方式
- 從 RAG 中：
  - 移除 resource nodes
  - 若 `Pi → R → Pj`，則建立 `Pi → Pj`


### 偵測方式
- **定期對 WFG 進行 cycle detection**
- 判斷規則：
  - 無 cycle → 無 deadlock
  - 有 cycle → **deadlock 存在**

📌 在單一 instance 系統中：
> **Cycle ⇔ Deadlock**


## 三、Algorithm 2：Detection with Banker’s Algorithm（多個 instances）

### 適用情境
- **每種資源有多個 instances**
- 與 Banker’s Algorithm 類似，但用途不同



### 資料結構
- **Allocation**：目前配置資源
- **Request**：尚未滿足的請求
- **Available**：系統可用資源



### 概念
- 嘗試找出一個 sequence，使所有 process 都能完成
- 若找不到 → 系統發生 deadlock



### 範例說明
- Total instances：A=7, B=2, C=6
- Available：A=0, B=0, C=0

若存在 safe-like sequence：
\<P0, P2, P3, P1, P4\>  
→ **系統沒有 deadlock**

若某 process（如 P2）提出額外請求而：
- **無法找到任何 sequence**
→ **Deadlock detected**

---

# Deadlock Recovery（死結復原）

## 一、Recovery 的基本原則
一旦 deadlock 被偵測到：

> **必須破壞 deadlock cycle，使系統能繼續運作。**



## 二、Method 1：Process Termination（終止行程）

### 作法
1. **Abort all deadlocked processes**
   - 快速但代價大
2. **Abort one process at a time**
   - 每終止一個就重新檢查是否仍有 deadlock


### Victim Selection（考試重點）
選擇哪個 process 終止？常見考量：
- Process 優先權
- 已執行時間
- 尚需多少資源
- 佔用資源數量
- 是否為互動式或背景程式


## 三、Method 2：Resource Preemption（資源搶占）

### 步驟
1. **Select a victim**
   - 決定從哪個 process 搶資源
2. **Rollback**
   - 將 process 回復到安全狀態
3. **Starvation prevention**
   - 避免同一個 process 一直被犧牲


### Resource Preemption 的問題
- 哪些資源適合搶占？
- Rollback 成本高
- 必須防止 starvation

---
>Deadlock 的本質是「資源分配順序錯誤」；作業系統不是預防它、避免它，就是在發生後偵測並打破它。