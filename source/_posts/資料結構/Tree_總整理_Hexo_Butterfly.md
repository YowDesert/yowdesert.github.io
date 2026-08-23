---
title: 資料結構 - 所有 Tree 整理
tags:
  - 資料結構
  - 第九章
abbrlink: f184578f
mathjax: true
date: '2026-8-22 11:30:00'
---

# Tree 總整理

## 一、各種 Tree 關係圖

```text
Tree
│
├── Binary Tree
│   │
│   ├── Full Binary Tree
│   ├── Complete Binary Tree
│   │   └── Heap
│   │       ├── Min Heap
│   │       └── Max Heap
│   │
│   ├── Binary Search Tree (BST)
│   │   │
│   │   ├── AVL Tree
│   │   ├── Red-Black Tree
│   │   └── Splay Tree
│   │
│   └── Huffman Tree
│
├── Multi-Way Search Tree
│   │
│   └── B-Tree
│       ├── 2-3 Tree
│       ├── 2-3-4 Tree
│       └── B+ Tree（B-Tree 的變形）
│
└── Heap 類
    ├── Binary Heap
    ├── Leftist Heap
    ├── Binomial Heap
    └── Fibonacci Heap
```

---

# 二、超級總表

| Tree | 是 BST？ | Balanced？ | Complete？ | 核心特色 |
|---|---|---|---|---|
| Binary Tree | ❌ | 不一定 | 不一定 | 每個 Node 最多 2 個 Child |
| Full Binary Tree | ❌ | 不一定 | 不一定 | 每個 Node 為 0 或 2 個 Child |
| Complete Binary Tree | ❌ | 不一定 | ✅ | 除最後一層外全滿，最後一層靠左 |
| BST | ✅ | 不一定 | 不一定 | Left < Root < Right |
| AVL Tree | ✅ | ✅ | ❌ | 左右高度差最多 1 |
| Red-Black Tree | ✅ | ✅ | ❌ | 利用紅黑顏色維持近似平衡 |
| Splay Tree | ✅ | Amortized | ❌ | 每次操作後將節點旋轉到 Root |
| Heap | ❌ | 高度平衡 | ✅ | Parent 與 Child 有大小關係 |
| Leftist Heap | ❌ | ❌ | ❌ | 適合 Merge |
| Binomial Heap | ❌ | ❌ | ❌ | 多棵 Binomial Tree |
| Fibonacci Heap | ❌ | ❌ | ❌ | Lazy Merge，Amortized 很快 |
| B-Tree | 類似 BST | ✅ | ❌ | 一個 Node 可以有很多 Key / Child |
| 2-3 Tree | 類似 BST | ✅ | ❌ | B-Tree Order 3 |
| 2-3-4 Tree | 類似 BST | ✅ | ❌ | B-Tree Order 4 |
| B+ Tree | 類似 BST | ✅ | ❌ | Data 主要存在 Leaf |
| Huffman Tree | ❌ | ❌ | ❌ | 最佳 Prefix Code |

> 注意：
>
> **Balanced 不代表 Complete。**
>
> AVL、RB Tree 雖然 Balanced，但通常不是 Complete Binary Tree。

---

# 三、Binary Tree

## Binary Tree

每個 Node：

$$
degree \le 2
$$

也就是最多有：

```text
Left Child
Right Child
```

Binary Tree 本身：

- 不要求排序
- 不要求平衡
- 不要求 Complete

所以：

> Binary Tree ≠ BST

---

## Full Binary Tree

又稱：

```text
Proper Binary Tree
Strict Binary Tree
```

每個 Node 的 Child 數只能是：

```text
0
或
2
```

如果：

- $E_0$ = Leaf 數
- $E_2$ = 有兩個 Child 的 Node 數

則：

$$
\boxed{E_0 = E_2 + 1}
$$

---

## Complete Binary Tree

特色：

1. 除了最後一層，前面的 Level 都要填滿
2. 最後一層由左到右填

Binary Heap 一定是：

$$
\boxed{Complete\ Binary\ Tree}
$$

所以很適合使用 Array 儲存。

---

# 四、Binary Search Tree

## BST

Binary Search Tree 的規則：

```text
Left Subtree < Root < Right Subtree
```

### Inorder Traversal

BST 做：

```text
Left → Root → Right
```

會得到 Sorted Order。

---

## BST 時間複雜度

令 Tree Height = $h$

| Operation | Complexity |
|---|---:|
| Search | $O(h)$ |
| Insert | $O(h)$ |
| Delete | $O(h)$ |

如果樹很平衡：

$$
h=O(\log n)
$$

但是 Worst Case 會退化成 Linked List：

$$
h=n
$$

