---
title: 基于Vue对主题修改的思考与实现
date: 2021-07-31 16:16:15
tags:
  - Vue
categories:
  - Web前端
---
> 主题更改虽然不是一个常用的功能，但是我还是想了解一下这个东西的思路和实现，个人思考之后，觉得就是动态绑定style的方法比较容易，特此记录。

先看看效果：  

![nicec.gif](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0117af83c3a0477e8c0205677854244a~tplv-k3u1fbpfcp-watermark.image)
### CSS函数之var

在敲代码之前，先说一下`var`函数。

`var`函数作为`css`的新特性之一，是用于插入自定义的属性值；

如果一个属性值在多处被使用，该方法就很有用，看一下demo：

```
:root {
  --main-bg-color: coral;
}
 
#div1 {
  background-color: var(--main-bg-color);
}
 
#div2 {
  background-color: var(--main-bg-color);
}
```

根据这个`var`函数，我们就可以事先定义一堆属性，然后给需要修改主题的地方赋值，这样就很方便了。

但是这个属性并不兼容`IE`等一些小众浏览器，看下图：

![error](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/909377d3853d4a408d007b74a2e3cd43~tplv-k3u1fbpfcp-watermark.image)


所以考虑兼容性的话，就不能使用`var`函数了，而是一个地方一个地方动态设置样式。

### 配置主题

主题应该是有一个或多个配置文件来定义的，可以是JS文件导出，也可以是JSON文件，当然也可以从后端请求回来：

```js
export default {
  default: {
    'link-color': 'green',
    'text-color': 'black',
    'body-bg': '#fff',
    'logo-filter': 'brightness(1)'
  },
  dark: {
    'link-color': '#fff',
    'text-color': 'orange',
    'body-bg': '#666',
    'logo-filter': 'brightness(2)'
  },
  light: {
    'link-color': 'skyblue',
    'text-color': 'green',
    'body-bg': 'aliceblue',
    'logo-filter': 'brightness(3)'
  }
}
```

这是我随便定义的一些主题色（很丑噶），我的思路就是每一个`Key`就是主题的名称，通过修改`Key`来得到不同的主题状态，从而更换网站的主题色。

因为主题基本都是影响多个页面的，所以这里我们需要引入`Vuex`来保存当前主题状态。

### 引入Vuex

引入`Vuex`后，我们需要设置一个`theme`属性来表示当前的主题状态，同样的会提供对应的`mutations`和`actions`里面的方法来修改这个`theme`属性。

```js
import Vue from 'vue'
import Vuex from 'vuex'
import themes from '../themes/index'
​
Vue.use(Vuex)
​
export default new Vuex.Store({
  state: {
    theme: 'default'
  },
  getters: {
    currTheme(state) {
      return themes[state.theme]
    }
  },
  mutations: {
    UPDATE_THEME(state, theme) {
      state.theme = theme
    }
  },
  actions: {
    updateTheme({ commit }, theme) {
      commit('UPDATE_THEME', theme)
    }
  }
})
​
```

上面还可以看到，我们还使用了`getters`这个对象，使用这个对象可以很方便的依赖`state`的变化而更新对应的主题状态，而我们只管修改`state`的属性即可。

### 设置CSS变量

因为我们要设置CSS的变量供其他页面使用`var`函数调用，所以一般设置在顶层组件，我们可以再`App.vue`里面设置：
```js
<template>
  <div id="app" :style="themeStyle">
    <img alt="Vue logo" src="./assets/logo.png">
    <HelloWorld msg="Welcome to Your Vue.js App"/>
  </div>
</template>
​
<script>
import HelloWorld from './components/HelloWorld.vue'
import { mapActions, mapGetters } from 'vuex'
import themes from './themes/index'
​
export default {
  name: 'App',
  components: {
    HelloWorld
  },
  computed: {
    ...mapGetters(['currTheme']),
    themeStyle() {
      const themeStyle = {}
      for (const key in this.currTheme) {
        themeStyle['--theme-' + key] = this.currTheme[key]
      }
      return themeStyle
    }
  },
  methods: {
    ...mapActions(['updateTheme']),
    handleChange(e) {
      this.updateTheme(e.currentTarget.value)
    }
  }
}
</script>
```
通过样式动态绑定的方式，我们就给`div#app`标签绑定上了CSS变量，可以看到如下图：


![error](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/00828da640354dc4a9fb4d8dabd7b16e~tplv-k3u1fbpfcp-watermark.image)

接着，要在需要随主题变化的元素设置对应主题色就行了，例如：
```css
p, h1, h2, h3 {
  color: var(--theme-text-color);
}
```
如果，考虑兼容性的话，就通过动态绑定吧，这样会繁琐一些。

### 主题切换

此时，我们只需要提供修改`state`的`theme`属性方式，就可以完成主题的修改啦，例如通过`select`来选择后修改`theme`属性：

```js
<template>
  <div id="app" :style="themeStyle">
    <img alt="Vue logo" src="./assets/logo.png">
    <HelloWorld msg="Welcome to Your Vue.js App"/>
    <div class="switch">
      切换主题：
      <!-- 可以通过遍历来生成 -->
      <select @change="handleChange">
        <option value="default">default</option>
        <option value="dark">dark</option>
        <option value="light">light</option>
      </select>
    </div>
  </div>
</template>
​
<script>
import HelloWorld from './components/HelloWorld.vue'
import { mapActions, mapGetters } from 'vuex'
​
export default {
  name: 'App',
  components: {
    HelloWorld
  },
  computed: {
    ...mapGetters(['currTheme']),
    themeStyle() {
      const themeStyle = {}
      for (const key in this.currTheme) {
        themeStyle['--theme-' + key] = this.currTheme[key]
      }
      return themeStyle
    }
  },
  methods: {
    ...mapActions(['updateTheme']),
    handleChange(e) {
      this.updateTheme(e.currentTarget.value)
    }
  }
}
</script>
​
```
### 总结
修改主题的方式还有其他的，比如通过`less`的`modifyVars`也可以完成（没看懂怎么费事），但是我觉得动态绑定`style`的方式比较好理解，不考虑兼容`ie`使用`var`函数还是很方便的，倘若考虑兼容就得累一点一个个动态绑定了。