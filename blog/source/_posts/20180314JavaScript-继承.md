---
title: JavaScript-继承
date: 2018-03-14 09:52:56
tags: JavaScript
---
<!-- 标签别名 -->
{% cq %} 雨打梨花深闭门，忘了青春，误了青春。<br/><br/><strong>—— 唐寅《一剪梅·雨打梨花深闭门》</strong> {% endcq %}
<!-- more -->

## 原型链 ##

构造函数、原型和实例的关系：每个构造函数都有一个原型对象，原型对象都包含一个指向构造函数的指针，而实例都包含一个指向原型对象的内部指针。而继承的思路就是 让原型对象等于另一个类型的实例。
如下例：

```javascript
function SuperType() {
    this.property = true;
}
SuperType.prototype.getSuperValue = function() {
    return this.property;
};

function SubType() {
    this.subproperty = false;
}

// 继承了 SuperType
SubType.prototype = new SuperType();

SubType.prototype.getSubValue = function() {
    return this.subproperty;
};

var instance = new SubType();
alert(instance.getSuperValue());  // true
```
SubType 继承了 SuperType，原来存在于 SuperType 的实例中的所有属性和方法，现在也存在于 SubType.prototype 中。
如下图：
![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3wd4np0sj30ph0e1jsv.jpg)
所有引用类型都默认继承了 Object，这个继承也是通过原型链实现。

### 定义方法 ###
子类要重写超类中的方法，或者需要添加超类中不存在的方法，要把代码放到替换原型的语句后。
如下：
```javascript
function SuperType() {
    this.property = true;
}
SuperType.prototype.getSuperValue = function() {
    return this.property;
};

function SubType() {
    this.subproperty = false;
}

// 继承了 SuperType
SubType.prototype = new SuperType();

// 添加方法
SubType.prototype.getSubValue = function() {
    return this.subproperty;
};

// 重写超类型中的方法
SubType.prototype.getSuperValue = function() {
    return false;
};

SubType.prototype.getSubValue = function() {
    return this.subproperty;
};

var instance = new SubType();
alert(instance.getSuperValue());  // false
```
在通过原型链实现继承时，不能使用对象字面量创建原型方法，这样会重写原型链。

### 原型链问题 ###
问题一：超类中包含引用类型的原型属性，就会被所有的实例共享。
如下：
```javascript
function SuperType() {
    this.colors = ["red", "blue", "green"];
}

function SubType() {
}

//继承了 SuperType
SubType.prototype = new SuperType();

var instance1 = new SubType();
instance1.colors.push("black");
alert(instance1.colors);  // "red, blue, green, black"

var instance2 = new SubType();
alert(instance2.colors);  // "red, blue, green, black"
```
实例 instance1 修改原型中的 colors 属性，在实例 instance2 中也会反映出来。
问题二：在创建子类型的实例时，不能向超类型的构造函数中传递参数。

## 借用构造函数（伪造对象或经典继承）##

思路：在子类型构造函数的内部调用超类型构造函数。

```javascript
function SuperType() {
    this.colors = ["red", "blue", "green"];
}

function SubType() {
    // 继承了 SuperType
    SuperType.call(this);
}

//继承了 SuperType
SubType.prototype = new SuperType();

var instance1 = new SubType();
instance1.colors.push("black");
alert(instance1.colors);  // "red, blue, green, black"

var instance2 = new SubType();
alert(instance2.colors);  // "red, blue, green"
```

通过使用 ``call()`` 方法（或者 ``apply()`` 方法），在 SubType 实例的环境下调用 SuperType 构造函数。这样 SubType 的每个实例就会具有自己的 colors 属性副本。

### 传递参数 ###

```javascript
function SuperType(name) {
    this.name = name;
}

function SubType() {
    // 继承了 SuperType，同时还传递了参数
    SuperType.call(this, "Nicholas");

    // 实例属性
    this.age = 29;
}

var instance = new SubType();
alert(instance.name);  // "Nicholas";
alert(instance.age);  // 29
```

### 借用构造函数的问题 ###

