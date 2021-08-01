---
title: 使用CSS原子类来简化样式代码
date: 2021-07-31 16:17:20
tags:
  - CSS
categories:
  - Web前端
---

> CSS原子类并不是什么新的特性，也不是什么预编译SCSS或LESS的语法，而是一种写法，其特点就是一个类名对应一个样式，从而通过在标签上附加不同的类名来生成对应的效果，可以有效的减少CSS的相关代码。

### 了解
写CSS的方法有很多种，可以通过比较流行的`BEM`命名规范来写，可以通过`SCSS`或`LESS`等预编译编写，但也有一种当下比较火的写法：原子CSS。  
我们说了，CSS原子类只是一种特别的写法，那么我们就可以很轻松的实现一个原子类，例如：

```
.flex {
    display: flex;
}
```

上面的`.flex`就是一个原子类，它的特点就是**一个类名对应一个CSS规则，并且命名应该是和规则有关系的**，使用的话，直接在标签里插入样式：

```
<ul class="flex">
    <li>1</li>
    <li>2</li>
    <li>3</li>
</ul>
```

这样的话，`ul`这个标签就具有`flex`布局了。倘若我们组织的原子类足够多的话，我们就可以通过直接在标签里写原子类完成布局了：

```
.flex { diaplay: flex; }
.flex-col { flex-direcotrion: column; }
.flex-row-center { justify-content: center; }
.flex-col-center { align-items: center; }
.p-15 { padding: 15px; }
```

```
<ul class="flex flex-col flex-row-center flex-col-center p-15">
    <li>1</li>
    <li>2</li>
    <li>3</li>
</ul>
```
这就是CSS原子类的写法，非常的简单，因为一个类名对应一个规则，所以我们往往需要大量的工具类，这就造成了它特点。

### 坏处

CSS原子类还是有一些缺点的：

1.  因为可能定制了一大堆工具类，这些使用的话需要开发人员的熟悉；
2.  虽然减少了CSS的代码，但是使得HTML的代码变得更臃肿了；
3.  CSS 规则插入顺序仍然很重要。

### 好处

当然我认为原子类的好处还是大于坏处的：

