---
title: 資料結構 - Search and Sort
categories:
  - 資料結構
  - 第七章
tags:
  - 資料結構
abbrlink: 513a4877
mathjax: true
date: '2026-8-28 10:18:00'
---

# Search and Sort

# Search

## Linear(Sequential) Search
- 由左至右一一比較
- Time Compexity : $O(1)$

## Binary Search
- 資料要先**排序**
- 需要保存在 **Array**
- Time Compexity : $O(log n)$

### 方法 
- 每次都與中間比較
    - 相等 : 找到
    - `x` < 中間值 : 在左半部再做一次
    - `x` > 中間值 : 在右半部再做一次

# Sort

- Internal Sorting : 資料量很小，**可以全部放在Memory中**
- External Sorting : 資料量太大，需要藉助外部儲存體
    - 常見External Sort : **Merge Sort**(也可以搭配**Sellection Tree**)、M-Way Tree、B Tree、B+ Tree

## Stable and unStable

- Input Data : ...,K,...,K+,...
- OutPut Data保證結果為...,K,K+,... 則為**Stable**
- UnStable Sort會比Stable**多不必要的資料交換**

例子 : 
- Stable Sort : Insertion,Bubble Sort,Merge Sort,Bucket Sort,Radix Sort,Counting Sort
- Unstable Sort : Seletion,Shell,Quick,heap

## Sorting In-Place
- 排序時，幾乎都直接在原本陣列裡面交換，**不另外開一個跟n一樣大的陣列**
- **Merge Sort** 不是 In-Place
- Linear-Time Sorting 也不是 Inplace

# 初等 Sort 方法

## Insertion Sort

概念：
- 將資料分成「已排序區」與「未排序區」
- 每次從未排序區拿一個元素，插入前面已排序區的正確位置
- 類似整理撲克牌
- 適合資料量小或 Almost Sorted 的資料

範例：

```text
5 2 4 1

[5] 2 4 1
[2 5] 4 1
[2 4 5] 1
[1 2 4 5]
```

時間複雜度：

| Case | Time |
|---|---:|
| Best | $O(n)$ |
| Average | $O(n^2)$ |
| Worst | $O(n^2)$ |

- Best：原本已經由小到大排列
- Worst：原本由大到小排列
- Worst 比較次數：

$$
1+2+\cdots +(n-1)=\frac{n(n-1)}{2}=O(n^2)
$$

特性：
- Space：$O(1)$
- In-Place：✅
- Stable：✅

---

## Selection Sort

概念：
- 每一回合從「尚未排序區」找出最小值
- 再把最小值與目前位置交換
- 每回合最多 Swap 一次

範例：

```text
5 2 4 1 3

找最小 1
1 | 2 4 5 3

找最小 2
1 2 | 4 5 3

找最小 3
1 2 3 | 5 4

1 2 3 4 5
```

比較次數固定：

$$
(n-1)+(n-2)+\cdots+1
$$

$$
=\frac{n(n-1)}{2}
=O(n^2)
$$

所以即使資料本來就已排序：

$$
\boxed{Best=O(n^2)}
$$

時間複雜度：

| Case | Time |
|---|---:|
| Best | $O(n^2)$ |
| Average | $O(n^2)$ |
| Worst | $O(n^2)$ |

特性：
- Space：$O(1)$
- In-Place：✅
- Stable：❌
- Swap 次數少，最多約 $n-1$ 次
- 適合 Data Movement 成本很高的情況
> 適合「交換 / 搬動資料成本高」的情況，因為 Selection Sort 的 Swap 次數少

Unstable 範例：

```text
5A 5B 3
```

Selection Sort 將 3 與第一個 5 交換：

```text
3 5B 5A
```

原本：

```text
5A 在 5B 前
```

排序後變成：

```text
5B 在 5A 前
```

所以 Unstable。

---

## Bubble Sort

概念：
- 每次比較相鄰兩個元素
- 若左邊 > 右邊，就 Swap
- 每一回合會把目前最大的元素「冒」到最右邊

範例：

```text
5 2 4 1

5 2 → Swap
2 5 4 1

5 4 → Swap
2 4 5 1

5 1 → Swap
2 4 1 5
        ↑
      最大值完成
```

若一整個 Pass 都沒有發生 Swap：

```text
flag = 0
```

代表資料已經排序完成，可以提早結束。

時間複雜度：

| Case | Time |
|---|---:|
| Best | $O(n)$ |
| Average | $O(n^2)$ |
| Worst | $O(n^2)$ |

