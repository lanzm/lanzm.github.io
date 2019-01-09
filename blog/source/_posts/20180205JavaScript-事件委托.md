---
title: JavaScript-事件委托
date: 2018-02-05 14:42:20
tags: JavaScript
---
<!-- 标签别名 -->
{% cq %} 事件委托利用了事件冒泡，只指定一个事件处理程序，就可以管理一个类型的所有事件。<br/><br/><strong>—— JavaScript高级程序设计</strong> {% endcq %}
<!-- more -->
## 事件流 ##
 页面上的一个按钮点击事件，当你触发事件，即点击了这个按钮时，实际上并不是单单只点击了这个按钮元素，也点击了整个页面。由于这两种差异，产生了两种处理事件的方式。事件流是指从页面接收事件的顺序。
 
1. 事件冒泡（event bubbling）
 ```html
<!DOCTYPE html>
<html>
    <head></head>
    <body>
        <div id="myDiv">click me</div>
    </body>
</html>
 ```
  当点击div时，click事件会按照 `` div -> body -> html -> document`` 的顺序传播。
  如下图所示：
![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3w92gxhbj30bw06xdfz.jpg)
现代所有的浏览器都支持事件冒泡。IE5.5及更早版本的事件冒泡会跳过 ``html`` 元素（从 ``body`` 直接跳到 ``document``），IE9、Firefox、Chrome和Safari则将事件一直冒泡到 ``window`` 对象。
2. 事件捕获（event capturing）
和事件冒泡顺序完全相反，如果以前面的例子为例，click事件会按照`` document -> html -> body -> div`` 的顺序传播。
  如下图所示：
![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3w9czjr4j30cb05wt8t.jpg)
建议使用事件冒泡，有特殊需要时在使用事件捕获。

## 事件委托 ##
添加到页面上的事件处理程序数量会影响到页面整体的运行性能。而这种 “事件处理程序过多”的解决方案就是事件委托。
```html
<ul id="myLinks">
    <li id="goSomewhere">Go somewhere</li>
    <li id="doSomething">Do something</li>
    <li id="sayHi">Say hi</li>
</ul>
```
给三个 ``li`` 添加点击事件。如果是按照以往的做法，会分别根据li的id获取到元素，然后添加点击事件。如果采用事件委托，只需要在DOM树的尽量最高的层次上添加一个事件处理程序，如下：
```javascript
var list = document.getElementById("myLinks");
EvnetUtil.addHandler(list, "click", function(event){
    event = EventUtil.getEvent(event);
    var target = EventUtil.getTarget(event);

    switch(target.id){
        case "doSomething":
            document.title = "I changed the document's title";
            break;
        case "goSomewhere":
            document.title = "http:...";
            break;
        case "sayHi":
            alert("hi");
            break;
    }
});
```
## 采用事件委托的优点 ##
- 只要可单击的元素呈现在页面上，就可以立即具备适当的功能。
- 只要添加一个事件处理程序，所需要的DOM引用更少，所花的时间更少。
- 整个页面占用的内存空间更少，能够提升整体的性能。

最适合采用事件委托技术的事件有 ``click``、``mousedown``、``mouseup``、``keydown``、``keyup``和``keypress``。
