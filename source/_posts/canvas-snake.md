---
title: 学习canvas并完成一个贪吃蛇
date: 2021-08-06 21:35:33
tags:
  - JavaScript
categories:
  - Web前端
  - canvas专栏
---

> 随着H5的到来，很多新特性都让人眼前一新。其中，canvas就是一个非常有用的特性，基于canvas我们可以完成图表、游戏等工具的实现。学前端这么久，都没好好看过canvas，这里打算正式学一下，这是第一站：看基础知识，并完成一个贪吃蛇。

贪吃蛇算是比较一个简单的例子了，看了基础语法后，还是磕碜的写出来了，先看看示例：

![nicej.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2cb8a465491149aa85b060ebc7f43b8c~tplv-k3u1fbpfcp-watermark.image)

[在线试玩](https://yangxiang.cc/canvas_demos/)

## 基础
通常使用`canvas`，我们需要在`html`里有至少一个`canvas`标签，以便来获取`canvas`对象，进而来创建出一个画布，然后就可以在画布上实现你的主意了。
### 创建画布
通过`getContext`这个`API`，我们就可以获取一个`canvas`的上下文对象，也就是我们需要操作的画布了。
```js
const canvas = document.getElementById('#canvas')  // canvas id
const ctx = canvas.getContext('2d') // 获取一个2d的canvas上下文，就是一个2d画布
```
在获取上下文对象之前，我们还可以通过给`canvas`设置`width`和`height`来定义画布的宽和高，这里要注意一点：  

**给canvas通过css设置的宽高和给canvas对象设置的宽高是不一样的；**  
**有效的宽高应该是给canvas对象设置，通过css设置的可能会造成显示模糊。**

> Tip：可以看到我们获取的context对象是2d的，那么有没有3d呢？哈哈，是没有的。或许是因为WebGL出来的原因，canvas可能已经不会推出3d的模式了吧。

### 基础方法
在获取画布对象后，我们需要在画布里面做操作的话，就需要了解基础的`API`了，这里我来介绍一下最基础的`API`使用：  
1. `beginPath`：开始或重置路径，这个`API`非常有用，当我们想画平行线时就得使用这个方法了，因为平行线我们没法一笔画完，所以得用`beginPath`来开始新路径。
```js
ctx.beginPath()
ctx.lineTo(10, 10)
ctx.lineTo(100, 100)
ctx.lineTo(60, 10)
ctx.lineTo(150, 100)
ctx.stroke()
```
看看以上没有使用`beginPath`绘制的结果：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/43badc4e52bf4ed8a881b1b77e18dd49~tplv-k3u1fbpfcp-watermark.image)

