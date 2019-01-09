---
title: JavaScript-定时器
date: 2018-03-08 15:02:31
tags: JavaScript
---
<!-- 标签别名 -->
{% cq %} 劝君莫惜金缕衣，劝君惜取少年时。花开堪折直须折，莫待无花空摘枝。<br/><br/><strong>—— 杜秋娘《金缕衣》</strong> {% endcq %}
<!-- more -->
## setTimeout ##
JavaScript运行在单线程的环境内，定时器是计划代码在未来的某个时间执行。在页面加载完后代码的运行、事件处理程序以及Ajax回调函数等浏览器负责排序，指派某段代码运行的时间点。
![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3wahy70lj30m904c3yo.jpg)
当进程处在空闲状态，用户触发点击事件，点击事件的代码就会被添加到队列中执行，如果不处于空闲状态，代码会等待到进程处于空闲状态时再进行。
定时器对队列的工作方式是：当定时器的设置的等待时间过去后将代码插入。而将代码插入后并不代表定时器会马上执行，定时器会等待进程处于空闲状态时执行。
下面一个例子：
```javascript
var btn = document.getElementById("my-btn");
btn.onclick = function() {
    setTimeout(function() {
        document.getElementById("message").style.visibility = "visible";
    }, 250);
    // 其他代码
};
```
代码给id为 ``my-btn`` 的元素添加了一个点击事件，点击事件设置了一个	延迟250ms后执行的定时器。他的执行图如下：
![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3watzkcyj30nq072t95.jpg)
只有在click事件程序结束后才能执行定时器程序。
## setInterval ##
和 setTimeout 不同的是 setInterval 会重复执行，而 setTimeout 只会执行一次。但是 setInterval 有两个问题：1、某些间隔会被跳过。2、多个定时器的代码执行之间的间隔可能会比预期的小。
![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3wb5wpucj30n40c80v0.jpg)
为了避免 setInterval 的问题，可以使用链式 setTimeout() 调用替代重复定时器。
下面是链式 setTimeout() 基本形式：
```javascript
setTimeout(function() {
    // 处理中
    setTimeout(arguments.callee, interval);
}, interval);
```
每次执行 setTimeout 时都是新建一个定时器，而 ``arguments.callee`` 用来保证在前一个定时器没执行完之前，不会向队列插入新的定时器。
```javascript
setTimeout(function() {
    var div = document.getElementById("myDiv");
    left = parseInt(div.style.left) + 5;
    div.style.left = left + "px";
    if (left < 200) {
        setTimeout(arguments.callee, 50);
    }
}, 50);
```
代码每次执行时，将id为 ``myDiv`` 元素向右移动，直到左坐标为 200px 时停止。
## Yielding Processes ##
Ajax从后台获取数据后遍历取出数据是一个很常见的动作。
```javascript
for ( var i = 0, len = data.length; i < len; i ++) {
    process(data[i]);
}
```
如果数据量大，会造成脚本长时间运行而导致超出脚本限制运行时间，浏览器会弹出窗口提示用户是否要继续执行还是停止它。
为了避免这种情况，可以用定时器来分割这个循环，叫做数组分块（array chunking）。前提是对循环数据不用同步完成以及不关心数据顺序。
基本模式如下：
```javascript
setTimeout(function() {
    // 取出下一个条目 并处理     
    var item = array.shift();
    process(item);
    // 若还有条目， 再设置另一个定时器
    if (array.length > 0) {
        setTimeout(arguments.callee, 100);
    }
}, 100);
```
一旦某个函数需要花 50ms 以上的时间完成，最好看是否能将任务分割成定时器小任务。
## 函数节流 ##
连续进行过多的DOM相关操作可能会导致浏览器挂起，甚至崩溃。尤其是在IE中使用 onresize 事件处理程序容易发生，当调整浏览器大小的时候，该事件会连续触发。为了解决问题，可以使用函数节流。
函数节流的做法是 在调用某个事件处理程序时，中间插入一个定时器，防止某些代码没有间断连续重复执行。
下面是函数节流的基本形式：
```javascript
var processor = {
    timeoutId: null,
    // 实际进行处理的方法
    performProcessing: function() {
        //实际执行的代码
    },
    // 初始处理调用的方法
    process: function() {
        clearTimeout(this.timeoutId);
        var that = this;
        this.timeoutId = setTimeout(function() {
            that.performProcessing();
        }, 100);
    }
};
//尝试开始执行
processor.process();
```
这个模式可以用 ``throttle()`` 函数简化：
```javascript
/**
* @params method 要执行的函数
* @params context 在哪个作用域中执行，不填则表示全局
*/
function throttle(method, context) {
    clearTimeout(method.tId);
    method.tId = setTimeout(function() {
        method.call(context);
    }, 100);
}
```
使用 call() 来确保方法在适当的环境中执行。节流在 resize 事件中最常用。下面是应用的例子：
```javascript
// 没有用 节流 时
window.onresize = function() {
    var div = document.getElementById("myDiv");
    div.style.height = div.offsetWidth + "px";
};
// 使用 节流 后
function resizeDiv() {
    var div = document.getElementById("myDiv");
    div.style.height = div.offsetWidth + "px";
}
window.onresize = function() {
    throttle(resizeDiv);
};
```