---
title: 'Vue3的defineEmits、defineProps结合TS' # 标题
date: '2023/2/15' # 发布时间
lastmod: '2022/3/10'
tags: [Vue3] # 标签
draft: false
summary: ''
images: # 文章封面 必须要
  [
    'https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a649bcaab497486a91335767525b8201~tplv-k3u1fbpfcp-zoom-crop-mark:1512:1512:1512:851.awebp?',
  ]
layout: PostLayout
---

废话不多说，直接进入操作环节

## defineEmits

```ts
<div  @click="handleItemClick(item)">...</div>

// 1.定义暴露出去的方法
const emits = defineEmits<{
  (e: 'itemClick', item: ICategorys): void
}>()

// 2.handleItemClick方法触发时通过emits把itemClick暴露出去
function handleItemClick(item: ICategorys) {
  emits('itemClick', item)
}
```

## defineProps 与 withDefaults

```ts
<div class="category-item"
    v-for="(item, index) in categorys"
    :key="item.id"
    @click="handleItemClick(item)">
</div>

import { ICategorys } from '@/store/home';

// 1.定义props类型
export interface IProps {
  categorys?: ICategorys[],
}

// 2.定义默认值
const props = withDefaults(defineProps<IProps>(), {
  categorys: () => []
})
```