可以看到，线条很奇怪的连接起来了，这是因为我们使用`lineTo`时没有开始新的绘制，看看正确的使用：
```js
ctx.beginPath()
ctx.lineTo(10, 10)
ctx.lineTo(100, 100)
ctx.stroke()
ctx.beginPath()
ctx.lineTo(60, 10)
ctx.lineTo(150, 100)
ctx.stroke()
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/262860af8c4646d29e4031332b5e618d~tplv-k3u1fbpfcp-watermark.image)

上面两个Demo就很好的看出了`beginPath`的使用，这个方法很常用的。

2. `closePath`：千万别把这个方法和`beginPath`搞混，其实他俩看起来像，实则没有关联，`closePath`是用来闭合路径的，看一个简单的例子就明白了。
```js
ctx.beginPath()
ctx.lineTo(10, 10)
ctx.lineTo(100, 100)
ctx.lineTo(100, 10)
ctx.closePath()
ctx.stroke()
```
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/18eae1bda18045c0b78ff111319d655e~tplv-k3u1fbpfcp-watermark.image)
`closePath`把上面两点闭合起来了，形成了一个直角三角形，没有`closePath`的话，上面就不会有一条边来封闭成图形了。
3. `moveTo`：把画笔放在某一个点，表示从这个点开始；
4. `lineTo`：把画笔移动到某一个点，其实和moveTo没有太大的区别。
5. `strokeStyle`：填充路径的样式，让线条呈现出不同的样式。
```js
ctx.beginPath()
ctx.strokeStyle = 'red'
ctx.lineTo(10, 10)
ctx.lineTo(100, 100)
ctx.stroke()
ctx.beginPath()
ctx.strokeStyle = 'blue'
ctx.lineTo(60, 10)
ctx.lineTo(150, 100)
ctx.stroke()
```
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8b0cb48164174889a0436610f83c2ba9~tplv-k3u1fbpfcp-watermark.image)

可以看到，平行线不再是单一的黑色了，而是变成了红蓝色平行线。这里要注意的是，`strokeStyle`会影响到后面的绘制，所以我们在绘制不同的路径时，看需求是否要重新设置`strokeStyle`。

6. `stroke`：以画线的方式完成路径的绘制，不调佣这个方法的话，无论怎么`lineTo`还是不会画出线条的。
7. `fillStyle`：填充区域的样式。
8. `fill`：以填充的方式完成路径的绘制，`fill`可以按照画的线条路径，在里面填充出图形。
```js
ctx.beginPath()
ctx.strokeStyle = 'red'
ctx.lineTo(10, 10)
ctx.lineTo(100, 100)
ctx.lineTo(200, 10)
ctx.fillStyle = 'red'
ctx.fill()
```
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d44880c3a106468fbdb8fe138af02d9e~tplv-k3u1fbpfcp-watermark.image)

9. `drawImage`：画一张图片在画布上，参数是：`Image`, `x`, `y`, `width`, `height`。
`Image`是通过`new Image`获得的实例对象：
```js
const img = new Image()
img.src = '...'
ctx.drawImage(img, 0, 0, 100, 100)
```
10. `clearRect`：清楚画布的一个范围，参数是：`x`, `y`, `width`, `height`。

以上仅仅10个非常简单的`API`，其实就足够我们完成许多的小`Demo`了，关键在于如何实现，接下来，我们就来想想怎么通过`canvas`画出一个贪吃蛇来。

#### 绘制图形

通过以上的基础`API`，我们就可以画出很多的图形了，并且可以封装一些常用图形库了。不过，`canvas`其实已经把常用的一些图形，给我们封装为方法了，所以可以直接使用了，例如`fillRect`就可以直接绘制出一个方形图形了。

贪多嚼不烂，我这里就先用熟简单的`API`。遇到要用的再去看，感兴趣的话可以先看看[Canvas API](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API)。

还有更多的，暂时不做赘婿，这里仅仅是入门，我们通过这些简单的`API`就可以完成很多小功能了，例如实现一个简单的贪吃蛇游戏。

## 贪吃蛇思路
要实现一个贪吃蛇游戏的话，我们需要考虑这些：
- 需要绘制的：
    - 地图：其实就是一个简单的方形平面，蛇需要在上面范围移动；
    - 蛇：蛇是有一个一个方形组成的，通过吃果实可以增加长度；
    - 果实：果实是随机产生在地图上的，可以被蛇吃下。
- 需要检测的：
    - 蛇：蛇在碰到果实时，会吃掉增加长度；碰到自己或者地图边界时会死亡；
    - 果实：被蛇碰到时会被吃掉。
 ### 开始
 为了方便写代码，我们可以先画上地图，这样可以看到每个方格，更直观，后期完成可以再去掉方格：
 ```js
function drawBoard () {
  for (let i = 1 ; i < boxNum ; i ++) {
    ctx.beginPath()
    ctx.moveTo(0, i * boxSize)
    ctx.lineTo(800, i * boxSize)
    ctx.stroke()
    ctx.beginPath()
    ctx.moveTo(i * boxSize, 0)
    ctx.lineTo(i * boxSize, 800)
    ctx.stroke()
  }
}
 ```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/552d14fb53bd44f7846683bc63882680~tplv-k3u1fbpfcp-watermark.image)

这样，我们在写蛇的移动、和果实生成时，就比较方便了。

### 小蛇的状态

蛇应该是由一个个小方格组成，状态我们通过一个对象数组来保存，里面的每个对象来记录`x`和`y`的坐标位置。
 ```js
