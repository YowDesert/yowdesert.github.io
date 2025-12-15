---
title: (資料庫系統) Advanced SQL
categories:
  - 資料庫系統
  - SQL
tags:
  - 資料庫系統
  - SQL
abbrlink: '31704117'
mathjax: true
date: 2025-12-14 15:00:00
---

# CH07 Advanced SQL（More SQL）

本章目標：  
從基本 SELECT–FROM–WHERE，進階到能撰寫 **實務等級的複雜 SQL 查詢**。

---

# More Complex SQL Retrieval Queries
SQL 提供額外功能以支援更複雜的查詢：
- Nested Queries（巢狀查詢）
- Joined Tables 與 Outer Joins
- Aggregate Functions 與 Grouping

---

# NULL 與 Three-Valued Logic（三值邏輯）

## NULL 的意義
- Unknown（未知）
- Unavailable / Withheld（不可用或被隱藏）
- Not applicable（不適用）

## 重要觀念
- NULL 不是一個值
- NULL ≠ NULL
- SQL 使用 TRUE / FALSE / UNKNOWN

## Three-Valued Logic（三值邏輯）
- TRUE
- FALSE
- UNKNOWN（通常來自 NULL）

### AND（且）

| AND       | TRUE   | FALSE  | UNKNOWN |
|-----------|--------|--------|---------|
| **TRUE**    | TRUE   | FALSE  | UNKNOWN |
| **FALSE**   | FALSE  | FALSE  | FALSE   |
| **UNKNOWN** | UNKNOWN| FALSE  | UNKNOWN |


### OR（或）

| OR        | TRUE   | FALSE  | UNKNOWN |
|-----------|--------|--------|---------|
| **TRUE**    | TRUE   | TRUE   | TRUE    |
| **FALSE**   | TRUE   | FALSE  | UNKNOWN |
| **UNKNOWN** | TRUE   | UNKNOWN| UNKNOWN |


### NOT（非）

| Operand  | NOT Result |
|----------|------------|
| TRUE     | FALSE      |
| FALSE    | TRUE       |
| UNKNOWN  | UNKNOWN    |

---

# 檢查 NULL：IS NULL / IS NOT NULL

```sql
SELECT Fname, Lname
FROM EMPLOYEE
WHERE Super_ssn IS NULL;
```

---

# Nested Queries（巢狀查詢）

在 WHERE 中嵌入另一個完整的 SELECT 查詢。
> 在一個 SQL 查詢中，**再放一個完整的 SELECT-FROM-WHERE 查詢**
> 關鍵用途：用「一個查詢的結果」當作「另一個查詢的條件」


## 什麼是 Nested Query（巢狀查詢）

### 定義
- **Nested Query（子查詢）**：  
  一個完整的 `SELECT-FROM-WHERE` 查詢，被放在另一個查詢裡
- 外層稱為 **Outer Query**
- 內層稱為 **Nested Subquery**

```sql
SELECT ...
FROM ...
WHERE column IN (SELECT column FROM ... WHERE ...);
```

## IN 運算子（最基本、最常考）

- 語意
    - v IN V
    比較一個值 v 是否屬於一個集合（或 multiset）V
    若 v 是集合中任一元素 → 回傳 TRUE
- 特性
    - 子查詢回傳「一個欄位（單一 column）」
    可搭配 OR、AND 使用

### 範例
```sql
SELECT DISTINCT Pnumber
FROM PROJECT
WHERE Pnumber IN (
    SELECT Pnumber
    FROM PROJECT, DEPARTMENT, EMPLOYEE
    WHERE Dnum = Dnumber
      AND Mgr_ssn = Ssn
      AND Lname = 'Smith'
)
OR Pnumber IN (
    SELECT Pno
    FROM WORKS_ON, EMPLOYEE
    WHERE Essn = Ssn
      AND Lname = 'Smith'
);
```
>找出 Smith 當經理的專案
或 Smith 有實際參與的專案

## 使用 Tuple（多欄位）做比較
- 語法
    - 可以把多個欄位包成一個 tuple
    放在比較條件中
```sql
(column1, column2) IN (subquery)
```

### Example
```sql
SELECT DISTINCT Essn
FROM WORKS_ON
WHERE (Pno, Hours) IN (
    SELECT Pno, Hours
    FROM WORKS_ON
    WHERE Essn = '123456789'
);
```

