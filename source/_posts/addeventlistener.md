---
title: 关于通过addEventListener添加事件监听的一些踩坑笔记
date: 2021-07-31 16:11:15
tags:
  - JavaScript
categories:
  - Web前端
---

## 定义和用法
`addEventListener()` 方法用于向指定元素添加事件句柄, 与`onclick`这类事件相比较的特点是可多次添加事件的监听，而不会被覆盖。  

## 语句
`element.addEventListener(event, function, useCapture)`

## 参数
|参数|值|描述|
|---|---|---|
|event|String|监听的事件，比如click（不要on）
|callback|functoin|回调函数，触发事件时要执行的函数
|useCapture|Boolean(false)|事件是否在捕获阶段触发

## 示例1

```js
// box是获取的DOM对象。
box.addEventListener('click', function () {
    console.log('Click Box.');
});
box.addEventListener('click', function () {
    console.log('Click Box.(Repeat)');
});
// 点击box后，打印出：
// Click Box.
// Click Box.(Repeat)
```

关于捕获和冒泡的话，个人理解为：

> 事件捕获就是从父级元素开始往子级元素一层一层执行，而冒泡是从子级到父级一层一层执行，所以**捕获事件会先与冒泡事件**。

## 示例2
```html
<body>
    <div id="box">
        <div id="box1">
    	    Box1
    	    <div id="box2">
    	    Box2
    	    </div>
        </div>
        Box
    </div>
    <script>
        var box = document.querySelector('#box');
        var box1 = document.querySelector('#box1');
        var box2 = document.querySelector('#box2');
        
        box.addEventListener('click', function () {
    	    console.log('Click Box.');
        });
        
        box1.addEventListener('click', function () {
    	    console.log('Click Box1.');
        });
        
        box2.addEventListener('click', function () {
	    console.log('Click Box2.');
        });
    </script>
</body>
```
上面代码，我没有传入第三个参数，默认的是事件触发在冒泡阶段，所以在点击Box2后应该是先从子级往父级执行，执行结果应该为：Click Box2. Click Box1. Click Box.  

**效果图**：
![](https://user-gold-cdn.xitu.io/2020/3/21/170fbac282cb0a0a?w=851&h=646&f=png&s=29820)

所以，如果我把box，box1，box2添加监听时的事件发生状态修改为捕获状态时，执行顺序就是从box到box2了。  

**小思考：如果box，box2为冒泡阶段发生事件，而box1为捕获阶段发生事件时，点击box2会怎么打印呢？**

答案是： box1, box2, box。因为box1是捕获阶段，所以肯定先执行box1；然后box2和box都是冒泡阶段，因为此时捕获阶段已完成，冒泡阶段是从子级往父级一层层触发，所以先执行box2，再执行box。

## 移除事件与其中的坑
移除监听是通过`removeEventListener`来实现的，需要注意的是，第一个参数和第三个参数要保持一致，那么第二个参数回调方法呢？这个地方其实有坑！  

首先，如果要移除方法最常规的是定义一个有名子的函数，通过传入函数名来监听事件，这样移除监听就比较方便了，例如：
```js
var doClick = function () {
    console.log('event.');
}

box.addEventListener('click', doClick, false);

setTimeout(() => {
    box.removeEventListener('click', doClick, false);
}, 2e3);
```
为什么不直接写匿名函数呢？因为匿名函数的地址是不同的，所以用以下方式无法移除监听：
```js
box.addEventListener('click', function () {
    console.log('event.');
}, false);

setTimeout(() => {
    box.removeEventListener('click', function () {
        console.log('event.');
    }, false);
}, 2e3);
```
那么，添加监听事件时如果传入的是匿名函数时就无法移除监听了吗？答案是：不是的。
```js
var callee = null;

box.addEventListener('click', function () {
    callee = arguments.callee;
    console.log('event');
}, false);

setTimeout(() => {
    box.removeEventListener('click', callee, false);
}, 2e3);
```
可以通过`arguments.callee`来获取回调函数，再通过`removeEventListener`来删除即可。

**关于移除事件另外的坑**
就是改变了`this`指向传入的回调函数不能被移除。
```js
var doClick = function () {
    console.log('Click Box.');
}

box.addEventListener('click', doClick.bind(this), false);

setTimeout(() => {
    box.removeEventListener('click', doClick.bind(this), false);
}, 2e3);
```
这段代码看起来都是传入了`doClick.bin(this)`，觉得是没问题的。  
可是实际上，这段代码是无法移除事件的监听的，所以再移除事件监听时要注意两点：  

- 匿名回调函数移除监听时需要`callee`协助；
- 回调函数改变了上下文无法移除监听（不知道有没有解决方法）。