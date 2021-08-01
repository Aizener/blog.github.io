---
title: 关于JS的原型与继承相关概念
date: 2021-07-31 16:13:15
tags:
  - JavaScript
categories:
  - Web前端
---

> 在ES6后，JavaScript已经可以通过class、extends等语法糖实现面向对象的写法，但是以前的原型对象也是必须得了解的，看文章的话记不清楚，自己总结一下。

### 构造函数

在说原型前，有必要了解一下以前的面向对象的写法，JavaScript的对象是通过构造函数实例获得的：

```
function Fo(msg) {
  this.msg = msg
}
```

`Person`就是一个构造函数，一般首字母大写。并且有两个形参：`name`和`sex`，赋值给了`this`对象。

此时，可以通过`new`的操作获取一个`Person`的实例：

```
const f = new Foo('Hello')
console.log(f.msg)  // Hello
```

这里有一个问题，为什么通过`man`这个实例对象能访问到`msg`呢，这里就要说一下`new`的操作了，其实在`new`实例化构造函数时，内部做了以下操作：

```
var obj = {}
obj.__proto__ = Person.prototype
Person.call(obj)
return obj
```

上面看到了两个奇怪的东西，就是`__proto__`和`prototype`，那这两个又是啥呢？

### 原型对象和原型链

`__proto__`：每个实例对象都有这一个属性，与构造函数的原型函数对应；

`prototype`：原型对象，每一个构造函数都有这一个对象。

从下面这张图，我们可以看到两者的关系：


![src=http___image.bubuko.com_info_201810_20181030000300294146.png&refer=http___image.bubuko.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b52e9cfb37a246de82e90b579c83a9d9~tplv-k3u1fbpfcp-watermark.image)

从图中可以看到，实例对象的`__proto__`是指向其构造函数的`prototype`的，并且有JavaScript在使用属性或方法时有一个特点就是，**在当前实例对象寻找需要调用的属性或方法，如果没有，则向实例对象的**proto**（也就是构造函数的prototype对象上找），直到找到或者到顶层的null为止。**

并且从代码中，可以看到是完全相等的：

```
console.log(f.__proto__ === Fo.prototype) // true
```

由`f.__proto__` => `Fo.prototype` => `Fo.prototype.__proto__` => `Object.prototype` => `null`，这就形成了一条链式的结构，这就是所谓的**原型链**。

### 继承

JavaScript的继承分为两种情况，第一种是继承属性，可以通过`call`方法来实现，第二种就是方法的继承，需要通过原型链来实现，看一下代码：

```
function Foo(msg) {
  this.msg = msg
}

Foo.prototype.printMsg = function () {
  console.log(this.msg)
}

function Fo(msg, greet) {
  Foo.call(this, msg)
  this.greet = greet
}

Fo.prototype = new Foo()
Fo.prototype.constructor = Fo

Fo.prototype.printGreet = function () {
  console.log(this.greet)
}

const fo = new Fo('Hello', 'How are you?')
console.log(fo)
```

打印这个`fo`，可以看到下图：

![7B@MK{E)KPVE95HFD55LU4B.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c9cb1d472c7c4f44ba128294d2aaf17b~tplv-k3u1fbpfcp-watermark.image)  
明显的，`greet`属性和`msg`属性就是属于`Fo`实例对象的，然后看看`printMsg`和`printGreet`这两个方法：
1. `printGreet`此时是在`fo`的`__proto__`上的，即`Fo.prototype`上；  
2. 又因为`Fo.prototype`又指向了`Foo`的实例对象,所以`Fo.prototype`其实是指向`Foo`的实例对象的；
3. 最后可以看到`printMsg`是挂在`Foo.prototype`原型对象上的。
所以，可以看出我们访问`fo`对象的属性时，本身就在实例对象上，可以直接访问到；  
而当访问方法时，是通过访问`fo.__proto__`即`Fo.prototype`即`Foo`实例对象上找的，又根据**原型链的特点**，访问不到对应方法时，会往`Foo`实例对象的`__proto__`属性即`Foo.prototype`上找，所以就实现了方法的继承。
### 总结
其实，JavaScript的对象是通过`new`构造函数得到的，`new`的过程内部已经帮我们处理过了；  
继承是通过原型链的特点来实现的，把“子类”的`prototype`指向“父类”的实例对象，这样通过原型链查找时，自然会找从子类找向父类。