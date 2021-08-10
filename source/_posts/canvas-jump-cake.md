---
title: 学习canvas之实现喵咪跳蛋糕
date: 2021-08-11 00:29:40
tags:
  - JavaScript
categories:
  - Web前端
  - canvas专栏

---

> 这几天学习canvas看了很多基础API，也用了不少，这里再做一个总结练习，做一个稍微有趣的：喵咪跳蛋糕小游戏，以便来再巩固总结canvas的学习，并且这个游戏也是我十分喜爱的，不过因为个人canvas水平是刚入门，所以只能做一个简版的了。

基础感觉差不多了，进阶的话还不知道怎么搞，后面看看开源库吧。  
废话不多说，先来看看效果图：

![nices.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/64a946886dee433d8243f2dc391afcf9~tplv-k3u1fbpfcp-watermark.image)


![nicet.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/95d4f1ce144d411e881fa5263d02aa94~tplv-k3u1fbpfcp-watermark.image)

PC/移动的在线试玩地址（Github Page需要代理，不然进入可能会很慢或者进不了）：  

试玩：[在线试玩](https://yangxiang.cc/canvas-jumpcake/)

因为接触canvas时间较短，做的时间也不长，也缺乏素材，所以效果很一般，不过也算勉强合格，至少能有基本的游戏逻辑。

## 开始

又要逼逼赖赖说一下用的`API`了，这里其实没用到什么新的`API`，就是多学习了一个绘制文本的`API`，这个`API`的效果还是非常棒的，这里介绍`fillText`了吧，估摸着`strokeText`也差不了多少：

先说一下`fillText`的4个参数：

|   参数    |                   作用                    |
| :-------: | :---------------------------------------: |
|   text    |         规定在画布上输出的文本。          |
|     x     | 开始绘制文本的 x 坐标位置（相对于画布）。 |
|     y     | 开始绘制文本的 y 坐标位置（相对于画布）。 |
| maxWidth* |   可选。允许的最大文本宽度，以像素计。    |

```js
ctx.font = '30px "微软雅黑"'
ctx.fillText('Hello', 30, 50)
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c2521991f7d0447d9eb5cca5ad64bf5a~tplv-k3u1fbpfcp-watermark.image)

其实，使用`fillText`这类`API`不仅仅只靠这几个参数，像通过`fillStyle`、`font`等我们也可以修改绘制的w文本，比如我现在说的这个比较有用的`textAlign`属性：  
很有意思，`textAlign`的`left`和`right`等属性是以坐标点来划分的。一张图看懂这个属性：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d9e5338e158642709eb181f1009e71e3~tplv-k3u1fbpfcp-watermark.image)

更多的属性具体还可以看看文本相关的`API`介绍：[Canvas文本APIMDN文档](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial/Drawing_text)。

## 喵咪跳蛋糕

接下来，我就说一下我使用`canvas`来做喵咪跳蛋糕小游戏的思路。其实，我觉得吧，`canvas`很多`API`其实单独使用起来是不难的，关键是组合起来完成某一个东西，难度就比较大了，有些还需要数学知识。  
废话就不多说了，开始来看看喵咪是怎么跳蛋糕的。

### 要素

喵咪跳蛋糕小游戏是这样的：  

1. 猫咪可以点击跳起来然后自由落体；
2. 蛋糕会从边界一边移动到另一边；
3. 小猫若是自由落体踩中则叠加起来，撞死则Game Over！
   所以，思路基本清楚了：  
   要画的的是猫咪和蛋糕，然后就是实现碰撞检测计算新位置即可！

### 创建方格地图

为啥喜欢在做某一些Demo时，我比较喜欢首先画一个方格地图呢，因为方便观察和计算物体绘画的点，对于刚学习的我来说，用处还是非常大的。

```js
initBoard(len) {
    const num = Math.floor(this.cvs.width / len)
    for (let i = num ; i >= 0 ; i --) {
      this.ctx.beginPath()
      this.ctx.strokeStyle = '#ccc'
      this.ctx.lineTo(i * len, 0)
      this.ctx.lineTo(i * len, this.cvs.width)
      this.ctx.stroke()
      this.ctx.beginPath()
      this.ctx.lineTo(0, i * len)
      this.ctx.lineTo(this.cvs.width, i * len)
      this.ctx.stroke()
    }
}
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4fa37ace436e452f9f0fb91329e4d786~tplv-k3u1fbpfcp-watermark.image)

方格可以弄小一点，通过观察方格，我们也可以更方便的给物体设置大小、位置、移动速度（以几个方格为标准）等属性。

### 创建Role也就是小喵咪

小喵咪的创建很简单，因为小喵咪除了在`Y轴`的方向会因为叠加蛋糕变化外，其他是不会变化的，所以我们只需要一开始把喵咪画出来，然你后动态增加`y`值即可：