Best：

```text
1 2 3 4 5
```

第一個 Pass 做完完全沒有 Swap，因此直接結束：

$$
T(n)=n-1=O(n)
$$

Worst：

```text
5 4 3 2 1
```

比較次數：

$$
(n-1)+(n-2)+\cdots+1
$$

$$
=\frac{n(n-1)}2
=O(n^2)
$$

特性：
- Space：$O(1)$
- In-Place：✅
- Stable：✅

Stable 原因：
只有：

```cpp
arr[j] > arr[j+1]
```

才交換。

如果兩個元素相等，就不交換，因此相同 Key 的相對順序不變。

---

## Shell Sort (補充)

概念：
- Insertion Sort 的改良版
- 不一開始只比較相鄰元素
- 先用較大的 Gap（Span）比較相隔較遠的元素
- Gap 逐漸縮小
- gap = k 時，把「索引相差 k」的元素分成同一組，每一組做 Insertion Sort。
- 最後一定要：

$$
\boxed{gap=1}
$$

最後一輪其實就是 Insertion Sort。

常見 Gap：

```text
n/2 → n/4 → n/8 → ... → 1
```

例如：

```text
n = 16

gap：
8 → 4 → 2 → 1
```

目的：
- 先讓距離很遠但位置差很多的元素快速移動
- 讓資料變成 Almost Sorted
- 最後再用 gap = 1 快速完成排序

時間複雜度：
- 與 Gap Sequence 有關
- 考試若依課本可記：

| Case | Time |
|---|---:|
| Best | 與 Gap 有關，目前已知最好$O(n^{7/6})$，考試可以寫 $O(n^{3/2})$ |
| Average | $O(n^2)$ |
| Worst | $O(n^2)$ |

特性：
- Space：$O(1)$
- In-Place：✅
- Stable：❌

Unstable 原因：
- Shell Sort 可能讓元素跨越很遠的位置交換
- 相同 Key 的相對順序可能因此改變

### Shell Sort Example

```text
8 5 3 9 1 6 4 7

gap = 4
→ 比距離 4 的元素
→ 1 5 3 7 8 6 4 9

gap = 2
→ 比距離 2 的元素
→ 1 5 3 6 4 7 8 9

gap = 1
→ 比相鄰元素
→ 1 3 5 4 6 7 8 9
→ 1 3 4 5 6 7 8 9
```
- Gap 逐漸縮小
- 可以讓元素一次移動很遠
- 最後一定要 gap = 1
- gap = 1 時就是最後的細部排序
- 先粗排再細排

---

## 初等 Sort 比較

| Sort | Best | Average | Worst | Space | Stable | In-Place |
|---|---:|---:|---:|---:|---|---|
| Insertion Sort | $O(n)$ | $O(n^2)$ | $O(n^2)$ | $O(1)$ | ✅ | ✅ |
| Selection Sort | $O(n^2)$ | $O(n^2)$ | $O(n^2)$ | $O(1)$ | ❌ | ✅ |
| Bubble Sort | $O(n)$ | $O(n^2)$ | $O(n^2)$ | $O(1)$ | ✅ | ✅ |
| Shell Sort | Gap 決定 | $O(n^2)$ | $O(n^2)$ | $O(1)$ | ❌ | ✅ |

---

# 高等 Sort 方法

## Quick Sort

Quick Sort 採用：

> **Divide and Conquer**

核心想法：

1. 選一個 **Pivot Key (`pk`)**
2. 執行 `Partition`
3. 將資料切成兩邊：
   - 左邊：$\le pk$
   - 右邊：$\ge pk$
4. 左右兩邊再分別做 Quick Sort

```text
原資料

[ 8, 2, 7, 1, 5, 6, 4 ]
                    ↑
                   Pivot

Partition 後

[ 2, 1, 3 ] [ 4 ] [ 7, 5, 6, 8 ]
               ↑
             Pivot

左半再 Quick Sort
右半再 Quick Sort
```

---

### Partition

Partition 的目的就是：

> 找出 Pivot 最後應該放的位置，並把資料分成左右兩邊。

例如：

```text
[2, 8, 7, 1, 3, 5, 6, 4]

Pivot = 4
```

Partition 後可能變成：

```text
[2, 1, 3] [4] [7, 5, 6, 8]
           ↑
         Pivot
```

因此：

```text
左邊 <= 4
右邊 >= 4
```

---

### Lomuto Partition

演算法課常見版本：

