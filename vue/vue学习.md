# 从零开始学VUE #

## 1. 初识vue

### Vue是一个渐进式的框架，什么是渐进式的呢？

+ 渐进式意味着你可以将Vue作为你应用的一部分嵌入其中，带来更丰富的交互体验
+ 或者如果你希望将更多的业务逻辑使用Vue实现，那么Vue的核心库以及其生态系统
+ 比如Core+Vue-router+Vuex，也可以满足你各种各样的需求

###  Vue的特点和Web开发中常见的高级功能

- 解耦视图和数据-->[MVVM](https://www.cnblogs.com/wzfwaf/p/10553160.html)
- 可复用的组件-->components
- 前端路由技术-->[router](https://router.vuejs.org/zh/installation.html)
- 状态管理-->[vuex](https://vuex.vuejs.org/zh/)
- 虚拟DOM-->[mastache](https://blog.csdn.net/weixin_43690348/article/details/113113439)

## 2. vue.js 起飞

### 安装

+ cdn引入

  ``` js
  <!-- 开发环境版本，包含了有帮助的命令行警告 -->
  <script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
  <!-- 生产环境版本，优化了尺寸和速度 -->
  <script src="https://cdn.jsdelivr.net/npm/vue@2"></script>
  ```
### 声明式渲染
``` vue
<template>
	<div id="app">
      {{ message }}
	</div>
</template>
<script>
	var app = new Vue({
      el: '#app',
          data: {
            message: 'Hello Vue!'
          }
	  })
</script>
```

