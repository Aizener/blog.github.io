---
title: 学习canvas之实现简单绘制
date: 2021-08-08 17:06:51
tags:
  - JavaScript
categories:
  - Web前端
  - canvas专栏
---

> 学习canvas断断续续已经有个小几天了，除了看下API，也不知道该怎么去学习。所以，又开始试图去模拟一些小工具，当然功能方面是能简单就简单，毕竟还在初学阶段，太复杂做不出来，又损信心，思来想去，想模拟一个绘制的工具，但是却没想到绘制难的一匹，差的被弄自闭了。

先来看看做出来的效果图吧：

![nicel.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/24debbaae44c4338875472714c9055cd~tplv-k3u1fbpfcp-watermark.image)

个人目前实力太菜了，能做到这里感觉已经花了很大精力了，绘制真的太难了。

## 知识点

除了刚开始学过用过的那些`API`以外，做绘制的时候，我又接触到了`strokeRect`，`arc`，`toDataURL`等`API`的用法，当然也比较简单：

这里先说说两个绘制图形的`API`吧：

1. `strokeRect`：就是通过以线条的方式来绘制一个矩形；
   
    - 参数：  
        `x`：开始的横坐标位置；  
        `y`：开始的纵坐标位置；  
        `width`：绘制的矩形宽度；  
        `height`：绘制的矩形高度；  
    
2. `arc`：就是通过以线条的方式来绘制一个矩形；
   
    - 参数：  
        `x`：圆心的横坐标位置；  
        `y`：圆心的纵坐标位置；  
        `r`：圆的半径大小；
        `sAngle`：起始角，以弧度计；  
        `eAngle`：结束角，以弧度计；  
        `counterclockwise`： 可选。规定应该逆时针还是顺时针绘图。False = 顺时针，true = 逆时针；

看一下简单的例子吧：
```js
ctx.beginPath()
ctx.strokeStyle = 'red'
ctx.strokeRect(10, 10, 100, 100)
ctx.beginPath()
ctx.strokeStyle = 'green'
ctx.arc(60, 60, 50, 0, 2 * Math.PI)
ctx.stroke()
```
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9671bedeffae4ec0a3371a4e8b8feae3~tplv-k3u1fbpfcp-watermark.image)

可以看到`arc`和`strokeRect`的使用方式还是有不同的一点，`arc`只是描述了路径，而绘制还是需要`stroke`来实现，从两者名字就可以看出来了。

## 实现绘制

实现一个简单的绘制功能还是不难的，当然如果要复杂一点的话，现在我觉得：那可真是太难太难了，要考虑和做的地方实在太多。

### 思考
实现一个简单的绘制的功能的话，得考虑几点三个要素：
- 落笔点；
- 移笔路径；
- 收笔点；

这三个要素的话，分别可以通过`mousedown`，`mousemove`，`mouseup`来实现：
|    API    |         概念         |
| :-------: | :------------------: |
| mousedown |   当鼠标按下时触发   |
| mousemove |   当鼠标移动时触发   |
|  mouseup  | 当鼠标按下弹起时触发 |

这里，因为`addEventListener`是可以重复监听来绑定事件的，在切换绘制图形的时候会造成`Bug`，也没找到好的方法解决；
所以我没有用这个`API`来绑定事件，而是用的以前的`onmousedown`这种形式。

### 开始

好，既然思路已经有了，那就开始实现吧，先来把鼠标的事件监听写好再谈下一步：
```js
cvs.onmousedown = function (e) {
  isStart = true
  console.log('mousedown')
  cvs.onmousemove = function (e) {
    isStart && console.log('onmousemove')
  }
}


cvs.onmouseup = function (e) {
  isStart = false
  console.log('onmouseup')
}
```
这里我们把`mousemove`事件要绑定在`mousedown`里面，因为我们期望的时：当笔落下移动时，在绘制。  
看了看浏览器的执行效果，发现了一个问题，就是`mousemove`的执行频率简直太快了：

![nicem.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6c62deb36a28487e9a2d9ee8ee8444d9~tplv-k3u1fbpfcp-watermark.image)

可以看到，我只是轻轻滑动几下，就执行力几百次的事件，如果后面再加上需要操作画布的代码的话，效率肯定会很慢的，所以我这里加了节流：

```js
const move = throttle(function (e) {
  console.log('mousemove')
})

cvs.onmousedown = function (e) {
  isStart = true
  console.log('mousedown')
  cvs.onmousemove = function (e) {
    isStart && move.call(this, e)
  }
}

function throttle (callback, delay = 10) {
  let timer = null
  return function () {
    const ctx = this
    if (timer) {
      return
    }
    timer = setTimeout(() => {
      callback.apply(ctx, arguments)
      timer = null
    }, delay)
  }
}
```

优化效果：

![nicen.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7f1ea8d7f30140c6a7c5c0e1f040943b~tplv-k3u1fbpfcp-watermark.image)

在使用节流后，可以明显的看到执行的频率不像之前那么快乐，这样就稍微能接受了。

### 绘制

在完成事件的监听后，我们就得思考怎么来绘制图形到画板上去了。

#### 直接绘制

直接绘制的画，其实很简单，我们只需要在`mousemove`的时候，使用`context`对象进行绘制每一个像素点即可达到目标：
```js
const move = throttle(function (e) {
  ctx.lineTo(e.offsetX, e.offsetY)
  ctx.stroke()
})
cvs.onmousedown = function (e) {
  isStart = true
  ctx.beginPath()
  ctx.strokeStyle = '#000'
  cvs.onmousemove = function (e) {
    isStart && move.call(this, e)
  }
}
```
![niceo.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c69022de23cf4721a713dd060ce24324~tplv-k3u1fbpfcp-watermark.image)