>找出「工作內容（專案 + 時數）
和員工 123456789 完全一樣」的其他員工

## IN 右邊可以直接寫常數集合

- 不一定要子查詢
```sql
SELECT DISTINCT mId
FROM Browse
WHERE pNo IN ('b30999', 'b10234', 'd11222');
```
>IN (集合) ≈ = 某個集合中的一個

## 子查詢可以參考外層查詢
- 概念
    - 子查詢可以使用「外層查詢的變數」
    這會導致：子查詢為外層的每一筆資料重新執行

### EXAMPLE
```sql
SELECT DISTINCT M.mId, B.pNo
FROM Browse AS B, Member AS M
WHERE name = '王小明'
  AND B.mId = M.mId
  AND pNo IN (
      SELECT pNo
      FROM Record AS R, Transaction AS T
      WHERE transMid = M.mId
        AND T.tNo = R.tNo
  );
```

> 先找出「王小明」的所有 mId
對 每一筆瀏覽紀錄
檢查該商品是否出現在「他實際購買的商品清單」中

---

# 集合比較運算子
- IN
- ANY / SOME
- ALL

##  ANY（或 SOME）運算子

### 語法
```sql
value = ANY (subquery)
```
或
```sql
value = SOME (subquery)
```

### 說明
- 若 `value` 等於子查詢結果中的 **任一個值**，則回傳 TRUE
- **等價於 `IN`**

```sql
value IN (subquery)
≡
value = ANY (subquery)
```

---

##  ALL 運算子（非常重要）

### 語法
```sql
value > ALL (subquery)
```

### 說明
- `value` 必須 **大於子查詢回傳的所有值**
- 常用於「最大 / 最小」或「比全部都大（小）」的語意

### 範例 1：薪資比較
```sql
SELECT Lname, Fname
FROM EMPLOYEE
WHERE Salary > ALL (
    SELECT Salary
    FROM EMPLOYEE
    WHERE Dno = 5
);
```

👉 意義：找出薪水 **高於部門 5 所有員工** 的人



### 範例 2：商品價格
```sql
SELECT pNo, pName
FROM Product
WHERE unitPrice > ALL (
    SELECT unitPrice
    FROM Product
    WHERE category = 'Book'
);
```

👉 意義：找出價格 **比所有書籍都貴** 的商品

---

## 比較運算子可搭配 ANY / ALL

可搭配的比較運算子包含：
- `>`、`>=`
- `<`、`<=`
- `=`、`<>`

```sql
value > ANY (subquery)
value <= ALL (subquery)
```

---

##  使用 Alias 避免模糊錯誤
### 為什麼需要 Alias？
- 不同資料表可能有 **相同欄位名稱**
- 若不使用 alias，SQL 會產生 **語意不明或錯誤**

### 範例（Query 16）
```sql
SELECT E.Fname, E.Lname
FROM EMPLOYEE AS E
WHERE E.Ssn IN (
    SELECT Essn
    FROM DEPENDENT AS D
    WHERE E.Fname = D.Dependent_name
      AND E.Sex = D.Sex
);
```

👉 意義：
- 找出「有家屬」
- 且家屬 **名字與性別都與員工相同** 的員工

---

# Correlated Nested Queries（相關巢狀查詢）

## 定義
- 子查詢 **會參考外層查詢的欄位**
- 子查詢會對外層的 **每一筆資料重新執行一次**

## 特性
- 表達能力強
- 但效能可能較差（若 DBMS 未最佳化）



## 改寫為單一查詢（JOIN）

```sql
SELECT E.Fname, E.Lname
FROM EMPLOYEE AS E, DEPENDENT AS D
WHERE E.Ssn = D.Essn
  AND E.Sex = D.Sex
  AND E.Fname = D.Dependent_name;
```

👉 與前一頁巢狀查詢 **語意完全相同**
👉 DBMS 內部通常會做這類最佳化

---

# EXISTS 函數

## 語法
```sql
EXISTS (subquery)
```

## 說明
- 檢查子查詢 **是否回傳至少一筆資料**
- 回傳值為 **TRUE / FALSE**
- 常與 **correlated nested query** 搭配使用



### 範例：找出有購買指定商品的會員

```sql
SELECT mId, name
FROM Member
WHERE EXISTS (
    SELECT *
    FROM Product, Record, Transaction
    WHERE pName = '系統分析理論與實務'
      AND Product.pNo = Record.pNo
      AND Record.tNo = Transaction.tNo
      AND mId = transMid
);
```

