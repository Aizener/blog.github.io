---
title: 关于H5新特性拖放API讲解
date: 2021-08-01 21:58:15
tags:
  - HTML5
  - JavaScript
categories:
  - Web前端
---

> HTML5出来了许多的新特性，拖放就是其中比较有用的特性之一，可以应用到许多业务场景中去，通过这个新特性，我们也可以实现许多有趣的例子。

先看一个通过拖放实现的小Demo：

![nicef.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/225149073fd14048aa2914014049e538~tplv-k3u1fbpfcp-watermark.image)

## 概念
拖放是H5的新特性，即抓取对象以后拖到另一个位置，但是前提是这个元素设置了属性`draggable=true`，才能让该元素具有拖放功能。

拖放具有三个过程，即：  
`开始拖动` -> `拖动中` -> `放置元素`  
- 开始拖动：指触发拖动的元素的过程，比如轻按元素时；
- 拖动中：移动拖动中的元素；
- 放置元素：把拖动的元素，放置到另一个元素里面的过程。

在不同的阶段，都有其相对应的`API`，下面我们就看看这些`API`。

## 事件
H5的拖放有如下6个`API`供使用：  
- `ondragstart`
- `ondrag`
- `ondragend`
- `ondragenter`
- `ondragover`
- `ondrop`
上面可以看到我写这些`API`分两种情况，前三个是**拖动的元素对应的API**，后三个是**放置的元素的API**。  

例如：我要把`div#box1`拖动到`div#box2`，那么`div#box1`就该监听`dragstart`和`dragend`，而`div#box2`就监听另外三个。  

|对象|事件|描述|
|:-:|:-:|:-:|
|拖动的元素|dragstart|触发拖动时就会执行|
||drag|触发拖动后拖动元素过程触发|
||dragend|拖动结束后触发|
|放置的元素|dragenter|当拖动的元素进入绑定的元素区域里就会触发|
||dragover|当拖动的元素在放置的元素区域里移动就会触发|
||drop|当拖动的元素在放置的元素区域里释放就会触发|

**注意：`drop`事件在一些浏览器上可能不会触发，这时需要在`dragover`和`drop`事件里面阻止浏览器默认事件：**
```js
addEventListener('dragover', e => {
    e.preventDfault()
})
addEventListener('drop', e => {
    e.preventDfault()
})
```

**另外，拖动移入移出的判定是通过鼠标的位置，而不是拖动的元素的位置。**

### 事件顺序：

- 拖动时（未进入放置元素区域）：
`ondragstart` -> `ondrag` -> `ondragend`
- 拖动时（进入放置元素区域）：
`ondragstart` -> `ondragenter` -> `ondragover` -> `ondrag` -> `ondrop` -> `ondragend`

## 数据传递
元素拖放的过程，往往是需要数据传递的，比如放置时要获取拖动元素的状态，其中就可以通过`dataTransfer`这个对象来传递数据。

通过在`dragstart`开始阶段设置状态，然后再`drop`时获取状态，再进行其他操作。

```js
addEventListener('dragstart', e => {
    e.dataTransfer.setData('msg', 'hello')
    // 传递对象
    // e.dataTransfer.setData('greet', JSON.stringify({ msg1: 'hello', msg2: 'good morning' }))
})
addEventListener('drop', e => {
    const msg = e.dataTransfer.getData('msg')
    // TODO
    e.preventDfault()
})
```

注意，这里的`setData`只能设置字符串，如果是对象的话，请通过`JSON.stringify`来序列化。

## 兼容性

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e303034363424ab680e1c57353c69077~tplv-k3u1fbpfcp-watermark.image)

兼容性还是存在一些问题，具体可以看看[caniuse](https://www.caniuse.com/?search=drag)。

## 总结
如果觉得写原生拖动麻烦，这里还推荐再次封装过的[sortable.js](https://github.com/SortableJS/Sortable)库，功能强大，使用也不难；  


拖动这个特性的应用还是比较广泛的，但因为是`H5`新特性，兼容在一些浏览器上还是存在一定问题，所以依据项目情况使用。  

这里也介绍的全是基础用法，详细的可以去看看[MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/API/HTML_Drag_and_Drop_API)。