```text
Partition(A, p, r)
{
    pk = A[r]
    i = p - 1

    for j = p to r-1
    {
        if A[j] <= pk
        {
            i++
            swap(A[i], A[j])
        }
    }

    swap(A[i+1], A[r])

    return i+1
}
```

概念：

```text
i：目前「小於等於 pivot 區域」的最後位置
j：負責掃描資料
```

最後：

```text
swap(A[i+1], pivot)
```

Pivot 就放到正確位置。

---

### Hoare Partition

資料結構課常見版本：

- Pivot 通常取最左邊
- `i` 從左往右找
- `j` 從右往左找

```text
i 找 >= Pivot 的值
j 找 <= Pivot 的值
```

如果：

```text
i < j
```

就交換。

```text
       i          j
       ↓          ↓
[小 小 大 ...... 小 大]
```

直到：

```text
i >= j
```

停止。

---

### Quick Sort 時間複雜度

| Case | Time |
|---|---:|
| Best | $O(n\log n)$ |
| Average | $O(n\log n)$ |
| Worst | $O(n^2)$ |

---

### Best Case

每次 Pivot 都剛好切成兩半：

```text
           n
        /     \
      n/2     n/2
     /  \     /  \
   n/4 n/4  n/4 n/4
```

因此：

$$
T(n)=2T(n/2)+cn
$$

共有：

$$
\log n
$$

層。

每層 Partition 總共處理：

$$
O(n)
$$

所以：

$$
\boxed{O(n\log n)}
$$

---

### Worst Case

如果每次 Pivot 都是：

```text
最小值
```

或：

```text
最大值
```

就會切成：

```text
0 + (n-1)
0 + (n-2)
0 + (n-3)
...
```

因此：

$$
T(n)=T(n-1)+cn
$$

總時間：

$$
n+(n-1)+(n-2)+\cdots+1
$$

$$
=\frac{n(n+1)}2
$$

所以：

$$
\boxed{O(n^2)}
$$

---

### Quick Sort Worst Case 常見情況

例如輸入已經排序：

```text
1 2 3 4 5 6 7
```

如果每次都選最右邊：

```text
Pivot = 最大值
```

則會一直變成：

```text
6 + 0
5 + 0
4 + 0
...
```

因此變成：

$$
O(n^2)
$$

---

### 改善 Worst Case

核心：

> 盡量不要讓 Pivot 一直選到 Min 或 Max。

#### Randomized Quick Sort

隨機選 Pivot：


複雜度：

| Case | Time |
|---|---:|
| Best | $O(n\log n)$ |
| Average | $O(n\log n)$ |
| Worst | $O(n^2)$ |

注意：

> Randomized Quick Sort 並沒有消滅 Worst Case，只是降低發生機率。

---

#### Median of Three

從：

```text
Left
Middle
Right
```

三個元素中選中位數當 Pivot。

例如：

```text
Left = 2
Middle = 8
Right = 4
```

排序：

```text
2 < 4 < 8
```

所以：

```text
Pivot = 4
```

目的：

> 避免 Pivot 太容易選到極端值。

---

#### Median of Medians

可以保證 Pivot 不會太差。

可使 Worst Case 達到：

$$
\boxed{O(n\log n)}
$$

但實作與常數成本較高。

---

### 所有元素都相同

例如：

```text
5 5 5 5 5 5 5
```

某些 Partition 寫法可能每次都切成：

```text
0 + (n-1)
```

導致：

$$
\boxed{O(n^2)}
$$

但若採用適當的 Hoare Partition 或 Three-way Partition，可以改善這個問題。

---

### Quick Sort 空間複雜度

Quick Sort 本身通常不需要另一個大小為 $n$ 的 Array。

主要額外空間來自：

> Recursive Call Stack

一般：

$$
O(\log n)
$$

Worst Case：

$$
O(n)
$$

所以：

$$
\boxed{O(\log n)\sim O(n)}
$$

---

### Quick Sort 性質

| 性質 | Quick Sort |
|---|---|
| Divide and Conquer | Yes |
| Stable | No |
| In-place | 通常 Yes |
| Best | $O(n\log n)$ |
| Average | $O(n\log n)$ |
| Worst | $O(n^2)$ |
| Extra Space | $O(\log n)\sim O(n)$ |

---

## Merge Sort

Merge Sort 也是：

> **Divide and Conquer**

但是它與 Quick Sort 最大差別：

```text
Quick Sort：
先 Partition
再遞迴

Merge Sort：
先 Divide
再遞迴
最後 Merge
```