方法都在构造函数中定义，函数无法做到复用。

## 组合继承（伪经典继承） ##

思路：使用原型链实现对原型属性和方法的继承，而通过借用构造函数来实现对实例属性的继承。这样，既通过在原型上定义方法实现了函数复用，又能保证每个实例都有自己的属性。
如下：
```javascript
function SuperType() {
    this.name = name;
    this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayName = function() {
    alert(this.name);
};

function SubType(name, age) {
    // 继承属性
    SuperType.call(this, name);
    this.age = age;
}

// 继承方法
SubType.prototype = new SuperType();

SubType.prototype.sayAge = function() {
    alert(this.age);
};

var instance1 = new SubType("Nicholas", 29);
instance1.colors.push("black");
alert(instance1.colors);  // "red, blue, green, black"
instance1.sayName();  // "Nicholas"
instance1.sayAge();  // 29

var instance2 = new SubType("Greg", 27);
alert(instance2.colors);  // "red, blue, green"
instance2.sayName();  // "Greg"
instance2.sayAge();  // 27
```
超类型 SuperType构造函数定义了两个属性：name 和 colors。SupertType 的原型定义了一个方法 ``sayName()`` 。SubType 构造函数在调用 SuperType 构造函数时传入 name 参数，紧接着又定义了自己的属性 age。然后，将 SuperType 的实例赋值给 SubType 的原型，然后又在该新原型上定义了方法 ``sayAge()`` 。这样，可以让不同的 SubType 实例既分别拥有自己的属性包括 colors 属性，又可以使用相同的方法。组合继承避免了原型链和借用构造函数的缺陷，融合了优点，是最常用的继承模式。

## 寄生式继承 ##

```javascript
function createAnother(original) {
    var clone = object(original);  // 通过调用函数 创建一个新对象
    clone.sayHi = function() {  // 以某种方式来增强这个对象
        alert("hi");
    };
    return clone;
}

var person = {
    name: "Nicholas",
    friends: ["Shelby", "Court", "Van"],
};

var anotherPerson = createAnother(person);
anotherPerson.sayHi();  // "hi"
```
代码中 ``createAnother()`` 函数接收参数，作为新对象的基础对象，将对象传递给 ``object()`` 函数， 将返回值结果赋值给 clone 。再给 clone 对象添加新方法 ``sayHi()`` ，最后返回 clone 对象。任何能够返回新对象的函数都适用这个模式。

## 寄生组合式继承 ##

组合继承最大的问题是 无论在什么情况下，都会调用两次超类型构造函数：一次是在创建子类型原型的时候，另一次是在子类型构造函数内部。
如下：
```javascript
function SuperType(name) {
    this.name = name;
    this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayName = function() {
    alert(this.name);
};

function SubType(name, age) {
    SuperType.call(this, name);  // 第二次调用 SuperType()

    this.age = age;
}

SubType.prototype = new SuperType();  // 第一次调用 SuperType()
SubType.prototype.constructor = SubType;
SubType.prototype.sayAge = function() {
    alert(this.age);
};
```
下图展现了上述代码过程：
![](http://ww1.sinaimg.cn/large/007ipZvVgy1fy3wdj1r2rj30os0h7q4n.jpg)
寄生组合式继承是通过借用构造函数来继承属性，通过原型链的混成形式来继承方法，其实就是不必为了指定子类型的原型而调用超类型的构造函数。
下面是寄生组合式继承的基本模式：
```javascript
function inheritPrototype(subType, superType) {
    var prototype = object(superType.prototype);  // 创建对象
    prototype.constructor = subType;  // 增强对象
    subType.prototype = prototype;  // 指定对象
}

function SuperType(name) {
    this.name = name;
    this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayName = function() {
    alert(this.name);
};

function SubType(name, age) {
    SuperType.call(this, name);

    this.age = age;
}
//
inheritPrototype（SubType, SuperType);

SubType.prototype.sayAge = function() {
    alert(this.age);
};
```
普遍认为寄生组合式继承是引用类型最理想的继承范式。