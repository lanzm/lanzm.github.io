---
title: 算法导论-第七章 快速排序
date: 2017-10-24 10:09:04
tags: 算法导论
---
<!-- 标签别名 -->
{% cq %} 时光只解催人老，不信多情，长恨离亭，泪滴春衫酒易醒。<br>梧桐昨夜西风急，淡月胧明，好梦频惊，何处高楼雁一声？<br/><br/><strong>—— 晏殊《采桑子·时光只解催人老》</strong> {% endcq %}
<!-- more -->
## 快速排序 ##
快速排序是实际排序应用中最好的选择。和归并排序一样，也用到了分治思想。
下面是实现快速排序的伪代码。
```
QUICKSORT(A, p, r)
1 if p < r
2     q = PARTITION(A, p, r)
3     QUICKSORT(A, p, q-1)
4     QUICKSORT(A, q+1, r)
```
其中 ``PARTITION`` 是整个算法的关键所在。它实现了对数组的原址排序。下面是 ``PARTITION`` 的伪代码。
```
PARTITION(A, p, r)
1 x = A[r]
2 i = p-1
3 for j = p to r-1
4      if A[j] ≤ x
5           i = i+1
6      exchange A[i] with A[j]
7 echange A[i+1] with A[r]
8 return i+1

```
下图显示了 ``PARTITION`` 流程。

![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3w744cwqj30k008bt9u.jpg)

取数组A的最后一个数，从数组的第一个数开始相对比，如果该值小于最后一个数，则 ``i`` 的计位加一，并且和计位 ``i`` 对应的数相交换，最后会发现，小于最后一个数的值都在计位数 ``i`` 的前面，而大于最后一个数的值都在后面，除了 最后一个数，再把 最后一个数和 ``i`` 加一交换，就完成了 ``PARTITION`` 的流程。
下面给出Java代码实现。
```java

    private int [] quickSort(int [] A, int p, int r){
        if (p < r){
            int q = partition(A, p, r);
            quickSort(A, p, q - 1);
            quickSort(A, q + 1, r);
        }
        return A;
    }

    private int partition(int [] A, int p, int r){
        int x = A[r];
        int i = p -1;
        for (int j = p; j < r; j ++){
            if (A[j] >= x){
                i += 1;
                exchang(A, i, j);
            }
        }
        exchang(A, i + 1, r);
        return i + 1;
    }

```
下面贴出快速排序的随机化版本。
```java

    private int [] RANDOMIZED_QUICKSORT(int [] A, int p, int r){
        if (p < r){
           int q = RANDOMIZED_PARTITION(A, p, r);
           RANDOMIZED_QUICKSORT(A, p, q - 1);
           RANDOMIZED_QUICKSORT(A, q + 1, r);
        }
        return A;
    }

    private int RANDOMIZED_PARTITION(int [] A, int p, int r){
        int i = (int) (Math.random() * ((r - p) + 1)) + p;
        exchange(A, i, r);
        return partition(A, p, r);
    }

    private int partition(int [] A, int p, int r){
        int x = A[r];
        int i = p -1;
        for (int j = p; j < r; j ++){
            if (A[j] <= x){
                i += 1;
                exchange(A, i, j);
            }
        }
        exchange(A, i + 1, r);
        return i + 1;
    }

```