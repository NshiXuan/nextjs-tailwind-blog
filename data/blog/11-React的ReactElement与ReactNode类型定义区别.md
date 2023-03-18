---
title: 'React的ReactElement与ReactNode类型定义区别' # 标题
date: '2023/3/18' # 发布时间
# lastmod: '2022/3/10'
tags: [React] # 标签
draft: false
summary: 'React的ReactElement与ReactNode类型定义区别'
images: # 文章封面 必须要
  [
    'https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/569ca2d5651844fb8001a3df9e71ee08~tplv-k3u1fbpfcp-zoom-crop-mark:1512:1512:1512:851.awebp?',
  ]
layout: PostLayout
---

## _ReactElement 与 ReactNode_

- 子组件的 `children` 为 `ReactNode` 类型， `children` 可以提供多个子元素
- 子组件的 `children` 为 _`ReactElement`_ 类型， `children` 只能有一个子元素
- 所以子组件的 `children` 推荐使用 `ReactNode` 类型

```ts
...

export interface IProps {
  children?: ReactNode
}

const ScrollView: FC<IProps> = function (props) {
  const { children } = props

  return (
    <div className=" overflow-hidden relative">
        {children}
    </div>
  )
}
```

- 如果 `children` 为 _`ReactElement`_ 类型报错， `ReactNode` 类型不报错

```ts
<ScrollView>
  <div>123</div>
  <div>123</div>
</ScrollView>
```