```js
this.role = { x: 0, y: 0, t: 0 }
this.initRole(this.cvs.width - 10)
```

```js
initRole(y) {
    this.role.x = this.cvs.width / 2
    this.role.y = y
    this.role.t = this.cvs.width - y
}
paintRole() {
    this.ctx.beginPath()
    this.ctx.arc(this.role.x - 10, this.role.y, 10, 0, 2 * Math.PI)
    this.ctx.fillStyle = 'red'
    this.ctx.fill()
}
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cb9380bee8d24ab9996db7d2cd343a5d~tplv-k3u1fbpfcp-watermark.image)

这里我用这个小球表示我们的`Role`喵咪，可以看到喵咪已经在底部中间的位置了，现在就差点击跳动了，这里的话实现方式应该会有很多种，我这里就随便简单实现了下：

```js
paintRole() {
    this.ctx.beginPath()
    if (this.isClick) {
      // 判断角色是否已到落地点（落地点可能在方块上）
      if (this.role.y >= this.cvs.width - this.role.t && this.type === 'down') {
        this.speed = 2.8
        this.role.y = this.cvs.width - this.role.t
        this.isClick = false
        this.type = 'up'
        return
      }
      this.role.y -= this.speed
      if (this.speed <= 0) {
        this.type = 'down'
      } else {
        this.type = 'up'
      }
      this.speed -= .1
    }
    this.ctx.arc(this.role.x - 10, this.role.y, 10, 0, 2 * Math.PI)
    this.ctx.fillStyle = 'red'
    this.ctx.fill()
}
```

![niceu.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bfa6671e7bef458ea18c3926d9777c55~tplv-k3u1fbpfcp-watermark.image)

当然，`paintRole`这个方法会一直通过定时器不停的重绘，都入门一段时间了，这些代码逻辑就不贴了。  
这里的主要思路就是：  

点击的时候是朝上减速的方向，然后又落体加速，所以我这里就直接通过`speed--`来实现了；  

然后再落下的时候即`type`为`down`的时候，我们要计算小球的当前位置和落地点位置（这里使用的是属性`t`来记录，因为落地点可能因为蛋糕的叠加而不同，所以单独储存），计算到达落地点后重置状态即可。

### 创建蛋糕并移动

创建蛋糕的方式也很简单，就是从一边到另一边，先从简单的情况开始：蛋糕只是变化`X轴`的位置，那么我们每次绘图时增加`x`的值即可。

```js
paintCake() {
    if (this.cake.x >= this.cvs.width) {
      this.cake.x = -this.cake.w
    }
    this.cake.x ++
    this.ctx.beginPath()
    this.ctx.fillStyle = 'skyblue'
    this.ctx.fillRect(this.cake.x, this.cake.y, this.cake.w, this.cake.h)
}
```

![nicev.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2202ec823a29468a934531e6f2d87fa2~tplv-k3u1fbpfcp-watermark.image)

这样即可实现方块的移动了，现在我们就要实现检测的机制了，这里的话检测稍微麻烦一点，因为小球不仅撞着要死，落体运动踩在蛋糕时是会叠加的。

### 碰撞检测

碰撞的检测还是挺麻烦的，这里说一下思路就行：  

1. 当小球落体时，出于蛋糕的范围`X轴`范围里面的话，当`Y轴`距离大于等于了蛋糕的`Y轴`点时即踩上蛋糕，其他碰撞则为失败。
2. 当小球踩上蛋糕后，我们得记录当前被踩的蛋糕的位置，引入`record`对象数组来记录；
3. 因为叠加了蛋糕，所以要重置小球的和蛋糕出现的`Y轴`点。
   看一下代码的简单实现：

```js
check() {
    if (
      this.role.x + 5 >= this.cake.x &&
      this.role.x - 5 <= this.cake.x + this.cake.w && 
      this.role.y >= this.cake.y
    ) {
      if (this.type === 'down') {
        if (this.cakes.length) {
          // 增加判定范围，方便重叠蛋糕
          const last = this.cakes[this.cakes.length - 1]
          const range = 5
          if (
            (this.cake.x > last.x - range && this.cake.x < last.x + range) ||
            (this.cake.x > last.x + last.w - range && this.cake.x < last.x + last.w + range)
          ) {
            this.scoreRatio += 1
            this.cake.x = last.x
            this.paintRatioText()
          } else {
            this.scoreRatio = 1
          }
        }
        this.moveEnd = true
        this.score += this.scoreRatio
        this.cakes.push({ x: this.cake.x, y: this.cake.y })
        this.initCake(this.cake.y - 10)
        this.initRole(this.role.y - 10)
      } else {
        this.ctx.font = '30px "微软雅黑"'
        this.ctx.textAlign = 'center'
        this.ctx.fillStyle = 'red'
        this.ctx.fillText('Game Over!', this.cvs.width / 2, this.cvs.width / 2)
        clearTimeout(this.timer)
        this.timer = null
        this.end = true
      }
    }
}
```

因为踩蛋糕，如果在同一位置的话，应该是分越加越高，但是按`1px`来计算的话，这样难以完全重叠，所以我设置了一个范围区间；  
这里我引入了一个`range`值，只要当前踩中的和上一个的位置误差不超过这个`range`，则算两块蛋糕重叠，并适当做连击提示：

```js
// 绘制连击
paintRatioText() {
    this.showRatioText = true
    this.ctx.font = '15px "微软雅黑"'
    this.ctx.textAlign = 'left'
    this.ctx.fillStyle = 'red'
    this.ctx.fillText('X' + this.scoreRatio, this.role.x + 30, this.role.y - 15)
    setTimeout(() => {
      this.showRatioText = false
    }, 1e3)
}
```

```js
// 重绘记录，也就是把踩中的蛋糕再绘上去paintReocrd() {    this.cakes.forEach(cake => {      this.ctx.beginPath()      this.ctx.fillStyle = 'orange'      this.ctx.fillRect(cake.x, cake.y, this.cake.w, this.cake.h)    })}
```

![nicew.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c807f9952b0546d898d7f8b13ce9dd0e~tplv-k3u1fbpfcp-watermark.image)

### 降低高度

因为蛋糕是会一个一个重叠上去的，所以我们得在一定高度后，降低蛋糕的高度，方法有两种：  

1. 直接删除数组得元素；
2. 把数组里面的对象`Y`值给改了；
   我这里用的第二个方法，因为想到就做了，没考虑太多...

```js
// 移动蛋糕，以免叠太高了超出范围check() {    ...    if(this.cakes.length && this.cakes.length % 10 === 0 && this.moveEnd) {        this.moving = true        this.moveEnd = false        setTimeout(() => {        this.moving = false        this.cake.y += this.movingY        this.role.t -= this.movingY        this.movingY = 0        }, 550)     }    this.initCake(this.cake.y - 10)    this.initRole(this.role.y - 10)    ...}
```

```js
initRole(y) {    this.role.x = this.cvs.width / 2    this.role.y = y    this.role.t = this.cvs.width - y}initCake(y) {    this.cake.x = -this.cake.w    this.cake.y = y}// 移动蛋糕moveCake() {    const y = this.cakes.length > 10 ? 2 : 1    this.cakes = this.cakes.map(cake => {      return {        ...cake,        y: cake.y + y      }    })}// 移动角色moveRole() {    const y = this.cakes.length > 10 ? 2 : 1    this.movingY += y    this.role.y += y}
```

这里我通过`moving`这个状态来保存：蛋糕是否在下移状态，当下移完成时，新蛋糕才重新绘制。  
然后使用`movingY`来保存下移的距离，使得小球和当前蛋糕的`Y`轴距离同步。

![nicex.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a19b3c7ecff34c12942d5c3c115228ae~tplv-k3u1fbpfcp-watermark.image)

这样的话，游戏的基本逻辑就已经大致完成了，剩下的就是一些优化部分了。

### 优化

例如，我们可以把小球和蛋糕替换成功图片，并且把方格背景去掉。  

像喵咪的话，我们还可以找一组素材，这里我找了一张跳的和一张坐的，这样的话，跳起来自由落体运动就不会看起来很单一，显得人物比较丰富。

图片的话，还是需要预先加载的，否则没有图片就开始游戏逻辑很傻瓜的。  

也可以加一些蛋糕依据速度的变化而变快，出来的方向不同等玩法，让游戏更加的丰富。

如果可以的话在连击时可以加一些特效的效果，甚至加上`websocket`使得可对战，当然这些都是后话了。

## 总结

游戏是简单完成了，效果得话很一般，可能是因为我绘画的一些逻辑没处理的太好吧，所以看起来只是功能比较正常，视觉就差了火候。

不过也好了，`canvas`完了几天入门应该还是入门了的，`API`的使用还是非常简单的，只是要通过这些`API`组合起来使用要完成某个功能就比较麻烦了。  

所以，我觉着吧，`canvas`这玩儿意，我觉得首先对代码设计的要好是一方面，另一方面觉得数学要好，不然计算坐标宽度等还是挺打脑壳的。

后续的话也不知道咋个进阶，太难的很难做出来，精力也不够，还要学`Vue`恰饭，再去了解了解开源库吧。

再看看效果把：  

![nices.gif](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/64a946886dee433d8243f2dc391afcf9~tplv-k3u1fbpfcp-watermark.image)


![nicet.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/95d4f1ce144d411e881fa5263d02aa94~tplv-k3u1fbpfcp-watermark.image)

试玩：[在线试玩](https://yangxiang.cc/canvas-jumpcake/)。

好了，`canvas`入门达成：获得称号：新手canvas玩家！