要点就是在`mousedown`的时候`beginPath`，重新开启绘制路径，以免和上一次的绘制连线；然后通过`mousemove`事件不断绘制就OK了。

#### 绘制直线/绘制矩形/绘制圆形

直接绘制的话很简单，但是绘制直线的话，就稍微麻烦了：因为直线是由两点构成，而我们`mousemove`时可是会有很多个点的，我思考过后，是这么实现的：  

记录`mouseup`的第一个位置，作为起始点； 然后记录每次`mousemove`的最后一次值来作为结束点，进而连城一条直线。那么`mousemove`的最后一次的点怎么来获得呢？ 

我这里通过每次`mousemove`时，`celarRect`画布信息，然后再`lineTo`第一个点，这样就可以实现效果了！

```js
const move = throttle(function (e) {
  ctx.clearRect(0, 0, 600, 600)
  ctx.moveTo(start.x, start.y)
  ctx.lineTo(e.offsetX, e.offsetY)
  ctx.stroke()
})

cvs.onmousedown = function (e) {
  isStart = true
  ctx.beginPath()
  ctx.strokeStyle = '#000'
  start.x = e.offsetX
  start.y = e.offsetY
  cvs.onmousemove = function (e) {
    isStart && move.call(this, e)
  }
}
```
这样看似好了。但是，却遇到了奇怪的事情：

![nicep.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c7ed8a5883a4955b2332dc58c1a3e8c~tplv-k3u1fbpfcp-watermark.image)

可以看到，每次绘制的直线没有被`clearRect`给清除掉，想了很久都没有找到原因，后面发现当在清除后，重新开启绘制路径就没问题了：
```js
const move = throttle(function (e) {
  ctx.clearRect(0, 0, 600, 600)
  ctx.beginPath()
  ctx.moveTo(start.x, start.y)
  ctx.lineTo(e.offsetX, e.offsetY)
  ctx.stroke()
})

cvs.onmousedown = function (e) {
  isStart = true
  ctx.beginPath()
  ctx.strokeStyle = '#000'
  start.x = e.offsetX
  start.y = e.offsetY
  cvs.onmousemove = function (e) {
    isStart && move.call(this, e)
  }
}
```

![niceq.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b28f30cdc9f24641b5b0998488b512af~tplv-k3u1fbpfcp-watermark.image)

这样的话，效果就正常了。绘制矩形和绘制圆也是一样的思路，就是调用`API`的时候，处理方面有一些不同。这样，看起来就实现效果了嘛？  
答案是NO，我们又发现了一个问题，就是通过不断的`clearRect`清除画布后，之前所画的就也会被清除掉，那这样，还怎么算画图！

#### 保存记录

思考一会儿后，我的思路就是引入一个保存画图记录的`record`对象数组，通过这个数组来保存每一次画图的操作，然后再`clearRect`清空画布后，再把记录里面的路径重新弄绘制上来（后面复杂点的话，通过这个记录甚至还能实现撤销的功能）。

```js
const move = throttle(function (e) {
  ctx.clearRect(0, 0, 600, 600)
  record.forEach(item => {
    ctx.beginPath()
    ctx.moveTo(item.x1, item.y1)
    ctx.lineTo(item.x2, item.y2)
    ctx.stroke()
  })
  ctx.beginPath()
  ctx.moveTo(start.x, start.y)
  ctx.lineTo(e.offsetX, e.offsetY)
  ctx.stroke()
})
cvs.onmousedown = function (e) {
  isStart = true
  ctx.beginPath()
  ctx.strokeStyle = '#000'
  start.x = e.offsetX
  start.y = e.offsetY
  cvs.onmousemove = function (e) {
    isStart && move.call(this, e)
  }
}

cvs.onmouseup = function (e) {
  isStart = false
  record.push({ x1: start.x, y1: start.y, x2: e.offsetX, y2: e.offsetY})
}
```
![nicer.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/58f96384e16d40cf96335607c0b286cb~tplv-k3u1fbpfcp-watermark.image)

这样，就可以保留之前所绘制的效果了。按照这种思路，绘制圆、矩形、和直接绘图都应该要保存记录，只是保存的方式略有差异而已。

#### 保存图片

绘制图形我们搞完了，后面就应该是保存图片了。保存图片的话，前面有说到一个`API`是`toDataURL`，这个`API`不是`context`上下文的，而是`canvas`的方法：

```js
const canvas = document.querySelector('canvas')
const url = canvas.toDataURL()
```
这个`API`有俩参数：
第一个是类型，默认为：`image/png`；
第二个是质量：默认为：0.92；

具体可以看看[toDataURL MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLCanvasElement/toDataURL)。

知道这个`API`怎么使用之后，接下来我们就可以完成保存图片的操作了：

```js
function saveImage () {
  const link = document.createElement('a')
  link.download = '文件名'
  link.href = cvs.toDataURL()
  link.click()
}
```
当然，通过`toDataURL`获取到数据后，还可以赋值给图片，实现预览效果，需要注意的是：  
1. 这里我通过`createElemennt`创建的图片，直接`src`赋值是无效的，而是要通过`new Image`对象来实现；
2. 图片其实是通过`base64`的方式来展示的。

## 总结

再贴贴最后的效果图吧：
![nicel.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/24debbaae44c4338875472714c9055cd~tplv-k3u1fbpfcp-watermark.image)

原本以为绘制比较简单，但没有想到实现一个最基本的绘制都遇到很多困难了。  
而且，想往后实现复杂点的功能话，难度是越来越大，不过还好，磕碜的弄出一个Demo来，也算是勉强合格吧。  
往后，还是先评估一下实现难度，再从简单的学起来吧。
