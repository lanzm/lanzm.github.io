---
title: 算法导论-第五章 随机算法
date: 2017-10-09 11:32:41
tags: 算法导论
---
<!-- 标签别名 -->
{% cq %} 韶华不为少年留，恨悠悠，几时休？<br>飞絮落花时候，一登楼。便作春江都是泪，流不尽，许多愁。<br/><br/><strong>—— 秦观《江城子·西城杨柳弄春柔》</strong> {% endcq %}
<!-- more -->
## 随机算法 ##
书中介绍两种随机算法

第一种算法是为数组的每个元素A[i]赋一个随机的优先级P[i]，然后依据优先级对数组A中的元素进行排序。例如，如果初始数组A=(1，2，3，4)，随机选择的优先级P=(36，3，62，19)，则将产生一个数组B=(2，4，1，3)，因为第2个优先级最小，接下来是第4个，然后第1个，最后第3个。我们称这个过程为PERMUTE-BY-SORTING:
```
PERMUTE-BY-SORTING(A)
  n = A.length
  let P[1..n] be a new array
  for i = 1 to n
    P[i] = RANDOM(1,n^3)
  sort A, using P as sort keys

```
P的值选取一个在1~n^3之间的随机数。我们使用范围1~n^3是为了让P中所有优先级尽可能唯一。
主要耗时的地方是 sort A 的过程，第八章将会解决相同的问题。

第二种算法是原址排列给定数组。过程RANDOMIZE-IN-PLACE在O(n)时间内完成。在进行第i次迭代时，元素A[i]是从元素A[i]到A[n]中随机选取的。第i次迭代以后，A[i]不再改变。我们假设所有的优先级都唯一。
```
RANDOMIZE-IN-PLACE(A)
  n = A.length
  for i = 1 to n
    swap A[i] with A[RANDOM(i,n)]
```
明显这个方法更加简洁。
<font color="red">注意：</font>RANDOM中的两个参数是 i 和 n ，至于为什么参数不是 1 到 n，习题5.3-3有解释。
下面贴上我写的Java实现：
```java
    public int[] Random(int [] a) {

        int n = a.length;
        for (int i = 0; i < n; i++) {
            swap(i, a);
        }
        return a;

    }
    private void swap(int i, int[] a) {
        int length = a.length;
        // RANDOM(i, length)
        int aRandom = (int) (Math.random()*(length - i) + i);
        int b = a[aRandom];
        a[aRandom] = a[i];
        a[i] = b;
    }
```
课后习题参考[这篇博文](http://blog.sina.com.cn/s/blog_879fcdf60102vu42.html)。