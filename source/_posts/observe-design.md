---
title: JavaScript之观察者模式
date: 2021-08-03 22:40:15
tags:
  - JavaScript
categories:
  - 设计模式
---

> 设计模式是开发人员在开发过程中面临的一般问题的解决方案。这些解决方案是众多开发人员经过相当长的一段时间的试验和错误总结出来的。其中，观察者模式就是前端比较常用的一种了，许多场景就是基于该模式下实现的。

### 概念

观察者模式和发布订阅者模式类似，但是发布订阅是通过中间层来通讯的，而观察者模式是由主题（Subject）直接通知观察者（Observe）来传递信息，生活中也有常见的例子：

1.  例如我每天早晨定的闹钟，就是我所观察的主题，每天早晨闹铃一响，就通知我该起床上班了；

2.  比如我上班回家后，肚子饿了，点份外卖（主题），外卖到了就会通知我去拿；

3.  吃完饭，洗衣服时，洗衣机洗完后会响起来告诉我洗完了，就可以凉衣服了。

    ......（场景有很多，就不一一举例了）

总而言之，言而总之，观察者模式的核心就是：

**一个主题可被多个观察者观察，观察者通过主题的通知来触发相应事件**

### 实现

实现一个简单的观察者模式很简单，就是一个`Subject`类作为主题，一个`Observe`类作为观察者。

`Subject`至少提供一个可以添加`Observe`对象的方法，还得有一个可通知`Observe`的方法；

`Observe`至少得有一个方法，在`Subject`类通知时，可做对应的操作。

```js
class Subject {
    constructor() {
        this.observes = []
    }
    add(observe) {
        this.observes.push(observe)
    }
    notifyAll() {
        this.observes.forEach(obs => obs.update())
    }
}

class Observe {
    constructor(name) {
        this.name = name
    }
    update() {
        console.log(this.name)
    }
}

const sub = new Subject()
const obs1 = new Observe('观察者1')
const obs2 = new Observe('观察者2')
sub.add(obs1)

sub.notifyAll()
```

可以看到，在执行`sub.notifyAll`后，打印出`obs1`观察者的名字，这是因为`obs1`观察者观察了`Subject`主题，而`obs2`没有观察`Subject`主题。

-   Subject：主题类，被观察的主题，一个主题可以被多个观察者观察；
-   Observe：观察者，观察主题，依赖主题的通知而做出响应；
-   add：添加观察者来观察主题；
-   notifyAll：通知所有的观察者；
-   update：当主题通知后，观察者做出的响应。

复杂一点的话，还可以给`Observe`类和`Subject`类添加更多的操作来满足其他的需求。当然，实现方式不只是通过使用类的面向对象来实现，也可以通过其他方式实现。

观察者应用还挺广泛，像浏览器事件的监听，还有非常经典的Vue的视图渲染，就是利用的观察者模式，我们也可以通过`Object.defineProperty`来实现一个简单的Demo：

```js
class Subject {
    constructor() {
        this.observes = []
    }
    add(observe) {
        this.observes.push(observe)
    }
    notifyAll() {
        this.observes.forEach(obs => obs.update())
    }
}

class Observe {
    constructor(subject) {
        this.subject = subject
        this.subject.add(this)
    }
    update() {
        console.log('视图更新啦！')
    }
}
const sub = new Subject()
const obs = new Observe(sub)

window.obj = {}
let greet 
Object.defineProperty(obj, 'greet', {
    set(val) {
        greet = val
        sub.notifyAll()
    },
    get() {
        return greet
    }
})
```

![nicei.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/275b0376d006470484a5b68b04d9e7a1~tplv-k3u1fbpfcp-watermark.image)

可以看到，当我修改`greet`属性的时候，就会由主题`sub`通知观察者`obs`来打印出文字，而修改其他属性的时候则不会有任何响应。  

这里除了`obs`是观察了`sub`以外，还发现了其实我们也依赖了`defineProperty`定义的属性，即也观察了`greet`，或许其实`Object.defineProperty`也是一种观察者模式实现的吧。

### 总结

设计模式与语言和语法无关，只是前人总结出来的一种思想，可以由任何语言实现出该模式；  
他的优点就是使得观察者和主题抽象耦合，且耦合程度很低，有助于扩展与重用；并且还能够进行简单的广播通信。
