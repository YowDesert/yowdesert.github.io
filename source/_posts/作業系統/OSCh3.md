---
title: OS-Hardware Resource Protection
categories:
  - 作業系統
  - 第三章
tags:
  - 作業系統
abbrlink: 7fac9819
mathjax: true
date: '2026-8-04 15:20:00'
---

# 基礎建設

## Dual Mode operation

為了安全區分兩種系統運作模式

  - the execution of OS code
  - user-defined code

**User Mode**
  - 不允許執行特權指令 

**Kernel mode**
  -  又叫做 supervisor mode、 system mode、priviledged mode、monitor mode(現已removed)
  - 可RUN Priviledged instruction、system call


> 需要額外HW:Mode bits 去操控 mode

OS要求服務要轉成 *Kernel mode*
系統一開機設為Kernel mode -> OS load and start 使用者應用程式 in user mode
*Trap* 、 *interrupt* 、 *system call* 可轉為 Kernel mode


## Privileged instruction

可能引起重大危害的指令 -> 設為特權指令
只允許Kernel mode 跑
如果在User mode 則視為 illegal and traps it to the OS

例如 : 
- switch to kernel mode 
- I/O instruction
- Set/Modify the base/limi register
- Set/Modify Timer的值
- Disable interrupt
- Clear Memory

# 硬體資源保護

## I/O protection
防止User process 誤用I/O 、 降低I/O Device 複雜度
User process只能委託Kernel

## Memory protection
防止User非法存取其他Process or kernel memory area
- 用Base/Limit Register

## CPU protection
防止CPU被USER Process長期占用不還給Kernel
- Use Timer解決