let snake = [{ x: 3, y: 5}, { x: 4, y: 5}, { x: 5, y: 5 }]
function drawSnake (snake) {
  snake.forEach((item, idx) => {
    ctx.beginPath()
    const x = (item.x - 1) * boxSize
    const y = (item.y - 1) * boxSize
    const w = boxSize
    const h = boxSize
    if (snake.length - 1 === idx) {
      ctx.fillStyle = 'red'
      ctx.fillRect(x, y, w, h)
    } else {
      ctx.fillStyle = 'green'
      ctx.fillRect(x, y, w, h)
    }
  })
}
 ```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/658cebb31f834e0dae036ab344121be6~tplv-k3u1fbpfcp-watermark.image)

这样，我们就可以在画布上画出蛇的一个轮廓了，接下来就是该考虑小蛇怎么来移动身体了。

### 蛇的移动
怎样让小蛇移动呢？我们所绘画出来的小蛇是通过保存状态的`snake`来生成的，所以我们就得考虑一点，当蛇移动的时候，该怎样修改呢？

假设，蛇的头部是数组最后一个对象，那么身子就是其他的对象，小蛇的移动除了头部是可变的（操作有上下左右），其他部分其实就是**身子的第一格移动到头部，第二格移动到第一格**，所以，我们就可以知道小蛇移动的规律了，通过规律生成新的小蛇对象。
```js
function getSnake() {
  const _snake = []
  for (let i = 1 ; i < snake.length ; i ++) {
    _snake.push(snake[i])
  }
  const head = snake[snake.length - 1]
  newHead = { x: head.x + 1, y: head.y } // 假设小蛇往右移动的
  _snake.push(newHead)

  return _snake
}
```

小蛇新的对象生成了，那么怎么来实现小蛇移动的动画效果呢？

### 实现动画效果

小蛇的移动肯定离不开定时器，当然有其他更好的方法也行，我这里使用的是`setTimeout`。  
思路就是每隔一段时间重绘一次，并且重绘后再次递归调用方法，通过在方法里不停的绘画每一帧的结果，就能实现动画效果了。

```js
let timer = null
move ()
function move () {
  timer = setTimeout(() => {
    ctx.clearRect(0, 0, 600, 600)
    drawBoard()
    snake = getSnake()
    drawSnake(snake)
    move()
  }, 100)
}
```
在每次调用`move`也就是移动小蛇的方法时，我们首先的清楚画布，所以要用到`clearRect`这个方法；  
然后再调用画地图的方法和画小蛇的方法，把地图再次画出来，并且把小蛇最新的状态画出来；因为每隔`100ms`我们会重新递归调用再绘画一次，所以看起来就会有移动的效果了：

![nicek.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2973777bcb83453cb8cdbbc59f7b48cf~tplv-k3u1fbpfcp-watermark.image)

### 修改方向

修改小蛇的方向很简单，无法就是通过监听键盘事件然后修改`x`和`y`的值，所以肯定得有一个保存当前方向的状态：
```js
const type = 'right'
function listenerKeyboard() {
    document.addEventListener('keyup', e => {
        switch(e.key) {
        case 'ArrowLeft': this.type !== 'right' && this.type = 'left'; break;
        case 'ArrowUp': this.type !== 'down' && this.type = 'up'; break;
        case 'ArrowRight': this.type !== 'left' && this.type = 'right'; break;
        case 'ArrowDown': this.type !== 'up' && this.type = 'down'; break;
        }
    })
}
```
通过判断按键的`key`值，再修改类型。这里，我判断了当按键为右时不能按左，上时不能按下，反之亦然。当然，这里还是会有`Bug`的，就是组合按键的时候亦然会出问题。

### 判断死亡

判断死亡很简单，就是判断小蛇头部是否与大于或小于边界的临界值，亦或者小蛇头部的坐标和身体的坐标重合：
```js
check() {
    const head = this.snake[this.snake.length - 1]
    if (
      head.x > this.boxNum ||
      head.x < 1 ||
      head.y > this.boxNum ||
      head.y < 1 ||
      this.isEatSelf(head)
    ) {
      clearTimeout(this.timer)
      this.timer = null
      alert('Game Over.')
      return true
    } {
      return false
    }
}
```
当小蛇达到死亡的条件后，就要清楚定时器并返回一个`boolean`值来作为判断。

### 生成果实

生成果实非常简单，就是两个随机数，有点问题的就是，当生成的果实与小蛇重叠时应该重新生成（或者就不允许生成出重叠的），我这里为了简单，就随便实现了：
```js
generate() {
    if (this.box.x) {
      const x = (this.box.x - 1) * this.boxSize
      const y = (this.box.y - 1) * this.boxSize
      this.ctx.beginPath()
      this.ctx.fillStyle = 'green'
      this.ctx.fillRect(x, y, this.boxSize, this.boxSize)
      return
    }
    let result = { x: Math.floor(Math.random() * 19 + 1), y: Math.floor(Math.random() * 19) + 1}
    const isRepeat = this.snake.some(item => item.x === result.x && item.y === result.y)
    if (isRepeat) {
      result = this.generate()
      return
    }
    this.ctx.fillStyle = 'green'
    const x = (result.x - 1) * this.boxSize
    const y = (result.y - 1) * this.boxSize
    this.ctx.fillRect(x, y, this.boxSize, this.boxSize)
    this.box = { x: result.x, y: result.y }
}
```
生成果实后，我们得有一个对象来保存果实的状态，因为如果小蛇没吃到果实是应该一直重新绘制在当前状态的。

### 吃掉果实

当小蛇头部与果实坐标相等时，小蛇就应该吃掉果实了，并且身体得加上一截，也就是数组得新增一个对象。这里，我们采用`unshift`的方法来添加元素，因为`snake`数组的最后一个元素是头部：
```js
handleEat() {
    const last = this.snake[this.snake.length - 1]
    if (last.x === this.box.x && last.y === this.box.y) {
      this.box = {}
      this.eatNum ++
      this.speed -= this.eatNum
      this.snake.unshift(this.first)
    }
}
```
这里的`first`其实是在上面`getSnake`时应该保存的`snake`第一个元素。

### 加载图片，美化效果

通过以上步骤，就能基本实现一个简单的贪吃蛇了。但是，现在的贪吃蛇看起来像是像素游戏，所以我们可通过`drawImage`这个方法来替代`fillRect`来绘制小蛇。  

但是，现在还有个问题，就是，通过图片绘制小蛇的话，一开始游戏加载的时候图片出不来怎么办？所以，我们就得事先加载玩图片后，再走后面的逻辑了：

```js
Promise.all([
  this.loadImg('headImgLeft', './head-left.png'),
  this.loadImg('headImgUp', './head-up.png'),
  this.loadImg('headImgRight', './head-right.png'),
  this.loadImg('headImgDown', './head-down.png'),
  this.loadImg('bodyImg', './body.webp'),
]).then(() => {
  this.headImg = this.headImgRight
  this.box = {}
  this.type = 'right'
  this.speed = 120
  this.eatNum = 0
  this.ctx = canvas.getContext('2d')
  this.snake = [{ x: 3, y: 5}, { x: 4, y: 5}, { x: 5, y: 5 }]
  this.init()
})

loadImg(name, src) {
    return new Promise((resolve) => {
      this[name] = new Image()
      this[name].src = src
      this[name].onload = () => resolve()
    })
}
```
通过`Promise`亦或者是`async/await`我们就可以在资源加载完后再执行后面代码，资源过大的话可以适当增加 进度条做提示。

### 结语

基本上，最简单基本的贪吃蛇就完成了。

![nicej.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2cb8a465491149aa85b060ebc7f43b8c~tplv-k3u1fbpfcp-watermark.image)

[在线试玩](https://yangxiang.cc/canvas_demos/)

当然，我们可以增加积分、加速、暂停、重玩等功能，这些使用`canvas`的最基本`API`就能完成了，增加什么功能，就是基于大家的`idea`了。

## 总结
我也是刚开始学，这是第一个小步，感觉就是`canvas`非常的强大，可以实现天马行空的效果。但是，前提是对`API`的熟练使用以及在复杂应用时，或许得需要较扎实数学知识吧。  
当然，也可以实现很实用的工具，像[html2canvas](https://github.com/niklasvh/html2canvas)还有[echarts图表](https://echarts.apache.org/en/index.html)等等。

不过，现在还差的很远，还得要慢慢学习，才能慢慢熟练`canvas`来使用。
