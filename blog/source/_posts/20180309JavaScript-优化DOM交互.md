---
title: JavaScript-优化DOM交互
date: 2018-03-09 16:23:55
tags: JavaScript
---
<!-- 标签别名 -->
{% cq %} 嗟余只影系人间，如何同生不同死？<br/><br/><strong>—— 陈衡恪《题春绮遗像》</strong> {% endcq %}
<!-- more -->
## 最小化现场更新 ##
下面一段代码是比较常见的拼接网页元素的遍历方法。
```javascript
var list = document.getElementById("myList"),
               item,
               i;
for (i = 0; i < 10; i ++) {
    item = document.createElement("li");
    list.appendChild(item);
    item.appendChild(document,creatTextNode("Item " + i));
}
```
可以使用文档碎片来构建DOM结构来优化性能，并且避免了现场更新和页面闪烁的问题。
如下：
```javascript
var list = document.getElementById("myList"),
    fragment = document.createDocumentFragment(),
    item,
    i;
for (i = 0; i < 10; i ++) {
    item = document.createElement("li");
    fragment.appendChild(item);
    item.appendChild(document,creatTextNode("Item " + i));
} 
list.appendChild(fragment);
```
文档碎片作为一个临时的占位符，放置新创建的项目。当给 ``appendChild()`` 传入文档碎片时，只有碎片的子节点会被添加到目标，碎片本身不会被添加。
## 使用 innerHTML ##
创建DOM节点的方法 createELement() 和 appendChild() 以及 innerHTML ，对于小的DOM更改，效率差不多，但是对于大的DOM更改，使用 innerHTML 要比其它方法创建同样的DOM结构快很多。
上例可改写如下：
```javascript
var list = document.getElementById("myList"),
    html = "",
    i;
for (i = 0; i < 10; i ++) {
    html += "<li>Item " + i + "</li>";
}
list.innerHTML = html;
```
但是要注意避免下面写法：
```javascript
var list = document.getElementById("myList"),
    i;
for (i = 0; i < 10; i ++) {
    list.innerHTML += "<li>Item " + i + "</li>";   // 避免
}
```