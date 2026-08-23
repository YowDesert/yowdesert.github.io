---
title: 資料結構 - 高等樹-題目注意
tags:
  - 資料結構
  - 第九章
abbrlink: 
mathjax: true
date: '2026-8-21 16:12:00'
---
- Double Priority Queue 
    - Insert(x)：插入任意元素
    - Delete-Max：刪除最大值
    - Delete-Min：刪除最小值
- SMMH :
    1. left sibling <= right sibling **(同一個爸爸的兩個子點)**
    > 如果沒有兄弟就不用比

    2. 如果 node `x` 有祖父，則祖父的**左子點**必定 <= X
    3. 如果 node `x` 有祖父，則祖父的**右子點**必定 >= X
- Deap : 才是跟隔壁家兄弟比
- 一個BT的左子樹與右子樹皆為AVL Tree，那這個樹**不一定**是AVL Tree
- 把BST 轉成 AVL Tree -> 花費 $O(n)$ 次Rotation
    ```
    1
     \
      2
       \ 
        3
         \
          ...
           \
            n
    ```
- Blanced BST 花 $O(Log n)$ 在刪除一個key
    - 找到key : $O(Log n)$
    - 刪除加調整 : $O(Log n)$
- 2-3 Tree 高度 h 有幾個點
    - 最少 : $2^h - 1$
    - 最多 : $3^h - 1 / (-2)$
- 插入元素到n個element 的 2-3 Tree 時間複雜度 : O(log n)
    - 尋找插入點 : 最高degree = 2 => $2^h - 1$ <= n => O(lg n)
    - 向上split : 最扯split到Root => O(h)
- B Tree裡面k個Keys，那一定只有k+1個children
- RB Tree 
    - insert : 2次rotation
    - delete : 3次rotation
- B Tree 不是BST，因為Degree >= 2
- Root Balance Factor = 0 : 左右子樹一樣高
    - 插入一個節點那{**不一定**}會失衡
- RB Tree的四個Property
    - Root一定是Black
    - 所有NIL node都是black
    - Red node小孩不能是Red
    - 任一node到所有NIL路徑 BH 都相同
- RB Tree最大高度是 2*$log_2 (n+1)$
    ```
    prove : 
    一個RB-Tree的black height = h
    那 n >= $2^h - 1$
    h = 1 : 1
    h = 2 : 3
    h = 3 : 7

    所以 : n+1 >= $2^bh$ -> bh <= $2^bh - 1$

    然後最差 黑紅交錯 : h <= 2bh
    h <= 2*$log_2 (n+1)$
    ```
- RB Tree是平衡的，因為bh要一樣高，所以可以保證Longest path <= 2 * shortest path