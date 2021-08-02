---
title: 简单聊聊防抖和节流
date: 2021-08-02 22:27:55
tags:
  - JavaScript
categories:
  - Web前端
---

## 简单聊聊防抖和节流

> 前端优化性能的方式有很多种，其中就包括防抖和节流，防抖和节流不光能优化性能，还能应用在一些特定的场景，这里就来简单叙述一下，其实就是想水这个挑战（我也不想水，奈何肚里墨水太少）。

### 概念
- 防抖：规定一个`time`时间段，在这个时间段内，不管操作多高频，我们只执行最后一次；
- 节流：规定一个`time`时间段，在高频的操作里，我们每隔`time`时间，执行一次事件。

看两张图防抖和节流的示例图：

防抖：

![niceh.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/23f347cf7f044f5b8b879f1a028c081e~tplv-k3u1fbpfcp-watermark.image)

节流：

![niceg.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/35199bf1181a4a5f8dfe1085a6ce5dd4~tplv-k3u1fbpfcp-watermark.image)

### 防抖

防抖就是在某一个时间内，不管事件触发多频繁，只执行一次事件，减少处理事件的次数，从而优化网站性能，看看简单的代码实现：

```js
function debounce(callback, delay = 500) {
    let timer = null
    return function () {
        clearTimeout(timer)
        timer = setTimeout(callback.bind(this, arguments), delay)
    }
}

const handleScroll = debounce(function() {
    console.log(this, arguments)
})
```

通过上面的代码就实现了一个简单的防抖函数，通过`handleScroll`函数来实现请求之类的业务操作，然后再把这个函数放在高频操作的场景里，即可。

但是，目前防抖是出于执行之后才开始执行，假如我们需要一开始操作就执行一次呢？实现也很简单：
```js
function debounce(callback, delay = 500, start = false) {
      let timer = null, _start = start
      return function () {
        if (_start) {
          callback.apply(this, arguments)
          _start = false
        }
        clearTimeout(timer)
        timer = setTimeout(() => {
          callback.apply(this, arguments)
          _start = start
        }, delay)
      }
    }

const handleScroll = debounce(function() {
    console.log(this, arguments)
}, 500, true）
```

在`debounce`返回的回调函数里面，我们判断是否开始操作：  
如果是，则立即先执行一次`callback`，再把`_start`重置为`false`，在本次防抖结束后，再把`_start`置为`true`，这样下一次事件开始时，又会走`if`判断了，这样就实现效果了。

这里使用`_start`而不是直接使用`start`的原因是因为，我们如果直接操作`start`会导致下一次防抖开始时`start`的状态就不正确了。

### 节流

节流和防抖有点类似，也是限制事件的执行，但是节流是在高频操作内，限制事件每隔一段时间执行一次，看看简单的代码实现：

```js
function throttle(callback, delay = 500) {
    let timer = null
    return function () {
        if (timer) return
        timer = setTimeout(() => {
            callback.apply(this, arguments)
            timer = null
        }, delay)
    }
}

 const handleScroll = throttle(function() {
     console.log(this, arguments)
 })
```

节流和防抖的实现很相似，我这里都是通过`setTimeout`来实现，区别就是在于：  
我这里没有`clearTimeout`来清楚定时器了，而是重置`timer`为`null`；因为在定时器执行期间，即使再次调用函数，也会在`if`判断里面返回，所以再清楚定时器也没有太大意义。

和防抖一样的，现在的节流在一开始执行时，也不会先调用一次函数，想实现这个效果的话相比较防抖就要复杂一点了：因为回调函数现在会每隔一段时间执行，所以再直接重置`start`就会造成函数的重复执行了。

所以，这里采用了再定义一个定时器来实现效果：

```js
function throttle(callback, delay = 500, start = false) {
    let timer = null, resetTimer = null
    return function () {
        if (timer) {
            clearTimeout(resetTimer)
            return
        }
        if (start) {
            callback.apply(this, arguments)
            start = false
        }
        timer = setTimeout(() => {
            callback.apply(this, arguments)
            timer = null
            resetTimer = setTimeout(() => start = true, delay)
        }, delay)
    }
}
```

因为，我们每隔`delay`这个时间段会执行一次`callback`，所以就可以通过再引入一个定时器`resetTimer`在`delay`时间段后重置`start`为`true`；  
因为如果用户一直在做类似滚动的操作时，我们会不停消除再定义，只有当用户在`delay`时间段不再操作后，`resetTimer`才会执行。

### 总结

防抖和节流的实现方式有很多种，例如时间错之类的，这里我只是间我比较喜欢的方式实现，因为我觉得`setTimeout`确实比较好用，也没遇到什么问题。

`debounce`和`throttle`各有特点，在不同 的场景要根据需求合理的选择策略。

如果事件触发是高频但是有停顿时，可以选择`debounce`；

在事件连续不断高频触发时，只能选择`throttling`，因为`debounce`可能会导致动作只被执行一次，界面出现跳跃。
