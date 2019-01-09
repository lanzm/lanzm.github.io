---
title: 算法导论-第四章 Strassen
date: 2017-09-14 17:07:04
tags: 算法导论
---
<!-- 标签别名 -->
{% cq %} 谁念西风独自凉，萧萧黄叶闭疏窗，沉思往事立残阳。<br>被酒莫惊春睡重，赌书消得泼茶香，当时只道是寻常。<br/><br/><strong>—— 纳兰性德《浣溪沙·谁念西风独自凉》</strong> {% endcq %}
<!-- more -->
# 矩阵乘法的Strassen算法 #
## 矩阵乘法定义 ##

![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3vpby82ij30mv0bt3yu.jpg)

## 矩阵乘法伪代码 ##

![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3vpmfknpj30lb078aay.jpg)

三重的for循环都执行到n步，第7行每次执行都花费常量时间，因此SQUARE-MATRIX-MULTIPLY花费Θ(n^3)时间。

## 递归分治算法解矩阵乘法 ##

![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3vpz88p2j30il05nt99.jpg)

主要的思路是 矩阵相乘分别是将A矩阵的第一行和B矩阵的第一列对应相乘，得出C矩阵，类似的步骤重复n次（如果AB矩阵是 n X n），递归分治的思想就是将这些步骤、小问题抽取出来，单独处理后再合并成最终的结果。

![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3vqovzs1j30hq096gn7.jpg)

书中提到这段伪代码忽略了一个重要的细节。就是第五行，将每个矩阵分解成为子矩阵。其实是指如何得到矩阵中的元素值，如果真的要把值一个个取出，需要遍历i和j下标，就是两次的for循环，花费Θ(n^2)的时间去得到元素，但是其实只需要利用下标就可以取到元素，花费Θ(1)的时间。所以运行时间的递归式是：

![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3vqzbqm2j30b401u0sn.jpg)

书中对递归式给出了很详细的解释。

![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3vrago9bj30kt06iwh2.jpg)

![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3vrgxeq7j30kt06iwh2.jpg)

## Strassen算法解矩阵乘法 ##

书中说Strassen算法是让递归树不那么茂盛一点，即只进行递归7次，而不是8次。

步骤：
1. 将ABC矩阵分解 n/2 X n/2 的子矩阵，运行时间为Θ(1)。
2. 创建10个子矩阵
![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3vrwp19jj306w0c3q35.jpg)

运行时间为Θ(n^2)。

3.递归地计算7个矩阵。

![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3vsjss21j30he054gm9.jpg)

运行时间为7T(n/2)。

4.最后将第三步的7个矩阵进行加减运算。

![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3vstlhf3j30ko05xjs7.jpg)

![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3vt11b45j30bu08wt90.jpg)

![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3vt8k2kcj30ho05k0t2.jpg)

共进行了8次的加减运算，花费时间为Θ(n^2)。（这个不太懂）
因此Strassen算法递归式应该为：

![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3vtjb7bbj30dm01hmx2.jpg)