---

### 基本概念

Merge Sort：

1. 將資料切成兩半
2. 左半做 Merge Sort
3. 右半做 Merge Sort
4. 將兩個排序好的部分 Merge

```text
[26, 5, 77, 1, 61, 11, 59, 15, 48, 19]

              ↓ divide

[26,5,77,1,61]        [11,59,15,48,19]

              ↓

繼續切到只剩一個元素

              ↓

再慢慢 Merge 回去
```

---

### Run

Merge Sort 常出現：

> **Run**

Run：

> 一段已經排序好的連續資料。

例如：

```text
[1, 5, 26, 77]
```

就是一個 Run。

Run Length：

> 每一個 Run 裡面的資料個數。

---

### Merge Two Runs

假設兩個已排序的 Runs：

```text
L = [1, 5, 26, 77]
M = [11, 15, 59, 61]
```

每次比較兩邊最前面的元素：

```text
1 vs 11 → 1
5 vs 11 → 5
26 vs 11 → 11
26 vs 15 → 15
26 vs 59 → 26
77 vs 59 → 59
77 vs 61 → 61
```

最後：

```text
[1, 5, 11, 15, 26, 59, 61, 77]
```

---

### Merge 的時間複雜度

假設：

```text
左 Run 大小 = n1
右 Run 大小 = n2
```

最多比較：

$$
n_1+n_2-1
$$

因此：

$$
\boxed{O(n)}
$$

其中：

$$
n=n_1+n_2
$$

---

## Recursive Merge Sort

程式概念：

```text
mergeSort(arr, l, r)
{
    if (l < r)
    {
        m = l + (r-l)/2

        mergeSort(arr, l, m)
        mergeSort(arr, m+1, r)

        merge(arr, l, m, r)
    }
}
```

流程：

```text
               [8 elements]

              /           \
          [4 elements]   [4 elements]

           /   \           /   \
         [2]   [2]       [2]   [2]

         / \   / \       / \   / \
        [1][1][1][1]   [1][1][1][1]

               ↓ Merge

               Sorted
```

---

### Merge Sort 時間複雜度

每次切成：

$$
\frac n2
$$

所以：

$$
T(n)=2T(n/2)+cn
$$

Tree 高度：

$$
\log_2 n
$$

而每一層所有 Merge 加起來都是：

$$
O(n)
$$

因此：

$$
n\times\log n
$$

所以：

$$
\boxed{O(n\log n)}
$$

---

### 為什麼 Best Case 也是 $O(n\log n)$？

即使原本就是：

```text
1 2 3 4 5 6 7 8
```

Merge Sort 還是會照樣：

```text
切
切
切
Merge
Merge
Merge
```

所以：

| Case | Time |
|---|---:|
| Best | $O(n\log n)$ |
| Average | $O(n\log n)$ |
| Worst | $O(n\log n)$ |

---

## Iterative Merge Sort

不使用 Recursive。

一開始：

```text
Run Length = 1
```

例如：

```text
[26] [5] [77] [1] [61] [11] [59] [15]
```

兩兩合併：

```text
[5,26] [1,77] [11,61] [15,59]
```

Run Length：

```text
2
```

再合併：

```text
[1,5,26,77] [11,15,59,61]
```

Run Length：

```text
4
```

最後：

```text
[1,5,11,15,26,59,61,77]
```

每次：

```text
1 → 2 → 4 → 8 → ...
```

所以總共約：

$$
\log_2 n
$$

輪。

每一輪：

$$
O(n)
$$

因此：

$$
\boxed{O(n\log n)}
$$

---

## Merge Sort 空間複雜度

Merge 時通常需要：

```text
Temporary Array
```

用來暫存合併結果。

需要：

$$
\boxed{O(n)}
$$

額外空間。

因此一般 Array Merge Sort：

> **不是 In-place Sorting**

---

### Merge Sort 為什麼 Stable？

例如：

```text
3A  5  3B
```

其中：

```text
3A 和 3B 的 Key 都是 3
```

Stable Sorting 要求排序後：

```text
3A
```

仍然要在：

```text
3B
```

前面。

Merge 時若相等使用：

```text
L[i] <= R[j]
```

優先拿左邊：

```text
3A → 3B
```

因此 Merge Sort 可以保持原本相同 Key 的相對順序。

所以：

$$
\boxed{\text{Merge Sort is Stable}}
$$

---

## Quick Sort vs Merge Sort

