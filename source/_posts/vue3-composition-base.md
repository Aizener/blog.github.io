---
title: Vue3的Composition简介
tags:
  - Vue
categories:
  - Web前端
---

> 这是一份个人学习Vue3的笔记整理，不会去记录某些Api的原理，只记录这些API的一些基本用法，有写错的地方，希望包涵指正。

### 安装

要体验Vue3的话，需要安装最新版的VueCLi（当然使用Vite也行），安装之后，通过`vue create demo`即可选择创建Vue3的项目。

### 组合式API

CompositionAPI（组合式API）是Vue3新的写法，从官方文档可以看到它的特点：**能够将同一个逻辑关注点相关的代码配置在一起，使得应用程序在可维护性和灵活性方面走得更远。**  

使用：

组合式API必须使用在`setup`函数里面，即`setup`函数是组合式API的入口函数。

### setup

setup函数的第一个特点就是：由于`setup`在组件实例前先执行，所以除了`props`以外无法访问`this`，`computed`,`methods`等等，但作为代替我们可以以其他的书写方式来实现：

```js
setup (props) {
    function greet () {
        console.log('Hellol')
    }
    watch(() => props.msg, (newVal, oldVal) => {
        console.log(newVal, oldVal)
    })
    const rMsg = computed(() => {
        return props.msg.split('').reverse().join('')
    })

    return {
        greet,
        rMsg
    }
}
```

可以看到，在`setup`里面通过组合式API添加了greet方法，监听了`props`的`msg`属性，以及添加了一个计算属性rMsg，这些新加的属性或方法必须返回之后才能在`template`正常使用。

在`setup`里面使用`props`的时候，依然需要定义`props`对象，来规定传递过来的属性名称和默认值。

### reactive

`reactive`是组合式API里面的响应式API，通过`reactive`可以返回一个具有响应式的副本对象（所有嵌套的对象都会被代理）：

```js
const state = reactive({
    msg: 'Hello.'
})
function reverseMsg () {
    state.msg = state.msg.split('').reverse().join('')
}
return {
    state,
    reverseMsg
}
```

此时，如果执行`reverseMsg`方法的话，模板里含有的`state.msg`的UI也会跟着刷新，但是通过`reactive`定义的对象副本有两个特点：

1. 必须是一个对象，否则UI不会更新，例如：num = reactive(0)，这种是不会更新UI的；
2. 里面定义的其他对象要修改时，必须重新赋值，否则也不会刷新UI界面，例如Date对象，不能通过setDate这样的API来刷新UI界面。

### readonly

`readonly`类似于`reactive`，只是它定义的副本是只读的，类似`const`定义变量那种，他的所有嵌套也是只读的。

```js
 const state = readonly({
     msg: 'Hello.'
 })
 function reverseMsg () {
     state.msg = state.msg.split('').reverse().join('')
 }
```

上面代码会有警告：`Set operation on key "msg" failed: target is readonly. {msg: "Hello."}`，并且所做的修改时无效的。

### shallowRecative

和`reactive`一样是创建一个响应式对象副本的，区别就是它只是第一层做了代理，里面的对象不做递归代理。

```js
const state = reactive({
    msg: 'Hello.',
    info: {
        title: 'vue3'
    }
})
console.log(state, state.info)
```

这是由`reactive`定义的对象副本，打印出来的是`Proxy {msg: "Hello.", info: {…}} Proxy {title: "vue3"}`，可以看到`info`也是被`Proxy`代理了的，再看看`shallowReactive`创建的对象副本：

```js
const state = shallowReactive({
    msg: 'Hello.',
    info: {
        title: 'vue3'
    }
})
console.log(state, state.info)
```

打印出来的是：`Proxy {msg: "Hello.", info: {…}} {title: "vue3"}`。

**他们两个的区别就是，因为`reactive`是递归来代理了所有对象的，所以不管修改外面的对象还是里面的对象，也会让UI刷新；而`shallowReactive`只有修改最外层对象时，才会做UI刷新。**

```js
const state = shallowReactive({
    msg: 'Hello.',
    info: {
        title: 'vue3'
    }
})
function reverseMsg () {
    // 因为是shallowReactive定义的，所以不能直接通过修改info面的title来刷新UI，必须得重新引用一个对象副本
    state.info = { title: state.info.title.split('').reverse().join('') }
    // reactive定义的对象副本，则可以通过这样修改值来刷新UI
    // state.info.title = state.info.title.split('').reverse().join('')
}
```

### shallowReadonly

标记自身的对象为可读，内部的对象则可以做修改，这是官方例子：

