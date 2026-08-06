---
title: OS-CH3
categories:
  - 作業系統
  - 第三章
tags:
  - 作業系統
abbrlink: 978b4c60
mathjax: true
date: '2026-8-04 16:00:00'
---

# System call
為User Process 與 Kernel 的溝通介面

**System call 經常利用 Trap(Software Interrupt) 進入 Kernel** 

System call 通常被 CPU 視為software interrupt

> User -> Trap -> Kernel mode -> OS Service -> Return User

現代系統會透過**Application Programming Interface(API)**來呼叫System Call
好處 : 
- program portability
- 更方便使用


Run Time Environment (RTE)
- 程式執行時需要的環境 (Compiler Libraries Loader...)
- 提供 System call Interface 

> Program -> API -> RTE -> System call ID -> Kernel -> OS Service -> Return

# Flow

1. User Process issue trap 帶著 system call ID、 parameter
2. 找到對應的System call handler
3. 設為Kernel Mode
4. 根據System call ID找到對應服務
5. 取得Parameter 可以放再 (Register、Stack、Memory)
6. Kernel mode -> User mode

# Parameter Passing of System call (重要 五星) 

{% note danger %}
五星 (重要)
{% endnote %}

**Register** 
- 優 : 簡單 、 速度快 、 不須記憶體
- 缺 : 不適用參數太多情況

**Memory** 
- Register 裡放 Memory Address
- 優 : 可以放很多參數
- 缺 : 需要存取Memory
> Linux 採混合方式；少參數 : Register；多參數： Memory

**Stack**
- User Program Push 參數到Stack、OS 再 Pop

用 Memory 多數！

# System call 種類 (4星 申論)

**Process Control** 
- 管理Process的生命
- create
- load、execute
- wait
- terminate

**File Management**
管理檔案建立、讀寫、刪除

**Device Management**
管理硬體裝置
- request
- release
- read
- write
- get device attributes

**Information maintenance**
管理系統資訊
- get/set time

**Communication**
IPC(interprocess Communication)
- Message Passing 
- Shared Memory

**Protection**
> 如果五大類 此類可不寫
管理權限

# Mechanism and policy

## Mechanism : How to do
- 維持不變、 甚少改變
- ex timer 作為 CPU protection

## Policy : What will be done
- ex. 決定Max CPU time quantum

> 為了增加Flexibility

## 中央常常考
{% note danger %}
What are the purpose of separation of mechanism and policy?
- system are easy to modify
- Flexibility to suit its needs
{% endnote %}

# Original UNIX
UNIX OS 包含兩個分開的部分
- 系統程式
- 核心服務(Run 在Kernel mode)
- 屬於Monolithic kernel type (MicroKernel 的相反詞) 
  - 所有Kernel Service 都在Kernel mode下執行
  - 優缺:與MicroKernel相反
  - 例子:除了MACH 以外都是

# Layered Approach(理論)

Top-Down decompossition
- 上層只可以用下層所提供的功能，下層不能用上層

Bottom-up testing and verifiction

OS可被切好幾層 layer 0: hardware；layer N:user interface

- 優點 : 方便維護、測試、debugging
- 缺點 : 困難去清除

# Monolithic Kernel vs. Microkernel 比較

| 比較項目 | Monolithic Kernel（單核心） | Microkernel（微核心） |
|----------|----------------------------|------------------------|
| 核心概念 | 所有 OS 服務都在 Kernel Mode 執行 | 只保留最基本功能在 Kernel，其餘移到 User Mode |
| Kernel 大小 | 大 | 小 |
| Kernel Mode | 大部分服務都在 Kernel Mode | 只有基本服務在 Kernel Mode |
| User Mode | 幾乎只有 Application | File System、Driver 等非必要服務 |
| IPC | 幾乎不用 Message Passing | 大量使用 Message Passing |
| 效能 | ⭐ 快 | ⭐ 較慢（IPC 開銷） |
| 安全性 | 較低 | 較高 |
| 穩定性 | 某個 Driver 出錯可能造成整個 Kernel Crash | 單一服務失敗通常不會影響 Kernel |
| 維護 | 較困難 | 較容易 |
| 擴充 | 較困難 | 容易新增服務 |
| Porting | 較困難 | 容易移植到新硬體 |
| 代表系統 | Linux、UNIX、Solaris | Mach |

---

# Monolithic Kernel

## 特點
- 所有 Kernel Services 都在 **Kernel Mode** 執行
- 包含：
  - Process Management
  - Memory Management
  - File System
  - Device Driver
  - Network

## 優點
- 執行速度快
- IPC 少
- Context Switch 少

## 缺點
- Kernel 很大
- 維護困難
- Driver 出錯容易造成整個 OS Crash
- 安全性較低

---

# Microkernel

## 核心理念
> Move as much from the kernel into user space.

將非必要服務搬到 User Space。

Kernel 僅保留：

- Process Management
- Memory Management（基本）
- IPC（Message Passing）

其餘：

- File System
- Device Driver
- Network
- GUI

都放到 User Mode。

---

## 優點