👉 子查詢中的 `mId` 來自外層 Member（相關子查詢）

---

## EXISTS + EXISTS（複合條件）

### 範例：找出「至少有一位家屬的主管」

```sql
SELECT Fname, Lname
FROM Employee
WHERE EXISTS (
    SELECT *
    FROM DEPENDENT
    WHERE Ssn = Essn
)
AND EXISTS (
    SELECT *
    FROM Department
    WHERE Ssn = Mgr_ssn
);
```

👉 同時滿足：
- 是主管
- 且至少有一位家屬

---

# NOT EXISTS

## 語法
```sql
NOT EXISTS (subquery)
```

## 說明
- 子查詢 **完全沒有回傳資料** 時為 TRUE

---

### 範例：找出不是由購物車產生的交易

```sql
SELECT tNo, transMid
FROM Transaction AS T
WHERE NOT EXISTS (
    SELECT *
    FROM Cart
    WHERE tNo = T.tNo
);
```

👉 意義：找出 **非由購物車而來的交易紀錄**

---

## 重點整理

- `IN` ≡ `= ANY`
- `> ALL`：比所有值都大
- ANY / ALL 可搭配各種比較運算子
- EXISTS 回傳 TRUE / FALSE（不看內容，只看有沒有資料）
- Correlated nested query：對外層每筆資料執行一次
- EXISTS / NOT EXISTS 非常適合表示「存在 / 不存在」

---


# JOINed Tables

## 核心觀念：JOIN 是在 FROM 先做、WHERE 再過濾

SQL 的邏輯執行順序（觀念上）：
1. `FROM`（含 JOIN / ON）先形成一張「中間結果表」
2. `WHERE` 再把不符合條件的列刪掉
3. `SELECT` 決定輸出欄位
4. `DISTINCT / GROUP BY / HAVING / ORDER BY` …

✅ 這會直接影響 **OUTER JOIN**：
- `ON`：決定「怎麼配對」以及「配不到要不要補 NULL」
- `WHERE`：會把結果再刪掉，可能把補 NULL 的列刪光 → **OUTER JOIN 失效**

---

## Joined table（在 FROM 直接指定 JOIN 出來的表）

### 定義
**Joined table**：讓你在 `FROM` 子句直接寫出 join 的結果表。

### 範例
```sql
SELECT Fname, Lname, Address
FROM (EMPLOYEE JOIN DEPARTMENT ON Dno = Dnumber)
WHERE Dname = 'Research';
```
- 先把 EMPLOYEE 與 DEPARTMENT 依 `Dno = Dnumber` 配對
- 再挑出部門名稱為 Research 的員工

> 小結：`JOIN` 也常被稱為 `INNER JOIN`（預設就是內連接）。

---

## JOIN 的種類總覽

### NATURAL JOIN
- **不寫 ON** 的 join
- 系統會自動用「兩張表中同名欄位」做等值連接（EQUIJOIN）
- 風險：同名欄位一多、或命名不一致，很容易 join 錯

### OUTER JOIN（LEFT / RIGHT / FULL）
- 用來保留「配不到」的資料列
- 配不到的一側欄位會 **補 NULL**

---

## NATURAL JOIN（自然連接）

### 概念
若兩張表 R 與 S 有同名欄位（例如都叫 `Dno`），則：
```sql
R NATURAL JOIN S
```
等價於：
- 對所有同名欄位做 `R.col = S.col` 的等值連接
- 並且輸出時同名欄位只留一份

### 為了 NATURAL JOIN，常需要先「改欄位名」

EXAMPLE：DEPARTMENT 原本是 `Dnumber`，EMPLOYEE 是 `Dno`
→ 名字不同就 NATURAL JOIN 不起來，所以先把 DEPARTMENT 的 Dnumber 改名為 Dno：

```sql
SELECT Fname, Lname, Address
FROM (
  EMPLOYEE NATURAL JOIN
  (DEPARTMENT AS DEPT (Dname, Dno, Mssn, Msdate))
)
WHERE Dname = 'Research';
```

📌 解釋：
- `DEPARTMENT AS DEPT (Dname, Dno, Mssn, Msdate)` 這段是在 **重新命名欄位**
- 讓 `EMPLOYEE.Dno` 能和 `DEPT.Dno` 同名
- 因此 NATURAL JOIN 的隱含條件就是：`EMPLOYEE.Dno = DEPT.Dno`

