---
title: (作業系統) File System
categories:
  - 作業系統
  - Storage & I/O Systems
tags:
  - 作業系統
abbrlink: cf6482ff
mathjax: true
date: '2025-12-21 13:15:00'
---

# 快速摘要 

1. 檔案與目錄是什麼抽象？OS 提供哪些操作？

    - 檔案本質上是「一串位元組（bytes）」的邏輯視圖

    - 目錄的工作是把 name → 檔案的 metadata（inode/FCB） 對起來

    - 常見操作：create/open/read/write/seek/close/delete/truncate，以及目錄的 search/rename/traverse 等

2. 檔案系統內部要維護哪些關鍵資料結構？（磁碟上 vs 記憶體中）

    - 磁碟上會有：boot block、volume control（superblock 概念）、每個檔案的 inode/FCB、目錄內容

    - 記憶體中會有：mount table、directory cache、system-wide open-file table、per-process fd table

    - 目的都是讓「路徑解析、開檔、讀寫」變快，同時維持一致性

3. 檔案資料實際要怎麼放到磁碟 blocks？空閒空間怎麼管理？（本章超級考點）

    - 配置方法重點比較：

        - Contiguous（快但外部碎裂、成長不易）

        - Linked / FAT（好成長但 random access 差）

        - Indexed / inode（random access 好但有 index overhead）

    - 空閒空間管理：bitmap、linked list、grouping、counting（看誰適合找連續空間、誰 overhead 小）

--- 

# File System 介紹
> File 作業系統幫你做的一層抽象（abstraction）

Why is a File ? 
- File 是 OS 建立與管理的 logical storage unit（邏輯儲存單位）

邏輯視角 vs 物理現實 :
- Logical View :
    - 抽象 for Users : 像是 有名字的一段**連續** bytes
    - 這個“連續”其實是一種錯覺：OS 提供這種 illusion
- Physical View :
    - 一格一格的 sectors / blocks
    - 資料可能 **散落（scattered）**在磁碟各處

File Attributes(MetaData) :
- Identifier（唯一 ID）：系統內部用
- Name（檔名）：人類看得懂的識別
- Type（型態）：內容/格式提示
- Location（位置）：檔案資料放在哪些 blocks
- Size（大小）
- Protection（權限）：誰可以讀/寫/執行
- Time stamps：最後存取、最後修改、建立時間…（時間戳很常用在備份/同步/排序）

## Basic File Operations（基本檔案操作）
OS 提供的 system calls（系統呼叫）

- Create / Open：建立/開檔
- Read / Write：讀/寫
- Repositioning（seek）：移動檔案指標（跳到某位置）
- Close：關檔
- Delete：刪除（移除目錄項目、回收空間）
- Truncate：截斷（清空內容或縮短成某長度，但檔案還在）
- Concatenating：串接（把兩個檔案內容接在一起）

## Open-File Table
> Open File Table 是 **Two-Level Tabel** 由 OS 維護

原因 :
- System-Wide : **共享**共同資訊
- Per-Process : 每個 process 保有自己**獨立狀態**

| 面向              | **Per-Process Open-File Table**（每個行程自己的表）| **System-Wide Open-File Tabl**e（全系統共享表）|
| --------------- | -------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| 範圍 (Scope)      | **單一 process 私有**                                                                                                                | **整個 OS 共享**                                                                                                                            |
| 「一筆 entry」代表什麼  | 這個 process 的「一個 open()」→ 通常對應一個 **file descriptor**                                                                              | 系統層級「某個被打開的檔案物件(open file object)」→ 被多個 process/FD 共同指到                                                                                 |
| 主要存什麼  | **跟 process 相關的狀態**（每個人各自不同）                                                                                                     | **跟檔案本體/開啟實體相關的共享狀態**（大家共用）                                                                                                             |
| 常見欄位        | - **current file pointer / offset**（讀到哪）<br>- **access rights / open mode**（R / RW / append…）<br>- **accounting** / flags（統計、一些開啟旗標） | - **disk location**（檔案在磁碟哪些 blocks / inode 指向）<br>- **file size / access dates**（檔案屬性）<br>- **open count / reference count**（被幾個 FD 指到） |
| 「同一檔案被多人打開」時會怎樣 | 每個 process 都有自己的 entry，所以 **file pointer 可以不一樣**| 會有共享的 system-wide entry，大家的 per-process entry **都指向它**                                                                                  |
| 關係       | **entry 會 pointer 到 system-wide entry**                                                                                          | 被多個 per-process entry 反向引用（靠 open count 計數）                                                                                             |

## Open-File Attributes (MetaData)

1) File Pointer（Per-Process） : 這個 fd 對應的檔案 目前位置（byte offset）
    - 存在 **per-process** open-file table

2) File Open Count（System-Wide）: 這個檔案目前「被開啟」的次數
    - 有多少個 file descriptors 正在 reference 它
    - 存在system-wide open-file table

3) Disk Location（System-Wide）: 檔案資料實際放在磁碟哪裡
    - 存在 system-wide open-file table
    > 這是 OS 把 logical file 轉成 physical disk I/O 的關鍵。