- 容易擴充（Extend）
- 容易移植（Port）:因為kernel mode 服務少
- 安全性高（Secure）
- 穩定性高（Reliable）
- Kernel 較小

{% note danger %}
大多於User 萬一Service Fail頂多想當於是一個user process掛而以
{% endnote %}
---

## 缺點

- Message Passing 很多
- User ↔ Kernel Context Switch 增加
- IPC Overhead
- 效能較差

---

# Microkernel 三個基本服務（★★★★★）

1. Process Management
2. Memory Management（基本，不含完整 Virtual Memory）
3. IPC（Inter-Process Communication）

其中：

> IPC 主要使用 **Message Passing**

---

# 考試最愛問

## Q1：Microkernel 為什麼比較慢？

答：

因為需要大量 **Message Passing**。

↓

User Process

↓

Microkernel

↓

File Server

↓

Microkernel

↓

Driver

↓

Microkernel

↓

User

IPC 次數增加，因此效能下降。

---

## Q2：Microkernel 為什麼比較安全？

因為：

大部分服務放在 User Mode。

某個 Driver Crash：

✔ 不會造成整個 Kernel Crash。

---

## Q3：Microkernel 為什麼容易維護？

因為：

Kernel 很小。

新增或修改 Service：

不用修改 Kernel。

---

## Q4：Monolithic 為什麼快？

因為：

所有服務都在 Kernel。

不用一直 Message Passing。

Context Switch 少。

# Modules
現代OS 多提供loadable kernel modules(LKMs)
- Use 物件導向approach
- Each core component is separate
- Each is loadable as needed whithin the kernel

例子 : **Linux** 、 Solaris

# Hybrid (考)

> 選擇

Linux 、 Solaris : monolithic + modular

> 三星 記

MacOS : 核心 **Darwin**
包含
- Mach microkernel
- BSD Unix parts
- I/OKit
- Kernel Extensions(Dynamic loadable modules)

# Virtual Machine

Host -> Hypervisor -> Guest

Host : Run 在 Hardware 上的 OS
Virtual Machine manager(VMM) or Hypervisor : create and run VM by providing interface that is identical to the host
Guest : Process provided with virtual copy of the host

# Implement of VMMs  
Type 0(硬體) 
Type 1(OS Kernel) : 類似一個OS但只提供Virtualization
Type 1(OS Kernel) : 除了一班OS 也提供 VMM 功能模組
Type 2 hypervisor(user mode): 跑在user mode

# 其他VM 變種

> 4 star

1. Paravirtualization
  - 向訪客展示類似硬體但不是的
  - Guest 一定要修改
2. Programing environment virtualization
3. Emulator

> 2 stars

4. Application Containment
- Container
- 不是virtualization
- Docker
- 共用Host Kernel

>平板p.13

# VM Benefits
- 一台電腦可以跑多個OS
- 各個VM 是獨立的
- 作為測試新版本OS的一個 良好附載平台
- Sharing
- 透過網路溝通
- **支援雲端計算與商業發展模式**
  - **Solidation** : 鞏固、合併 ； 把很多低負載伺服器 整合成 大主機 ex. 1台Server跑10個VM
  - **Freeze,suspend**
    - Resume
    - Snapshot : 存點之後壞掉還原回來
    - Clone
  - **Template** : 可以一直產生很多VM
  - **Live Migration (及時移轉)** : VM 正在跑，搬去另一台HOST，User不知道

造就cloud computing 與功能服務(IaaS)

# Cloud Computing

> 依部署方式分類

Public cloud : 透過 Internet 提供給一般使用者付費使用
Private cloud : 由公司自己建立，供公司內部使用
Hybrid Cloud

> 依服務內容分類

Software as a Service(SaaS) : 直接提供可使用的應用程式。: 直接使用成品。
Platform as a Service(PaaS) : 提供開發或執行應用程式所需的平台與軟體環境 :給開發者的平台、 database 、 金流、物流、APIs。
Infrastructure as a Service(IaaS) : 提供基礎硬體資源。: 租虛擬硬體。

# Virtualization Implementation
- 實作困難 : 層次劃分困難
- 效能不佳 : 比真實的慢
- 需要硬體支援

> Host = 真正提供硬體資源的那一端。
> Guest = 跑在虛擬機裡面的 OS 或程式。
> Guest Application
>        ↓
> Guest OS
>        ↓
> Hypervisor / VMM
>        ↓
> Host OS
>        ↓
> Host Hardware

# Java Virtual Machine (JVM)
JVM 只是一個虛擬機器規格
Byte Code 可以跨任何平台執行

> Byte Code : Java Compiler 編譯出來的中間碼
> Java -> javac -> .class (Byte Code)

JVM 三大部分 :
- Class Loader : 載入 class。
- Class Verifier :　驗證 Byte Code 安全
- Java Interpreter : 真正執行 Byte Code

不提供pointer 
提供garbage collection
> Garbage Collection（GC）會自動回收不用的記憶體。
Java Interpreter 程式解譯效率不好，可用Just-In-Time-Compiler(JITC):將Byte code 轉為 target CPU的object code 以利執行