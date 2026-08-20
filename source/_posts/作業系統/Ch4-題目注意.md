---
title: OS-CH4-補充加強觀念
categories:
  - 研究所題目
  - 作業系統
  - 第四章
tags:
  - 作業系統
abbrlink: 603b4b38
mathjax: true
date: '2026-08-14 17:49:00'
---

- I/O Device Queue 是OS管理的
- **Timer** 不在PCB裡 ， 他是用來產生Interrupt的，不是CPU使用時間資訊 
- Which Schedulers can be used to control the degree of multiprogramming?
    - 問誰可以增/減Memory的Process
    - Job Scheduler(Long Term) : 看誰要進Memory、Medium-Term Scheduler : Swapper to Disk
- Job Scheduler = Long-Term Scheduler
- CPU Scheduler = Short-term Scheduler
- Disk scheduler => Disk I/O request 哪一個先處理
- Printing 表 I/O 完成了

- (A)Synchronous I/O

| 類型                   | 發出 I/O 後       |
| -------------------- | -------------- |
| **Synchronous I/O**  | 等 I/O 完成再繼續    |
| **Asynchronous I/O** | 不用等完成，可以先做其他工作 |

- Context Switch 除了要儲存與載入Process Context 還可能破壞 Cache Locality 造成 Cache Miss

- Thread **沒有共用 PC** -> 可能自己做到自己的位置
- Exec()被呼叫-> **Process ID 不換** : 因為只是把執行的程式換掉 

- **fork()** = 從「目前這個位置」複製一個 process，接著從下一步程式碼開始

- **wait()** : 只負責等待子結束，不能收集孩子的資訊，只能知道他是不是正常結束 

- 每個 Thread **不共用 PC**

- SJF 不適合 Short-Term Scheduler : 因為執行時間太頻繁了，來不及預測

- RR 是設計給 Time-Sharing

- RR 是使用 FIFO **Queue**

- $A \subset B$ 是代表 A包含於B 但不等於
- CPU Efficiency = 真正做工作的時間 / (真正工作時間 + overhead)
  - T : Process 平均跑`T`會被Block；S:Switch 時間；Q : RR Quantum
  - S < Q < T : $*Q / (Q + S)$
  - Q => 無限 : $ T / (T + S) $

- Interactive processes usually produce short **CPU Burst**
- CPU-Bound 要給他 **低priority**
- SJF是一種策略**改善Waiting Time**

- Process 的 Thread 共享 **Heap** 、 **Global variable**、**Static Local Variable**、Code Text、Open File
  > Threads share Code、Data、Heap、Files；各自有 Stack、PC、Registers。

- 描述 Thread / Process Switch
  - Thread Switch : 只需儲存舊Thread Data，PC、Stack、Registers等**私有Data**，但同一Process的**共享資訊**不需要切換
  - Process Switch : 需要儲存舊Process 的狀態，並且恢復下一個Process的資訊，除了**CPU Register**以外，還有**記憶體配置Data**，還可能需要**刷新Data Cache、Instruction Cache**
- LWP 只連接一個Kernel Thread
- User Thread 之間的排程與Context Switch，**由Thread Library負責**，不需要Kernel
- Kernel **只負責 Kernel Thread**
- LWP 會儲存目前正在它上面執行的User Thread的**Register Set**
- Linux / Windows / macOS => **one-to-one** Model
- Many-to-one => 舊Java、**Green**
- Many-to-Many => 舊Solaris、Implicit Thread
> 現在都是 One-to-One

- Process / Thread 分別需要甚麼執行

|Resource|Thread|Process|
|---|---|---|
|Memory|**X**|O|
|PC|O|O|
|Stack|O|O|
|Register|O|O|
|I/O Resource|**X**|O|

- Context Switch : CPU 從這個Process  **換** 跑另一個Process
- Linux 有 Hard/Soft Addinity
- process 結束要求OS用**Exit()**刪除他
- Android裡，`Empty process`=> 主要留下來當Cache，方便之後快速啟動
- 正在跟使用者互動的 => **Foreground Process**
- **Visible Process** 使用者看的到，但不一定在操作
- 影響context Switch Time
  - Register 數量
  - Special Instruction : 可以一次存很多，與一次存一個
  - Memory Speed
  - ~~Disk Size 沒有~~
- 有 **idle Process**
- MLFQ 通常是高層先跑完，全部跑完一輪空了 => 下一層繼續 => 直到最後一層
- C library 需要OS System call才能Access檔案
- SJN / SJF : 主要是最小化Average Waiting Time
- 區域變數可以**有效利用記憶體**，因為它使用的空間會在需要的時候才配置，用完之後再釋放
- 同一Process 不能共享區域變數
- User Thread 比 Kernel Thread的優點在 : **切換快、建立快**
- Thread v.s. Process
  - Process : 是在執行的Program ， 通常包含 Code/Data Section、Heap、Stack，另外會用PCB來保存Data : 有PC、CPU Register
  - Thread : 是 CPU Utilicaiton unit，一個Process可以有很多Thread，共享Code 、 Data、Heap、OS Resource，但每個Thrad自己有TID、PC、Stack
- Thread v.s Process優點與限制


| 比較項目             | Process                                                     | Thread                                                 |
| ---------------- | ----------------------------------------------------------- | ------------------------------------------------------ |
| Alias            | **Heavyweight process**                                     | **Lightweight process**                                |
| Model            | Single-threaded model                                       | Multithreading model                                   |
| Responsiveness   | 一旦 process 做 blocking system call，**整個 process 都會 blocked** | 某個 thread blocked 時，若還有其他可執行 thread，**process 仍可繼續執行** |
| Economy          | Process creation、context switch **慢、成本高**                   | Thread creation、context switch **快、成本低**               |
| Scalability      | Single-threaded process 通常只能使用 **一個 CPU/core**              | 同一 process 的 threads 可在不同 CPU/cores **平行執行**           |
| Resource Sharing | 不同 processes 原則上**不共享 memory / OS resources**               | 同一 process 的 threads 可共享 **code、data、OS resources**    |
| Synchronization  | 一般不需同步；若使用 shared memory 則需要                                | **需要 synchronization**，避免 race condition               |