| | Quick Sort | Merge Sort |
|---|---:|---:|
| 方法 | Divide & Conquer | Divide & Conquer |
| Best | $O(n\log n)$ | $O(n\log n)$ |
| Average | $O(n\log n)$ | $O(n\log n)$ |
| Worst | $O(n^2)$ | $O(n\log n)$ |
| Stable | No | Yes |
| In-place | Yes（通常） | No（Array 版本） |
| Extra Space | $O(\log n)\sim O(n)$ | $O(n)$ |
| 核心操作 | Partition | Merge |
| 主要優點 | 實務通常很快 | 時間穩定、Stable |


## Heap Sort
Heap Sort 的核心：

> 先建立 **Max Heap**，再不斷把 Root（最大值）放到陣列最後。

### 方法

1. 用 **Bottom-Up** 建立 Max Heap：$O(n)$
2. 將 Root 與目前最後一個元素 Swap
3. 排除已經排好的最後一格
4. 對 Root 做 Adjust / Heapify：$O(\log n)$
5. 重複 $n-1$ 回合

```text
Max Heap

        77
      /    \
    61      59
   /  \    / \
  ...

Swap Root 與最後一個元素
→ 77 放到最後
→ 剩下部分重新 Adjust 成 Max Heap
→ 再取下一個最大值
```

### 時間複雜度

建立 Heap：

$$
O(n)
$$

排序階段共約 $n-1$ 回合，每回合 Adjust：

$$
O(\log n)
$$

所以：

$$
O(n)+O(n\log n)=\boxed{O(n\log n)}
$$

| Case | Time |
|---|---:|
| Best | $O(n\log n)$ |
| Average | $O(n\log n)$ |
| Worst | $O(n\log n)$ |

特性：
- Space：$O(1)$
- In-Place：✅
- Stable：❌

### Heap Sort Example

假設：

```text
A = [4, 10, 3, 5, 1]
```

Heap Sort 要做遞增排序，所以先建立 **Max Heap**。

#### Step 1：Bottom-Up 建 Max Heap

原本：

```text
        4
       / \
     10   3
    / \
   5   1
```

建立 Max Heap：

```text
       10
      /  \
     5    3
    / \
   4   1
```

Array：

```text
[10, 5, 3, 4, 1]
```

---

#### Step 2：Root 與最後一個元素 Swap

```text
[1, 5, 3, 4, 10]
```

此時 `10` 已經排好，不再參與 Heap。

對剩下的資料 Adjust：

```text
       5
      / \
     4   3
    /
   1
```

得到：

```text
[5, 4, 3, 1, 10]
```

---

#### Step 3：再次 Swap + Adjust

Root `5` 與目前 Heap 最後的 `1` Swap：

```text
[1, 4, 3, 5, 10]
```

剩下重新 Adjust：

```text
       4
      / \
     1   3
```

得到：

```text
[4, 1, 3, 5, 10]
```

---

#### Step 4：繼續 Swap + Adjust

```text
[4, 1, 3, 5, 10]

→ Swap

[3, 1, 4, 5, 10]

→ Swap

[1, 3, 4, 5, 10]
```

排序完成。

### Heap Sort 流程

```text
[4,10,3,5,1]

建 Max Heap
↓
[10,5,3,4,1]

Swap + Adjust
↓
[5,4,3,1,10]

Swap + Adjust
↓
[4,1,3,5,10]

Swap + Adjust
↓
[3,1,4,5,10]

Swap
↓
[1,3,4,5,10]
```

記法：

1. **Bottom-Up 建 Max Heap**
2. **Root 和 Heap 最後一個元素交換**
3. **Heap Size - 1**
4. **對 Root 做 Adjust**
5. 重複直到排序完成

> Heap Sort 每一回合都把「目前最大值」放到 Array 最後面，放好之後就不再參與 Heap。


---

## Selection Tree

用途：

> 加速 **k-way Merge**，從 $k$ 個已排序 Runs 中反覆找出最小值。

例如有 $k$ 個 Runs：

```text
Run 1：5  10 20
Run 2：6  16 26
Run 3：4  14 24
Run 4：3  13 23
...
```

每次只需要比較每個 Run 最前面的元素。

Selection Tree 是一棵有 $k$ 個 Leaf 的 Binary Tree，高度約：

$$
\log_2 k
$$

### Winner Tree

- Leaf：各 Run 目前最前面的值
- 兩兩比較
- 較小者往上晉級
- Root 為目前所有 Runs 的最小值