| Operation | Average | Worst |
|---|---:|---:|
| Search | $O(\log n)$ | $O(n)$ |
| Insert | $O(\log n)$ | $O(n)$ |
| Delete | $O(\log n)$ | $O(n)$ |

---

# 五、Balanced BST

Balanced BST 的主要目的就是：

> 避免 BST 退化成 Linked List。

常見：

```text
BST
├── AVL Tree
├── Red-Black Tree
└── Splay Tree
```

---

# 六、AVL Tree

AVL Tree：

$$
\boxed{AVL = Balanced\ BST}
$$

## Balance Factor

$$
BF = Height(Left)-Height(Right)
$$

AVL 要求：

$$
\boxed{|BF|\le1}
$$

## AVL Rotation

| Case | 修正 |
|---|---|
| LL | Right Rotation |
| RR | Left Rotation |
| LR | Left Rotation + Right Rotation |
| RL | Right Rotation + Left Rotation |

## AVL Complexity

| Operation | Complexity |
|---|---:|
| Search | $O(\log n)$ |
| Insert | $O(\log n)$ |
| Delete | $O(\log n)$ |

Insert rotation：

$$
\boxed{O(1)}
$$

Delete rotation：

$$
\boxed{O(\log n)}
$$

---

# 七、Red-Black Tree

Red-Black Tree：

$$
\boxed{Balanced\ BST}
$$

## RB Tree 五大規則

1. 每個 Node 是 Red 或 Black
2. Root 是 Black
3. NIL Leaf 是 Black
4. Red Node 的 Child 一定是 Black
5. 任一 Node 到其 Descendant NIL 的 Black Node 數相同

高度：

$$
h\le2\log_2(n+1)
$$

所以：

$$
\boxed{h=O(\log n)}
$$

## RB Tree Complexity

| Operation | Complexity |
|---|---:|
| Search | $O(\log n)$ |
| Insert | $O(\log n)$ |
| Delete | $O(\log n)$ |

Insert 最多：

$$
\boxed{\le2\ rotations}
$$

Delete 最多：

$$
\boxed{\le3\ rotations}
$$

Recolor 可能：

$$
O(\log n)
$$

---

# 八、AVL vs Red-Black Tree

| | AVL | Red-Black |
|---|---|---|
| BST | ✅ | ✅ |
| Balanced | 很嚴格 | 較寬鬆 |
| 高度 | 較低 | 可能稍高 |
| Search | 較快 | 稍慢 |
| Insert/Delete | Rotation 較多 | 通常較少 |
| Search | $O(\log n)$ | $O(\log n)$ |
| Insert | $O(\log n)$ | $O(\log n)$ |
| Delete | $O(\log n)$ | $O(\log n)$ |

---

# 九、Splay Tree

Splay Tree：

$$
\boxed{BST}
$$

每次 Search / Insert / Delete 後，把操作節點旋轉到 Root。

## Splay Complexity

單次 Worst Case：

$$
O(n)
$$

Amortized：

$$
\boxed{O(\log n)}
$$

| Operation | Amortized |
|---|---:|
| Search | $O(\log n)$ |
| Insert | $O(\log n)$ |
| Delete | $O(\log n)$ |

---

# 十、BST / AVL / RB / Splay 比較

| | BST | AVL | RB Tree | Splay |
|---|---:|---:|---:|---:|
| 是 BST | ✅ | ✅ | ✅ | ✅ |
| 保證 Balanced | ❌ | ✅ | ✅ | ❌ |
| Search Worst | $O(n)$ | $O(\log n)$ | $O(\log n)$ | $O(n)$ |
| Insert Worst | $O(n)$ | $O(\log n)$ | $O(\log n)$ | $O(n)$ |
| Delete Worst | $O(n)$ | $O(\log n)$ | $O(\log n)$ | $O(n)$ |
| Amortized | - | - | - | $O(\log n)$ |

---

# 十一、Heap

Heap：

$$
\boxed{不是\ BST}
$$

但 Binary Heap：

$$
\boxed{一定是\ Complete\ Binary\ Tree}
$$

## Max Heap

```text
Parent ≥ Children
```

Root = Maximum

## Min Heap

```text
Parent ≤ Children
```

Root = Minimum

## Binary Heap Complexity

| Operation | Complexity |
|---|---:|
| Find Max / Min | $O(1)$ |
| Insert | $O(\log n)$ |
| Delete Max / Min | $O(\log n)$ |
| Build Heap | $O(n)$ |
| Search 任意值 | $O(n)$ |

---

# 十二、Leftist Heap

Leftist Heap：

- Heap Ordered
- 不一定 Complete
- 不是 BST
- 核心是 Merge

Leftist Property：

$$
\boxed{NPL(left)\ge NPL(right)}
$$

## Complexity

