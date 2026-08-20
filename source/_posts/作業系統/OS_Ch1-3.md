---
title: OS-CH1-3 題目注意
categories:
  - 研究所題目
  - 作業系統
  - 第1~3章
tags:
  - 作業系統
abbrlink: 841a1f01
mathjax: true
date: '2026-8-05 16:00:00'
---
# 第一章 到 第三章 題目提醒

1. Bootstrap program 存在 **ROM** 中
> ROM : 唯讀記憶體 : 資料寫入後就不能隨便更改，關機後資料也不會消失。

2. Flash memory is another form of **electronic disk**.

3. 不是所有系統都同時有 **CLI** and **GUI**

4. MultiProcessor 優點
  - 產能上升
  - Economy of scale
  - 增加Reliability
  - 不保證提高 **CPU utilization**

5. Major Version of UNIX 支持 Soft Real Time System

6. Hard Real Time 依賴 **RAM、ROM**，**避免使用Secondary Storage**(Disk、Paging、Virtual Memory)

7. Industrial control 使用 Hard-real-Time

8. masked interrupt , 如果有正在處理的仍然會先通知 CPU , 但不會先處理

9. DMA 不適用 **character Devices**(Keyboard、Mouse)，一次只有幾 Bytes。

10. ~~Software~~ Hardware interrupt also goes through interrupt controller.

11. Thermostat 是 Hard Real Time 因為 控制系統 慢幾秒可能會損壞

12. DMA 適用 High-Speed Devices (Disk)
> Disk 相對於鍵盤、滑鼠是快的
> DMA : 適合高速或大量、區塊式 I/O 傳輸，例如 Disk、SSD、Network。
> Polling／Interrupt I/O 較適合低速、少量資料裝置，例如 Keyboard、Mouse。

13. I/O completion、clock interrupt 是 Hardware Interrupt

14. DMA 與 依靠中斷驅動的輸入輸出相比的優點 缺點?
  - DMA 自己負責 Device 和 Memory 之間的傳輸 不須 IO Pooling 可以提升CPU utilization
  - DMA 適用 block-transfer 中斷頻率低，無須耗費太多時間在Interrupt
  - DMAC 增加硬體複雜度，而且會與CPU 搶 Memory ， 所以需要Cycle Stealing 技術解決，會拖累CPU對memory的存取時間與速度 

15. 兩個**memory-mapped device** 可以直接DMA傳輸，不用經過RAM。
  > Device A buffer -> DMA -> Device B buffer

16. DMA 技術需要利用一個額外的專用硬體 DMAC

17. User Program 只看到 Virtual Memory

18. Deadlock 有 Deadlock Prevention/Avoidence/Detection/Recovery/Ignore

19. Copying String 通常不需要System Call
> 可視為一般記憶體內複製

20. Displaying a windows on screen 有至少一個System call

21. Iterrupt vector 存ISR的程式入口位置

22. Dual Mode 有幾個必要
  - 有mode bit
  - 有Priviliedge instruction
  - OS 在高權限模式執行

23. 如果發生System Call
  - 不一定application 會暫停
  - Os Service Routine 一定 ， 因為System call 就是呼叫 OS Service Routine

24. MacOS : **Mach** 、 BSD Unix Part(Monolithoc) 、 I/O kit、Dynamic Loadable Module

25. Guest 必須**修改**才能跑在Para-Virtualization

26. Message Passing
  - 放CPU Register
  - Memory Block : 放Parameter Block 的 起始位置
  - Stack : Parameter 被 Push 進去 Stack，OS PoP。

27. Para-virtualization 提供與 Guest **相似但不完全一樣**的環境。

28. SaaS : 給應用程式 ; PaaS : 給開發/執行平台 ; IaaS : 給虛擬硬體資源

29. **Module** : Linux、Solaris

30. Linux : Monolithic with Dynamically Loadable Drivers

31. Shared memory 建立與設定需要依靠OS

32. Container 沒有Guest OS ，只有
  - App
  - Librabry
  - Runtime
  - Dependencies

33. Container 最大的特色是 : 共享Kernel

34. Interrupt Vector 是一個位址，指向Interrupt Handler(ISR)
