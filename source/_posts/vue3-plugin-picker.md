---
title: vue3-plugin-picker
date: 2021-08-22 19:11:20
tags:
  - Vue
  - Vue3
categories:
  - Web前端
---

> Vue3出现一段时间了，也有看过基础知识，所以准备小小造一个轮子。Picker组件的话，是H5里面比较常用的一个组件，这里就拿它开刀，实现一个非常简单的Picker组件（当时别人问我自己有写过插件没，我说开源有很多能用的，还没遇到自己写的，就被一脸鄙视了，于是也动动手）。

废话不多说，先看看实现的效果图：

![nicey.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/393358d7b7ff4affad1667822cc4b9e4~tplv-k3u1fbpfcp-watermark.image)

## 开发环境

这里的话还是用的`Vue3.0`的版本，还没有去用`script setup`这种语法（虽然3.0也支持-_-||）。

```json
"dependencies": {
    "vue": "^3.0.0"
},
"devDependencies": {
    "@vue/cli-plugin-typescript": "~4.5.0",
    "@vue/cli-service": "~4.5.0",
    "@vue/compiler-sfc": "^3.0.0",
    "typescript": "~4.1.5"
}
```

## 实现步骤

实现一个Picker插件分仨步骤：
`创建自定义插件` -> `实现插件的功能` -> `使用插件`。

### 自定义插件的创建

在`Vue`中，插件其实就是**一个对象且这个对象里面有一个`install`方法，这个方法可以接受一个`Vue`实例**：
```js
import { App } from 'vue'
import Picker from './Picker.vue'

const pickerPlugin = {
  install: (app: App) => {
    app.component('x-picker', Picker)
  }
}

export default pickerPlugin
```

这里，我通过自定义插件的方式创建了一个全局组件，这个组件就是我们接下来要写的`Picker`组件。

### 创建组件Picker

最简单的`Picker`组件至少都应该接收一个`list`数组和一个可修改显示状态的`value`，所以这里的`props`我是这样写的：
```js
props: {
    list: {
      type: Array,
      default: () => []
    },
    modelValue: {
      type: Boolean,
      default: false
    }
}
```
我这里接收的是`modelValue`，所以父组件可以通过`v-model`的方式来双向绑定使用。

接着，在思考过后，`Picker`组件包含两个部分：一个是渲染列表并实现拖动，第二个就是派发事件显示或隐藏。所以，我这里就抽成了两个方法，具体的实现就在方法里面写。  

这样也是利用了`CompositionAPI`的特点嘛，写起来是挺舒服的。

```js
import useList from './list'
import useEvent from './event'

setup(props, { emit }) {
    const { ctx }: any = getCurrentInstance()
    const $useList = useList(props)
    const $useEvent = useEvent({ emit, ctx })
    return {
      ...$useList,
      ...$useEvent
    }
}
```

### 组件列表的拖动实现

组件拖动的话，我们得知道`Vue`提供的`@touchstart`和`@touchmove`两个手势方法。  
通过记录点击时的位置和手指移动时的位置，计算差值就可以获得移动的一个距离；然后通过修改`transform`的方式来使得列表移动。  

当然在实现时我还是推荐弄一个函数节流，因为`move`的方法触发频率挺高。

```js
touchstart(e: TouchEvent) {
    // 记录开始的位置 e.touches[0].clientY
},
touchmove(e: TouchEvent) {
    // 计算差值offY，然后修改列表transform的translate属性
},
const getOffsetY = computed(() => {
  return {
    transform: `translate(-50%, ${offY.value}px)`
  }
})
```

这样的话，是能实现拖动效果得，但是却发现一个很严重的问题，就是移动时列表一顿一顿的，于是我添加了`transition`属性来过渡，虽然改善了一些，但效果还是存在很大问题。

于是，于是我就去`VantUI`官网看了看[VantUI Picker](https://vant-contrib.gitee.io/vant/#/zh-CN/picker)组件，发现他们的过渡是这样写的：

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ec3d9c602ee4eadaf7d710eb621b3ff~tplv-k3u1fbpfcp-watermark.image)

```css
transition-timing-function: cubic-bezier(0.23, 1, 0.68, 1);
```
然后，我加了这个后，发现效果是比之前棒很多了，但是操作的感觉比起`VantUI`还是差了很多，这个问题还没发现怎么回事，应该是实现的方案存在问题吧。

### 组件拖动后回弹

完成组件拖动后，我们发现一个问题就是，拖动超出顶部和底部是存在问题的。所以，需要给一个安全距离，当超过这个距离后，就不允许拖动。并且，超过顶部或底部就要回弹到边界上去。

通过`@touchend`事件，就可以很好的解决该问题：
```js
touchend() {
    setTimeout(() => {
        offY.value = 边界的值
    }, 100)
}
```
只要在手势事件结束后，重新设置组件偏移的位置即可。这里加`setTimeout`的原因是因为：  
有时候发现操作快了之后，计算属性得到的结果是对的，但是视图渲染的位置不对。

### 事件触发/显示隐藏

完成了拖动的功能后，就得去完成触发事件和显示隐藏的功能了，这个也比较简单：
1. 事件的触发我们可以通过`setup`里面的第二个参数解构获得`emit`方法来实现：
```js
setup(props, { emit }) {
    handleConfirm() {
        emit('confirm', ctx.)
    }
}
```
2. `v-model`的实现同样也利用`emit`方法来派发一个事件，父组件使用`v-model`后会自动监听该事件并做对应的赋值操作：
```js
setup(props, { emit }) {
    handleConfirm() {
        emit('update:modelValue', bool)
    }
}
```
3. 通过事件我们可以修改组件的状态（显示/隐藏），现在就要做显示和隐藏这个功能。利用常规的`v-show`我们就可以很轻易的达到目的，但是这样显得很生硬，所以这里利用了`transition`组件：
```css
.slide-picker-enter-active, .slide-picker-leave-active {
  transition: all .5s;
}
.slide-picker-enter-from, .slide-picker-leave-to {
  transform: translateY(100%);
}
```
这里有一点，`v-enter`变为了`v-enter-from`了。

到目前位置，我们的插件就完成了，接下来就可以试试发布到`npm`啦。

## 发布到NPM

哈哈，完成了一个小插件后，就可以发布到`npm`里面存起来。当然，发布的方式也很简单：  
1. 把源换成官方的源：  
    我这里使用的是`nrm`，所以直接`nrm use npm`即可；  
    
2. 登录`npm`：  
    `npm login`，按提示输入账号，密码，邮箱即可；
    
3. 发布包：
    在发布前，去[npm](https://www.npmjs.com/)官网查看有没有重复的包名（重复会发布不成功），包名合法的话，然后`npm publish`即可成功发布。

然后，等一下下，你就可以在[npm](https://www.npmjs.com/)的官网中看到自己的包啦；接着，你就可以通过`npm`的方式引用了：  
```js
npm isntall --save xxx`
```

```js
import xxx from 'xxx'
app.use(xxx)
```

## 总结

我这里只是完成了最简单的一个`Picker`插件，效果也并不完美，但是过程还是学到很多的；   
具体代码放在了`Git`仓库中：[Git仓库地址](https://github.com/Aizener/x-picker)；
哈哈哈，完成一个轮子的简单制作还是挺开心啦！  