```text
          1
        /   \
       3     1
      / \   / \
     5   3 2   1
    /\  /\ /\ /\
   5 6 4 3 7 2 9 1
```

#### Example

假設目前有 4 個 Runs：

```text
R1 = 7
R2 = 3
R3 = 9
R4 = 5
```

兩兩比較：

```text
7 vs 3
→ 3 Winner

9 vs 5
→ 5 Winner

3 vs 5
→ 3 Winner
```

所以 Winner Tree：

```text
            3
          /   \
         3     5
        / \   / \
       7   3 9   5
      R1  R2 R3  R4
```

Root 的 `3` 就是目前所有 Runs 中的最小值。

假設輸出 `3` 後，R2 的下一筆資料是 `8`：

```text
R1 = 7
R2 = 8
R3 = 9
R4 = 5
```

只需要沿著 R2 原本的路徑重新比較：

```text
7 vs 8
→ 7

7 vs 5
→ 5
```

新的 Winner Tree：

```text
            5
          /   \
         7     5
        / \   / \
       7   8 9   5
      R1  R2 R3  R4
```

所以新的 Winner 為：

```text
5
```

輸出 Winner 後，只需：

1. 從 Winner 所屬 Run 補入下一筆資料
2. 沿著原本 Winner 的路徑重新比較到 Root

因此每輸出一筆只需：

$$
\boxed{O(\log k)}
$$

建樹：

$$
O(k)
$$

若總共有 $n$ 筆資料：

$$
\boxed{O(k+n\log k)}
$$

通常寫成：

$$
\boxed{O(n\log k)}
$$

---

### Loser Tree

概念與 Winner Tree 類似，但：

- 內部節點記錄每次比較的 **Loser**
- 最後 Winner 另外保存
- Winner 更新後，只需要沿原路與之前的 Loser 比較
- 把輸的留下來
- 贏的晉級上去 ， 最後比完 贏的那一格要遞補一個繼續比
#### Example

使用同樣的資料：

```text
R1 = 7
R2 = 3
R3 = 9
R4 = 5
```

第一次：

```text
7 vs 3

Winner = 3
Loser = 7
```

第二次：

```text
9 vs 5

Winner = 5
Loser = 9
```

最後：

```text
3 vs 5

Winner = 3
Loser = 5
```

所以：

```text
          Winner = 3
               |
               5
             /   \
            7     9
```

樹中的：

```text
7、9、5
```

都是每一次比較留下來的 Loser。

真正的 Winner：

```text
3
```

另外保存。

可以簡單記：

```text
Winner Tree：
贏的人存在 Internal Node

Loser Tree：
輸的人存在 Internal Node
最後 Winner 另外保存
```

主要用途同樣是：

> **k-way Merge**

複雜度同樣：

$$
\boxed{O(n\log k)}
$$
---

# Linear-Time Sorting Methods

這類排序不是 Comparison-Based Sorting。

| Method(資結版) |Method(演算法版)| 核心概念 |
|---|---|---|
| LSD Radix Sort| Radix Sort | 從最低位數一路排到最高位數 |
| MSD Radix Sort | Bucket Sort | 將資料分配到不同 Bucket，再各自排序 |
| Counting Sort | Counting Sort | 統計每個 Key 出現次數 |

---

## LSD Radix Sort

採用：

> **Distribution + Merge**

不是靠兩兩比較大小排序。

假設：
- $n$：Data 數量
- $d$：最大值的位數
- $r$：Base，也就是 Bucket 數量

例如十進位：

$$
r=10
$$

需要 Bucket：

```text
0 1 2 3 4 5 6 7 8 9
```

### 方法

從 **最低位數 → 最高位數**，共執行 $d$ 回合。

例如：

```text
179, 208, 306, 93, 859, 984, 55, 9, 271, 33
```

```text
Pass 1：個位數
Pass 2：十位數
Pass 3：百位數
```

每一回合：

1. Distribution：依目前位數放入對應 Bucket
2. Merge：依 Bucket `0 → r-1` 合併回去

Bucket 中資料必須維持 **FIFO**，才能保持 Stable。

### 時間複雜度

每一回合：

```text
Distribution：O(n)
Merge：O(n+r)
```

共 $d$ 回合：

$$
\boxed{O(d(n+r))}
$$

若 $d$、$r$ 可視為常數：

$$
\boxed{O(n)}
$$

特性：
- Stable：✅
- Comparison-Based：❌

### LSD Radix Sort Example

假設：

