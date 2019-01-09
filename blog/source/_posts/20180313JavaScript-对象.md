---
title: JavaScript-对象
date: 2018-03-13 09:37:30
tags: JavaScript
---
<!-- 标签别名 -->
{% cq %} 空山新雨后，天气晚来秋。<br/><br/><strong>—— 王维《山居秋暝》</strong> {% endcq %}
<!-- more -->
## 构造函数模式 ##
工厂模式创建一个简单的对象：
```javascript
function createPerson(name, age, job) {
    var o = new Object();
    o.name = name;
    o.age = age;
    o.job = job;
    o.sayName = function() {
        alert(this.name);
    };
    return o;
}
var person1 = createPerson("Nicholas", 29, "Software Engineer");
var person2 = createPerson("Greg", 27, "Doctor");
```
也可以采用 构造函数 的方法来创建对象
```javascript
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = function() {
        alert(this.name);
    };
}
var person1 = new Person("Nicholas", 29, "Software Engineer");
var person2 = new Person("Greg", 27, "Doctor");
```
要创建 Person 的新实例，必须用 new 操作符。以这种方式调用构造器会经历以下四个步骤：
1. 创建一个新对象；
2. 将构造函数的作用域赋给新对象（this指向新对象）；
3. 执行构造函数中的代码（给新对象添加属性）；
4. 返回新对象。

### 将构造函数当作函数 #####
```javascript
var person = new Person("Nicholas", 29, "Software Engineer");
person.sayName(); // "Nicholas"

// 作为普通函数调用
Person("Greg", 27, "Doctor");  // 添加到 window
window.sayName();  // "Greg"

// 在另一个对象的作用域中调用
var o = new Object();
Person.call(o, "Kristen", 25, "Nurse");
o.sayName();  // "Kristen"
```
例子展示了构造函数的三种调用方法。第一个是用 new 来创建新对象。第二个是将属性和方法都添加给 window 对象。 最后一个是使用 ``call()`` （或者 ``apply()`` ）在对象o的作用域中调用 Person() 函数。

### 构造函数的问题 #####
每新建一个实例时，构造函数中的方法就会重新创建一遍，如上例 Person 的 ``sayName`` 方法。这个问题可以用 原型模式 来解决。
## 原型模式 ##
每个函数都有 prototype（原型）属性，这个属性是指针，指向一个对象。可以不必在构造函数中定义对象实例的信息，可以添加到原型对象中。
如下：
```javascript
function Person() {
}

Person.prototype.name = "Nicholas";
Person.prototype.age = 29;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function() {
    alert(this.name);
};

var person1 = new Person();
person1.sayName();  // "Nicholas"

var person2 = new Person();
person2.sayName();  // "Nicholas"
alert(person1.sayName == person2.sayName);  // true
```
从最后一行代码可以看出新对象的这些属性和方法是由所有实例共享的。即 person1 和 person2 访问的都是同一组属性和 sayName() 方法。这样就解决了上述构造器的问题。

### 理解原型对象 #####
无论什么时候，只要创建了一个新函数，就会根据一组特定的规则为该函数创建一个 prototype 属性，这个属性指向函数的原型对象。默认情况下，原型对象会自动获得一个 constructor （构造函数）属性，这个属性包含一个指向 prototype 属性所在函数的指针。如下：
![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3wc0o227j30p707gt94.jpg)
图中 Person.prototype 指向原型对象，而 Person.prototype.constructor又指回了 Person。Person 的每个实例 person1 和 person2 都包含一个内部属性，该属性只指向了 Person.prototype，它们和构造函数都没有直接关系。
``isPrototypeOf()`` 方法可以确定对象之间是否存在关系。
```javascript
alert(Person.prototype.isPrototypeOf(person1));   // true
alert(Person.prototype.isPrototypeOf(person2));   // true
```
``Object.getPrototypeOf()`` 返回 [[Prototype]] 的值。
```javascript
alert(Object.getPrototypeOf(person1) == Person.prototype);  // true
alert(Object.getPrototypeOf(person1).name);  // "Nicholas"
```
当给对象实例添加一个属性，这个属性回屏蔽原型对象中保存的同名属性。可以使用 delete 操作符来删除实例属性，访问原型属性。如下图：
![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3wcbn6htj30o70fgjt2.jpg)
```javascript
function Person() {
}

Person.prototype.name = "Nicholas";
Person.prototype.age = 29;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function() {
    alert(this.name);
};

var person1 = new Person();
var person2 = new Person();

alert(person1.hasOwnProperty("name"));  // false

person1.name = "Greg";
alert(person1.name);  // "Greg" -- 实例
alert(person2.name);  // "Nicholas" -- 原型

alert(person1.hasOwnProperty("name"));  // true
alert(person2.hasOwnProperty("name"));  // false

delete person1.name;
alert(person1.name);  // "Nicholas" -- 原型

alert(person1.hasOwnProperty("name"));  // false
```
``hasOwnProperty()`` 用来检测一个属性是否存在于实例中。

### 原型与 in 操作符 #####

