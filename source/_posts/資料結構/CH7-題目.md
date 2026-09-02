---
title: 資料結構 - Search and Sort - 題目注意
categories:
  - 研究所題目
  - 資料結構
  - 第七章
tags:
  - 資料結構
abbrlink: '514e5281'
mathjax: true
date: '2026-8-21 16:12:00'
---
- Ordered-Linked-List 就算有排序也不能直接Binary Sort
- 兩個Array，大的n個，小的m個，要找兩個有沒有相同的 => $O(nlogm)$
    - `m` Array 先 Sort => O(mlogm)
    - 把 `n` Array從頭跑到尾，並且在`m` Array Binary Search => O(n)*O(logm) = $O(nlogm)$
- Singly Linked List 最適合使用 **Insertion Sort** 可以減少Data Movement
    - Array
        - → Random Access 很強
        - → Quick Sort / Heap Sort 很方便

    - Linked List
        - → 插入、合併很強
        - → Insertion Sort / Merge Sort 很方便

- 在限定使用**Comparison-Based** Skill 情況下，最快 $O(n log n)$
- 如果沒有這個限制最快到 $O(n)$
- Q : All Sorting Method 最快可到達 $\omega(n \log n)$ : **False**
  - EX : Counting Sort : O(n+k) if k = O(n) => O(n)
- 要證明 $\omega(n \log n)$ 要用Desision Tree
- substitution mthod : 代入法/展開法
- Qucik Sort 的 Recursive formula
  - Best : 2*T(n/2) + cn
  - Worst : T(n-1) + T(0) + cn
- 讓Unstable變得Stable 方法
  - Quick Sort : 每一個 record 使用兩個 key：
    - Primary key = 原本要排序的 key
    - Secondary key = 原本在 Array 的 index
  - Merge 兩個 array 的時候，如果遇到相同 key，先放第一個 array 的元素
  - **Perfect** Hashing的Worst Case : O(1)
- External Sort 名詞
  - Run 一段已經排好的資料
  - k-way Merge : 同時合併k個Runs
  - Pass : 所有目前Runs 都完整處理一輪 : 16 -> 4 -> 1 : 2 Passes
- Heap Sort 的經典不穩定例子是 [2a, 2b, 1]，且 Heapify 左右一樣大時標準做法一律優先拉「左邊」。
- 一樣大就「留在原地不交換」， 因為 沒有「嚴格大於」
- Merge Sort 三步驟 : Divide -> Conquer -> Combine
- merge / bubble / insertion 為Stable


- 用途：找第 $k$ 小，不需將整個 Array 排序。
  - 方法：
    1. 選 Pivot 做 Partition
    2. 判斷 Pivot 是第幾小
    3. 若：
      - Pivot位置 = k → 找到
      - Pivot位置 > k → 找左邊
      - Pivot位置 < k → 找右邊
    4. 每次只需處理其中一邊

    時間複雜度：
    - Average：$O(n)$
- Worst：$O(n^2)$

- Deterministic Quick Sort : 只選第一個或最後一個
- Perfect sorted : 已經完全排好

- Prim : O(ElogV)
- Kruskal : O(ElogE)