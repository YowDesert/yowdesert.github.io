---
title: (資料庫系統) Basic SQL (Structured Query Language)
categories:
  - 資料庫系統
  - Basic SQL
tags:
  - 資料庫系統
abbrlink: 4f5b5fb4
mathjax: true
---

# 簡短說明：
- 主題：Basic SQL
- 本章重點：
    - SQL 的由來與「關聯模型」差異
    - SQL 三種語言：DDL / DML / DCL
    - Schema / Catalog / CREATE TABLE
    - 資料型態 & Domain
    - 各種 Constraints（PK / FK / UNIQUE / NOT NULL / CHECK / DEFAULT）
    - 基本查詢：SELECT-FROM-WHERE、DISTINCT、集合運算
    - INSERT / DELETE / UPDATE 修改資料

---
# 1. Basic SQL & 關聯模型差異
## 1-1 SQL的由來
 - 起源：從 tuple calculus（關聯謂詞演算）發展出語言 SQUARE → SEQUEL → SQL

 - SQL 原本是 SEQUEL to SQUARE 的縮寫，後來因版權改成 SQL（Structured Query Language）
→ 目前都習慣叫「Structured Query Language」。

## 1-2 名詞對照（數學 vs 實作）

理論的關聯模型使用比較「數學」的名稱：

| 關聯模型 | SQL 實務名稱 | 中文 |
|---------|--------------|------|
| Relation | Table | 資料表 |
| Tuple | Record / Row | 紀錄/列 |
| Attribute | Field / Column | 欄位/行 |

 - SQL可以不需定義主鍵
 - SQL資料表可以有兩筆序列值一模一樣
 - 資料表中的紀錄是有次序的

---
# 2. SQL 提供的三種語言

## 2-1 DDL（Data Definition Language）
用途：定義資料庫結構 & 各種物件
 - 物件：Database、Table、View、Index、Schema、Domain、Stored Procedure、Trigger…等
 - 常見指令:
   - CREATE：建立物件（table、schema、view、index…）
   - ALTER：變更物件（加欄位、改 constraint…）
   - DROP：刪除物件

## 2-2 DML（Data Manipulation Language）
用途：操作資料表的資料

 - 常見指令：
   - INSERT：新增資料
   - UPDATE：修改資料

## 2-3 DCL（Data Control Language）
用途：權限控制 & 交易控制
 - 常見指令：
  - GRANT：給使用者權限
  - REVOKE：收回權限
  - COMMIT：提交交易（真的寫進 DB）
  - ROLLBACK：異常時回復到交易開始前的狀態
  - DELETE：刪除資料

>小抄：
DDL：建東西 → CREATE / ALTER / DROP
DML：改資料 → INSERT / UPDATE / DELETE
DCL：權限＋交易 → GRANT / REVOKE / COMMIT / ROLLBACK

---
# 3. Schema & Catalog & CREATE TABLE
## 3-1 Schema 概念
- SQL schema：
  - 有一個 schema 名稱
  - 有一個授權識別碼（帳號 / 使用者），表示誰是 owner
  - 裡面包含各種元素：tables、constraints、views、domains 等
- 建立語法：

> Each statement in SQL ends with a semicolon(;) 

```SQL
CREATE SCHEMA COMPANY AUTHORIZATION 'Jsmith';
```
→ 建一個 schema 叫 COMPANY，擁有者是 Jsmith。
## CREATE TABLE：定義資料表
- 目的：定義一個 base table（基礎關聯）
 - CREATE TABLE 建立的是 base table：
   - 真正會在磁碟上建立檔案來存 row
 - 若是 CREATE VIEW 則是 virtual relation（view）：
   - 不對應實體檔案，只是查詢的視圖

```SQL
CREATE TABLE EMPLOYEE (
    Fname      VARCHAR(15),
    Minit      CHAR(1),
    Lname      VARCHAR(15),
    Ssn        CHAR(9),
    Bdate      DATE,
    Address    VARCHAR(30),
    Sex        CHAR(1),
    Salary     DECIMAL(10,2),
    Super_ssn  CHAR(9),
    Dno        INT,
    PRIMARY KEY (Ssn)
);
```
>欄位：欄位名 資料型態 [NOT NULL] [DEFAULT ...] [CHECK(...)]
下方可以定義：PRIMARY KEY、UNIQUE、FOREIGN KEY 等 constraint

![alt text](/img/Crete.png)
## 3-3 Catalog 概念
 - Catalog : 一個SQL environment中，很多個schema的集合。

# 4 資料型態 & Domain
## 4.1 數值型態（Numeric Types）

###  整數型態
- `INT`、`INTEGER`：整數  
- 無小數位、不可表示浮點

###  小數型態（定點數）
- `DECIMAL(i, j)`、`NUMERIC(i, j)`、`DEC(i, j)`
  - `i`：總位數（precision）
  - `j`：小數位數（scale）

**例：**
- `DECIMAL(3,1)` → 例如 `12.5`  
- `DECIMAL(5,2)` → 例如 `123.45`

> 若未指定 `(i, j)`，則使用 DBMS 預設值。

---

## 4.2 字串型態（Character Types）

### 固定長度字串
```sql
CHAR(n)
CHARACTER(n)
```
儲存長度不足會自動補空白（padding）

### 可變長度字串

```sql
VARCHAR(n)
CHAR VARYING(n)
```
- 最長 n 個字元
- 實務上幾乎都用 VARCHAR
- 在支援 Unicode 的 DBMS 中，一個中文字通常算一個字元，否則兩個。

## 4.3 Bit 串與 Boolean 

### Bit-string data types
  - BIT(n)：固定長度 bit
  - BIT VARYING(n)：可變長度 bit

### Boolean data types
 - TRUE / FALSE / NULL

### DATE data type
 - Ten positions
 - YEAR,MONTH,DAY in form YYYY-MM-DD

### TIME data type
 - 標準的時間欄是
  - hh:mm::ss
 ```sql
 TIME
 TIME(p)              -- 精度 p，可記小數秒 TIME(2)表示藥劑載到百分之一秒(10^-2)
 TIME WITH TIME ZONE  -- 包含時區
```
- 範例：
  - TIME '13:45:59'
  - TIME '13:45:59.25' -- 小數秒

### Timestamp(DATETIME) data type
```sql
TIMESTAMP
TIMESTAMP(p)
TIMESTAMP WITH TIME ZONE
```
- 代表日期 + 時間
 - 2024-01-06 13:22:11

### INTERVAL（時間區間）
- INTERVAL 表示一段相對的時間，可加入 DATE / TIMESTAMP 使用：
```
DATE '2024-01-01' + INTERVAL '15' DAY
TIMESTAMP '2024-01-01 00:00:00' + INTERVAL '2' HOUR
```

## 4.3 BLOB / CLOB（大型物件）