| Operation | Complexity |
|---|---:|
| Merge | $O(\log n)$ |
| Insert | $O(\log n)$ |
| Delete-Min | $O(\log n)$ |
| Find-Min | $O(1)$ |

---

# 十三、Binomial Tree

$B_k$：

### Node 數

$$
\boxed{2^k}
$$

### Height

$$
\boxed{k}
$$

### Root Degree

$$
\boxed{k}
$$

---

# 十四、Binomial Heap

Binomial Heap = 很多棵 Binomial Tree 組成的 Forest。

規定：

> 同一個 Degree 最多只能有一棵 Tree。

## Complexity

| Operation | Complexity |
|---|---:|
| Make Heap | $O(1)$ |
| Find-Min | $O(\log n)$ |
| Insert | $O(\log n)$ |
| Merge | $O(\log n)$ |
| Extract-Min | $O(\log n)$ |
| Decrease-Key | $O(\log n)$ |
| Delete | $O(\log n)$ |

---

# 十五、Fibonacci Heap

Fibonacci Heap 可以想成：

> 更 Lazy 的 Binomial Heap。

Binomial Heap：

```text
Merge 時馬上整理相同 Degree
```

Fibonacci Heap：

```text
Merge 時先放著
Extract-Min 時才整理
```

稱為 Lazy Consolidation。

## Complexity

| Operation | Amortized |
|---|---:|
| Make Heap | $O(1)$ |
| Find-Min | $O(1)$ |
| Insert | $O(1)$ |
| Merge / Union | $O(1)$ |
| Extract-Min | $O(\log n)$ |
| Decrease-Key | $O(1)$ |
| Delete | $O(\log n)$ |

---

# 十六、Heap 家族比較

| Operation | Binary Heap | Leftist Heap | Binomial Heap | Fibonacci Heap |
|---|---:|---:|---:|---:|
| Find-Min | $O(1)$ | $O(1)$ | $O(\log n)$ | $O(1)$ |
| Insert | $O(\log n)$ | $O(\log n)$ | $O(\log n)$ | $O(1)^*$ |
| Delete-Min | $O(\log n)$ | $O(\log n)$ | $O(\log n)$ | $O(\log n)^*$ |
| Merge | $O(n)$ | $O(\log n)$ | $O(\log n)$ | $O(1)$ |
| Decrease-Key | $O(\log n)$ | - | $O(\log n)$ | $O(1)^*$ |

> $*$ = Amortized Complexity

---

# 十七、B-Tree

B-Tree：

$$
\boxed{Balanced\ M-Way\ Search\ Tree}
$$

不是 Binary Search Tree。

## B-Tree Order m

最大 Child：

$$
\boxed{m}
$$

最大 Key：

$$
\boxed{m-1}
$$

非 Root Node 最少 Child：

$$
\boxed{\left\lceil\frac m2\right\rceil}
$$

最少 Key：

$$
\boxed{\left\lceil\frac m2\right\rceil-1}
$$

所有 Leaf：

$$
\boxed{都在同一層}
$$

所以一定 Balanced。

## Complexity

| Operation | Complexity |
|---|---:|
| Search | $O(\log n)$ |
| Insert | $O(\log n)$ |
| Delete | $O(\log n)$ |

---

# 十八、2-3 Tree

$$
\boxed{B\text{-}Tree\ of\ Order\ 3}
$$

可以有：

```text
2 children
或
3 children
```

所以一個 Node 有：

```text
1 key
或
2 keys
```

---

# 十九、2-3-4 Tree

$$
\boxed{B\text{-}Tree\ of\ Order\ 4}
$$

可以有：

```text
2 children
3 children
4 children
```

所以一個 Node 有：

```text
1 key
2 keys
3 keys
```

重要關係：

$$
\boxed{2\text{-}3\text{-}4\ Tree\leftrightarrow Red\text{-}Black\ Tree}
$$

---

# 二十、B+ Tree

B+ Tree 是 B-Tree 的變形。

特色：

```text
Internal Node
→ 主要存 Index / Key

Leaf
→ 存真正 Record / Data
```

Leaf 彼此通常用 Linked List 串起來。

## Complexity

| Operation | Complexity |
|---|---:|
| Search | $O(\log n)$ |
| Insert | $O(\log n)$ |
| Delete | $O(\log n)$ |
| Range Search | 非常適合 |

---

# 二十一、B-Tree vs B+ Tree

| | B-Tree | B+ Tree |
|---|---|---|
| Internal Node | 可放 Data | 主要放 Index |
| Leaf | 可放 Data | 放全部 Data |
| Leaf Linked List | 不一定 | ✅ |
| Range Search | 普通 | 很強 |
| Balanced | ✅ | ✅ |

---

# 二十二、Huffman Tree

Huffman Tree 用於 Data Compression，產生 Optimal Prefix Code。

