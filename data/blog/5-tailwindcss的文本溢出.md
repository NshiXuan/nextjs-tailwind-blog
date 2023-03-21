---
title: 'tailwindcss的文本溢出' # 标题
date: '2023/3/15' # 发布时间
# lastmod: '2022/3/10'
tags: [tailwindcss] # 标签
draft: false
summary: 'tailwindcss的文本溢出'
images: # 文章封面 必须要
  [
    'https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fb81176f617e46c486660bb9fda59fa4~tplv-k3u1fbpfcp-zoom-crop-mark:1512:1512:1512:851.awebp?',
  ]
layout: PostLayout
---

## 单行文本溢出

如果只定义一行文本溢出，直接使用 `tailwindcss` 定义的类即可

```ts
<div className="truncate">{itemData.name}</div>
```

## 多行文本溢出

安装插件

```css
pnpm add @tailwindcss/line-clamp
```

- 在 `tailwind.config.js` 中导入插件

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  ...
  plugins: [require('@tailwindcss/line-clamp')]
}
```

- 使用

```ts
<div className="line-clamp-2">{itemData.name}</div>
```
