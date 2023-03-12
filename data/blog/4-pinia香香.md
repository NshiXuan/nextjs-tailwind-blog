---
title: 'pinia香香' # 标题
date: '2023/3/15' # 发布时间
# lastmod: '2022/3/10'
tags: [vue3, pinia] # 标签
draft: false
summary: 'pinia香香'
images: # 文章封面 必须要
  [
    'https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/608cf18a1257486594b8538dfd00a0ae~tplv-k3u1fbpfcp-zoom-crop-mark:1512:1512:1512:851.awebp?',
  ]
layout: PostLayout
---

官网：[Installation | Pinia (vuejs.org)](https://pinia.vuejs.org/getting-started.html)

## 安装

```
yarn add pinia
# 或者
npm install pinia
```

## 基本使用

在 `main.js` 中引入 `pinia`

```ts
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'

const app = createApp(App)

app.use(createPinia())

app.mount('#app')
```

在 `src` 文件夹下创建 `counter` 文件

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c5da28cf5e60434e9bdf33d5ab4eb8cc~tplv-k3u1fbpfcp-watermark.image?style="zoom:33%;")

```ts
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', {
  state: () => {
    return { count: 0 }
  },
  actions: {
    increment() {
      this.count++
    },
  },
})
```

在组件中使用

- 直接从 `counter` 拿的值是响应式的，中可以直接修改
- 如果是解构出来的值，不是响应式的，需要转成响应式，可以使用 `toRefs` 或者 `Pinia` 提供的 `storeToRefs`
- `pinia` 提供了一个重置的方法 `$reset` ，调用可以重置为 `store` 中的内容

```ts
<template>
  <div>{{counter.count}}</div>
  <div>{{count}}</div>
  <button @click="counter.increment">+1</button>
  <button @click="increment">+1</button>
  <button @click="counter.$reset">重置</button>
</template>

<script setup>
import { storeToRefs } from 'pinia';
import { toRefs } from 'vue';
import { useCounterStore } from './stores/counter';

const counter = useCounterStore()

const increment = () => {
  counter.count++
}

// const { count } = storeToRefs(counter)
const { count } = toRefs(counter)
</script>
```

## State

`$patch` 修改多个 `State` 数据

- `store`

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8322f3b636c44141bcb24615b5f58d06~tplv-k3u1fbpfcp-watermark.image?)

```ts
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', {
  state: () => {
    return {
      count: 0,
      name: 'zs',
    }
  },
})
```

- 使用 `$patch`

```ts
<template>
  <div>{{counter.count}}</div>
  <div>{{counter.name}}</div>
  <button @click="changeState">修改</button>
</template>

<script setup>
import { useCounterStore } from './stores/counter';

const changeState = () => {
  counter.$patch({ count: 100, name: 'ls' })
}
</script>
```

`$reset` 重置

```ts
<template>
  <div>{{counter.count}}</div>
  <button @click="counter.increment">+1</button>
  <button @click="counter.$reset">重置</button>
</template>

<script setup>
import { useCounterStore } from './stores/counter';

const counter = useCounterStore()
</script>
```

`$state`

## Getters

`store`

```ts
import { defineStore } from 'pinia'
import { useUserStore } from './user';

export const useCounterStore = defineStore('counter', {
  state: () => {
    return { count: 0 }
  },
  getters: {
    // 1.直接使用
    doubleCount: (state) => state.count * 2,
    // 2.getter中使用getter
    / 使用this的时候，不能使用箭头函数
    doubleCountAddOne() {
      return this.doubleCount + 1
    },
    // 3.getter也支持返回一个函数
    getFriendById: (state) => {
      return (id) => {
        for (const friend of state.friends) {
          if (id === friend.id) return friend
        }
      }
    },
    // 4.getter中使用到别的store中的数据
    showMessage: () => {
      const user = useCounterStore()
      ...
    }
  }
})
```

在组件中使用

```ts
<template>
  <div>{{counter.count}}</div>
  <div>{{counter.doubleCount}}</div>
  <div>doubleCountAddOne: {{counter.doubleCountAddOne}}</div>
  <!-- 111会传递给id -->
  <div>friends: {{counter.getFriendById(111)}}</div>
</template>

<script setup>
import { useCounterStore } from './stores/counter';

const counter = useCounterStore()
</script>
```

## Actions

### 基本使用

在 `store` 中定义 `actions`

```ts
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', {
  state: () => {
    return {
      count: 0,
    }
  },
  actions: {
    increment() {
      this.count++
    },
    // 使用this时不能使用箭头函数
    incrementNum(payload) {
      this.count += payload
    },
  },
})
```

`actions` 中的函可以直接调用 `xxx.increment`

```ts
<template>
  <div>{{counter.count}}</div>
  <button @click="counter.increment">+1</button>
  <button @click="counter.incrementNum(5)">+5</button>
</template>

<script setup>
import { useCounterStore } from './stores/counter';

const counter = useCounterStore()
</script>
```

### 请求数据

```ts
import { defineStore } from 'pinia'

export const useHomeStore = defineStore('home', {
  state: () => ({
    banners: [],
    recommends: [],
  }),
  actions: {
    async fetchHomeMultidata() {
      const res = await fetch('<http://123.207.32.32:8000/home/multidata>')
      // fetch这里才会发送真正的网络请求
      const data = await res.json()
      this.banners = data.data.banner.list
      this.recommends = data.data.recommend.list
    },
  },
})
```

展示数据

```ts
<template>
  <li v-for="(item,index) in home.banners">{{item.title}}</li>
</template>

<script setup>
import { useHomeStore } from './stores/home';

const home = useHomeStore()
home.fetchHomeMultidata()
</script>
```

## storeToRefs

从 `store` 中解构出来的数据不是响应式的，需要使用 `storeToRefs` 或者 `ref` 让它变为响应式，如下的 `categories`

```ts
<template>
  <div class="categories">
    <template v-for="(item,index) in categories">
      <div class="item">
        <img :src="item.pictureUrl" alt="">
        <div class="text">{{item.title}}</div>
      </div>
    </template>
  </div>
</template>

<script setup>
import { storeToRefs } from 'pinia';
import useHomeStore from '../../../stores/modules/home';

const homeStore = useHomeStore()
// 将解构出来的数据变为响应式数据
const { categories } = storeToRefs(homeStore)

// 从storeToRefs拿到的数据是一个ref 在script中使用需要使用.value
const copyCategories = categories.value

// 修改
homeStore.categories = []
</script>
```
