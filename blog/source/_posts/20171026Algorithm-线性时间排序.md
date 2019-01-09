---
title: 算法导论-第八章 线性时间排序
date: 2017-10-26 09:29:02
tags: 算法导论
---
<!-- 标签别名 -->
{% cq %} 林花谢了春红，太匆匆。无奈朝来寒雨晚来风。<br>胭脂泪，相留醉，几时重。自是人生长恨水长东。<br/><br/><strong>—— 李煜《相见欢·林花谢了春红》</strong> {% endcq %}
<!-- more -->
**比较排序**，各个元素的次序依赖于它们之间的比较。任何比较排序在最坏情况下都要经过Ω(n lgn)次比较。

## 计数排序 ##
计数排序的思想是：在待排序序列中，如果我们能统计出有多少元素小于或等于某一个元素，我们也就知道了该元素的正确位置。例如，对于待排序序列 ``{2,5,3,0,2,3,0,3}``，我们统计出有8个元素小于等于5（包括5自己），那么5这个元素就应该被排序到第8位。
```
COUNTING-SORT(A, B, k)
1 let C[0..k] be a new array
2 for i = 0 to k
3      C[i] = 0
4 for j = 1 to A.length
5      C[A[j]] = C[A[j]] + 1
6 // C[i] now contains the number of elements equal to i.
7 for i = 1 to k
8      C[i] = C[i] + C[i-1]
9 // C[i] now contains the number fo elements less than or equal to i.
10 for j = A.length downto 1
11     B[C[A[j]]] = A[j]
12     C[A[j]] = C[A[j]] - 1

```

下图是伪代码实现图解。

![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3w7q8fvrj30le0a2dip.jpg)

Java代码实现。
```java

   private int [] COUNT_SORT(int [] A, int k){

        int [] B = new int[A.length];
        int [] C = new int[k];
        for (int i = 0; i < A.length; i ++){
            C[A[i]] += 1;
        }
        for (int j = 1; j < k; j ++){
            C[j] = C[j] + C[j - 1];
        }
        for (int g = A.length - 1; g > -1; g --){
            B[C[A[g]] - 1] = A[g];  // 注意这里的B下标要减去一，因为是从0开始
            C[A[g]] = C[A[g]] - 1;
        }
        return B;
    }

```
在伪代码中，第2~3行时间代价θ(k)；第4~5行时间为θ(n)；第7~8行时间为θ(k)，第10~12行时间为θ(n)。因此，总的运行时间是θ(k+n)。当k= O(n)时，运行时间为θ(n)。

## 基数排序 ##
书中描述有点抽象，这里有一篇写的很形象的[文章](http://blog.csdn.net/tangbo713/article/details/41204313)。
基数排序其实是将 个十百 上的数依次进行排列，文中的Java代码在排序上选择计数排序。

```java

public static void main(String[] args) {  
        int[] array = {9,8,10,56,333,45566,2345678,78,990,12,432,56};    
        print(radixSort(array,10,7));    
    }  
    /* 
     *array为待排序数组,radix代表基数,distance代表排序元素的位数 
     */  
    private static int [] radixSort(int[] array,int radix, int distance) //基于计数排序的基数排序算法  
    {  
          
        int length = array.length;  
        int[] temp = new int[length];//用于暂存元素  
        int[] count = new int[radix];//用于计数排序  
        int divide = 1;  
  
        for (int i = 0; i < distance; i++) {  
  
            System.arraycopy(array, 0,temp, 0, length);  
            Arrays.fill(count, 0);  
  
            for (int j = 0; j < length; j++) {  
                int tempKey = (temp[j]/divide)%radix;  
                count[tempKey]++;  
            }  
  
            for (int j = 1; j < radix; j++) {  
                count [j] = count[j] + count[j-1];  
            }  
  
            //个人觉的运用计数排序实现计数排序的重点在下面这个方法              
            for (int j = length - 1; j >= 0; j--) {  
                int tempKey = (temp[j]/divide)%radix;  
                count[tempKey]--;  
                array[count[tempKey]] = temp[j];  
            }  
  
            divide = divide * radix;                  
  
        }  
        return array;  
    }  
    static void print(int []array)//打印函数  
    {  
        for(int i=0;i<array.length;i++)  
        {  
            System.out.print(array[i]+" ");  
        }  
        System.out.println();  
    }  

```
## 桶排序 ##
桶排序的思想是把区间[0,1)划分成n个相同大小的子区间，称为桶，然后将n个输入数分布到各个桶中去，因为输入数均匀且独立分布在[0,1)上，所以，一般不会有很多数落在一个桶中的情况，为了得到结果，先对各个桶中的数进行排序，然后按次序把各桶中的元素列出来。就是按照数组中元素的最高位分别放置到对应桶中，在各个桶中排序后再合并成排序好的数组。
桶排序伪代码：
```
BUCKET-SORT(A)
1 n = A.length
2 let B[0..n - 1] be a new array
3 for i = 0 to n - 1
4      make B[i] an empty list
5 for i = 1 to n 
6      insert A[i] into list B[nA[i]]
7 for i = 0 to n - 1
8      sort list B[i] with insertion sort
9 concatenate the lists B[0],B[1],...B[n - 1] together in order 
```
![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3w85vkapj309n0cs403.jpg)