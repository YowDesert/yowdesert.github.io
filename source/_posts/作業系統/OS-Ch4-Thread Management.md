---
title: OS-CH4-Thread Management
categories:
  - 作業系統
  - 第四章
tags:
  - 作業系統
abbrlink: 374236ea
mathjax: true
date: '2026-08-13 22:05:00'
---

# Thread

> 為了讓同一程式可以同時處理不同工作。

- 是輕量級的Process
- OS 分配 CPU Time的基本單位
- **私有的**組成包含 : 
    - Thread ID
    - PC 
    - Register Set
    - Stack
- 同一個Process 共享此Process的 (global)code/data section、OS Resource、...

## (考選) 表格
| **Private（每個 thread 自己有）** | **Shared with other threads of the same process（同一 process 內其他 threads 共用）** |
| -------------------------- | ---------------------------------------------------------------------------- |
| Program Counter (PC)       | Code section (text section)                                                  |
| CPU Register Set           | Data section（global data / global variables）                                 |
| Stack                      | **Heap**(指標)                                                                         |
| Local Variables            | Other OS resources（open files, signals, I/O resources, etc.）                 |
| Thread ID (TID)            | Static local variables（本質上也在 data section）                                   |

## Benefits (申論)
1. **Responsiveness**(回應程度) : 某個 Thread 被 blocked，其他 Thread 還可以繼續執行。
>程式會不會因為某個工作卡住，就整個沒反應？

2. **Resource Sharing**（資源共享）
3. **Economy**（經濟性 / 低成本）:
    - Thread 比 Process：
        - 建立（creation）較快
        - Context switch 較快
        - 使用記憶體較少
4. **Scalability** : Utilization of Multiprocessors Architectures 
    - 不同 Threads 可以分配到不同 CPU cores，真正平行執行（Parallelism）。
> 當 CPU core 變多時，程式能不能利用更多 cores，讓效能跟著提升？

> Process : 汽車 ; Thread : 引擎    

## Process v.s. Thread

| 比較項目             | Process                                                     | Thread                                                 |
| ---------------- | ----------------------------------------------------------- | ------------------------------------------------------ |
| Alias            | **Heavyweight process**                                     | **Lightweight process**                                |
| Model            | Single-threaded model                                       | Multithreading model                                   |
| Responsiveness   | 一旦 process 做 blocking system call，**整個 process 都會 blocked** | 某個 thread blocked 時，若還有其他可執行 thread，**process 仍可繼續執行** |
| Economy          | Process creation、context switch **慢、成本高**                   | Thread creation、context switch **快、成本低**               |
| Scalability      | Single-threaded process 通常只能使用 **一個 CPU/core**              | 同一 process 的 threads 可在不同 CPU/cores **平行執行**           |
| Resource Sharing | 不同 processes 原則上**不共享 memory / OS resources**               | 同一 process 的 threads 可共享 **code、data、OS resources**    |
| Synchronization  | 一般不需同步；若使用 shared memory 則需要                                | **需要 synchronization**，避免 race condition               |

# Multicore Programming
>Multithreading =「把程式拆成多條 thread」
>Multicore Programming =「讓工作有效分配到多個 core」

Multithreading  Programming
- 更有效利用 multiple CPU cores
- 提高 **concurrency**
- 在多核心下進一步做到 parallelism

## Concurrency vs. Parallelism

|        | Concurrency（並行／並發） | Parallelism（平行）       |
| ------ | ------------------ | --------------------- |
| 意思     | 多個 tasks 都在「推進」    | 多個 tasks **同一時間真的執行** |
| 是否同時執行 | 不一定                | 是                     |
| 單核心可以嗎 | ✅ 可以               | ❌ 不行                  |
| 多核心可以嗎 | ✅                  | ✅                     |

> Concurrency ≠ 同時執行
> Parallelism = 真正同時執行

## 平行化 的 5 個挑戰

1. Identifying Tasks
   - 找出可獨立、平行執行的 tasks

2. Balance
   - 各 task 工作量應平均
   - 避免某些 cores idle

3. Data Splitting
   - 將資料分割給不同 tasks / cores

4. Data Dependency
   - 有相依性的 tasks 需要 synchronization

5. Testing & Debugging
   - concurrent programs 執行順序不固定
   - 較難測試與除錯

## Type of Parallelism (記)
### Data Parallelism
- 將 data 切成多個 subsets
- 不同 cores 處理不同資料
- **執行相同 operation**
- Different Data + Same Operation
> 例子 :
Core 1：1,2 → 每個 ×2
Core 2：3,4 → 每個 ×2
Core 3：5,6 → 每個 ×2
Core 4：7,8 → 每個 ×2

