---
title: 来弄一个Github的个人介绍页吧~
date: 2021-07-31 16:12:15
tags:
  - Github
categories:
  - Git
---
> 现在GitHub可以添加个人介绍页了，这里小小的分享一下吧~
先来看看效果是啥样：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0edd62cedc6b43b48ef8cc85ad2e59d5~tplv-k3u1fbpfcp-watermark.image)

**是不是感觉效果还不错？实现起来还是非常简单的，开搞！**

### 新建仓库
首先，需要新建一个和自己名字一样的仓库，名字一样的话会有一个小提示哦 o(*￣︶￣*)o
![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bcd5e779ccde476a922897f09cc90fc2~tplv-k3u1fbpfcp-watermark.image)

### 自我介绍

建好仓库后，就把仓库Clone下来，然后新建一个`README.md`文件，在里面写自我介绍；   
其实，Github就是把你同名的仓库的`README.md`文件表现现实在首页而已，很好理解的。

自我介绍咋个写我就不多说了，这里说两个东西：  

- Github Stats Card：就是我截图里面的一个统计卡片；
- Emoji： 就是一些可以放在`Markdown`文件的表情。
#### Github Stats Card
这个非常简单，贴上[Github仓库](https://github.com/anuraghazra/github-readme-stats)，使用非常的简单，只需要引入下面代码：
```js
[![Anurag's github stats](//p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1188110e11c84be89e6c60130622635a~tplv-k3u1fbpfcp-zoom-1.image)](https://github.com/anuraghazra/github-readme-stats)
```
下面就是代码实现的一个Demo：  

![Anurag's github stats](https://github-readme-stats.vercel.app/api?username=anuraghazra&count_private=true)

只需要吧`username`改成自己的Github用户名即可，如果不喜欢这个默认的主题，还可以自由配置主题。
#### Emoji
这个玩意儿也非常的实用，可以让你很轻松的嵌入表情，进入[Emoji官网](https://www.webfx.com/tools/emoji-cheat-sheet/)选择喜欢的表情代码，复制粘贴即可。  

例如：微笑 -> :smile:  (滑稽，掘金的`Markdown`编辑器好像解析不出来，我用Typora是可以的)

**以上就是如何弄一个Github的个人介绍页，喜欢的点点赞吧，Thanks♪(･ω･)ﾉ。**