```js
const state = shallowReadonly({
  foo: 1,
  nested: {
    bar: 2
  }
})

// 改变状态本身的property将失败
state.foo++
// ...但适用于嵌套对象
isReadonly(state.nested) // false
state.nested.bar++ // 适用
```

### ref

`ref`和`reactive`类似，但是它可以接收基本变量，并且使用的时候需要加上`.value`。

```js
const name = ref('lihua')
console.log(name.value)	// lihua
```

`ref`定义的属性还有一个特点，就是在`template`里面使用的话，不需要加上`.value`，模板会自动解析出对应的`value`值。

### toRef

可以将源对象的某个属性创建一个对应的响应式副本`ref`并返回回去：

```js
const msg = toRef(state, 'msg')
msg.value = '.olleH'
console.log(state.msg)	// .olleH
```

可以看到，创建的副本修改的值会影响到原来的`reactive`创建的对象，并且也会刷新UI。

### toRefs

可以将源对象创建为一个普通对象，并且里面的属性都通过了`ref`的包装，来指向原对象。

```js
const state = reactive({
    msg: 'Hello.',
    info: {
        title: 'vue3'
    }
})

const state = toRefs(state)
console.log(state)
```

可以看到打印出：`{msg: ObjectRefImpl, info: ObjectRefImpl}`。这个API可以在解构等方面的时候起到非常有用的地方：

```js
// 这种用法，在template的话需要通过state.msg来使用
return {
    state
}
// 这样使用扩展运算符的话，基本属性的修改会无效
return {
    ...state
}
// 通过toRefs来使用的话，因为基本属性也通过ref来包装了一层，所以可以直接使用msg
return {
    ...toRefs(state)
}
```

在传递`props`的时候也非常有用：

```js
// 某个子组件，接收了父组件传过来的title
const myTitle = props.title
watch(() => props.title, newVal => {
    console.log(newVal)
})
return {
    myTitle
}
```

上述代码，父组件修改`title`时，子组件是会监听到`title`的变化的，但是`myTitle`不会改变，UI并不会刷新（可是我看文档上说的是，通过`.`的方式也是响应式的额，奇怪了！）。

改成使用`toRefs`的方式：

```js
const { title: myTitle } = toRefs(props)
watch(() => props.title, newVal => {
    console.log(myTitle.value, newVal)
})
return {
    myTitle
}
```

这样的话，当修改父组件的`title`时，子组件也会跟着刷新UI，但是使用的时候要记得加`.value`，模板上不加。

### 路由

在Vue3里面使用路由的话，需要安装最新版本的（4.x)：

```shell
yarn add vue-router@next
npm isntall --save vue-router@next
```

官方例子：

```js
import { createRouter, createWebHashHistory } from 'vue-router'
import Home from '../views/Home.vue'

const routes = [
  {
    path: '/',
    name: 'Home',
    component: Home
  },
  {
    path: '/about',
    name: 'About',
    // route level code-splitting
    // this generates a separate chunk (about.[hash].js) for this route
    // which is lazy-loaded when the route is visited.
    component: () => import(/* webpackChunkName: "about" */ '../views/About.vue')
  }
]

const router = createRouter({
  history: createWebHashHistory(),
  routes
})
```

除了通过`new`改成了函数创建路由的方式，其他的配置还是没什么变化，如果要在`setup`里面使用路由对象的话，需要单独引入：

```js
import { useRouter, useRoute } from 'vue-router'
setup () {
    const router = useRouter()
    const route = useRoute()
}
```

```js
// main.js
import router from '@/src/router/index'
...
app.use(router)
```



其他用法可以参考官方的文档：[导航守卫](https://next.router.vuejs.org/zh/guide/advanced/navigation-guards.html#%E5%8F%AF%E7%94%A8%E7%9A%84%E9%85%8D%E7%BD%AE-api)和[Vue Router 和 组合式 API](https://next.router.vuejs.org/zh/guide/advanced/composition-api.html)。

### Vuex

和路由一样，Vuex需要安装最新版本的：

```shell
yarn add vuex@next
npm install --save vuex@next
```

官方例子：

```js
import { createStore } from 'vuex'

export default createStore({
  state: {
  },
  mutations: {
  },
  actions: {
  },
  modules: {
  }
})
```

```js
// main.js
import store from '@/src/store/index'
...
app.use(store)
```

在`setup`里面使用的话，也需要单独引入（我没找到其他方法o(╥﹏╥)o）。

**Over,暂时就记录到这儿吧，有问题还请指正，Tanks.**