```text
A = [179, 208, 306, 93, 859, 984, 55, 9, 271, 33]
```

最大值有 3 位數，所以：

```text
d = 3
```

因此總共做：

```text
3 Pass
```

不足的位數視為 `0`。

---

#### Pass 1：看個位數

```text
179 → 9
208 → 8
306 → 6
 93 → 3
859 → 9
984 → 4
 55 → 5
  9 → 9
271 → 1
 33 → 3
```

Distribution：

```text
Bucket 0：

Bucket 1：271

Bucket 2：

Bucket 3：93, 33

Bucket 4：984

Bucket 5：55

Bucket 6：306

Bucket 7：

Bucket 8：208

Bucket 9：179, 859, 9
```

依照 Bucket：

```text
0 → 1 → 2 → ... → 9
```

Merge：

```text
271, 93, 33, 984, 55, 306, 208, 179, 859, 9
```

---

#### Pass 2：看十位數

使用上一輪的結果：

```text
271 → 7
 93 → 9
 33 → 3
984 → 8
 55 → 5
306 → 0
208 → 0
179 → 7
859 → 5
  9 → 0
```

Distribution：

```text
Bucket 0：306, 208, 9

Bucket 1：

Bucket 2：

Bucket 3：33

Bucket 4：

Bucket 5：55, 859

Bucket 6：

Bucket 7：271, 179

Bucket 8：984

Bucket 9：93
```

Merge：

```text
306, 208, 9, 33, 55, 859, 271, 179, 984, 93
```

---

#### Pass 3：看百位數

```text
306 → 3
208 → 2
  9 → 0
 33 → 0
 55 → 0
859 → 8
271 → 2
179 → 1
984 → 9
 93 → 0
```

Distribution：

```text
Bucket 0：9, 33, 55, 93

Bucket 1：179

Bucket 2：208, 271

Bucket 3：306

Bucket 4：

Bucket 5：

Bucket 6：

Bucket 7：

Bucket 8：859

Bucket 9：984
```

最後 Merge：

```text
9, 33, 55, 93, 179, 208, 271, 306, 859, 984
```

排序完成。

> LSD Radix Sort 就是「個位 → 十位 → 百位 → ...」，每一輪都 Distribution 再 Merge。

---

## Bucket Sort

核心：

> 將資料依範圍分配到不同 Bucket，各 Bucket 自己排序，最後依序合併。

### 方法

1. 建立 Buckets
2. 將每筆資料分配到對應 Bucket
3. 每個 Bucket 各自 Sort
4. 按 Bucket 順序合併

例如資料位於 $[0,1)$：

```text
0.78, 0.17, 0.39, 0.26, 0.72,
0.94, 0.21, 0.12, 0.23, 0.68
```

可以依第一位小數分 Bucket：

```text
Bucket 1：0.17, 0.12
Bucket 2：0.26, 0.21, 0.23
Bucket 3：0.39
Bucket 6：0.68
Bucket 7：0.78, 0.72
Bucket 9：0.94
```

各 Bucket 排好後再合併即可。

> 分派與最後合併各做一次，不像 LSD Radix Sort 要依位數做 $d$ 回合。

---

## Counting Sort

適合 Key 是有限整數範圍：

$$
0\sim k
$$

核心：

> 不比較元素，而是直接統計每個 Key 出現幾次。

假設：

```text
A = [2, 5, 3, 0, 2, 3, 0, 3]
Key Range = 0 ~ 5
```

### Step 1：Counting

建立：

```text
C[0...k]
```

統計次數：

```text
Key： 0 1 2 3 4 5
C  ： 2 0 2 3 0 1
```

### Step 2：Prefix Sum

令：

$$
C[i]=C[i]+C[i-1]
$$

得到：

```text
Key： 0 1 2 3 4 5
C  ： 2 2 4 7 7 8
```

此時：

> `C[x]` 表示 Key = x 的資料，在 Output 中最後可以放到的位置。

### Step 3：由後往前放入 Output

```text
for j = A.length downto 1
```

依照：

```text
Out[C[A[j]]] = A[j]
C[A[j]]--
```

最後：

```text
0 0 2 2 3 3 3 5
```

### 為什麼要從後往前？

為了保持相同 Key 的原本順序，因此 Counting Sort 可以是：

$$
\boxed{Stable}
$$

這也很重要，因為 Counting Sort 常被拿來當 **Radix Sort 的 Subroutine**。
- 要選Stable的

### 複雜度

初始化 $C$：

$$
O(k)
$$