建立方法：

1. 找最小的兩個 Frequency
2. 合併
3. 放回 Priority Queue
4. 重複

Huffman Tree 通常是：

$$
\boxed{Full\ Binary\ Tree}
$$

所以：

$$
E_0=E_2+1
$$

但：

- ❌ 不是 BST
- ❌ 不一定 Balanced
- ❌ 不一定 Complete

如果使用 Min Heap：

$$
\boxed{O(n\log n)}
$$

---

# 二十三、Tree 最重要時間複雜度總表

| Structure | Search | Insert | Delete | 特殊操作 |
|---|---:|---:|---:|---:|
| BST | $O(h)$ | $O(h)$ | $O(h)$ | - |
| BST Worst | $O(n)$ | $O(n)$ | $O(n)$ | - |
| AVL | $O(\log n)$ | $O(\log n)$ | $O(\log n)$ | Rotation |
| Red-Black | $O(\log n)$ | $O(\log n)$ | $O(\log n)$ | Rotation + Recolor |
| Splay | $O(\log n)^*$ | $O(\log n)^*$ | $O(\log n)^*$ | Splay |
| Binary Heap | $O(n)$ | $O(\log n)$ | $O(\log n)$ | Min/Max $O(1)$ |
| Leftist Heap | - | $O(\log n)$ | $O(\log n)$ | Merge $O(\log n)$ |
| Binomial Heap | - | $O(\log n)$ | $O(\log n)$ | Merge $O(\log n)$ |
| Fibonacci Heap | - | $O(1)^*$ | $O(\log n)^*$ | Merge $O(1)$ |
| B-Tree | $O(\log n)$ | $O(\log n)$ | $O(\log n)$ | Disk Search |
| B+ Tree | $O(\log n)$ | $O(\log n)$ | $O(\log n)$ | Range Search |
| Huffman | - | - | - | Build $O(n\log n)$ |

> $*$ = Amortized Complexity

---

# 二十四、考試最容易搞混

## Balanced ≠ Complete

AVL、RB Tree、B-Tree 都是 Balanced，但不代表 Complete。

## Heap ≠ BST

Heap：

```text
Parent ≥ Child
```

或：

```text
Parent ≤ Child
```

BST：

```text
Left < Root < Right
```

完全不同。

## AVL 是 BST

$$
\boxed{AVL\subset BST}
$$

## Red-Black Tree 是 BST

$$
\boxed{RB\ Tree\subset BST}
$$

## Splay Tree 是 BST

$$
\boxed{Splay\ Tree\subset BST}
$$

## 2-3 / 2-3-4 Tree 是 B-Tree 特例

$$
\boxed{2\text{-}3\ Tree=B\text{-}Tree\ Order\ 3}
$$

$$
\boxed{2\text{-}3\text{-}4\ Tree=B\text{-}Tree\ Order\ 4}
$$

## Red-Black Tree 與 2-3-4 Tree

$$
\boxed{RB\ Tree\leftrightarrow2\text{-}3\text{-}4\ Tree}
$$

---

# 二十五、最後考前速記表

| Tree | 一句話記憶 |
|---|---|
| Binary Tree | 每個 Node 最多 2 個 Child |
| Full Binary Tree | 每個 Node 只能有 0 或 2 個 Child |
| Complete Binary Tree | 一層一層、從左到右填滿 |
| BST | Left < Root < Right |
| AVL | BST + 高度差最多 1 |
| Red-Black | BST + 顏色維持近似平衡 |
| Splay | BST + 用過的搬到 Root |
| Binary Heap | Complete + Parent/Child 大小關係 |
| Leftist Heap | 為 Merge 而生 |
| Binomial Heap | 很多棵不同 Degree 的 Binomial Tree |
| Fibonacci Heap | Lazy Binomial Heap |
| B-Tree | Balanced Multi-Way Search Tree |
| 2-3 Tree | B-Tree Order 3 |
| 2-3-4 Tree | B-Tree Order 4 |
| B+ Tree | Data 放 Leaf，Leaf 串起來 |
| Huffman | 最小兩個一直合併，做 Prefix Code |

---

# 二十六、最重要的包含關係

```text
AVL Tree
   ↓
  BST
   ↓
Binary Tree
```

```text
Red-Black Tree
      ↓
     BST
      ↓
 Binary Tree
```

```text
Splay Tree
    ↓
   BST
    ↓
Binary Tree
```

```text
2-3 Tree
   ↓
 B-Tree
```

```text
2-3-4 Tree
     ↓
   B-Tree
```

```text
Red-Black Tree
      ↕
  2-3-4 Tree
```

```text
Binary Heap
     ↓
Complete Binary Tree
     ↓
 Binary Tree
```

但：

```text
Binary Heap ≠ BST
```