4) Access Rights（Per-Process）: 這個 fd 是用什麼方式開的
    - 存在 per-process open-file table

5) File Types
    - 這些是 給 OS**一個提示**： 用哪個方法打開

---
# Access Method

| 方法              | 你看到的檔案模型       | 常用操作                             | 優點         | 缺點               | 適用情境           |
| --------------- | -------------- | -------------------------------- | ---------- | ---------------- | -------------- |
| **Sequential**      | 從頭到尾線性         | `read_next`, `write_next`,`reset`,`skip(n)`,`rewind(n)`        | 實作簡單、整檔掃描快 | 不能常跳讀、否則無效率| 串流、log 全檔掃描    |
| **Direct Acces**s/Relative Access(Random) | 編號 records，可定位 | `read(n)`, `write(n,data)`, `seek(n)` | 可跳讀、也能做順序讀 | 需要能定位（offset 計算） | 資料庫、查詢、更新特定筆   |
| **Index**           | 先查索引再跳資料(看在哪個block/record)       | index file = `(key,pointer)`  | 用 key **快速查找** | 索引本身要空間/維護、檔案越大 → 索引也越大 → 記憶體/效能壓力       | 大型資料、需要 key 查詢 |

> Sequential 指令 :
>   - `reset` : Set Position = 0 , `skip(n)` = Position += n , `rewind(n)` = Position -= n

---

# Directory Structure

## Partition , volume , Directory

Partition : 把一顆實體磁碟（physical storage device）做邏輯切割後得到的區塊
- Raw partition（裸分割區）: 
    - 沒有 file system 結構
    - 可以「直接用 block/sector 存取」

- Formatted partition（格式化分割區）
    - 你在上面建立 file system

Volume : 一個「可被 OS 掛載（mount）」並且有活躍 file system 的單位
- 包含 file system 的 metadata
- 可以對應 **單一 partition**
- 可以跨 **多顆磁碟**
- 可以是一顆磁碟的**一部分**

Special cases : 
- Legacy devices：像 floppy 有些不能分割

- Modern abstraction：logical volumes 可以把多個 physical partitions 聚合起來

| 名詞        | 是什麼                      | 有沒有 file system     | 能不能 mount    |
| --------- | ------------------------ | ------------------- | ------------ |
| Partition | 磁碟的邏輯切割區                 | 不一定（raw/ formatted） | formatted 才行 |
| Volume    | 可 mount 的 file-system 單位 | 有                   | ✅            |

## Directory Structure
在一個 volume 內 組織檔案、並存放檔案 metadata 的映射資訊

Relationships :
- 情境 A：一般檔案系統
    Disk → Partition → Format → Volume → Mount → Directory/Files

- 情境 B：raw partition（不經 FS）
    Disk → Partition → Raw (swap/DB)

**Directory vs. File** : 

File（檔案） : 包含實際資料內容（actual content）
Directory（目錄）: 是「包含檔案資訊的 nodes 集合」：也就是 mappings
    - 也是檔案的一種（special file）
>兩個都實際存在Disk上

## Directory 結構演進

| 結構            | 解決了什麼         | 主要缺點                        |
| ------------- | ------------- | --------------------------- |
| Single-level  | 最簡單           | 撞名、難找、不能分類、沒隱私              |
| Two-level     | 多使用者隔離        | 不能共享、層級淺                    |
| Tree          | 任意深度、好組織      | 共享不方便                       |
| Acyclic-graph | **共享**（links） | 要避免 cycles                  |
| General graph | 更自由           | cycles → reference count 失效 |

Directory Structures 比較 :

| 類型                                    | 結構                                       | 解決了什麼            | 優點                                  | 缺點                                                                      | 關鍵字                                      |
| ------------------------------------- | ------------------------------------------ | ---------------- | ----------------------------------- | --------------------------------------------------------------------------- | ------------------------------------------ |
| **Single-Level Directory**（單層）| 全系統只有 **一個 directory**，所有檔案都在同一層| —| 最簡單；目錄小時操作快| **4大缺點**：1) **命名衝突**（全域唯一）2) **線性搜尋慢** 3) **不能分類** 4) **沒隱私/存取控制**| Unique filename requirement, linear search |
| **Two-Level Directory**（雙層）| **MFD → UFD**：每個 user 一個 UFD| 修正單層的「多使用者撞名/隱私」 | User isolation；不同 user 可同名；隱私/基本組織↑ | 1) **UFD 內仍像單層**（user 內檔名仍唯一、仍可能線性搜尋）2) **使用者間難共享** 3) 系統檔不好組 4) **層級只有兩層** | **MFD**(管人), **UFD**(管檔)|
| **Tree-Structured Directory**（樹狀）| 任意深度階層；directory 可含 file + subdir| 提供更強組織能力| 分類清楚、可擴充層級（現代最常用基本型）| 樹狀下「共享同一份檔案」不方便（通常得 copy）| **Absolute path**, **Relative path**|
| **Acyclic-Graph Directory**（無環圖） | 允許 **多個 parent**（用 links 共享），但**禁止 cycle** | 解決 tree 的共享痛點    | 共享同一份檔案/目錄，不必複製、省空間| 管理更複雜；要避免 cycle（否則回收/遍歷會炸）| links, shared file, no cycles|

