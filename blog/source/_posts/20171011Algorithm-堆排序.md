---
title: 算法导论-第六章 堆排序
date: 2017-10-11 11:08:09
tags: 算法导论
---
<!-- 标签别名 -->
{% cq %} 待到秋来九月八，我花开后百花杀。<br>冲天香阵透长安，满城尽带黄金甲。<br/><br/><strong>—— 黄巢《不第后赋菊》</strong> {% endcq %}
<!-- more -->
## 堆排序 ##
这里有一张表，统计各个算法所用的时间复杂度。

![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3w33f1u7j30mc067js5.jpg)

先理解以下几个概念。

![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3w3fqz11j30ma0dl40g.jpg)



数组 ``A[i]`` 可以根据 ``PARENT（i）`` 、 ``LEFT（i）`` 和 ``RIGHT（i）`` 算出父结点，左孩子和右孩子。

**节点高度(height of node)**：从在该节点下的最低的叶子向上，该节点所在的层数。
**节点深度(depth of node)**：从根节点向下，经过的层数。
如： 
节点2 高度为2，深度为1；
节点3 高度为1，深度为1。

二叉堆分为**最大堆**和**最小堆**。
**最大堆**：除了根以外的所有结点 ``i`` 都要满足 ``A[PARENT(i)] ≥ A[i]``
（上图的二叉树就是最大堆）
**最小堆**：除了根以外的所有结点 ``i`` 都要满足 ``A[PARENT(i)] ≤ A[i]``

``MAX-HEAPIFY`` 用于维护某个结点的最大堆。

![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3w3tpif4j30bt074dga.jpg)

![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3w43wciij30md0du418.jpg)

对于一个高度为h的结点来说，``MAX-HEAPIFY`` 的时间复杂度为O（h）。

``BUILD-MAX-HEAP(A)`` 将数组 ``A[i]`` 排列成 最大堆。
![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3w4edy5oj30ab030wei.jpg)

![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3w4pmv5dj30d80ditat.jpg)
下面贴上根据伪代码写的Java实现。
```java
    public void MAXHEAPIFY(int [] A, int i){

        int length = A.length - 1;
        int largest;
        int LEFT = 2 * i + 1;
        int RIGHT = 2 * i + 2;

        if (LEFT <= length && A[i] <= A[LEFT]){
            largest = LEFT;
        }else{
            largest = i;
        }

        if (RIGHT <= length && A[largest] <= A[RIGHT]) {
            largest = RIGHT;
        }

        if (largest != i){
            int exchange = A[i];
            A[i] = A[largest];
            A[largest] = exchange;

            MAXHEAPIFY(A, largest);
        }
    }

    public void BUILDMAXHEAP(int [] A){

        int length = A.length;
        for (int i = (length - 1)/2; i >= 0; i --){

            MAXHEAPIFY(A, i);

        }

    }
```
<font color="red">注意</font>：在Java里面，数组的起始下标是从0开始，因此 ``LEFT`` 和 ``RIGHT`` 要多加一个1。

我们发现数组的第一个数也就是根的值总是最大的，将最大值放到数组的最后，并把最后之前的数再重新排列成最大堆，如此重复即可将数组有序排序。

![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3w52xiybj30bb0440sw.jpg)

![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3w5bqwfkj30lr09ydi2.jpg)

在伪代码中，``HEAPSORT`` 方法每次遍历都会将数组的长度减一，以保证每次第一位和最后一位数字交换以后，最后一位不再进行 ``MAX-HEAPIFY``。
 ``MAX-HEAPIFY`` 加一个参数 ``heapSize``，每次循环减掉后的数组长度 ``heapSize`` 传到 ``MAX-HEAPIFY`` 方法中，在判断越界时  ``LEFT(RIGHT) > heapSize`` 不进行交换，以确保后几位排序好的数不会受到影响。下面贴上Java代码。
```java
    public void MAXHEAPIFY(int [] A, int heapSize, int i){

        int largest;
        int LEFT = 2 * i + 1;
        int RIGHT = 2 * i + 2;

        if (LEFT <= heapSize && A[i] <= A[LEFT]){
            largest = LEFT;
        }else{
            largest = i;
        }

        if (RIGHT <= heapSize && A[largest] <= A[RIGHT]) {
            largest = RIGHT;
        }

        if (largest != i){
            int exchange = A[i];
            A[i] = A[largest];
            A[largest] = exchange;

            MAXHEAPIFY(A, heapSize, largest);
        }
    }

    public void BUILDMAXHEAP(int [] A){

        int length = A.length -1;
        for (int i = (length)/2; i >= 0; i --){

            MAXHEAPIFY(A, length, i);

        }
    }

    public void HEAPSORT(int [] A){

        int length = A.length;
        BUILDMAXHEAP(A);
        for (int i = length - 1; i >= 1; i -- ){

            int exchange = A[0];
            A[0] = A[i];
            A[i] = exchange;

            int heapSize = i - 1;
            MAXHEAPIFY(A, heapSize, 0);

        }
    }
```
堆常见的应用在**优先队列**。
优先队列也有两种形式：**最大优先队列**和**最小优先队列**。
一个**最大优先队列**支持以下操作：


- ``INSERT(S,x)``：把元素x插入到集合S



- ``MAXIMUM(S)``：返回S中具有最大关键字的元素



- ``EXTRACT_MAX(S)``：去掉并返回S中的具有最大关键字的元素



- ``INCREASE_KEY(S,x,k)``：将元素x的关键字的值增加到k，这里k值不能小于x的原关键字的值。

``INCREASE_KEY(S,x,k)`` 伪代码如下：
```
1 HEAP_INCREASE_KEY(A,i,key)
2    if key < A[i]
3      then error
4    A[i] = key
5    while i > 1 && A[PARENT(i)] < A[i]
6        do exchange A[i] <-> A[PARENT(i)]
7        i = PARENT(i)
```
![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3w5pqfhnj30m20ditbo.jpg)



习题6.5-6：在 ``Heap_Increase_Key`` 的第5行操作中，一般需要通过三次赋值来完成。想一想如何利用 ``Insertion_Sort`` 内循环部分的思想，只用一次赋值就完成这一交换操作？

插入排序的思想是在插入之前先和前面数字对比，不符合的数字就往后移动，找到合适的位置再插入。而 ``Heap_Increase_Key`` 中只要不停地操作对比两个数， ``PARENT`` 和 ``key`` ，插入的 ``key`` 比 ``PARENT`` 大，两者互相交换，并继续对比爸爸的爸爸...直到根。

```
1 Heap-Increase-Key(A, i, key)
2     if A[i] < key
3         error "new key is smaller than original"
4     while i > 1 and A[Parent(i)] < key
5         A[i] = A[Parent(i)]
6         i = Parent(i)
7     A[i] = key
```

习题6.5-9 见[链接](http://www.cnblogs.com/Anker/archive/2013/01/24/2874569.html)。