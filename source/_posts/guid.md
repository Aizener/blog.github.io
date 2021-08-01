---
title: 基于Vue实现页面的引导操作
date: 2021-07-31 16:17:15
tags:
  - Vue
categories:
  - Web前端
---
> 引导操作是一个非必须的功能，但是有这个功能的话，对第一次进入系统的人来说，是一个比较友好的体验，能快速的介绍自己系统的一些简单功能，我们可以基于[driver.js](https://github.com/kamranahmedse/driver.js)很轻松的实现这个效果。

**看看效果**

![图片丢失](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8f7c9bb3c2014cbc97bc7d3ca40bbfaa~tplv-k3u1fbpfcp-zoom-1.image)

## 安装

首先，我们需要通过`npm`或者`yarn`安装`driver.js`：

**npm**

`npm install --save driver.js`

**yarn**

`yarn add driver.js`

## 开始

因为`driver.js`是需要操作`dom`节点的，所以使用的时候需要等到节点渲染完成，需要在`onMounted`里面使用：

```
setup() {
    const driver = new Driver()
    const show = () => {
        driver.highlight(
            {
                element: '#logo',
                popover: {
                    title: 'Title for the Popover',
                    description: 'Description for it',
                    position: 'bottom',
                    offset: 20,
                }
            }
        )
    }
​
    onMounted(() => {
        setTimeout(() => {
            show()
        })
    })
}
```

**这里我也不知道为啥，在`onMounted`里面直接调用`driver.js`的方法没用，甚至加了`nextTick`也不会有效果，只有在`setTimeout`里面才会有效果，有大佬指教？**

`highlight`就是`driver.js`提供的高亮效果，但是我们的引导操作并会只有一个，一般会有许多个这样的操作，所以这样一个一个设置就不大现实了。

`driver.js`提供了`defineSteps`这个方法，来实现一系列的引导操作：

```
setup() {
    const driver = new Driver()
    const show = () => {
      driver.defineSteps([
        {
          element: '#logo',
          popover: {
            title: '图标',
            description: '这是Vue的图标',
            position: 'bottom',
            offset: 20,
          }
        }
      ])
​
      driver.start()
    }
}
```

通过`defineSteps`定义之后，需要使用`start`方法来开始引导。

## 公共配置

我们可以在实例化`Driver`的时候，传入一些公共配置，这样的话，就很方便了，例如下一步，上一步的文案原来是英文的，我们可以在实例化时直接传入：

```
setup() {
    const driver = new Driver({
      doneBtnText: '完成',
      closeBtnText: '关闭',
      nextBtnText: '下一步',
      prevBtnText: '上一步'
    })
    const show = () => {
      driver.defineSteps([
        {
          element: '#logo',
          popover: {
            title: '图标',
            description: '这是Vue的图标',
            position: 'bottom',
            offset: 20
          }
        }
      ])
​
      driver.start()
    }
}
```

当然，这个配置也***可以在`options`里面覆盖掉***，`driver.js`还提供了许多的内置配置：

```
const driver = new Driver({
  className: 'scoped-class',        // className to wrap driver.js popover
  animate: true,                    // Whether to animate or not
  opacity: 0.75,                    // Background opacity (0 means only popovers and without overlay)
  padding: 10,                      // Distance of element from around the edges
  allowClose: true,                 // Whether the click on overlay should close or not
  overlayClickNext: false,          // Whether the click on overlay should move next
  doneBtnText: 'Done',              // Text on the final button
  closeBtnText: 'Close',            // Text on the close button for this step
  stageBackground: '#ffffff',       // Background color for the staged behind highlighted element
  nextBtnText: 'Next',              // Next button text for this step
  prevBtnText: 'Previous',          // Previous button text for this step
  showButtons: false,               // Do not show control buttons in footer
  keyboardControl: true,            // Allow controlling through keyboard (escape to close, arrow keys to move)
  scrollIntoViewOptions: {},        // We use `scrollIntoView()` when possible, pass here the options for it if you want any
  onHighlightStarted: (Element) => {}, // Called when element is about to be highlighted
  onHighlighted: (Element) => {},      // Called when element is fully highlighted
  onDeselected: (Element) => {},       // Called when element has been deselected
  onReset: (Element) => {},            // Called when overlay is about to be cleared
  onNext: (Element) => {},                    // Called when moving to next step on any step
  onPrevious: (Element) => {},                // Called when moving to previous step on any step
});
```

***`driver.js`生成的`popover`，会根据目标节点的大小生成的，所以如果`popover`过不合适的话，可以看看原来的`dom`节点是什么样子的。***

## 内置方法和属性

`driver.js`还内置了一些比较方便的方法，像`defineSteps`和`start`就是内置的方法：

-   isActivated：是否正在引导过程中；

<!---->

-   defineSteps： 定义引导步骤；
-   start(step = 0)：开始引导，可以传入一个`number`参数，表示从第几步开始，比如步骤过多的情况，我们可以做一个记录，下次用户点进来就可以从中间开始查看引导了。
-   moveNext：移动到下一个步骤的引导；
-   movePrevious：移动到上一个步骤的引导；
-   hasNextStep：是否有下一个步骤的引导；
-   hasPreviousStep：是否有上一个步骤的引导；

其他的还有很多方法，可以查看官网