掃描 Input：

$$
O(n)
$$

Prefix Sum：

$$
O(k)
$$

建立 Output：

$$
O(n)
$$

所以：

$$
\boxed{O(n+k)}
$$

| Case | Time |
|---|---:|
| Best | $O(n+k)$ |
| Average | $O(n+k)$ |
| Worst | $O(n+k)$ |

Space：

$$
\boxed{O(n+k)}
$$

特性：
- Stable：✅
- In-Place：❌
- Comparison-Based：❌

若 $k=O(n)$：

$$
O(n+k)=O(n)
$$

所以可達 Linear Time。




---

# Selection Problem

Selection Problem：

> 從未排序的 $n$ 筆資料中找指定順位的元素，例如 Min、Max 或第 $i$ 小。

## 同時找 Min 與 Max

最直接的方法：

1. 用 $n-1$ 次比較找 Min
2. 剩餘資料再找 Max

約需要：

$$
(n-1)+(n-2)=2n-3
$$

### 改善方法：兩兩比較

每兩個元素先互相比較：

```text
A[i] < A[j]
```

則：

```text
較小者 → 只需要參加 Min 的比較
較大者 → 只需要參加 Max 的比較
```

每一對：
- 先比較彼此：1 次
- 較小者與目前 Min 比：1 次
- 較大者與目前 Max 比：1 次

所以每兩筆資料大約只需要 3 次比較。

總比較次數約：

$$
\boxed{\frac{3}{2}n}
$$

比原本的：

$$
2n-3
$$

更少。

# 排序可以到達多快?
- 在限定使用**Comparison-Based** Skill 情況下，最快 $O(n log n)$
- 如果沒有這個限制最快到 $O(n)$
- Q : All Sorting Method 最快可到達 $\omega(n \log n)$ : **False**
  - EX : Counting Sort : O(n+k) if k = O(n) => O(n)
- 要證明 $\omega(n \log n)$ 要用Decision Tree
  - Decision Tree  的樹高，可以用leaf多少去反推，例如5個點，共有5!種結果，那樹高就是>=log(5!) + 1 

# Sort 總整理

## Sorting Methods 總表

| Sort | 類型 | Best | Average | Worst | Space | Stable | In-Place | Comparison-Based | 核心概念 |
|---|---|---:|---:|---:|---:|---|---|---|---|
| Insertion Sort | 初等 | $O(n)$ | $O(n^2)$ | $O(n^2)$ | $O(1)$ | ✅ | ✅ | ✅ | 將元素插入已排序區正確位置 |
| Selection Sort | 初等 | $O(n^2)$ | $O(n^2)$ | $O(n^2)$ | $O(1)$ | ❌ | ✅ | ✅ | 每回合找最小值放到前面 |
| Bubble Sort | 初等 | $O(n)$ | $O(n^2)$ | $O(n^2)$ | $O(1)$ | ✅ | ✅ | ✅ | 相鄰比較，把最大值往右冒 |
| Shell Sort | 初等 / 補充 | Gap 決定 | $O(n^2)$ | $O(n^2)$ | $O(1)$ | ❌ | ✅ | ✅ | Gap 逐漸縮小的 Insertion Sort |
| Quick Sort | 高等 | $O(n\log n)$ | $O(n\log n)$ | $O(n^2)$ | $O(\log n)\sim O(n)$ | ❌ | ✅ | ✅ | Pivot + Partition |
| Merge Sort | 高等 | $O(n\log n)$ | $O(n\log n)$ | $O(n\log n)$ | $O(n)$ | ✅ | ❌ | ✅ | Divide + Merge |
| Heap Sort | 高等 | $O(n\log n)$ | $O(n\log n)$ | $O(n\log n)$ | $O(1)$ | ❌ | ✅ | ✅ | 建 Max Heap，再反覆取最大值 |
| LSD Radix Sort | Linear-Time | $O(d(n+r))$ | $O(d(n+r))$ | $O(d(n+r))$ | 額外 Bucket | ✅ | ❌ | ❌ | 從最低位數排到最高位數 |
| Bucket Sort | Linear-Time | 視資料分布 | 視資料分布 | 視桶內排序 | 額外 Bucket | ✅* | ❌ | ❌ | 分 Bucket，各桶排序後合併 |
| Counting Sort | Linear-Time | $O(n+k)$ | $O(n+k)$ | $O(n+k)$ | $O(n+k)$ | ✅ | ❌ | ❌ | 統計 Key 次數 + Prefix Sum |