>✅ 結論：NATURAL JOIN 的本質仍是 **等值連接**，只是條件隱藏了。

---

## INNER JOIN vs OUTER JOIN

### INNER JOIN（內連接）
- 只有「兩邊都配得到」的列才會出現在結果中

###  LEFT OUTER JOIN（左外連接）
- **左表每一列一定要出現**
- 如果右表配不到 → 右表欄位補 NULL

### RIGHT OUTER JOIN（右外連接）
- **右表每一列一定要出現**
- 如果左表配不到 → 左表欄位補 NULL

> FULL OUTER JOIN：左右兩邊都保留（有些 DB 不支援）

---

## LEFT OUTER JOIN 範例（員工與主管）

### 需求
列出每位員工與他的主管（如果沒有主管，也要列出員工）。

```sql
SELECT E.Lname AS Employee_Name,
       S.Lname AS Supervisor_Name
FROM (EMPLOYEE AS E LEFT OUTER JOIN EMPLOYEE AS S
      ON E.Super_ssn = S.Ssn);
```

📌 解讀：
- 這是「自我連接」（EMPLOYEE 跟自己 JOIN）
- 主管資訊在同一張表，用 `Super_ssn` 指到主管的 `Ssn`
- 若某員工 `Super_ssn` 是 NULL 或找不到對應主管
  → 仍保留員工列，主管欄位補 NULL

### 舊式語法（了解即可）
有些系統曾用 `+=`、`=+`、`+=+` 表示 outer join。
現在考試多以標準 SQL 的 `LEFT/RIGHT/FULL OUTER JOIN` 為主。

---

## Multiway JOIN（多表 JOIN）

### 概念
多張表 JOIN 時，可以把 JOIN **巢狀包起來**（nest join specifications）。

### 範例（Q2A）
```sql
SELECT Pnumber, Dnum, Lname, Address, Bdate
FROM (
  (PROJECT JOIN DEPARTMENT ON Dnum = Dnumber)
  JOIN EMPLOYEE ON Mgr_ssn = Ssn
)
WHERE Plocation = 'Stafford';
```

📌 解讀流程：
1. `PROJECT` 先 JOIN `DEPARTMENT`（專案屬於哪個部門）
2. 再把上一步結果 JOIN `EMPLOYEE`（該部門的經理是誰）
3. 最後 WHERE 篩選專案地點 `Stafford`

---

## OUTER JOIN 的「超級陷阱」：條件放 WHERE 會讓 OUTER JOIN 失效

### 題目需求
「列出每一位會員的會員編號、姓名，以及 **2005 年**所瀏覽的商品編號（如果有的話）」

也就是：
- **每個會員都要列出**（所以要 LEFT OUTER JOIN）
- 但瀏覽紀錄只要 2005 年的

---

###  錯誤寫法（WRONG）
```sql
SELECT M.mId, name, pNo
FROM Member AS M LEFT OUTER JOIN Browse AS B
     ON M.mId = B.mId
WHERE to_char(browseTime, 'yyyy') = '2005';
```

❌ 為什麼錯？
- LEFT OUTER JOIN 先把「沒有瀏覽紀錄」的會員補了一列（B.* = NULL）
- 但 `WHERE to_char(browseTime, 'yyyy') = '2005'`
  對於 `browseTime = NULL` 的列，條件結果會是 UNKNOWN → 被 WHERE 過濾掉
- 結果：**沒有 2005 瀏覽紀錄的會員整個不見**
➡️ LEFT OUTER JOIN 的「保留所有會員」效果被破壞

###  正確寫法（把條件放 ON）
```sql
SELECT M.mId, name, pNo
FROM Member AS M LEFT OUTER JOIN Browse AS B
ON (
  M.mId = B.mId
  AND to_char(browseTime, 'yyyy') = '2005'
);
```

✅ 為什麼這樣對？
- `ON` 的條件是「配對規則」：只把 2005 的瀏覽紀錄拿來配
- 沒有 2005 瀏覽紀錄的會員：仍會保留（右表補 NULL）
- 完全符合題意：「如果有的話」

---

##  什麼時候放 ON？什麼時候放 WHERE？

✅ **放 ON（配對條件）**：
- 你要保留 outer join 的「配不到也要留」效果
- 右表的篩選條件（例如年份、狀態）又不能把左表刪掉

