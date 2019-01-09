---
title: 算法导论-第四章 Maximum subarray problem
date: 2017-09-12 14:23:00
tags: 算法导论
---
<!-- 标签别名 -->
{% cq %} 花褪残红青杏小。小燕子飞时，绿水人家绕。枝上柳绵吹又少。天涯何处无芳草！<br>墙里秋千墙外道。墙外行人，墙里佳人笑。笑渐不闻声渐悄。多情却被无情恼。<br/><br/><strong>—— 李煜《相见欢·林花谢了春红》</strong> {% endcq %}
<!-- more -->
## 最大子数组 ##
寻找数组中的和最大的非空连续子数组，这样的连续最大子数组，这样的连续子数组称为 最大子数组。
![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3vnklfmbj30gv043q3m.jpg)
## 暴力求解最大子数组问题 ##
```java
 public int maximumSubarray(int [] a){

        int max = 0;
        int low = 0;
        int high = 0;
        for (int i = 0; i < a.length; i ++ ){

            int sum = 0;
            for (int j = i; j < a.length; j ++){

                sum = sum + a[j];
                if (sum > max){
                    
                    max = sum;
                    low = i;
                    high = j;
                    
                }
            }
        }
        return max;
    }
```
每次内循环都利用上次累加的结果，避免重复运算。外层循环执行n次，第i次外循环内层循环执行n-i+1次，所以总的时间复杂度为Θ(n^2)
## 用分治策略求解最大子数组问题 ##
在分治策略中，递归求解一个问题需要三个步骤：


1. 分解（Divide） 将问题划分为一些子问题，子问题的形式和原问题一样，只是规模更小。
2. 解决（Conquer）递归地求解子问题，如果规模足够小，则停止递归，直接求解。
3. 合并（Combine）将子问题的解组合成原问题的解。

![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3vnw51j8j30kb063q4d.jpg)

找到子数组A[low...high]的中间位置，把子数组拆成两个子数组A[low...mid]和A[mid+1...high]，而A[low...high]任何连续的子数组A[i...j]所处的位置必然是下面三种情况：

- 完全在A[low...mid]中
- 完全在A[mid+1...high]中
- 跨越了中点mid

![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3vodaxhmj30bk0esaas.jpg)

上图为实现子数组A[i...j]所处的位置跨越了中点mid的情况伪代码，其时间复杂度为Θ(n)。