> 補充 : Two Level Directory
>   - MFD (Master File Directory) 、 UFD (User File Directory) 
>   - Path Name = user Name + File name

Path、Links、刪除規則

- Path 類型(Tree Structure)
    | Path 類型           | 定義                 | 長相（Unix）      | 直覺        |
    | ----------------- | ------------------ | ------------- | --------- |
    | **Absolute path** | 從 **root** 開始的完整路徑 | **以 `/` 開頭**  | GPS 完整地址  |
    | **Relative path** | 從**目前工作目錄**開始      | **不以 `/` 開頭** | 從你現在位置怎麼走 |

- Hard link vs Symbolic link (Acyclic-Graph Directory)
    | Link 類型                  | 指向什麼                 | 本體是什麼                 | 刪除目標會怎樣                            | 必背關鍵字                        |
    | ------------------------ | -------------------- | --------------------- | ---------------------------------- | ---------------------------- |
    | **Hard link**            | **同一個 inode**（同一份檔案） | 只是多一個 directory entry | 只要 **link count > 0**，檔案內容不會消失     | link count / reference count |
    | **Symbolic link (soft Link)** | **路徑字串**（path）(Like : 指向其他資料夾) | 一個特殊檔案，內容存 path       | 目標刪掉 → **dangling/broken symlink** | dangling link / broken link  |

- Multiple Absolute Paths & Deletion (Acyclic-Graph Directory)
    | Scenario | 你做了什麼                           | 結果                                | 關鍵判斷                    |
    | -------- | ------------------------------- | --------------------------------- | ----------------------- |
    | 1        | 刪掉其中一個 **hard link**（仍有其他 link） | 只刪 directory entry，資料仍在           | **link count 仍 > 0**    |
    | 2        | 刪掉最後一個 **hard link**            | 檔案內容才真正回收                         | **link count → 0 才刪資料** |
    | 3 (**Dangling Pointer**)        | 刪掉 **symlink** 指向的目標檔           | symlink 還在，但變 **broken/dangling** | symlink 存的是「路徑」不是 inode |

    > Dangling : 只剩pointer 沒有資料了
    >   - 就像是 /bob/reports.txt 指向 /Alice/reports.txt 
    >   - /Alice/reports.txt 被刪除
    >   - /bob/reports.txt 指向 /Alice/reports.txt (**dangling**!)

允許 Cycles 的問題與解法

- General Graph Directory (with cycles)  
    
    | 主題                 | 重點                                                       |
    | ------------------ | -------------------------------------------------------- |
    | 為何會出現 cycle        | 尤其「對 directory 的 symbolic link」很容易形成環                    |
    | 最大考點：Cycle problem | **Reference counting 會失敗**：cycle 內互相指著，count 不會降到 0      |
    | 後果                 | 外部已不可達（unreachable）但不會回收 → **storage leak**（類似 GC 的典型陷阱） |

- Solution 
    
    | 解法                                    | 怎麼做                                  | 優點                  | 缺點            | 一句背法    |
    | ------------------------------------- | ------------------------------------ | ------------------- | ------------- | ------- |
    | **Garbage Collection (Mark & Sweep)** | Mark：從 root 走訪標記可達；Sweep：回收未標記 inode | **能處理 cycles**      | 檔案多時很慢、很耗資源   | 「定期大掃除」 |
    | **Cycle detection on link creation**  | 建 link 前先檢查是否形成 cycle                | **預防 cycle**，不用事後大掃 | 每次建立 link 成本↑ | 「建立時先驗」 |

--- 

# File system Mounting and File Sharing


## File System Mounting（掛載）

Mounting 是什麼？
- **Mounting**：把一個 file system **接到（attach）目錄樹的某個位置**。
- **一定要 mount 才能存取**：因為 OS 需要把不同裝置/分割區的 FS 整合成「同一棵」目錄樹。

為什麼需要掛載？ 
- 系統可能同時有多個 storage devices
- **每個裝置都有自己的 file system**
- OS 必須把它們**整合到統一的 directory tree**，才能用一致路徑存取檔案。

Mount point（掛載點）
- **Mount point**：一個 directory，file system 被 attach 到這個 directory 上（attachment point）。

Mount timing（何時掛載）:

| 類型 | 何時發生 | 例子 |
|---|---|---|
| Boot Time Mounting | 開機時掛好 | 系統分割區：`/`, `/home`, `/usr` |
| Automatic Runtime Mounting | 插入後自動掛載 | USB、外接硬碟 |
| Manual Runtime Mounting | 需要手動 mount | 網路 FS：NFS、CIFS/Samba |

## File Sharing on Multiple Users（多使用者共享與權限）

User identity：每個 user 的兩個 ID
- **UID (User ID)**：使用者唯一識別（例：alice = 1001）
- **GID (Group ID)**：主要群組識別
- `id alice` 可看到 uid/gid 與所屬群組列表（groups）

