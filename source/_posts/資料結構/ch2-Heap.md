---
title: 資料結構-Heap、Disjoint Sets
categories:
  - 資料結構
  - 第二章
tags:
  - 資料結構
abbrlink: 1f21266d
mathjax: true
date: '2026-8-09 14:16:00'
---

# Heap
1. MAX heap
    - complete binary tree
    - 父點要 **大於** 左右子點
    - Root為**最大值**
2. min heap

**適合用Array保存** (因為compelete)


| Heap 操作                      | 中文意思      |        時間複雜度 | 為什麼                        |
| ---------------------------- | --------- | -----------: | -------------------------- |
| **Insert X**                 | 插入元素      |  (O(log n)) | 插入最後，再往上調整，最多走樹高           |
| **Delete-Max**               | 刪除最大值     |  (O(log n)) | 刪 Root 後，要向下 Heapify       |
| **Heapify / Adjust**         | 調整 Heap   |  (O(log n)) | 最差從 Root 一路調到 Leaf         |
| **Find-Max**                 | 找最大值      |   **(O(1))** | Max-Heap 最大值就在 Root        |
| **Build a Heap (Bottom-Up)** | 建立 Heap   | **(O(n))** ⭐ | Bottom-Up Heapify          |
| **Decrease Key（Min-Heap）**   | 降低某節點的值   |  (O(log n)) | 可能一路往上調整                   |
| **Merge two Heaps**          | 合併兩個 Heap |   **(O(n))** | 合併後重新 Bottom-Up Build Heap |
| **Search X**                 | 搜尋某個值     |   **(O(n))** | Heap 沒有 BST 的左右大小關係，最差全部找  |

# Heap Adjust

## Top Down(O(nlogn))
- 連續insert n 次
- 每動作均維持Heap性質

- 方法 : 
    1. 找出最大值{左子、右子}
    2. if x >= MAX : 符合
    3. 否則 交換

- 分析 : 
    insert X : O(log i) : i 為當時的Node數
    Total = $\sum_{i=1}^n (\log i) = log(n!) $ = O (nlogn)

## Buttom-Up (重要) (O(n)) 
- input Array視為Compelete Binary Tree (直接不管 先全部插進去)
- 從**Last Parent**開始，調整其子樹成為Heap，依序往前到Root (將子點比較大的往上拉)


- 分析
    - Bottom up : n 個 data -> 高度 k = $log_2 (n+1)$
    - 每個點走的高度不同 : 只有Wrost 才是全都是Wrost
    - 葉子有 : n/2 個 ; 葉子上一層有 : n/4 ...
    - 所以 n/4 * 1 + n/8 * 2 +... + 1 * log(n) => n*(1/4+2/8+3/16+..) => O(n)
    > (1/4+2/8+3/16+..) = O (1)

# Merge two Heap
作法 : 令兩個heap H1 H2皆為[1...n] Array
1. 將H1、H2寫入到一Array[1...2n]中=>Time : O(n)
2. 針對[1...2n] Array 以 Botton up 調成 Heap : O(n) 

# Del-MAX
1. 移走Root
2. 將最後一點Node x 刪除且暫時置放於Root
3. 往下調整

# Tree 化為 Binary Tree
方法 : Leftmost-child-Next-Right-Sibling
1. 建立Sibling之間的平行Links
2. 刪除父子之間的Links 但保留最左子點Links
3. 順時針轉45度即次右兄弟拉下成右子點最左子點保留成左子點

> 反向就反過來

# Forest 化成 Binary Tree
- 將各樹平行Links，其餘雨上述一樣
- 反向就反過來

# Catalan Number

$$
C_n=\frac{1}{n+1}\binom{2n}{n}
$$

常見應用：

1. 括號匹配
2. Stack 出棧序列
3. 不同 Binary Tree / BST 的數量

## 簡單證明

從 $(0,0)$ 走到 $(n,n)$，共走 $2n$ 步：

* 往右 $n$ 次
* 往上 $n$ 次

所以全部走法：

$$
\binom{2n}{n}
$$

合法路徑要求任何時刻：

$$
\text{上} \le \text{右}
$$

也就是不能越過對角線。

利用反射法，不合法路徑數為：

$$
\binom{2n}{n-1}
$$

因此：

$$
C_n

=
\binom{2n}{n}

\binom{2n}{n-1}
=
\frac{1}{n+1}\binom{2n}{n}
$$

## 為什麼可以算不同 Binary Tree？

一棵有 $n$ 個節點的 Binary Tree，root 先占一個節點。

假設左子樹有 $i$ 個節點，右子樹就有：

$$
n-1-i
$$

所以這種分法有：

$$
C_iC_{n-1-i}
$$

種。

將所有左右子樹的可能分法加起來：

$$
C_n
=
\sum_{i=0}^{n-1}
C_iC_{n-1-i}
$$

這正是 Catalan Number 的遞迴式。

所以：

> Catalan Number 可以用來計算 $n$ 個節點能形成多少種不同形狀的 Binary Tree。

前幾項：

$$
1,\ 1,\ 2,\ 5,\ 14,\ 42,\dots
$$

# Disjoint Sets
互斥的Sets組成

# Disjoint Sets 表示法
1. Link List
    - Sets中，任選一元素作為Root,其餘的Parent link指向Root
    - Root's parent link指向自己或null
    - Node Data Structure : Dara,Parent link

    - Example：
        假設有一個 Set：

        $$
        {1,2,3,4}
        $$

        選 1 當 Root，則：

            1
            ↗ ↑ ↖
            2  3  4

        Parent link：

        1 → null
        2 → 1
        3 → 1
        4 → 1
