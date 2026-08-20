---
title: OS-CH4-CPU Scheduling
categories:
  - 作業系統
  - 第四章
tags:
  - 作業系統
abbrlink: 6f2552db
mathjax: true
date: '2026-08-13 10:30:00'
---

# Scheduling Critiiria and Goal

1. **CPU utilization** : Process execution time / CPU Total Time
    > - CPU Total Time : 
    >    - Process exec Time 
    >    - Context Switch Time
    >    - Idle
2. **Throughput** : 單位時間內完成的工作數目

3. **Turnaround Time** : process 進入到工作完成的時間 = finished time - arrival time
4. **Waiting Time** : Process 待在Ready Queue 等待時間 = Turnaround Time - Cpu Burst Time - I/O op. TIme
5. **Response Time** : 系統產生第一個回應時間

# I/O Bound / CPU Bound
I/O Bound process : 大部分時間花在I/O
- 多、短 CPU
- ex : DBMS,成績計算,外部search
CPU-Bound process : 大部分時間花在處理
- 少長CPU
- ex. 氣象預估,基因比對,AI Model 訓練

另一重點 : 
- I/O Bound 對 CPU 優先權高 (因為CPU Burst短)
- CPU Bound 對 I/O Device較高

# CPU Scheduler 
1. Running -> Waiting
2. Running -> Ready
3. Waiting -> Ready
4. Running -> Terminated
這四個情況要CPU Scheduler再選一個
> 1,4 : non Preemptive or **Cooperative** (記)
> 其他的都是 Preemptive

## Preemptive , non-preemptive
**non-preemptive** : 
- 不能搶走Process的CPU
- context switch 次數少、Process完工較能預期
**Preemptive** :
- 能搶走Process的CPU
- 優缺與上面相反
> 現在os 大多Preemptiive

## Preemptive , non-preemptive Kernel (要知道)
Process 正在被 **Kernel Mode** 執行可不可以搶？
> Preemptive : Race Condition 可能發生
> 方法有二解決
> - 插入Preemptive point 確保那裏是安全ㄉ(現已Removed)
> - 提供互斥存取 => 缺點可能產生**Priority Inversion**

**non-preemptive Kernel** : 
- 必須等到System call or I/O complete才可以 **context switch**
- 對Real Time 非常不利！
**Preemptive Kernel** :
隨時可以被User Process 插隊，所以需要mutex lock 來防止 race condition

### Priority Inversion 
即使高優先權先取得了CPU，但是因為共享Data/Resources恰巧被低優先權把持，形成高等低的狀況就是**優先權反轉**,且低優先權等不到CPU而一直無法釋放資源。
解法 : **Priority Inheritance**
- **暫時讓低優先權拿到高優先權的權重** -> 再恢復原本的優先權

# CPU Scheduling Alogorithm
- **FCFS** (First come First Serve)
    - 最先到的最先解決
    - 效益最差
    - **Convoy effect** : 前面如果超久，後面都要等，平均等待時間會大幅增加
    - nonpreemptive、公平
- **SJF**(Shortest-Job-First)
    - 最短的cpu burst先執行
    - SJF是**optimal**
    - 分為 non-preemptive、preemptive
        - Non-Preemptive SJF->SJF
        - Preemptive SFJ -> SRTF
    - 有Starvation Problem
    - 不適合用在**short-term scheduler**(預估值可能不太精確)
    - 決定下一次CPU Burst
    $$
    \tau_{n+1} = \alpha t_n + (1 - \alpha) \tau_n
    $$
    >$ \tau_{n+1} $ : 下一次預估值
    > tn : 本次burst預估值
- **SRTF** (shrotest-remaining-Time-First)
    - 效益最佳
    - 剩餘CPU-burst 最短的process**優先插隊**取得CPU
- **RR**(Round Robin)
    - OS 規定一個Time Quantum,如果沒有在這時間內完成 => 發出Time-Out interrupt => 放回Ready Queue尾端 (以FIFO order)
    - 適合**Time-sharing**
    - 平均waiting time often long
    - 效能取決於Quantum大小
        - q => Large => 退化成FIFO
        - q => Snall => CPU utilization 幾乎 = 0
        - 經驗法則大小要時間內**80%的Job**可以完成(**八二法則**)
    
- **Priority Scheduling**
    - 優先權**越高先執行**，如果一樣就FCFS    
    - non-preemptive 、 Preemptive
    - 有Starvation、用 Aging解
    - 適用於Real-Time
    - 例子
        - Arrival Time小,優先權高 => FCFS
        - Burst Time 小 => SJF,SRTF
