---
title: OS-CH4-Process Management
categories:
  - 作業系統
  - 第四章
tags:
  - 作業系統
abbrlink: 12e18ab3
mathjax: true
date: '2026-08-12 14:30:00'
---
# Process Management

## Process Concept
Process : 執行中的程式
- Memory Layout的內容
    - **Text Section** (Code Section):Program Code
    - **Stack** : Prameter、Return Address、**Local** Variable
    - **Data Section** : **Global** Variable 
    - **Heap** : 記憶體在Run time動態分配
- 執行的一些 Status Data
    - **Current activity** : PC、PProcrssor registers 。通常在**PCB**紀錄(OS負責管理在Kernel Memory Area)

```
由低位址 → 高位址：

High Address
┌─────────────────────┐
│ argc, argv           │
├─────────────────────┤
│ Stack ↓              │ ← local variable、parameter、return address
│                      │
│         ↑            │
│ Heap  ↑              │ ← malloc / new 動態配置
├─────────────────────┤
│ Uninitialized Data   │ ← 未初始化 global / static（BSS）
├─────────────────────┤
│ Initialized Data     │ ← 已初始化 global / static
├─────────────────────┤
│ Text / Code          │ ← 程式機器碼
└─────────────────────┘
Low Address
```

## Program v.s. Process
Program : Passive Entity 存在 Storage Devices
Process : Active Entity 帶有 PC 是執行中的程式

## Process 5-State Model

五個基本狀態：
- New
    Process 正在建立
    OS 建立 PCB、配置資源
- Ready
    已經在 memory
    除了 CPU 以外，執行條件都準備好了，等待 CPU
- Running
    正在 CPU 上執行
- Waiting / Blocked
    等待某個 event
    例如：I/O completion
    ⚠️ 此時不是在等 CPU
- Terminated
    Process 執行完成
    OS 回收相關資源

## Short/Medium/Long Term Scheduler
- Short Term(CPU Scheduler)
    - Ready → Running
    - 執行頻率最高，因此必須非常快
- Medium Term (Swapper)
    - Memory ↔ Disk
    - Swap Out / Swap In
    - 記憶體不足時暫時把 process 移出去與 Suspended state 有關
- Long Term Scheduler(Job Scheduler)
    - New → Ready
    - 決定哪些 job 可以進入 memory
    - 控制 Degree of Multiprogramming
    - 執行頻率低

## STD (Process State Transition Diagram)

- **New -> Ready(Admit)**
當OS分配Process Memory Space -> Memory中。
在Batch System -> **Long Term Scheduler**
Real Time,Time-Sharing -> 不用Long-Term

- **Ready -> Running(Scheler Dispatch)**
由 **Short-Term Scheduler**(or CPU/Process Scheduler) 決定分派給CPU執行。
- **Running -> ready(interrupt)**
執行中的Process因為某些事件發生(time-out/interrupt)**被迫**放棄CPU -> 回Ready
- **Running -> Blocked(I/O event wait)**
要等待I/O or event **自願**放棄CPU
- **Blocked -> Ready(I/O event complete)**
I/O or Event 做完了->Ready

- **Running -> Terminated(Exit)**
Process 完成所有工作，release 資源。

## 7-State Model
5-State：
New / Ready / Running / Waiting(Blocked) / Terminated

7-State 多兩個：

- Ready/Suspend
- Blocked/Suspend

> Suspend = Process 被 **swap out 到 Disk**
> 因此不在 Main Memory 中。

- 為什麼需要 Suspend？
    - Main Memory 不足時：
    Medium-Term Scheduler 可以把 process：**Memory → Disk**
    稱為：**Swap Out / Suspend**
    目的：-> Free Memory Space
    - Memory 足夠時：Disk → Memory
    稱為：**Swap In / Activate**

### 7-State 重要 Transition
- 原本 5-State
    1. New → Ready
    2. Ready → Running       : Dispatch
    3. Running → Ready       : Time out / Preemption
    4. Running → Blocked     : Event wait / I/O
    5. Blocked → Ready       : Event occurs
    6. Running → Exit        : Release
- 加入 Suspend
    - **Blocked → Blocked/Suspend**
        - Swap Out
        - Process 還在等 event
        - 只是被移到 Disk
        - 由**Midium-Term Scheduler** 選
    - **Blocked/Suspend → Blocked**(也許可以加入)
        - Swap In
        - Event 還沒完成
        - 這是Poor Design
        - 相信馬上Ready 所以會優先 Swap in
    - **Ready → Ready/Suspend**
        - Swap Out
        - 理由1 : Swap out 才有足夠的Memory 
        - 理由2 : 所有Blocked Process優先權皆高於Ready，且OS相信馬上會Ready
    - **Ready/Suspend → Ready**
        - Swap In
        - 當Memory 足夠ㄌ 

    - New -> Ready/Suspend
        - 沒有足夠Memory  
    > 以下虛線
    - Running -> Ready/Suspend
        - This is Poor Design 
        - 有些Block/suspend -> Ready/Suspend 且 是Highest-Priority => OS可**強迫**低優先Running Process同時放掉CPU 、 Memory Running Process