### Task Parallelism
- 將不同 tasks/threads 分配到不同 cores
- 不同 cores **執行不同 operation**
- Different Tasks / Operations
> 例子
Core 1：讀取影片
Core 2：影像處理
Core 3：聲音處理
Core 4：儲存檔案


Data parallelism 與 Task parallelism 可以混合使用


## Amdahl's Law

Speedup ≤ 1 / [S + (1-S)/N]

- S：serial portion
- 1-S：parallel portion
- N：number of cores

N → ∞：
Speedup_max = 1/S

重點：
- 增加 cores 不代表效能等比例增加
- Serial portion 限制最大 speedup
- 提升平行化比例，比一直增加 cores 更重要

# User Thread vs. Kernel Thread
工作是由誰負責支援來區分thread種類

| 比較                      | User-Level Thread                      | Kernel-Level Thread           |
| ----------------------- | -------------------------------------- | ----------------------------- |
| 管理者                     | **User-level thread library**          | **OS Kernel**                 |
| Kernel 是否知道每條 thread    | ❌ 不知道                                  | ✅ 知道                          |
| 建立 / Context Switch     | **快**                                  | **較慢**                        |
| Blocking                | 一條 blocking，可能造成**整個 process blocked** | 一條 blocked，其他 thread **仍可執行** |
| Multicore / Parallelism | 傳統純 user-thread 模型下 ❌                  | ✅ 可以                          |
| Scheduling              | Thread library 排程                      | Kernel scheduler 排程           |

## User-Level Thread
- Managed by user-level thread library
- Kernel is **unaware** of individual user threads
- creation / context switch：Fast
- 優點：不用 kernel involvement，overhead 小
- 缺點：
  - blocking system call 可能使整個 process blocked
  - 傳統 many-to-one 下不能真正利用 multicore parallelism

## Kernel-Level Thread
- Managed directly by **OS kernel**
- Kernel **knows each thread**
- creation / context switch：Slower
- 優點：
  - one thread blocked → other threads can run
  - 可在 multiple CPU cores 上 parallel execution
- 缺點：kernel involvement → overhead 較高

## 必背
User Thread：快，但 kernel 看不到
Kernel Thread：慢一些，但 kernel 可個別排程

Kernel knows thread
→ independent scheduling
→ blocking 不影響其他 threads
→ 支援 multicore parallelism

| Items                                   | **User Thread**            | **Kernel Thread**    |
| --------------------------------------- | -------------------------- | -------------------- |
| **Supported in**                        | User level（Thread library） | Kernel level（Kernel） |
| **Responsiveness**                      | ❌ 較差                       | ✅ 較好                 |
| **Scalability**                         | ❌ 較差                       | ✅ 較好                 |
| **Speed for creation / context switch** | **Fast**                   | **Slow**             |
> User Thread：Responsiveness : Kernel 看不到 T1、T2、T3，它只知道「這是一個 process」。
> Scability : Kernel 也不知道 T1、T2、T3 可以分開排程。→ 難以真正平行跑在不同 core。

# User-Thread 對應 Kernel-Thread

| Model            | Mapping | Parallelism | 一條 Blocking    | Creation / Switch | 主要問題                   |
| ---------------- | ------- | ----------- | -------------- | ----------------- | ---------------------- |
| **Many-to-One**  | M:1     | ❌           | 整個 process 受影響 | **快**             | 無法利用 multicore         |
| **One-to-One**   | 1:1     | ✅           | 其他 threads 可繼續 | **較慢**            | Kernel thread overhead |
| **Many-to-Many** | M:N     | ✅           | 其他 threads 可繼續 | 折衷                | **實作複雜**               |

## Many-to-One
Many-to-one : Many User Thread -> 1 Kernel Thread
- Thread 由 thread libraby in **User Space** => Efficient
缺點 : 
- 喪失Reponsiveness : 只要Block 全部不能執行
- 喪失Scability : 只有一個Kernel Thread -> Scalability 差

## One-to-One
- Reponsiveness好 : 一個Block -> 其他還能跑
- Scability 好 : 多條Kernel Thread 可以分到多個CPU Core 
缺 : 一條對一條所以如果建立大量很耗效能

## Many-to-Many
很多 User Threads → 較少/相等的 Kernel Threads

Many-to-One 的優點 :
- 可以建立很多 User Threads
One-to-One 的優點 : 
- 多核心 Parallelism
- 一條 thread block 時，其他 thread 繼續執行

## Many-to-Many Variation：Two-Level Model
允許某一個 User Thread **綁定**到特定 Kernel Thread。
> 基本架構是 M:N，但又允許部分 thread 採用 1:1 binding