ID association:
- **每個 process 都帶著 user 的 UID/GID 在跑**
- 所有檔案操作都會用這些 ID 做權限檢查（OS check permissions）

每個檔案的權限三類 : 
- **Owner**（檔案擁有者）
- **Group**（檔案群組內成員）
- **Others**（其他所有人）
- 常見 `ls -l` 會看到 `-rwxr-xr--`（分成 owner/group/others 三段）

Key rules of privileges（權限規則）:
- **Owner controls all permissions**（擁有者可設定權限）
- **Group members have access, not control**（群組通常是存取者，不一定能管理）
- **Only root can change ownership**（只有 root 可改擁有者）


##  Access-Control List (ACL) — Per-User ACL（理想但不實際）

**ACL**：列出哪些 users/groups 能存取檔案，以及如何存取（r/w/x）。

Approach 1：Per-User ACL（每檔列出每個 user）
- 每個檔案維護一串 `(user, permissions)` 條目  
  例：`project.txt`  
  - `alice: rwx (7)`  
  - `bob: rw- (6)`  
  - …

- Advantages（優點）
    - **Extremely flexible**：可對每位 user 指定精確權限
    - **Fine-grained control**：細粒度控制

- Problems（問題）
    - 使用者數量大 → ACL 可能**無限長**
    - **Storage overhead**：大量檔案 × 大量使用者 → 不實際
    - **Management nightmare**：新增 user 可能要更新很多檔案 ACL
    - **Performance**：檢查權限要掃長 list


## ACL — Three-Class System（Unix 標準：三類固定）

Approach 2：Three-Class System（把 ACL 縮成固定三類）
把權限縮成三個固定類別（每檔固定大小）：
1. **Owner**：一個特定 user
2. **Group**：一個特定 group
3. **Others**：其他所有人

➡️ 每個檔案固定只有 **3 條 ACL entries**（fixed size）

RWX permissions：9 bits（超常考）
- 3 classes × 3 permissions（r/w/x）= **9 bits**

| 類別 | 權限例 | 十進位 | 二進位 | 解釋 |
|---|---|---:|---:|---|
| Owner | `rwx` | 7 | 111 | 讀/寫/執行 |
| Group | `rw-` | 6 | 110 | 讀/寫 |
| Others | `r--` | 4 | 100 | 只讀 |

- Advantages（優點）
    - **Fixed size**：每檔固定 9 bits
    - **Efficient**：權限檢查 **O(1)**（不隨 user 數量成長）
    - **Manageable**：新增 user 主要靠加入 group，不必改遍所有檔案
    - **Sharing through groups**：要分享給特定人 → 建 group、加人、設 group 權限


## File Protection（檔案保護：安全 + 可靠性）

File protection
1. **Unauthorized access（Security / protection）**：不該存取的人來存取  
2. **Physical damage/loss（Reliability）**：硬體壞掉導致資料遺失

Protection from Improper Access（Security）:
- 防未授權存取：定義 **WHO can access** 與 **HOW**
- 常見方法：
  1. **ACL**：定義誰能做什麼
  2. **Password protection**：需密碼才能存取
  3. **Encryption**：加密，無 key 讀不懂內容

Protection from Physical Damage（Reliability）
- 防資料遺失（hardware failure）
- 常見方法：
  1. **RAID**：多磁碟冗餘（redundancy）
  2. **Backups**：備份到別處
  3. **Versioning / Snapshots**：保留多個時間點版本

Security vs Reliability（快速分辨）
| 類型 | 解決問題 | 常見手段 |
|---|---|---|
| Security | 防未授權存取 | ACL / 密碼 / 加密 |
| Reliability | 防硬體故障與資料遺失 | RAID / Backup / Snapshot |

---

# File-System Structure


## File-System Structure（Sector → Block → File ops）

I/O Transfer Hierarchy
- **Sector（硬體最小單位）**：常見 512B（新磁碟可能 4KB）
- **Block（OS/FS 傳輸與配置單位）**：常見 4KB（≈ 8×512B sectors），可含 1+ sectors
- **File system operations（邏輯抽象）**：open/read/write… 由 OS 轉成 block I/O

Storage Units

| 名詞 | 是什麼 | 典型大小 | 重點 |
|---|---|---:|---|
| Sector | 最小物理單位 | 512B / 4KB | hardware 規格 |
| Block | FS 配置/讀寫單位 | 常見 4KB | OS 以 block 管理空間與 I/O |
| Clustering | 多 blocks 合併 | 依 FS | 對大檔降低指標/管理開銷 |

Multiple File System Support & VFS
- OS 同時支援多種 FS（Windows：NTFS/FAT32/exFAT/ReFS；Linux：ext4/Btrfs/XFS…）
- **VFS(Virtual File System)**：抽象層，讓上層 API 不依賴 FS 類型

Two-layer design problem

| 層 | 目標 |
|---|---|
| User Interface Layer | 給 user/app 一致 API |
| Physical Storage Layer | 高效率磁碟 block 管理 |

## Layered FS Architecture

1) Application Programs
- 例：`read("file.txt", buffer, 100)`，透過 **system calls** 要求檔案操作

