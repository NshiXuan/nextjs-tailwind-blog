---
title: 'text-align的居中与justify详解' # 标题
date: '2023/4/10' # 发布时间
# lastmod: '2022/3/10'
tags: [CSS] # 标签
draft: false
summary: 'text-align的居中与justify详解'
images: # 文章封面 不要就留个''空字符串
  [
    'https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0f92344366c14b5099ff522ee698209e~tplv-k3u1fbpfcp-zoom-crop-mark:1512:1512:1512:851.awebp?',
  ]
layout: PostLayout
---

## center 居中对齐

`text-align` 可以对文本、图片进行居中显示，那么它对 `div` 盒子可以居中显示吗？它到底对什么元素才能居中？

`text-align: center` 对行内级盒子相对于它的块级父元素居中对齐

```html
<style>
  .box {
    background-color: #f00;
    height: 300px;
    text-align: center;
  }

  .content {
    background-color: #0f0;
    height: 200px;
    width: 200px;

    /* 转成行内块 */
    display: inline-block;
  }
</style>

<div class="box">
  <div class="content"></div>
</div>
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cc54d0f66c854709a525048aec1e6c79~tplv-k3u1fbpfcp-watermark.image?)

## justify 两端对齐

`justify` 对最后一行没有用 如果希望最后一个有用需要使用 `text-align-last`

```html
<style>
  .box {
    width: 200px;
    background-color: red;
    color: #fff;
    text-align: justify;

    /* justify对最后一行没有用 如果希望最后一个有用需要使用text-align-last */
    text-align-last: justify;
  }
</style>

<div class="box">
  Container queries enable you to apply styles to an element based on the size of the element's
  container. If, for example
</div>
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/72d22e3fdfeb491abb6e88a569593f58~tplv-k3u1fbpfcp-watermark.image?)

- 添加 `text-align-last` 后

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a72f7eac70ba4021b3b98c1b89561a92~tplv-k3u1fbpfcp-watermark.image?)