Many-to-Many 很難製作 ， 現在都朝向 1-1 因為Thread進步可以多到用不完那樣

# Thread Library 
提供Programmer 相關的APIs與建立及管理Threads

兩種 : 
1. 沒有Kernel 支持，全部在User Space
   - Fast、Overhead小
2. 全部都在Kernel-Level
   - 需要System Call


> Pthread 可以是 **Kernel、User Thread**

## 各 OS 用 Thread
```
UNIX / Linux / macOS
        ↓
     Pthreads

      Windows 
        ↓
 Windows Threads API

      Java 
        ↓
 Java Thread API
        ↓
 JVM 使用 Host OS 提供的 native thread
```
Java：**無 global variable**，但可用 object/static data 做 shared data。

## Asynchronous vs Synchronous Threading

- Asynchronous
   - Parent 建立 Child 後**立即繼續執行**
   - Parent / Child independently execute
   - 通常 **little** data sharing
   - 常用於 responsive UI / server

- Synchronous
   - Parent 建立 Child 後**等待 Child 完成**
   - Child finish → join Parent
   - 通常 **significant** data sharing
   - Parent 常負責收集 Child 的結果

## Pthread 
Pthreads 是 POSIX(IEEE) 定義的 Thread **API specification**，不是某一個固定的 implementation。

### Important APIs
- `pthread_t tid`：Thread ID
- `pthread_attr_t attr`：Thread attributes
- `pthread_attr_init()`：初始化 default attributes

- `pthread_create(&tid, &attr, func, arg)`
  - 建立 Thread
  - tid：Thread ID
  - attr：屬性
  - func：Thread 要執行的 function
  - arg：傳入參數

- `pthread_join(tid, NULL)`
  - 等待指定 Thread 結束
  - 類似 Process 的 `wait()`
  - 父等子完成

- `pthread_exit()`
  - 結束目前 Thread

# Threading Issue
在Single Thread不會發生，但在**Multi-Thread**會發生的

## Fork() and exec() System Call
- Fork()只要複製callling thread
- Fork()要複製所有Thread

如果Fork()後面接Exec()則只需複製Calling Thread : 因為Exec 會被新程式取代

## Signal Handling (考過一次)

**定義 (考過)** : UNIX System 通知 Process 有特定事件發生 
**兩種模式 (考過)** : 
- **Synchronous signals** : 目前程式自己造成的
- **Asynchronous signals** : 外部事件造成的

Multithreaded Process收到Signal 要給誰? 
四種處理方法 : 
- 給造成Signal的Thread(sync. signal)
- 給Process的所有Thread
- 給某些特定的Thread
- 設定專門的Signal Handler Thread
   - Default
   - User-Defined

## Thread Cancellation (選擇)
Thread 還沒做完你就把它終止了！
被**取消的**Thread : **Target Thread**
2種Cacellation : 
1. **Asynchrounous** cancellation : 立刻中止Target Thread
   - 有可能還在修改Shared Data、還把Mutex鎖住
2. **Deffered**(延遲) cancellation : 等到Safe Point再中止

> pthread_testcancel() : 主動建立/檢查cancellation point

## Thread Local Storage(課本放在補充)
有些資料需要**每個Thread獨立保**存 => TLS
- 同一 Process 的 Threads 通常共享 Data / Heap
- TLS：讓每個 Thread 對某變數擁有**自己的獨立** copy
- 各 Thread 修改互不影響
- 用途：儲存 Thread-specific data
- POSIX：`pthread_key_t`

### TLS 和 local variable 不一樣
- Local variable 
   - function 開始-> x 存在 -> function 結束 -> x 消失
   - 只在一次 function invocation 中存在。
- TLS
   - Thread 開始 -> TLS data 存在 -> function A -> function B -> function C -> 仍然是同一份 TLS
   - TLS 可以跨 function invocation 存在。
   - 每個 thread 有自己的 copy。

# Implicit Threading 
提供一些工具，讓developer減少平行化程式撰寫困難
- Explicit threading：
  - Programmer **自己 create / manage threads**
- Implicit threading：
  - Programmer 只定義可平行的 **tasks**
  - Compiler / Runtime Library 負責 thread creation & management
- 優點：
  - 降低 multithread programming 複雜度
  - 適合大量 threads/tasks
- 常以 **many-to-many** 的方式管理

## 五個方法

## Thread Pool
**預先建立**一批thread 放在**pool**裡，需要就直接拿來用，用完再放回去
如果有人想用但沒有Threads了就要等。

- 如果Request 才建立thread有兩個問題
   1. 建立刪除有成本 => 對Client 不是很迅速
   2. 如果沒限制數量,則系統會建立過多的threads