2) Logical File System（Locate FCB, check permissions）
- 管 **metadata、directory structure、file naming**
- 提供 API：open/close/read/write
- 維護 **FCB**（inode/MFT entry 概念）、security/permissions
- 關鍵：directory management（路徑解析）

3) File-Organization Module（Translate logical→physical blocks）
- **Logical blocks → Physical blocks**
- 管 free space、disk allocation
- 實作 allocation methods（contiguous/linked/indexed）
- key structure：**FAT 或 inodes**

4) Basic File System（Request blocks from cache/disk）
- 發出通用 block I/O 請求給 driver
- 管 **buffers/caches**
- 處理 **block I/O scheduling**
5) I/O Control（Device Drivers）
- 硬體特定 driver code
- 高階命令 → 硬體指令
- 管 interrupts、DMA transfers
6) Physical Devices
- 真硬體（HDD/SSD/optical…）

---

# File System Implementation

## On-Disk File System Structures（磁碟上的四大結構）

1) **Boot Control Block（per partition）**
- partition 第 1 個 block（block 0），放 boot code；空的＝不可開機

2) **Partition Control Block（per partition）**
- 管整個 partition/volume：total blocks、block size、free block/FCB counts、root dir location…
- 例：UFS/ext4 superblock；NTFS MFT header（概念）

3) **File Control Block FCB（per file）**
- 每檔 metadata：permissions/owner、size、timestamps、**pointers to data blocks**
- 例：inode（ext4/UFS）、MFT entry（NTFS）

4) **Directory structure（per FS）**
- 階層組織：通常是 name → inode 的對應（或 B+tree 等）


FCB 內容（常見欄位）
- permissions、dates（create/access/write）、owner/group/ACL、size、data blocks/pointers

Partition 內部概念布局
- Boot block（optional）→ Partition control block → Directory/FCB lists → Data blocks

> FCB 是「檔案身分證 + 指到資料」  
> partition 由控制資訊區（metadata）+ 大量 data blocks 組成

## In-Memory File System Structures（RAM 內的結構）

In-memory **Partition Table**（**追蹤所有已掛載 volumes**）
- mount point、FS type、superblock copy pointer、device id、mount flags（ro/noexec…）
- mount/unmount 時更新

In-memory Directory Cache（加速路徑解析）
-  **cache 最近用的 directory entries**
- 避免重複 disk read；常用 **LRU** 替換

System-wide Open-File Table（全系統共享）
- **FCB copy**、current offset、open count、disk location、access mode flags
- 第一次 open 建；open count=0 刪

Per-process Open-File Table（每行程一份）
- fd/handle（整數索引）→ 指到 system-wide entry
- per-process offset、此 process 的 access permissions
- **多個 fd 可以指向同一個 system-wide entry**

## File Creation Procedure（建立檔案流程）

1) **Allocate FCB**
- 找 free inode/MFT entry，初始化 metadata（timestamp、permissions、owner/group、size=0、data pointers=null）

2) **Load Parent Directory**
- 讀父目錄到 RAM；檢查寫權限與檔名重複

3) **Update Directory Structure**
- 加 entry：filename → FCB pointer  
- 更新父目錄 metadata（mod time、entry count）

4) **Synchronization options（何時寫回磁碟）**
- immediate / deferred（close/sync）/ journaled（先 log 再 commit）

5) **File visibility**
- 立刻出現在 listing、可在 close 前操作；但實際 disk write 可能延後

> 建檔：**配 FCB → 更新父目錄（name→FCB）→ 同步策略**  

---

#  VFS（抽象層 + 四大物件）

VFS 的角色
- 介於 applications（system calls）與各 FS 之間的抽象層  
- 提供不同 FS 的統一介面（unified interface）

四大物件（OOP 設計）
| 物件 | 代表 | 記法 |
|---|---|---|
| Superblock | **filesystem** instance | 這個掛載的 FS |
| Inode | file/dir **metadata** | 檔案/目錄身分證 |
| Dentry | **Directory** entry (name → inode) | 目錄中的一筆 |
| File | **open file** instance | 一次 open 的實體 |

- 物件通常有 operations table（function pointers）→ 不同 FS 可掛不同實作

- VFS 讓 syscall 介面一致  
- 四物件：**superblock / inode / dentry / file**（搭 operations table）

## VFS（好處 + 流程 + 多型）

好處
- 同一套 syscall 介面對所有 FS
- App 不必知道 FS type
- 易加入新 FS；同機可混用多 FS

運作流程（open）
1. App：open("/path/file")
2. VFS：parse path、找 mount point
3. VFS：決定 FS type
4. VFS：呼叫 FS-specific routine（ext4_open/ntfs_open/nfs_open…）
5. 回傳結果

關鍵洞察：Polymorphism（多型）
- **同介面**（open/read/write）  
- **不同實作**（ext4/ntfs/nfs 各自實作）

- VFS 的精髓：**同介面，不同實作（polymorphism）**  
- open 時先找 mount point，再 dispatch 到對應 FS

## Directory Implementation（線性表 vs 雜湊）