✅ **放 WHERE（結果過濾）**：
- 你就是要把不符合條件的列刪掉（不管 outer join）
- 或你在 INNER JOIN 情境下過濾

---

## JOIN整理

- `JOIN` 預設是 `INNER JOIN`
- `NATURAL JOIN`：用同名欄位自動等值連接（不寫 ON）
- `LEFT OUTER JOIN`：左表全保留；右表配不到補 NULL
- `RIGHT OUTER JOIN`：右表全保留；左表配不到補 NULL
- 多表 JOIN：可以巢狀寫 `(... JOIN ...) JOIN ...`
- OUTER JOIN 篩選條件：
  - 放 `WHERE` 可能讓 OUTER JOIN 變成 INNER JOIN
  - 放 `ON` 才能保留「配不到也要留」

---

# Aggregate Functions & GROUP BY / HAVING

## Aggregate Functions（彙總函數）
用來把多筆 tuple 的資訊彙總成「單筆結果」或「每組一筆結果」。

常見內建函數：
- `COUNT`, `SUM`, `MAX`, `MIN`, `AVG`

###  NULL 與 aggregate
- 對欄位做 `SUM/MAX/MIN/AVG` 時：**NULL 會被忽略（discarded）**
- `COUNT(*)`：算列數（包含該列某些欄位為 NULL 也會算）
- `COUNT(col)`：只算 `col` **不是 NULL** 的列

---

## Aggregation 結果改欄位名、取別名（AS）
###
```sql
SELECT SUM(Salary), MAX(Salary), MIN(Salary), AVG(Salary)
FROM EMPLOYEE;
```

```sql
SELECT
  SUM(Salary) AS Total_Sal,
  MAX(Salary) AS Highest_Sal,
  MIN(Salary) AS Lowest_Sal,
  AVG(Salary) AS Average_Sal
FROM EMPLOYEE;
```

---

## Aggregation + WHERE（先篩選，再彙總）
### 範例
```sql
SELECT SUM(Salary), MAX(Salary), MIN(Salary), AVG(Salary)
FROM (EMPLOYEE JOIN DEPARTMENT ON Dno = Dnumber)
WHERE Dname = 'Research';
```

### COUNT 範例
```sql
-- 全公司員工數
SELECT COUNT(*)
FROM EMPLOYEE;

-- Research 部門員工數
SELECT COUNT(*)
FROM EMPLOYEE, DEPARTMENT
WHERE Dno = Dnumber AND Dname = 'Research';
```

---

##  SOME / ALL 也可當「布林集合函數」
把一串布林值（TRUE/FALSE/UNKNOWN）當集合看：
- `SOME`：只要**至少一個 TRUE** → TRUE（像 OR）
- `ALL`：必須**全部 TRUE** → TRUE（像 AND）

> 這裡是「函數」語意，不是前面比較運算子的 `= ANY`, `> ALL` 那種用法。

---

## GROUP BY（分組）
把資料依某些欄位分成多組，然後 **每組各自做 aggregate**。

### 規則（必背）
- `SELECT` 中若同時出現「一般欄位」與「aggregate」：
  - 一般欄位必須出現在 `GROUP BY`

### 範例（Q24：每個部門的員工數與平均薪資）
```sql
SELECT Dno, COUNT(*), AVG(Salary)
FROM EMPLOYEE
GROUP BY Dno;
```

### NULL 分組
- 分組欄位若可能為 NULL → **NULL 會形成獨立的一組**


##  GROUP BY 套在 JOIN 結果上
### 範例（Q25：每個專案有多少員工參與）
```sql
SELECT Pnumber, Pname, COUNT(*)
FROM PROJECT, WORKS_ON
WHERE Pnumber = Pno
GROUP BY Pnumber, Pname;
```

---

## HAVING（篩選「整組」）
- `WHERE`：逐列（tuple by tuple）篩選
- `HAVING`：逐組（group）篩選（常搭配 aggregate 條件）

### 範例（Q26：參與人數 > 2 的專案）
```sql
SELECT Pnumber, Pname, COUNT(*)
FROM PROJECT, WORKS_ON
WHERE Pnumber = Pno
GROUP BY Pnumber, Pname
HAVING COUNT(*) > 2;
```

---