### Thread Pool 三個優點
1. 服務請求迅速
2. 限制Thread 生成數量 : Pool固定數量
3. 工作排程彈性 : 工作（Task）不用自己綁死某一個 Thread，可以先丟進工作佇列，再由 Thread Pool 決定哪個 Thread 來執行。

## 以下在補充

## Fork-Join
Fork : 把大工作拆成數個小工作
Join : 等兩邊都算完再把結果拿回來

## OpenMP(一次)
是一組compiler directives(#...) + API 

像是這段幫我**平行**

```c
#pragma omp parallel
{
    printf("Hello");
}
```

把 for loop 的 iterations **分給多個 threads**。
```c
#pragma omp parallel for

for (i = 0; i < N; i++) {
    C[i] = A[i] + B[i];
}
```

## Grand Central Dispatch(GCD)(選擇)(1~2次)
是**Apple**出的
GCD 將要排程的Task放置於Dispatch Queue,當Tasks要用就從pool找可以用的thread分派給task用
**提供兩種型態 :(考)**
### Serial Queue (考)
Serial = 一次只能執行一個 Task
取出的順序是：**FIFO**
所以：
```
時間 →
A A A A
        B B B
              C C C
```
```
加入：
A → B → C

取出：
A → B → C
```
> Serial Queue 可以確保 tasks sequential execution。

### Concurrent Queue (考)
**按照 FIFO 取出**
但**一次可以取出多個 tasks** 交給不同 threads 執行。
可以是這樣
```
Thread 1：AAAAAAAA
Thread 2：  BBB
Thread 3：    CCCCCCCCC
```
所以 B 可能比 A 更早完成

|             | Serial Queue | Concurrent Queue |
| ----------- | ------------ | ---------------- |
| 取出順序        | FIFO         | FIFO             |
| 同時執行 task   | 1 個          | 多個               |
| Parallelism | ❌            | ✅ 可以             |
| 完成順序        | 通常照 FIFO     | 不一定              |

## Intel Threading Building block (沒說到)

# Scheduler Activations
LWP = Lightweight Process
LWP 位於 User - Kernel 之間，且位於**Kernel Mode**
```
User Thread
     ↓
    LWP
     ↓
Kernel Thread
     ↓
 Physical CPU
```
Thread Library : 
- 建立 / 刪除 Thread
- 決定哪個 User Thread 要執行
- 在 User Threads 之間做 context switch

> Thread Library = User Space 裡管理 User Threads 的管理員。
> Thread Library 通常位於 User Space；但它管理 Thread 時，可能完全在 User Space 做，也**可能呼叫 Kernel** 來完成。
> User Threads 由 **user-level thread library** 管理；Kernel Threads 則由 **OS kernel** 直接管理。

```
User Space
┌──────────────────────────────┐
│ Process                      │
│                              │
│ User Thread A                │
│ User Thread B                │
│ User Thread C                │
│         ↓                    │
│   Thread Library             │
│   ├─ create thread           │
│   ├─ delete thread           │
│   ├─ schedule thread         │
│   └─ context switch          │
└──────────────┬───────────────┘
               ↓
──────────── Kernel Space ────────────
               ↓
        Kernel Thread / LWP
               ↓
              CPU
```
- **一個 LWP 綁一個 Kernel Thread**
- 很多 User Threads -> 幾個 LWP : many-to-many  
- 如果Kernel Thread Block -> LWP Block -> User Attach Block

## CPU-bound 為什麼只需要一個 LWP？
假設在Single Processor 同一時間只能有一個thread 真正使用 CPU，所以 1 LWP 就夠了。

## I/O-intensive 為什麼需要多個 LWP？
因為如果一個Blocked 其他的還可以跑。

## Upcall
Kernel 怎麼告訴 user thread library：「欸，你的 LWP block 了！」？

- 一般 System Call : Application ->  Kernel : read(),write()
- Upcall 反過來 : Kernel -> Application : Kernel 通知 User 卡住了


# Linux `clone()` (中央常考)
Linux 除了 Fork() 以外還有 `Clone()`
- Linux 使用 `task` 表示 execution flow
- `clone()` 透過 **flags** 決定 parent / child 資源共享程度

常見 flags：
- `CLONE_VM`：share memory ⭐
- `CLONE_FILES`：share open files
- `CLONE_SIGHAND`：share signal handlers
- `CLONE_FS`：share file-system info

- 沒有設定的話 -> **則沒有sharing**
- 少 / 無 sharing → **Process-like / fork-like**
- 多 sharing → **Thread-like**

> 核心：`clone()` 可以建立「像 process」或「像 thread」的 task。