目錄要有效率支援的操作:
- Search / Insert / Delete / List / Rename / Traverse  
（取捨：Simplicity vs Performance）

Implementation 1：**Linear List**
- array 或 linked list，entry 順序排列
- entry：filename（可變長）+ inode/file ID 指標
- 優：簡單  
- 缺：search O(n)，大目錄效能差

Implementation 2：**Hash Table**
- hash(filename) → index
- bucket 指向同 hash 的 entries（linked list）
- 平均 search/insert/delete O(1)
- 缺：table size 固定（碰撞與空間取捨）

| 作法 | Search | Insert/Delete | 優點 | 缺點 |
|---|---:|---:|---|---|
| Linear list | O(n) | 偏慢 | 最簡單 | 大目錄很慢 |
| Hash table | avg O(1) | avg O(1) | 查找快 | fixed size、**碰撞處理** |

---

# File Allocation Methods

##  File Allocation Methods（總覽）

**磁碟上的 blocks 要怎麼分配給檔案？**  
檔案系統必須把「檔案（連續的 bytes）」映射到「磁碟 blocks（離散位置）」。

三種主流策略：

1. **Contiguous allocation（連續配置）**：同一個檔案放在連續 blocks  
2. **Linked allocation（鏈結配置）**：檔案 blocks 散在各處，用指標串起來  
3. **Indexed allocation（索引配置）**：用一個 index block 收集「所有 data block 指標」

## Contiguous Allocation（連續配置）

做法
- 每個檔案佔用 **一段連續 blocks**（consecutive blocks）
- 目錄/metadata 只要存：
  - `start_block`（起始 block）
  - `length`（連續長度，或檔案大小）

優點
- **最少 seek**（磁頭移動少）→ 效能很好
- 同時對 **sequential / random access** 都快  
  - random：`start + offset` 直接算出來

問題
1) **External fragmentation（外部碎裂）**  
- 空洞散落在磁碟上，雖然總空間夠，但找不到足夠大的「連續洞」

→ 解法：**Compaction（磁碟整理/搬移）**  
- 把檔案搬來搬去，把空洞集中成大洞（但成本高、很麻煩）

2) **File growth（檔案成長）**  
- 檔案變大時，後面不一定還有連續空間可擴

→ 解法：**Extent-based allocation（用多段連續區塊）**

## Extent-Based File System（Extent：多段連續）
> 「檔案不是一定要 1 段連續，而是允許多段連續的片段」。

Extent 是什麼？
- **Extent = 一段連續 blocks 的區間（contiguous chunk）**
- 一個檔案由 **一個或多個 extents** 組成  
- 典型格式：`(start_block, length, next_extent_ptr)`
  

這是「改良版 contiguous」
- 每個 extent 內部仍是連續的 → seek 少、順序讀取快  
- 但檔案成長時 **可以再掛一段新的 extent**，不用整個搬家

優點（相對純 contiguous）
- 檔案更容易成長（append 友善）
- 減少 compaction 的需求
- 大部分檔案仍維持不錯的連續性 → **常見情境效能好**
- 空間利用更彈性

缺點
- **仍可能有 internal + external fragmentation**
  - internal：最後一段 extent/cluster 可能沒填滿 (ex. block size = 4 , data size =10 ,浪費2)
  - external：extent 之間仍可能被切成碎片
- 若 extents 變很多，**random access 成本上升**（要先定位在哪個 extent）

##  Linked Allocation（鏈結配置）

做法
- 一個檔案是「磁碟 blocks 的 linked list」
- 每個 data block 末端放 **next pointer** 指向下一個 block  
- Directory 只需存：`first_block`、`last_block`（或 start/end）

優點
- **No external fragmentation（沒有外部碎裂）**
  - 因為不要求連續，有空就能放
- 檔案成長容易（直接把新 block 掛到尾巴）
- 配置邏輯簡單
- append 效率不錯（有 last pointer 的話）

缺點
- **Sequential access 可能變慢**：blocks 散在各處 → seek 多
- **Random access 很糟**：第 n 個 block 必須從頭走 n 步
- **Pointer overhead**：每個 block 都要留 pointer，data 可用空間變少
- **Reliability**：指標壞掉 → 後面整段檔案可能找不到

> Linked allocation：**不怕外碎裂、好成長**  
> 但 blocks 分散 → seek 多；random access 幾乎悲劇；還有 pointer/可靠度問題


## Optimization for Linked Allocation：Clustering（把 N 個 blocks 當一個節點）

核心想法
- 不是「1 block = 1 node」  
- 而是「**N blocks = 1 node（cluster）**」，cluster 裡面連續  
- 指標只放在 cluster → 指到下一個 cluster

例：cluster size = 4 blocks  
- cluster = blocks 1,2,3,4（連續）→ 指標指向下一個 cluster

好處
- **Fewer seeks**：1 次 seek 讀一整個 cluster（而非每個 block）
- **順序讀更快**（cluster 內連續）
- **Less pointer overhead**：每 N blocks 才需要 1 個指標

缺點
- **Internal fragmentation**：最後一個 cluster 可能沒填滿 → 浪費
- 配置/回收更複雜（一次要處理 N blocks）