2. Array
    Parent 
    - 非Root : 紀錄父的index no.
    - Root : 紀錄 **- (size)**,size = set中元素個數
    - Example：
        一樣假設：

        $$
        {1,2,3,4}
        $$

        且 1 為 Root。

        因為這個 Set 有 4 個元素，所以：

        $$
        Parent[1]=-4
        $$

        其他 Node 的 Parent 都是 1：

        $$
        Parent[2]=1,\quad
        Parent[3]=1,\quad
        Parent[4]=1
        $$

        index	1	2	3	4
        Parent	-4	1	1	1

# Arbitrary(任意的) Union and Simple Find(i) operations

## Arbitrary Union

任意把一棵樹的 Root 接到另一棵樹的 Root。
Time : O(1)

## Find

方法 :

```C++
j = i;
while(j->parent < 0) do
    j=j->parent
return j;

```
Time : O(h)
> Find(i) = 找 Root，不是找另一個節點。

## Arbitrary Union 缺點

初始：n 個 singleton sets
      {1},{2},...,{n}

做 n-1 次 Union
→ 合成 1 個 Set

但因為是任意連接
→ Tree 可能退化成一條線
→ height = n-1
→ Find 最差 O(n)

# Union by Weighting Rule
使用node較少的Tree 作為 Node 較多的Tree
如果相同,則任意Union
> 用了 Weighting Rule 後，樹高最多 O(logn)
所以：
Find(x)=O(logn)

小樹高度每增加 1，所在 Tree 的 node 數至少**翻倍**
>因為
高度1 : 1個點
高度2 : 兩個1個點
高度3 : 兩個2個點
高度4 : 兩個4個點 = > 兩個高度3的才可以

# Union by Weighting/Height/rank

| 方法              | 比較什麼    |
| --------------- | ------- |
| Union by Weight | node 數量 |
| Union by Height | Tree 高度 |
| Union by Rank   | rank    |
| 效果              | 都是避免樹太高 |

# Collapsing Rule / Path Compression
Weighted Union 是： Union 的時候避免 Tree 長太高。
Path Compression 是：Find 的時候順便把 Tree 壓扁。

**Find 路徑上的 node → parent 全改成 Root**
	​
方法
1. Iterative 

```cpp
Find_with_Collapsing(i)
{
    j = i;

    // ① 找 root
    while (j->parent != j)
        j = j->parent;

    k = i;

    // ② 把路徑全部接到 root
    while (k != j)
    {
        t = k->parent;
        k->parent = j;
        k = t;
    }

    return j;
}
```

2. Algorithm

```cpp
FIND-SET(x)
{
    if x != x.p
        x.p = FIND-SET(x.p);

    return x.p;
}
```

# Union by Rank + Path Compression
小 Rank 的 Root 接到大 Rank 的 Root。
如果：
1. rank(x) < rank(y) = > x.parent = y
2. rank(x)>rank(y) => y.parent = x
3. 如果 Rank 一樣 => 新 Root 的 rank +1

# Time Complexity

| 方法                             |  Union |     Find 最差 |
| ------------------------------ | -----: | ----------: |
| Arbitrary + Simple Find        | (O(1)) |      (O(n)) |
| Weighting + Simple Find        | (O(1)) | $(O(\log n))$ |
| Rank/Weight + Path Compression | (O(1)) |   $O( \alpha (n) )$ 幾乎等於 O(1) |
​
> 其中 α(n) 是 Inverse Ackermann Function

# Finding Equivalence Sets
```
如果：
1 = 5
4 = 2
7 = 11
9 = 10
8 = 5
7 = 9
4 = 6
3 = 12
12 = 1
```
> a = b
代表：
a,b 屬於同一個 equivalence set。

同一棵 Tree = 同一個 equivalence class。
就做：
Union(a,b)
最後會形成幾棵獨立的 Tree。

# Thread Binary Tree (引線2元樹)

```
Data Structure :
┌────────────┬────────┬──────┬────────┬─────────────┐
│ LeftThread │ Lchild │ Data │ Rchild │ RightThread │
└────────────┴────────┴──────┴────────┴─────────────┘
```
> LeftThread、RightThread : bool

1. 先把第一張圖的 Inorder 寫出來
2. Left Thread : x 在 Inorder 的前一個 Node (Inorder predecessor（前驅）)
3. Right Thread : x 在 Inorder 的下一個 Node (Inorder successor（後繼）)

## LeftThread、RightThread : True / False 怎麼看

1. LeftThread
    - True
        → Lchild 是 Thread
        → 指向 Inorder 前驅
    - False
        → Lchild 是真正的左小孩
2. RightThread
    - True
        → Rchild 是 Thread
        → 指向 Inorder 後繼
    - False
        → Rchild 是真正的右小孩

## Head Node
額外還有一個**Head Node** 不存Data
1. Case 1 : empty
2. Case 2 : Not Empty

用途 :
1. 讓 traversal 有固定起點
2. 最後一個 inorder 節點的 thread 可以指回 Head，這樣就知道「走完了」。

## 簡化中序
```
            Insuc(x)
               │
       x 有真正右子樹？
          /          \
        NO            YES
        ↓              ↓
 x.right thread    先到 x.right
 直接是答案        再一路往左
 ```
→ 不需 Stack / Recursion

## 把新節點 T 插入成 S 的右小孩

```cpp
// T 繼承 S 原本右邊的東西。
t->rightthread = s->rightthread;
t->rchild = s->rchild;

// T 的 predecessor=S

t->leftthread = true;
t->lchild = s;

// 現在 S 已經真的有右小孩 T 了。
s->rightthread = false;
s->rchild = t;

// 若原本有右子樹，還需修正 T successor 的 left-thread。
if (t->rightthread == false)
{
    temp = Insuc(t);
    temp->lchild = t;
}
```