1.  写法真的变简单了，比如添加一个常用的规则时，直接就加原子类，而不是定义一个类名，再去`style`里面写样式；
2.  因为原子类是定义的公共类，所以当我们移动一段标签到其他页面时，样式也就是有的，以前的话还需要拷贝样式，当然有组件概念后，这个到不是什么问题；
3.  当我们使用原子类的时候，我们修改页面的样式时往往不会再去修改`CSS`代码了，而是直接修改原子类名，这一点是不是和以前修改按钮时编写`btn-primary`一样？因为，已经有了基于原子类这种写法的`CSS`框架了，例如[tailwindcss](https://www.tailwindcss.cn/)。

### tailwindcss

`tailwindcss`是什么？摘自官网一段引用过：

> Tailwind CSS 是一个功能类优先的 CSS 框架，它集成了诸如 `flex`, `pt-4`, `text-center` 和 `rotate-90` 这样的的类，它们能直接在脚本标记语言中组合起来，构建出任何设计。

是不是就是CSS原子类的写法了？当然，`tailwindcss`不止于此，它还有很多强大的功能和配置，有兴趣的可以去看看，我这里就试一下其原子类的功能：

#### 安装

请查看：[tailwindcss的安装文档](https://www.tailwindcss.cn/docs/installation)

#### 使用

安装好之后，我们就可以在标签上通过原子类来编写样式了，但是有一个前提就是我们得熟悉`tailwindcss`的原子类规则了，这也就是前面说的一个缺点所在，我简单基于Vue写一个Demo：

![niced.gif](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d4525ad5bcd64fe4bb6be6e97f115606~tplv-k3u1fbpfcp-watermark.image)

上面效果，一行CSS都不写，完全依靠`tailwindcss`提供的工具类：
```html
<div id="app" class="flex">
    <div class="mx-auto flex p-2 w-full border border-solid border-gray-500 rounded">
      <img class="hidden sm:inline" alt="Vue logo" src="./assets/logo.png">
      <div class="ml-5 flex flex flex-col">
        <div class="flex flex-col items-center ss:flex-row ss:justify-between ss:items-baseline">
          <p class="text-xl">这是一篇文章的标题</p>
          <p class="text-xs">
            <span class="mr-5">作者：作者名</span>
            <span>2021-07-27</span>
          </p>
        </div>
        <div class="mt-2">
          这是文章的内容这是文章的内容这是文章的内容这是文章的内容这是文章的内容
          这是文章的内容这是文章的内容这是文章的内容这是文章的内容这是文章的内容
          这是文章的内容这是文章的内容这是文章的内容这是文章的内容这是文章的内容
          这是文章的内容这是文章的内容这是文章的内容这是文章的内容这是文章的内容
          这是文章的内容这是文章的内容这是文章的内容这是文章的内容这是文章的内容
          这是文章的内容这是文章的内容这是文章的内容这是文章的内容这是文章的内容
        </div>
      </div>
    </div>
</div>
```
### 自定义工具类
像`tailwindcss`这类框架，是非常的强大的，有许多的功能，而我们可能只想就用用一些常用的工具类的话，尽管`tailwindcss`在生产环境可以压缩优化，但是那一堆配置还是挺难记得。所以，我们可以自定义一些工具类，方便自己来使用，例如我就利用`SCSS`自定义了一些公共工具类：
```scss
ul { list-style: none; }
* { margin: 0; padding: 0; box-sizing: border-box; }

@for $i from 10 through 60 {
  .fs-#{$i} { font-size: $i * 1rpx; }
  .pl-#{$i} { padding-left: $i * 1rpx; }
  .pt-#{$i} { padding-top: $i * 1rpx; }
  .pr-#{$i} { padding-right: $i * 1rpx; }
  .pb-#{$i} { padding-bottom: $i * 1rpx; }
  .px-#{$i} { padding-left: $i * 1rpx; padding-right: $i * 1rpx; }
  .py-#{$i} { padding-top: $i * 1rpx; padding-bottom: $i * 1rpx; }
  .p-#{$i} { padding: $i * 1rpx; }
  .ml-#{$i} { margin-left: $i * 1rpx; }
  .mt-#{$i} { margin-top: $i * 1rpx; }
  .mr-#{$i} { margin-right: $i * 1rpx; }
  .mb-#{$i} { margin-bottom: $i * 1rpx; }
  .mx-#{$i} { margin-left: $i * 1rpx; margin-right: $i * 1rpx; }
  .my-#{$i} { margin-top: $i * 1rpx; margin-bottom: $i * 1rpx; }
  .m-#{$i} { margin: $i * 1rpx; }
}

$colors: #000, #333, #666, #999, #ccc, #ddd, #eee, #fff, #1E90FF, #00BFFF	;
$colorName: '000', '333', '666', '999', 'ccc', 'ddd', 'eee', 'fff', '0ff', 'bff';

@for $i from 1 through length($colors) {
  $color: nth($colors, $i);
  $name: nth($colorName, $i);
  .color-#{$name} { color: $color; }
  .bg-#{$name} { background-color: $color;}
  .bc-#{$name} { border-color: $color !important;}
}

.flex { display: flex; }
.flex-1 { flex: 1; }
.flex-2 { flex: 2; }
.flex-3 { flex: 3; }
.flex-col { flex-direction: column; }
.flex-row-start { justify-content: flex-start; }
.flex-row-center { justify-content: center; }
.flex-row-end { justify-content: flex-end; }
.flex-row-between { justify-content: space-between; }
.flex-col-start { align-items: flex-start; }
.flex-col-center { align-items: center; }
.flex-col-end { align-items: flex-end; }
.flex-col-baseline { align-items: baseline; }

.p-relative { position: relative; }
.p-absolute { position: absolute; }
.p-fixed { position: fixed; }
.l-0 { left: 0; }
.t-0 { top: 0; }
.r-0 { right:0; }
.b-0 { bottom: 0; }
.w-100 { width: 100%; }
.w-100i { width: 100% !important; }
.h-100 { height: 100%; }
.h-100i { height: 100% !important; }
.vh-100 { height: 100vh; }
.fw-bold { font-weight: bold; }

.mx-auto { margin-left: auto; margin-right: auto; }

.bb-1 { border-bottom: 1px solid; }
.bt-1 { border-top: 1px solid; }
.scroll-y { overflow-y: auto; }
.scroll-x { overflow-x: auto; }

.z-1 { z-index: 1; }
.z-2 { z-index: 2; }
.z-3 { z-index: 3; }
```
这样，一来是我们自己定义的好记，而来体积也并不大，方便开发。我上面定义的类，不全是CSS原子类，像`.bb-1`这种我就是组合的，这样也可以在标签里少写一点。

### 总结

已有前辈总结其特点：

> 需要**预先开发好**一个不错的实用工具/原子样式表，然后才能开始开发新功能。  
>如果实用工具/原子 CSS 是由别人制作的，你将不得不首先学习类命名约定(即使你知道 CSS 的一切)。这种约定是**有主观性**的，很可能你不喜欢它。  
>有时，你需要一些**额外的 CSS**，而实用工具/原子 CSS 并不提供这些 CSS。没有约定好的方法来提供这些一次性样式。

个人认为的话，CSS原子类和以前在`style`里面写样式并不冲突，可能因为我的项目都是一个人，所以自己也没有啥规范约束，自己两者都混合用，开发起来，觉得真的挺方便的。  

这个写法最初还是一个姑娘介绍给我的，当时我还觉得烦，现在好想念。