- **MultiLevel Queue**
    - 用多條Ready Queue
    - Queue之間可以Preemptive priority
    - 每一條Queue可以有自己的排班法則,但不允許process在不同queue移動
    > 一旦Process被放到某個queue就不能在不同Queue之間移動

- **MultiLevel FeedBack Queue**(MFQS)
    - 允許Process **move** between Queues => 防止Starvation
    - Process 等太久也可以 move to higer-priority Queue
    - 可參數畫項目，包含 :
        - Queue的數量
        - 每個Queue的Scheduling演算法 
        - Queue 需要服務的時候應該進哪個Queue
        - Queue之間的排班法則
        - 決定何時升級 降級
# Multiple-Processor Scheduling
Asymmetric multiprocessing
- 採取**Master-Slave**
    - 缺點 : Master Server becomes a 淺在地 **bottleneck** => 系統效能不佳

SMP (Symmetric multiprocessing)的排班 => 每個CPU自己排班
共有兩種 :
- 所有CPU共享common ready Queue
    - 可能有Race Condition 在Shared ready Queue
    - 要用**同步機制來防止此** - race condition
    - 不用考慮**負載不平衡**
- 每個CPU有自己的(private)Ready Queue 非共享
    -  沒有Race Condition
    - 考慮**負載不平衡**

## Load Balancing 技術
當每個CPU有自己的Ready Queue時候需要,因為如果是共享只要變Idle就會立刻抓來用
兩種方法 : 
- **Push migration** : 忙的那邊把工作推出去
- **Pull migration** :　閒的CPU主動把工作拉過來
> 沒有一定是互斥

## Processor Affinity
Process 傾向 留在原本的CPU執行，以利用原本建立好的warm cache
Load Balancing => 追求負載平均
Processor Affinity 追求 => Cache locality
>因此兩者存在 trade-off。

兩個方法 :
- Soft Affinity : OS 盡量process 留在同一個CPU，但不保證。
- Hard Affinity : 限制process只能在哪些CPU上執行
> 衝突 : Load Balancing 想搬 Process；Processor Affinity 不想搬 Process。

NUMA
→ Local memory：快
→ Remote memory：慢
→ Migration 可能增加 memory access time

# Multicore Processor Scheduling
- Memory stall：CPU 等待 memory，常因 cache miss
- 1 core 可有多個 HW threads
- 一條 HW thread stall → 改跑另一條，提高 CPU utilization
- Logical CPU = Core × HW threads/core
- Two-level scheduling：
    - L1 OS scheduler：SW thread → HW thread
    - L2 HW scheduler：決定 core 執行哪條 HW thread

# Real-Time System 

U = Σ(Ci / Pi) <= 1 才成立 : 表示N個Processes加總的CPU消耗率<=1 

<=1 : Scheduable
\>1 : Non-Scheduable

目前能處理的型態 : **Synchronus** Real Time System 
> **Synchronus** : 有清楚的Time、定期的事件
> Asynchronus : 不定期事件


## Real Time System 2 Algorithm
- Rate Monotonic Scheduling
    - 採取Preemptive **Static** Priority : process **優先權一旦決定就不能改**
    - 優先權規則 : Period Time 越小，優先權越高　=> 發生週期小(頻率高的)
- EDF Scheduling (Earliest Deadline First Scheduling)
    - 採取Preemptive **Dynamic** Priority : process **優先權可以動態改變**
    - 優先權規則 : **Deadline 越早**優先權越高
    - 是**Optimal** Solution : 所有processes 皆可以滿足(DeadLine內)
    - 實務上，CPU utilization cannot be 100%

---

# 補充 (只考過一次)(選)
## Solaris & Linux CFS 補充

### Solaris
- Priority-based scheduling
- Quantum 用完 → priority ↓（偏 CPU-bound）
- I/O 完成、return from sleep → priority ↑（偏 I/O-bound）

### Linux CFS
- nice：-20 ~ +19，越小 priority 越高
- 每個 task 維護 `vruntime`
- 選 **vruntime 最小** 的 task 執行
- 使用 **Red-Black Tree**=>O(log(n)) (考過)

### Scheduling Algorithm Evaluation (中央一次)
4 種方法：
1. **Deterministic Modeling**：固定已知的資料直接算
2. **Queueing Model**：數學模型
   - Little's Law：`n = λW`
3. **Simulation**：程式模擬
4. **Implementation**：實際 OS 測試，最準但成本最高