```javascript
function Person() {
}

Person.prototype.name = "Nicholas";
Person.prototype.age = 29;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function() {
    alert(this.name);
};

var person1 = new Person();
var person2 = new Person();

alert(person1.hasOwnProperty("name"));  // false
alert("name" in person1);  // true

person1.name = "Greg";
alert(person1.hasOwnProperty("name"));  // true
alert("name" in person1);  // true

```
无论属性存在实例中还是原型中，调用 "name" in person1 始终会返回 true。同时使用 hasOwnProperty() 方法，可以确定该属性是存在对象还是在原型中，如下：
```javascript
function hasPrototypeProperty(object, name) {
    return !object.hasOwnProperty(name) && (name in object);
}
```
如果需要遍历取出属性，如下：
```javascript
var o = {
    toString: function() {
        return "my object";
    }
}

for (var prop in o) {
    if (prop == "toString") {
        alert("Found toString");  // 在IE中 不会显示
    }
}
```
要取得对象中所有可以枚举的实例属性，可以用 ``Object.keys()`` 方法。获得所有的实例属性可以用 ``Object.getOwnPropertyNames()`` 方法。
```javascript
function Person() {
}

Person.prototype.name = "Nicholas";
Person.prototype.age = 29;
Person.prototype.job = "Software Engineer";
Person.prototype.sayName = function() {
    alert(this.name);
};

var keys = Object.keys(Person.prototype);
alert(keys);  // "name, age, job, sayName"

var keyAll = Object.getOwnPropertyNames(Person.prototype);
alert(keyAll);  // "constructor, name, age, job, sayName"
```

### 更简单的原型语法 #####

用对象字面量来重写原型对象，如下：
```javascript
function Person() {
}

Person.prototype = {
    name: "Nicholas",
    age: 29,
    job: "Software Engineer",
    sayName: function() {
        alert(this.name);
    }
};
```
但是 constructor 属性也不再指向 person ，而是指向 Object 构造函数。因此需要重新指定。
```javascript
function Person() {
}

Person.prototype = {
    constructor: Person,  // 重新指定
    name: "Nicholas",
    age: 29,
    job: "Software Engineer",
    sayName: function() {
        alert(this.name);
    }
};
```
以这种方式重设 constructor 属性会导致 [[Enumerable]] 特性被设为 true 。
```javascript
function Person() {
}

Person.prototype = {
    constructor: Person,  // 重新指定
    name: "Nicholas",
    age: 29,
    job: "Software Engineer",
    sayName: function() {
        alert(this.name);
    }
};

// 重设构造函数，只适用于 ECMAScript5兼容的浏览器
Object.defineProperty(Person.prototype, "constructor", {
    enumerable: false,
    value: Person
});
```
### 原型对象的问题 #####
原型中所有的属性都是被实例共享的，对于包含引用类型值的属性，就会有问题。
如下：
```javascript
function Person() {
}

Person.prototype = {
    constructor: Person, 
    name: "Nicholas",
    age: 29,
    job: "Software Engineer",
    friends: ["Shelby", "Court"],
    sayName: function() {
        alert(this.name);
    }
};

var person1 = new Person();
var person2 = new Person();

person1.friends.push("Van");

alert(person1.friends);  // "Shelby, Court, Van"
alert(person2.friends);  // "Shelby, Court, Van"
alert(person1.friends == person2.friends);  // true
```
修改了 person1 的数组 friends，在 person2 实例中也反映出来。

## 组合使用构造函数模式和原型模式 ##

这个方式是创建自定义类型最常见的方式。构造函数模式用于定义实例属性，而原型模式用于定义方法和共享的属性。如下：
```javascript
function Person(name, age, job) {
    this.name = name;
    this.age = age;
    this.job = job;
    this.friends = ["Shelby", "Court"];
}

Person.prototype = {
    constructor: Person,
    sayName: function() {
        alert(this.name);
    }
}

var person1 = new Person("Nicholas", 29, "Software Engineer");
var person2 = new Person("Greg", 27, "Doctor");

person1,friends,push("Van");
alert(person1.friends);  // "Shelby, Court, Van"
alert(person2.friends);  // "Shelby, Court"
alert(person1.friends == person2.friends);  // false
alert(person1.sayName == person2.sayName);  // true
```
这是用来定义引用类型的一种默认模式。

## 动态原型模式 ##

可以通过检查某个应该存在的方法是否有效，来决定是否需要初始化原型。
如下：
```javascript
function Person(name, age, job) {
    // 属性
    this.name = name;
    this.age = age;
    this.job = job;

    // 方法
    if (typeof this.sayName != "function") {
        Person.prototype.sayName = function() {
            alert(this.name);
        );
    }
}

var friend = new Person("Nicholas", 29, "Software Engineer");
friend.sayName();
```
只有在 ``sayName()`` 方法不存在的情况下，才会将它添加到原型中。这段代码只会在初次调用构造函数时才会执行。

## 寄生构造函数模式 ##

在前述几种模式都不适用的情况下，用寄生（parasitic）构造函数模式。
模式基本思想是创建一个函数，函数的作用是封装创建对象的代码，返回新创建的对象，如下：
```javascript
function Person(name, age, job) {
    var o = new Object();
    o.name = name;
    o.age = age;
    o.job = job; 
    o.sayName = function() {
        alert(this.name);
    };
    return o;
}

var friend = new Person("Nicholas", 29, "Software Engineer");
friend.sayName();  // "Nicholas"
```
寄生构造函数返回的对象与构造函数或者与构造函数的原型属性之间没有关系。

## 稳妥构造函数模式 ##

稳妥对象是指没有公共属性，其方法不引用 this 的对象，稳妥对象适合在一些安全环境下或者防止数据被其他应用程序改动时使用。

```javascript
function Person(name, age, job) {
    // 创建要返回的对象
    var o = new Object();
    
    // 可以在这里定义私有变量和函数
    
    // 添加方法
    o.sayName = function() {
        alert(name);
    };
    // 返回对象
    return o;
}
```
除了使用 ``sayName()`` 方法外，没有其他方法可以访问到 name 的值。