## WHERE + HAVING 的常見錯誤與正確寫法
需求：
> 各部門中「薪資 > 40000 的員工數」，但只統計「部門總員工數 > 5」的部門。

### 錯誤（會把部門總人數也限制成薪資 > 40000 的人數）
```sql
SELECT Dno, COUNT(*)
FROM EMPLOYEE
WHERE Salary > 40000
GROUP BY Dno
HAVING COUNT(*) > 5;
```

### 正確（先選出「總員工數 > 5」的部門，再統計該部門的高薪員工數）
```sql
SELECT Dno, COUNT(*)
FROM EMPLOYEE
WHERE Salary > 40000
  AND Dno IN (
    SELECT Dno
    FROM EMPLOYEE
    GROUP BY Dno
    HAVING COUNT(*) > 5
  )
GROUP BY Dno;
```

---

## 快速對照
- `WHERE`：篩「列」
- `GROUP BY`：分「組」
- `HAVING`：篩「組」
- `COUNT(*)`：算列數
- `COUNT(col)`：算 col 非 NULL 的列數   


---


# WITH Clause（CTE
- 定義只在單一查詢中使用的暫時表（temporary view）
- 用來提升可讀性，把複雜查詢拆成步驟
- 功能等同子查詢，但更好讀

## WITH 範例（改寫子查詢
```sql
WITH BIGDEPTS(Dno) AS (
  SELECT Dno
  FROM EMPLOYEE
  GROUP BY Dno
  HAVING COUNT(*) > 5
)
SELECT Dno, COUNT(*)
FROM EMPLOYEE
WHERE Salary > 40000 AND Dno IN BIGDEPTS
GROUP BY Dno;
```
- 與 `IN (SELECT ... HAVING ...)` 結果相同

## CASE（條件表達式）
- SQL 的 if–else
- 可用於 SELECT / UPDATE / INSERT

```sql
CASE
  WHEN condition THEN value
  ELSE value
END
```

## Recursive Queries 概念 
- 用於階層式資料（如員工與主管）
- 同一資料表重複引用自己

## Recursive CTE 範例
```sql
WITH RECURSIVE SUP_EMP(SupSsn, EmpSsn) AS (
  SELECT SupervisorSsn, Ssn FROM EMPLOYEE
  UNION
  SELECT E.Ssn, S.SupSsn
  FROM EMPLOYEE E, SUP_EMP S
  WHERE E.SupervisorSsn = S.EmpSsn
)
SELECT * FROM SUP_EMP;
```
- Base case + Recursive step
- 重複直到無新資料

## SQL 查詢完整結構
```
SELECT → FROM → WHERE → GROUP BY → HAVING → ORDER BY
```
- WHERE：逐列
- HAVING：逐組

## 7. ASSERTION（語意約束
- 定義超出 PK / FK / CHECK 的限制
- 多數 DBMS 不支援，考試知道即可

## 8. TRIGGER
- Event + Condition + Action
- 事件發生時自動執行
```sql
CREATE TRIGGER ...
AFTER INSERT ON ...
FOR EACH ROW
WHEN (condition)
...
```

## 13. Recursive Queries

```sql
WITH RECURSIVE SUP_EMP AS (
  SELECT SupervisorSsn, Ssn
  FROM EMPLOYEE
  UNION
  SELECT E.SupervisorSsn, S.Ssn
  FROM EMPLOYEE E, SUP_EMP S
  WHERE E.SupervisorSsn = S.Ssn
)
SELECT * FROM SUP_EMP;
```

---

#  View
- View 是 **虛擬資料表 (Virtual Table)** 
- 由其他資料表（defining tables）查詢結果定義
- View 本身 **不實際存資料**
- 常見用途：
  - 將複雜查詢包成 View 重複使用
  - 隱藏欄位、做權限控管
> 作者敢想 : 我感覺他很像Function 可以重複拿出來用

## CREATE VIEW 語法
```sql
CREATE VIEW view_name (col1, col2, ...)
AS SELECT ...
```
- 定義 View 名稱 + 查詢內容
- 範例：計算每筆交易總金額
```sql
CREATE VIEW Transaction_total(tNo, totalAmount)
AS
SELECT tNo, SUM(salePrice)
FROM Record
GROUP BY tNo;
```

## 欄位命名方式差異
- V1：View 欄位沿用原資料表欄位名稱
- V2：在 CREATE VIEW 時 **重新指定欄位名稱**
- 常用在：聚合結果、報表型 View

