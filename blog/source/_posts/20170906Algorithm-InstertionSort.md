---
title: 算法导论-第二章 INSERTION-SORT
date: 2017-09-06 15:10:40
tags: 算法导论
---
<!-- 标签别名 -->
{% cq %} 若教眼底无离恨，不信人间有白头。<br/><br/><strong>—— 辛弃疾《鹧鸪天·代人赋》</strong> {% endcq %}
<!-- more -->
## 插入排序 ##

对于元素比较少的排序来说，插入排序是一个有效的算法。

## 描述 ##


书中用一个很形象的例子，排序扑克牌来形容插入算法。开场只有两只空手和一堆乱序的扑克牌，一只手拿排好的牌，另一只手拿要插入的牌，从右到左比较牌的大小选择正确的位置插入排好的牌。
![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3vlqaas5j309f07vq3f.jpg)
## Java代码实现 ##
数组a可以看作没有排序的扑克牌，从左开始依次进一张牌，并对左边已经抓的牌比较选择正确的位置放置。（书中的顺序是从右到左）
```java
public class insert {
 public void insTest(int [] a){

        int length = a.length;
        for (int i = 0; i < length - 1; i ++){

            int key = a[i + 1];
            while (i >= 0 && key < a[i]){
                a[i + 1] = a[i];
                a[i] = key;
                i = i - 1;
            }

        }

        for (int j = 0; j < length; j ++){
            System.out.print(a[j] + ",");
        }
    }

    public static void main(String [] args){
        int [] a = {3,67,4,6,90,12,4,55};
        insert insert = new insert();
        insert.insTest(a);

    }

}
```
通过循环不变式的三条性质可以验证上述的插入排序代码是否正确。


- 初始化：循环的第一次迭代之前，它为真。


- 保持：如果循环的某次迭代之前它为真，那么下次迭代之前它仍为真。


- 终止：在循环终止时，不变式为我们提供一个有用的性质，该性质有助于证明算法时正确的。

但是在认真看过书中所写的伪代码后，修改上面的insTest方法。
```java
   public void insTest(int [] a){

        int length = a.length;
        for (int i = 0; i < length - 1; i ++){

            int key = a[i + 1];
            int j = i;
            while (j >= 0 && key < a[j]){
                a[j + 1] = a[j];
                j = j - 1;
            }
            a[j + 1] = key;

         }

        for (int j = 0; j < length; j ++){
            System.out.print(a[j] + ",");
        }
    }
```
很明显可以看出这关系到时间复杂度的问题。