## FAT（File Allocation Table）File System

FAT 是什麼？
- FAT 可以視為「**把 linked allocation 的指標從 data block 內，搬到一張集中表**」。

- 傳統 linked：指標 **在 data block 裡**
- FAT：指標 **在 FAT 表裡**

目錄 entry 只要存 start block。  
接下來要找下一塊，就查 FAT：`next = FAT[current]`

重要特性：FAT 可以快取在記憶體
- 若 FAT 表（或其熱區）被 cache 在 RAM：
  - 走鏈結就不必一直讀磁碟上的指標
  - 找下一塊更快、更穩定

## Indexed Allocation（索引配置）

做法
- 把「所有 data block 指標」集中放到 **一個 index block**
- 檔案 = **Index block + Data blocks**
- Directory 存：這個檔案的 **index block 位置**

優點
- **Efficient direct / random access**
  - 要第 i 塊：直接看 index[i]
- **No external fragmentation**
  - 不要求 data blocks 連續
- 建檔容易（先配 index block，再慢慢補資料）

缺點
1) **Index block 的空間成本**
- 每個檔案都要一個 index block（小檔案也要）→ overhead

2) **Index block 大小問題**
- index block 要能放多少 pointers？
- 檔案太大時，一個 index block 不夠放

## Indexed v.s FAT
>Indexed：每檔一張索引表 → 隨機快
>FAT：全碟一張 next 表 → 走鏈，隨機慢但實作簡單

| 面向                | Indexed Allocation（索引配置）                            | FAT（File Allocation Table）                 |
| ----------------- | --------------------------------------------------- | ------------------------------------------ |
| 指標存放位置            | **每個檔案**一個 index block（或 inode 的 pointer 區）         | **整個系統**一張 FAT（全域表）                        |
| “第 i 個 block” 怎麼找 | 讀 index block，直接用 `index[i]` 指到 block → **接近 O(1)** | 從起點一路 follow “next” 指標走 i 次 → **O(i)**（鏈式） |
| 讀取效能              | 隨機存取很強（尤其大檔案）                                       | 順序讀還行；隨機跳很慢（要走鏈）                           |
| 空間開銷              | 每個檔案要多一個 index block / 指標區（小檔案也要負擔）                 | FAT 佔用磁碟一大塊（全域），通常還要快取到 RAM                |
| 可靠性/損壞影響          | 壞一個檔案的 index → **影響那個檔案**                           | FAT 壞/被破壞 → **可能影響整個檔案系統的鏈結資訊**            |
| 最大檔案大小限制          | 受 index block 可放的指標數限制（可用 multi-level index 擴充）     | 受 FAT entry 數量與設計限制（以及 cluster size 等）     |
| 本質上像什麼            | 「目錄（array）一次列出所有 blocks」                            | 「linked list，但把 next 指標集中放在表裡」             |


## Index Block Size：index 不夠大怎麼辦？ (碩士會考歐) 

三種解法：

1) **Linked scheme**
- 把多個 index blocks 串成 linked list  
- 優：概念簡單  
- 缺：random access 還是要走鏈結（第 k 個 index block 要走 k 次）

2) **Multilevel index**
- index 的 index（階層化）
- 例：第一層指向第二層 index blocks，再指到 data blocks  
- 優：可支援超大檔案  
- 缺：多幾次間接存取（多一次讀 index）

3) **Combined scheme（Unix inode）**
- **Hybrid：Direct + Indirect pointers**
- 小檔案：直接指標就夠（快）  
- 大檔案：再用 single/double/triple indirect 擴展（可很大）

---

## Combined Scheme（Unix inode）詳細計算

已知：
- block size = **4KB**
- pointer size = **4 bytes**
- pointers per block = 4KB / 4B = **1024 pointers**

inode（或類似結構）包含：
- **Direct blocks：12 個**
- **Single indirect：1 層**
- **Double indirect：2 層**
- **Triple indirect：3 層**

可支援容量計算

| 類型 | 可指到的 data blocks 數 | 可支援資料量 |
|---|---:|---:|
| Direct (12) | 12 | 12 × 4KB = **48KB** |
| Single indirect | 1024 | 1024 × 4KB = **4MB** |
| Double indirect | 1024² = 1,048,576 | 1,048,576 × 4KB = **4GB** |
| Triple indirect | 1024³ = 1,073,741,824 | 1,073,741,824 × 4KB = **4TB** |

Total max file size：
- **48KB + 4MB + 4GB + 4TB ≈ 4TB**（因為 4TB 遠大於其他）

##  總比較表