## 使用與刪除 View
- View 可直接用在 `FROM`，像一般資料表
```sql
SELECT Fname, Lname
FROM WORKS_ON1
WHERE Pname = 'Seena';
```
- View 永遠是 **即時資料（up-to-date）**
- 刪除 View：
```sql
DROP VIEW WORKS_ON1;
```


## 透過 View 修改資料（可更新 View）
- 若 View 對應 **單一資料表**
- 且修改有明確對應關係
→ 更新 View = 更新原資料表

```sql
UPDATE Cheap_product
SET unitPrice = unitPrice * 0.9;
```
➡ 實際修改的是 `Product`

## 不能更新的 View（三大類）
以下 View **不可更新**：
1. View 定義中包含 **聚合函數（SUM / AVG / COUNT）**
2. View 不包含 **key（無法唯一對應一筆資料）**
3. View 由 **多個資料表 JOIN** 而成


## 更新的問題
```sql
CREATE VIEW Trans_product AS
SELECT tNo, pName
FROM Record NATURAL JOIN Product;
```
- 嘗試 UPDATE View 時
- DBMS 無法判斷要更新哪個 base table
- 因此通常禁止

## 等價的正確更新方式
- View 更新不被允許
- 但可直接更新 base tables

```sql
UPDATE Product
SET pName = 'OLAP進階'
WHERE pNo = 'b30999';

UPDATE Record
SET pNo = 'b20666'
WHERE tNo = '91100' AND pNo = 'b30999';
```

## View 作為權限機制
- View 可限制使用者能看到的資料
- 常搭配 GRANT / REVOKE

```sql
CREATE VIEW DEPT5EMP AS
SELECT *
FROM EMPLOYEE
WHERE Dno = 5;
```
➡ 使用者只能看到部門 5 的員工



## Schema Change（結構演進）
- Schema evolution：在系統運作中修改結構
- 不需重編整個資料庫
- 常見如：
  - ALTER TABLE
  - ADD / DROP 欄位


---

# DROP Command 在做什麼
**目的：刪除 schema 元素**
- 可刪除：schema、table、view、domain、constraint
- 兩種行為：
  - `CASCADE`：連同相依物件一起刪
  - `RESTRICT`：若有相依物件就拒絕

```sql
DROP SCHEMA COMPANY CASCADE;
```
➡ schema 及其 tables / views / constraints 全部消失

## ALTER TABLE Command 在做什麼
**目的：在資料庫運作中修改表格結構**
- 可做的事：
  - 新增 / 刪除欄位
  - 修改欄位定義
  - 新增 / 刪除 constraint

```sql
ALTER TABLE COMPANY.EMPLOYEE
ADD COLUMN Job VARCHAR(12);
```

---

## 新增與刪除 Constraints
**目的：動態管理約束**
- 可針對「具名 constraint」操作
- 常見於 foreign key / check

```sql
ALTER TABLE COMPANY.EMPLOYEE
DROP CONSTRAINT EMPSUPERFK CASCADE;
```



## 刪除欄位與 Default 值
### 刪除欄位
- 必須指定：`CASCADE` 或 `RESTRICT`
- 若有 view 參考該欄位：
  - CASCADE：連 view 一起刪
  - RESTRICT：禁止

```sql
ALTER TABLE COMPANY.EMPLOYEE
DROP COLUMN Address CASCADE;
```

### 修改 Default 值
```sql
ALTER TABLE COMPANY.DEPARTMENT
ALTER COLUMN Mgr_ssn DROP DEFAULT;

ALTER TABLE COMPANY.DEPARTMENT
ALTER COLUMN Mgr_ssn SET DEFAULT '333445555';
```


## SQL 語法總覽
**目的：當作語法查表**
- CREATE / DROP / ALTER TABLE
- SELECT（含 JOIN / GROUP BY / HAVING / ORDER BY）
- INSERT / DELETE / UPDATE
- CREATE / DROP VIEW
- CREATE / DROP INDEX（非標準 SQL）



## DBMS 差異補充（很愛考觀念）
- 有些 DBMS：
  - 用 `MINUS` 取代 `EXCEPT`
  - 不支援 `CREATE ASSERTION`
  - 用 `CREATE TRIGGER` 模擬 assertion
  - 不支援 `NATURAL JOIN`
  - 對 JOIN View 的更新限制較寬鬆

---
> 先這樣