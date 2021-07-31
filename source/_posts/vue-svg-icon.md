---
title: 基于Vue实现自定义的图标组件
tags:
  - Vue
  - icon
categories:
  - Web前端
---

> 在项目开发中，使用图标的方式有很多种，可以在[iconfont](https://www.iconfont.cn/)上面找到合适的图标，通过http或者直接下载使用，这里我分享一种通过实现自定义组件的方式引入图标。

### 搭建环境

这里通过@vue/cli 4.5.13新建项目，并且需要安装依赖`svg-sprite-loader`，用来处理对应的`svg`图标，方便我们使用。

安装： `npm install --save-dev svg-sprite-loader`

#### 配置vue.config.js

在安装`svg-sprite-loader`后，新建`vue.config.js`来配置依赖：

```js
// vue.config.js
const { resolve } = require('path')

module.exports = {
  chainWebpack(config) {
    config
      .module
      .rule('svg')
      .exclude
      .add(resolve('src/icons'))
      .end()
    config
      .module
      .rule('icons')
      .test(/\.svg$/)
      .include
      .add(resolve('src/icons'))
      .end()
      .use('svg-sprite-loader')
      .loader('svg-sprite-loader')
      .options({
        symbolId: 'icon-[name]'
      })
  }
}
```

这里通过`chainWebpack`来做了两项配置：

1. 第一个是让原来的其他处理`svg`的依赖不处理`src/icons`下我们的自定义图标文件

2. 通过`svg-sprite-loader`来处理自定义的图标文件，`options`里面的设置表示，生成的`svg`的`symbolId`为icon和文件名的拼接。

### 新建图标组件

在components目录下新建一个`SvgIcon.vue`文件：

```vue
<template>
  <i class="icon">
    <!-- aria-hidden, 帮助残障人士阅读(设备读取内容时会跳过这个标签以免混淆) -->
    <svg aria-hidden="true" :width="size" :height="size" :fill="fillColor">
      <use :xlink:href="iconName"></use>
    </svg>
  </i>
</template>

<script lang="ts">
import { PropType, toRefs } from 'vue'

export default {
  props: {
    size: {
      type: Number as PropType<number>,
      default: 14
    },
    fillColor: {
      type: String as PropType<string>,
      default: '#000'
    },
    iconName: {
      type: String as PropType<string>,
      required: true
    }
  },
  setup(props: any) {
    const { size, fillColor, iconName: _iconName } = toRefs(props)
    const iconName = `#${_iconName.value}`

    return {
      size,
      fillColor,
      iconName
    }
  }
}
</script>
```

然后，新建一个`icons`目录，并且新建一个`index`文件，用来挂在组件和引入`svg`图标：

```typescript
// index.ts
import SvgIcon from '@/components/SvgIcon.vue'
import { App } from 'vue'

export default (app: App) => {
  app.component('svg-icon', SvgIcon)
}

const ctx = require.context('./svg', false, /\.svg$/)
const requestAll = (ctx: __WebpackModuleApi.RequireContext) => ctx.keys().forEach(ctx)

requestAll(ctx)
```

```typescript
// main.ts
import { createApp } from 'vue'
import App from './App.vue'
import installSvgIcon from '@/icons/index'

const app = createApp(App)

installSvgIcon(app)
app.mount('#app')
```



这个文件做了两件事：

1. 通过导出一个方法来挂载全局组件`svg-icon`；
2. 通过`require.context`来实现自动化引入`svg`目录下的图标文件。

### 使用组件

首先，我们要在`icons/svg`目录下存放`svg`图标文件（在[iconfont](https://www.iconfont.cn/)上找自己需要的）；

然后，就可以在其他地方使用啦：

```vue
<template>
  <img alt="Vue logo" src="./assets/logo.png">
  <svg-icon icon-name="icon-dashboard"></svg-icon>
  <HelloWorld msg="Welcome to Your Vue.js + TypeScript App"/>
</template>
```

直接通过组件`svg-icon`的方式引入，然后传入`icon-name`即可，`icon-name`的值由icon拼接svg文件名组成。

### 总结

这种自定义方式引入`svg`图标的方式，还是挺方便的，想要加上一个图标的时候，几步即可：

1. 直接下载好`svg`文件放入对应目录中；
2. 接着通过`svg-icon`组件来引入。

***但是，修改样式时不能通过`css`来修改，这点就不太方便了。***