| 方法 | Metadata/指標放哪 | Sequential | Random | 碎裂 | 成長性 | 典型問題 |
|---|---|---|---|---|---|---|
| Contiguous | (start, length) | ✅ 很快 | ✅ 很快 | ❌ 外碎裂 | ❌ 難 | 需要 compaction、成長麻煩 |
| Extent-based | 多段 (start, length) | ✅ 快 | ◑ 看 extent 數 | ◑ 仍可能有內/外碎裂 | ✅ 好 | extents 太多時 random 變貴 |
| Linked | 指標在 data block | ◑ blocks 分散會 seek 多 | ❌ 很糟 | ✅ 無外碎裂 | ✅ 很好 | pointer overhead、可靠度 |
| Clustered linked | 指標每 N blocks 一個 | ✅ 比 linked 好 | ❌ 仍差 | ✅ 無外碎裂 | ✅ 好 | 內碎裂（最後 cluster） |
| FAT | 指標集中在 FAT 表 | ✅ 不錯（可 cache） | ◑ 比 linked 好 | ✅ 無外碎裂 | ✅ 好 | FAT 表可能很大 |
| Indexed | 指標集中在 index block | ✅ 不錯 | ✅ 很好 | ✅ 無外碎裂 | ✅ 好 | 每檔案 index block 成本、index 擴展 |
| inode（combined） | direct + 多層 indirect | ✅ 小檔超快 | ✅ 大檔也可 | ✅ 無外碎裂 | ✅ 很好 | 多層間接會增加存取次數 |


---

# Free-Space Management

## Free Space Management

核心問題：OS 必須追蹤哪些 disk blocks 是 free
當你 `create/write` 檔案時，檔案系統要去「挑」可用 blocks；  
當你 `delete/truncate` 檔案時，要把 blocks 釋放回 free pool。

Free-Space List（空間表）不一定真的是 list
追蹤 free blocks 的資料結構很多種，常見 4 類：

- **Bit vector (Bitmap)**：每個 block 用 1 bit 表示 free/used
- **Linked list**：把所有 free blocks 串起來（像 linked allocation）
- **Grouping**：一個 free block 內存一批 free block 位址（像 linked-index）
- **Counting**：用 `(start_block, count)` 表示一段連續 free blocks（像 contiguous/extent）

free space 管理和檔案配置很像
- 你用什麼方式配置檔案 blocks（linked / indexed / contiguous）  
  常常就會用相似方式去管理 free blocks（linked list / grouping / counting）。


## Method 1: Bit Vector (Bitmap)

做法
- **1 個 disk block 對應 1 個 bit**
  - `bit[i] = 1` 表示 block[i] **free**
  - `bit[i] = 0` 表示 block[i] **occupied**

優點
- **空間非常省**：1 bit / block（overhead 最小）
- **實作簡單**：用 bit 操作（AND/OR/shift）就能標記/
缺點
- **找 free block 需要掃描（linear scan）**
  - 即使 bitmap 在 cache/記憶體中，仍可能要找很久才遇到 1
- **超大磁碟不適合整張常駐記憶體**
  bitmap 太大放不下 RAM → 必須分段載入/快取> 小補充（幫你背考點）：  
> Bitmap 的強項是「**省空間、好做 bit 操作**」，弱點是「**找洞要掃**
✅ **本頁重點**
- *1 bit / block**（省空間、易實作）  不保證連續

# Method 3: Grouping & Method 4: Counting

Method 3：Grouping（把一批 free block 位址塞進一個 free block）
- 概念：不要每個 free block 都只存 1 個 next 指標  
  而是「**一個 free block 裡存 N 個 free blocks 的位址**」
- 每個 group block 內容通常是：
  - `N` 個 free block addresses（例如 100 個）
  - `1` 個 pointer 指向下一個 group block

**優點**
- 一次拿到一大批 free blocks → 比 linked list 少走很多步（I/O/指標追蹤少）
- 適合大量配置（一次 allocate 多個 blocks）

**缺點**
- group block 本身也要管理、格式較複雜
- 還是不好「直接保證連續洞」（只比較容易拿到很多空的）

Method 4：Counting（記錄連續 free 區段）
- 概念：很多 free blocks 通常是「連續的一段」  
  所以用 `(first_block, count)` 表示一段連續 free blocks
- 例如：`(200, 50)` 表示 blocks 200~249 都是 free

**優點**
- 非常適合 **contiguous / extent-based** 配置  
  因為你本來就想找「連續的一段」
- 記錄很省（用區段描述取代列舉每個 block）

**缺點**
- 若 free space 很零碎（嚴重碎裂），就會有很多小段要記  
  metadata 會變大、管理也更麻煩


## 總比較表

| 方法 | 怎麼表示 free space | 優點 | 缺點 | 特別適合 |
|---|---|---|---|---|
| Bitmap | 1 bit / block（1=free, 0=used） | 超省空間、bit 操作快 | 找洞常要掃描；超大盤不易全放 RAM | 想快速判斷某 block 是否空 |
| Linked list | free blocks 串成鏈 | 幾乎零額外空間、好實作 | 難找連續洞；挑洞效率差 | 只求「拿得到某個空 block」 |
| Grouping | 一個 free block 存 N 個 free block 位址 + next | 一次取得一批空 blocks，少走很多步 | 結構較複雜；不保證連續 | 一次配置很多 blocks |
| Counting | (start, count) 表示連續 free 區段 | 很省、連續洞好找 | 碎裂嚴重時段數多、管理難 | contiguous / extent-based 配置 |

---

> 作者我要期末考了 要完蛋啦!!!
> 之後有空研究所我才會細分 現在就先大概就好了歐!!!