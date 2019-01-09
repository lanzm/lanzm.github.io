
---
title: JavaScript-惰性载入函数
date: 2018-03-08 10:37:09
tags: JavaScript
---
<!-- 标签别名 -->
{% cq %} 曾伴浮云归晚翠，犹陪落日泛秋声。世间无限丹青手，一片伤心画不成。<br/><br/><strong>—— 高蟾《金陵晚望》</strong> {% endcq %}
<!-- more -->
## 惰性载入 ##
```javascript
function createXHR() {
    if (typeof XMLHttpRequest() != "undefined") {
        return new XMLHttpRequest();
    } else if (typeof ActionXObject != "undefined") {
        if (typeof arguments.callee.activeXString != "string") {
            var version = ["MSXML2.XMLHttp.6.0", "MSXML2.XMLHttp.3.0", "MSXML2.XMLHttp"], i, len;
            for (i = 0, len = version.length; i < len; i++) {
                try {
                    new ActiveXObject(versions[i]);
                    arguments.callee.activeXString = versions[i];
                    break;
                } catch (ex){
                    // 
                }
            }
        }
        return new AcctiveXObject(arguments.callee.activeXString);
    } else {
        throw new Error("No XHR object available.");
    }
}
```
``createXHR()`` 函数是对浏览器所支持的能力检查，先检查内置的XHR，接着测试有没有基于ActiveX的XHR。类似于这种的能力检查，实际只要执行一次即可，就像如果检查出了用户所用的浏览器是支持内置XHR，则说明这个浏览器就是支持，没有必要再用 ``if`` 去判断。
下面有两个方式去实现惰性载入。
## 惰性载入方式一 ##
在函数被调用时再处理函数。
```javascript
function createXHR() {
    if (typeof XMLHttpRequest() != "undefined") {
        creatXHR = function() {
            return new XMLHttpRequest();		
        };
    } else if (typeof ActionXObject != "undefined") {
        creatXHR = function() {
            if (typeof arguments.callee.activeXString != "string") {
                var version = ["MSXML2.XMLHttp.6.0", "MSXML2.XMLHttp.3.0", "MSXML2.XMLHttp"], i, len;
                for (i = 0, len = version.length; i < len; i++) {
                    try {
                        new ActiveXObject(versions[i]);
                        arguments.callee.activeXString = versions[i];
                        break;
                    } catch (ex){
                        // 
                    }
                }
            }
            return new AcctiveXObject(arguments.callee.activeXString);
        };
    } else {
        creatXHR = function() {
            throw new Error("No XHR object available.");
        };
    }
    return creatXHR();
}
```
第一次执行时，函数会给 ``creatXHR`` 变量赋值，当第二次执行时，函数会直接执行 ``creatXHR`` 所分配的函数。
## 惰性载入方式二 ##
在声明函数时，就指定适当的函数。
```javascript
var createXHR = (function() {
    if (typeof XMLHttpRequest() != "undefined") {
        return function() {
            return new XMLHttpRequest();		
        };
    } else if (typeof ActionXObject != "undefined") {
        return function() {
            if (typeof arguments.callee.activeXString != "string") {
                var version = ["MSXML2.XMLHttp.6.0", "MSXML2.XMLHttp.3.0", "MSXML2.XMLHttp"], i, len;
                for (i = 0, len = version.length; i < len; i++) {
                    try {
                        new ActiveXObject(versions[i]);
                        arguments.callee.activeXString = versions[i];
                        break;
                    } catch (ex){
                        // 
                    }
                }
            }
            return new AcctiveXObject(arguments.callee.activeXString);
        };
    } else {
        return function() {
            throw new Error("No XHR object available.");
        };
    }
})();
```