## 6-State-Model
只有一個suspend : Blocked -> Suspend -> Ready

# Process Control Block

> OS 用來記錄、管理一個 Process 所有重要資訊的資料結構。

- 每個 Process 都有 PCB
- PCB 存在 **Kernel Memory**
- **Context Switch** 時非常重要

## PCB 內容
1. **Process State** : new,ready...
2. **PC(Program Counter)** : 下一個要執行的 instruction address
3. **CPU Register** : 
    - General registers
    - Stack Pointer
    - Status registers
    - ...
4. **CPU-scheduling information**
    - Priority
    - Scheduling queue pointers
    - Scheduling parameters 
5. **Memory-management information**
    - Base / Limit Register
    - Page Table
    - Segment Table
6. **Accounting information** 
    - CPU time
    - Time limit
    - Process / Job number
7. **I/O status information**
    - Allocated I/O devices
    - Open files
> 以前有8
8. **PID** : Process ID

# Dispatcher (申論)
分派器 : 負責將CPU 交給 **短期排班器選出的**Process
主要三個工作 :
- Switch context
- switching to user mode(if user process)
- 跳到 Process 下一個要執行的位置

**分派延遲** : Dispatcher 停止一個 Process，並開始執行另一個 Process 所需要的時間。

```
流程：

Pi Running
    ↓
Save Pi → PCB_i
    ↓
Restore Pk ← PCB_k
    ↓
Switch to User Mode
    ↓
Jump to Pk's PC
    ↓
Pk Running
```

# Context Switch (申論)
目前Process -> 另一個Process使用前，OS必須
- Save the state -> Load the saved state for the new process

Process Context : 儲存在PCB的內容
Context switch time is **OverHead**:系統無法執行Processes
- Context 時間長短接取決於硬體因素
    - 所有Process共用一套Register -> 數量多Context switch 長

# Process Creation
1. 任何process均可建立自己的**child process**
Parent process -> child process
2. **目的** : 
    - 執行與parent 相同的工作
    - 執行與parent 不同的工作
3. 資源共享
    - Parent、child Share **All** Resources
    - Children share **subset** of parent's resources
    - Share **NO** Resources
4. Child Parent 互動
    - **並行**執行
    - Parent 等待 Children 結束
5. Child Process 占用的 Address Space
    - child **duplicate** of parent (fork())
    - Fork 是**複製**爸爸，再由**下一行程式碼**繼續 不會從頭
    - Child has a program **loaded** into it(新內容不同於Parent內容)(exec())

## UNIX Process creation
- fork()
    - 建立 Child Process
    - Child 一開始**複製** Parent 的 address space
    - Parent / Child 為**不同 address space**
    - fork() 後兩者可 **concurrent** execution
    - Return：
        - Child → 0
        - Parent → Child PID (> 0)
        - Fail → -1
        - 現代 OS：Copy-on-Write (COW)
    > fork 生小孩：爸拿 Child PID、小孩拿 0；記憶體先共享，誰寫誰 copy。
- exec()
    - 不建立新的 process
    - 將目前 process 的 program image 換成新程式
    - 成功 → 不 return
    - 常見：fork() → exec()
- wait() : Parent 等待 Child terminate
> **wait()** : 只負責等待子結束，不能收集孩子的資訊，只能知道他是不是正常結束

- getpid()
- exit()

## Root Process : init()
- 定 init() 為所有child的root
- PID = 1
- 系統開機第一個生出的Process
- 後來用system 取代 init()
- 提供指令 ps -el 、 pstree

## Process Termination
1. Process執行完->exit()
    - child 透過 **wait** 傳回satus data
    - Dealllocate By OS
2. Parent 也可能直接終止 children process,使用abort()
    - Child 使用**超過**配置給他的資源
    - 指定給child工作**已經不再需要**
    - **Parent 終止**，且OS**不允許**他的child繼續執行

3. Cascading Termination
**連帶終止**

- Parent 終止後,不允許child processes繼續執行
- 由OS initiated

4. 可以使用wait()等children終止: pid = wait(&status)

5. Zombie Process : Child 終止了,但parent 還沒 call wait() -> call 完了才會收回child's PID PCB空間
6. Orphans Process : parent 尚未執行到wait(),就被終止了。

|        | Zombie            | Orphan    |
| ------ | ----------------- | --------- |
| Parent | **活著，但還沒 wait**   | **死了**    |
| Child  | **死了**            | **活著**    |
| 關鍵     | Child 等 Parent 收屍 | Child 沒爸爸 |
| 背法     | **子死父未收**         | **父死子還活** |

## Android Hierachy

Android 會按照「Process 對使用者的重要程度」決定誰先被 terminate。

1. Foreground Process   ← 最重要
        ↓
2. Visible Process
        ↓
3. Service Process
        ↓
4. Background Process
        ↓
5. Empty Process        ← 最不重要

# (補充) (A)Synchronous I/O

| 類型                   | 發出 I/O 後       |
| -------------------- | -------------- |
| **Synchronous I/O**  | 等 I/O 完成再繼續    |
| **Asynchronous I/O** | 不用等完成，可以先做其他工作 |
