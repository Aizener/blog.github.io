---
title: 关于移动端适配的实现
tags:
  - CSS
  - 布局
categories:
  - Web前端
---

> 平时在PC端，我们一般做的可能就是固定一个中间宽度，然后用媒介查询去做响应式之类的操作。但是，在移动端设备上，由于市面上机型的屏幕宽度非常多，我这里就总结一下适配的方案。

移动端适配，就是在不同宽度设备的屏幕上，显示正常，而不会漏出空白或溢出等，看看效果：

![nicee.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/488979eb67c44c949c3904bae1b1e6b3~tplv-k3u1fbpfcp-watermark.image)


可以看到上图，图片和标题都是随着屏幕的大小改变的，适配的方式主要有两种。

### Rem布局

`rem`布局主要是利用`1rem`等于根节点大小来实现的，通过屏幕宽度来动态改变根字体大小，从而实现不同屏幕的适配：

假设我们根据设计宽度为`375px`来，可以得出`1rem`对应的根字体大小应为：

```css
html { font-size: 1px; }
body { font-size: 16px; }   // 因为html字体会影响其他的，所以body重置为默认大小
```

这样，`1vw`就对应`1px`大小了，但是`rem`的大小并不会随着屏幕变化，所以我们就得要根据屏幕宽度来改变了，有两种方式：第一种通过`js`来动态改变；这里，我介绍另一种，通过`vw`来改变：

```css
html { font-size: calc(100vw / 375); }
body { margin: 0; font-size: 16px; }
```
当然，比例可以设置不同的，按照自己的习惯可以设置为其他的大小。

其原理就是：需要根字体大小 / 设计稿的宽度 = **需要vw大小** / 100vw。

### vw布局

`vw`是根据视口来计算宽度的，一个`vw`就表示视口的1/100，如果是`100vw`则表示为整个视口宽度，利用`vw`属性和`scss`预编译，就可以很简单实现移动端适配布局：

```css
@width: 375;    // 设计稿的宽度
@function vw($px) {
    return ($px / $width) * 100vw;
}
```

这样就通过`scss`实现了一个可以适配的函数，通过调用这个函数，我们就可以根据不同的屏幕宽度来获得不同的单位：

```css
img {
    width: vw(375); // 在375px的屏幕下，就表示宽度为375
}
```

其原理就是：设计稿目标的宽度 / 设计稿的宽度 = **需要vw大小** / 100vw。

### postcss-plugin-px2rem

通过上面的`vw`布局和`rem`布局，我们能实现移动端的适配，但是功能还不够强大，通过`postcss`的插件，我们可以配置更多选项的适配。

#### 安装

-   npm：`npm i --save postcss-plugin-px2rem`

<!---->

-   yarn：`yarn add postcss-plugin-px2rem`

#### 使用

基于Vue可以在`vue.config.js`或者新建`postcss.config.js`文件配置`px2rem`插件：

```js
module.exports = {
  plugins: [require('postcss-plugin-px2rem')({
    rootValue: 100,
    exclude: /(node_module)/,
    mediaQuery: false
  })]
}
```

这里的`rootValue`表示`1px`等于`1/100rem`大小，当然也可以使用对象，例如：`{ px: 50, rpx: 100 }`；

`exclude`表示排除的文件（这些文件不会把`px`单位编译为`rem`）；

`mediaQuery`表示排除媒介查询的样式（这些文件不会把`px`单位编译为`rem`）。

配置完成之后，我们还是得设置HTML的根字体大小，这里因为`rootValue`是`100`，假设设计稿是`750px`大小的话，设置如下：

```css
html { font-size: calc(100 * 100vw / 750); }
body { font-size: 16px; }
```

除了可以转换`px`外，我们还可以使用之前说的给`rootValue`配置为对象的方式，来设置成小程序那种`rpx`的布局方式：

```js
module.exports = {
  plugins: [require('postcss-plugin-px2rem')({
    rootValue: {
        px: 50,
        rpx: 100
    },
    exclude: /(node_module)/,
    mediaQuery: false
  })]
}
```

```css
html { font-size: calc(100 * 100vw / 750); }
body { font-size: 16px; }
​
img {
    width: 750rpx; // 等同于 375px
}
```

这样的话，就可以使用`rpx`来像小程序那样完成布局了，更多配置可以看[postcss-plugin-px2rem](https://github.com/pigcan/postcss-plugin-px2rem#readme)。

### 总结

适配的方案无非就是`rem`和`vw`两种方式，主要是换算公式可能比较麻烦，甚至有些时候难以理解，个人觉得多上手试试，慢慢还是能理解的。